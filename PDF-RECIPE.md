# **PDF自動生成ページ レシピ v2.0**
**AIによるワンクリック・ダウンロードページ生成の完全ガイド**

---

## **📌 このレシピについて**

生成AIがユーザーの指示に基づき、PDFファイルをブラウザ上で生成・ダウンロードできる**単一HTMLファイル**を自動作成するための完全な指示書です。

### **v2.0の特徴**
- 🎯 実装ルールを優先順位別に明確化
- 🔍 エラー対処の手順と実例を完備
- 💡 段階的な実装例で学習しやすい
- 📑 ビジネス文書テンプレート5種類以上
- 🔒 セキュリティ機能（パスワード保護等）対応

---

## **🚨 実装ルール（優先順位順）**

### **🔴 レベル1: 絶対禁止事項（変更・削除厳禁）**

以下の要素は**一切変更してはいけません**：

#### **1.1 reportlabライブラリのインストール処理**
```javascript
await pyodide.loadPackage("micropip");
await pyodide.runPythonAsync(`
    import micropip
    await micropip.install('reportlab')
`);
```

**なぜ絶対に必要か：**
- reportlabはPDFファイル生成のための外部ライブラリ
- ブラウザのPyodide環境には標準で含まれていない
- micropipを使用してランタイムでインストールする必要がある
- **削除すると必ず発生するエラー**: `ModuleNotFoundError: No module named 'reportlab'`

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
from reportlab.pdfgen import canvas
import base64

# ❌ 間違い（インデントあり）
    from reportlab.pdfgen import canvas  # IndentationError発生
    import base64
```

**理由**: HTMLのインデントに影響されてPythonコードにインデントを追加するとエラーになる

#### **2.2 必須のインポート文**
```python
from reportlab.lib.pagesizes import A4
from io import BytesIO
import base64
import js  # JavaScriptとの連携に必須
```

**省略した場合のエラー**: `NameError: name 'js' is not defined`

#### **2.3 Base64データの受け渡し**
Pythonコードの最終部で必ず実行：
```python
js.pdf_base64_data = pdf_base64
```

**理由**: JavaScriptがPDFデータを受け取るための必須処理

### **🟢 レベル3: カスタマイズ可能要素**

以下は自由に変更できます：
- ページタイトル（`<title>`タグと`<h1>`タグ）
- Pythonコード内のPDF内容
- PDFのレイアウト・デザイン
- ダウンロードファイル名
- ページサイズや余白

---

## **📝 HTMLテンプレート（完全版）**

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- カスタマイズ可能: ページタイトル -->
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

    <!-- カスタマイズ可能: Pythonコード（ルールに従って） -->
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

                // reportlabのインストール（削除禁止）
                statusMessage.textContent = 'ライブラリをインストール中...';
                await pyodide.loadPackage("micropip");
                await pyodide.runPythonAsync(`
                    import micropip
                    await micropip.install('reportlab')
                `);

                statusMessage.textContent = 'ファイルを生成しています...';
                const pythonCode = document.getElementById('python-code').textContent;
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

---

## **🔧 エラー診断ガイド**

### **よくあるエラーと解決方法**

#### **1. ModuleNotFoundError**
```
ModuleNotFoundError: No module named 'reportlab'
```
**原因**: reportlabのインストール処理が削除されている
**解決**: `await micropip.install('reportlab')` を復元

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

#### **4. TypeError: onPage**
```
TypeError: build() got an unexpected keyword argument 'onPage'
```
**原因**: onPage引数をdoc.build()に誤って渡している
**解決**: onPageはSimpleDocTemplateの作成時に指定

#### **5. Font Error**
```
Font '...' not found
```
**原因**: 日本語フォントが正しく登録されていない
**解決**: CIDフォントを使用（HeiseiKakuGo-W5等）

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
from reportlab.lib.pagesizes import A4
from reportlab.platypus import SimpleDocTemplate, Paragraph
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.cidfonts import UnicodeCIDFont
from io import BytesIO
import base64
import js

pdf_buffer = BytesIO()
doc = SimpleDocTemplate(pdf_buffer, pagesize=A4)

# 日本語フォント
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiKakuGo-W5'))
styles = getSampleStyleSheet()
styles['Normal'].fontName = 'HeiseiKakuGo-W5'

# 単純な文書
story = []
story.append(Paragraph("シンプルなPDF文書", styles['Normal']))

doc.build(story)

pdf_buffer.seek(0)
pdf_base64 = base64.b64encode(pdf_buffer.read()).decode('utf-8')
js.pdf_base64_data = pdf_base64
```

