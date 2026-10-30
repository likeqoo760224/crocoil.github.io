<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>數獨自訂題庫與邏輯推演系統</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            user-select: none;
            -webkit-user-select: none;
            -webkit-tap-highlight-color: transparent;
            touch-action: manipulation; /* 防止手機快速連點時畫面意外放大 */
            overscroll-behavior-y: contain;
        }

        /* 數獨 9x9 主網格與 3x3 粗邊框設計 */
        .sudoku-grid {
            display: grid;
            grid-template-columns: repeat(9, minmax(0, 1fr));
            grid-template-rows: repeat(9, minmax(0, 1fr));
            aspect-ratio: 1 / 1;
            max-height: min(88vw, 430px);
            margin: 0 auto;
            border: 2.5px solid #0f172a;
            border-radius: 0.75rem;
            overflow: hidden;
            background-color: #ffffff;
            box-shadow: 0 8px 22px -6px rgba(15, 23, 42, 0.14);
            touch-action: manipulation;
        }

        .sudoku-cell {
            position: relative;
            display: flex;
            align-items: center;
            justify-content: center;
            border-right: 1px solid #cbd5e1;
            border-bottom: 1px solid #cbd5e1;
            cursor: pointer;
            transition: background-color 0.08s ease;
        }

        /* 每逢第 3、6 欄加粗右邊框 */
        .sudoku-cell.border-r-thick {
            border-right: 2.5px solid #0f172a;
        }
        /* 第 9 欄移除右邊框 */
        .sudoku-cell.border-r-none {
            border-right: none;
        }
        /* 每逢第 3、6 列加粗下邊框 */
        .sudoku-cell.border-b-thick {
            border-bottom: 2.5px solid #0f172a;
        }
        /* 第 9 列移除下邊框 */
        .sudoku-cell.border-b-none {
            border-bottom: none;
        }

        /* 同 3x3 九宮格區塊柔和底色 */
        .cell-box-highlight {
            background-color: #f1f5f9 !important;
        }

        /* 直式與橫式顯眼高亮樣式 */
        .cell-cross-highlight {
            background-color: #fef08a !important; /* 鮮明暖黃色橫直高亮 */
        }
        .cell-cross-highlight::after {
            content: '';
            position: absolute;
            inset: 0;
            border: 1px solid rgba(234, 179, 8, 0.35);
            pointer-events: none;
        }

        /* 相同數字高亮 */
        .cell-same-number {
            background-color: #bfdbfe !important; /* 亮藍色提示同數字 */
        }

        /* 邏輯推演目標格高亮 */
        .cell-hint-target {
            background-color: #bbf7d0 !important;
            box-shadow: inset 0 0 0 3px #16a34a !important;
            animation: pulseHint 1.5s infinite;
        }
        /* 邏輯推演關聯排除格高亮 */
        .cell-hint-reason {
            background-color: #ddd6fe !important;
            box-shadow: inset 0 0 0 2px #7c3aed !important;
        }

        @keyframes pulseHint {
            0%, 100% { box-shadow: inset 0 0 0 3px #16a34a; }
            50% { box-shadow: inset 0 0 0 5px #22c55e; }
        }

        /* 當前選中格高亮 */
        .cell-selected {
            background-color: #fb923c !important; /* 醒目橘色代表目前游標所在格 */
            color: #ffffff !important;
            z-index: 10;
            box-shadow: inset 0 0 0 2.5px #9a3412;
        }

        /* 衝突錯誤格樣式 */
        .cell-conflict {
            background-color: #fecaca !important;
        }
        .cell-selected.cell-conflict {
            background-color: #ef4444 !important;
        }

        /* 小筆記 3x3 內部網格 */
        .notes-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            grid-template-rows: repeat(3, 1fr);
            width: 100%;
            height: 100%;
            padding: 1px;
            pointer-events: none;
        }

        .note-item {
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: clamp(0.5rem, 2.3vw, 0.75rem);
            line-height: 1;
            font-weight: 600;
            color: #475569;
        }

        .cell-selected .note-item {
            color: #ffffff;
            font-weight: 700;
        }

        .note-item.note-highlight {
            color: #1d4ed8;
            font-weight: 800;
            background-color: rgba(147, 197, 253, 0.65);
            border-radius: 2px;
        }
        .cell-selected .note-item.note-highlight {
            color: #fef08a;
            background-color: rgba(0, 0, 0, 0.25);
        }

        /* 隱藏手機橫向捲軸但保留滑動彈性 */
        .no-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .no-scrollbar {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
    </style>
</head>
<body class="bg-slate-100 min-h-[100dvh] text-slate-800 flex flex-col items-center justify-between sm:justify-start py-1.5 px-2 sm:py-5">

    <!-- 懸浮訊息通知 Toast (手機上不推擠盤面，避免點擊時畫面跳動) -->
    <div id="statusBanner" class="hidden fixed top-2.5 left-1/2 -translate-x-1/2 z-40 w-[94%] max-w-md rounded-xl px-3.5 py-2 text-xs sm:text-sm font-bold flex items-center justify-between shadow-lg transition-all">
        <span id="statusBannerText" class="leading-snug"></span>
        <button onclick="hideBanner()" class="ml-2.5 px-1.5 py-0.5 rounded bg-black/5 hover:bg-black/10 font-bold shrink-0">✕</button>
    </div>

    <!-- 主容器 -->
    <div class="w-full max-w-xl flex flex-col gap-1.5 sm:gap-2.5 my-auto sm:my-0">

        <!-- 頂部標題、難度選擇與計時統計列 (手機版緊湊化設計) -->
        <header class="bg-white rounded-2xl p-2.5 sm:p-4 shadow-xs border border-slate-200 flex flex-col gap-2">
            <div class="flex items-center justify-between gap-1.5">
                <div class="flex items-center gap-2 min-w-0">
                    <div class="w-8 h-8 sm:w-10 sm:h-10 rounded-xl bg-indigo-600 flex items-center justify-center text-white font-extrabold text-base sm:text-xl shadow-xs shrink-0">
                        9
                    </div>
                    <div class="min-w-0">
                        <div class="flex items-center gap-1.5">
                            <h1 class="text-sm sm:text-lg font-extrabold text-slate-900 truncate">數獨大師・行動推演版</h1>
                            <span id="saveIndicator" class="hidden sm:inline-block text-[11px] font-semibold px-2 py-0.5 rounded-full bg-slate-100 text-slate-500 shrink-0">準備就緒</span>
                        </div>
                        <p id="modeSubtitle" class="hidden sm:block text-xs text-slate-500 font-medium mt-0.5">支援四段難度唯一解生成、單手連續點填與逐步邏輯推演</p>
                    </div>
                </div>

                <!-- 計時器、暫停與統計按鈕整合在右上角，節省手機垂直空間 -->
                <div class="flex items-center gap-1 shrink-0">
                    <div id="timerControls" class="flex items-center gap-1">
                        <div class="px-2 py-1 rounded-lg bg-slate-100 text-slate-700 font-mono text-xs font-bold flex items-center gap-1">
                            <span>⏱️</span>
                            <span id="timerText">00:00</span>
                        </div>
                        <button id="pauseBtn" onclick="togglePauseGame()" class="px-2 py-1 rounded-lg bg-slate-800 active:bg-slate-950 text-white text-xs font-bold transition flex items-center gap-0.5">
                            <span id="pauseBtnIcon">⏸️</span>
                            <span id="pauseBtnText" class="hidden xs:inline">暫停</span>
                        </button>
                    </div>
                    <button onclick="openStatsModal()" class="px-2 py-1 rounded-lg bg-slate-100 active:bg-slate-200 text-slate-700 text-xs font-bold transition flex items-center gap-1" title="查看解題數據統計">
                        <span>📊</span>
                        <span class="hidden sm:inline">統計</span>
                    </button>
                    <span id="modeBadge" class="hidden sm:inline-block px-2 py-1 rounded-full text-[11px] font-bold bg-emerald-100 text-emerald-800 border border-emerald-300">
                        🎮 遊玩中
                    </span>
                </div>
            </div>

            <!-- 難度與模式切換列 (手機支援單行橫向滑動或自適應等寬) -->
            <div class="flex items-center justify-between gap-1 pt-1.5 border-t border-slate-100 overflow-x-auto no-scrollbar">
                <div class="flex items-center gap-1 w-full">
                    <button onclick="startNewGeneratedGame('easy')" id="diffBtn-easy" class="diff-btn flex-1 sm:flex-none px-2 py-1.5 text-xs font-bold rounded-lg transition bg-indigo-600 text-white text-center whitespace-nowrap">
                        簡單
                    </button>
                    <button onclick="startNewGeneratedGame('medium')" id="diffBtn-medium" class="diff-btn flex-1 sm:flex-none px-2 py-1.5 text-xs font-bold rounded-lg transition bg-slate-100 text-slate-700 text-center whitespace-nowrap">
                        中等
                    </button>
                    <button onclick="startNewGeneratedGame('hard')" id="diffBtn-hard" class="diff-btn flex-1 sm:flex-none px-2 py-1.5 text-xs font-bold rounded-lg transition bg-slate-100 text-slate-700 text-center whitespace-nowrap">
                        困難
                    </button>
                    <button onclick="startNewGeneratedGame('expert')" id="diffBtn-expert" class="diff-btn flex-1 sm:flex-none px-2 py-1.5 text-xs font-bold rounded-lg transition bg-slate-100 text-slate-700 text-center whitespace-nowrap">
                        專家
                    </button>
                    <button onclick="enterCustomSetupMode()" id="diffBtn-custom" class="diff-btn flex-1 sm:flex-none px-2.5 py-1.5 text-xs font-bold rounded-lg transition bg-amber-50 text-amber-800 border border-amber-200 text-center whitespace-nowrap">
                        ✏️ 自訂題目
                    </button>
                </div>
            </div>

            <!-- 出題模式專用控制列 -->
            <div id="setupControls" class="hidden flex-col gap-2 pt-2 border-t border-amber-200 bg-amber-50/70 -mx-2.5 -mb-2.5 sm:-mx-4 sm:-mb-4 p-2.5 sm:p-3.5 rounded-b-2xl">
                <div class="flex items-center justify-between gap-2">
                    <span class="text-xs font-bold text-amber-900">📝 點擊下方格子輸入題目，系統將驗證唯一解</span>
                    <button onclick="clearEntireBoard()" class="px-2.5 py-1 text-xs font-bold rounded-lg bg-white active:bg-rose-50 text-rose-600 border border-rose-200 transition shrink-0">
                        清空盤面
                    </button>
                </div>
                <button onclick="confirmPuzzleAndStart()" class="w-full py-2 px-4 rounded-xl bg-indigo-600 active:bg-indigo-700 text-white font-bold text-xs sm:text-sm shadow-xs transition flex items-center justify-center gap-1.5">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 13l4 4L19 7"></path>
                    </svg>
                    驗證題庫並正式開始遊玩
                </button>
            </div>
        </header>

        <!-- 視覺圖例說明與當前選中座標 (手機精簡單行顯示) -->
        <div class="flex items-center justify-between px-1 text-[11px] sm:text-xs text-slate-600 font-medium">
            <div class="flex items-center gap-2">
                <span class="flex items-center gap-1">
                    <span class="w-2.5 h-2.5 rounded-xs bg-orange-400 inline-block border border-orange-700"></span>選中
                </span>
                <span class="flex items-center gap-1">
                    <span class="w-2.5 h-2.5 rounded-xs bg-yellow-200 inline-block border border-yellow-400"></span>直橫列
                </span>
                <span class="flex items-center gap-1">
                    <span class="w-2.5 h-2.5 rounded-xs bg-blue-200 inline-block border border-blue-400"></span>同數
                </span>
            </div>
            <div id="coordIndicator" class="font-mono text-slate-500 font-semibold truncate">
                未選擇格子
            </div>
        </div>

        <!-- 數獨 9x9 盤面容器 (含暫停防偷看遮罩 & 邏輯推演浮層卡片) -->
        <div class="relative w-full">
            <div id="sudokuBoard" class="sudoku-grid w-full">
                <!-- 由 JS 動態生成 81 個格子 -->
            </div>

            <!-- 暫停時遮蔽整個數獨盤面 -->
            <div id="pauseOverlay" class="hidden absolute inset-0 bg-slate-900/95 backdrop-blur-md rounded-xl z-20 flex flex-col items-center justify-center text-white p-6 text-center">
                <div class="w-12 h-12 rounded-2xl bg-indigo-600/30 border border-indigo-400/40 flex items-center justify-center text-2xl mb-2">
                    ⏸️
                </div>
                <h2 class="text-lg font-extrabold mb-1">遊戲已暫停</h2>
                <p class="text-xs text-slate-300 max-w-xs mb-4">為了維持公平的解題計時挑戰，暫停期間盤面已自動遮蔽。</p>
                <button onclick="togglePauseGame()" class="px-6 py-2.5 rounded-xl bg-indigo-500 active:bg-indigo-600 text-white font-bold text-sm shadow-lg transition">
                    ▶️ 繼續解題
                </button>
            </div>
        </div>

        <!-- 邏輯推演解說面板 (緊湊卡片設計) -->
        <div id="logicHintPanel" class="hidden bg-emerald-50 border-2 border-emerald-500 rounded-2xl p-2.5 sm:p-3.5 shadow-md flex flex-col gap-1.5">
            <div class="flex items-center justify-between gap-2">
                <div class="flex items-center gap-1.5 min-w-0">
                    <span class="px-2 py-0.5 rounded-md bg-emerald-600 text-white text-[11px] font-extrabold shrink-0" id="logicTechniqueBadge">邏輯推演</span>
                    <h3 class="text-xs sm:text-sm font-extrabold text-emerald-950 truncate" id="logicHintTitle">下一步解題思路</h3>
                </div>
                <button onclick="clearLogicHintHighlight()" class="text-emerald-700 hover:text-emerald-950 font-bold text-xs px-1.5 py-0.5 shrink-0">✕ 關閉</button>
            </div>
            <p id="logicHintDesc" class="text-xs text-emerald-900 leading-relaxed font-medium"></p>
            <div class="flex items-center justify-end pt-0.5">
                <button id="applyLogicStepBtn" onclick="applyCurrentLogicHint()" class="w-full sm:w-auto px-3 py-1.5 rounded-xl bg-emerald-600 active:bg-emerald-700 text-white text-xs font-bold shadow-xs transition">
                    ✨ 直接填入此步答案
                </button>
            </div>
        </div>

        <!-- 手機人體工學操作面板：大按鈕工具列 + 單手連續點填模式 + 高觸控數字鍵盤 -->
        <div class="bg-white rounded-2xl p-2.5 sm:p-3 shadow-xs border border-slate-200 flex flex-col gap-2">
            
            <!-- 數字鍵盤與「剩餘還沒使用的數字」統計區 (移至上方更靠近拇指熱區) -->
            <div>
                <div class="flex items-center justify-between mb-1.5 px-0.5">
                    <button id="numLockBtn" onclick="toggleNumberLockMode()" class="px-2.5 py-1 rounded-lg text-xs font-bold border transition flex items-center gap-1 bg-slate-50 border-slate-200 text-slate-700">
                        <span id="numLockIcon">🔓</span>
                        <span id="numLockText">先選格再填字 (點擊切換固定數字連填)</span>
                    </button>
                    <span id="remainingSummaryText" class="text-xs font-bold text-indigo-600 shrink-0">尚有 81 格</span>
                </div>

                <!-- 1 ~ 9 大觸控按鈕列 -->
                <div id="numpadContainer" class="grid grid-cols-9 gap-1 sm:gap-1.5">
                    <!-- 由 JS 動態生成 1-9 按鈕與剩餘數量 -->
                </div>
            </div>

            <!-- 輔助工具列：手機加大觸控面積 (5 欄圖示上字下結構) -->
            <div class="grid grid-cols-5 gap-1.5 pt-1 border-t border-slate-100">
                <!-- 小筆記切換按鈕 -->
                <button id="noteModeBtn" onclick="toggleNoteMode()" class="py-2 px-1 rounded-xl font-bold text-xs border-2 transition flex flex-col items-center justify-center gap-0.5 bg-slate-50 border-slate-200 text-slate-700 active:scale-95">
                    <div class="flex items-center gap-1">
                        <span class="text-sm">✏️</span>
                        <span id="noteStatusText" class="px-1 py-0.2 rounded text-[10px] bg-slate-200 text-slate-600">關</span>
                    </div>
                    <span class="text-[11px]">小筆記</span>
                </button>

                <!-- 橡皮擦 -->
                <button onclick="eraseSelectedCell()" class="py-2 px-1 rounded-xl font-semibold text-xs bg-slate-50 active:bg-slate-200 border border-slate-200 text-slate-700 transition flex flex-col items-center justify-center gap-0.5 active:scale-95">
                    <span class="text-sm">🧹</span>
                    <span class="text-[11px]">橡皮擦</span>
                </button>

                <!-- 復原 Undo -->
                <button id="undoBtn" onclick="undoAction()" class="py-2 px-1 rounded-xl font-semibold text-xs bg-slate-50 active:bg-slate-200 border border-slate-200 text-slate-700 transition flex flex-col items-center justify-center gap-0.5 disabled:opacity-35 disabled:pointer-events-none active:scale-95">
                    <span class="text-sm">↩️</span>
                    <span class="text-[11px]">復原</span>
                </button>

                <!-- 重做 Redo -->
                <button id="redoBtn" onclick="redoAction()" class="py-2 px-1 rounded-xl font-semibold text-xs bg-slate-50 active:bg-slate-200 border border-slate-200 text-slate-700 transition flex flex-col items-center justify-center gap-0.5 disabled:opacity-35 disabled:pointer-events-none active:scale-95">
                    <span class="text-sm">↪️</span>
                    <span class="text-[11px]">重做</span>
                </button>

                <!-- 自動計算全部候選筆記 -->
                <button onclick="autoFillAllNotes()" class="py-2 px-1 rounded-xl font-semibold text-xs bg-slate-50 active:bg-indigo-100 border border-slate-200 text-indigo-700 transition flex flex-col items-center justify-center gap-0.5 active:scale-95">
                    <span class="text-sm">🔢</span>
                    <span class="text-[11px]">全盤候選</span>
                </button>
            </div>

            <!-- 邏輯推演與快速提示列 -->
            <div id="hintToolbar" class="grid grid-cols-3 gap-1.5">
                <button onclick="requestLogicalHint()" class="col-span-2 py-2.5 px-3 rounded-xl font-bold text-xs sm:text-sm bg-emerald-600 active:bg-emerald-700 text-white shadow-xs transition flex items-center justify-center gap-1.5 active:scale-[0.99]">
                    <span>🧠</span>
                    <span>邏輯推演提示 (下一步看哪裡)</span>
                </button>
                <button onclick="fillDirectHint()" class="py-2.5 px-2 rounded-xl font-bold text-xs bg-amber-50 active:bg-amber-100 text-amber-800 border border-amber-200 transition flex items-center justify-center gap-1 active:scale-95">
                    <span>💡</span>
                    <span>直接填一格</span>
                </button>
            </div>
        </div>

        <!-- 快捷鍵小提示 (僅在電腦寬螢幕顯示，節省手機高度) -->
        <footer class="hidden sm:block text-center text-xs text-slate-400 pb-2">
            💡 鍵盤操作：方向鍵移動、<kbd class="px-1.5 py-0.5 bg-white border rounded text-slate-600">1-9</kbd> 填字、<kbd class="px-1.5 py-0.5 bg-white border rounded text-slate-600">N</kbd> 切換筆記、<kbd class="px-1.5 py-0.5 bg-white border rounded text-slate-600">Z / Y</kbd> 復原與重做、<kbd class="px-1.5 py-0.5 bg-white border rounded text-slate-600">H</kbd> 邏輯推演
        </footer>
    </div>

    <!-- 勝利結算與數據統計彈跳視窗 (Modal) -->
    <div id="statsModal" class="hidden fixed inset-0 bg-slate-900/60 backdrop-blur-xs z-50 flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl max-w-md w-full p-5 shadow-2xl border border-slate-200 flex flex-col gap-4">
            <div class="flex items-center justify-between border-b border-slate-100 pb-3">
                <div class="flex items-center gap-2">
                    <span class="text-2xl" id="modalIcon">🏆</span>
                    <div>
                        <h2 class="text-lg font-extrabold text-slate-900" id="modalTitle">解題數據統計</h2>
                        <p class="text-xs text-slate-500" id="modalSubtitle">各難度最佳紀錄與累計完成局數</p>
                    </div>
                </div>
                <button onclick="closeStatsModal()" class="w-8 h-8 rounded-lg bg-slate-100 hover:bg-slate-200 text-slate-600 font-bold">✕</button>
            </div>

            <!-- 本局過關祝賀區塊 (僅在過關時顯示) -->
            <div id="victorySummaryBox" class="hidden bg-emerald-50 border border-emerald-200 rounded-xl p-3.5 text-center">
                <p class="text-sm font-extrabold text-emerald-900" id="victoryDetailText">恭喜完成數獨挑戰！</p>
            </div>

            <!-- 四大難度統計表格 -->
            <div class="overflow-hidden rounded-xl border border-slate-200">
                <table class="w-full text-left border-collapse text-xs sm:text-sm">
                    <thead>
                        <tr class="bg-slate-50 text-slate-600 border-b border-slate-200 font-bold">
                            <th class="py-2.5 px-3">難度</th>
                            <th class="py-2.5 px-3 text-center">完成局數</th>
                            <th class="py-2.5 px-3 text-center">最佳時間</th>
                            <th class="py-2.5 px-3 text-center">平均時間</th>
                        </tr>
                    </thead>
                    <tbody id="statsTableBody" class="divide-y divide-slate-100 font-medium text-slate-700">
                        <!-- 由 JS 渲染 -->
                    </tbody>
                </table>
            </div>

            <div class="flex items-center justify-end gap-2 pt-1">
                <button onclick="closeStatsModal()" class="px-4 py-2 rounded-xl bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold transition">
                    關閉視窗
                </button>
                <button onclick="closeStatsModal(); startNewGeneratedGame(currentDifficulty === 'custom' ? 'easy' : currentDifficulty);" class="px-4 py-2 rounded-xl bg-indigo-600 hover:bg-indigo-700 text-white text-xs font-bold transition">
                    再玩一局
                </button>
            </div>
        </div>
    </div>

    <!-- 雲端自動存檔與數據同步模組 (Firestore) -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        let db = null;
        let auth = null;
        let currentUser = null;
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'sudoku-master-app';

        async function initCloudStorage() {
            try {
                if (typeof __firebase_config === 'undefined' || !__firebase_config) return;
                const firebaseConfig = JSON.parse(__firebase_config);
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                // RULE 3: Auth Before Queries
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    const cred = await signInWithCustomToken(auth, __initial_auth_token);
                    currentUser = cred.user;
                } else {
                    const cred = await signInAnonymously(auth);
                    currentUser = cred.user;
                }

                if (!currentUser) return;

                // 讀取統計資料與上次未完成的進度
                await loadCloudStats();
                await loadSavedGameFromCloud();
            } catch (err) {
                console.warn("Cloud sync unavailable, running in local session mode.");
            }
        }

        async function saveGameToCloud(serializedState) {
            if (!db || !currentUser) return;
            try {
                const saveRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'sudokuData', 'savedGame');
                await setDoc(saveRef, {
                    stateJson: JSON.stringify(serializedState),
                    updatedAt: Date.now()
                });
                const ind = document.getElementById('saveIndicator');
                if (ind) ind.textContent = '☁️ 已自動存檔';
            } catch (e) {
                // Ignore transient errors
            }
        }

        async function loadSavedGameFromCloud() {
            if (!db || !currentUser) return;
            try {
                const saveRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'sudokuData', 'savedGame');
                const snap = await getDoc(saveRef);
                if (snap.exists() && snap.data()?.stateJson) {
                    const parsed = JSON.parse(snap.data().stateJson);
                    if (window.restoreSavedGameState) {
                        window.restoreSavedGameState(parsed);
                    }
                }
            } catch (e) {}
        }

        async function saveStatsToCloud(statsObj) {
            if (!db || !currentUser) return;
            try {
                const statsRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'sudokuData', 'stats');
                await setDoc(statsRef, {
                    statsJson: JSON.stringify(statsObj),
                    updatedAt: Date.now()
                });
            } catch (e) {}
        }

        async function loadCloudStats() {
            if (!db || !currentUser) return;
            try {
                const statsRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'sudokuData', 'stats');
                const snap = await getDoc(statsRef);
                if (snap.exists() && snap.data()?.statsJson) {
                    const parsed = JSON.parse(snap.data().statsJson);
                    if (window.mergeLoadedStats) {
                        window.mergeLoadedStats(parsed);
                    }
                }
            } catch (e) {}
        }

        window.cloudSaveGame = saveGameToCloud;
        window.cloudSaveStats = saveStatsToCloud;
        initCloudStorage();
    </script>

    <script>
        /**
         * 數獨核心狀態管理
         */
        let gameMode = 'play'; // 'setup' (自訂題庫模式) | 'play' (正式遊玩模式)
        let currentDifficulty = 'easy'; // 'easy' | 'medium' | 'hard' | 'expert' | 'custom'
        let isNoteMode = false;
        let isNumberLockMode = false; // 手機專用：鎖定某個數字後點擊盤面即可連續填入
        let lockedNumber = 1;         // 當前鎖定的數字 (1~9)
        let isPaused = false;
        let isGameWon = false;
        let selectedCell = { r: 0, c: 0 };
        let bannerTimeout = null;

        // 復原 (Undo) 與 重做 (Redo) 堆疊
        let undoStack = [];
        let redoStack = [];

        // 計時器
        let timerInterval = null;
        let elapsedSeconds = 0;

        // 9x9 盤面與當前唯一解答矩陣
        let board = [];
        let solutionGrid = null; // 9x9 2D array of numbers 1..9

        // 當前邏輯推演高亮資訊
        let currentLogicHint = null; // { target: {r, c, val}, reasons: [{r, c}], title, desc, technique }

        // 各難度統計資料
        let playerStats = {
            easy: { label: '簡單', gamesWon: 0, bestSeconds: null, totalSeconds: 0 },
            medium: { label: '中等', gamesWon: 0, bestSeconds: null, totalSeconds: 0 },
            hard: { label: '困難', gamesWon: 0, bestSeconds: null, totalSeconds: 0 },
            expert: { label: '專家', gamesWon: 0, bestSeconds: null, totalSeconds: 0 },
            custom: { label: '自訂題庫', gamesWon: 0, bestSeconds: null, totalSeconds: 0 }
        };

        const DIFFICULTY_GIVENS = {
            easy: 42,    // 留 42 格已知數
            medium: 34,  // 留 34 格已知數
            hard: 28,    // 留 28 格已知數
            expert: 24   // 留 24 格已知數
        };

        function initEmptyBoard() {
            board = Array.from({ length: 9 }, () =>
                Array.from({ length: 9 }, () => ({
                    fixedValue: 0,
                    userValue: 0,
                    notes: new Set()
                }))
            );
            undoStack = [];
            redoStack = [];
            currentLogicHint = null;
            isGameWon = false;
            updateUndoRedoButtons();
        }

        /**
         * 清空整個盤面 (供自訂出題模式使用)
         */
        function clearEntireBoard() {
            if (isPaused) return;
            saveHistory();
            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    board[r][c].fixedValue = 0;
                    board[r][c].userValue = 0;
                    board[r][c].notes.clear();
                }
            }
            clearLogicHintHighlight();
            renderAll();
            showBanner('已清空整個數獨盤面，請點選格子輸入您的自訂題目！', 'info');
        }

        /**
         * ==========================================
         * 復原 (Undo) 與 重做 (Redo) 歷史管理
         * ==========================================
         */
        function cloneBoardState() {
            return board.map(row =>
                row.map(cell => ({
                    fixedValue: cell.fixedValue,
                    userValue: cell.userValue,
                    notes: Array.from(cell.notes)
                }))
            );
        }

        function restoreBoardFromSnapshot(snapshot) {
            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    board[r][c].fixedValue = snapshot[r][c].fixedValue;
                    board[r][c].userValue = snapshot[r][c].userValue;
                    board[r][c].notes = new Set(snapshot[r][c].notes || []);
                }
            }
        }

        function saveHistory() {
            undoStack.push(cloneBoardState());
            if (undoStack.length > 60) undoStack.shift();
            redoStack = [];
            updateUndoRedoButtons();
        }

        function undoAction() {
            if (isPaused || undoStack.length === 0) return;
            redoStack.push(cloneBoardState());
            const prev = undoStack.pop();
            restoreBoardFromSnapshot(prev);
            clearLogicHintHighlight();
            updateUndoRedoButtons();
            renderAll();
            triggerAutoSave();
        }

        function redoAction() {
            if (isPaused || redoStack.length === 0) return;
            undoStack.push(cloneBoardState());
            const next = redoStack.pop();
            restoreBoardFromSnapshot(next);
            clearLogicHintHighlight();
            updateUndoRedoButtons();
            renderAll();
            triggerAutoSave();
        }

        function updateUndoRedoButtons() {
            const undoBtn = document.getElementById('undoBtn');
            const redoBtn = document.getElementById('redoBtn');
            if (undoBtn) undoBtn.disabled = (undoStack.length === 0);
            if (redoBtn) redoBtn.disabled = (redoStack.length === 0);
        }

        /**
         * ==========================================
         * 盤面數值與同行/列/九宮格衝突檢查
         * ==========================================
         */
        function getCurrentNumberGrid() {
            return Array.from({ length: 9 }, (_, r) =>
                Array.from({ length: 9 }, (_, c) => getCellValue(r, c))
            );
        }

        function hasConflict(r, c) {
            const val = getCellValue(r, c);
            if (val === 0) return false;

            // 檢查同行與同列
            for (let i = 0; i < 9; i++) {
                if (i !== c && getCellValue(r, i) === val) return true;
                if (i !== r && getCellValue(i, c) === val) return true;
            }

            // 檢查同 3x3 九宮格
            const startR = Math.floor(r / 3) * 3;
            const startC = Math.floor(c / 3) * 3;
            for (let dr = 0; dr < 3; dr++) {
                for (let dc = 0; dc < 3; dc++) {
                    const nr = startR + dr;
                    const nc = startC + dc;
                    if ((nr !== r || nc !== c) && getCellValue(nr, nc) === val) {
                        return true;
                    }
                }
            }
            return false;
        }

        /**
         * ==========================================
         * 計時器與防偷看暫停功能
         * ==========================================
         */
        function formatTime(sec) {
            if (sec === null || sec === undefined) return '--:--';
            const m = Math.floor(sec / 60);
            const s = sec % 60;
            return `${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;
        }

        function updateTimerDisplay() {
            const el = document.getElementById('timerText');
            if (el) el.textContent = formatTime(elapsedSeconds);
        }

        function startTimer(reset = false) {
            if (timerInterval) clearInterval(timerInterval);
            if (reset) elapsedSeconds = 0;
            updateTimerDisplay();

            timerInterval = setInterval(() => {
                if (!isPaused && gameMode === 'play' && !isGameWon) {
                    elapsedSeconds++;
                    updateTimerDisplay();
                }
            }, 1000);
        }

        function stopTimer() {
            if (timerInterval) {
                clearInterval(timerInterval);
                timerInterval = null;
            }
        }

        function togglePauseGame() {
            if (gameMode !== 'play' || isGameWon) return;
            isPaused = !isPaused;
            const overlay = document.getElementById('pauseOverlay');
            const icon = document.getElementById('pauseBtnIcon');
            const text = document.getElementById('pauseBtnText');

            if (isPaused) {
                overlay.classList.remove('hidden');
                if (icon) icon.textContent = '▶️';
                if (text) text.textContent = '繼續';
            } else {
                overlay.classList.add('hidden');
                if (icon) icon.textContent = '⏸️';
                if (text) text.textContent = '暫停';
            }
        }

        /**
         * ==========================================
         * 進階邏輯推演提示系統 (Logical Deduction Engine)
         * ==========================================
         */
        function clearLogicHintHighlight() {
            currentLogicHint = null;
            const panel = document.getElementById('logicHintPanel');
            if (panel) panel.classList.add('hidden');
            if (board.length === 9) {
                renderBoard();
            }
        }

        function findNextLogicalStep() {
            const grid = getCurrentNumberGrid();

            // 1. 先檢查玩家目前是否有填錯（與唯一解不符或衝突）的格子
            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    const userVal = board[r][c].userValue;
                    if (userVal !== 0) {
                        if (hasConflict(r, c) || (solutionGrid && solutionGrid[r][c] !== userVal)) {
                            return {
                                isErrorFix: true,
                                target: { r, c, val: solutionGrid ? solutionGrid[r][c] : 0 },
                                reasons: [],
                                technique: '錯誤偵測',
                                title: `第 ${r + 1} 列、第 ${c + 1} 行填寫有誤`,
                                desc: `目前您在 (${r + 1}, ${c + 1}) 填入的數字「${userVal}」與正確解答或行列規則衝突，建議先修正此格！`
                            };
                        }
                    }
                }
            }

            // 計算每一格的合法候選數與排除來源
            function getCandidates(r, c) {
                const cands = [];
                for (let n = 1; n <= 9; n++) {
                    if (isValidPlacement(grid, r, c, n)) cands.push(n);
                }
                return cands;
            }

            // 2. 唯一餘數法 (Naked Single)：某格只剩下唯一 1 個合法候選數字
            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    if (grid[r][c] === 0) {
                        const cands = getCandidates(r, c);
                        if (cands.length === 1) {
                            const ans = cands[0];
                            const reasons = [];
                            for (let i = 0; i < 9; i++) {
                                if (grid[r][i] !== 0) reasons.push({ r, c: i });
                                if (grid[i][c] !== 0) reasons.push({ r: i, c });
                            }
                            const br = Math.floor(r / 3) * 3;
                            const bc = Math.floor(c / 3) * 3;
                            for (let dr = 0; dr < 3; dr++) {
                                for (let dc = 0; dc < 3; dc++) {
                                    if (grid[br + dr][bc + dc] !== 0) reasons.push({ r: br + dr, c: bc + dc });
                                }
                            }
                            return {
                                target: { r, c, val: ans },
                                reasons,
                                technique: '唯一餘數法 (Naked Single)',
                                title: `第 ${r + 1} 列、第 ${c + 1} 行必定是「${ans}」`,
                                desc: `觀察綠框格 (${r + 1}, ${c + 1})：受同列、同行與同九宮格內已有的紫框數字排除影響，1~9 中其他 8 個數字皆無法填入，只剩唯一候選數「${ans}」。`
                            };
                        }
                    }
                }
            }

            // 3. 宮內排除法 (Hidden Single in 3x3 Box)
            for (let boxR = 0; boxR < 3; boxR++) {
                for (let boxC = 0; boxC < 3; boxC++) {
                    for (let num = 1; num <= 9; num++) {
                        let exists = false;
                        const possibleCells = [];
                        for (let dr = 0; dr < 3; dr++) {
                            for (let dc = 0; dc < 3; dc++) {
                                const r = boxR * 3 + dr;
                                const c = boxC * 3 + dc;
                                if (grid[r][c] === num) exists = true;
                                else if (grid[r][c] === 0 && isValidPlacement(grid, r, c, num)) {
                                    possibleCells.push({ r, c });
                                }
                            }
                        }
                        if (!exists && possibleCells.length === 1) {
                            const target = possibleCells[0];
                            const reasons = [];
                            for (let rr = 0; rr < 9; rr++) {
                                for (let cc = 0; cc < 9; cc++) {
                                    if (grid[rr][cc] === num) reasons.push({ r: rr, c: cc });
                                }
                            }
                            return {
                                target: { r: target.r, c: target.c, val: num },
                                reasons,
                                technique: '宮內排除法 (Hidden Single)',
                                title: `九宮格內只剩 (${target.r + 1}, ${target.c + 1}) 可填「${num}」`,
                                desc: `觀察此 3x3 九宮格：盤面上的其他「${num}」（紫框格）透過橫列與直欄交叉排除後，這個九宮格內只剩綠框格 (${target.r + 1}, ${target.c + 1}) 能夠放入「${num}」。`
                            };
                        }
                    }
                }
            }

            // 4. 行列排除法 (Hidden Single in Row / Column)
            for (let r = 0; r < 9; r++) {
                for (let num = 1; num <= 9; num++) {
                    if (grid[r].includes(num)) continue;
                    const possibleCols = [];
                    for (let c = 0; c < 9; c++) {
                        if (grid[r][c] === 0 && isValidPlacement(grid, r, c, num)) {
                            possibleCols.push(c);
                        }
                    }
                    if (possibleCols.length === 1) {
                        const c = possibleCols[0];
                        const reasons = [];
                        for (let rr = 0; rr < 9; rr++) {
                            for (let cc = 0; cc < 9; cc++) {
                                if (grid[rr][cc] === num) reasons.push({ r: rr, c: cc });
                            }
                        }
                        return {
                            target: { r, c, val: num },
                            reasons,
                            technique: '橫列排除法 (Row Hidden Single)',
                            title: `第 ${r + 1} 列只剩第 ${c + 1} 行能填「${num}」`,
                            desc: `檢視第 ${r + 1} 列：其他空格皆受到同直欄或同九宮格內的「${num}」（紫框）阻擋，因此第 ${r + 1} 列的「${num}」必定落在綠框格 (${r + 1}, ${c + 1})。`
                        };
                    }
                }
            }

            for (let c = 0; c < 9; c++) {
                for (let num = 1; num <= 9; num++) {
                    let exists = false;
                    const possibleRows = [];
                    for (let r = 0; r < 9; r++) {
                        if (grid[r][c] === num) exists = true;
                        else if (grid[r][c] === 0 && isValidPlacement(grid, r, c, num)) {
                            possibleRows.push(r);
                        }
                    }
                    if (!exists && possibleRows.length === 1) {
                        const r = possibleRows[0];
                        const reasons = [];
                        for (let rr = 0; rr < 9; rr++) {
                            for (let cc = 0; cc < 9; cc++) {
                                if (grid[rr][cc] === num) reasons.push({ r: rr, c: cc });
                            }
                        }
                        return {
                            target: { r, c, val: num },
                            reasons,
                            technique: '直欄排除法 (Col Hidden Single)',
                            title: `第 ${c + 1} 行只剩第 ${r + 1} 列能填「${num}」`,
                            desc: `檢視第 ${c + 1} 行：其他空格皆受到同橫列或同九宮格內的「${num}」（紫框）阻擋，因此第 ${c + 1} 行的「${num}」必定落在綠框格 (${r + 1}, ${c + 1})。`
                        };
                    }
                }
            }

            // 5. 進階候選數推演保底 (從唯一解選取候選數最少的格子引導)
            if (solutionGrid) {
                let bestCell = null;
                let minLen = 10;
                for (let r = 0; r < 9; r++) {
                    for (let c = 0; c < 9; c++) {
                        if (grid[r][c] === 0) {
                            const cands = getCandidates(r, c);
                            if (cands.length < minLen) {
                                minLen = cands.length;
                                bestCell = { r, c, cands };
                            }
                        }
                    }
                }
                if (bestCell) {
                    const ans = solutionGrid[bestCell.r][bestCell.c];
                    return {
                        target: { r: bestCell.r, c: bestCell.c, val: ans },
                        reasons: [],
                        technique: '進階數對與鏈結推演',
                        title: `建議觀察 (${bestCell.r + 1}, ${bestCell.c + 1})，正確數字為「${ans}」`,
                        desc: `目前基本排除法已用盡，此格僅剩候選數 [${bestCell.cands.join(', ')}]。透過進階區塊交叉推演可確定此格應填入「${ans}」。`
                    };
                }
            }

            return null;
        }

        function requestLogicalHint() {
            if (isPaused || gameMode !== 'play' || isGameWon) return;

            const hint = findNextLogicalStep();
            if (!hint) {
                showBanner('盤面已全部填滿或無可推演的空格！', 'info');
                return;
            }

            currentLogicHint = hint;
            selectedCell = { r: hint.target.r, c: hint.target.c };

            const panel = document.getElementById('logicHintPanel');
            document.getElementById('logicTechniqueBadge').textContent = hint.technique;
            document.getElementById('logicHintTitle').textContent = hint.title;
            document.getElementById('logicHintDesc').textContent = hint.desc;
            panel.classList.remove('hidden');

            renderAll();
        }

        function applyCurrentLogicHint() {
            if (!currentLogicHint || !currentLogicHint.target) return;
            const { r, c, val } = currentLogicHint.target;
            if (!val) return;

            saveHistory();
            board[r][c].userValue = val;
            board[r][c].notes.clear();
            autoCleanPeerNotes(r, c, val);
            selectedCell = { r, c };

            clearLogicHintHighlight();
            renderAll();
            triggerAutoSave();
            checkWinCondition();
        }

        function fillDirectHint() {
            if (isPaused || gameMode !== 'play' || isGameWon) return;
            const hint = findNextLogicalStep();
            if (!hint || !hint.target || !hint.target.val) {
                showBanner('目前沒有可填入的空格！', 'info');
                return;
            }
            const { r, c, val } = hint.target;
            saveHistory();
            board[r][c].userValue = val;
            board[r][c].notes.clear();
            autoCleanPeerNotes(r, c, val);
            selectedCell = { r, c };

            clearLogicHintHighlight();
            renderAll();
            triggerAutoSave();
            showBanner(`💡 已為您填入第 ${r + 1} 列、第 ${c + 1} 行的正確數字「${val}」！`, 'info');
            checkWinCondition();
        }

        /**
         * ==========================================
         * 勝利驗證、數據統計視窗與自動存檔
         * ==========================================
         */
        function checkWinCondition() {
            if (gameMode !== 'play' || isGameWon) return;

            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    if (getCellValue(r, c) === 0 || hasConflict(r, c)) {
                        return;
                    }
                }
            }

            // 全部填滿且無衝突 -> 挑戰成功
            isGameWon = true;
            stopTimer();
            clearLogicHintHighlight();

            const stat = playerStats[currentDifficulty] || playerStats.easy;
            stat.gamesWon += 1;
            stat.totalSeconds += elapsedSeconds;
            if (stat.bestSeconds === null || elapsedSeconds < stat.bestSeconds) {
                stat.bestSeconds = elapsedSeconds;
            }

            saveStatsToLocalAndCloud();
            triggerAutoSave();
            openStatsModal(true);
        }

        function openStatsModal(isVictory = false) {
            const modal = document.getElementById('statsModal');
            const victoryBox = document.getElementById('victorySummaryBox');
            const victoryText = document.getElementById('victoryDetailText');
            const titleEl = document.getElementById('modalTitle');

            if (isVictory) {
                const diffLabel = (playerStats[currentDifficulty] || playerStats.easy).label;
                titleEl.textContent = '🎉 恭喜過關！數獨挑戰成功';
                victoryBox.classList.remove('hidden');
                victoryText.textContent = `難度：${diffLabel} ｜ 本局解題耗時：${formatTime(elapsedSeconds)}`;
            } else {
                titleEl.textContent = '解題數據統計';
                victoryBox.classList.add('hidden');
            }

            renderStatsTable();
            modal.classList.remove('hidden');
        }

        function closeStatsModal() {
            document.getElementById('statsModal').classList.add('hidden');
        }

        function renderStatsTable() {
            const tbody = document.getElementById('statsTableBody');
            if (!tbody) return;
            tbody.innerHTML = '';

            ['easy', 'medium', 'hard', 'expert', 'custom'].forEach(key => {
                const item = playerStats[key];
                const avgSec = item.gamesWon > 0 ? Math.round(item.totalSeconds / item.gamesWon) : null;
                const tr = document.createElement('tr');
                if (key === currentDifficulty) {
                    tr.className = 'bg-indigo-50/70 font-bold text-indigo-950';
                }
                tr.innerHTML = `
                    <td class="py-2.5 px-3">${item.label}</td>
                    <td class="py-2.5 px-3 text-center">${item.gamesWon} 局</td>
                    <td class="py-2.5 px-3 text-center font-mono">${formatTime(item.bestSeconds)}</td>
                    <td class="py-2.5 px-3 text-center font-mono">${formatTime(avgSec)}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        function triggerAutoSave() {
            try {
                const payload = {
                    gameMode,
                    currentDifficulty,
                    elapsedSeconds,
                    isGameWon,
                    board: cloneBoardState(),
                    solutionGrid
                };
                localStorage.setItem('sudoku_master_saved_state', JSON.stringify(payload));
                if (window.cloudSaveGame) {
                    window.cloudSaveGame(payload);
                }
            } catch (e) {}
        }

        function saveStatsToLocalAndCloud() {
            try {
                localStorage.setItem('sudoku_master_stats', JSON.stringify(playerStats));
                if (window.cloudSaveStats) {
                    window.cloudSaveStats(playerStats);
                }
            } catch (e) {}
        }

        window.restoreSavedGameState = function (saved) {
            if (!saved || !saved.board || saved.isGameWon) return;
            try {
                gameMode = saved.gameMode || 'play';
                currentDifficulty = saved.currentDifficulty || 'easy';
                elapsedSeconds = saved.elapsedSeconds || 0;
                solutionGrid = saved.solutionGrid || null;
                initEmptyBoard();
                restoreBoardFromSnapshot(saved.board);
                updateModeUI();
                startTimer(false);
                renderAll();
            } catch (e) {}
        };

        window.mergeLoadedStats = function (loaded) {
            if (!loaded) return;
            ['easy', 'medium', 'hard', 'expert', 'custom'].forEach(k => {
                if (loaded[k]) {
                    playerStats[k].gamesWon = Math.max(playerStats[k].gamesWon, loaded[k].gamesWon || 0);
                    playerStats[k].totalSeconds = Math.max(playerStats[k].totalSeconds, loaded[k].totalSeconds || 0);
                    if (loaded[k].bestSeconds !== null && loaded[k].bestSeconds !== undefined) {
                        if (playerStats[k].bestSeconds === null || loaded[k].bestSeconds < playerStats[k].bestSeconds) {
                            playerStats[k].bestSeconds = loaded[k].bestSeconds;
                        }
                    }
                }
            });
        };

        /**
         * ==========================================
         * 數獨唯一解求解器與題目生成器 (Generator & Solver)
         * ==========================================
         */
        function isValidPlacement(grid, r, c, num) {
            for (let i = 0; i < 9; i++) {
                if (grid[r][i] === num) return false;
                if (grid[i][c] === num) return false;
            }
            const br = Math.floor(r / 3) * 3;
            const bc = Math.floor(c / 3) * 3;
            for (let dr = 0; dr < 3; dr++) {
                for (let dc = 0; dc < 3; dc++) {
                    if (grid[br + dr][bc + dc] === num) return false;
                }
            }
            return true;
        }

        function shuffleArray(arr) {
            const copy = [...arr];
            for (let i = copy.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [copy[i], copy[j]] = [copy[j], copy[i]];
            }
            return copy;
        }

        // 尋找最佳空格 (Minimum Remaining Values 啟發式加速求解)
        function findBestEmptyCell(grid) {
            let bestCell = null;
            let minCount = 10;
            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    if (grid[r][c] === 0) {
                        let count = 0;
                        for (let n = 1; n <= 9; n++) {
                            if (isValidPlacement(grid, r, c, n)) count++;
                        }
                        if (count < minCount) {
                            minCount = count;
                            bestCell = { r, c };
                            if (count <= 1) return bestCell;
                        }
                    }
                }
            }
            return bestCell;
        }

        // 隨機產生完整合法的 9x9 終盤
        function generateFullBoard() {
            const grid = Array.from({ length: 9 }, () => Array(9).fill(0));
            function fillRecursive() {
                const empty = findBestEmptyCell(grid);
                if (!empty) return true;
                const { r, c } = empty;
                const nums = shuffleArray([1, 2, 3, 4, 5, 6, 7, 8, 9]);
                for (const n of nums) {
                    if (isValidPlacement(grid, r, c, n)) {
                        grid[r][c] = n;
                        if (fillRecursive()) return true;
                        grid[r][c] = 0;
                    }
                }
                return false;
            }
            fillRecursive();
            return grid;
        }

        // 計算盤面的解數量 (limit = 2 用於快速判斷是否為唯一解)
        function countSolutionsAndGetOne(inputGrid, limit = 2) {
            const grid = inputGrid.map(row => [...row]);
            let count = 0;
            let firstSolution = null;

            function backtrack() {
                if (count >= limit) return;
                const empty = findBestEmptyCell(grid);
                if (!empty) {
                    count++;
                    if (!firstSolution) {
                        firstSolution = grid.map(row => [...row]);
                    }
                    return;
                }
                const { r, c } = empty;
                for (let n = 1; n <= 9; n++) {
                    if (isValidPlacement(grid, r, c, n)) {
                        grid[r][c] = n;
                        backtrack();
                        if (count >= limit) return;
                        grid[r][c] = 0;
                    }
                }
            }
            backtrack();
            return { count, solution: firstSolution };
        }

        // 根據難度挖洞並嚴格確保每一步挖洞後都維持「唯一解」
        function generateSudokuByDifficulty(difficulty) {
            const targetGivens = DIFFICULTY_GIVENS[difficulty] || 36;
            const fullGrid = generateFullBoard();
            const puzzleGrid = fullGrid.map(row => [...row]);

            // 打亂所有 81 個座標依序嘗試挖空
            const positions = [];
            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    positions.push({ r, c });
                }
            }
            const shuffledPos = shuffleArray(positions);
            let currentGivens = 81;

            for (const { r, c } of shuffledPos) {
                if (currentGivens <= targetGivens) break;
                const backup = puzzleGrid[r][c];
                puzzleGrid[r][c] = 0;

                // 檢查挖空後是否仍然只有唯一解
                const { count } = countSolutionsAndGetOne(puzzleGrid, 2);
                if (count !== 1) {
                    puzzleGrid[r][c] = backup; // 產生多解則還原
                } else {
                    currentGivens--;
                }
            }

            return { puzzleGrid, solution: fullGrid };
        }

        /**
         * 啟動指定難度的新遊戲
         */
        function startNewGeneratedGame(difficulty) {
            currentDifficulty = difficulty;
            gameMode = 'play';
            isPaused = false;
            document.getElementById('pauseOverlay').classList.add('hidden');
            clearLogicHintHighlight();
            hideBanner();

            initEmptyBoard();
            const { puzzleGrid, solution } = generateSudokuByDifficulty(difficulty);
            solutionGrid = solution;

            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    board[r][c].fixedValue = puzzleGrid[r][c];
                }
            }

            updateModeUI();
            startTimer(true);
            renderAll();
            triggerAutoSave();

            const diffNames = { easy: '簡單', medium: '中等', hard: '困難', expert: '專家' };
            showBanner(`已為您生成「${diffNames[difficulty]}」難度題目（已驗證唯一解），祝您解題順利！`, 'info');
        }

        /**
         * 切換至自訂出題設定模式
         */
        function enterCustomSetupMode() {
            currentDifficulty = 'custom';
            gameMode = 'setup';
            isPaused = false;
            document.getElementById('pauseOverlay').classList.add('hidden');
            stopTimer();
            clearLogicHintHighlight();

            // 將玩家已填數字清除，保留固定題庫供使用者修改或清空
            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    board[r][c].userValue = 0;
                    board[r][c].notes.clear();
                }
            }
            undoStack = [];
            redoStack = [];
            updateUndoRedoButtons();
            updateModeUI();
            renderAll();
            showBanner('已進入「自訂題目模式」！您可以修改盤面或點選「清空盤面」輸入自己的數獨題目。', 'warning');
        }

        function updateModeUI() {
            // 更新難度按鈕高亮
            ['easy', 'medium', 'hard', 'expert', 'custom'].forEach(d => {
                const btn = document.getElementById(`diffBtn-${d}`);
                if (!btn) return;
                if (d === currentDifficulty) {
                    btn.className = "diff-btn flex-1 sm:flex-none px-2 py-1.5 text-xs font-bold rounded-lg transition bg-indigo-600 text-white shadow-xs text-center whitespace-nowrap";
                } else if (d === 'custom') {
                    btn.className = "diff-btn flex-1 sm:flex-none px-2.5 py-1.5 text-xs font-bold rounded-lg transition bg-amber-50 text-amber-800 border border-amber-200 text-center whitespace-nowrap";
                } else {
                    btn.className = "diff-btn flex-1 sm:flex-none px-2 py-1.5 text-xs font-bold rounded-lg transition bg-slate-100 text-slate-700 text-center whitespace-nowrap";
                }
            });

            const setupControls = document.getElementById('setupControls');
            const hintToolbar = document.getElementById('hintToolbar');
            const badge = document.getElementById('modeBadge');

            if (gameMode === 'setup') {
                setupControls.classList.remove('hidden');
                setupControls.classList.add('flex');
                hintToolbar.classList.add('hidden');
                badge.className = "hidden sm:inline-block px-2 py-1 rounded-full text-[11px] font-bold bg-amber-100 text-amber-800 border border-amber-300";
                badge.textContent = "📝 出題模式";
            } else {
                setupControls.classList.add('hidden');
                setupControls.classList.remove('flex');
                hintToolbar.classList.remove('hidden');
                badge.className = "hidden sm:inline-block px-2 py-1 rounded-full text-[11px] font-bold bg-emerald-100 text-emerald-800 border border-emerald-300";
                badge.textContent = "🎮 遊玩模式";
            }
        }

        /**
         * 切換手機版「📌 鎖定數字連續點填模式」
         */
        function toggleNumberLockMode() {
            isNumberLockMode = !isNumberLockMode;
            if (isNumberLockMode) {
                const currentVal = (selectedCell.r !== null && selectedCell.c !== null)
                    ? getCellValue(selectedCell.r, selectedCell.c)
                    : 0;
                if (currentVal >= 1 && currentVal <= 9) {
                    lockedNumber = currentVal;
                }
            }
            updateNumberLockUI();
            renderAll();
        }

        function updateNumberLockUI() {
            const btn = document.getElementById('numLockBtn');
            const icon = document.getElementById('numLockIcon');
            const text = document.getElementById('numLockText');
            if (!btn) return;

            if (isNumberLockMode) {
                btn.className = "px-2.5 py-1 rounded-lg text-xs font-bold border-2 transition flex items-center gap-1 bg-amber-50 border-amber-500 text-amber-900 shadow-xs";
                icon.textContent = '📌';
                text.textContent = `已鎖定【數字 ${lockedNumber}】點擊盤面即可連續填寫`;
            } else {
                btn.className = "px-2.5 py-1 rounded-lg text-xs font-bold border transition flex items-center gap-1 bg-slate-50 border-slate-200 text-slate-700";
                icon.textContent = '🔓';
                text.textContent = '一般模式 (點此開啟：固定數字連續點填)';
            }
        }

        /**
         * 取得某格目前呈現的有效主數字 (0 代表空格)
         */
        function getCellValue(r, c) {
            return board[r][c].fixedValue || board[r][c].userValue || 0;
        }

        /**
         * 選擇盤面上的格子 (支援手機連續點填鎖定模式)
         */
        function selectCell(r, c) {
            if (isPaused) return;
            selectedCell = { r, c };

            // 若開啟了手機「鎖定數字連續點填」模式，點選空格或可編輯格時直接填入該鎖定數字
            if (isNumberLockMode && lockedNumber >= 1 && lockedNumber <= 9) {
                const targetCell = board[r][c];
                if (gameMode === 'setup' || targetCell.fixedValue === 0) {
                    applyNumberToSelectedCell(lockedNumber);
                    return;
                }
            }
            renderAll();
        }

        /**
         * 點擊下方數字鍵盤 (1~9)
         */
        function handleNumberInput(num) {
            if (isPaused || isGameWon) return;

            // 若開啟了「鎖定數字連續點填」模式，點擊數字鍵代表切換目前鎖定的數字
            if (isNumberLockMode) {
                lockedNumber = num;
                updateNumberLockUI();
                renderAll();
                return;
            }

            applyNumberToSelectedCell(num);
        }

        /**
         * 實際將數字寫入當前選中格 (支援一般填字、小筆記、出題模式)
         */
        function applyNumberToSelectedCell(num) {
            const { r, c } = selectedCell;
            if (r === null || c === null) return;

            const cell = board[r][c];
            clearLogicHintHighlight();

            // 1. 出題設定模式
            if (gameMode === 'setup') {
                saveHistory();
                cell.notes.clear();
                cell.fixedValue = (cell.fixedValue === num) ? 0 : num;
                renderAll();
                return;
            }

            // 2. 正式遊玩模式
            if (cell.fixedValue !== 0) {
                showBanner('此格為題目固定數字（黑字粗體），無法修改！', 'warning');
                return;
            }

            saveHistory();

            if (isNoteMode) {
                if (cell.userValue !== 0) {
                    cell.userValue = 0;
                }
                if (cell.notes.has(num)) {
                    cell.notes.delete(num);
                } else {
                    cell.notes.add(num);
                }
            } else {
                if (cell.userValue === num) {
                    cell.userValue = 0;
                } else {
                    cell.userValue = num;
                    cell.notes.clear();
                    // 自動消除同行、同列、同九宮格內衝突的候選筆記
                    autoCleanPeerNotes(r, c, num);
                }
            }

            renderAll();
            triggerAutoSave();
            checkWinCondition();
        }

        /**
         * 填入正式數字時，自動消去同行、同列、同宮格中對應的候選小筆記
         */
        function autoCleanPeerNotes(r, c, num) {
            for (let i = 0; i < 9; i++) {
                board[r][i].notes.delete(num);
                board[i][c].notes.delete(num);
            }
            const startR = Math.floor(r / 3) * 3;
            const startC = Math.floor(c / 3) * 3;
            for (let dr = 0; dr < 3; dr++) {
                for (let dc = 0; dc < 3; dc++) {
                    board[startR + dr][startC + dc].notes.delete(num);
                }
            }
        }

        /**
         * 一鍵為所有空格自動計算並填入合法候選數筆記
         */
        function autoFillAllNotes() {
            if (isPaused || gameMode !== 'play') return;
            saveHistory();
            const currentGrid = getCurrentNumberGrid();
            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    if (currentGrid[r][c] === 0) {
                        board[r][c].notes.clear();
                        for (let n = 1; n <= 9; n++) {
                            if (isValidPlacement(currentGrid, r, c, n)) {
                                board[r][c].notes.add(n);
                            }
                        }
                    }
                }
            }
            renderAll();
            triggerAutoSave();
            showBanner('已為所有空格自動標記目前合法的候選數小筆記！', 'info');
        }

        /**
         * 清除目前選中格子的內容與筆記
         */
        function eraseSelectedCell() {
            if (isPaused || isGameWon) return;
            const { r, c } = selectedCell;
            if (r === null || c === null) return;
            const cell = board[r][c];

            clearLogicHintHighlight();

            if (gameMode === 'setup') {
                if (cell.fixedValue === 0 && cell.notes.size === 0) return;
                saveHistory();
                cell.fixedValue = 0;
                cell.notes.clear();
            } else {
                if (cell.fixedValue !== 0) {
                    showBanner('題庫的固定數字無法清除喔！', 'warning');
                    return;
                }
                if (cell.userValue === 0 && cell.notes.size === 0) return;
                saveHistory();
                cell.userValue = 0;
                cell.notes.clear();
            }
            renderAll();
            triggerAutoSave();
        }

        /**
         * 切換小筆記模式
         */
        function toggleNoteMode() {
            isNoteMode = !isNoteMode;
            const btn = document.getElementById('noteModeBtn');
            const statusText = document.getElementById('noteStatusText');

            if (isNoteMode) {
                btn.className = "py-2 px-1 rounded-xl font-bold text-xs border-2 transition flex flex-col items-center justify-center gap-0.5 bg-indigo-50 border-indigo-500 text-indigo-700 shadow-xs active:scale-95";
                statusText.className = "px-1 py-0.2 rounded text-[10px] bg-indigo-600 text-white";
                statusText.textContent = "開";
            } else {
                btn.className = "py-2 px-1 rounded-xl font-bold text-xs border-2 transition flex flex-col items-center justify-center gap-0.5 bg-slate-50 border-slate-200 text-slate-700 active:scale-95";
                statusText.className = "px-1 py-0.2 rounded text-[10px] bg-slate-200 text-slate-600";
                statusText.textContent = "關";
            }
            renderAll();
        }

        /**
         * 確認自訂題庫、驗證唯一解並進入正式遊玩模式
         */
        function confirmPuzzleAndStart() {
            let fixedCount = 0;
            const customGrid = Array.from({ length: 9 }, () => Array(9).fill(0));

            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    const val = board[r][c].fixedValue;
                    customGrid[r][c] = val;
                    if (val !== 0) {
                        fixedCount++;
                        if (hasConflict(r, c)) {
                            selectCell(r, c);
                            showBanner(`第 ${r + 1} 列、第 ${c + 1} 行的數字「${val}」發生同行/列/宮重複衝突，請先修正！`, 'error');
                            return;
                        }
                    }
                }
            }

            if (fixedCount === 0) {
                showBanner('盤面目前是全空的！請先輸入幾個題目數字，或點選上方難度自動生成題庫。', 'warning');
                return;
            }

            // 執行唯一解驗證
            const { count, solution } = countSolutionsAndGetOne(customGrid, 2);
            if (count === 0) {
                showBanner('❌ 驗證失敗：這組自訂題目「無解」，請檢查是否有輸入錯誤的題目數字！', 'error');
                return;
            }

            solutionGrid = solution;
            gameMode = 'play';
            undoStack = [];
            redoStack = [];
            updateUndoRedoButtons();
            updateModeUI();
            startTimer(true);

            if (count === 1) {
                showBanner(`✅ 驗證通過！此題目擁有完美的「唯一解」（共 ${fixedCount} 個提示數），正式開始計時！`, 'success');
            } else {
                showBanner(`⚠️ 注意：此題目提示數較少，存在多重解，系統已鎖定其中一組標準解供邏輯推演使用。`, 'warning');
            }
            renderAll();
            triggerAutoSave();
        }

        /**
         * 狀態通知橫幅 (懸浮式自動消失，不推擠手機版面)
         */
        function showBanner(message, type = 'info') {
            const banner = document.getElementById('statusBanner');
            const textEl = document.getElementById('statusBannerText');
            textEl.textContent = message;

            banner.classList.remove(
                'hidden',
                'bg-indigo-600', 'text-white', 'border-indigo-700',
                'bg-emerald-600', 'border-emerald-700',
                'bg-amber-500', 'text-slate-950', 'border-amber-600',
                'bg-rose-600', 'border-rose-700'
            );
            banner.classList.add('border');

            if (type === 'success') {
                banner.classList.add('bg-emerald-600', 'text-white', 'border-emerald-700');
            } else if (type === 'warning') {
                banner.classList.add('bg-amber-500', 'text-slate-950', 'border-amber-600');
            } else if (type === 'error') {
                banner.classList.add('bg-rose-600', 'text-white', 'border-rose-700');
            } else {
                banner.classList.add('bg-indigo-600', 'text-white', 'border-indigo-700');
            }

            if (bannerTimeout) clearTimeout(bannerTimeout);
            bannerTimeout = setTimeout(() => {
                hideBanner();
            }, 3500);
        }

        function hideBanner() {
            document.getElementById('statusBanner').classList.add('hidden');
        }

        /**
         * 渲染整個畫面 (9x9 盤面 + 剩餘數字鍵盤 + 座標提示)
         */
        function renderAll() {
            renderBoard();
            renderNumpadAndRemaining();
            renderCoordIndicator();
        }

        function renderCoordIndicator() {
            const indicator = document.getElementById('coordIndicator');
            if (selectedCell.r === null || selectedCell.c === null) {
                indicator.textContent = '未選擇格子';
                return;
            }
            const val = getCellValue(selectedCell.r, selectedCell.c);
            indicator.textContent = `座標：第 ${selectedCell.r + 1} 列 / 第 ${selectedCell.c + 1} 行 ${val ? `(數字 ${val})` : '(空格)'}`;
        }

        /**
         * 渲染 9x9 數獨網格 (包含橫直高亮、九宮格區塊高亮、相同數字高亮與邏輯推演標記)
         */
        function renderBoard() {
            const container = document.getElementById('sudokuBoard');
            container.innerHTML = '';

            const selR = selectedCell.r;
            const selC = selectedCell.c;
            const selectedVal = isNumberLockMode
                ? lockedNumber
                : ((selR !== null && selC !== null) ? getCellValue(selR, selC) : 0);
            const selBoxR = selR !== null ? Math.floor(selR / 3) : -1;
            const selBoxC = selC !== null ? Math.floor(selC / 3) : -1;

            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    const cellData = board[r][c];
                    const val = getCellValue(r, c);
                    const cellEl = document.createElement('div');

                    let classes = ['sudoku-cell'];
                    if (c === 2 || c === 5) classes.push('border-r-thick');
                    if (c === 8) classes.push('border-r-none');
                    if (r === 2 || r === 5) classes.push('border-b-thick');
                    if (r === 8) classes.push('border-b-none');

                    const isSelected = (r === selR && c === selC);
                    const isSameRowOrCol = (selR !== null && selC !== null) && (r === selR || c === selC);
                    const isSameBox = (Math.floor(r / 3) === selBoxR && Math.floor(c / 3) === selBoxC);
                    const isSameNumber = (selectedVal !== 0 && val === selectedVal && !isSelected);
                    const isConflict = hasConflict(r, c);

                    const isHintTarget = currentLogicHint?.target && currentLogicHint.target.r === r && currentLogicHint.target.c === c;
                    const isHintReason = currentLogicHint?.reasons && currentLogicHint.reasons.some(pt => pt.r === r && pt.c === c);

                    // 高亮層級：選中格 > 邏輯推演目標格 > 相同數字 > 直式/橫式高亮 > 3x3 九宮格高亮
                    if (isHintTarget) {
                        classes.push('cell-hint-target');
                    } else if (isSelected) {
                        classes.push('cell-selected');
                    } else if (isHintReason) {
                        classes.push('cell-hint-reason');
                    } else if (isSameNumber) {
                        classes.push('cell-same-number');
                    } else if (isSameRowOrCol) {
                        classes.push('cell-cross-highlight');
                    } else if (isSameBox) {
                        classes.push('cell-box-highlight');
                    }

                    if (isConflict) {
                        classes.push('cell-conflict');
                    }

                    cellEl.className = classes.join(' ');
                    cellEl.onclick = () => selectCell(r, c);

                    // 顯示內容：固定題目數字（黑色特粗體） vs 玩家作答數字（藍色一般粗體）
                    if (val !== 0) {
                        const numSpan = document.createElement('span');
                        numSpan.textContent = val;

                        if (cellData.fixedValue !== 0) {
                            numSpan.className = `text-xl sm:text-2xl font-black leading-none ${isSelected && !isHintTarget ? 'text-white' : (isConflict ? 'text-rose-700' : 'text-slate-900')}`;
                        } else {
                            numSpan.className = `text-xl sm:text-2xl font-semibold leading-none ${isSelected && !isHintTarget ? 'text-white' : (isConflict ? 'text-rose-600' : 'text-blue-600')}`;
                        }
                        cellEl.appendChild(numSpan);
                    } else if (cellData.notes.size > 0) {
                        const notesGrid = document.createElement('div');
                        notesGrid.className = 'notes-grid';

                        for (let n = 1; n <= 9; n++) {
                            const noteItem = document.createElement('div');
                            noteItem.className = 'note-item';
                            if (cellData.notes.has(n)) {
                                noteItem.textContent = n;
                                if (selectedVal === n) {
                                    noteItem.classList.add('note-highlight');
                                }
                            }
                            notesGrid.appendChild(noteItem);
                        }
                        cellEl.appendChild(notesGrid);
                    }

                    container.appendChild(cellEl);
                }
            }
        }

        /**
         * 渲染下方數字 1~9 按鈕與「剩餘還沒使用的數字」數量
         */
        function renderNumpadAndRemaining() {
            const numpad = document.getElementById('numpadContainer');
            numpad.innerHTML = '';

            const counts = Array(10).fill(0);
            let totalFilled = 0;

            for (let r = 0; r < 9; r++) {
                for (let c = 0; c < 9; c++) {
                    const val = getCellValue(r, c);
                    if (val >= 1 && val <= 9) {
                        counts[val]++;
                        totalFilled++;
                    }
                }
            }

            const remainingCells = 81 - totalFilled;
            const summaryEl = document.getElementById('remainingSummaryText');
            summaryEl.textContent = remainingCells === 0
                ? '已填滿！'
                : `剩 ${remainingCells} 格`;

            const currentCell = (selectedCell.r !== null && selectedCell.c !== null)
                ? board[selectedCell.r][selectedCell.c]
                : null;

            for (let num = 1; num <= 9; num++) {
                const usedCount = counts[num];
                const remaining = Math.max(0, 9 - usedCount);
                const isCompleted = (usedCount >= 9);
                const hasNoteInSelected = !isNumberLockMode && isNoteMode && currentCell && currentCell.fixedValue === 0 && currentCell.notes.has(num);
                const isLockedTarget = isNumberLockMode && lockedNumber === num;

                const btn = document.createElement('button');
                btn.onclick = () => handleNumberInput(num);

                let btnClasses = "flex flex-col items-center justify-center py-2.5 sm:py-2 px-0.5 rounded-xl border transition select-none active:scale-95 ";

                if (isLockedTarget) {
                    btnClasses += "bg-amber-500 border-amber-600 text-slate-950 shadow-sm ring-2 ring-amber-300";
                } else if (hasNoteInSelected) {
                    btnClasses += "bg-indigo-600 border-indigo-700 text-white shadow-xs";
                } else if (isCompleted && !isNoteMode) {
                    btnClasses += "bg-emerald-50/80 border-emerald-200 text-emerald-700 opacity-65";
                } else {
                    btnClasses += "bg-slate-50 active:bg-indigo-100 border-slate-200 text-slate-800 shadow-xs";
                }

                btn.className = btnClasses;

                const digitSpan = document.createElement('span');
                digitSpan.className = "text-xl sm:text-xl font-extrabold leading-none mb-1";
                digitSpan.textContent = num;

                const remainSpan = document.createElement('span');
                if (isLockedTarget) {
                    remainSpan.className = "text-[10px] font-extrabold text-slate-900 leading-none";
                    remainSpan.textContent = `鎖定(${remaining})`;
                } else if (hasNoteInSelected) {
                    remainSpan.className = "text-[10px] font-bold text-indigo-100 leading-none";
                    remainSpan.textContent = `已筆記`;
                } else if (isCompleted) {
                    remainSpan.className = "text-[10px] font-bold text-emerald-600 leading-none";
                    remainSpan.textContent = `已滿`;
                } else {
                    remainSpan.className = "text-[10px] font-semibold text-slate-500 leading-none";
                    remainSpan.textContent = `剩${remaining}`;
                }

                btn.appendChild(digitSpan);
                btn.appendChild(remainSpan);
                numpad.appendChild(btn);
            }
        }

        /**
         * 綁定實體鍵盤操作
         */
        window.addEventListener('keydown', (e) => {
            if (isPaused) return;
            if (selectedCell.r === null || selectedCell.c === null) return;

            if (e.key >= '1' && e.key <= '9') {
                e.preventDefault();
                handleNumberInput(parseInt(e.key, 10));
                return;
            }

            if (e.key === 'Backspace' || e.key === 'Delete' || e.key === '0') {
                e.preventDefault();
                eraseSelectedCell();
                return;
            }

            if (e.key === 'n' || e.key === 'N') {
                e.preventDefault();
                toggleNoteMode();
                return;
            }

            if (e.key === 'z' || e.key === 'Z') {
                e.preventDefault();
                undoAction();
                return;
            }

            if (e.key === 'y' || e.key === 'Y') {
                e.preventDefault();
                redoAction();
                return;
            }

            if (e.key === 'h' || e.key === 'H') {
                e.preventDefault();
                requestLogicalHint();
                return;
            }

            let { r, c } = selectedCell;
            if (e.key === 'ArrowUp' || e.key === 'w' || e.key === 'W') {
                e.preventDefault();
                selectCell((r + 8) % 9, c);
            } else if (e.key === 'ArrowDown' || e.key === 's' || e.key === 'S') {
                e.preventDefault();
                selectCell((r + 1) % 9, c);
            } else if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') {
                e.preventDefault();
                selectCell(r, (c + 8) % 9);
            } else if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') {
                e.preventDefault();
                selectCell(r, (c + 1) % 9);
            }
        });

        // 啟動時預設生成一局「簡單」難度唯一解題目
        window.onload = function () {
            try {
                const localStats = localStorage.getItem('sudoku_master_stats');
                if (localStats) window.mergeLoadedStats(JSON.parse(localStats));
            } catch (e) {}
            startNewGeneratedGame('easy');
        };
    </script>
</body>
</html>
