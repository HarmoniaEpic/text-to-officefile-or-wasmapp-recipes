# **PDF自動生成ページ レシピ v1.2**

このドキュメントは、生成AIがユーザーの指示に基づき、「ワンクリック・ダウンロードページ」を自動生成するためのレシピ（指示書＋テンプレート）です。

AIはこのレシピに従い、最終的な成果物として単一のHTMLファイルを出力します。

## **🔴 必須要件 - 必ず最初に確認してください 🔴**

### **1. 外部ライブラリのインストール（絶対に省略禁止）**

**⚠️ 最重要事項：reportlabライブラリのインストール処理は絶対に削除・省略してはいけません**

HTMLテンプレート内の以下のコードブロックは**必須**です：
```javascript
// このコードブロックは削除厳禁
await pyodide.loadPackage("micropip");
await pyodide.runPythonAsync(`
    import micropip
    await micropip.install('reportlab')  # ← これがないとPDFファイルを生成できません
`);
```

**なぜ必要か？**
- reportlabはPDFファイル（.pdf）を生成するための外部ライブラリです
- ブラウザ環境では標準では利用できないため、micropipでインストールが必要です
- このインストール処理を削除すると、`ModuleNotFoundError: No module named 'reportlab'`エラーが発生します

### **2. Pythonコードのインデント規則（エラー防止のため厳守）**

**⚠️ IndentationErrorを防ぐため、以下の規則を必ず守ってください**

- **scriptタグ内のPythonコードは、絶対に左端から開始すること**
- HTMLのインデントに影響されてPythonコードにインデントを追加してはいけない
- `<script type="text/python" id="python-code">` の直後の行から、インデントなしでPythonコードを記述する

**正しい例：**
```html
    <script type="text/python" id="python-code">
from reportlab.pdfgen import canvas  # ← 左端から開始（正しい）
import base64  # ← インデントなし（正しい）
```

**間違った例：**
```html
    <script type="text/python" id="python-code">
        from reportlab.pdfgen import canvas  # ← ❌ 不要なインデント（エラーの原因）
        import base64  # ← ❌ IndentationErrorが発生
```

## **📋 AIへの実装指示**

以下のHTML骨子テンプレートを使用し、指定された2箇所のみをユーザーの要求に応じて書き換えてください。**それ以外の部分（特にライブラリインストール部分）は一切変更してはいけません。**

### **書き換える箇所（2箇所のみ）**

1. **ページのタイトルと見出しを書き換える**  
   * `<!-- AIがここを書き換える: ページのタイトル -->` とコメントされている2箇所を、生成するPDF文書の内容に合わせた具体的なタイトル（例: 請求書のダウンロード、証明書のダウンロード）に書き換えてください。

2. **Pythonコードを書き換える**  
   * `<script type="text/python" id="python-code">` タグの内部を、要求されたPDF文書を生成するための ReportLab コードに完全に置き換えてください。
   * **重要**: Pythonコードは必ず左端から開始し、HTMLのインデントを無視してください
   * このPythonコードは、必ず**最終行で `js.pdf_base64_data` にBase64文字列を代入する**ルールに従う必要があります。

### **Pythonコード作成のルール**

1. **必須のインポート文**（左端から記述）
```python
from reportlab.lib.pagesizes import letter, A4
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib.units import inch, mm
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle, PageBreak, Image
from reportlab.lib import colors
from reportlab.lib.enums import TA_CENTER, TA_RIGHT, TA_LEFT, TA_JUSTIFY
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont
from reportlab.pdfbase.cidfonts import UnicodeCIDFont
from io import BytesIO
import base64
import js
```