### **レベル2: 基本的な書式付き文書**
```python
from reportlab.lib.pagesizes import A4
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib import colors
from reportlab.lib.units import mm
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.cidfonts import UnicodeCIDFont
from io import BytesIO
import base64
import js

pdf_buffer = BytesIO()
doc = SimpleDocTemplate(
    pdf_buffer,
    pagesize=A4,
    rightMargin=20*mm,
    leftMargin=20*mm,
    topMargin=20*mm,
    bottomMargin=20*mm
)

# 日本語フォント設定
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiKakuGo-W5'))
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiMin-W3'))

styles = getSampleStyleSheet()
styles['Normal'].fontName = 'HeiseiKakuGo-W5'
styles['Title'].fontName = 'HeiseiMin-W3'
styles['Title'].fontSize = 20
styles['Heading1'].fontName = 'HeiseiKakuGo-W5'
styles['Heading1'].fontSize = 16

story = []

# タイトル
story.append(Paragraph("業務報告書", styles['Title']))
story.append(Spacer(1, 20))

# セクション1
story.append(Paragraph("1. 概要", styles['Heading1']))
story.append(Spacer(1, 10))
story.append(Paragraph("今月の業務について、以下の通り報告いたします。", styles['Normal']))
story.append(Spacer(1, 10))

# 箇条書き
bullet_items = [
    "• プロジェクトAの完了",
    "• 新規顧客3社の獲得",
    "• 売上目標の達成（120%）"
]
for item in bullet_items:
    story.append(Paragraph(item, styles['Normal']))
    story.append(Spacer(1, 5))

story.append(Spacer(1, 15))

# セクション2
story.append(Paragraph("2. 詳細データ", styles['Heading1']))
story.append(Spacer(1, 10))

# 表
data = [
    ['項目', '計画', '実績', '達成率'],
    ['売上', '1000万円', '1200万円', '120%'],
    ['顧客数', '10社', '13社', '130%'],
    ['案件数', '15件', '18件', '120%'],
]

table = Table(data, colWidths=[40*mm, 35*mm, 35*mm, 35*mm])
table.setStyle(TableStyle([
    ('BACKGROUND', (0, 0), (-1, 0), colors.grey),
    ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),
    ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
    ('FONTNAME', (0, 0), (-1, -1), 'HeiseiKakuGo-W5'),
    ('FONTSIZE', (0, 0), (-1, 0), 12),
    ('BOTTOMPADDING', (0, 0), (-1, 0), 12),
    ('BACKGROUND', (0, 1), (-1, -1), colors.beige),
    ('GRID', (0, 0), (-1, -1), 1, colors.black),
]))

story.append(table)

doc.build(story)

pdf_buffer.seek(0)
pdf_base64 = base64.b64encode(pdf_buffer.read()).decode('utf-8')
js.pdf_base64_data = pdf_base64
```

### **レベル3: ビジネス文書（請求書）**
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

pdf_buffer = BytesIO()
doc = SimpleDocTemplate(
    pdf_buffer,
    pagesize=A4,
    rightMargin=20*mm,
    leftMargin=20*mm,
    topMargin=20*mm,
    bottomMargin=20*mm
)

# 日本語フォント
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiKakuGo-W5'))

# スタイル設定
styles = getSampleStyleSheet()
styles['Normal'].fontName = 'HeiseiKakuGo-W5'
styles['Title'].fontName = 'HeiseiKakuGo-W5'
styles['Title'].fontSize = 24
styles['Title'].alignment = TA_CENTER

# 右寄せスタイル
right_style = ParagraphStyle(
    'RightAlign',
    parent=styles['Normal'],
    fontName='HeiseiKakuGo-W5',
    alignment=TA_RIGHT,
)

story = []

# タイトル
title = Paragraph("請求書", styles['Title'])
story.append(title)
story.append(Spacer(1, 20))

# 請求番号と日付
invoice_info = Table([
    ['請求番号: INV-2025-001', f'発行日: {date.today().strftime("%Y年%m月%d日")}']
], colWidths=[90*mm, 80*mm])
invoice_info.setStyle(TableStyle([
    ('FONTNAME', (0, 0), (-1, -1), 'HeiseiKakuGo-W5'),
    ('ALIGN', (0, 0), (0, 0), 'LEFT'),
    ('ALIGN', (1, 0), (1, 0), 'RIGHT'),
]))
story.append(invoice_info)
story.append(Spacer(1, 20))

# 宛先
story.append(Paragraph("〇〇株式会社 御中", styles['Normal']))
story.append(Paragraph("経理部", styles['Normal']))
story.append(Spacer(1, 20))

# 請求金額（強調）
total_style = ParagraphStyle(
    'TotalAmount',
    parent=styles['Normal'],
    fontName='HeiseiKakuGo-W5',
    fontSize=18,
    textColor=colors.red,
)
story.append(Paragraph("請求金額: ¥341,000（税込）", total_style))
story.append(Spacer(1, 20))

