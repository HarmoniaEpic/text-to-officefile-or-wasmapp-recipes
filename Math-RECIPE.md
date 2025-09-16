# **Math-RECIPE v2.0 (エラー対応強化版)**
**数式PNG自動生成ページ レシピ**

---

## **📌 このレシピについて**

生成AIがユーザーの数式指示を解釈し、高品質な**数式PNGファイル**をブラウザ上で生成・ダウンロードできる、**単一HTMLファイル**を自動作成するための指示書です。

### **v2.0の変更点 (重要)**
- **エラーオーバーレイ機能の搭載**: `PPTX-RECIPE`から高機能なエラー報告機能を移植しました。エラー発生時に詳細な情報が表示され、ワンクリックで内容をコピーしてAIに修正を依頼できます。
- 🐛 **安定性の向上**: 以前のバージョンでエラーの原因となっていた、不要なフォントキャッシュ再構築処理を削除済みです。

### **v1.9からの主な機能**
- 🚀 **最新の実行環境**: Pyodide `v0.28.2` を使用し、高速な初期化とパフォーマンスを実現。
- 🖋️ **フォント選択**: 'Computer Modern', 'STIX', 'DejaVu Sans'など複数のフォントから選択可能。
- 🎨 **フルカスタマイズ**: 文字色、背景色、透明度などを自由に設定。

---

## **🎯 AI実装の手順**

### **ステップ1: ユーザー入力の解釈**

AIは以下の手順で数式を処理してください：

1.  **入力形式の判定**
    ```
    - TeX形式（\や$を含む）→ そのまま使用
    - 自然言語 → TeX形式に変換
    - その他の数式記法 → TeX形式に変換
    ```

2.  **TeX形式への変換例**
    ```
    入力: "xの2乗プラスyの2乗イコール1"
    変換: x^2 + y^2 = 1
    ```

3.  **変換結果の埋め込み**
    ```html
    <!-- HTMLのこの部分に変換結果を挿入 -->
    <textarea id="formula-input">ここにTeX形式の数式</textarea>
    ```

### **ステップ2: HTMLファイルの生成**

変換したTeX形式をHTMLテンプレートの指定箇所に埋め込んでください。

---

## **🚨 実装ルール (v2.0)**

### **🔴 絶対禁止事項（変更・削除厳禁）**

1.  **Matplotlibのロード**
    ```javascript
    await pyodide.loadPackage("matplotlib");
    ```

2.  **Pyodideのロード**
    ```html
    <script src="https://cdn.jsdelivr.net/pyodide/v0.28.2/full/pyodide.js"></script>
    ```
3.  **エラーオーバーレイのHTML/CSS/JS**
    - エラー表示用のHTML構造 (`<div id="error-overlay">`)
    - エラー表示用のCSSクラス (`.error-overlay`, `.error-card`等)
    - `showError`関数を含むエラーハンドリング用のJavaScriptコード

### **🟡 必須ルール**

1.  **Pythonコードの明示的実行**
    `main`関数内で、Pyodideの初期化とライブラリ読み込みが完了した**後**に、`<script>`タグ内のPythonコード全体を読み込んで実行してください。
    ```javascript
    const pythonCode = document.getElementById('python-code').textContent;
    await pyodide.runPythonAsync(pythonCode);
    await runGeneration(true); // 初回実行
    ```

---

