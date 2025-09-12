# **DOCX自動生成ページ レシピ v2.1**

このドキュメントは、生成AIがユーザーの指示に基づき、「ワンクリック・ダウンロードページ」を自動生成するためのレシピ（指示書＋テンプレート）です。

AIはこのレシピに従い、最終的な成果物として単一のHTMLファイルを出力します。

## **🔴 必須要件 - 必ず最初に確認してください 🔴**

### **1. 外部ライブラリのインストール（絶対に省略禁止）**

**⚠️ 最重要事項：python-docxライブラリのインストール処理は絶対に削除・省略してはいけません**

HTMLテンプレート内の以下のコードブロックは**必須**です：
```javascript
// このコードブロックは削除厳禁
await pyodide.loadPackage("micropip");
await pyodide.runPythonAsync(`
    import micropip
    await micropip.install('python-docx')  # ← これがないとWordファイルを生成できません
`);
```

**なぜ必要か？**
- python-docxはWordファイル（.docx）を生成するための外部ライブラリです
- ブラウザ環境では標準では利用できないため、micropipでインストールが必要です
- このインストール処理を削除すると、`ModuleNotFoundError: No module named 'docx'`エラーが発生します

### **2. Pythonコードのインデント規則（エラー防止のため厳守）**

**⚠️ IndentationErrorを防ぐため、以下の規則を必ず守ってください**

- **scriptタグ内のPythonコードは、絶対に左端から開始すること**
- HTMLのインデントに影響されてPythonコードにインデントを追加してはいけない
- `<script type="text/python" id="python-code">` の直後の行から、インデントなしでPythonコードを記述する

**正しい例：**
```html
    <script type="text/python" id="python-code">
from docx import Document  # ← 左端から開始（正しい）
import base64  # ← インデントなし（正しい）
```

**間違った例：**
```html
    <script type="text/python" id="python-code">
        from docx import Document  # ← ❌ 不要なインデント（エラーの原因）
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
- 配置位置の説明（例：「章の冒頭に配置」「説明図として本文中に挿入」）
- 推奨サイズ（幅×高さ）
- 使用目的（ロゴ、挿絵、図表、グラフ、写真など）
- キャプションの提案

### **画像選択の基準**
AIは以下の基準で画像の用途を判定します：
- **アスペクト比**：横長→ヘッダー/図表、縦長→挿絵、正方形→アイコン/ロゴ
- **ファイル名**：「logo」→ヘッダー位置、「chart」「graph」→データセクション、「fig」→図表
- **画像の数**：複数ある場合は文書の流れに沿って適切に配置
- **文書の構成**：章立て、セクション、段落に応じた配置

### **ユーザーへのメリット**
- 複雑な画像埋め込み処理が不要
- 後から手動で最適な画像を選択・配置可能
- プロフェッショナルな文書レイアウトの提案を受けられる
- ファイルサイズの軽量化
- Wordの画像挿入機能で簡単に置き換え可能

## **📋 AIへの実装指示**

以下のHTML骨子テンプレートを使用し、指定された箇所をユーザーの要求に応じて書き換えてください。**それ以外の部分（特にライブラリインストール部分）は一切変更してはいけません。**

### **書き換える箇所**

1. **ページのタイトルと見出しを書き換える**  
   * `<!-- AIがここを書き換える: ページのタイトル -->` とコメントされている2箇所を、生成する文書の内容に合わせた具体的なタイトルに書き換えてください。

2. **Pythonコードを書き換える**  
   * `<script type="text/python" id="python-code">` タグの内部を、要求された文書を生成するための python-docx コードに完全に置き換えてください。
   * **画像が添付された場合**：画像プレースホルダーを適切な位置に追加してください
   * **重要**: Pythonコードは必ず左端から開始し、HTMLのインデントを無視してください
   * このPythonコードは、必ず**最終行で `js.docx_base64_data` にBase64文字列を代入する**ルールに従う必要があります。

### **Pythonコード作成のルール**

1. **必須のインポート文**（左端から記述）
```python
from docx import Document
from docx.shared import Inches, Pt, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.enum.style import WD_STYLE_TYPE
from io import BytesIO
import base64
import js
```

2. **文書作成の基本構造**
```python
# 文書の作成
doc = Document()