# 請求明細
data = [
    ['品目', '数量', '単価', '金額'],
    ['コンサルティング料', '20時間', '¥10,000', '¥200,000'],
    ['システム開発費', '一式', '¥100,000', '¥100,000'],
    ['保守サポート費', '1ヶ月', '¥10,000', '¥10,000'],
    ['', '', '小計', '¥310,000'],
    ['', '', '消費税(10%)', '¥31,000'],
    ['', '', '合計', '¥341,000'],
]

table = Table(data, colWidths=[70*mm, 30*mm, 35*mm, 35*mm])
table.setStyle(TableStyle([
    # ヘッダー
    ('BACKGROUND', (0, 0), (-1, 0), colors.HexColor('#2C3E50')),
    ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),
    ('FONTNAME', (0, 0), (-1, -1), 'HeiseiKakuGo-W5'),
    ('FONTSIZE', (0, 0), (-1, 0), 11),
    ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
    
    # データ行
    ('BACKGROUND', (0, 1), (-1, -4), colors.HexColor('#ECF0F1')),
    ('GRID', (0, 0), (-1, -4), 1, colors.black),
    
    # 小計・税・合計
    ('BACKGROUND', (2, -3), (-1, -1), colors.HexColor('#BDC3C7')),
    ('FONTSIZE', (2, -1), (-1, -1), 12),
    ('TEXTCOLOR', (2, -1), (-1, -1), colors.HexColor('#C0392B')),
    
    # 右寄せ
    ('ALIGN', (1, 1), (-1, -1), 'RIGHT'),
]))

story.append(table)
story.append(Spacer(1, 30))

# 振込先情報
story.append(Paragraph("【お振込先】", styles['Heading2']))
bank_info = """
〇〇銀行 △△支店
普通預金 1234567
カ）サンプルカイシャ
"""
story.append(Paragraph(bank_info, styles['Normal']))
story.append(Spacer(1, 20))

# 備考
story.append(Paragraph("【備考】", styles['Heading2']))
story.append(Paragraph("• お支払期限: 翌月末日", styles['Normal']))
story.append(Paragraph("• ご不明な点がございましたらお問い合わせください", styles['Normal']))

# 会社情報（フッター的な内容）
story.append(Spacer(1, 30))
company_info = """
株式会社サンプル
〒100-0001 東京都千代田区千代田1-1-1
TEL: 03-1234-5678 / FAX: 03-1234-5679
Email: info@sample.co.jp
"""
footer_style = ParagraphStyle(
    'Footer',
    parent=styles['Normal'],
    fontName='HeiseiKakuGo-W5',
    fontSize=9,
    alignment=TA_CENTER,
    textColor=colors.grey,
)
story.append(Paragraph(company_info, footer_style))

doc.build(story)

pdf_buffer.seek(0)
pdf_base64 = base64.b64encode(pdf_buffer.read()).decode('utf-8')
js.pdf_base64_data = pdf_base64
```

### **レベル4: 高度な文書（複数ページ、ヘッダー/フッター付き）**
```python
from reportlab.lib.pagesizes import A4
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle, PageBreak
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib import colors
from reportlab.lib.units import mm, cm
from reportlab.lib.enums import TA_CENTER, TA_RIGHT, TA_JUSTIFY
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.cidfonts import UnicodeCIDFont
from io import BytesIO
import base64
import js
from datetime import date

# ヘッダー・フッター関数
def header_footer(canvas, doc):
    canvas.saveState()
    
    # ヘッダー
    canvas.setFont('HeiseiKakuGo-W5', 9)
    canvas.drawString(30*mm, A4[1] - 20*mm, "年次報告書 2025")
    canvas.drawRightString(A4[0] - 30*mm, A4[1] - 20*mm, 
                          date.today().strftime("%Y年%m月%d日"))
    
    # ヘッダーライン
    canvas.setStrokeColor(colors.grey)
    canvas.setLineWidth(0.5)
    canvas.line(30*mm, A4[1] - 25*mm, A4[0] - 30*mm, A4[1] - 25*mm)
    
    # フッター
    canvas.drawString(30*mm, 20*mm, "株式会社サンプル")
    canvas.drawCentredString(A4[0] / 2, 20*mm, f"- {doc.page} -")
    canvas.drawRightString(A4[0] - 30*mm, 20*mm, "Confidential")
    
    # フッターライン
    canvas.line(30*mm, 25*mm, A4[0] - 30*mm, 25*mm)
    
    canvas.restoreState()

# ドキュメント作成
pdf_buffer = BytesIO()
doc = SimpleDocTemplate(
    pdf_buffer,
    pagesize=A4,
    rightMargin=30*mm,
    leftMargin=30*mm,
    topMargin=40*mm,
    bottomMargin=35*mm,
    title="年次報告書",
    author="株式会社サンプル",
    subject="2025年度業績報告",
    creator="自動生成システム",
)

