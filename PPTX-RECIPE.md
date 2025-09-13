# **PPTX自動生成ページ レシピ v3.4**
**AIによるワンクリック・ダウンロードページ生成の完全ガイド**

---

## **📌 このレシピについて**

生成AIがユーザーの指示に基づき、PowerPointファイル（.pptx）をブラウザ上で生成・ダウンロードできる**単一HTMLファイル**を自動作成するための完全な指示書です。

### **v3.4の特徴**
- 🎯 実装ルールを優先順位別に明確化
- 🖼️ 画像はすべてプレースホルダー方式（直接埋め込みなし）
- 📝 詳細な理由説明を含む充実した内容
- 🔍 エラー対処の手順と実例を完備
- 📊 実用的な配置ガイドを提供

---

## **🚨 実装ルール（優先順位順）**

### **🔴 レベル1: 絶対禁止事項（変更・削除厳禁）**

以下の要素は**一切変更してはいけません**：

#### **1.1 python-pptxライブラリのインストール処理**
```javascript
await pyodide.loadPackage("micropip");
await pyodide.runPythonAsync(`
    import micropip
    await micropip.install('python-pptx')
`);
```

**なぜ絶対に必要か：**
- python-pptxはPowerPointファイル生成のための外部ライブラリ
- ブラウザのPyodide環境には標準で含まれていない
- micropipを使用してランタイムでインストールする必要がある
- **削除すると必ず発生するエラー**: `ModuleNotFoundError: No module named 'pptx'`

#### **1.2 エラーオーバーレイのHTML/CSS/JavaScript**
- エラーオーバーレイのHTML構造（`<div id="error-overlay">`）
- エラーハンドリングのJavaScriptコード
- CSSクラス名（`.error-overlay`, `.error-card`等）

**理由**: エラー時のユーザビリティを確保する重要な機能

#### **1.3 Pyodideのロード処理**
```html
<script src="https://cdn.jsdelivr.net/pyodide/v0.25.0/full/pyodide.js"></script>
```

**理由**: Python実行環境の基盤となる必須ライブラリ

### **🟡 レベル2: 必須ルール（必ず従う）**

#### **2.1 Pythonコードのインデント規則**
```python
# ✅ 正しい（左端開始）
from pptx import Presentation
import base64

# ❌ 間違い（インデントあり）
    from pptx import Presentation  # IndentationError発生
    import base64
```

**理由**: HTMLのインデントに影響されてPythonコードにインデントを追加するとエラーになる

#### **2.2 必須のインポート文**
```python
from pptx import Presentation
from io import BytesIO
import base64
import js  # JavaScriptとの連携に必須
```

**省略した場合のエラー**: `NameError: name 'js' is not defined`

#### **2.3 Base64データの受け渡し**
Pythonコードの最終部で必ず実行：
```python
js.pptx_base64_data = pptx_base64
```

**理由**: JavaScriptがPPTXデータを受け取るための必須処理

### **🟢 レベル3: カスタマイズ可能要素**

以下は自由に変更できます：
- ページタイトル（`<title>`タグと`<h1>`タグ）
- Pythonコード内のプレゼンテーション内容
- スライドのデザイン・レイアウト
- ダウンロードファイル名
- プレースホルダーの配置と内容

---

## **🖼️ 画像プレースホルダー機能**

### **基本方針**
**すべての画像はプレースホルダーとして処理されます**
- 画像の直接埋め込みは行いません（base64エンコードによるコンテキストウィンドウ超過を防ぐため）
- 実際の画像挿入はユーザーがPowerPointを開いて行います
- AIは最適な配置位置と推奨サイズを提案します

### **なぜプレースホルダー方式なのか**
- **HTMLファイルの軽量化**: 数KBで済む（画像埋め込みだと数MB〜数十MB）
- **コンテキストウィンドウの節約**: AIの処理限界を超えない
- **柔軟性**: 後から自由に画像を変更可能
- **共有性**: 複数人での使い回しが容易
- **著作権**: 画像の権利問題を回避