2. **PDF作成の基本構造**
```python
# BytesIOオブジェクトの作成
pdf_buffer = BytesIO()

# PDFドキュメントの作成
doc = SimpleDocTemplate(
    pdf_buffer,
    pagesize=A4,
    rightMargin=72,
    leftMargin=72,
    topMargin=72,
    bottomMargin=18,
)

# スタイルの取得
styles = getSampleStyleSheet()

# 日本語フォントの設定（CIDフォント使用）
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiKakuGo-W5'))
styles['Normal'].fontName = 'HeiseiKakuGo-W5'
styles['Title'].fontName = 'HeiseiKakuGo-W5'
styles['Heading1'].fontName = 'HeiseiKakuGo-W5'

# コンテンツリスト
story = []

# タイトルの追加
title = Paragraph("タイトルテキスト", styles['Title'])
story.append(title)
story.append(Spacer(1, 12))

# 段落の追加
text = Paragraph("本文テキスト", styles['Normal'])
story.append(text)
story.append(Spacer(1, 12))

# 表の追加
data = [
    ['項目1', '項目2', '項目3'],
    ['データ1', 'データ2', 'データ3'],
]
table = Table(data)
table.setStyle(TableStyle([
    ('BACKGROUND', (0, 0), (-1, 0), colors.grey),
    ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),
    ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
    ('FONTNAME', (0, 0), (-1, -1), 'HeiseiKakuGo-W5'),
    ('FONTSIZE', (0, 0), (-1, 0), 14),
    ('BOTTOMPADDING', (0, 0), (-1, 0), 12),
    ('BACKGROUND', (0, 1), (-1, -1), colors.beige),
    ('GRID', (0, 0), (-1, -1), 1, colors.black),
]))
story.append(table)

# PDFの構築
doc.build(story)
```

3. **必須の終了処理**（必ず含める）
```python
# BytesIOからデータを取得
pdf_buffer.seek(0)
pdf_bytes = pdf_buffer.read()

# Base64エンコード
pdf_base64 = base64.b64encode(pdf_bytes).decode('utf-8')

# JavaScriptに渡す（この行は必須）
js.pdf_base64_data = pdf_base64

print("PDF file has been generated in memory.")
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
    <title>PDF文書のダウンロード</title>

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
            PDF文書
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
      AIがここを書き換える: ReportLabのコード
      重要: Pythonコードは左端から開始（インデントなし）
    -->
    <script type="text/python" id="python-code">
from reportlab.lib.pagesizes import A4
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.cidfonts import UnicodeCIDFont
from io import BytesIO
import base64
import js

# BytesIOオブジェクトの作成
pdf_buffer = BytesIO()

# PDFドキュメントの作成
doc = SimpleDocTemplate(pdf_buffer, pagesize=A4)

# 日本語フォントの登録
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiKakuGo-W5'))

# スタイルの取得と設定
styles = getSampleStyleSheet()
styles['Normal'].fontName = 'HeiseiKakuGo-W5'
styles['Title'].fontName = 'HeiseiKakuGo-W5'

# コンテンツ
story = []
title = Paragraph("AIによって自動生成されたPDF", styles['Title'])
story.append(title)
story.append(Spacer(1, 12))

text = Paragraph("このPDF文書は動的に作成されました。", styles['Normal'])
story.append(text)

# PDFの構築
doc.build(story)

# Base64エンコード
pdf_buffer.seek(0)
pdf_bytes = pdf_buffer.read()
pdf_base64 = base64.b64encode(pdf_bytes).decode('utf-8')

# JavaScriptに渡す（必須）
js.pdf_base64_data = pdf_base64

print("PDF file has been generated in memory.")
    </script>

    <!-- 
      ⚠️⚠️⚠️ 警告: 以下のPyodideとreportlabインストール部分は削除厳禁 ⚠️⚠️⚠️
      この部分を削除すると、PDFファイルの生成が不可能になります
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

                // 🔴🔴🔴 削除厳禁: reportlabライブラリのインストール 🔴🔴🔴
                // このコードブロックを削除するとModuleNotFoundErrorが発生します
                statusMessage.textContent = 'ライブラリをインストール中...';
                await pyodide.loadPackage("micropip");
                await pyodide.runPythonAsync(`
                    import micropip
                    await micropip.install('reportlab')  # PDF生成に必須のライブラリ
                `);
                // 🔴🔴🔴 ここまで削除厳禁 🔴🔴🔴

                statusMessage.textContent = 'ファイルを生成しています...';
                const pythonCode = document.getElementById('python-code').textContent;
                // JavaScriptのグローバルスコープにデータを渡すための準備
                window.pdf_base64_data = null; 
                await pyodide.runPythonAsync(pythonCode);
                const base64Data = window.pdf_base64_data;

                if (!base64Data) {
                    throw new Error("Python code did not produce any data.");
                }

                statusMessage.textContent = '準備が完了しました！';
                loadingSpinner.style.display = 'none';
                downloadButton.disabled = false;
                downloadButton.addEventListener('click', () => downloadFile(base64Data, 'document.pdf'));

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
                const blob = new Blob([byteArray], { type: 'application/pdf' });

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

### **ModuleNotFoundError: No module named 'reportlab'**
**原因**: micropipでreportlabをインストールする処理が削除または省略されている  
**対処法**: HTMLテンプレートの中の `await micropip.install('reportlab')` の部分を復元する

### **IndentationError: unexpected indent**
**原因**: Pythonコードに不要なインデントが含まれている  
**対処法**: すべてのPythonコードを左端から開始する

### **NameError: name 'js' is not defined**
**原因**: `import js` が抜けている  
**対処法**: 必須インポート文をすべて含める

### **TypeError: ... got an unexpected keyword argument 'onPage'**
**原因**: onPage引数をdoc.build()メソッドに誤って渡している  
**対処法**: onPageはSimpleDocTemplateの**作成時**に指定する（セクション9参照）

### **Font '...' not found**
**原因**: 日本語フォントが正しく登録されていない  
**対処法**: CIDフォントを使用するか、フォントファイルを埋め込む

### **UnicodeDecodeError**
**原因**: 日本語テキストのエンコーディング問題  
**対処法**: UTF-8エンコーディングを確実に使用

## **💡 実装例：請求書の作成**

```python
from reportlab.lib.pagesizes import A4
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle, Paragraph, Spacer
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib import colors
from reportlab.lib.units import mm
from reportlab.lib.enums import TA_RIGHT, TA_CENTER
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.cidfonts import UnicodeCIDFont
from io import BytesIO
import base64
import js
from datetime import date