# 日本語フォント
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiKakuGo-W5'))
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiMin-W3'))

# スタイル
styles = getSampleStyleSheet()
styles['Normal'].fontName = 'HeiseiKakuGo-W5'
styles['Normal'].fontSize = 10
styles['Normal'].leading = 14

styles['Title'].fontName = 'HeiseiMin-W3'
styles['Title'].fontSize = 24
styles['Title'].leading = 28
styles['Title'].alignment = TA_CENTER

styles['Heading1'].fontName = 'HeiseiKakuGo-W5'
styles['Heading1'].fontSize = 16
styles['Heading1'].leading = 20
styles['Heading1'].spaceAfter = 10

styles['Heading2'].fontName = 'HeiseiKakuGo-W5'
styles['Heading2'].fontSize = 14
styles['Heading2'].leading = 18

# 本文スタイル（両端揃え）
body_style = ParagraphStyle(
    'BodyText',
    parent=styles['Normal'],
    fontName='HeiseiKakuGo-W5',
    fontSize=10,
    leading=16,
    alignment=TA_JUSTIFY,
    spaceAfter=10,
)

story = []

# 表紙ページ
story.append(Spacer(1, 100))
story.append(Paragraph("2025年度", styles['Title']))
story.append(Spacer(1, 20))
story.append(Paragraph("年次報告書", styles['Title']))
story.append(Spacer(1, 50))
story.append(Paragraph("株式会社サンプル", styles['Title']))
story.append(PageBreak())

# 目次
story.append(Paragraph("目次", styles['Title']))
story.append(Spacer(1, 20))

toc_data = [
    ['1. 経営者メッセージ', '3'],
    ['2. 事業概要', '4'],
    ['3. 財務ハイライト', '5'],
    ['4. 事業別業績', '6'],
    ['5. 今後の展望', '7'],
]

toc_table = Table(toc_data, colWidths=[140*mm, 20*mm])
toc_table.setStyle(TableStyle([
    ('FONTNAME', (0, 0), (-1, -1), 'HeiseiKakuGo-W5'),
    ('FONTSIZE', (0, 0), (-1, -1), 12),
    ('ALIGN', (1, 0), (1, -1), 'RIGHT'),
    ('LINEBELOW', (0, 0), (-1, -1), 0.5, colors.grey),
    ('BOTTOMPADDING', (0, 0), (-1, -1), 8),
]))
story.append(toc_table)
story.append(PageBreak())

# 1. 経営者メッセージ
story.append(Paragraph("1. 経営者メッセージ", styles['Heading1']))
story.append(Spacer(1, 10))
message_text = """
株主・投資家の皆様へ

2025年度は、当社にとって大きな転換期となりました。新型コロナウイルスの影響から完全に回復し、
デジタルトランスフォーメーションの推進により、業績は過去最高を更新することができました。

売上高は前年比20%増の120億円、営業利益は25%増の15億円となり、
全ての事業セグメントで成長を達成しました。これも皆様のご支援の賜物と深く感謝申し上げます。

今後も持続可能な成長を目指し、ESG経営を推進してまいります。
引き続きご支援のほど、よろしくお願い申し上げます。

代表取締役社長
山田 太郎
"""
for paragraph in message_text.strip().split('\n\n'):
    story.append(Paragraph(paragraph, body_style))
story.append(PageBreak())

# 2. 事業概要
story.append(Paragraph("2. 事業概要", styles['Heading1']))
story.append(Spacer(1, 10))

story.append(Paragraph("2.1 主要事業", styles['Heading2']))
story.append(Spacer(1, 5))

business_data = [
    ['事業部門', '売上構成比', '主要製品・サービス'],
    ['ITソリューション', '45%', 'システム開発、クラウドサービス'],
    ['コンサルティング', '30%', '経営コンサルティング、DX支援'],
    ['プロダクト', '25%', 'パッケージソフトウェア、SaaS'],
]

business_table = Table(business_data, colWidths=[50*mm, 35*mm, 70*mm])
business_table.setStyle(TableStyle([
    ('BACKGROUND', (0, 0), (-1, 0), colors.HexColor('#34495E')),
    ('TEXTCOLOR', (0, 0), (-1, 0), colors.white),
    ('FONTNAME', (0, 0), (-1, -1), 'HeiseiKakuGo-W5'),
    ('FONTSIZE', (0, 0), (-1, -1), 10),
    ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
    ('GRID', (0, 0), (-1, -1), 1, colors.grey),
    ('BACKGROUND', (0, 1), (-1, -1), colors.HexColor('#ECF0F1')),
]))
story.append(business_table)
story.append(Spacer(1, 15))

