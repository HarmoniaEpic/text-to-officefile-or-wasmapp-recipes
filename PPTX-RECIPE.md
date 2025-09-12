# **PPTX自動生成ページ レシピ v3.1**

このドキュメントは、生成AIがユーザーの指示に基づき、「ワンクリック・ダウンロードページ」を自動生成するためのレシピ（指示書＋テンプレート）です。

AIはこのレシピに従い、最終的な成果物として単一のHTMLファイルを出力します。

## **🔴 必須要件 - 必ず最初に確認してください 🔴**

### **1. 外部ライブラリのインストール（絶対に省略禁止）**

**⚠️ 最重要事項：python-pptxライブラリのインストール処理は絶対に削除・省略してはいけません**

HTMLテンプレート内の以下のコードブロックは**必須**です：
```javascript
// このコードブロックは削除厳禁
await pyodide.loadPackage("micropip");
await pyodide.runPythonAsync(`
    import micropip
    await micropip.install('python-pptx')  # ← これがないとPowerPointファイルを生成できません
`);
```

**なぜ必要か？**
- python-pptxはPowerPointファイル（.pptx）を生成するための外部ライブラリです
- ブラウザ環境では標準では利用できないため、micropipでインストールが必要です
- このインストール処理を削除すると、`ModuleNotFoundError: No module named 'pptx'`エラーが発生します

### **2. Pythonコードのインデント規則（エラー防止のため厳守）**

**⚠️ IndentationErrorを防ぐため、以下の規則を必ず守ってください**

- **scriptタグ内のPythonコードは、絶対に左端から開始すること**
- HTMLのインデントに影響されてPythonコードにインデントを追加してはいけない
- `<script type="text/python" id="python-code">` の直後の行から、インデントなしでPythonコードを記述する

**正しい例：**
```html
    <script type="text/python" id="python-code">
from pptx import Presentation  # ← 左端から開始（正しい）
import base64  # ← インデントなし（正しい）
```

**間違った例：**
```html
    <script type="text/python" id="python-code">
        from pptx import Presentation  # ← ❌ 不要なインデント（エラーの原因）
        import base64  # ← ❌ IndentationErrorが発生
```

## **🖼️ 画像プレースホルダー機能について**

### **画像が添付された場合の動作**
- AIは添付された画像を分析し、効果的な配置位置を提案します
- 生成される文書には画像そのものではなく、配置推奨位置のプレースホルダーが挿入されます
- **すべての添付画像を使用する必要はありません**（最適なものだけを提案します）
- プレースホルダーは後から手動で画像に置き換えることを想定しています

### **プレースホルダーの内容**
各プレースホルダーには以下の情報が含まれます：
- 推奨する画像ファイル名
- 配置位置の説明（例：「メインビジュアルとして中央配置」）
- 推奨サイズ（幅×高さ）
- 使用目的（ロゴ、メインビジュアル、説明図、背景など）

### **画像選択の基準**
AIは以下の基準で画像の用途を判定します：
- **アスペクト比**：横長→背景/ヘッダー、正方形→アイコン/ロゴ
- **ファイル名**：「logo」→ロゴ位置、「chart」→データセクション
- **画像の数**：複数ある場合は優先順位を付けて提案
- **スライドの内容**：タイトルスライド、コンテンツスライドなどに応じた配置

### **ユーザーへのメリット**
- 複雑な画像埋め込み処理が不要
- 後から手動で最適な画像を選択・配置可能
- プロフェッショナルなレイアウト提案を受けられる
- ファイルサイズの軽量化

## **📋 AIへの実装指示**

以下のHTML骨子テンプレートを使用し、指定された箇所をユーザーの要求に応じて書き換えてください。**それ以外の部分（特にライブラリインストール部分）は一切変更してはいけません。**

### **書き換える箇所**

1. **ページのタイトルと見出しを書き換える**  
   * `<!-- AIがここを書き換える: ページのタイトル -->` とコメントされている2箇所を、生成するプレゼンテーションの内容に合わせた具体的なタイトルに書き換えてください。

2. **Pythonコードを書き換える**  
   * `<script type="text/python" id="python-code">` タグの内部を、要求されたプレゼンテーションを生成するための python-pptx コードに完全に置き換えてください。
   * **画像が添付された場合**：画像プレースホルダーを適切な位置に追加してください
   * **重要**: Pythonコードは必ず左端から開始し、HTMLのインデントを無視してください
   * このPythonコードは、必ず**最終行で `js.pptx_base64_data` にBase64文字列を代入する**ルールに従う必要があります。

### **Pythonコード作成のルール**