# BytesIOオブジェクトの作成
pdf_buffer = BytesIO()

# PDFドキュメントの作成
doc = SimpleDocTemplate(
    pdf_buffer,
    pagesize=A4,
    rightMargin=20*mm,
    leftMargin=20*mm,
    topMargin=20*mm,
    bottomMargin=20*mm
)

# 日本語フォントの登録
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiKakuGo-W5'))

# スタイルの設定
styles = getSampleStyleSheet()
styles['Normal'].fontName = 'HeiseiKakuGo-W5'
styles['Title'].fontName = 'HeiseiKakuGo-W5'
styles['Title'].fontSize = 24
styles['Title'].alignment = TA_CENTER

# 右寄せスタイルの作成
right_style = ParagraphStyle(
    'RightAlign',
    parent=styles['Normal'],
    fontName='HeiseiKakuGo-W5',
    alignment=TA_RIGHT,
)

# コンテンツリスト
story = []

# タイトル
title = Paragraph("請求書", styles['Title'])
story.append(title)
story.append(Spacer(1, 20))

# 日付
date_text = Paragraph(f"発行日: {date.today().strftime('%Y年%m月%d日')}", right_style)
story.append(date_text)
story.append(Spacer(1, 20))

# 宛先
story.append(Paragraph("〇〇株式会社 御中", styles['Normal']))
story.append(Spacer(1, 10))
story.append(Paragraph("経理部", styles['Normal']))
story.append(Spacer(1, 20))

# 請求内容の表
data = [
    ['品目', '数量', '単価', '金額'],
    ['商品A', '10', '¥1,000', '¥10,000'],
    ['商品B', '5', '¥2,000', '¥10,000'],
    ['商品C', '3', '¥3,000', '¥9,000'],
    ['', '', '小計', '¥29,000'],
    ['', '', '消費税(10%)', '¥2,900'],
    ['', '', '合計', '¥31,900'],
]

# 表のスタイル設定
table = Table(data, colWidths=[80*mm, 30*mm, 30*mm, 30*mm])
table.setStyle(TableStyle([
    # ヘッダー行
    ('BACKGROUND', (0, 0), (-1, 0), colors.grey),
    ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),
    ('FONTNAME', (0, 0), (-1, -1), 'HeiseiKakuGo-W5'),
    ('FONTSIZE', (0, 0), (-1, 0), 12),
    ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
    
    # データ行
    ('BACKGROUND', (0, 1), (-1, -4), colors.beige),
    ('GRID', (0, 0), (-1, -4), 1, colors.black),
    
    # 合計行
    ('BACKGROUND', (2, -3), (-1, -1), colors.lightgrey),
    ('FONTSIZE', (2, -1), (-1, -1), 14),
    ('TEXTCOLOR', (2, -1), (-1, -1), colors.red),
    
    # 右寄せ
    ('ALIGN', (1, 1), (-1, -1), 'RIGHT'),
]))

story.append(table)
story.append(Spacer(1, 30))

# 振込先情報
story.append(Paragraph("【振込先】", styles['Heading2']))
story.append(Paragraph("〇〇銀行 △△支店", styles['Normal']))
story.append(Paragraph("普通預金 1234567", styles['Normal']))
story.append(Paragraph("カ）サンプルカイシャ", styles['Normal']))