story.append(Paragraph("2.2 グローバル展開", styles['Heading2']))
story.append(Spacer(1, 5))
global_text = """
当社は現在、アジア・北米・欧州の3地域で事業を展開しています。
海外売上比率は35%に達し、特にアジア地域での成長が顕著です。
今後は東南アジアへの展開を加速し、2030年までに海外売上比率50%を目指します。
"""
story.append(Paragraph(global_text, body_style))
story.append(PageBreak())

# 3. 財務ハイライト
story.append(Paragraph("3. 財務ハイライト", styles['Heading1']))
story.append(Spacer(1, 10))

financial_data = [
    ['項目', '2023年度', '2024年度', '2025年度', '前年比'],
    ['売上高（億円）', '90', '100', '120', '+20%'],
    ['営業利益（億円）', '10', '12', '15', '+25%'],
    ['純利益（億円）', '7', '8.5', '11', '+29%'],
    ['ROE（%）', '12.5', '14.2', '16.8', '+2.6pt'],
    ['配当金（円）', '30', '35', '40', '+14%'],
]

financial_table = Table(financial_data, colWidths=[40*mm, 28*mm, 28*mm, 28*mm, 25*mm])
financial_table.setStyle(TableStyle([
    ('BACKGROUND', (0, 0), (-1, 0), colors.HexColor('#2C3E50')),
    ('TEXTCOLOR', (0, 0), (-1, 0), colors.white),
    ('FONTNAME', (0, 0), (-1, -1), 'HeiseiKakuGo-W5'),
    ('FONTSIZE', (0, 0), (-1, -1), 9),
    ('ALIGN', (1, 0), (-1, -1), 'RIGHT'),
    ('ALIGN', (0, 0), (0, -1), 'LEFT'),
    ('GRID', (0, 0), (-1, -1), 1, colors.grey),
    ('BACKGROUND', (0, 1), (-1, -1), colors.white),
    ('ROWBACKGROUNDS', (0, 1), (-1, -1), [colors.white, colors.HexColor('#F8F9FA')]),
]))
story.append(financial_table)

# SimpleDocTemplateでヘッダー・フッターを設定
doc.build(story, onFirstPage=header_footer, onLaterPages=header_footer)

pdf_buffer.seek(0)
pdf_base64 = base64.b64encode(pdf_buffer.read()).decode('utf-8')
js.pdf_base64_data = pdf_base64
```

---

## **📑 ビジネス文書テンプレート**

### **領収書**
```python
# 領収書テンプレート
from reportlab.lib.units import mm
from reportlab.lib.colors import HexColor

# タイトル
story.append(Paragraph("領収書", styles['Title']))
story.append(Spacer(1, 20))

# 宛名と金額
story.append(Paragraph("〇〇株式会社 様", styles['Normal']))
story.append(Spacer(1, 10))

amount_style = ParagraphStyle(
    'Amount',
    fontName='HeiseiKakuGo-W5',
    fontSize=20,
    alignment=TA_CENTER,
)
story.append(Paragraph("¥100,000-", amount_style))
story.append(Spacer(1, 10))
story.append(Paragraph("（金壱拾万円也）", styles['Normal']))

# 但し書き
story.append(Spacer(1, 15))
story.append(Paragraph("但し、コンサルティング料として", styles['Normal']))

# 日付と発行者
story.append(Spacer(1, 20))
story.append(Paragraph(f"発行日: {date.today().strftime('%Y年%m月%d日')}", styles['Normal']))

# 収入印紙欄
stamp_data = [['収入印紙']]
stamp_table = Table(stamp_data, colWidths=[30*mm], rowHeights=[30*mm])
stamp_table.setStyle(TableStyle([
    ('BOX', (0, 0), (-1, -1), 1, colors.black),
    ('ALIGN', (0, 0), (-1, -1), 'CENTER'),
    ('VALIGN', (0, 0), (-1, -1), 'MIDDLE'),
    ('FONTNAME', (0, 0), (-1, -1), 'HeiseiKakuGo-W5'),
]))
story.append(stamp_table)
```

### **証明書・賞状**
```python
# 証明書テンプレート
from reportlab.lib.enums import TA_CENTER

cert_title_style = ParagraphStyle(
    'CertTitle',
    fontName='HeiseiMin-W3',
    fontSize=28,
    alignment=TA_CENTER,
    spaceAfter=30
)

cert_body_style = ParagraphStyle(
    'CertBody',
    fontName='HeiseiKakuGo-W5',
    fontSize=14,
    alignment=TA_CENTER,
    leading=24,
)

story.append(Spacer(1, 50))
story.append(Paragraph("修了証書", cert_title_style))
story.append(Spacer(1, 40))

