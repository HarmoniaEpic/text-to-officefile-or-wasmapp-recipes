# **ASMSCRIPT-OPTIMIZER-RECIPE-HTMLTEMPLATE v1.4.0**
**AssemblyScript WebAssembly HTMLテンプレート**

このファイルは、ASMSCRIPT-OPTIMIZER-RECIPE v1.4.0で使用する完全なHTMLテンプレートコードです。

## **HTMLテンプレート全文**

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AssemblyScript WebAssembly Compiler</title>
    <style>
        /* ============================================
           グローバルスタイル
        ============================================ */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        :root {
            --primary-color: #667eea;
            --secondary-color: #764ba2;
            --success-color: #48bb78;
            --error-color: #f56565;
            --warning-color: #ed8936;
            --bg-primary: #1a202c;
            --bg-secondary: #2d3748;
            --bg-tertiary: #4a5568;
            --text-primary: #e2e8f0;
            --text-secondary: #a0aec0;
            --border-color: #4a5568;
            --code-bg: #2d3748;
            --sidebar-width: 320px;
            --header-height: 60px;
        }
        
        body {
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Helvetica', Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: var(--text-primary);
            min-height: 100vh;
            overflow-x: hidden;
        }
        
        /* ============================================
           レイアウト構造
        ============================================ */
        .app-container {
            display: flex;
            min-height: 100vh;
            background: var(--bg-primary);
        }
        
        /* ============================================
           サイドバー
        ============================================ */
        .sidebar {
            width: var(--sidebar-width);
            background: var(--bg-secondary);
            border-right: 1px solid var(--border-color);
            display: flex;
            flex-direction: column;
            transition: transform 0.3s ease;
            position: fixed;
            height: 100vh;
            z-index: 1000;
            overflow-y: auto;
        }
        
        .sidebar.closed {
            transform: translateX(calc(-1 * var(--sidebar-width)));
        }
        
        .sidebar-header {
            padding: 20px;
            border-bottom: 1px solid var(--border-color);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        .sidebar-title {
            font-size: 1.2rem;
            font-weight: 600;
            color: var(--primary-color);
        }
        
        /* ============================================
           タブシステム
        ============================================ */
        .tab-navigation {
            display: flex;
            border-bottom: 1px solid var(--border-color);
            background: var(--bg-tertiary);
        }
        
        .tab-button {
            flex: 1;
            padding: 12px;
            background: transparent;
            border: none;
            color: var(--text-secondary);
            cursor: pointer;
            font-size: 0.9rem;
            transition: all 0.2s;
        }
        
        .tab-button.active {
            color: var(--primary-color);
            background: var(--bg-secondary);
            border-bottom: 2px solid var(--primary-color);
        }
        
        .tab-content {
            display: none;
            padding: 20px;
            animation: fadeIn 0.3s;
        }
        
        .tab-content.active {
            display: block;
        }
        
        /* ============================================
           コンパイルオプション
        ============================================ */
        .option-section {
            margin-bottom: 24px;
        }
        
        .option-section-title {
            font-size: 0.85rem;
            text-transform: uppercase;
            letter-spacing: 0.05em;
            color: var(--text-secondary);
            margin-bottom: 12px;
        }
        
        .option-group {
            display: flex;
            flex-direction: column;
            gap: 8px;
        }
        
        .option-item {
            display: flex;
            align-items: center;
            padding: 8px 12px;
            background: var(--bg-tertiary);
            border-radius: 6px;
            transition: background 0.2s;
        }
        
        .option-item:hover {
            background: rgba(102, 126, 234, 0.1);
        }
        
        .option-item input[type="checkbox"] {
            margin-right: 10px;
            width: 16px;
            height: 16px;
            cursor: pointer;
        }
        
        .option-item label {
            flex: 1;
            cursor: pointer;
            font-size: 0.9rem;
        }
        
        .option-item select {
            background: var(--bg-secondary);
            color: var(--text-primary);
            border: 1px solid var(--border-color);
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 0.9rem;
        }
        
        /* ============================================
           プリセットボタン
        ============================================ */
        .preset-buttons {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 8px;
            margin-bottom: 20px;
        }
        
        .preset-button {
            padding: 10px;
            background: var(--bg-tertiary);
            border: 1px solid var(--border-color);
            border-radius: 6px;
            color: var(--text-primary);
            cursor: pointer;
            transition: all 0.2s;
            font-size: 0.85rem;
        }
        
        .preset-button:hover {
            background: var(--primary-color);
            transform: translateY(-1px);
        }
        
        /* ============================================
           メインコンテンツ
        ============================================ */
        .main-content {
            flex: 1;
            margin-left: 0;
            display: flex;
            flex-direction: column;
            transition: margin-left 0.3s ease;
        }
        
        .main-content.sidebar-open {
            margin-left: var(--sidebar-width);
        }
        
        /* ============================================
           ヘッダー
        ============================================ */
        .header {
            height: var(--header-height);
            background: var(--bg-secondary);
            border-bottom: 1px solid var(--border-color);
            display: flex;
            align-items: center;
            padding: 0 20px;
            justify-content: space-between;
        }
        
        .hamburger-menu {
            width: 30px;
            height: 24px;
            cursor: pointer;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
        }
        
        .hamburger-line {
            width: 100%;
            height: 3px;
            background: var(--text-primary);
            border-radius: 2px;
            transition: all 0.3s;
        }
        
        .header-title {
            font-size: 1.5rem;
            font-weight: 600;
            background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
            -webkit-background-clip: text;
            background-clip: text;
            -webkit-text-fill-color: transparent;
        }
        
        .header-actions {
            display: flex;
            gap: 12px;
        }
        
        .action-button {
            padding: 8px 16px;
            background: var(--primary-color);
            border: none;
            border-radius: 6px;
            color: white;
            cursor: pointer;
            font-size: 0.9rem;
            transition: all 0.2s;
            display: flex;
            align-items: center;
            gap: 6px;
        }
        
        .action-button:hover {
            background: var(--secondary-color);
            transform: translateY(-1px);
        }
        
        .action-button.secondary {
            background: var(--bg-tertiary);
        }
        
        /* ============================================
           コンテンツエリア
        ============================================ */
        .content-area {
            flex: 1;
            display: flex;
            flex-direction: column;
            padding: 20px;
            overflow-y: auto;
        }
        
        /* ============================================
           エディターとビューポート
        ============================================ */
        .workspace {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            flex: 1;
            min-height: 0;
        }
        
        .workspace-panel {
            background: var(--bg-secondary);
            border: 1px solid var(--border-color);
            border-radius: 8px;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }
        
        .panel-header {
            padding: 12px 16px;
            background: var(--bg-tertiary);
            border-bottom: 1px solid var(--border-color);
            font-weight: 500;
            font-size: 0.9rem;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        .panel-content {
            flex: 1;
            padding: 16px;
            overflow: auto;
        }
        
        /* ============================================
           コードエディター
        ============================================ */
        #code-editor {
            width: 100%;
            height: 100%;
            min-height: 400px;
            background: var(--code-bg);
            color: var(--text-primary);
            border: none;
            outline: none;
            font-family: 'Fira Code', 'Consolas', monospace;
            font-size: 14px;
            line-height: 1.6;
            padding: 16px;
            resize: none;
        }
        
        /* ============================================
           アプリケーションビューポート
        ============================================ */
        #app-viewport {
            background: #f8f9fa;
            border-radius: 4px;
            padding: 20px;
            min-height: 400px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: #2d3748;
        }
        
        /* ============================================
           統計情報パネル
        ============================================ */
        .stats-panel {
            background: var(--bg-secondary);
            border: 1px solid var(--border-color);
            border-radius: 8px;
            padding: 16px;
            margin-top: 20px;
        }
        
        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 16px;
        }
        
        .stat-item {
            background: var(--bg-tertiary);
            padding: 12px;
            border-radius: 6px;
        }
        
        .stat-label {
            font-size: 0.85rem;
            color: var(--text-secondary);
            margin-bottom: 4px;
        }
        
        .stat-value {
            font-size: 1.2rem;
            font-weight: 600;
            color: var(--primary-color);
        }
        
        /* ============================================
           プログレスバー
        ============================================ */
        .progress-container {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            height: 4px;
            background: var(--bg-secondary);
            z-index: 2000;
            opacity: 0;
            transition: opacity 0.3s;
        }
        
        .progress-container.active {
            opacity: 1;
        }
        
        .progress-bar {
            height: 100%;
            background: linear-gradient(90deg, var(--primary-color), var(--secondary-color));
            transition: width 0.3s ease;
            box-shadow: 0 0 10px rgba(102, 126, 234, 0.5);
        }
        
        /* ============================================
           エラーオーバーレイ
        ============================================ */
        .error-overlay {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0, 0, 0, 0.9);
            display: none;
            align-items: center;
            justify-content: center;
            z-index: 3000;
        }
        
        .error-overlay.active {
            display: flex;
        }
        
        .error-content {
            background: var(--bg-secondary);
            border: 2px solid var(--error-color);
            border-radius: 8px;
            padding: 24px;
            max-width: 600px;
            width: 90%;
            max-height: 80vh;
            overflow-y: auto;
        }
        
        .error-header {
            display: flex;
            align-items: center;
            margin-bottom: 16px;
            color: var(--error-color);
        }
        
        .error-icon {
            font-size: 24px;
            margin-right: 12px;
        }
        
        .error-title {
            font-size: 1.2rem;
            font-weight: 600;
        }
        
        .error-message {
            background: var(--code-bg);
            padding: 16px;
            border-radius: 4px;
            font-family: 'Fira Code', monospace;
            font-size: 0.9rem;
            line-height: 1.5;
            margin-bottom: 16px;
            white-space: pre-wrap;
            word-break: break-all;
        }
        
        .error-close {
            padding: 8px 16px;
            background: var(--error-color);
            border: none;
            border-radius: 4px;
            color: white;
            cursor: pointer;
            font-size: 0.9rem;
        }
        
        /* ============================================
           アニメーション
        ============================================ */
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
        }
        
        .loading {
            animation: pulse 1.5s infinite;
        }
        
        /* ============================================
           レスポンシブデザイン
        ============================================ */
        @media (max-width: 1024px) {
            .workspace {
                grid-template-columns: 1fr;
            }
        }
        
        @media (max-width: 768px) {
            :root {
                --sidebar-width: 280px;
            }
            
            .preset-buttons {
                grid-template-columns: 1fr;
            }
            
            .stats-grid {
                grid-template-columns: 1fr;
            }
        }
        
        /* ============================================
           コンパイラ情報パネル
        ============================================ */
        .compiler-info {
            background: var(--bg-tertiary);
            border: 1px solid var(--border-color);
            border-radius: 6px;
            padding: 12px;
            margin-bottom: 20px;
            font-size: 0.85rem;
        }
        
        .compiler-info-row {
            display: flex;
            justify-content: space-between;
            margin-bottom: 6px;
        }
        
        .compiler-info-row:last-child {
            margin-bottom: 0;
        }
        
        .compiler-info-label {
            color: var(--text-secondary);
        }
        
        .compiler-info-value {
            color: var(--text-primary);
            font-weight: 500;
        }
        
        .cdn-badge {
            display: inline-block;
            padding: 2px 6px;
            border-radius: 3px;
            font-size: 0.8rem;
            font-weight: 600;
        }
        
        .cdn-badge.jsdelivr {
            background: #e84d39;
            color: white;
        }
        
        .cdn-badge.unpkg {
            background: #2196f3;
            color: white;
        }
        
        /* ============================================
           ユーティリティクラス
        ============================================ */
        .hidden {
            display: none !important;
        }
        
        .text-center {
            text-align: center;
        }
        
        .mb-2 { margin-bottom: 8px; }
        .mb-3 { margin-bottom: 12px; }
        .mb-4 { margin-bottom: 16px; }
    </style>