# タイトルの追加
doc.add_heading('文書タイトル', 0)

# 段落の追加
p = doc.add_paragraph('通常のテキスト')
p.alignment = WD_ALIGN_PARAGRAPH.LEFT

# 太字・斜体の追加
p = doc.add_paragraph()
p.add_run('太字のテキスト').bold = True
p.add_run(' と ')
p.add_run('斜体のテキスト').italic = True

# 箇条書きの追加
doc.add_paragraph('項目1', style='List Bullet')
doc.add_paragraph('項目2', style='List Bullet')

# 番号付きリストの追加
doc.add_paragraph('手順1', style='List Number')
doc.add_paragraph('手順2', style='List Number')

# 表の追加
table = doc.add_table(rows=2, cols=3)
table.style = 'Light Grid Accent 1'
# セルにアクセス
cell = table.cell(0, 0)
cell.text = 'ヘッダー1'
```

3. **画像プレースホルダーの追加方法**
```python
# 画像プレースホルダーの追加（画像が添付された場合）

# 例1：文書冒頭のロゴプレースホルダー
p = doc.add_paragraph()
p.alignment = WD_ALIGN_PARAGRAPH.RIGHT
run = p.add_run('【画像挿入推奨位置】')
run.bold = True
run.font.color.rgb = RGBColor(0, 0, 255)
p.add_run('\nファイル: logo.png\n用途: 企業ロゴ\n推奨サイズ: 2インチ幅\n配置: ヘッダー右上')

# 枠線付きのプレースホルダー（表を使用）
table = doc.add_table(rows=1, cols=1)
table.style = 'Table Grid'
cell = table.cell(0, 0)
cell.text = '【画像プレースホルダー】\nファイル: main_image.jpg\n推奨サイズ: 幅4インチ\n用途: メインビジュアル\nキャプション: 製品の全体像'
# セルの背景色を設定（視認性向上）
from docx.oxml import parse_xml
from docx.oxml.ns import nsdecls
shading_elm = parse_xml(r'<w:shd {} w:fill="E0E0E0"/>'.format(nsdecls('w')))
cell._element.get_or_add_tcPr().append(shading_elm)

# 例2：本文中の説明図プレースホルダー
doc.add_paragraph()  # 空行
p = doc.add_paragraph('【図1 挿入位置】')
p.alignment = WD_ALIGN_PARAGRAPH.CENTER
run = p.runs[0]
run.bold = True
run.font.size = Pt(10)
run.font.color.rgb = RGBColor(100, 100, 100)

p = doc.add_paragraph('ファイル: diagram.png または flowchart.jpg')
p.alignment = WD_ALIGN_PARAGRAPH.CENTER
p.add_run('\n推奨: 幅5インチ、中央配置')
p.add_run('\n説明: プロセスフローを示す図表')
doc.add_paragraph()  # 空行
```

4. **必須の終了処理**（必ず含める）
```python
# BytesIOに保存
docx_io = BytesIO()
doc.save(docx_io)
docx_io.seek(0)

# Base64エンコード
docx_bytes = docx_io.read()
docx_base64 = base64.b64encode(docx_bytes).decode('utf-8')

# JavaScriptに渡す（この行は必須）
js.docx_base64_data = docx_base64

print("DOCX file has been generated in memory.")
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
    <title>Word文書のダウンロード</title>

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
            Word文書
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
      AIがここを書き換える: python-docxのコード
      重要: Pythonコードは左端から開始（インデントなし）
    -->
    <script type="text/python" id="python-code">
from docx import Document
from io import BytesIO
import base64
import js

# 文書の作成
doc = Document()

# タイトル
doc.add_heading('AIによって自動生成された文書', 0)

# 段落
doc.add_paragraph('このWord文書は動的に作成されました。')

# BytesIOに保存
docx_io = BytesIO()
doc.save(docx_io)
docx_io.seek(0)

# Base64エンコード
docx_bytes = docx_io.read()
docx_base64 = base64.b64encode(docx_bytes).decode('utf-8')

# JavaScriptに渡す（必須）
js.docx_base64_data = docx_base64