story.append(Paragraph("氏名: 山田 太郎 殿", cert_body_style))
story.append(Spacer(1, 30))

cert_text = """
あなたは、当社の実施した
「プロジェクトマネジメント研修」
を優秀な成績で修了されたことを
ここに証明いたします。
"""
for line in cert_text.strip().split('\n'):
    story.append(Paragraph(line, cert_body_style))
    story.append(Spacer(1, 10))

story.append(Spacer(1, 40))
story.append(Paragraph(date.today().strftime("%Y年%m月%d日"), cert_body_style))
story.append(Spacer(1, 20))
story.append(Paragraph("株式会社サンプル", cert_body_style))
story.append(Paragraph("代表取締役 印", cert_body_style))
```

### **契約書**
```python
# 契約書テンプレート
contract_style = ParagraphStyle(
    'Contract',
    fontName='HeiseiKakuGo-W5',
    fontSize=10,
    leading=16,
    alignment=TA_JUSTIFY,
)

story.append(Paragraph("業務委託契約書", styles['Title']))
story.append(Spacer(1, 20))

contract_text = """
〇〇株式会社（以下「甲」という）と△△株式会社（以下「乙」という）は、
以下の通り業務委託契約（以下「本契約」という）を締結する。

第1条（目的）
甲は乙に対し、別紙記載の業務（以下「本業務」という）を委託し、
乙はこれを受託する。

第2条（業務内容）
本業務の詳細は、別途定める仕様書による。

第3条（契約期間）
本契約の有効期間は、2025年4月1日から2026年3月31日までとする。

第4条（報酬）
甲は乙に対し、本業務の対価として月額金100,000円（消費税別）を支払う。

第5条（秘密保持）
甲及び乙は、本契約に関して知り得た相手方の秘密情報を、
相手方の事前の書面による承諾なく第三者に開示してはならない。
"""

for paragraph in contract_text.strip().split('\n\n'):
    story.append(Paragraph(paragraph, contract_style))
    story.append(Spacer(1, 10))

# 署名欄
story.append(Spacer(1, 30))
signature_data = [
    ['甲', '', '乙', ''],
    ['住所:', '_______________', '住所:', '_______________'],
    ['会社名:', '_______________', '会社名:', '_______________'],
    ['代表者:', '_______________', '代表者:', '_______________'],
    ['印', '', '印', ''],
]

signature_table = Table(signature_data, colWidths=[20*mm, 60*mm, 20*mm, 60*mm])
signature_table.setStyle(TableStyle([
    ('FONTNAME', (0, 0), (-1, -1), 'HeiseiKakuGo-W5'),
    ('FONTSIZE', (0, 0), (-1, -1), 10),
    ('LINEBELOW', (1, 1), (1, 3), 0.5, colors.black),
    ('LINEBELOW', (3, 1), (3, 3), 0.5, colors.black),
]))
story.append(signature_table)
```

### **見積書**
```python
# 見積書テンプレート
story.append(Paragraph("御見積書", styles['Title']))
story.append(Spacer(1, 20))

story.append(Paragraph("〇〇株式会社 御中", styles['Normal']))
story.append(Spacer(1, 15))

estimate_summary = Table([
    ['御見積金額', '¥1,100,000（税込）'],
    ['件名', 'Webサイト制作一式'],
    ['納期', '契約締結後2ヶ月'],
    ['有効期限', '発行日より30日間'],
], colWidths=[40*mm, 100*mm])

estimate_summary.setStyle(TableStyle([
    ('FONTNAME', (0, 0), (-1, -1), 'HeiseiKakuGo-W5'),
    ('FONTSIZE', (0, 0), (0, 0), 12),
    ('FONTSIZE', (1, 0), (1, 0), 14),
    ('TEXTCOLOR', (1, 0), (1, 0), colors.red),
    ('BACKGROUND', (0, 0), (0, -1), colors.HexColor('#E8E8E8')),
    ('ALIGN', (0, 0), (0, -1), 'RIGHT'),
    ('BOX', (0, 0), (-1, -1), 1, colors.black),
    ('GRID', (0, 0), (-1, -1), 0.5, colors.grey),
]))

story.append(estimate_summary)
story.append(Spacer(1, 20))

# 明細
story.append(Paragraph("【明細】", styles['Heading2']))
story.append(Spacer(1, 10))

detail_data = [
    ['項目', '数量', '単価', '金額'],
    ['デザイン費', '一式', '¥300,000', '¥300,000'],
    ['コーディング費', '10ページ', '¥30,000', '¥300,000'],
    ['システム開発費', '一式', '¥200,000', '¥200,000'],
    ['ディレクション費', '2ヶ月', '¥50,000', '¥100,000'],
    ['', '', '小計', '¥900,000'],
    ['', '', '消費税(10%)', '¥100,000'],
    ['', '', '合計', '¥1,000,000'],
]