# PDFの構築
doc.build(story)

# Base64エンコード
pdf_buffer.seek(0)
pdf_bytes = pdf_buffer.read()
pdf_base64 = base64.b64encode(pdf_bytes).decode('utf-8')

# JavaScriptに渡す
js.pdf_base64_data = pdf_base64

print("Invoice PDF has been generated in memory.")
```

## **📚 ReportLabの主要機能リファレンス**

### **ページ設定**
```python
from reportlab.lib.pagesizes import A4, A3, letter, legal
from reportlab.lib.units import inch, cm, mm

# カスタムページサイズ
custom_pagesize = (210*mm, 297*mm)  # A4サイズ

# マージン設定
doc = SimpleDocTemplate(
    pdf_buffer,
    pagesize=A4,
    rightMargin=72,   # 1インチ
    leftMargin=72,
    topMargin=72,
    bottomMargin=18,
)
```

### **テキストスタイル**
```python
from reportlab.lib.styles import ParagraphStyle
from reportlab.lib.enums import TA_LEFT, TA_CENTER, TA_RIGHT, TA_JUSTIFY

# カスタムスタイル作成
custom_style = ParagraphStyle(
    'CustomStyle',
    fontName='HeiseiKakuGo-W5',
    fontSize=12,
    leading=14,
    alignment=TA_LEFT,
    textColor=colors.black,
    spaceAfter=6,
)
```

### **表（テーブル）**
```python
from reportlab.platypus import Table, TableStyle

# 表の作成
data = [['A', 'B'], ['C', 'D']]
table = Table(data)

# 表のスタイル設定
table.setStyle(TableStyle([
    ('BACKGROUND', (0, 0), (-1, 0), colors.grey),
    ('GRID', (0, 0), (-1, -1), 1, colors.black),
    ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
    ('VALIGN', (0, 0), (-1, -1), 'MIDDLE'),
]))
```

### **画像の挿入**
```python
from reportlab.platypus import Image

# 画像の追加（Base64データから）
import io
from PIL import Image as PILImage

# Base64画像データをPDFに追加
img_data = base64.b64decode(base64_image_string)
img = Image(io.BytesIO(img_data), width=100, height=100)
story.append(img)
```

### **改ページとスペーサー**
```python
from reportlab.platypus import PageBreak, Spacer

# 改ページ
story.append(PageBreak())

# スペース（1インチの空白）
story.append(Spacer(1, 1*inch))
```

### **線と図形**
```python
from reportlab.platypus import HRFlowable
from reportlab.lib.colors import black

# 水平線
story.append(HRFlowable(width="100%", thickness=1, color=black))
```

## **🌏 日本語対応**

### **CIDフォントの使用（推奨）**
```python
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.cidfonts import UnicodeCIDFont

# 日本語CIDフォントの登録
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiKakuGo-W5'))  # ゴシック体
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiMin-W3'))      # 明朝体

# スタイルに適用
styles['Normal'].fontName = 'HeiseiKakuGo-W5'
```

### **利用可能な日本語CIDフォント**
- `HeiseiKakuGo-W5`: 平成角ゴシック
- `HeiseiMin-W3`: 平成明朝
- `HeiseiMin-W7`: 平成明朝（太字）

## **✅ 最終チェックリスト**

AIがHTMLを生成する際の確認事項：

- [ ] **reportlabのインストール処理（`await micropip.install('reportlab')`）が含まれているか**
- [ ] Pythonコードは左端から開始されているか
- [ ] 必須のimport文がすべて含まれているか
- [ ] `js.pdf_base64_data = pdf_base64` の行が存在するか
- [ ] 日本語フォントが正しく設定されているか
- [ ] ページタイトルは適切に変更されているか
- [ ] MIMEタイプが正しく設定されているか（application/pdf）
- [ ] エラーメッセージの表示処理が含まれているか
- [ ] Pyodideのロード処理が含まれているか

## **🎯 応用例**

### **証明書テンプレート**
```python
# 証明書のタイトル
title_style = ParagraphStyle(
    'CertTitle',
    fontName='HeiseiMin-W7',
    fontSize=28,
    alignment=TA_CENTER,
    spaceAfter=30
)