### **プレースホルダーに含める情報**
1. **推奨ファイル名または説明**（例：`logo.png`、`メイン画像`）
2. **配置位置**（例：「右下」「中央」「左側」）
3. **推奨サイズ**（例：`800x600px`、`4x3 inches`）
4. **用途**（例：「企業ロゴ」「メインビジュアル」「背景画像」）

### **実装時の判断基準**
ユーザーが画像について言及した場合：
- 「ロゴを入れて」→ ロゴプレースホルダー作成
- 「画像を配置」→ 汎用プレースホルダー作成
- 「写真スペースを」→ 写真用プレースホルダー作成
- 画像ファイルが添付された場合 → ファイル名を参考にプレースホルダー作成

### **推奨配置パターン**
| スライドタイプ | 画像の種類 | 推奨位置 | 推奨サイズ |
|---------------|-----------|---------|-----------|
| タイトル | ロゴ | 右下/左下 | 2×1 inches |
| コンテンツ | メイン画像 | 右側 | 4×3 inches |
| データ | グラフ/図表 | 中央 | 6×4 inches |
| まとめ | アイコン | 箇条書き横 | 1×1 inches |
| 背景 | 装飾画像 | 全面 | 10×5.625 inches |

---

## **📝 HTMLテンプレート（完全版）**

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- カスタマイズ可能: ページタイトル -->
    <title>プレゼンテーションのダウンロード</title>

    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { font-family: 'Inter', sans-serif; }
        button:disabled { cursor: not-allowed; opacity: 0.6; }
        .spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            width: 36px;
            height: 36px;
            border-radius: 50%;
            border-left-color: #09f;
            animation: spin 1s ease infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        /* エラーオーバーレイ（変更禁止） */
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
<body class="bg-gray-100 flex items-center justify-center min-h-screen">

    <div class="bg-white rounded-2xl shadow-lg p-8 sm:p-12 text-center max-w-md w-full">
        <!-- カスタマイズ可能: 見出し -->
        <h1 class="text-2xl sm:text-3xl font-bold text-gray-800 mb-4">
            プレゼンテーション
        </h1>

        <p id="status-message" class="text-gray-600 mb-6 h-12 flex items-center justify-center">
            ファイルの準備をしています...
        </p>

        <div id="loading-spinner" class="spinner mx-auto mb-6"></div>

        <button id="download-button" class="w-full bg-blue-600 text-white font-bold py-3 px-6 rounded-lg shadow-md hover:bg-blue-700 transition duration-300 transform hover:scale-105" disabled>
            ファイルをダウンロード
        </button>
    </div>

    <!-- カスタマイズ可能: Pythonコード（ルールに従って） -->
    <script type="text/python" id="python-code">
from pptx import Presentation
from io import BytesIO
import base64
import js

# プレゼンテーションの作成
prs = Presentation()

# タイトルスライドの追加
slide_layout = prs.slide_layouts[0]
slide = prs.slides.add_slide(slide_layout)
title = slide.shapes.title
subtitle = slide.placeholders[1]

title.text = "AIによって自動生成"
subtitle.text = "このページは動的に作成されました"

# BytesIOに保存
pptx_io = BytesIO()
prs.save(pptx_io)
pptx_io.seek(0)

# Base64エンコード
pptx_bytes = pptx_io.read()
pptx_base64 = base64.b64encode(pptx_bytes).decode('utf-8')

# JavaScriptに渡す（必須）
js.pptx_base64_data = pptx_base64