detail_table = Table(detail_data, colWidths=[60*mm, 30*mm, 35*mm, 35*mm])
# スタイル設定（請求書と同様）
```

---

## **🔒 PDF特有の高度な機能**

### **ウォーターマーク（透かし）**
```python
def add_watermark(canvas, doc):
    """各ページに透かしを追加"""
    canvas.saveState()
    
    # 透かしテキストの設定
    canvas.setFont("HeiseiKakuGo-W5", 60)
    canvas.setFillColorRGB(0.8, 0.8, 0.8, alpha=0.3)  # 薄いグレー、透明度30%
    
    # 45度回転
    canvas.translate(A4[0]/2, A4[1]/2)
    canvas.rotate(45)
    
    # センター配置
    canvas.drawCentredString(0, 0, "CONFIDENTIAL")
    
    canvas.restoreState()

# SimpleDocTemplateで使用
doc = SimpleDocTemplate(
    pdf_buffer,
    pagesize=A4,
    onFirstPage=add_watermark,
    onLaterPages=add_watermark
)
```

### **QRコード生成**
```python
# qrcodeライブラリを使用（要インストール）
import qrcode
from io import BytesIO
from reportlab.platypus import Image

# QRコード生成
qr = qrcode.QRCode(
    version=1,
    error_correction=qrcode.constants.ERROR_CORRECT_L,
    box_size=10,
    border=4,
)
qr.add_data('https://example.com/invoice/12345')
qr.make(fit=True)

# 画像として保存
img = qr.make_image(fill_color="black", back_color="white")
img_buffer = BytesIO()
img.save(img_buffer, format='PNG')
img_buffer.seek(0)

# PDFに追加
qr_image = Image(img_buffer, width=30*mm, height=30*mm)
story.append(qr_image)
```

### **パスワード保護**
```python
from reportlab.lib import pdfencrypt

# 暗号化設定
encrypt = pdfencrypt.StandardEncryption(
    userPassword="user123",      # 閲覧パスワード
    ownerPassword="owner456",     # 管理者パスワード
    canPrint=1,                  # 印刷許可
    canModify=0,                 # 編集禁止
    canCopy=0,                   # コピー禁止
    canAnnotate=0                # 注釈禁止
)

# ドキュメント作成時に暗号化を適用
doc = SimpleDocTemplate(
    pdf_buffer,
    pagesize=A4,
    encrypt=encrypt
)
```

### **メタデータ設定**
```python
# PDFのプロパティ設定
doc = SimpleDocTemplate(
    pdf_buffer,
    pagesize=A4,
    title="請求書 No.2025-001",
    author="株式会社サンプル",
    subject="2025年1月分請求書",
    creator="自動生成システム v2.0",
    producer="ReportLab",
    keywords="請求書,2025年,1月",
    creationDate=date.today()
)
```

### **カスタムページサイズ**
```python
from reportlab.lib.pagesizes import landscape, portrait

# A4横向き
doc = SimpleDocTemplate(pdf_buffer, pagesize=landscape(A4))

# カスタムサイズ（名刺サイズ）
BUSINESS_CARD = (91*mm, 55*mm)
doc = SimpleDocTemplate(pdf_buffer, pagesize=BUSINESS_CARD)

# B5サイズ
B5 = (182*mm, 257*mm)
doc = SimpleDocTemplate(pdf_buffer, pagesize=B5)
```

---

## **🌏 日本語対応ガイド**

### **利用可能な日本語CIDフォント**
```python
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.cidfonts import UnicodeCIDFont

# ゴシック体
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiKakuGo-W5'))

# 明朝体
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiMin-W3'))

# 明朝体（太字）
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiMin-W7'))
```

### **縦書き対応**
```python
# 縦書きスタイル（実験的機能）
from reportlab.platypus import Paragraph
from reportlab.lib.styles import ParagraphStyle

vertical_style = ParagraphStyle(
    'Vertical',
    fontName='HeiseiMin-W3',
    fontSize=12,
    wordWrap='CJK',  # 日中韓文字の改行
)

# 注：完全な縦書きはReportLabでは制限あり
# 必要に応じて他のライブラリとの併用を検討
```

### **和暦表示**
```python
from datetime import date

def to_japanese_era(dt):
    """西暦を和暦に変換"""
    year = dt.year
    if year >= 2019:
        era = "令和"
        era_year = year - 2018
    elif year >= 1989:
        era = "平成"
        era_year = year - 1988
    elif year >= 1926:
        era = "昭和"
        era_year = year - 1925
    else:
        return f"{year}年"
    
    return f"{era}{era_year}年"