story.append(Paragraph("修了証書", title_style))
story.append(Spacer(1, 30))
story.append(Paragraph("氏名: 山田太郎 様", styles['Normal']))
story.append(Spacer(1, 20))
story.append(Paragraph("あなたは本講座を優秀な成績で修了されたことを証明いたします。", styles['Normal']))
```

### **レポートテンプレート**
```python
# 目次の作成
toc_data = [
    ['目次', ''],
    ['1. はじめに', '1'],
    ['2. 調査方法', '3'],
    ['3. 結果', '5'],
    ['4. 考察', '10'],
    ['5. まとめ', '15'],
]

toc_table = Table(toc_data, colWidths=[150*mm, 20*mm])
story.append(toc_table)
story.append(PageBreak())
```

### **グラフの埋め込み**
```python
# matplotlibでグラフを作成してPDFに埋め込む
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
from io import BytesIO

# グラフの作成
fig, ax = plt.subplots()
ax.plot([1, 2, 3, 4], [1, 4, 2, 3])

# BytesIOに保存
img_buffer = BytesIO()
plt.savefig(img_buffer, format='png')
img_buffer.seek(0)

# PDFに追加
from reportlab.platypus import Image
img = Image(img_buffer, width=400, height=300)
story.append(img)
```

## **🔧 高度な機能**

### **ヘッダーとフッター**

各ページに共通のヘッダーやフッター（ページ番号など）を追加するには、SimpleDocTemplateの**初期化時**にonPage引数を使用します。onPageには、ページが描画されるたびに呼び出される関数を指定します。

**⚠️ 重要:** onPageはSimpleDocTemplate(...)のコンストラクタ引数です。doc.build(...)の引数ではないため、間違えないように注意してください。

```python
from reportlab.platypus import SimpleDocTemplate
from reportlab.lib.pagesizes import A4

# 各ページで呼び出される描画関数
def header_footer(canvas, doc):
    canvas.saveState()
    canvas.setFont('HeiseiKakuGo-W5', 9)
    # ヘッダー
    canvas.drawString(72, A4[1] - 40, "ヘッダーテキスト")
    # フッター（ページ番号）
    canvas.drawString(72, 40, f"ページ {doc.page}")
    canvas.restoreState()

# --- 正しい例 ---
# SimpleDocTemplateの作成時に onPage を渡す
doc = SimpleDocTemplate(
    pdf_buffer,
    pagesize=A4,
    onPage=header_footer  # <-- ✅ 正しい指定方法
)
# buildメソッドには渡さない
doc.build(story)

# --- 間違った例 ---
# doc = SimpleDocTemplate(pdf_buffer, pagesize=A4)  # ここで onPage を指定しない
# doc.build(story, onPage=header_footer)  # <-- ❌ TypeErrorの原因
```

### **複数カラムレイアウト**
```python
from reportlab.platypus import BaseDocTemplate, Frame, PageTemplate

# 2カラムレイアウトの作成
frame1 = Frame(doc.leftMargin, doc.bottomMargin, 
               doc.width/2-6, doc.height, id='col1')
frame2 = Frame(doc.leftMargin+doc.width/2+6, doc.bottomMargin,
               doc.width/2-6, doc.height, id='col2')

doc.addPageTemplates([PageTemplate(id='TwoCol', frames=[frame1, frame2])])
```

## **📝 更新履歴**

- **v1.2** (2025-01-17): 外部ライブラリインストールの重要性を強調
  - 必須要件セクションを冒頭に追加
  - HTMLテンプレート内にコメントで警告を追加
  - チェックリストにreportlab確認項目を追加
  - エラー対処法にModuleNotFoundErrorを追加
- **v1.1** (2025-09-12): ヘッダー/フッター機能の記述を強化
  - onPage引数の正しい使い方と間違った使い方を明記
  - TypeErrorを「よくあるエラー」に追加
- **v1.0** (2025-01-17): 初版リリース
  - ReportLabによるPDF生成機能の実装
  - 日本語CIDフォント対応
  - インデントエラー防止機能を継承

## **📄 ライセンス**

本レシピはMITライセンスの下で提供されています。自由に使用、改変、再配布、商用利用が可能です。

## **🆘 トラブルシューティング**

もしAIがこのレシピを正しく実行できない場合：

1. **最初に必須要件セクションを確認** - 特にreportlabのインストール処理
2. **HTMLテンプレートの警告コメントを確認** - 削除厳禁の箇所が残っているか
3. **チェックリストを使用して検証** - すべての項目が満たされているか
4. **エラーメッセージを確認** - ModuleNotFoundErrorの場合はライブラリインストールの問題

**それでも問題が解決しない場合は、このレシピの最新版を確認してください。**