print("DOCX file has been generated in memory.")
    </script>

    <!-- 
      ⚠️⚠️⚠️ 警告: 以下のPyodideとpython-docxインストール部分は削除厳禁 ⚠️⚠️⚠️
      この部分を削除すると、Wordファイルの生成が不可能になります
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

                // 🔴🔴🔴 削除厳禁: python-docxライブラリのインストール 🔴🔴🔴
                // このコードブロックを削除するとModuleNotFoundErrorが発生します
                statusMessage.textContent = 'ライブラリをインストール中...';
                await pyodide.loadPackage("micropip");
                await pyodide.runPythonAsync(`
                    import micropip
                    await micropip.install('python-docx')  # Word文書生成に必須のライブラリ
                `);
                // 🔴🔴🔴 ここまで削除厳禁 🔴🔴🔴

                statusMessage.textContent = 'ファイルを生成しています...';
                const pythonCode = document.getElementById('python-code').textContent;
                // JavaScriptのグローバルスコープにデータを渡すための準備
                window.docx_base64_data = null; 
                await pyodide.runPythonAsync(pythonCode);
                const base64Data = window.docx_base64_data;

                if (!base64Data) {
                    throw new Error("Python code did not produce any data.");
                }

                statusMessage.textContent = '準備が完了しました！';
                loadingSpinner.style.display = 'none';
                downloadButton.disabled = false;
                downloadButton.addEventListener('click', () => downloadFile(base64Data, 'document.docx'));

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
                const blob = new Blob([byteArray], { type: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document' });

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

### **ModuleNotFoundError: No module named 'docx'**
**原因**: micropipでpython-docxをインストールする処理が削除または省略されている  
**対処法**: HTMLテンプレートの中の `await micropip.install('python-docx')` の部分を復元する

### **IndentationError: unexpected indent**
**原因**: Pythonコードに不要なインデントが含まれている  
**対処法**: すべてのPythonコードを左端から開始する

### **NameError: name 'js' is not defined**
**原因**: `import js` が抜けている  
**対処法**: 必須インポート文をすべて含める

### **AttributeError: 'Document' object has no attribute ...**
**原因**: python-docxのAPIを間違って使用している  
**対処法**: 正しいメソッド名を確認（add_heading, add_paragraph, add_table等）

## **💡 実装例：画像プレースホルダー付き文書**

```python
from docx import Document
from docx.shared import Pt, Inches, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.oxml import parse_xml
from docx.oxml.ns import nsdecls
from io import BytesIO
import base64
import js

doc = Document()

# ヘッダー部分（ロゴプレースホルダー）
p = doc.add_paragraph()
p.alignment = WD_ALIGN_PARAGRAPH.RIGHT
run = p.add_run('【ロゴ画像挿入位置】')
run.bold = True
run.font.size = Pt(8)
run.font.color.rgb = RGBColor(150, 150, 150)

# メインタイトル
title = doc.add_heading('業務報告書', 0)
title.alignment = WD_ALIGN_PARAGRAPH.CENTER

# サブタイトル
doc.add_heading('2025年1月 月次レポート', level=1)

# カバー画像プレースホルダー
table = doc.add_table(rows=1, cols=1)
table.style = 'Table Grid'
cell = table.cell(0, 0)
cell.text = ('【カバー画像プレースホルダー】\n'
             'ファイル: cover_image.jpg\n'
             '推奨サイズ: 幅6インチ\n'
             '用途: レポートのメインビジュアル\n'
             'キャプション: 2025年1月の成果')
# 背景色設定
shading_elm = parse_xml(r'<w:shd {} w:fill="F0F0F0"/>'.format(nsdecls('w')))
cell._element.get_or_add_tcPr().append(shading_elm)

doc.add_page_break()

# 概要セクション
doc.add_heading('1. 概要', level=2)
p = doc.add_paragraph('今月の業務について、以下の通り報告いたします。')

# 詳細セクション
doc.add_heading('2. 実施内容', level=2)
doc.add_paragraph('プロジェクトA: 完了', style='List Bullet')
doc.add_paragraph('プロジェクトB: 進行中（80%）', style='List Bullet')
doc.add_paragraph('プロジェクトC: 計画段階', style='List Bullet')

# グラフプレースホルダー
doc.add_paragraph()
p = doc.add_paragraph('【図1: 進捗グラフ挿入位置】')
p.alignment = WD_ALIGN_PARAGRAPH.CENTER
run = p.runs[0]
run.bold = True
run.font.color.rgb = RGBColor(0, 100, 200)

table = doc.add_table(rows=1, cols=1)
table.style = 'Table Grid'
table.alignment = WD_ALIGN_PARAGRAPH.CENTER
cell = table.cell(0, 0)
cell.text = ('progress_chart.png または progress_graph.xlsx のグラフ\n'
             '推奨: 幅5インチ\n'
             'Excelで作成したグラフを画像として挿入')
# 薄い青の背景
shading_elm = parse_xml(r'<w:shd {} w:fill="E6F2FF"/>'.format(nsdecls('w')))
cell._element.get_or_add_tcPr().append(shading_elm)

doc.add_paragraph()

# 表の追加
doc.add_heading('3. 進捗状況', level=2)
table = doc.add_table(rows=4, cols=3)
table.style = 'Light Grid Accent 1'

# ヘッダー行
hdr_cells = table.rows[0].cells
hdr_cells[0].text = 'プロジェクト'
hdr_cells[1].text = '進捗率'
hdr_cells[2].text = 'ステータス'

# データ行
data = [
    ('プロジェクトA', '100%', '完了'),
    ('プロジェクトB', '80%', '進行中'),
    ('プロジェクトC', '20%', '計画中')
]

for i, (proj, prog, status) in enumerate(data, 1):
    row_cells = table.rows[i].cells
    row_cells[0].text = proj
    row_cells[1].text = prog
    row_cells[2].text = status

# 写真ギャラリープレースホルダー
doc.add_heading('4. 活動記録', level=2)
p = doc.add_paragraph('今月の主な活動を写真で振り返ります。')

# 2x2のグリッドで写真プレースホルダー
table = doc.add_table(rows=2, cols=2)
table.style = 'Table Grid'
for i in range(2):
    for j in range(2):
        cell = table.cell(i, j)
        num = i * 2 + j + 1
        cell.text = f'【写真{num}】\nphoto{num}.jpg\n3インチ四方'

# 備考
doc.add_heading('5. 備考', level=2)
p = doc.add_paragraph()
p.add_run('重要: ').bold = True
p.add_run('来月までに全プロジェクトの完了を目指します。')

# サイドバー画像プレースホルダー（小さなアイコン的使用）
doc.add_paragraph()
p = doc.add_paragraph('【参考資料アイコン】')
run = p.runs[0]
run.font.size = Pt(9)
run.font.color.rgb = RGBColor(128, 128, 128)
p.add_run('\nicon_document.png (1インチ) - 関連文書へのリンク用アイコン')

# 保存とBase64変換
docx_io = BytesIO()
doc.save(docx_io)
docx_io.seek(0)

docx_bytes = docx_io.read()
docx_base64 = base64.b64encode(docx_bytes).decode('utf-8')

js.docx_base64_data = docx_base64

print("DOCX file with image placeholders has been generated.")
```

## **📚 python-docxの主要機能リファレンス**

### **文書構造**
- `Document()`: 新規文書の作成
- `add_heading(text, level)`: 見出しの追加（level 0-9）
- `add_paragraph(text, style)`: 段落の追加
- `add_page_break()`: 改ページの挿入

### **テキスト書式**
- `run.bold = True`: 太字
- `run.italic = True`: 斜体
- `run.underline = True`: 下線
- `run.font.size = Pt(12)`: フォントサイズ
- `run.font.color.rgb = RGBColor(255, 0, 0)`: 文字色

### **段落配置**
- `WD_ALIGN_PARAGRAPH.LEFT`: 左寄せ
- `WD_ALIGN_PARAGRAPH.CENTER`: 中央寄せ
- `WD_ALIGN_PARAGRAPH.RIGHT`: 右寄せ
- `WD_ALIGN_PARAGRAPH.JUSTIFY`: 両端揃え

### **画像プレースホルダーの配置戦略**

#### **文書タイプ別の推奨位置**

1. **レポート・報告書**
   - ロゴ: ヘッダー右上
   - カバー画像: タイトル下
   - グラフ・図表: 該当セクション内
   - 写真: ギャラリー形式または本文中

2. **契約書・公式文書**
   - 社印・ロゴ: ヘッダーまたはフッター
   - 署名欄: 文書末尾
   - 参考図: 付録セクション

3. **マニュアル・手順書**
   - スクリーンショット: 手順説明の直後
   - 図解: 概念説明の箇所
   - アイコン: 注意事項の横

4. **提案書**
   - メインビジュアル: 表紙
   - 製品画像: 製品説明セクション
   - 比較表: データセクション

### **プレースホルダーのスタイル設定**

```python
# 色付き背景のプレースホルダー
from docx.oxml import parse_xml
from docx.oxml.ns import nsdecls

table = doc.add_table(rows=1, cols=1)
cell = table.cell(0, 0)
cell.text = "プレースホルダーテキスト"

# 背景色の設定
shading_elm = parse_xml(r'<w:shd {} w:fill="COLOR_HEX"/>'.format(nsdecls('w')))
cell._element.get_or_add_tcPr().append(shading_elm)

# 色の例:
# "E0E0E0" - 薄いグレー
# "E6F2FF" - 薄い青
# "FFF0E6" - 薄いオレンジ
# "E6FFE6" - 薄い緑
```

## **✅ 最終チェックリスト**

AIがHTMLを生成する際の確認事項：

- [ ] **python-docxのインストール処理（`await micropip.install('python-docx')`）が含まれているか**
- [ ] Pythonコードは左端から開始されているか
- [ ] 必須のimport文がすべて含まれているか
- [ ] `js.docx_base64_data = docx_base64` の行が存在するか
- [ ] 画像が添付された場合、適切なプレースホルダーを追加しているか
- [ ] プレースホルダーに必要な情報（ファイル名、用途、サイズ、キャプション）が含まれているか
- [ ] ページタイトルは適切に変更されているか
- [ ] MIMEタイプが正しく設定されているか
- [ ] エラーメッセージの表示処理が含まれているか
- [ ] Pyodideのロード処理が含まれているか

## **🎯 応用例**

### **インライン画像プレースホルダー**
```python
# 文章の途中に画像を挿入する場合
p = doc.add_paragraph('製品の特徴について説明します。')
p.add_run(' [画像: product_detail.jpg, 2インチ幅] ')
p.add_run('このように、細部まで丁寧に作られています。')
```

### **サイドバイサイド配置**
```python
# 表を使って文章と画像を並列配置
table = doc.add_table(rows=1, cols=2)
# 左側: テキスト
cell_left = table.cell(0, 0)
cell_left.text = '説明文がここに入ります。'
# 右側: 画像プレースホルダー
cell_right = table.cell(0, 1)
cell_right.text = '【画像】\nside_image.jpg\n幅2.5インチ'
```

### **条件付きプレースホルダー**
```python
# 文書の内容に応じて動的にプレースホルダーを配置
if "データ分析" in content:
    # データビジュアライゼーション用プレースホルダー
    add_chart_placeholder(doc)

if "手順説明" in content:
    # スクリーンショット用プレースホルダー
    add_screenshot_placeholders(doc)
```

## **📝 更新履歴**

- **v2.1** (2025-01-17): 外部ライブラリインストールの重要性を強調
  - 必須要件セクションを冒頭に追加
  - HTMLテンプレート内にコメントで警告を追加
  - チェックリストにpython-docx確認項目を追加
  - エラー対処法にModuleNotFoundErrorを追加
- **v2.0** (2025-01-17): 画像プレースホルダー機能を追加
  - 画像配置のサジェスチョン機能
  - プレースホルダーの視覚的表現（表と背景色）
  - 画像選択の自動判定基準
  - キャプション提案機能
- **v1.0** (2025-01-17): 初版リリース

## **📄 ライセンス**

本レシピはMITライセンスの下で提供されています。自由に使用、改変、再配布、商用利用が可能です。

## **🆘 トラブルシューティング**

もしAIがこのレシピを正しく実行できない場合：

1. **最初に必須要件セクションを確認** - 特にpython-docxのインストール処理
2. **HTMLテンプレートの警告コメントを確認** - 削除厳禁の箇所が残っているか
3. **チェックリストを使用して検証** - すべての項目が満たされているか
4. **エラーメッセージを確認** - ModuleNotFoundErrorの場合はライブラリインストールの問題

**それでも問題が解決しない場合は、このレシピの最新版を確認してください。**