# 使用例
today = date.today()
japanese_date = to_japanese_era(today) + today.strftime("%m月%d日")
story.append(Paragraph(f"発行日: {japanese_date}", styles['Normal']))
```

---

## **✅ 実装チェックリスト**

### **必須確認項目**
- [ ] reportlabインストール処理が存在する
- [ ] Pythonコードが左端から開始されている
- [ ] `import js` が含まれている
- [ ] `js.pdf_base64_data` への代入がある
- [ ] エラーオーバーレイのHTMLが完全である

### **日本語対応確認項目**
- [ ] CIDフォントが登録されている
- [ ] スタイルにfontNameが設定されている
- [ ] 日本語テキストが正しく表示される

### **動作確認項目**
- [ ] ページが正常に読み込まれる
- [ ] ダウンロードボタンが有効になる
- [ ] PDFファイルがダウンロードできる
- [ ] エラー時にオーバーレイが表示される
- [ ] エラーコピーボタンが機能する

---

## **📊 ReportLabリファレンス**

### **基本要素**
| クラス | 用途 | 例 |
|--------|------|-----|
| `SimpleDocTemplate` | PDF文書の基本構造 | `doc = SimpleDocTemplate(buffer, pagesize=A4)` |
| `Paragraph` | 段落テキスト | `Paragraph("テキスト", style)` |
| `Table` | 表 | `Table(data, colWidths=[...])` |
| `Spacer` | 空白スペース | `Spacer(1, 20)` |
| `PageBreak` | 改ページ | `PageBreak()` |
| `Image` | 画像 | `Image(buffer, width=100, height=100)` |

### **ページサイズ**
```python
from reportlab.lib.pagesizes import A4, A3, letter, legal, landscape

# 標準サイズ
A4 = (210*mm, 297*mm)
A3 = (297*mm, 420*mm)
letter = (8.5*inch, 11*inch)
legal = (8.5*inch, 14*inch)

# 向き
portrait(A4)   # 縦向き（デフォルト）
landscape(A4)  # 横向き
```

### **色の定義**
```python
from reportlab.lib import colors

# 基本色
colors.black, colors.white, colors.red, colors.blue, colors.green

# RGB色
colors.Color(0.5, 0.5, 0.5)  # グレー

# HEX色
colors.HexColor('#2C3E50')

# 透明度付き
colors.Color(0, 0, 0, alpha=0.5)
```

### **単位**
```python
from reportlab.lib.units import inch, cm, mm, pt

# 変換
1*inch = 72*pt
1*cm = 28.35*pt
1*mm = 2.835*pt
```

---

## **🚀 クイックスタート**

### **最速実装（3ステップ）**

1. **HTMLテンプレートをコピー**
2. **タイトルを変更**（`<title>`と`<h1>`の2箇所）
3. **Pythonコードで文書内容を定義**

```python
# 最小限の変更例
doc = SimpleDocTemplate(pdf_buffer, pagesize=A4)
story = []
story.append(Paragraph("あなたの文書タイトル", styles['Title']))
story.append(Paragraph("本文をここに", styles['Normal']))
doc.build(story)
# ... 残りは保存処理（テンプレート通り）
```

---

## **📝 更新履歴**

- **v2.0** (2025-01-17)
  - 実装ルールを優先順位別に再構成
  - エラーUI機能を追加（フルスクリーンオーバーレイ）
  - 実装例を4段階に拡充
  - ビジネス文書テンプレート5種類追加
  - PDF特有の高度な機能を追加（暗号化、メタデータ等）

- **v1.2** (2025-01-17)
  - 外部ライブラリインストールの重要性を強調
  - エラー対処法を追加

- **v1.1** (2025-09-12)
  - ヘッダー/フッター機能の記述を強化

- **v1.0** (2025-01-17)
  - 初版リリース

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
   - 特にreportlabのインストール処理

4. **実装例を参考に**
   - レベル1の最小限実装から始める
   - 段階的に機能を追加

### **よくある質問**

**Q: 日本語が文字化けします**
A: CIDフォント（HeiseiKakuGo-W5等）を使用し、全てのスタイルでfontNameを設定してください。

**Q: 複雑なレイアウトは可能ですか？**
A: Frame、PageTemplateを使用して多段組みレイアウトが可能です。ただし、HTMLのような柔軟性は限定的です。

**Q: 既存のPDFを編集できますか？**
A: ReportLabは新規作成が主目的です。既存PDF編集にはPyPDF2等の併用を検討してください。

**Q: グラフを埋め込めますか？**
A: matplotlibで作成したグラフを画像として埋め込むことが可能です（レベル4の例参照）。

---

**これでレシピv2.0の全文です。AIが正しく理解し、適切なPDF生成ページを作成できるよう、詳細な説明と実例を含めました。**