1. **必須のインポート文**（左端から記述）
```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor
from io import BytesIO
import base64
import js
```

2. **プレゼンテーション作成の基本構造**
```python
# プレゼンテーションの作成
prs = Presentation()

# スライドの追加
slide_layout = prs.slide_layouts[0]  # 0: タイトルスライド, 1: タイトルとコンテンツ
slide = prs.slides.add_slide(slide_layout)

# タイトルと内容の設定
title = slide.shapes.title
title.text = "スライドタイトル"

# コンテンツの設定（レイアウト1の場合）
content = slide.placeholders[1]
content.text = "コンテンツテキスト"
```

3. **画像プレースホルダーの追加方法**
```python
# 画像プレースホルダーの追加（画像が添付された場合）
# 例：タイトルスライドにロゴ配置の提案
left = Inches(7)  # 右下配置
top = Inches(4)
width = Inches(2)
height = Inches(1)
textbox = slide.shapes.add_textbox(left, top, width, height)
text_frame = textbox.text_frame
text_frame.text = "【画像推奨】\nファイル: logo.png\n用途: 企業ロゴ\n推奨サイズ: 200x100px"

# 枠線を追加して視認性を向上
textbox.line.color.rgb = RGBColor(200, 200, 200)
textbox.line.width = Pt(1)

# 例：コンテンツスライドにメイン画像配置の提案
left = Inches(5)  # 右側配置
top = Inches(1.5)
width = Inches(4)
height = Inches(3)
textbox = slide.shapes.add_textbox(left, top, width, height)
text_frame = textbox.text_frame
text_frame.text = "【画像推奨】\nファイル: main_visual.jpg\n用途: メインビジュアル\n推奨サイズ: 800x600px\n説明: このエリアに製品画像を配置すると効果的です"
```

4. **必須の終了処理**（必ず含める）
```python
# BytesIOに保存
pptx_io = BytesIO()
prs.save(pptx_io)
pptx_io.seek(0)

# Base64エンコード
pptx_bytes = pptx_io.read()
pptx_base64 = base64.b64encode(pptx_bytes).decode('utf-8')

# JavaScriptに渡す（この行は必須）
js.pptx_base64_data = pptx_base64

print("PPTX file has been generated in memory.")
```

## **📄 HTML骨子テンプレート**