print("PPTX file has been generated in memory.")
    </script>

    <!-- エラーオーバーレイ（変更禁止） -->
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

    <script src="https://cdn.jsdelivr.net/pyodide/v0.25.0/full/pyodide.js"></script>
    <script type="module">
        const statusMessage = document.getElementById('status-message');
        const downloadButton = document.getElementById('download-button');
        const loadingSpinner = document.getElementById('loading-spinner');
        const overlay = document.getElementById('error-overlay');
        const errorText = document.getElementById('error-text');
        const copyBtn = document.getElementById('copy-error');
        const closeBtn = document.getElementById('close-error');

        // エラーUI用イベントリスナー
        copyBtn?.addEventListener('click', async () => {
            try {
                await navigator.clipboard.writeText(errorText.textContent || '');
                copyBtn.textContent = 'コピーしました';
                setTimeout(() => (copyBtn.textContent = 'エラー全文をコピー'), 1200);
            } catch (_) {
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

        async function main() {
            try {
                statusMessage.textContent = '実行環境を読み込んでいます...';
                const pyodide = await loadPyodide({
                    indexURL: "https://cdn.jsdelivr.net/pyodide/v0.25.0/full/",
                });

                // python-pptxのインストール（削除禁止）
                statusMessage.textContent = 'ライブラリをインストール中...';
                await pyodide.loadPackage("micropip");
                await pyodide.runPythonAsync(`
                    import micropip
                    await micropip.install('python-pptx')
                `);

                statusMessage.textContent = 'ファイルを生成しています...';
                const pythonCode = document.getElementById('python-code').textContent;
                window.pptx_base64_data = null; 
                await pyodide.runPythonAsync(pythonCode);
                const base64Data = window.pptx_base64_data;

                if (!base64Data) {
                    throw new Error("Python code did not produce any data.");
                }

                statusMessage.textContent = '準備が完了しました！';
                loadingSpinner.style.display = 'none';
                downloadButton.disabled = false;
                downloadButton.addEventListener('click', () => downloadFile(base64Data, 'presentation.pptx'));

            } catch (error) {
                console.error("An error occurred:", error);
                
                try {
                    const details = (error && (error.stack || error.message || String(error))) || 'Unknown error';
                    errorText.textContent = details.trim();
                } catch(_) {
                    errorText.textContent = 'Unknown error';
                }
                
                overlay.style.display = 'flex';
                statusMessage.textContent = '処理を中断しました。';
                loadingSpinner.style.display = 'none';
            }
        }

        function downloadFile(base64Data, fileName) {
            try {
                const byteCharacters = atob(base64Data);
                const byteNumbers = new Array(byteCharacters.length);
                for (let i = 0; i < byteCharacters.length; i++) {
                    byteNumbers[i] = byteCharacters.charCodeAt(i);
                }
                const byteArray = new Uint8Array(byteNumbers);
                const blob = new Blob([byteArray], { type: 'application/vnd.openxmlformats-officedocument.presentationml.presentation' });

                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.style.display = 'none';
                a.href = url;
                a.download = fileName;
                document.body.appendChild(a);
                a.click();
                
                setTimeout(() => {
                    URL.revokeObjectURL(url);
                    document.body.removeChild(a);
                }, 100);

            } catch (error) {
                console.error("Download failed:", error);
                statusMessage.textContent = 'ダウンロードに失敗しました。';
            }
        }

        window.onload = main;
    </script>
</body>
</html>
```

---

## **🔧 エラー診断ガイド**

### **よくあるエラーと解決方法**

#### **1. ModuleNotFoundError**
```
ModuleNotFoundError: No module named 'pptx'
```
**原因**: python-pptxのインストール処理が削除されている
**解決**: `await micropip.install('python-pptx')` を復元

#### **2. IndentationError**
```
IndentationError: unexpected indent
```
**原因**: Pythonコードが左端から開始されていない
**解決**: すべてのPythonコードのインデントを削除

#### **3. NameError**
```
NameError: name 'js' is not defined
```
**原因**: `import js` が抜けている
**解決**: 必須インポート文に `import js` を追加

#### **4. AttributeError**
```
AttributeError: 'NoneType' object has no attribute 'text'
```
**原因**: スライドレイアウトに該当プレースホルダーが存在しない
**解決**: レイアウトインデックスを確認、または `add_textbox()` を使用

### **エラー発生時の対処手順（4ステップ）**

1. **自動表示**: エラーオーバーレイがフルスクリーンで表示される
   - 見逃すことがない高視認性デザイン
   - エラーの詳細（スタックトレース）が完全表示

2. **ワンクリックコピー**: 「エラー全文をコピー」ボタンをクリック
   - クリップボードに自動コピー
   - 「コピーしました」の確認表示

3. **AIに貼り付け**: チャットエリアにそのまま貼り付け
   - AIが自動的にエラー内容を分析
   - 原因を特定

4. **解決**: AIが提示する修正版を使用
   - 具体的な修正内容の説明
   - 修正済みコードの提供

---

## **💡 実装例**

### **レベル1: 最小限の実装**
```python
from pptx import Presentation
from io import BytesIO
import base64
import js

prs = Presentation()
slide = prs.slides.add_slide(prs.slide_layouts[0])
slide.shapes.title.text = "シンプルなタイトル"

pptx_io = BytesIO()
prs.save(pptx_io)
pptx_io.seek(0)
js.pptx_base64_data = base64.b64encode(pptx_io.read()).decode('utf-8')
```

### **レベル2: 複数スライドの基本構成**
```python
from pptx import Presentation
from pptx.util import Inches, Pt
from io import BytesIO
import base64
import js

prs = Presentation()

# スライド1: タイトル
slide = prs.slides.add_slide(prs.slide_layouts[0])
slide.shapes.title.text = "2025年度事業計画"
slide.placeholders[1].text = "株式会社サンプル"

# スライド2: 内容
slide = prs.slides.add_slide(prs.slide_layouts[1])
slide.shapes.title.text = "主要目標"
content = slide.placeholders[1]
content.text = "• 売上20%増加\n• 新規顧客100社獲得\n• 顧客満足度95%達成"

# スライド3: まとめ
slide = prs.slides.add_slide(prs.slide_layouts[1])
slide.shapes.title.text = "まとめ"
slide.placeholders[1].text = "目標達成に向けて全社一丸となって取り組みます"

# 保存と変換
pptx_io = BytesIO()
prs.save(pptx_io)
pptx_io.seek(0)
js.pptx_base64_data = base64.b64encode(pptx_io.read()).decode('utf-8')
```

### **レベル3: 画像プレースホルダー付き**
```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor
from io import BytesIO
import base64
import js

prs = Presentation()

# タイトルスライド（ロゴプレースホルダー付き）
slide = prs.slides.add_slide(prs.slide_layouts[0])
slide.shapes.title.text = "年次報告書 2025"
slide.placeholders[1].text = "株式会社サンプル"

# ロゴプレースホルダー
textbox = slide.shapes.add_textbox(
    left=Inches(7.5), top=Inches(4.5),
    width=Inches(2), height=Inches(1)
)
text_frame = textbox.text_frame
text_frame.text = "【画像推奨】\nlogo.png\n企業ロゴ\n200x100px"
textbox.line.color.rgb = RGBColor(200, 200, 200)
textbox.line.width = Pt(0.5)

# コンテンツスライド（メイン画像プレースホルダー付き）
slide = prs.slides.add_slide(prs.slide_layouts[1])
slide.shapes.title.text = "製品紹介"
slide.placeholders[1].text = "• 高性能\n• 使いやすい\n• コストパフォーマンス"

# メイン画像プレースホルダー
textbox = slide.shapes.add_textbox(
    left=Inches(5), top=Inches(1.5),
    width=Inches(4), height=Inches(3)
)
text_frame = textbox.text_frame
text_frame.text = "【画像推奨】\nproduct_main.jpg\nメイン製品画像\n800x600px\n\n製品の全体像が\nよく分かる画像を\n配置してください"
text_frame.paragraphs[0].alignment = PP_ALIGN.CENTER
textbox.fill.solid()
textbox.fill.fore_color.rgb = RGBColor(240, 240, 240)

# 保存
pptx_io = BytesIO()
prs.save(pptx_io)
pptx_io.seek(0)
js.pptx_base64_data = base64.b64encode(pptx_io.read()).decode('utf-8')
```

### **レベル4: 条件分岐を含む高度な実装**
```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor
from io import BytesIO
import base64
import js

def add_image_placeholder(slide, position, size, description):
    """汎用的な画像プレースホルダー追加関数"""
    left, top = position
    width, height = size
    
    textbox = slide.shapes.add_textbox(left, top, width, height)
    text_frame = textbox.text_frame
    text_frame.text = description
    
    # 視覚的な枠線
    textbox.line.color.rgb = RGBColor(150, 150, 150)
    textbox.line.width = Pt(1)
    
    # 背景色
    textbox.fill.solid()
    textbox.fill.fore_color.rgb = RGBColor(250, 250, 250)
    
    return textbox

prs = Presentation()

# スライドタイプに応じた処理
slide_configs = [
    {"type": "title", "title": "プレゼンテーション", "subtitle": "サンプル株式会社"},
    {"type": "content", "title": "概要", "content": "主要なポイント"},
    {"type": "image", "title": "ビジュアル", "content": None}
]

for config in slide_configs:
    if config["type"] == "title":
        slide = prs.slides.add_slide(prs.slide_layouts[0])
        slide.shapes.title.text = config["title"]
        slide.placeholders[1].text = config["subtitle"]
        
        # ロゴプレースホルダー
        add_image_placeholder(
            slide,
            position=(Inches(7), Inches(4.5)),
            size=(Inches(2), Inches(1)),
            description="【ロゴ】\nlogo.png\n200x100px"
        )
        
    elif config["type"] == "content":
        slide = prs.slides.add_slide(prs.slide_layouts[1])
        slide.shapes.title.text = config["title"]
        if config["content"]:
            slide.placeholders[1].text = config["content"]
            
    elif config["type"] == "image":
        slide = prs.slides.add_slide(prs.slide_layouts[5])  # 白紙
        
        # タイトル追加
        title_box = slide.shapes.add_textbox(
            Inches(0.5), Inches(0.3),
            Inches(9), Inches(0.8)
        )
        title_box.text_frame.text = config["title"]
        title_box.text_frame.paragraphs[0].font.size = Pt(32)
        title_box.text_frame.paragraphs[0].font.bold = True
        
        # 大きな画像プレースホルダー
        add_image_placeholder(
            slide,
            position=(Inches(1), Inches(1.5)),
            size=(Inches(8), Inches(4)),
            description="【メイン画像】\nmain_visual.jpg\n1600x800px\n高解像度推奨"
        )

# 保存
pptx_io = BytesIO()
prs.save(pptx_io)
pptx_io.seek(0)
js.pptx_base64_data = base64.b64encode(pptx_io.read()).decode('utf-8')
```

---

## **✅ 実装チェックリスト**

### **必須確認項目**
- [ ] python-pptxインストール処理が存在する
- [ ] Pythonコードが左端から開始されている
- [ ] `import js` が含まれている
- [ ] `js.pptx_base64_data` への代入がある
- [ ] エラーオーバーレイのHTMLが完全である

### **プレースホルダー確認項目**
- [ ] 画像の直接埋め込みを行っていない
- [ ] プレースホルダーに必要な情報が含まれている
- [ ] 視覚的に分かりやすい枠線がある
- [ ] 推奨サイズが明記されている

### **動作確認項目**
- [ ] ページが正常に読み込まれる
- [ ] ダウンロードボタンが有効になる
- [ ] PPTXファイルがダウンロードできる
- [ ] エラー時にオーバーレイが表示される
- [ ] エラーコピーボタンが機能する

---

## **📊 python-pptxリファレンス**

### **スライドレイアウト**
| Index | レイアウト名 | 用途 | プレースホルダー |
|-------|-------------|------|-----------------|
| 0 | タイトルスライド | 表紙 | title, subtitle |
| 1 | タイトルとコンテンツ | 標準 | title, content |
| 2 | セクションヘッダー | 章区切り | title, content |
| 3 | 2つのコンテンツ | 比較 | title, content×2 |
| 4 | 比較 | 対比表示 | title, content×2 |
| 5 | 白紙 | 自由配置 | なし |
| 6 | コンテンツとキャプション | 説明付き | title, content, caption |

### **よく使うメソッド**
```python
# プレゼンテーション作成
prs = Presentation()

# スライド追加
slide = prs.slides.add_slide(prs.slide_layouts[n])

# テキスト設定
slide.shapes.title.text = "タイトル"
slide.placeholders[1].text = "内容"

# テキストボックス追加
textbox = slide.shapes.add_textbox(left, top, width, height)
textbox.text_frame.text = "テキスト"

# フォント設定
from pptx.util import Pt
paragraph = textbox.text_frame.paragraphs[0]
paragraph.font.size = Pt(24)
paragraph.font.bold = True

# 色設定
from pptx.dml.color import RGBColor
paragraph.font.color.rgb = RGBColor(255, 0, 0)

# 配置設定
from pptx.enum.text import PP_ALIGN
paragraph.alignment = PP_ALIGN.CENTER

# 枠線設定
textbox.line.color.rgb = RGBColor(0, 0, 0)
textbox.line.width = Pt(1)

# 背景色設定
textbox.fill.solid()
textbox.fill.fore_color.rgb = RGBColor(255, 255, 255)
```

### **単位の扱い**
```python
from pptx.util import Inches, Pt, Cm

# インチ単位
left = Inches(1)  # 1インチ

# ポイント単位（フォントサイズ等）
size = Pt(24)  # 24ポイント

# センチメートル単位
width = Cm(10)  # 10cm
```

---

## **🚀 クイックスタート**

### **最速実装（3ステップ）**

1. **HTMLテンプレートをコピー**
2. **タイトルを変更**（`<title>`と`<h1>`の2箇所）
3. **Pythonコードでスライド内容を定義**

```python
# 最小限の変更例
prs = Presentation()
slide = prs.slides.add_slide(prs.slide_layouts[0])
slide.shapes.title.text = "あなたのタイトルをここに"
slide.placeholders[1].text = "サブタイトルをここに"
# ... 残りは保存処理（テンプレート通り）
```

### **画像が必要な場合**

1. **プレースホルダーを配置**
```python
textbox = slide.shapes.add_textbox(
    Inches(5), Inches(2),  # 位置
    Inches(4), Inches(3)   # サイズ
)
textbox.text_frame.text = "【画像】\nここに画像を配置"
```

2. **ユーザーがPowerPointで画像を追加**
   - ダウンロードしたPPTXファイルを開く
   - プレースホルダーを削除
   - 実際の画像を挿入

---

## **📝 更新履歴**

- **v3.4** (2025-01-17)
  - 画像処理方針を明確化（すべてプレースホルダー）
  - v3.1.5の詳細説明を復活
  - エラー対処手順を完備
  - 推奨配置パターンを追加

- **v3.3** (2025-01-17)
  - 実装ルールを優先順位別に再構成
  - エラーメッセージの実例を追加
  - 構造を論理的に最適化

- **v3.1.5** (2025-01-17)
  - エラーUI機能を強化
  - ワンクリックコピー機能を実装

- **v3.1** (2025-01-17)
  - 必須要件の明確化
  - エラー防止の強化

---

## **📄 ライセンス**

MITライセンス - 自由に使用・改変・再配布・商用利用可能

---

## **🆘 サポート**

### **問題が発生した場合の対処**

1. **エラーメッセージをコピー**
   - エラーオーバーレイの「エラー全文をコピー」ボタンを使用

2. **AIに貼り付けて相談**
   - コピーした内容をチャットエリアに貼り付け
   - AIが原因を分析し、修正版を提供

3. **チェックリストで確認**
   - 必須項目が漏れていないか確認
   - 特にpython-pptxのインストール処理

4. **実装例を参考に**
   - レベル1の最小限実装から始める
   - 段階的に機能を追加

### **よくある質問**

**Q: 画像を直接埋め込みたい**
A: base64エンコードによるコンテキストウィンドウ超過を防ぐため、プレースホルダー方式を採用しています。

**Q: プレースホルダーの見た目を変えたい**
A: テキストボックスの`line`、`fill`プロパティで自由にカスタマイズ可能です。

**Q: 特殊なレイアウトを作りたい**
A: レイアウト5（白紙）を使用し、`add_textbox()`で自由に配置してください。

---

**これでレシピv3.4の全文です。AIが正しく理解し、適切なPPTX生成ページを作成できるよう、詳細な説明と実例を含めました。**