[index.html](https://github.com/user-attachments/files/23896470/index.html)
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>才藝班點名管理系統 Pro</title>
    
    <!-- 1. 引入 Tailwind CSS (樣式庫) -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- 2. 引入 Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;500;700&family=Inter:wght@300;400;500;600&display=swap" rel="stylesheet">

    <!-- 3. 引入 SheetJS (用於讀取 Excel) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

    <!-- 4. 引入 SortableJS (用於拖曳排序) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Sortable/1.15.0/Sortable.min.js"></script>

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'Noto Sans TC', 'sans-serif'],
                    },
                    colors: {
                        slate: {
                            50: '#f8fafc',
                            100: '#f1f5f9',
                            200: '#e2e8f0',
                            300: '#cbd5e1',
                            400: '#94a3b8',
                            500: '#64748b',
                            600: '#475569',
                            700: '#334155',
                            800: '#1e293b',
                            900: '#0f172a',
                        },
                        brand: {
                            500: '#6366f1', // Indigo
                            600: '#4f46e5',
                        }
                    }
                }
            }
        }
    </script>

    <style>
        /* 隱藏捲軸但保留功能 */
        .no-scrollbar::-webkit-scrollbar { width: 6px; height: 6px; }
        .no-scrollbar::-webkit-scrollbar-track { background: transparent; }
        .no-scrollbar::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 3px; }
        .no-scrollbar::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
        
        /* 動畫效果 */
        .fade-in { animation: fadeIn 0.4s cubic-bezier(0.16, 1, 0.3, 1); }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* Modal 背景模糊 */
        .modal-backdrop { background-color: rgba(15, 23, 42, 0.4); backdrop-filter: blur(4px); }
        
        /* 表格列互動 */
        .table-row-hover:hover td { background-color: #f8fafc; }
        .sticky-col { position: sticky; z-index: 10; background-color: white; }
        .table-row-hover:hover .sticky-col { background-color: #f8fafc; }
        
        /* 統計列樣式 */
        .summary-row td { background-color: #f1f5f9; font-weight: 600; color: #475569; }
        .summary-row .sticky-col { background-color: #f1f5f9; }

        /* Action Buttons Visibility */
        .group:hover .action-buttons { opacity: 1; }

        /* Checkbox Custom Style */
        .weekday-checkbox:checked + label {
            background-color: #4f46e5;
            color: white;
            border-color: #4f46e5;
        }

        /* Sortable Styles */
        .sortable-ghost { opacity: 0.4; background-color: #e2e8f0; }
        .sortable-drag { cursor: grabbing; background-color: #fff; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06); }
        .drag-handle { cursor: grab; color: #cbd5e1; transition: color 0.2s; }
        .drag-handle:hover { color: #64748b; }
        .group:hover .drag-handle { opacity: 1; }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen p-4 sm:p-8 antialiased selection:bg-brand-500 selection:text-white">

    <!-- 隱藏的檔案輸入框 (用於匯入) -->
    <input type="file" id="import-file-input" accept=".xlsx, .xls, .csv" class="hidden">

    <!-- 應用程式容器 -->
    <div id="app" class="max-w-[1600px] mx-auto fade-in">
        
        <!-- 載入中畫面 -->
        <div id="loading-screen" class="flex flex-col items-center justify-center py-40">
            <div class="relative w-16 h-16">
                <div class="absolute inset-0 border-4 border-slate-200 rounded-full"></div>
                <div class="absolute inset-0 border-4 border-brand-500 border-t-transparent rounded-full animate-spin"></div>
            </div>
            <p class="text-slate-400 font-medium mt-6 tracking-wide">系統載入中...</p>
        </div>

        <!-- 系統訊息 Banner -->
        <div id="system-banner" class="hidden p-4 rounded-xl mb-8 shadow-sm transition-all duration-300 border border-transparent">
            <div class="flex items-start gap-3">
                <span class="text-xl">⚠️</span>
                <div>
                    <p class="font-bold mb-1" id="banner-title">系統訊息</p>
                    <p id="banner-text" class="text-sm opacity-90"></p>
                </div>
            </div>
        </div>

        <!-- 主介面 (預設隱藏) -->
        <div id="main-interface" class="hidden flex flex-col h-full gap-8">
            
            <!-- Header -->
            <header class="flex flex-col md:flex-row justify-between items-end md:items-center gap-4 pb-2">
                <div>
                    <h1 class="text-3xl font-bold text-slate-900 tracking-tight flex items-center gap-3">
                        <span class="bg-brand-500 text-white rounded-lg p-1.5 text-xl shadow-lg shadow-brand-500/30">📅</span> 
                        才藝班管理系統 <span class="text-slate-400 font-light text-xl">Pro</span>
                    </h1>
                    <p class="text-slate-500 text-sm mt-2 font-medium tracking-wide">智慧點名 · 自動排程 · 雲端同步</p>
                </div>
                <div class="flex items-center gap-4">
                    <div class="text-xs font-mono text-slate-400 bg-white px-3 py-1.5 rounded-full border border-slate-200 shadow-sm" id="user-display">
                        ID: 檢查中...
                    </div>
                    
                    <button id="export-btn" class="flex items-center gap-2 bg-white text-slate-700 border border-slate-300 hover:bg-slate-50 hover:text-slate-900 px-4 py-2.5 rounded-xl font-medium shadow-sm transition-all active:scale-95">
                        <span class="text-lg leading-none">📊</span> 匯出 Excel
                    </button>
                    
                    <button id="save-btn" class="flex items-center gap-2 bg-slate-900 hover:bg-slate-800 text-white px-6 py-2.5 rounded-xl font-medium shadow-xl shadow-slate-900/10 transition-all hover:-translate-y-0.5 active:scale-95 disabled:opacity-50 disabled:cursor-not-allowed">
                        <span>儲存變更</span>
                    </button>
                </div>
            </header>

            <div class="flex flex-col lg:flex-row gap-8 h-full items-start">
                
                <!-- 左側：課程選單 -->
                <nav class="lg:w-72 flex-shrink-0 flex flex-col gap-6 w-full">
                    <div class="bg-white rounded-2xl shadow-[0_2px_20px_rgb(0,0,0,0.04)] border border-slate-100 overflow-hidden sticky top-6">
                        <div class="bg-white px-6 py-5 border-b border-slate-100 flex justify-between items-center">
                            <span class="font-bold text-slate-700">課程列表</span>
                            <span class="text-xs bg-slate-100 text-slate-500 px-2 py-1 rounded-md" id="class-count-badge">0</span>
                        </div>
                        <div id="class-list" class="flex flex-col max-h-[60vh] overflow-y-auto no-scrollbar p-2 gap-1">
                            <!-- 課程列表將由 JS 動態生成 -->
                        </div>
                        <div class="p-4 border-t border-slate-50 bg-slate-50/50">
                            <button onclick="window.openClassModal('add')" class="w-full group flex justify-center items-center gap-2 py-3 bg-white hover:bg-brand-50 text-slate-600 hover:text-brand-600 rounded-xl transition-all text-sm font-semibold border border-dashed border-slate-300 hover:border-brand-300 shadow-sm">
                                <span class="bg-slate-100 group-hover:bg-brand-100 rounded-md w-5 h-5 flex items-center justify-center text-xs transition-colors">＋</span>
                                新增課程
                            </button>
                        </div>
                    </div>
                </nav>

                <!-- 右側：點名表格 -->
                <main class="flex-1 overflow-hidden min-h-[600px] w-full bg-white rounded-2xl shadow-[0_2px_20px_rgb(0,0,0,0.04)] border border-slate-100 flex flex-col">
                    <div id="grid-container" class="flex flex-col h-full hidden">
                        
                        <!-- Toolbar & Info -->
                        <div class="px-8 py-6 border-b border-slate-100 bg-white rounded-t-2xl z-20">
                            <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-6 mb-6">
                                <div>
                                    <h2 id="current-class-title" class="text-2xl font-bold text-slate-800 tracking-tight">課程名稱</h2>
                                    <div class="flex flex-wrap gap-4 mt-3">
                                        <span id="class-schedule-display" class="inline-flex items-center gap-1.5 px-3 py-1 rounded-full text-xs font-medium bg-indigo-50 text-indigo-700 border border-indigo-100">
                                            🕒 --
                                        </span>
                                        <span id="class-fee-display" class="inline-flex items-center gap-1.5 px-3 py-1 rounded-full text-xs font-medium bg-emerald-50 text-emerald-700 border border-emerald-100">
                                            💰 --
                                        </span>
                                    </div>
                                </div>
                                <div class="flex gap-3">
                                    <button onclick="window.triggerImport()" class="flex items-center gap-2 bg-white text-slate-600 hover:bg-slate-50 border border-slate-200 px-4 py-2.5 rounded-xl shadow-sm transition-all hover:-translate-y-0.5 active:scale-95 text-sm font-medium">
                                        <span class="text-lg leading-none">📥</span> 匯入名單
                                    </button>
                                    <button onclick="window.openStudentModal('add')" class="flex items-center gap-2 bg-brand-600 hover:bg-brand-700 text-white px-5 py-2.5 rounded-xl shadow-lg shadow-brand-500/20 transition-all hover:-translate-y-0.5 active:scale-95 text-sm font-medium">
                                        <span class="text-lg leading-none">+</span> 新增學生
                                    </button>
                                </div>
                            </div>

                            <div class="flex gap-6 text-xs font-medium pt-4 border-t border-slate-100 text-slate-500">
                                <div class="flex items-center gap-2"><span class="w-2.5 h-2.5 rounded-full bg-emerald-500 shadow-sm"></span> 出席</div>
                                <div class="flex items-center gap-2"><span class="w-2.5 h-2.5 rounded-full bg-rose-500 shadow-sm"></span> 缺席</div>
                                <div class="flex items-center gap-2"><span class="w-2.5 h-2.5 rounded-full bg-amber-400 shadow-sm"></span> 請假</div>
                            </div>
                        </div>

                        <!-- 表格區域 -->
                        <div class="overflow-auto flex-1 no-scrollbar relative">
                            <table class="w-full text-left border-collapse">
                                <thead class="bg-slate-50 sticky top-0 z-30 shadow-sm">
                                    <tr id="table-header">
                                        <!-- JS 生成表頭 -->
                                    </tr>
                                </thead>
                                <tbody id="table-body" class="text-slate-600 text-sm font-light divide-y divide-slate-50">
                                    <!-- JS 生成內容 -->
                                </tbody>
                                <tfoot id="table-footer" class="bg-slate-100 border-t-2 border-slate-200 text-slate-600 text-xs font-bold sticky bottom-0 z-30">
                                    <!-- JS 生成統計列 -->
                                </tfoot>
                            </table>
                        </div>
                        
                        <div class="bg-slate-50 px-8 py-4 text-xs text-slate-400 border-t border-slate-100 flex justify-between items-center rounded-b-2xl">
                            <span class="flex items-center gap-2"><span class="bg-slate-200 w-4 h-4 rounded text-center leading-4 text-[10px] text-slate-500">?</span> 點擊格子切換狀態</span>
                            <span id="last-updated" class="font-mono opacity-70">最後更新: 剛剛</span>
                        </div>
                    </div>

                    <!-- 空狀態 -->
                    <div id="empty-state" class="flex flex-col items-center justify-center h-full text-slate-300 py-20">
                        <div class="w-24 h-24 bg-slate-50 rounded-full flex items-center justify-center mb-6 text-4xl shadow-inner">👋</div>
                        <p class="text-slate-500 font-medium text-lg">請從左側選擇課程</p>
                        <p class="text-sm mt-2">開始管理您的學員與出勤紀錄</p>
                    </div>
                </main>
            </div>
        </div>
    </div>

    <!-- Modals (彈窗) -->

    <!-- 新增/編輯 課程 Modal -->
    <div id="class-modal" class="hidden fixed inset-0 z-50 flex items-center justify-center modal-backdrop fade-in">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-md p-8 m-4 transform transition-all scale-100">
            <h3 id="class-modal-title" class="text-xl font-bold text-slate-800 mb-6 flex items-center gap-2">
                <span class="bg-brand-100 text-brand-600 w-8 h-8 rounded-lg flex items-center justify-center text-sm">＋</span>
                建立新課程
            </h3>
            <form id="form-class" class="space-y-5">
                <input type="hidden" name="mode" id="class-mode" value="add">
                <input type="hidden" name="originalClassName" id="class-original-name">
                
                <div>
                    <label class="block text-xs font-bold text-slate-500 uppercase tracking-wider mb-2">課程名稱</label>
                    <input type="text" name="className" id="input-class-name" required placeholder="例如：兒童圍棋 A 班" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-brand-500 focus:border-brand-500 focus:bg-white transition-all outline-none text-slate-700 placeholder:text-slate-400">
                </div>
                
                <div>
                    <label class="block text-xs font-bold text-slate-500 uppercase tracking-wider mb-2">上課日期設定 (可複選)</label>
                    <div class="flex gap-2 mb-3 overflow-x-auto pb-1 no-scrollbar">
                        <!-- Checkboxes for Weekdays -->
                        <div class="flex-shrink-0">
                            <input type="checkbox" name="weekdays" value="1" id="wd-1" class="weekday-checkbox hidden">
                            <label for="wd-1" class="block w-10 h-10 leading-10 text-center rounded-lg border border-slate-200 bg-slate-50 text-slate-500 font-bold cursor-pointer transition-all hover:border-brand-300">一</label>
                        </div>
                        <div class="flex-shrink-0">
                            <input type="checkbox" name="weekdays" value="2" id="wd-2" class="weekday-checkbox hidden">
                            <label for="wd-2" class="block w-10 h-10 leading-10 text-center rounded-lg border border-slate-200 bg-slate-50 text-slate-500 font-bold cursor-pointer transition-all hover:border-brand-300">二</label>
                        </div>
                        <div class="flex-shrink-0">
                            <input type="checkbox" name="weekdays" value="3" id="wd-3" class="weekday-checkbox hidden">
                            <label for="wd-3" class="block w-10 h-10 leading-10 text-center rounded-lg border border-slate-200 bg-slate-50 text-slate-500 font-bold cursor-pointer transition-all hover:border-brand-300">三</label>
                        </div>
                        <div class="flex-shrink-0">
                            <input type="checkbox" name="weekdays" value="4" id="wd-4" class="weekday-checkbox hidden">
                            <label for="wd-4" class="block w-10 h-10 leading-10 text-center rounded-lg border border-slate-200 bg-slate-50 text-slate-500 font-bold cursor-pointer transition-all hover:border-brand-300">四</label>
                        </div>
                        <div class="flex-shrink-0">
                            <input type="checkbox" name="weekdays" value="5" id="wd-5" class="weekday-checkbox hidden">
                            <label for="wd-5" class="block w-10 h-10 leading-10 text-center rounded-lg border border-slate-200 bg-slate-50 text-slate-500 font-bold cursor-pointer transition-all hover:border-brand-300">五</label>
                        </div>
                        <div class="flex-shrink-0">
                            <input type="checkbox" name="weekdays" value="6" id="wd-6" class="weekday-checkbox hidden">
                            <label for="wd-6" class="block w-10 h-10 leading-10 text-center rounded-lg border border-slate-200 bg-slate-50 text-slate-500 font-bold cursor-pointer transition-all hover:border-brand-300">六</label>
                        </div>
                        <div class="flex-shrink-0">
                            <input type="checkbox" name="weekdays" value="0" id="wd-0" class="weekday-checkbox hidden">
                            <label for="wd-0" class="block w-10 h-10 leading-10 text-center rounded-lg border border-slate-200 bg-slate-50 text-slate-500 font-bold cursor-pointer transition-all hover:border-brand-300">日</label>
                        </div>
                    </div>
                    
                    <div class="grid grid-cols-2 gap-4">
                        <div>
                            <label class="block text-xs font-bold text-slate-500 uppercase tracking-wider mb-2">第一堂課日期</label>
                            <input type="date" name="startDate" id="input-start-date" required class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-brand-500 outline-none text-slate-700 cursor-pointer">
                        </div>
                        <div>
                            <label class="block text-xs font-bold text-slate-500 uppercase tracking-wider mb-2">上課時間</label>
                            <input type="time" name="time" id="input-time" required value="18:00" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-brand-500 outline-none text-slate-700 cursor-pointer">
                        </div>
                    </div>
                </div>
                
                <div>
                    <label class="block text-xs font-bold text-slate-500 uppercase tracking-wider mb-2">老師收費方式</label>
                    <input type="text" name="feeMethod" id="input-fee" required placeholder="例如：期繳 3000 元 / 12 堂" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-brand-500 focus:bg-white transition-all outline-none text-slate-700 placeholder:text-slate-400">
                </div>

                <div class="bg-indigo-50 p-4 rounded-xl text-xs text-indigo-700 flex items-start gap-2 mt-2">
                    <span class="text-lg">💡</span>
                    <span>系統將從「第一堂課日期」開始，依照勾選的星期，自動推算 <b>12 堂課</b> 的日期。</span>
                </div>

                <div class="flex justify-end gap-3 pt-6 border-t border-slate-100 mt-2">
                    <button type="button" onclick="window.closeModal('class-modal')" class="px-5 py-2.5 text-slate-500 hover:bg-slate-100 hover:text-slate-700 rounded-xl font-medium transition-colors">取消</button>
                    <button type="submit" class="px-6 py-2.5 bg-brand-600 text-white rounded-xl hover:bg-brand-700 font-medium shadow-lg shadow-brand-500/30 transition-all hover:-translate-y-0.5">確認</button>
                </div>
            </form>
        </div>
    </div>

    <!-- 新增/編輯 學生 Modal -->
    <div id="student-modal" class="hidden fixed inset-0 z-50 flex items-center justify-center modal-backdrop fade-in">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-sm p-8 m-4 transform transition-all scale-100">
            <h3 id="student-modal-title" class="text-xl font-bold text-slate-800 mb-6 flex items-center gap-2">
                <span class="bg-emerald-100 text-emerald-600 w-8 h-8 rounded-lg flex items-center justify-center text-sm">＋</span>
                新增學生
            </h3>
            <form id="form-student" class="space-y-5">
                <input type="hidden" name="mode" id="student-mode" value="add">
                <input type="hidden" name="studentId" id="student-id">

                <div>
                    <label class="block text-xs font-bold text-slate-500 uppercase tracking-wider mb-2">學生姓名</label>
                    <input type="text" name="studentName" id="input-student-name" required placeholder="請輸入姓名" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:bg-white transition-all outline-none text-slate-700">
                </div>
                <div>
                    <label class="block text-xs font-bold text-slate-500 uppercase tracking-wider mb-2">班級 (年級)</label>
                    <div class="relative">
                        <select name="level" id="input-student-level" class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-emerald-500 outline-none text-slate-700 appearance-none cursor-pointer">
                            <option value="小">小班</option>
                            <option value="中" selected>中班</option>
                            <option value="大">大班</option>
                            <option value="國小">國小</option>
                        </select>
                        <div class="absolute right-4 top-1/2 -translate-y-1/2 pointer-events-none text-slate-400">▼</div>
                    </div>
                </div>
                <div class="flex justify-end gap-3 pt-6 border-t border-slate-100 mt-2">
                    <button type="button" onclick="window.closeModal('student-modal')" class="px-5 py-2.5 text-slate-500 hover:bg-slate-100 hover:text-slate-700 rounded-xl font-medium transition-colors">取消</button>
                    <button type="submit" class="px-6 py-2.5 bg-emerald-500 text-white rounded-xl hover:bg-emerald-600 font-medium shadow-lg shadow-emerald-500/30 transition-all hover:-translate-y-0.5">確認</button>
                </div>
            </form>
        </div>
    </div>


    <!-- 3. 應用程式邏輯 (JavaScript Module) -->
    <script type="module">
        // 匯入 Firebase SDK (使用 CDN)
        import { initializeApp } from 'https://www.gstatic.com/firebasejs/11.0.2/firebase-app.js';
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'https://www.gstatic.com/firebasejs/11.0.2/firebase-auth.js';
        import { getFirestore, doc, onSnapshot, collection, setDoc, deleteDoc } from 'https://www.gstatic.com/firebasejs/11.0.2/firebase-firestore.js';

        // --- 全域變數與狀態 ---
        let db, auth;
        let currentUser = null;
        let appId = 'default-app-id';
        let sortableInstance = null; // Sortable 實例
        
        // 應用程式狀態 (State)
        const state = {
            classes: {},      // 儲存所有班級資料
            selectedClass: null,
            loading: true,
            demoMode: false
        };

        // 模擬資料 (Mock Data)
        const MOCK_CLASSES = [
            {
                className: '舞蹈 (週五班)',
                schedule: '每週五 16:30',
                feeMethod: '期繳 3000元',
                // 擴充至 8-12 堂範例
                dates: ['8/2', '8/9', '8/16', '8/23', '8/30', '9/6', '9/13', '9/20'], 
                students: [
                    { id: '1', name: '陳恬恩', level: '中', paid: true, attendance: { '8/2': 'present', '8/9': 'present' } },
                    { id: '2', name: '黃羽萱', level: '中', paid: true, attendance: { '8/2': 'present', '8/9': 'leave' } },
                    { id: '3', name: '陳泱合', level: '大', paid: false, attendance: { '8/2': 'absent', '8/9': 'present' } },
                    { id: '4', name: '尤彥程', level: '小', paid: false, attendance: { '8/2': 'present', '8/9': 'present' } },
                    { id: '5', name: '李宥芯', level: '國小', paid: true, attendance: {} },
                ],
            },
            {
                className: '美語 (週二班)',
                schedule: '每週二 18:00',
                feeMethod: '月繳 1200元',
                dates: ['8/6', '8/13', '8/20', '8/27', '9/3', '9/10'], 
                students: [
                    { id: '1', name: '林侑謙', level: '中', paid: true, attendance: { '8/6': 'present' } },
                    { id: '2', name: '陳安欣', level: '大', paid: true, attendance: { '8/6': 'present' } },
                    { id: '3', name: '謝思芹', level: '小', paid: false, attendance: { '8/6': 'absent' } },
                ],
            },
        ];

        // --- 初始化 App ---
        async function initApp() {
            try {
                const configStr = typeof __firebase_config !== 'undefined' ? __firebase_config : '{}';
                const firebaseConfig = JSON.parse(configStr);
                
                if (Object.keys(firebaseConfig).length === 0) {
                    console.warn("未偵測到 Firebase 設定，啟動展示模式。");
                    startDemoMode();
                    return;
                }

                if (typeof __app_id !== 'undefined') appId = __app_id;

                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                const token = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
                if (token) {
                    await signInWithCustomToken(auth, token);
                } else {
                    await signInAnonymously(auth);
                }

                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        currentUser = user;
                        document.getElementById('user-display').innerText = `ID: ${user.uid.slice(0, 6)}`;
                        startDataSync(); 
                    } else {
                        currentUser = { uid: 'anonymous' };
                        startDataSync();
                    }
                });

            } catch (error) {
                console.error("Initialization failed, switching to Demo Mode:", error);
                startDemoMode();
            }
        }

        function startDemoMode() {
            state.demoMode = true;
            state.loading = false;
            
            MOCK_CLASSES.forEach(c => {
                state.classes[c.className] = JSON.parse(JSON.stringify(c));
            });

            const banner = document.getElementById('system-banner');
            banner.className = "bg-amber-50 border border-amber-200 text-amber-800 p-4 rounded-xl mb-8 shadow-sm block";
            document.getElementById('banner-title').innerText = "⚠️ 展示模式 (Demo Mode)";
            document.getElementById('banner-text').innerHTML = "目前使用<b>本機模擬資料</b>。您可以自由操作，但重新整理後資料將重置。";
            document.getElementById('system-banner').classList.remove('hidden');

            document.getElementById('user-display').innerText = "Demo";
            finishLoading();
        }

        function startDataSync() {
            const colPath = `artifacts/${appId}/public/data/class_rosters_grid`;
            const colRef = collection(db, colPath);

            seedDataIfNeeded(colRef);

            onSnapshot(colRef, (snapshot) => {
                const newClasses = {};
                snapshot.forEach(doc => {
                    newClasses[doc.id] = doc.data();
                });
                state.classes = newClasses;
                
                // 如果目前選中的課程被刪除了，清空選取
                if (state.selectedClass && !newClasses[state.selectedClass]) {
                    state.selectedClass = null;
                }
                
                finishLoading();
            }, (error) => {
                console.error("Data sync error:", error);
                startDemoMode();
            });
        }

        async function seedDataIfNeeded(colRef) {
            for (const c of MOCK_CLASSES) {
                await setDoc(doc(colRef, c.className), c, { merge: true });
            }
        }

        function finishLoading() {
            document.getElementById('loading-screen').classList.add('hidden');
            document.getElementById('main-interface').classList.remove('hidden');

            if (!state.selectedClass && Object.keys(state.classes).length > 0) {
                selectClass(Object.keys(state.classes)[0]);
            } else if (state.selectedClass) {
                renderGrid(state.classes[state.selectedClass]);
            } else {
                // 如果沒有選中且沒有課程
                document.getElementById('grid-container').classList.add('hidden');
                document.getElementById('empty-state').classList.remove('hidden');
            }
            renderSidebar();
        }

        // --- Helper: 日期計算邏輯 (修正版) ---
        // 根據起始日與選定的星期，推算後續日期
        function calculateClassDates(startDateStr, selectedWeekdays, count = 12) {
            const results = [];
            const start = new Date(startDateStr);
            let current = new Date(start);
            
            // 避免無窮迴圈防呆
            let safetyCounter = 0;
            
            while (results.length < count && safetyCounter < 365) {
                const day = current.getDay(); // 0-6
                
                if (selectedWeekdays.includes(day)) {
                    results.push(`${current.getMonth()+1}/${current.getDate()}`);
                }
                
                // 加一天
                current.setDate(current.getDate() + 1);
                safetyCounter++;
            }
            
            return results;
        }

        // --- UI 渲染邏輯 ---

        function renderSidebar() {
            const listEl = document.getElementById('class-list');
            listEl.innerHTML = ''; 
            document.getElementById('class-count-badge').innerText = Object.keys(state.classes).length;

            Object.values(state.classes).forEach(cls => {
                const div = document.createElement('div');
                const isSelected = state.selectedClass === cls.className;
                
                div.className = `w-full px-4 py-3.5 rounded-xl transition-all duration-200 group flex justify-between items-center cursor-pointer ${
                    isSelected 
                    ? 'bg-slate-900 text-white shadow-md shadow-slate-900/10' 
                    : 'bg-white text-slate-600 hover:bg-slate-50 hover:text-slate-900'
                }`;
                
                div.innerHTML = `
                    <div class="flex-1 min-w-0 pr-3" onclick="window.selectClass('${cls.className}')">
                        <div class="font-medium truncate text-sm">${cls.className}</div>
                        <div class="text-xs mt-1 ${isSelected ? 'text-slate-300' : 'text-slate-400'}">${cls.students.length} 位學生</div>
                    </div>
                    <div class="flex gap-2 opacity-0 group-hover:opacity-100 transition-opacity">
                         <button onclick="window.openClassModal('edit', '${cls.className}')" class="p-1 hover:bg-white/20 rounded text-xs" title="編輯">✏️</button>
                         <button onclick="window.deleteClass('${cls.className}')" class="p-1 hover:bg-red-500/20 hover:text-red-400 rounded text-xs" title="刪除">🗑️</button>
                    </div>
                `;
                
                listEl.appendChild(div);
            });
        }

        function renderGrid(clsData) {
            if (!clsData) return;

            document.getElementById('current-class-title').innerText = clsData.className;
            document.getElementById('class-schedule-display').innerHTML = `🕒 ${clsData.schedule || '未設定'}`;
            document.getElementById('class-fee-display').innerHTML = `💰 ${clsData.feeMethod || '未設定'}`;

            document.getElementById('grid-container').classList.remove('hidden');
            document.getElementById('empty-state').classList.add('hidden');

            // 1. 渲染表頭 (Sticky Header)
            const thead = document.getElementById('table-header');
            // 座號、姓名等固定欄
            let headerHTML = `
                <th class="py-4 px-4 text-left font-bold text-xs uppercase tracking-wider text-slate-400 w-16 sticky left-0 bg-slate-50 z-20 border-b border-slate-100">座號</th>
                <th class="py-4 px-4 text-left font-bold text-xs uppercase tracking-wider text-slate-400 w-40 sticky left-16 bg-slate-50 z-20 border-b border-slate-100 shadow-[2px_0_5px_-2px_rgba(0,0,0,0.05)]">姓名 / 操作</th>
                <th class="py-4 px-4 text-center font-bold text-xs uppercase tracking-wider text-slate-400 w-20 border-b border-slate-100">班別</th>
                <th class="py-4 px-4 text-center font-bold text-xs uppercase tracking-wider text-slate-400 w-24 border-b border-slate-100">繳費</th>
            `;
            
            // 日期欄
            clsData.dates.forEach(date => {
                headerHTML += `
                    <th class="py-3 px-2 text-center border-b border-slate-100 min-w-[70px]">
                        <div class="inline-block px-2.5 py-1 rounded-md bg-white border border-slate-100 shadow-sm text-xs font-medium text-slate-600">${date}</div>
                    </th>
                `;
            });
            thead.innerHTML = headerHTML;

            // 2. 渲染表格內容
            const tbody = document.getElementById('table-body');
            tbody.innerHTML = ''; 
            
            // 初始化每日出席計數器
            const attendanceCounts = {}; 
            clsData.dates.forEach(d => attendanceCounts[d] = 0);

            clsData.students.forEach((student, index) => {
                const tr = document.createElement('tr');
                tr.className = "table-row-hover transition-colors group";
                // 設定 data-id 供 Sortable 使用
                tr.setAttribute('data-id', student.id);
                
                const levelColors = {
                    '大': 'bg-indigo-50 text-indigo-600 border border-indigo-100',
                    '中': 'bg-emerald-50 text-emerald-600 border border-emerald-100',
                    '小': 'bg-rose-50 text-rose-600 border border-rose-100',
                    '國小': 'bg-orange-50 text-orange-600 border border-orange-100' // 新增國小樣式
                };
                const levelClass = levelColors[student.level] || 'bg-slate-100 text-slate-600';

                const paidClass = student.paid ? 'bg-brand-500' : 'bg-slate-200';
                const knobClass = student.paid ? 'translate-x-6' : 'translate-x-1';
                const paidText = student.paid 
                    ? '<span class="text-brand-600 font-bold">已繳</span>' 
                    : '<span class="text-slate-400">未繳</span>';

                // 座號改為顯示 index + 1
                let rowHTML = `
                    <td class="py-4 px-4 text-left whitespace-nowrap sticky left-0 bg-white z-10 sticky-col font-mono text-slate-400 text-sm border-b border-slate-50">${index + 1}</td>
                    <td class="py-4 px-4 text-left font-bold text-slate-700 sticky left-16 bg-white z-10 sticky-col shadow-[2px_0_5px_-2px_rgba(0,0,0,0.05)] border-b border-slate-50 group">
                        <div class="flex items-center justify-between">
                            <div class="flex items-center gap-2">
                                <span class="drag-handle cursor-grab opacity-0 group-hover:opacity-100 transition-opacity" title="拖曳排序">⋮⋮</span>
                                <span>${student.name}</span>
                            </div>
                            <div class="action-buttons opacity-0 group-hover:opacity-100 transition-opacity flex gap-1">
                                <button onclick="window.openStudentModal('edit', '${student.id}')" class="text-xs text-slate-400 hover:text-brand-600 p-1">✏️</button>
                                <button onclick="window.deleteStudent('${student.id}')" class="text-xs text-slate-400 hover:text-red-600 p-1">🗑️</button>
                            </div>
                        </div>
                    </td>
                    <td class="py-4 px-4 text-center border-b border-slate-50">
                        <span class="px-2.5 py-1 rounded-lg text-xs font-semibold ${levelClass}">${student.level}</span>
                    </td>
                    <td class="py-4 px-4 text-center cursor-pointer border-b border-slate-50 group/paid" onclick="window.togglePayment('${student.id}')">
                         <div class="w-11 h-6 mx-auto rounded-full relative transition-colors duration-200 ease-in-out ${paidClass} shadow-inner">
                            <div class="absolute w-4 h-4 bg-white rounded-full top-1 transition-transform duration-200 ease-in-out shadow-sm ${knobClass}"></div>
                         </div>
                         <div class="text-[10px] mt-1.5 opacity-80 group-hover/paid:opacity-100 transition-opacity">${paidText}</div>
                    </td>
                `;

                clsData.dates.forEach(date => {
                    const status = student.attendance[date] || 'none';
                    
                    // 統計人數
                    if (status === 'present') attendanceCounts[date]++;

                    let cellContent = '<div class="w-2 h-2 rounded-full bg-slate-100 group-hover:bg-slate-200 transition-colors"></div>'; 
                    let cellContainerClass = 'hover:bg-slate-50';

                    if (status === 'present') {
                        cellContent = '<div class="w-8 h-8 rounded-full bg-emerald-100 text-emerald-600 flex items-center justify-center shadow-sm">✓</div>';
                    } else if (status === 'absent') {
                        cellContent = '<div class="w-8 h-8 rounded-full bg-rose-100 text-rose-600 flex items-center justify-center shadow-sm">✕</div>';
                    } else if (status === 'leave') {
                        cellContent = '<div class="w-8 h-8 rounded-full bg-amber-100 text-amber-600 flex items-center justify-center shadow-sm">○</div>';
                    }

                    rowHTML += `
                        <td class="py-3 px-2 text-center border-l border-slate-50 cursor-pointer select-none transition-colors ${cellContainerClass}" 
                            onclick="window.toggleAttendance('${student.id}', '${date}')">
                            <div class="flex items-center justify-center transition-transform active:scale-95">
                                ${cellContent}
                            </div>
                        </td>
                    `;
                });

                tr.innerHTML = rowHTML;
                tbody.appendChild(tr);
            });

            // 3. 渲染頁尾統計列 (Table Footer)
            const tfoot = document.getElementById('table-footer');
            let footerHTML = `
                <tr class="summary-row">
                    <td class="py-4 px-4 sticky left-0 z-20 sticky-col border-t border-slate-200" colspan="2">📊 出席統計</td>
                    <td class="py-4 px-4 border-t border-slate-200" colspan="2"></td>
            `;
            
            clsData.dates.forEach(date => {
                const count = attendanceCounts[date];
                footerHTML += `
                    <td class="py-4 px-2 text-center border-t border-slate-200 border-l border-slate-200 text-slate-600">
                        ${count} 人
                    </td>
                `;
            });
            footerHTML += `</tr>`;
            tfoot.innerHTML = footerHTML;

            // 4. 初始化 SortableJS
            if (sortableInstance) sortableInstance.destroy();
            
            sortableInstance = new Sortable(tbody, {
                handle: '.drag-handle', // 只能透過這個 class 拖曳
                animation: 150,
                ghostClass: 'sortable-ghost',
                dragClass: 'sortable-drag',
                onEnd: async function (evt) {
                    // 更新本地資料順序
                    const currentClass = state.classes[state.selectedClass];
                    const item = currentClass.students.splice(evt.oldIndex, 1)[0];
                    currentClass.students.splice(evt.newIndex, 0, item);
                    
                    // 重新渲染以更新座號 (index)
                    renderGrid(currentClass);
                    
                    // 存檔
                    await saveDataToDB(state.selectedClass, currentClass);
                },
            });
        }

        // --- 互動邏輯 ---

        window.selectClass = (className) => {
            state.selectedClass = className;
            renderSidebar(); 
            renderGrid(state.classes[className]);
        };

        window.togglePayment = (studentId) => {
            const cls = state.classes[state.selectedClass];
            const student = cls.students.find(s => s.id === studentId);
            if (student) {
                student.paid = !student.paid;
                renderGrid(cls);
            }
        };

        window.toggleAttendance = (studentId, date) => {
            const cls = state.classes[state.selectedClass];
            const student = cls.students.find(s => s.id === studentId);
            if (student) {
                const current = student.attendance[date] || 'none';
                let next = 'present';
                if (current === 'present') next = 'absent';
                else if (current === 'absent') next = 'leave';
                else if (current === 'leave') next = 'none';
                
                student.attendance[date] = next;
                renderGrid(cls);
            }
        };

        // --- 匯出 Excel (CSV) 邏輯 ---
        document.getElementById('export-btn').onclick = () => {
            if (!state.selectedClass) {
                alert('請先選擇一個課程');
                return;
            }
            const clsData = state.classes[state.selectedClass];
            
            // 準備 CSV 內容
            let csvContent = "座號,姓名,班級,繳費狀態," + clsData.dates.join(",") + "\n";
            
            clsData.students.forEach((student, index) => {
                let row = [
                    index + 1, // 座號使用 index + 1
                    student.name,
                    student.level + "班",
                    student.paid ? "已繳" : "未繳"
                ];
                
                clsData.dates.forEach(date => {
                    const status = student.attendance[date];
                    let statusText = "";
                    if (status === 'present') statusText = "出席";
                    else if (status === 'absent') statusText = "缺席";
                    else if (status === 'leave') statusText = "請假";
                    row.push(statusText);
                });
                
                csvContent += row.join(",") + "\n";
            });

            const counts = clsData.dates.map(date => {
                return clsData.students.filter(s => s.attendance[date] === 'present').length;
            });
            csvContent += ",,總計人數,," + counts.join("人,") + "人\n";

            const blob = new Blob(["\uFEFF" + csvContent], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            const link = document.createElement("a");
            link.setAttribute("href", url);
            link.setAttribute("download", `${clsData.className}_點名表.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        };

        // --- 匯入 Excel 邏輯 ---
        
        window.triggerImport = () => {
            document.getElementById('import-file-input').click();
        };

        document.getElementById('import-file-input').onchange = (e) => {
            const file = e.target.files[0];
            if (!file) return;

            if (!state.selectedClass) {
                alert("請先選擇要匯入的班級！");
                e.target.value = ''; // Reset input
                return;
            }

            const reader = new FileReader();
            reader.onload = async (e) => {
                try {
                    const data = new Uint8Array(e.target.result);
                    const workbook = XLSX.read(data, { type: 'array' });
                    
                    // 假設讀取第一個工作表
                    const firstSheetName = workbook.SheetNames[0];
                    const worksheet = workbook.Sheets[firstSheetName];
                    
                    // 轉為 JSON
                    const jsonData = XLSX.utils.sheet_to_json(worksheet);
                    
                    if (jsonData.length === 0) {
                        alert("Excel 檔案中沒有資料！");
                        return;
                    }

                    // 處理資料並加入目前班級
                    const currentClass = state.classes[state.selectedClass];
                    let addedCount = 0;

                    jsonData.forEach(row => {
                        // 嘗試尋找姓名欄位 (支援多種常見命名)
                        const name = row['姓名'] || row['Name'] || row['name'] || row['學生姓名'];
                        // 嘗試尋找班級欄位
                        let level = row['班級'] || row['Level'] || row['level'] || row['年級'] || '中'; // 預設中班

                        if (name) {
                            // 簡單的年級正規化
                            if (level.includes('小')) level = '小';
                            else if (level.includes('大')) level = '大';
                            else if (level.includes('國小')) level = '國小';
                            else if (level.includes('中')) level = '中';
                            else level = '中'; // Fallback

                            const newId = Date.now().toString() + Math.floor(Math.random() * 1000); // 確保唯一ID
                            const newStudent = {
                                id: newId,
                                name: String(name).trim(),
                                level: level,
                                paid: false,
                                attendance: {}
                            };
                            
                            currentClass.students.push(newStudent);
                            addedCount++;
                        }
                    });

                    if (addedCount > 0) {
                        await saveDataToDB(state.selectedClass, currentClass);
                        renderGrid(currentClass);
                        renderSidebar();
                        alert(`成功匯入 ${addedCount} 位學生！`);
                    } else {
                        alert("找不到符合格式的資料，請確認 Excel 包含「姓名」欄位。");
                    }

                } catch (error) {
                    console.error("Import error:", error);
                    alert("讀取檔案失敗，請確認檔案格式正確 (.xlsx, .xls, .csv)。");
                } finally {
                    document.getElementById('import-file-input').value = ''; // Reset
                }
            };
            reader.readAsArrayBuffer(file);
        };


        // --- Modal & Form Logic (新版) ---

        window.closeModal = (id) => {
            document.getElementById(id).classList.add('hidden');
        };

        // 開啟課程 Modal (新增或編輯)
        window.openClassModal = (mode, className = null) => {
            document.getElementById('class-modal').classList.remove('hidden');
            const form = document.getElementById('form-class');
            form.reset();
            
            // 清除 Checkbox 選擇
            document.querySelectorAll('input[name="weekdays"]').forEach(cb => cb.checked = false);
            
            document.getElementById('class-mode').value = mode;
            
            if (mode === 'edit' && className) {
                const cls = state.classes[className];
                document.getElementById('class-modal-title').innerText = '✏️ 編輯課程';
                document.getElementById('class-original-name').value = className;
                document.getElementById('input-class-name').value = cls.className;
                document.getElementById('input-fee').value = cls.feeMethod;
                // 解析時間 (簡易版)
                const parts = cls.schedule.split(' ');
                if (parts.length > 1) {
                    // 這裡的 parsing 可能比較脆弱，如果格式不同可能需要調整
                    // 假設格式 "每週X 18:00" 或 "每週一、三 18:00"
                    const timePart = parts[parts.length - 1]; // 取最後一段當時間
                    if (timePart.includes(':')) {
                        document.getElementById('input-time').value = timePart;
                    }
                }
            } else {
                document.getElementById('class-modal-title').innerHTML = '<span class="bg-brand-100 text-brand-600 w-8 h-8 rounded-lg flex items-center justify-center text-sm">＋</span> 建立新課程';
                document.getElementById('input-time').value = '18:00';
            }
        };

        // 處理課程提交 (支援多日)
        document.getElementById('form-class').onsubmit = async (e) => {
            e.preventDefault();
            const formData = new FormData(e.target);
            const mode = formData.get('mode');
            const className = formData.get('className');
            const originalClassName = formData.get('originalClassName');
            const startDateStr = formData.get('startDate'); // YYYY-MM-DD
            const time = formData.get('time');
            const feeMethod = formData.get('feeMethod');
            
            // 取得所有勾選的星期
            const weekdays = [];
            document.querySelectorAll('input[name="weekdays"]:checked').forEach(cb => {
                weekdays.push(parseInt(cb.value));
            });

            if (!className) return;

            let dates = [];
            let scheduleStr = "";

            if (startDateStr && weekdays.length > 0) {
                // 如果有選日期與星期 (新增模式 or 編輯且有改)
                dates = calculateClassDates(startDateStr, weekdays, 12);

                const weekNames = ["日", "一", "二", "三", "四", "五", "六"];
                // 排序選中的星期以利顯示 (例如：週一、三、五)
                weekdays.sort((a,b) => {
                    // 把週日(0)排到最後顯示，或者照 1-6, 0 順序? 
                    // 習慣上週一~週日: 1,2,3,4,5,6,0
                    const map = [7, 1, 2, 3, 4, 5, 6];
                    return map[a] - map[b];
                });
                
                const dayNames = weekdays.map(d => weekNames[d]).join("、");
                scheduleStr = `每週${dayNames} ${time}`;

            } else if (mode === 'edit') {
                // 編輯模式且沒改日期 -> 沿用舊資料
                const oldClass = state.classes[originalClassName];
                dates = oldClass.dates;
                // 嘗試更新時間部分，保留前面敘述
                const parts = oldClass.schedule.split(' ');
                // 假設最後一部分是時間，替換掉
                parts[parts.length - 1] = time;
                scheduleStr = parts.join(' ');
            } else {
                // 新增模式但未填寫完整 (雖然有 required，但多重防護)
                if(weekdays.length === 0) {
                    alert("請至少選擇一個上課日 (週一~週日)");
                    return;
                }
            }

            // 準備新物件
            const newClass = {
                className,
                schedule: scheduleStr,
                feeMethod,
                dates: dates,
                students: []
            };

            if (mode === 'edit') {
                const oldClass = state.classes[originalClassName];
                // 保留舊學生資料
                newClass.students = oldClass.students;
                
                if (originalClassName !== className) {
                    delete state.classes[originalClassName]; // 本地刪除舊key
                    await deleteDataFromDB(originalClassName); // DB刪除舊doc
                }
            }

            state.classes[className] = newClass;
            state.selectedClass = className; 
            
            await saveDataToDB(className, newClass);
            
            window.closeModal('class-modal');
            renderSidebar();
            renderGrid(newClass);
        };
        
        // 刪除課程
        window.deleteClass = async (className) => {
            if (!confirm(`確定要刪除「${className}」嗎？此動作無法復原。`)) return;
            
            event.stopPropagation();

            delete state.classes[className];
            if (state.selectedClass === className) {
                state.selectedClass = null;
                document.getElementById('grid-container').classList.add('hidden');
                document.getElementById('empty-state').classList.remove('hidden');
            }
            
            await deleteDataFromDB(className);
            renderSidebar();
        };


        // 開啟學生 Modal (新增或編輯)
        window.openStudentModal = (mode, studentId = null) => {
            document.getElementById('student-modal').classList.remove('hidden');
            const form = document.getElementById('form-student');
            form.reset();
            
            document.getElementById('student-mode').value = mode;
            const currentClass = state.classes[state.selectedClass];

            if (mode === 'edit' && studentId) {
                document.getElementById('student-modal-title').innerText = '✏️ 編輯學生';
                document.getElementById('student-id').value = studentId;
                
                const student = currentClass.students.find(s => s.id === studentId);
                if (student) {
                    document.getElementById('input-student-name').value = student.name;
                    document.getElementById('input-student-level').value = student.level;
                }
            } else {
                document.getElementById('student-modal-title').innerHTML = '<span class="bg-emerald-100 text-emerald-600 w-8 h-8 rounded-lg flex items-center justify-center text-sm">＋</span> 新增學生';
            }
        };

        // 處理學生提交
        document.getElementById('form-student').onsubmit = async (e) => {
            e.preventDefault();
            const formData = new FormData(e.target);
            const mode = formData.get('mode');
            const studentId = formData.get('studentId');
            const name = formData.get('studentName');
            const level = formData.get('level');
            
            const currentClass = state.classes[state.selectedClass];
            if (!currentClass) return;

            if (mode === 'add') {
                const newId = Date.now().toString() + Math.floor(Math.random() * 1000); // 確保唯一ID
                const newStudent = {
                    id: newId,
                    name: name,
                    level: level,
                    paid: false,
                    attendance: {}
                };
                currentClass.students.push(newStudent);
            } else {
                // Edit
                const student = currentClass.students.find(s => s.id === studentId);
                if (student) {
                    student.name = name;
                    student.level = level;
                }
            }

            await saveDataToDB(state.selectedClass, currentClass);
            window.closeModal('student-modal');
            renderGrid(currentClass);
            renderSidebar();
        };

        // 刪除學生
        window.deleteStudent = async (studentId) => {
            if (!confirm('確定要移除這位學生嗎？')) return;
            
            const currentClass = state.classes[state.selectedClass];
            currentClass.students = currentClass.students.filter(s => s.id !== studentId);
            
            await saveDataToDB(state.selectedClass, currentClass);
            renderGrid(currentClass);
            renderSidebar();
        };


        // --- 資料庫儲存 ---

        async function saveDataToDB(docId, data) {
            if (state.demoMode) return;
            try {
                const docPath = `artifacts/${appId}/public/data/class_rosters_grid/${docId}`;
                await setDoc(doc(db, `artifacts/${appId}/public/data/class_rosters_grid`, docId), data);
            } catch (e) {
                console.error("Save error:", e);
                alert("儲存失敗: " + e.message);
            }
        }
        
        async function deleteDataFromDB(docId) {
            if (state.demoMode) return;
            try {
                 await deleteDoc(doc(db, `artifacts/${appId}/public/data/class_rosters_grid`, docId));
            } catch (e) {
                console.error("Delete error:", e);
            }
        }

        document.getElementById('save-btn').onclick = async () => {
            const btn = document.getElementById('save-btn');
            const originalText = btn.innerHTML;
            btn.innerHTML = '<span>儲存中...</span>';
            btn.disabled = true;
            btn.classList.add('opacity-70');

            if (state.demoMode) {
                setTimeout(() => {
                    btn.innerHTML = '<span>僅本機更新</span>';
                    setTimeout(() => { 
                        btn.innerHTML = originalText; 
                        btn.disabled = false;
                        btn.classList.remove('opacity-70');
                    }, 1500);
                }, 500);
                return;
            }

            if (!currentUser || !state.selectedClass) {
                btn.innerHTML = originalText;
                btn.disabled = false;
                return;
            }

            try {
                const clsData = state.classes[state.selectedClass];
                await saveDataToDB(state.selectedClass, clsData);
                
                btn.innerHTML = '<span>✓ 已儲存</span>';
                setTimeout(() => {
                    btn.innerHTML = originalText;
                    btn.disabled = false;
                    btn.classList.remove('opacity-70');
                }, 1500);
            } catch (e) {
                btn.innerHTML = originalText;
                btn.disabled = false;
            }
        };

        // 啟動 App
        initApp();

    </script>
</body>
</html>
