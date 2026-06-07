
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="theme-color" content="#3b82f6">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <link id="manifest-link" rel="manifest" href="">
    <title>IMSLP 간편 검색기</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;500;700&display=swap');

        body {
            font-family: 'Noto Sans KR', sans-serif;
            background-color: #f3f4f6;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 1rem;
        }

        .search-container {
            background: white;
            padding: 1.5rem;
            border-radius: 1.5rem;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.05);
            width: 100%;
            max-width: 800px;
            text-align: center;
        }

        @media (min-width: 640px) {
            .search-container {
                padding: 3rem;
            }
        }

        .search-input-wrapper {
            position: relative;
            margin-top: 2rem;
            margin-bottom: 1.5rem;
        }

        .search-input {
            width: 100%;
            padding: 1.25rem 1.25rem 1.25rem 3.5rem;
            font-size: 1.125rem;
            border: 2px solid #e5e7eb;
            border-radius: 9999px;
            outline: none;
            transition: all 0.3s ease;
        }

        .search-input:focus {
            border-color: #3b82f6;
            box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.2);
        }

        .search-icon {
            position: absolute;
            left: 1.25rem;
            top: 50%;
            transform: translateY(-50%);
            color: #9ca3af;
            font-size: 1.25rem;
        }

        .search-button {
            background-color: #3b82f6;
            color: white;
            font-weight: 600;
            padding: 1rem 2.5rem;
            border-radius: 9999px;
            border: none;
            cursor: pointer;
            font-size: 1.125rem;
            transition: background-color 0.3s ease, transform 0.1s ease;
        }

        .search-button:hover {
            background-color: #2563eb;
        }

        .search-button:active {
            transform: scale(0.98);
        }

        .examples {
            margin-top: 2rem;
            color: #6b7280;
            font-size: 0.875rem;
        }

        .example-tag {
            display: inline-block;
            background-color: #f3f4f6;
            padding: 0.25rem 0.75rem;
            border-radius: 9999px;
            margin: 0.25rem;
            cursor: pointer;
            transition: background-color 0.2s;
        }

        .example-tag:hover {
            background-color: #e5e7eb;
            color: #374151;
        }

        /* 요약 결과창 마크다운 스타일 */
        .result-content { text-align: left; line-height: 1.6; font-size: 0.95rem; }
        .result-content h1, .result-content h2, .result-content h3 { font-weight: 700; color: #1f2937; margin-top: 1.5rem; margin-bottom: 0.75rem; }
        .result-content h1 { font-size: 1.25rem; border-bottom: 1px solid #e5e7eb; padding-bottom: 0.5rem; }
        .result-content p { color: #4b5563; margin-bottom: 0.75rem; }
        .result-content ul { list-style-type: disc; padding-left: 1.5rem; margin-bottom: 0.75rem; color: #4b5563; }
        .result-content strong { color: #111827; }
    </style>
</head>
<body>

    <div class="search-container">
        <!-- PWA 설치 버튼 -->
        <div id="installContainer" class="hidden mb-4 text-right">
            <button id="installBtn" class="bg-gray-800 text-white text-sm px-4 py-2 rounded-full font-medium shadow-md hover:bg-gray-700 transition">
                <i class="fas fa-download mr-2"></i>앱으로 설치하기
            </button>
        </div>

        <h1 class="text-3xl font-bold text-gray-800 mb-2">클래식 악보 검색</h1>
        <p class="text-gray-500 mb-6">작곡가와 곡명을 간략히 입력하면 IMSLP 페이지로 연결합니다.</p>

        <form id="searchForm" onsubmit="handleSearch(event)">
            <div class="search-input-wrapper">
                <i class="fas fa-search search-icon"></i>
                <input type="text" id="searchInput" class="search-input" placeholder="예: 쇼팽 녹턴 9-2" required autocomplete="off">
            </div>
            <button type="submit" class="search-button">IMSLP에서 찾기</button>
        </form>

        <!-- 로딩 표시기 -->
        <div id="loadingIndicator" class="hidden mt-8">
            <i class="fas fa-circle-notch fa-spin text-4xl text-blue-500 mb-3"></i>
            <p class="text-gray-500 font-medium animate-pulse">인공지능이 악보 정보를 찾아 요약하는 중입니다...</p>
        </div>

        <!-- 검색 결과 영역 -->
        <div id="resultsArea" class="hidden mt-8 pt-8 border-t border-gray-100 text-left">
            <h2 class="text-xl font-bold text-gray-800 mb-4"><i class="fas fa-list-alt text-blue-500 mr-2"></i>검색 요약</h2>
            <div id="summaryText" class="result-content bg-gray-50 p-6 rounded-xl mb-6 shadow-inner border border-gray-100"></div>
            <div id="linksArea"></div>
        </div>

        <div class="examples" id="examplesArea">
            <p class="mb-2">검색 예시 (클릭하여 입력)</p>
            <div>
                <span class="example-tag" onclick="setExample('beethoven symphony 9')">베토벤 교향곡 9번</span>
                <span class="example-tag" onclick="setExample('chopin ballade 1')">쇼팽 발라드 1번</span>
                <span class="example-tag" onclick="setExample('mozart requiem')">모차르트 레퀴엠</span>
                <span class="example-tag" onclick="setExample('liszt la campanella')">리스트 라 캄파넬라</span>
            </div>
        </div>
    </div>

    <script>
        // --- PWA (Progressive Web App) 동적 설정 ---
        (function initPWA() {
            const iconSvg = '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512"><rect width="512" height="512" rx="100" fill="#3b82f6"/><path d="M220 350a50 50 0 1 1-50-50v-150l150-30v180a50 50 0 1 1-50-50v-110l-100 20z" fill="#fff"/></svg>';
            const iconDataUrl = `data:image/svg+xml;charset=utf-8,${encodeURIComponent(iconSvg)}`;

            const manifest = {
                name: "IMSLP 간편 검색기",
                short_name: "IMSLP검색",
                description: "클래식 악보를 간편하게 검색하고 요약해줍니다.",
                start_url: ".",
                display: "standalone",
                background_color: "#f3f4f6",
                theme_color: "#3b82f6",
                icons: [
                    { src: iconDataUrl, sizes: "192x192", type: "image/svg+xml", purpose: "any maskable" },
                    { src: iconDataUrl, sizes: "512x512", type: "image/svg+xml", purpose: "any maskable" }
                ]
            };
            const manifestBlob = new Blob([JSON.stringify(manifest)], { type: 'application/json' });
            document.getElementById('manifest-link').href = URL.createObjectURL(manifestBlob);

            const swCode = `
                self.addEventListener('install', (e) => self.skipWaiting());
                self.addEventListener('activate', (e) => e.waitUntil(clients.claim()));
                self.addEventListener('fetch', (e) => {
                    e.respondWith(fetch(e.request).catch(() => new Response('오프라인 상태입니다. 인터넷 연결을 확인해주세요.')));
                });
            `;
            const swBlob = new Blob([swCode], { type: 'application/javascript' });
            if ('serviceWorker' in navigator) {
                navigator.serviceWorker.register(URL.createObjectURL(swBlob))
                    .catch(err => console.log('Service Worker 등록 실패:', err));
            }

            let deferredPrompt;
            const isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);
            if (isMobile) {
                document.getElementById('installContainer').classList.remove('hidden');
            }

            window.addEventListener('beforeinstallprompt', (e) => {
                e.preventDefault();
                deferredPrompt = e;
                document.getElementById('installContainer').classList.remove('hidden');
            });

            document.getElementById('installBtn').addEventListener('click', async () => {
                if (deferredPrompt) {
                    deferredPrompt.prompt();
                    const { outcome } = await deferredPrompt.userChoice;
                    if (outcome === 'accepted') {
                        document.getElementById('installContainer').classList.add('hidden');
                    }
                    deferredPrompt = null;
                } else {
                    alert("자동 설치가 차단되었거나 지원되지 않는 환경입니다.\\n\\n[수동으로 홈 화면에 추가하는 방법]\\n\\n🍎 아이폰 (Safari): 하단 '공유(내보내기)' 아이콘 ➔ '홈 화면에 추가'\\n🤖 안드로이드 (Chrome): 우측 상단 '⋮' 메뉴 ➔ '홈 화면에 추가' 또는 '앱 설치'");
                }
            });
        })();

        // --- 재시도 로직을 포함한 API 호출 함수 ---
        async function fetchWithRetry(url, options, retries = 5) {
            const delays = [1000, 2000, 4000, 8000, 16000];
            for (let i = 0; i < retries; i++) {
                try {
                    const response = await fetch(url, options);
                    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
                    return await response.json();
                } catch (error) {
                    if (i === retries - 1) throw error;
                    await new Promise(resolve => setTimeout(resolve, delays[i]));
                }
            }
        }

        // --- 검색 처리 함수 ---
        async function handleSearch(event) {
            event.preventDefault(); 
            
            const query = document.getElementById('searchInput').value.trim();
            if (!query) return;

            document.getElementById('loadingIndicator').classList.remove('hidden');
            document.getElementById('resultsArea').classList.add('hidden');
            document.getElementById('examplesArea').classList.add('hidden');

            const apiKey = ""; 
            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;

            const payload = {
                contents: [{ parts: [{ text: `"${query}" 클래식 악보에 대한 정보를 IMSLP에서 찾아서 요약해줘.` }] }],
                tools: [{ google_search: {} }],
                systemInstruction: {
                    parts: [{ text: "당신은 클래식 악보 검색 도우미입니다. 구글 검색을 사용해 사용자의 요청과 관련된 imslp.org 웹페이지를 찾으세요. 곡의 정식 명칭, 작곡가, 작품 번호(Opus), 주요 악기 편성 등을 짧고 명확하게 한국어로 요약해 주세요. 마크다운 형식을 사용하여 보기 좋게 작성하세요." }]
                }
            };

            try {
                const result = await fetchWithRetry(url, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                const text = result.candidates?.[0]?.content?.parts?.[0]?.text;
                const sources = result.candidates?.[0]?.groundingMetadata?.groundingAttributions?.map(a => ({ uri: a.web?.uri, title: a.web?.title })) || [];

                if (!text) throw new Error("결과 생성에 실패했습니다.");

                // 마크다운 렌더링
                document.getElementById('summaryText').innerHTML = marked.parse(text);
                
                // IMSLP 관련 링크 필터링
                const imslpLinks = sources.filter(s => s.uri && s.uri.toLowerCase().includes('imslp.org'));
                
                if (imslpLinks.length > 0) {
                    // 중복 제거
                    const uniqueLinks = Array.from(new Map(imslpLinks.map(item => [item.uri, item])).values());
                    
                    const linksHtml = uniqueLinks.map(s => `
                        <a href="${s.uri}" target="_blank" class="flex items-center p-4 bg-blue-50 text-blue-700 rounded-xl hover:bg-blue-100 transition duration-200 mb-3 shadow-sm border border-blue-100">
                            <i class="fas fa-file-pdf text-2xl mr-4 text-red-500"></i>
                            <div class="truncate text-left" style="width: 100%;">
                                <p class="font-bold truncate text-sm">${s.title || 'IMSLP 페이지로 이동'}</p>
                                <p class="text-xs text-blue-500 truncate mt-1">${s.uri}</p>
                            </div>
                            <i class="fas fa-chevron-right ml-auto pl-2"></i>
                        </a>
                    `).join('');
                    
                    document.getElementById('linksArea').innerHTML = `
                        <h3 class="font-bold text-gray-700 mb-3 mt-2"><i class="fas fa-link mr-2 text-gray-500"></i>바로가기 링크</h3>
                        ${linksHtml}
                    `;
                } else {
                    document.getElementById('linksArea').innerHTML = `
                        <p class="text-gray-500 text-sm p-4 bg-gray-50 rounded-lg border border-gray-200"><i class="fas fa-info-circle mr-2"></i>직접적인 IMSLP 악보 링크를 찾지 못했습니다. 곡명이 정확한지 확인해주세요.</p>
                    `;
                }
                
            } catch (error) {
                console.error("Search API Error:", error);
                document.getElementById('summaryText').innerHTML = `
                    <div class="text-red-500 flex items-center p-4 bg-red-50 rounded-lg">
                        <i class="fas fa-exclamation-triangle text-2xl mr-3"></i>
                        <div>
                            <strong>검색 중 오류가 발생했습니다.</strong><br>
                            <span class="text-sm">잠시 후 다시 시도해주세요. (${error.message})</span>
                        </div>
                    </div>`;
                document.getElementById('linksArea').innerHTML = '';
            } finally {
                document.getElementById('loadingIndicator').classList.add('hidden');
                document.getElementById('resultsArea').classList.remove('hidden');
            }
        }

        function setExample(text) {
            const input = document.getElementById('searchInput');
            input.value = text;
            input.focus();
        }
    </script>
</body>
</html>