**⚠️ 重要：以下のテンプレートの中で、赤字でマークされた部分は絶対に削除・変更しないでください**

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- AIがここを書き換える: ページのタイトル -->
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
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen">

    <div class="bg-white rounded-2xl shadow-lg p-8 sm:p-12 text-center max-w-md w-full">
        <!-- AIがここを書き換える: ページのタイトル -->
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

    <!-- 
      AIがここを書き換える: python-pptxのコード
      重要: Pythonコードは左端から開始（インデントなし）
    -->
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

    <!-- 
      ⚠️⚠️⚠️ 警告: 以下のPyodideとpython-pptxインストール部分は削除厳禁 ⚠️⚠️⚠️
      この部分を削除すると、PowerPointファイルの生成が不可能になります
    -->
    <script src="https://cdn.jsdelivr.net/pyodide/v0.25.0/full/pyodide.js"></script>
    <script type="module">
        const statusMessage = document.getElementById('status-message');
        const downloadButton = document.getElementById('download-button');
        const loadingSpinner = document.getElementById('loading-spinner');

        async function main() {
            try {
                statusMessage.textContent = '実行環境を読み込んでいます...';
                const pyodide = await loadPyodide({
                    indexURL: "https://cdn.jsdelivr.net/pyodide/v0.25.0/full/",
                });

                // 🔴🔴🔴 削除厳禁: python-pptxライブラリのインストール 🔴🔴🔴
                // このコードブロックを削除するとModuleNotFoundErrorが発生します
                statusMessage.textContent = 'ライブラリをインストール中...';
                await pyodide.loadPackage("micropip");
                await pyodide.runPythonAsync(`
                    import micropip
                    await micropip.install('python-pptx')  # PowerPoint生成に必須のライブラリ
                `);
                // 🔴🔴🔴 ここまで削除厳禁 🔴🔴🔴

                statusMessage.textContent = 'ファイルを生成しています...';
                const pythonCode = document.getElementById('python-code').textContent;
                // JavaScriptのグローバルスコープにデータを渡すための準備
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
                statusMessage.textContent = `エラーが発生しました: ${error.message}`;
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

## **⚠️ よくあるエラーと対処法**

### **ModuleNotFoundError: No module named 'pptx'**
**原因**: micropipでpython-pptxをインストールする処理が削除または省略されている  
**対処法**: HTMLテンプレートの中の `await micropip.install('python-pptx')` の部分を復元する

### **IndentationError: unexpected indent**
**原因**: Pythonコードに不要なインデントが含まれている  
**対処法**: すべてのPythonコードを左端から開始する

### **NameError: name 'js' is not defined**
**原因**: `import js` が抜けている  
**対処法**: 必須インポート文をすべて含める

### **AttributeError: 'NoneType' object has no attribute 'text'**
**原因**: プレースホルダーのインデックスが間違っている  
**対処法**: `slide.placeholders[1]` のインデックスを確認

## **💡 実装例：画像プレースホルダー付きプレゼンテーション**

```python
from pptx import Presentation
from pptx.util import Pt, Inches
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor
from io import BytesIO
import base64
import js

prs = Presentation()

# スライド1: タイトル（ロゴプレースホルダー付き）
slide_layout = prs.slide_layouts[0]
slide = prs.slides.add_slide(slide_layout)
title = slide.shapes.title
subtitle = slide.placeholders[1]
title.text = "年次報告書 2025"
subtitle.text = "株式会社サンプル"

# ロゴプレースホルダーを右下に配置
left = Inches(7.5)
top = Inches(4.5)
width = Inches(2)
height = Inches(1)
textbox = slide.shapes.add_textbox(left, top, width, height)
text_frame = textbox.text_frame
text_frame.text = "【画像推奨】\nlogo.png\n企業ロゴ\n200x100px"
textbox.line.color.rgb = RGBColor(200, 200, 200)
textbox.line.width = Pt(0.5)

# スライド2: コンテンツ（メイン画像プレースホルダー付き）
slide_layout = prs.slide_layouts[1]
slide = prs.slides.add_slide(slide_layout)
title = slide.shapes.title
content = slide.placeholders[1]
title.text = "製品紹介"
content.text = "• 高性能\n• 使いやすい\n• コストパフォーマンス"

# メイン画像プレースホルダーを右側に配置
left = Inches(5)
top = Inches(1.5)
width = Inches(4)
height = Inches(3.5)
textbox = slide.shapes.add_textbox(left, top, width, height)
text_frame = textbox.text_frame
text_frame.text = "【画像推奨】\nproduct_main.jpg\nメイン製品画像\n800x700px\n\n製品の全体像が\nよく分かる画像を\n配置してください"
text_frame.paragraphs[0].alignment = PP_ALIGN.CENTER
textbox.line.color.rgb = RGBColor(100, 100, 100)
textbox.line.width = Pt(1)
textbox.fill.solid()
textbox.fill.fore_color.rgb = RGBColor(240, 240, 240)

# スライド3: データ（グラフ画像プレースホルダー付き）
slide_layout = prs.slide_layouts[5]  # 白紙
slide = prs.slides.add_slide(slide_layout)

# タイトルを手動で追加
left = Inches(0.5)
top = Inches(0.3)
width = Inches(9)
height = Inches(0.8)
title_box = slide.shapes.add_textbox(left, top, width, height)
title_frame = title_box.text_frame
p = title_frame.add_paragraph()
p.text = "売上推移"
p.font.size = Pt(32)
p.font.bold = True
p.alignment = PP_ALIGN.CENTER

# グラフプレースホルダー
left = Inches(1)
top = Inches(1.5)
width = Inches(8)
height = Inches(4)
textbox = slide.shapes.add_textbox(left, top, width, height)
text_frame = textbox.text_frame
text_frame.text = "【画像推奨】\nsales_chart.png または sales_graph.jpg\n売上グラフ・チャート\n推奨: 1600x800px\n\nExcelやその他のツールで作成した\nグラフ画像をここに配置してください"
text_frame.paragraphs[0].alignment = PP_ALIGN.CENTER
for paragraph in text_frame.paragraphs:
    paragraph.font.size = Pt(14)
textbox.line.color.rgb = RGBColor(50, 50, 200)
textbox.line.width = Pt(2)

# 保存とBase64変換
pptx_io = BytesIO()
prs.save(pptx_io)
pptx_io.seek(0)

pptx_bytes = pptx_io.read()
pptx_base64 = base64.b64encode(pptx_bytes).decode('utf-8')

js.pptx_base64_data = pptx_base64

print("PPTX file with image placeholders has been generated.")
```

## **📚 python-pptxの主要機能リファレンス**

### **基本操作**
- `Presentation()`: 新規プレゼンテーションの作成
- `prs.slides.add_slide(layout)`: スライドの追加
- `slide.shapes.title`: タイトルへのアクセス
- `slide.placeholders[n]`: プレースホルダーへのアクセス

### **テキストボックス（画像プレースホルダー用）**
```python
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor

# テキストボックスの追加
textbox = slide.shapes.add_textbox(left, top, width, height)
text_frame = textbox.text_frame
text_frame.text = "テキスト"

# 枠線の設定
textbox.line.color.rgb = RGBColor(200, 200, 200)
textbox.line.width = Pt(1)

# 背景色の設定
textbox.fill.solid()
textbox.fill.fore_color.rgb = RGBColor(240, 240, 240)

# テキストの配置
text_frame.paragraphs[0].alignment = PP_ALIGN.CENTER
```

### **レイアウトの種類**
- `0`: タイトルスライド
- `1`: タイトルとコンテンツ
- `2`: セクションヘッダー
- `3`: 2つのコンテンツ
- `4`: 比較
- `5`: 白紙
- `6`: コンテンツとキャプション
- `7`: 画像とキャプション

### **画像プレースホルダーの配置戦略**

#### **スライドタイプ別の推奨位置**
1. **タイトルスライド**
   - ロゴ: 右下または左下（小さめ）
   - 背景画像: 全体（透明度調整の注記付き）

2. **コンテンツスライド**
   - メイン画像: 右側半分
   - 補助画像: 下部に横並び

3. **データスライド**
   - グラフ: 中央大きく
   - 参考画像: 隅に小さく

4. **まとめスライド**
   - アイコン: 箇条書きの横
   - イメージ画像: 背景または上部

## **✅ 最終チェックリスト**

AIがHTMLを生成する際の確認事項：

- [ ] **python-pptxのインストール処理（`await micropip.install('python-pptx')`）が含まれているか**
- [ ] Pythonコードは左端から開始されているか
- [ ] 必須のimport文がすべて含まれているか
- [ ] `js.pptx_base64_data = pptx_base64` の行が存在するか
- [ ] 画像が添付された場合、適切なプレースホルダーを追加しているか
- [ ] プレースホルダーに必要な情報（ファイル名、用途、サイズ）が含まれているか
- [ ] ページタイトルは適切に変更されているか
- [ ] MIMEタイプが正しく設定されているか
- [ ] エラーメッセージの表示処理が含まれているか
- [ ] Pyodideのロード処理が含まれているか

## **🎯 応用例**

### **複数画像のグリッド配置プレースホルダー**
```python
# 4つの画像をグリッド配置
positions = [
    (Inches(1), Inches(2)),   # 左上
    (Inches(5), Inches(2)),   # 右上
    (Inches(1), Inches(4)),   # 左下
    (Inches(5), Inches(4))    # 右下
]

for i, (left, top) in enumerate(positions, 1):
    textbox = slide.shapes.add_textbox(left, top, Inches(3.5), Inches(1.5))
    text_frame = textbox.text_frame
    text_frame.text = f"【画像{i}】\nimage{i}.jpg\n600x400px"
    textbox.line.color.rgb = RGBColor(150, 150, 150)
```

### **条件付きプレースホルダー**
```python
# ユーザーの要求に応じて配置を変更
if "logo" in attached_images:  # 仮定: attached_imagesに画像名リスト
    # ロゴプレースホルダーを追加
    add_logo_placeholder(slide)

if "chart" in content_keywords:  # 仮定: content_keywordsにキーワード
    # グラフプレースホルダーを追加
    add_chart_placeholder(slide)
```

## **📝 更新履歴**

- **v3.1** (2025-01-17): 外部ライブラリインストールの重要性を強調
  - 必須要件セクションを冒頭に追加
  - HTMLテンプレート内にコメントで警告を追加
  - チェックリストにpython-pptx確認項目を追加
  - エラー対処法にModuleNotFoundErrorを追加
- **v3.0** (2025-01-17): 画像プレースホルダー機能を追加
  - 画像配置のサジェスチョン機能
  - プレースホルダーの視覚的表現
  - 画像選択の自動判定基準
- **v2.0** (2025-01-17): インデントエラー防止のための明確な指示を追加
- **v1.0** (2025-01-16): 初版リリース

## **📄 ライセンス**

本レシピはMITライセンスの下で提供されています。自由に使用、改変、再配布、商用利用が可能です。

## **🆘 トラブルシューティング**

もしAIがこのレシピを正しく実行できない場合：

1. **最初に必須要件セクションを確認** - 特にpython-pptxのインストール処理
2. **HTMLテンプレートの警告コメントを確認** - 削除厳禁の箇所が残っているか
3. **チェックリストを使用して検証** - すべての項目が満たされているか
4. **エラーメッセージを確認** - ModuleNotFoundErrorの場合はライブラリインストールの問題

**それでも問題が解決しない場合は、このレシピの最新版を確認してください。**