## **📝 HTMLテンプレート (v2.0 エラー対応強化版)**

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <title>数式PNGジェネレーター v2.0</title>

    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; }
        button:disabled { cursor: not-allowed; opacity: 0.6; }
        .spinner { border: 4px solid rgba(0, 0, 0, 0.1); width: 36px; height: 36px; border-radius: 50%; border-left-color: #3b82f6; animation: spin 1s ease infinite; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        .preview-area { min-height: 240px; background: linear-gradient(to bottom, #ffffff, #fafafa); border: 2px solid #e5e7eb; border-radius: 16px; padding: 24px; display: flex; align-items: center; justify-content: center; margin-bottom: 24px; box-shadow: inset 0 2px 4px rgba(0,0,0,0.06); overflow: auto; position: relative; }
        .preview-area.transparent-bg { background: repeating-conic-gradient(#f3f4f6 0% 25%, transparent 0% 50%) 50% / 20px 20px; }
        .settings-section { background: #f9fafb; border: 1px solid #e5e7eb; border-radius: 12px; padding: 16px; margin-top: 20px; }
        .settings-header { display: flex; align-items: center; justify-content: space-between; cursor: pointer; user-select: none; }
        
        /* エラーオーバーレイ (PPTX-RECIPEより移植) */
        .error-overlay { position: fixed; inset: 0; background: rgba(17,24,39,0.25); display: none; align-items: center; justify-content: center; padding: 16px; z-index: 9999; }
        .error-card { width: 100%; max-width: 880px; background: rgba(255,255,255,0.96); border: 1px solid rgba(55,65,81,0.25); border-radius: 16px; box-shadow: 0 10px 30px rgba(0,0,0,0.25); padding: 20px; }
        .error-title { display:flex; align-items:center; gap:8px; font-weight:800; font-size:18px; color:#991b1b; }
        .error-subtitle { margin-top:6px; color:#374151; line-height:1.55; }
        .error-actions { margin-top:12px; display:flex; gap:8px; flex-wrap:wrap; }
        .copy-btn { border:1px solid rgba(55,65,81,.3); padding:8px 12px; border-radius:10px; font-weight:600; background:#fff; cursor: pointer; }
        .copy-btn:hover { background: #f3f4f6; }
        .error-pre { margin-top:12px; background:#0b1020; color:#e5e7eb; border-radius:12px; padding:12px 14px; max-height:320px; overflow:auto; white-space:pre-wrap; word-break:break-word; font-family:ui-monospace,SFMono-Regular,Menlo,Consolas,'Courier New',monospace; font-size:13px; line-height:1.45; }
    </style>
</head>
<body class="bg-gradient-to-br from-blue-50 via-indigo-50 to-purple-50 min-h-screen p-4">

    <div class="max-w-4xl mx-auto py-8">
        <div class="bg-white rounded-2xl shadow-xl p-8">
            <div class="text-center mb-8">
                <h1 class="text-3xl font-bold text-gray-800 mb-2">数式PNGジェネレーター</h1>
                <p class="text-gray-600">美しい数式を自由な色彩とフォントでPNG形式に変換</p>
                <p class="text-xs text-gray-500 mt-2">v2.0 - Pyodide v0.28.2</p>
            </div>

            <div class="mb-6">
                <label class="block mb-2 text-sm font-semibold text-gray-700" for="formula-input">TeX形式の数式</label>
                <textarea id="formula-input" class="w-full p-3 border-2 border-gray-200 rounded-lg font-mono text-sm focus:border-blue-500 focus:ring-1 focus:ring-blue-500 transition" rows="3" placeholder="例: \int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}"></textarea>
            </div>

            <div id="status-container" class="mb-6 h-12 flex items-center justify-center">
                <p id="status-message" class="text-gray-600">準備しています...</p>
            </div>

            <div class="preview-area" id="preview-area">
                <div class="preview-placeholder">
                    <div id="loading-spinner" class="spinner mx-auto mb-4"></div>
                    <p>数式を生成中...</p>
                </div>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                <button id="regenerate-button" class="w-full bg-indigo-600 text-white font-bold py-3 px-4 rounded-lg hover:bg-indigo-700 transition" disabled>⟳ 再生成</button>
                <button id="download-button" class="w-full bg-blue-600 text-white font-bold py-3 px-4 rounded-lg hover:bg-blue-700 transition" disabled>⬇ PNGをダウンロード</button>
            </div>
            
            <details class="settings-section">
                <summary class="settings-header"><span class="font-semibold text-gray-800">詳細設定</span></summary>
                <div class="mt-4 space-y-6">
                    <div>
                        <h3 class="font-semibold text-gray-700 mb-2 border-b pb-2">🎨 カラー設定</h3>
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-2">
                            <div>
                                <label class="block text-sm font-medium text-gray-600">文字色</label>
                                <div class="flex items-center gap-2 mt-1">
                                    <input type="color" id="text-color-picker" value="#000000" class="h-8 w-8 p-0 border-0 rounded cursor-pointer">
                                    <input type="text" id="text-color" value="#000000" class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm font-mono">
                                </div>
                            </div>
                            <div>
                                <label class="block text-sm font-medium text-gray-600">背景色</label>
                                <div class="flex items-center gap-2 mt-1">
                                    <input type="color" id="bg-color-picker" value="#ffffff" class="h-8 w-8 p-0 border-0 rounded cursor-pointer">
                                    <input type="text" id="bg-color" value="#ffffff" class="w-full px-2 py-1 border border-gray-300 rounded-md text-sm font-mono">
                                </div>
                            </div>
                        </div>
                         <div class="flex items-center gap-2 mt-4">
                            <input type="checkbox" id="transparent-bg" class="h-4 w-4 rounded border-gray-300 text-blue-600 focus:ring-blue-500">
                            <label for="transparent-bg" class="text-sm text-gray-700">背景を透明にする</label>
                        </div>
                    </div>
                     <div>
                        <h3 class="font-semibold text-gray-700 mb-2 border-b pb-2">⚙️ 基本設定</h3>
                        <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mt-2">
                            <div>
                                <label for="font-size" class="block text-sm font-medium text-gray-600">フォントサイズ</label>
                                <input type="number" id="font-size" value="24" class="w-full mt-1 px-2 py-1 border border-gray-300 rounded-md text-sm">
                            </div>
                            <div>
                                <label for="dpi" class="block text-sm font-medium text-gray-600">解像度 (DPI)</label>
                                <input type="number" id="dpi" value="200" class="w-full mt-1 px-2 py-1 border border-gray-300 rounded-md text-sm">
                            </div>
                             <div class="col-span-2">
                                <label for="font-type" class="block text-sm font-medium text-gray-600">フォントタイプ</label>
                                <select id="font-type" class="w-full mt-1 px-2 py-1 border border-gray-300 rounded-md text-sm">
                                    <option value="cm" selected>Computer Modern (セリフ)</option>
                                    <option value="stix">STIX (セリフ)</option>
                                    <option value="stixsans">STIX Sans (サンセリフ)</option>
                                    <option value="dejavusans">DejaVu Sans (サンセリフ)</option>
                                    <option value="dejavuserif">DejaVu Serif (セリフ)</option>
                                </select>
                            </div>
                        </div>
                    </div>
                </div>
            </details>
        </div>
    </div>

    <!-- エラーオーバーレイ (ここから移植) -->
    <div id="error-overlay" class="error-overlay" role="dialog" aria-modal="true" aria-labelledby="error-title">
        <div class="error-card">
            <div class="error-title" id="error-title">エラーが発生しました：AIのチャットエリアに以下のエラーメッセージを貼り付けて修正を依頼して下さい</div>
            <div class="error-subtitle">エラーの全文をコピーして、そのままAIに投げてください。原因の特定とパッチを提案します。</div>
            <div class="error-actions">
                <button id="copy-error" class="copy-btn" aria-label="エラーメッセージをコピーする">エラー全文をコピー</button>
                <button id="close-error" class="copy-btn" aria-label="このエラー表示を閉じる">閉じる</button>
            </div>
            <pre id="error-text" class="error-pre" tabindex="0" aria-live="polite"></pre>
        </div>
    </div>
    <!-- (ここまで移植) -->

    <script type="text/python" id="python-code">
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
from io import BytesIO
import base64
import js

def hex_to_rgb(hex_color):
    hex_color = hex_color.lstrip('#')
    if len(hex_color) == 3: hex_color = ''.join([c*2 for c in hex_color])
    try: return tuple(int(hex_color[i:i+2], 16) / 255.0 for i in (0, 2, 4))
    except (ValueError, TypeError): return (0, 0, 0)

def create_math_png(formula, options):
    fontsize = options.get('fontsize', 24)
    dpi = options.get('dpi', 200)
    fontset = options.get('fontset', 'cm')
    text_color = options.get('text_color', '#000000')
    bg_color = options.get('bg_color', '#ffffff')
    transparent = options.get('transparent', False)

    if not formula.strip().startswith('$'):
        formula = '$' + formula + '$'
    
    plt.rcParams['mathtext.fontset'] = fontset
    plt.rcParams['font.size'] = fontsize
    
    fig_facecolor = 'none' if transparent else hex_to_rgb(bg_color)
    fig = plt.figure(figsize=(10, 2), facecolor=fig_facecolor)
    ax = fig.add_subplot(111)
    ax.set_facecolor(fig_facecolor)
    ax.axis('off')

    try:
        ax.text(0.5, 0.5, formula, ha='center', va='center', transform=ax.transAxes,
                fontsize=fontsize, color=hex_to_rgb(text_color))
    except Exception as e:
        ax.text(0.5, 0.5, f"Error: {str(e)}", ha='center', va='center', transform=ax.transAxes,
                fontsize=12, color='red')
    
    png_io = BytesIO()
    plt.savefig(png_io, format='png', dpi=dpi, bbox_inches='tight', pad_inches=0.2,
                transparent=transparent, facecolor=fig.get_facecolor())
    plt.close(fig)
    
    png_io.seek(0)
    return png_io.read()

def generate_formula():
    formula_input = js.document.getElementById('formula-input')
    formula = formula_input.value if formula_input.value else r"\int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}"
    
    options = {
        'fontsize': int(js.document.getElementById('font-size').value),
        'dpi': int(js.document.getElementById('dpi').value),
        'fontset': js.document.getElementById('font-type').value,
        'text_color': js.document.getElementById('text-color').value,
        'bg_color': js.document.getElementById('bg-color').value,
        'transparent': js.document.getElementById('transparent-bg').checked
    }
    
    png_bytes = create_math_png(formula, options)
    png_base64 = base64.b64encode(png_bytes).decode('utf-8')
    
    js.png_base64_data = png_base64
    js.png_preview_src = f"data:image/png;base64,{png_base64}"
    
    return True
    </script>
    
    <script src="https://cdn.jsdelivr.net/pyodide/v0.28.2/full/pyodide.js"></script>
    <script type="module">
        let pyodideInstance = null;
        
        const statusMessage = document.getElementById('status-message');
        const downloadButton = document.getElementById('download-button');
        const regenerateButton = document.getElementById('regenerate-button');
        const previewArea = document.getElementById('preview-area');
        
        // エラーオーバーレイ用ELEMENTS (ここから移植)
        const overlay = document.getElementById('error-overlay');
        const errorText = document.getElementById('error-text');
        const copyBtn = document.getElementById('copy-error');
        const closeBtn = document.getElementById('close-error');

        // エラー表示用ヘルパー関数
        function showError(error) {
            console.error("An error occurred:", error);
            try {
                const details = (error && (error.stack || error.message || String(error))) || 'Unknown error';
                errorText.textContent = details.trim();
            } catch(_) {
                errorText.textContent = 'Unknown error';
            }
            overlay.style.display = 'flex';
        }
        
        // エラーUI用イベントリスナー
        copyBtn?.addEventListener('click', async () => {
            try {
                await navigator.clipboard.writeText(errorText.textContent || '');
                copyBtn.textContent = 'コピーしました';
                setTimeout(() => (copyBtn.textContent = 'エラー全文をコピー'), 1200);
            } catch (_) {
                // フォールバック
                const r = document.createRange(); 
                r.selectNodeContents(errorText);
                const sel = window.getSelection(); 
                sel.removeAllRanges(); 
                sel.addRange(r);
            }
        });

        closeBtn?.addEventListener('click', () => { 
            overlay.style.display = 'none'; 
        });
        // (ここまで移植)


        async function main() {
            try {
                statusMessage.textContent = '実行環境を初期化しています...';
                const pyodide = await loadPyodide({ indexURL: "https://cdn.jsdelivr.net/pyodide/v0.28.2/full/" });
                pyodideInstance = pyodide;

                statusMessage.textContent = 'ライブラリを読み込んでいます...';
                await pyodide.loadPackage("matplotlib");

                statusMessage.textContent = 'Python関数を準備しています...';
                const pythonCode = document.getElementById('python-code').textContent;
                await pyodide.runPythonAsync(pythonCode);

                await runGeneration(true);

            } catch (error) {
                // 初期化エラーハンドリングを更新
                showError(error);
                statusMessage.textContent = '初期化に失敗しました。';
                previewArea.innerHTML = `<p class="text-red-500">エラーが発生しました。詳細はエラー表示を確認してください。</p>`;
            }
        }
        
        async function runGeneration(isInitial = false) {
            if (!pyodideInstance) return;
            try {
                statusMessage.textContent = isInitial ? '数式を生成しています...' : '数式を再生成しています...';
                previewArea.innerHTML = `<div class="preview-placeholder"><div class="spinner mx-auto mb-4"></div><p>${statusMessage.textContent}</p></div>`;
                regenerateButton.disabled = true;
                downloadButton.disabled = true;
                
                await pyodideInstance.globals.get('generate_formula')();
                
                const previewSrc = window.png_preview_src;
                if (previewSrc) {
                    previewArea.innerHTML = `<img src="${previewSrc}" alt="生成された数式" class="max-w-full h-auto block mx-auto">`;
                } else {
                    throw new Error("PNG generation returned no data.");
                }
                
                statusMessage.textContent = '✓ 生成完了';
            } catch (error) {
                // 生成エラーハンドリングを更新
                showError(error);
                statusMessage.textContent = '生成に失敗しました';
                previewArea.innerHTML = `<p class="text-red-500">生成エラー: ${error.message}</p>`;
            } finally {
                regenerateButton.disabled = false;
                downloadButton.disabled = false;
            }
        }

        function downloadFile(base64Data, fileName) {
            const byteCharacters = atob(base64Data);
            const byteNumbers = new Array(byteCharacters.length);
            for (let i = 0; i < byteCharacters.length; i++) {
                byteNumbers[i] = byteCharacters.charCodeAt(i);
            }
            const byteArray = new Uint8Array(byteNumbers);
            const blob = new Blob([byteArray], { type: 'image/png' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = fileName;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        }
        
        // --- Event Listeners Setup ---
        regenerateButton.addEventListener('click', () => runGeneration(false));
        downloadButton.addEventListener('click', () => {
            if (window.png_base64_data) {
                downloadFile(window.png_base64_data, `formula-${Date.now()}.png`);
            }
        });
        const textColorPicker = document.getElementById('text-color-picker');
        const textColorInput = document.getElementById('text-color');
        const bgColorPicker = document.getElementById('bg-color-picker');
        const bgColorInput = document.getElementById('bg-color');
        textColorPicker.addEventListener('input', (e) => textColorInput.value = e.target.value);
        textColorInput.addEventListener('change', (e) => textColorPicker.value = e.target.value);
        bgColorPicker.addEventListener('input', (e) => bgColorInput.value = e.target.value);
        bgColorInput.addEventListener('change', (e) => bgColorPicker.value = e.target.value);

        const transparentBg = document.getElementById('transparent-bg');
        transparentBg.addEventListener('change', () => {
            previewArea.classList.toggle('transparent-bg', transparentBg.checked);
            bgColorPicker.disabled = transparentBg.checked;
            bgColorInput.disabled = transparentBg.checked;
        });

        window.onload = main;
    </script>
</body>
</html>
```

---

## **🤖 AI実装ガイド**

### **AIがこのレシピを使用する手順**

1.  **ユーザーの数式を受け取る**
    ユーザーからの「xの積分をグラフにして」といった自然言語の指示や、直接的なTeX形式の入力を受け取ります。

2.  **TeX形式に変換**
    入力が自然言語の場合、AIの能力で適切なTeX形式の文字列に変換します。
    -   入力: `"xの2乗"` → 変換: `"x^2"`
    -   入力: `"シグマ k=1からn"` → 変換: `\sum_{k=1}^{n}`

3.  **HTMLテンプレートに埋め込み**
    変換して得られたTeX形式の文字列を、上記のHTMLテンプレート内にある`<textarea>`要素の中に挿入します。
    ```html
    <textarea id="formula-input" ... >ここに変換したTeX形式を挿入</textarea>
    ```

4.  **完成したHTMLファイルを提供**
    最終的に完成した単一のHTMLファイルをユーザーに提供します。

---

## **📄 ライセンス**

MITライセンス - 自由に使用・改変・再配布・商用利用可能