</head>
<body>
    <!-- プログレスバー -->
    <div id="progress-container" class="progress-container">
        <div id="progress-bar" class="progress-bar" style="width: 0%"></div>
    </div>
    
    <!-- エラーオーバーレイ -->
    <div id="error-overlay" class="error-overlay">
        <div class="error-content">
            <div class="error-header">
                <span class="error-icon">⚠️</span>
                <h3 class="error-title">コンパイルエラー</h3>
            </div>
            <pre id="error-message" class="error-message"></pre>
            <button class="error-close" onclick="hideError()">閉じる</button>
        </div>
    </div>
    
    <!-- アプリケーションコンテナ -->
    <div class="app-container">
        <!-- サイドバー -->
        <aside id="sidebar" class="sidebar">
            <div class="sidebar-header">
                <h2 class="sidebar-title">⚡ WebAssembly</h2>
                <span style="cursor: pointer;" onclick="toggleSidebar()">✕</span>
            </div>
            
            <!-- タブナビゲーション -->
            <div class="tab-navigation">
                <button class="tab-button active" onclick="switchTab('app')">アプリ設定</button>
                <button class="tab-button" onclick="switchTab('wasm')">WASM設定</button>
            </div>
            
            <!-- アプリ設定タブ -->
            <div id="app-tab" class="tab-content active">
                <div class="option-section">
                    <h3 class="option-section-title">アプリケーション設定</h3>
                    <div class="option-group">
                        <div class="option-item">
                            <input type="checkbox" id="auto-compile" checked>
                            <label for="auto-compile">自動コンパイル</label>
                        </div>
                        <div class="option-item">
                            <input type="checkbox" id="show-stats" checked>
                            <label for="show-stats">統計情報を表示</label>
                        </div>
                    </div>
                </div>
                
                <!-- コンパイラ情報（デバッグモード時のみ表示） -->
                <div id="compiler-info" class="compiler-info hidden">
                    <div class="compiler-info-row">
                        <span class="compiler-info-label">バージョン:</span>
                        <span id="compiler-version" class="compiler-info-value">-</span>
                    </div>
                    <div class="compiler-info-row">
                        <span class="compiler-info-label">取得元:</span>
                        <span id="version-source" class="compiler-info-value">-</span>
                    </div>
                    <div class="compiler-info-row">
                        <span class="compiler-info-label">CDN:</span>
                        <span id="cdn-provider" class="compiler-info-value">-</span>
                    </div>
                    <div class="compiler-info-row">
                        <span class="compiler-info-label">読み込み時間:</span>
                        <span id="load-time" class="compiler-info-value">-</span>
                    </div>
                </div>
            </div>
            
            <!-- WebAssembly設定タブ -->
            <div id="wasm-tab" class="tab-content">
                <!-- プリセット -->
                <div class="option-section">
                    <h3 class="option-section-title">プリセット</h3>
                    <div class="preset-buttons">
                        <button class="preset-button" onclick="applyPreset('simple')">シンプル</button>
                        <button class="preset-button" onclick="applyPreset('debug')">デバッグ</button>
                        <button class="preset-button" onclick="applyPreset('release')">リリース</button>
                        <button class="preset-button" onclick="applyPreset('minimal')">最小サイズ</button>
                    </div>
                </div>
                
                <!-- 最適化オプション -->
                <div class="option-section">
                    <h3 class="option-section-title">最適化</h3>
                    <div class="option-group">
                        <div class="option-item">
                            <input type="checkbox" id="opt-optimize">
                            <label for="opt-optimize">最適化を有効化</label>
                        </div>
                        <div class="option-item">
                            <label for="opt-optimizeLevel">最適化レベル:</label>
                            <select id="opt-optimizeLevel">
                                <option value="">デフォルト</option>
                                <option value="0">0 (なし)</option>
                                <option value="1">1 (基本)</option>
                                <option value="2">2 (標準)</option>
                                <option value="3">3 (最大)</option>
                            </select>
                        </div>
                        <div class="option-item">
                            <label for="opt-shrinkLevel">縮小レベル:</label>
                            <select id="opt-shrinkLevel">
                                <option value="">デフォルト</option>
                                <option value="0">0 (なし)</option>
                                <option value="1">1 (標準)</option>
                                <option value="2">2 (最大)</option>
                            </select>
                        </div>
                    </div>
                </div>
                
                <!-- デバッグオプション -->
                <div class="option-section">
                    <h3 class="option-section-title">デバッグ</h3>
                    <div class="option-group">
                        <div class="option-item">
                            <input type="checkbox" id="opt-debug">
                            <label for="opt-debug">デバッグ情報</label>
                        </div>
                        <div class="option-item">
                            <input type="checkbox" id="opt-sourceMap">
                            <label for="opt-sourceMap">ソースマップ</label>
                        </div>
                        <div class="option-item">
                            <input type="checkbox" id="opt-measure">
                            <label for="opt-measure">パフォーマンス測定</label>
                        </div>
                        <div class="option-item">
                            <input type="checkbox" id="opt-validate">
                            <label for="opt-validate">バリデーション</label>
                        </div>
                    </div>
                </div>
                
                <!-- ランタイムオプション -->
                <div class="option-section">
                    <h3 class="option-section-title">ランタイム</h3>
                    <div class="option-group">
                        <div class="option-item">
                            <label for="opt-runtime">ランタイム:</label>
                            <select id="opt-runtime">
                                <option value="">デフォルト</option>
                                <option value="stub">Stub (最小)</option>
                                <option value="minimal" selected>Minimal (基本)</option>
                                <option value="incremental">Incremental (GC)</option>
                            </select>
                        </div>
                        <div class="option-item">
                            <input type="checkbox" id="opt-exportRuntime">
                            <label for="opt-exportRuntime">ランタイムエクスポート</label>
                        </div>
                        <div class="option-item">
                            <input type="checkbox" id="opt-noAssert">
                            <label for="opt-noAssert">アサート無効化</label>
                        </div>
                    </div>
                </div>
                
                <!-- 高度なオプション -->
                <div class="option-section">
                    <h3 class="option-section-title">高度な設定</h3>
                    <div class="option-group">
                        <div class="option-item">
                            <input type="checkbox" id="opt-importMemory">
                            <label for="opt-importMemory">メモリインポート</label>
                        </div>
                        <div class="option-item">
                            <input type="checkbox" id="opt-sharedMemory">
                            <label for="opt-sharedMemory">共有メモリ</label>
                        </div>
                        <div class="option-item">
                            <input type="checkbox" id="opt-exportTable">
                            <label for="opt-exportTable">テーブルエクスポート</label>
                        </div>
                        <div class="option-item">
                            <input type="checkbox" id="opt-explicitStart">
                            <label for="opt-explicitStart">明示的スタート</label>
                        </div>
                        <div class="option-item">
                            <input type="checkbox" id="opt-lowMemoryLimit">
                            <label for="opt-lowMemoryLimit">低メモリ制限</label>
                        </div>
                    </div>
                </div>
            </div>
        </aside>
        
        <!-- メインコンテンツ -->
        <main id="main-content" class="main-content sidebar-open">
            <!-- ヘッダー -->
            <header class="header">
                <div class="hamburger-menu" onclick="toggleSidebar()">
                    <div class="hamburger-line"></div>
                    <div class="hamburger-line"></div>
                    <div class="hamburger-line"></div>
                </div>
                
                <h1 class="header-title">AssemblyScript Compiler</h1>
                
                <div class="header-actions">
                    <button class="action-button secondary" onclick="resetApp()">
                        🔄 リセット
                    </button>
                    <button class="action-button" onclick="compile()">
                        ⚡ コンパイル
                    </button>
                </div>
            </header>
            
            <!-- コンテンツエリア -->
            <div class="content-area">
                <!-- ワークスペース -->
                <div class="workspace">
                    <!-- コードエディター -->
                    <div class="workspace-panel">
                        <div class="panel-header">
                            <span>📝 AssemblyScript コード</span>
                            <span id="compile-status"></span>
                        </div>
                        <div class="panel-content">
                            <textarea id="code-editor" spellcheck="false"></textarea>
                        </div>
                    </div>
                    
                    <!-- アプリケーションビューポート -->
                    <div class="workspace-panel">
                        <div class="panel-header">
                            <span>🚀 アプリケーション</span>
                        </div>
                        <div class="panel-content">
                            <div id="app-viewport">
                                <p class="loading">初期化中...</p>
                            </div>
                        </div>
                    </div>
                </div>
                
                <!-- 統計情報パネル -->
                <div id="stats-panel" class="stats-panel">
                    <div class="stats-grid">
                        <div class="stat-item">
                            <div class="stat-label">コンパイル時間</div>
                            <div class="stat-value" id="stat-compile-time">-</div>
                        </div>
                        <div class="stat-item">
                            <div class="stat-label">WASMサイズ</div>
                            <div class="stat-value" id="stat-wasm-size">-</div>
                        </div>
                        <div class="stat-item">
                            <div class="stat-label">実行回数</div>
                            <div class="stat-value" id="stat-exec-count">0</div>
                        </div>
                        <div class="stat-item">
                            <div class="stat-label">ステータス</div>
                            <div class="stat-value" id="stat-status">待機中</div>
                        </div>
                    </div>
                </div>
            </div>
        </main>
    </div>
    
    <script type="module">
        // ============================================
        // アプリケーションモジュール
        // ============================================
        const AppModule = {
            name: "Counter Demo",
            description: "シンプルなカウンターアプリケーション",
            
            sourceCode: `
// カウンターのグローバル変数
let counter: i32 = 0;

// カウンターを増やす
export function increment(): i32 {
    counter++;
    return counter;
}

// カウンターを減らす
export function decrement(): i32 {
    counter--;
    return counter;
}

// 現在の値を取得
export function getValue(): i32 {
    return counter;
}

// カウンターをリセット
export function reset(): void {
    counter = 0;
}

// メモリサイズを取得（デバッグ用）
export function getMemorySize(): i32 {
    return memory.size();
}
            `.trim(),
            
            defaultOptions: {
                optimize: false,
                runtime: 'minimal'
            },
            
            async init(wasmExports) {
                const viewport = document.getElementById('app-viewport');
                viewport.innerHTML = `
                    <div style="text-align: center;">
                        <h2 style="margin-bottom: 20px;">カウンターデモ</h2>
                        <div style="font-size: 48px; font-weight: bold; margin: 20px 0;" id="counter-value">0</div>
                        <div style="display: flex; gap: 10px; justify-content: center;">
                            <button onclick="window.counterOp('increment')" style="padding: 10px 20px; font-size: 18px; cursor: pointer;">➕ 増加</button>
                            <button onclick="window.counterOp('decrement')" style="padding: 10px 20px; font-size: 18px; cursor: pointer;">➖ 減少</button>
                            <button onclick="window.counterOp('reset')" style="padding: 10px 20px; font-size: 18px; cursor: pointer;">🔄 リセット</button>
                        </div>
                        <div style="margin-top: 20px; font-size: 14px; color: #666;">
                            メモリサイズ: <span id="memory-size">0</span> ページ
                        </div>
                    </div>
                `;
                
                // グローバル関数として登録
                window.counterOp = (op) => {
                    let value;
                    switch(op) {
                        case 'increment':
                            value = wasmExports.increment();
                            break;
                        case 'decrement':
                            value = wasmExports.decrement();
                            break;
                        case 'reset':
                            wasmExports.reset();
                            value = 0;
                            break;
                    }
                    document.getElementById('counter-value').textContent = value;
                    document.getElementById('memory-size').textContent = wasmExports.getMemorySize();
                    
                    // 実行回数を更新
                    const execCount = parseInt(document.getElementById('stat-exec-count').textContent) + 1;
                    document.getElementById('stat-exec-count').textContent = execCount;
                };
                
                // 初期メモリサイズを表示
                document.getElementById('memory-size').textContent = wasmExports.getMemorySize();
            }
        };
        
        // ============================================
        // グローバル変数
        // ============================================
        let wasmExports = null;
        let compilerReady = false;
        
        // ============================================
        // バージョン管理とメタデータ
        // ============================================
        const COMPILER_METADATA = {
            version: null,
            versionSource: null,
            fetchedFrom: "NPM Registry via CDN",
            generatedAt: new Date().toISOString(),
            cdnProvider: null,
            loadTime: null,
            attempts: 0,
            
            async initialize() {
                const startTime = performance.now();
                try {
                    this.version = await VersionManager.getLatestVersion();
                    this.versionSource = 'Dynamic fetch';
                    console.log(`[Version] Fetched: v${this.version}`);
                } catch (error) {
                    this.version = "0.28.8";
                    this.versionSource = 'Hardcoded fallback';
                    console.warn('[Version] Using fallback version');
                }
                const fetchTime = Math.round(performance.now() - startTime);
                console.log(`[Version] Time: ${fetchTime}ms`);
            }
        };
        
        // ============================================
        // CDNプロバイダー定義
        // ============================================
        const CDN_PROVIDERS = [
            {
                name: 'jsDelivr',
                baseUrl: 'https://cdn.jsdelivr.net/npm',
                timeout: 10000,
                priority: 1
            },
            {
                name: 'UNPKG',
                baseUrl: 'https://unpkg.com',
                timeout: 10000,
                priority: 2
            }
        ];
        
        // ============================================
        // バージョン管理システム
        // ============================================
        const VersionManager = {
            cache: {
                get() {
                    try {
                        const cached = sessionStorage.getItem('asc_version_cache');
                        if (cached) {
                            const data = JSON.parse(cached);
                            if (Date.now() - data.timestamp < 3600000) {
                                return data.version;
                            }
                        }
                    } catch (e) {}
                    return null;
                },
                set(version) {
                    try {
                        sessionStorage.setItem('asc_version_cache', JSON.stringify({
                            version,
                            timestamp: Date.now()
                        }));
                    } catch (e) {}
                }
            },
            
            async getLatestVersion() {
                const cached = this.cache.get();
                if (cached) {
                    console.log(`[Version] Using cached: ${cached}`);
                    return cached;
                }
                
                const strategies = [
                    { name: 'jsDelivr API', fn: () => this.fetchFromJsDelivr() },
                    { name: 'jsDelivr Package', fn: () => this.fetchJsDelivrPackageJson() },
                    { name: 'Unpkg', fn: () => this.fetchFromUnpkg() }
                ];
                
                for (const strategy of strategies) {
                    try {
                        console.log(`[Version] Trying ${strategy.name}...`);
                        const version = await Promise.race([
                            strategy.fn(),
                            this.timeout(3000)
                        ]);
                        
                        if (version && this.isValidStableVersion(version)) {
                            console.log(`[Version] Success: ${version}`);
                            this.cache.set(version);
                            return version;
                        }
                    } catch (e) {
                        console.warn(`[Version] ${strategy.name} failed`);
                    }
                }
                
                return "0.28.8";
            },
            
            async fetchFromJsDelivr() {
                const response = await fetch('https://data.jsdelivr.com/v1/package/npm/assemblyscript');
                const data = await response.json();
                const stable = data.versions.filter(v => this.isValidStableVersion(v));
                return stable[0];
            },
            
            async fetchJsDelivrPackageJson() {
                const response = await fetch('https://cdn.jsdelivr.net/npm/assemblyscript@latest/package.json');
                const data = await response.json();
                return data.version;
            },
            
            async fetchFromUnpkg() {
                const response = await fetch('https://unpkg.com/assemblyscript@latest/package.json');
                const data = await response.json();
                return data.version;
            },
            
            isValidStableVersion(version) {
                const semverRegex = /^(\d+)\.(\d+)\.(\d+)$/;
                if (!version.match(semverRegex)) return false;
                
                const preRelease = ['alpha', 'beta', 'rc', 'dev', '-'];
                return !preRelease.some(indicator => 
                    version.toLowerCase().includes(indicator)
                );
            },
            
            timeout(ms) {
                return new Promise((_, reject) => 
                    setTimeout(() => reject(new Error('Timeout')), ms)
                );
            }
        };
        
        // ============================================
        // コンパイラローダー
        // ============================================
        const CompilerLoader = {
            compiler: null,
            
            async load() {
                await COMPILER_METADATA.initialize();
                return await loadCompilerWithCDNFallback();
            }
        };
        
        // CDNフォールバック対応ローダー
        async function loadCompilerWithCDNFallback() {
            const version = COMPILER_METADATA.version;
            
            for (let i = 0; i < CDN_PROVIDERS.length; i++) {
                const cdn = CDN_PROVIDERS[i];
                const startTime = performance.now();
                
                try {
                    updateProgress(30 + (i * 35), `コンパイラ読み込み中 (${cdn.name})...`);
                    
                    const webJsUrl = `${cdn.baseUrl}/assemblyscript@${version}/dist/web.js`;
                    console.log(`[Loader] Attempting ${cdn.name}: ${webJsUrl}`);
                    
                    await loadScriptTag(webJsUrl);
                    await waitForImportMap();
                    
                    const asc = await import("assemblyscript/asc");
                    const compiler = asc.default || asc;
                    
                    await validateCompiler(compiler);
                    
                    const loadTime = Math.round(performance.now() - startTime);
                    COMPILER_METADATA.cdnProvider = cdn.name;
                    COMPILER_METADATA.loadTime = loadTime;
                    COMPILER_METADATA.attempts = i + 1;
                    
                    console.log(`[Loader] Success with ${cdn.name} in ${loadTime}ms`);
                    updateProgress(100, 'コンパイラ準備完了');
                    
                    return compiler;
                    
                } catch (error) {
                    console.error(`[Loader] ${cdn.name} failed:`, error.message);
                    cleanupFailedScripts();
                }
            }
            
            throw new Error(`All CDN providers failed for AssemblyScript ${version}`);
        }
        
        function loadScriptTag(url) {
            return new Promise((resolve, reject) => {
                if (window.ASSEMBLYSCRIPT_VERSION) {
                    console.log('[Loader] AssemblyScript already loaded');
                    resolve();
                    return;
                }
                
                const script = document.createElement('script');
                script.src = url;
                script.dataset.assemblyScript = 'loading';
                
                const timeout = setTimeout(() => {
                    script.remove();
                    reject(new Error(`Timeout loading: ${url}`));
                }, 10000);
                
                script.onload = () => {
                    clearTimeout(timeout);
                    script.dataset.assemblyScript = 'loaded';
                    resolve();
                };
                
                script.onerror = () => {
                    clearTimeout(timeout);
                    script.remove();
                    reject(new Error(`Failed to load: ${url}`));
                };
                
                document.head.appendChild(script);
            });
        }
        
        async function waitForImportMap() {
            const maxAttempts = 20;
            const waitTime = 100;
            
            for (let i = 0; i < maxAttempts; i++) {
                if (window.ASSEMBLYSCRIPT_VERSION && window.ASSEMBLYSCRIPT_IMPORTMAP) {
                    console.log('[Loader] Import map ready');
                    return;
                }
                await new Promise(resolve => setTimeout(resolve, waitTime));
            }
            
            throw new Error('Import map setup timeout');
        }
        
        async function validateCompiler(compiler) {
            if (!compiler || typeof compiler.main !== 'function') {
                throw new Error('Invalid compiler structure');
            }
        }
        
        function cleanupFailedScripts() {
            const scripts = document.querySelectorAll('script[data-assembly-script]');
            scripts.forEach(script => {
                if (script.dataset.assemblyScript !== 'loaded') {
                    script.remove();
                }
            });
            
            delete window.ASSEMBLYSCRIPT_VERSION;
            delete window.ASSEMBLYSCRIPT_IMPORTMAP;
        }
        
        // ============================================
        // コンパイラコア
        // ============================================
        const CompilerCore = {
            compiler: null,
            
            init(ascInstance) {
                this.compiler = ascInstance;
                return true;
            },
            
            async compile(sourceCode, options) {
                if (!this.compiler) {
                    throw new Error('Compiler not initialized');
                }
                
                const compileOptions = this.buildOptions(options);
                let wasmBinary = null;
                let compileStats = '';
                
                const { error, stdout, stderr } = await this.compiler.main(compileOptions, {
                    readFile: (filename) => {
                        if (filename === "main.ts") return sourceCode;
                        return null;
                    },
                    writeFile: (filename, contents) => {
                        if (filename === "main.wasm") {
                            wasmBinary = contents;
                        }
                    },
                    listFiles: () => ["main.ts"]
                });
                
                if (error) throw new Error(stderr || error.toString());
                if (!wasmBinary) throw new Error('No WebAssembly binary generated');
                
                if (stdout) compileStats = stdout;
                
                return { wasmBinary, stats: compileStats };
            },
            
            buildOptions(userOptions) {
                const flags = ["main.ts", "-o", "main.wasm"];
                
                Object.entries(userOptions).forEach(([key, value]) => {
                    if (typeof value === 'boolean' && value) {
                        flags.push(`--${key}`);
                    } else if (typeof value !== 'boolean' && value !== '') {
                        flags.push(`--${key}`, value.toString());
                    }
                });
                
                return flags;
            }
        };
        
        // ============================================
        // オプション管理システム
        // ============================================
        const OptionsManager = {
            current: {},
            
            init() {
                this.current = { runtime: 'minimal' };
                this.loadFromStorage();
                this.updateUI();
            },
            
            apply(preset) {
                if (PRESETS[preset]) {
                    this.current = { ...PRESETS[preset] };
                    this.updateUI();
                    this.saveToStorage();
                }
            },
            
            collectFromUI() {
                const options = {};
                
                ['optimize', 'exportRuntime', 'noAssert', 'debug', 'sourceMap',
                 'measure', 'validate', 'importMemory', 'sharedMemory',
                 'exportTable', 'explicitStart', 'lowMemoryLimit'].forEach(opt => {
                    const el = document.getElementById(`opt-${opt}`);
                    if (el && el.checked) options[opt] = true;
                });
                
                ['optimizeLevel', 'shrinkLevel', 'runtime'].forEach(opt => {
                    const el = document.getElementById(`opt-${opt}`);
                    if (el && el.value) options[opt] = el.value;
                });
                
                this.current = options;
                this.saveToStorage();
                return options;
            },
            
            updateUI() {
                Object.keys(this.current).forEach(key => {
                    const el = document.getElementById(`opt-${key}`);
                    if (el) {
                        if (el.type === 'checkbox') {
                            el.checked = this.current[key] === true;
                        } else {
                            el.value = this.current[key] || '';
                        }
                    }
                });
            },
            
            saveToStorage() {
                try {
                    localStorage.setItem('asmOptions', JSON.stringify(this.current));
                } catch(e) {}
            },
            
            loadFromStorage() {
                try {
                    const saved = localStorage.getItem('asmOptions');
                    if (saved) this.current = JSON.parse(saved);
                } catch(e) {}
            }
        };
        
        // ============================================
        // WebAssembly実行管理
        // ============================================
        const WasmRunner = {
            instance: null,
            
            async instantiate(wasmBinary, imports = {}) {
                const defaultImports = {
                    env: {
                        abort: (msg, file, line, column) => {
                            console.error(`Abort: ${msg} at ${file}:${line}:${column}`);
                        },
                        trace: (msg, n, ...args) => {
                            console.log(`Trace: ${msg}`, ...args);
                        },
                        seed: () => Date.now(),
                        memory: new WebAssembly.Memory({ initial: 1 })
                    },
                    Math: Math,
                    Date: Date
                };
                
                const mergedImports = this.mergeImports(defaultImports, imports);
                
                let buffer;
                if (wasmBinary instanceof ArrayBuffer) {
                    buffer = wasmBinary;
                } else if (wasmBinary instanceof Uint8Array) {
                    buffer = wasmBinary.buffer.slice(wasmBinary.byteOffset, 
                                                     wasmBinary.byteOffset + wasmBinary.byteLength);
                } else {
                    throw new Error('Unexpected binary format');
                }
                
                const wasmModule = await WebAssembly.compile(buffer);
                this.instance = await WebAssembly.instantiate(wasmModule, mergedImports);
                
                return this.instance.exports;
            },
            
            mergeImports(defaults, custom) {
                const merged = { ...defaults };
                Object.keys(custom).forEach(key => {
                    if (typeof custom[key] === 'object' && !Array.isArray(custom[key])) {
                        merged[key] = { ...defaults[key], ...custom[key] };
                    } else {
                        merged[key] = custom[key];
                    }
                });
                return merged;
            }
        };
        
        // ============================================
        // プリセット定義
        // ============================================
        const PRESETS = {
            simple: {
                optimize: false,
                runtime: 'minimal'
            },
            debug: {
                optimize: false,
                optimizeLevel: '0',
                debug: true,
                sourceMap: true,
                measure: true
            },
            release: {
                optimize: true,
                optimizeLevel: '3',
                shrinkLevel: '1',
                noAssert: true
            },
            minimal: {
                optimize: true,
                optimizeLevel: '3',
                shrinkLevel: '2',
                runtime: 'stub',
                noAssert: true
            }
        };
        
        // ============================================
        // UI関数
        // ============================================
        function updateProgress(percent, message = '') {
            const container = document.getElementById('progress-container');
            const bar = document.getElementById('progress-bar');
            
            if (percent > 0 && percent < 100) {
                container.classList.add('active');
                bar.style.width = `${percent}%`;
                
                if (message) {
                    document.getElementById('stat-status').textContent = message;
                    console.log(`[Progress] ${message} (${percent}%)`);
                }
            } else if (percent >= 100) {
                bar.style.width = '100%';
                setTimeout(() => {
                    container.classList.remove('active');
                }, 500);
                
                if (message) {
                    document.getElementById('stat-status').textContent = message;
                }
            }
        }
        
        function showError(error) {
            const overlay = document.getElementById('error-overlay');
            const message = document.getElementById('error-message');
            
            message.textContent = error.message || error.toString();
            overlay.classList.add('active');
            
            console.error('[Error]', error);
        }
        
        window.hideError = function() {
            document.getElementById('error-overlay').classList.remove('active');
        };
        
        window.toggleSidebar = function() {
            const sidebar = document.getElementById('sidebar');
            const mainContent = document.getElementById('main-content');
            
            sidebar.classList.toggle('closed');
            mainContent.classList.toggle('sidebar-open');
        };
        
        window.switchTab = function(tabName) {
            document.querySelectorAll('.tab-button').forEach(btn => {
                btn.classList.remove('active');
            });
            document.querySelectorAll('.tab-content').forEach(content => {
                content.classList.remove('active');
            });
            
            event.target.classList.add('active');
            document.getElementById(`${tabName}-tab`).classList.add('active');
        };
        
        window.applyPreset = function(preset) {
            OptionsManager.apply(preset);
            console.log(`[Options] Applied preset: ${preset}`);
        };
        
        window.resetApp = function() {
            document.getElementById('code-editor').value = AppModule.sourceCode;
            OptionsManager.init();
            compile();
        };
        
        function updateStats(compileTime, wasmSize) {
            document.getElementById('stat-compile-time').textContent = `${compileTime}ms`;
            document.getElementById('stat-wasm-size').textContent = formatBytes(wasmSize);
        }
        
        function formatBytes(bytes) {
            if (bytes < 1024) return bytes + ' B';
            if (bytes < 1048576) return (bytes / 1024).toFixed(2) + ' KB';
            return (bytes / 1048576).toFixed(2) + ' MB';
        }
        
        function updateCompilerInfo() {
            if (COMPILER_METADATA.version) {
                document.getElementById('compiler-version').textContent = `v${COMPILER_METADATA.version}`;
                document.getElementById('version-source').textContent = COMPILER_METADATA.versionSource;
                
                if (COMPILER_METADATA.cdnProvider) {
                    const cdnEl = document.getElementById('cdn-provider');
                    cdnEl.innerHTML = `<span class="cdn-badge ${COMPILER_METADATA.cdnProvider.toLowerCase()}">${COMPILER_METADATA.cdnProvider}</span>`;
                }
                
                if (COMPILER_METADATA.loadTime) {
                    document.getElementById('load-time').textContent = `${COMPILER_METADATA.loadTime}ms`;
                }
            }
        }
        
        // ============================================
        // コンパイル関数
        // ============================================
        window.compile = async function() {
            if (!compilerReady) {
                console.warn('[Compile] Compiler not ready yet');
                return;
            }
            
            const sourceCode = document.getElementById('code-editor').value;
            const options = OptionsManager.collectFromUI();
            
            try {
                updateProgress(10, 'コンパイル開始...');
                const startTime = performance.now();
                
                const { wasmBinary, stats } = await CompilerCore.compile(sourceCode, options);
                
                updateProgress(50, 'WebAssembly生成完了...');
                
                wasmExports = await WasmRunner.instantiate(wasmBinary, AppModule.getImports ? AppModule.getImports() : {});
                
                updateProgress(80, '初期化中...');
                
                if (AppModule.init) {
                    await AppModule.init(wasmExports);
                }
                
                if (AppModule.execute) {
                    AppModule.execute(wasmExports);
                }
                
                const compileTime = Math.round(performance.now() - startTime);
                const wasmSize = wasmBinary.byteLength || wasmBinary.length;
                
                updateStats(compileTime, wasmSize);
                updateProgress(100, '実行中');
                
                console.log('[Compile] Success:', stats);
                
            } catch (error) {
                updateProgress(0);
                showError(error);
                document.getElementById('stat-status').textContent = 'エラー';
            }
        };
        
        // ============================================
        // 初期化
        // ============================================
        async function initialize() {
            console.log('[Init] Starting initialization...');
            updateProgress(5, '初期化開始...');
            
            try {
                // デバッグモード確認
                const isDebug = localStorage.getItem('debug') === 'true';
                if (isDebug) {
                    document.getElementById('compiler-info').classList.remove('hidden');
                }
                
                // コンパイラ読み込み
                updateProgress(10, 'コンパイラ読み込み準備...');
                const compiler = await CompilerLoader.load();
                
                CompilerCore.init(compiler);
                compilerReady = true;
                
                // コンパイラ情報更新
                updateCompilerInfo();
                
                // オプション初期化
                OptionsManager.init();
                
                // エディター初期設定
                const editor = document.getElementById('code-editor');
                editor.value = AppModule.sourceCode;
                
                // AppModule デフォルトオプション適用
                if (AppModule.defaultOptions) {
                    Object.entries(AppModule.defaultOptions).forEach(([key, value]) => {
                        const el = document.getElementById(`opt-${key}`);
                        if (el) {
                            if (el.type === 'checkbox') {
                                el.checked = value === true;
                            } else {
                                el.value = value;
                            }
                        }
                    });
                }
                
                // 統計表示設定の確認
                const showStats = document.getElementById('show-stats');
                const statsPanel = document.getElementById('stats-panel');
                if (!showStats.checked) {
                    statsPanel.style.display = 'none';
                }
                
                showStats.addEventListener('change', (e) => {
                    statsPanel.style.display = e.target.checked ? 'block' : 'none';
                });
                
                // 自動コンパイル
                const autoCompile = document.getElementById('auto-compile');
                if (autoCompile.checked) {
                    await compile();
                }
                
                console.log('[Init] Initialization complete');
                
            } catch (error) {
                updateProgress(0);
                showError(error);
                console.error('[Init] Initialization failed:', error);
            }
        }
        
        // ============================================
        // アプリケーション起動
        // ============================================
        window.addEventListener('DOMContentLoaded', () => {
            console.log('[App] DOM Content Loaded');
            console.log('[App] Module:', AppModule.name);
            initialize();
        });
    </script>
</body>
</html>
```

## **デフォルトデモについて**

このHTMLテンプレートには、シンプルなカウンターアプリがデフォルトデモとして含まれています。

### **カウンターアプリの機能**
- ➕ **増加**: カウンターを1増やす
- ➖ **減少**: カウンターを1減らす  
- 🔄 **リセット**: カウンターを0にリセット
- メモリサイズの表示（WebAssemblyページ数）

### **AppModuleの置き換え**

ユースケースに応じて、`AppModule`オブジェクトを別の実装に置き換えることで、様々なアプリケーションを作成できます。

---

**ファイルサイズ**: 約65KB（HTML形式）  
**バージョン**: 1.4.0  
**最終更新**: 2025年9月
