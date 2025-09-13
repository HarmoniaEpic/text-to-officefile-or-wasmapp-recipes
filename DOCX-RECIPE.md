# **DOCX自動生成ページ レシピ v3.0**
**AIによるワンクリック・ダウンロードページ生成の完全ガイド**

---

## **📌 このレシピについて**

生成AIがユーザーの指示に基づき、Word文書（.docx）をブラウザ上で生成・ダウンロードできる**単一HTMLファイル**を自動作成するための完全な指示書です。

### **v3.0の特徴**
- 🎯 実装ルールを優先順位別に明確化
- 🖼️ 画像はすべてプレースホルダー方式（直接埋め込みなし）
- 📝 Word特有の高度な機能を網羅
- 🔍 エラー対処の手順と実例を完備
- 📊 段階的な実装例で学習しやすい

---

## **🚨 実装ルール（優先順位順）**

### **🔴 レベル1: 絶対禁止事項（変更・削除厳禁）**

以下の要素は**一切変更してはいけません**：

#### **1.1 python-docxライブラリのインストール処理**
```javascript
await pyodide.loadPackage("micropip");
await pyodide.runPythonAsync(`
    import micropip
    await micropip.install('python-docx')
`);
```

**なぜ絶対に必要か：**
- python-docxはWord文書生成のための外部ライブラリ
- ブラウザのPyodide環境には標準で含まれていない
- micropipを使用してランタイムでインストールする必要がある
- **削除すると必ず発生するエラー**: `ModuleNotFoundError: No module named 'docx'`

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
from docx import Document
import base64

# ❌ 間違い（インデントあり）
    from docx import Document  # IndentationError発生
    import base64
```

**理由**: HTMLのインデントに影響されてPythonコードにインデントを追加するとエラーになる

#### **2.2 必須のインポート文**
```python
from docx import Document
from io import BytesIO
import base64
import js  # JavaScriptとの連携に必須
```

**省略した場合のエラー**: `NameError: name 'js' is not defined`

#### **2.3 Base64データの受け渡し**
Pythonコードの最終部で必ず実行：
```python
js.docx_base64_data = docx_base64
```

**理由**: JavaScriptがDOCXデータを受け取るための必須処理

### **🟢 レベル3: カスタマイズ可能要素**

以下は自由に変更できます：
- ページタイトル（`<title>`タグと`<h1>`タグ）
- Pythonコード内の文書内容
- 文書のスタイル・レイアウト
- ダウンロードファイル名
- プレースホルダーの配置と内容

---

## **🖼️ 画像プレースホルダー機能**

### **基本方針**
**すべての画像はプレースホルダーとして処理されます**
- 画像の直接埋め込みは行いません（base64エンコードによるコンテキストウィンドウ超過を防ぐため）
- 実際の画像挿入はユーザーがWordを開いて行います
- AIは最適な配置位置と推奨サイズを提案します

### **なぜプレースホルダー方式なのか**
- **HTMLファイルの軽量化**: 数KBで済む（画像埋め込みだと数MB〜数十MB）
- **コンテキストウィンドウの節約**: AIの処理限界を超えない
- **柔軟性**: 後から自由に画像を変更可能
- **共有性**: 複数人での使い回しが容易
- **著作権**: 画像の権利問題を回避

### **プレースホルダーに含める情報**
1. **推奨ファイル名または説明**（例：`logo.png`、`メイン画像`）
2. **配置位置**（例：「ヘッダー右上」「本文中央」）
3. **推奨サイズ**（例：`600x400px`、`3x2 inches`）
4. **用途**（例：「企業ロゴ」「説明図」「グラフ」）
5. **キャプション案**（例：「図1: システム構成図」）

### **実装時の判断基準**
ユーザーが画像について言及した場合：
- 「ロゴを入れて」→ ヘッダーにロゴプレースホルダー作成
- 「図表を配置」→ 本文中に図表プレースホルダー作成
- 「写真を追加」→ 写真用プレースホルダー作成
- 画像ファイルが添付された場合 → ファイル名を参考にプレースホルダー作成

### **推奨配置パターン**
| 文書タイプ | 画像の種類 | 推奨位置 | 推奨サイズ |
|-----------|-----------|---------|-----------|
| レポート | ロゴ | ヘッダー右上 | 2×1 inches |
| レポート | カバー画像 | タイトル下 | 6×4 inches |
| マニュアル | スクリーンショット | 手順説明後 | 5×3 inches |
| 提案書 | 製品画像 | セクション内 | 4×3 inches |
| 契約書 | 社印 | フッター | 1×1 inches |

---

## **📝 HTMLテンプレート（完全版）**

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- カスタマイズ可能: ページタイトル -->
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

    <!-- カスタマイズ可能: Pythonコード（ルールに従って） -->
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

                // python-docxのインストール（削除禁止）
                statusMessage.textContent = 'ライブラリをインストール中...';
                await pyodide.loadPackage("micropip");
                await pyodide.runPythonAsync(`
                    import micropip
                    await micropip.install('python-docx')
                `);

                statusMessage.textContent = 'ファイルを生成しています...';
                const pythonCode = document.getElementById('python-code').textContent;
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

---

## **🔧 エラー診断ガイド**

### **よくあるエラーと解決方法**

#### **1. ModuleNotFoundError**
```
ModuleNotFoundError: No module named 'docx'
```
**原因**: python-docxのインストール処理が削除されている
**解決**: `await micropip.install('python-docx')` を復元

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
AttributeError: 'Document' object has no attribute 'add_heading'
```
**原因**: python-docxのAPIを間違って使用
**解決**: 正しいメソッド名を確認（add_heading, add_paragraph等）

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
from docx import Document
from io import BytesIO
import base64
import js

doc = Document()
doc.add_heading('シンプルな文書', 0)
doc.add_paragraph('基本的な段落です。')

docx_io = BytesIO()
doc.save(docx_io)
docx_io.seek(0)
js.docx_base64_data = base64.b64encode(docx_io.read()).decode('utf-8')
```

### **レベル2: 基本的な文書構成**
```python
from docx import Document
from docx.shared import Pt, Inches
from docx.enum.text import WD_ALIGN_PARAGRAPH
from io import BytesIO
import base64
import js

doc = Document()

# タイトル
doc.add_heading('業務報告書', 0)
doc.add_heading('2025年1月度', level=1)

# 概要
doc.add_heading('概要', level=2)
doc.add_paragraph('今月の業務内容について報告いたします。')

# 箇条書き
doc.add_heading('実施項目', level=2)
doc.add_paragraph('データ分析', style='List Bullet')
doc.add_paragraph('レポート作成', style='List Bullet')
doc.add_paragraph('会議参加', style='List Bullet')

# 番号付きリスト
doc.add_heading('今後の予定', level=2)
doc.add_paragraph('要件定義の完了', style='List Number')
doc.add_paragraph('設計書の作成', style='List Number')
doc.add_paragraph('実装開始', style='List Number')

# 表の追加
doc.add_heading('進捗状況', level=2)
table = doc.add_table(rows=4, cols=3)
table.style = 'Light Grid Accent 1'

# ヘッダー行
hdr_cells = table.rows[0].cells
hdr_cells[0].text = 'タスク'
hdr_cells[1].text = '進捗'
hdr_cells[2].text = 'ステータス'

# データ行
data = [
    ('分析', '100%', '完了'),
    ('設計', '60%', '進行中'),
    ('実装', '0%', '未着手')
]

for i, (task, progress, status) in enumerate(data, 1):
    row_cells = table.rows[i].cells
    row_cells[0].text = task
    row_cells[1].text = progress
    row_cells[2].text = status

# 保存
docx_io = BytesIO()
doc.save(docx_io)
docx_io.seek(0)
js.docx_base64_data = base64.b64encode(docx_io.read()).decode('utf-8')
```

### **レベル3: 画像プレースホルダー付き**
```python
from docx import Document
from docx.shared import Inches, Pt, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.oxml import parse_xml
from docx.oxml.ns import nsdecls
from io import BytesIO
import base64
import js

doc = Document()

# ヘッダーロゴプレースホルダー
p = doc.add_paragraph()
p.alignment = WD_ALIGN_PARAGRAPH.RIGHT
run = p.add_run('【ロゴ画像挿入位置】')
run.bold = True
run.font.size = Pt(8)
run.font.color.rgb = RGBColor(150, 150, 150)
p.add_run('\nlogo.png (2×1 inches)')

# メインタイトル
doc.add_heading('製品提案書', 0)

# カバー画像プレースホルダー
table = doc.add_table(rows=1, cols=1)
table.style = 'Table Grid'
cell = table.cell(0, 0)
cell.text = ('【カバー画像プレースホルダー】\n'
             'cover_image.jpg\n'
             '推奨: 6×4 inches\n'
             '製品の魅力を伝えるメインビジュアル')

# セルの背景色
shading_elm = parse_xml(r'<w:shd {} w:fill="E6F2FF"/>'.format(nsdecls('w')))
cell._element.get_or_add_tcPr().append(shading_elm)

doc.add_page_break()

# 本文
doc.add_heading('1. 製品概要', level=1)
doc.add_paragraph('弊社の製品について説明します。')

# 製品画像プレースホルダー
doc.add_paragraph()
p = doc.add_paragraph('【図1: 製品画像】')
p.alignment = WD_ALIGN_PARAGRAPH.CENTER
run = p.runs[0]
run.bold = True
run.font.color.rgb = RGBColor(0, 100, 200)

table = doc.add_table(rows=1, cols=1)
table.alignment = WD_ALIGN_PARAGRAPH.CENTER
cell = table.cell(0, 0)
cell.text = ('product_image.png\n'
             '推奨: 4×3 inches\n'
             '製品の外観')

doc.add_heading('2. 特徴', level=1)
doc.add_paragraph('• 高性能', style='List Bullet')
doc.add_paragraph('• 低価格', style='List Bullet')
doc.add_paragraph('• 使いやすい', style='List Bullet')

# グラフプレースホルダー
doc.add_heading('3. 性能比較', level=1)
p = doc.add_paragraph('【グラフ挿入位置】')
p.alignment = WD_ALIGN_PARAGRAPH.CENTER
p.add_run('\nperformance_chart.png (5×3 inches)')
p.add_run('\n他社製品との性能比較グラフ')

# 保存
docx_io = BytesIO()
doc.save(docx_io)
docx_io.seek(0)
js.docx_base64_data = base64.b64encode(docx_io.read()).decode('utf-8')
```

### **レベル4: 高度な書式設定とスタイル**
```python
from docx import Document
from docx.shared import Inches, Pt, RGBColor, Cm
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.enum.style import WD_STYLE_TYPE
from docx.enum.section import WD_SECTION
from io import BytesIO
import base64
import js

def add_image_placeholder(doc, position, size, description):
    """汎用的な画像プレースホルダー追加関数"""
    table = doc.add_table(rows=1, cols=1)
    table.style = 'Table Grid'
    cell = table.cell(0, 0)
    cell.text = description
    
    # 背景色設定
    from docx.oxml import parse_xml
    from docx.oxml.ns import nsdecls
    shading_elm = parse_xml(r'<w:shd {} w:fill="F0F0F0"/>'.format(nsdecls('w')))
    cell._element.get_or_add_tcPr().append(shading_elm)
    
    if position == 'center':
        table.alignment = WD_ALIGN_PARAGRAPH.CENTER
    elif position == 'right':
        table.alignment = WD_ALIGN_PARAGRAPH.RIGHT
    
    return table

doc = Document()

# ページ設定
section = doc.sections[0]
section.page_height = Cm(29.7)  # A4
section.page_width = Cm(21.0)   # A4
section.left_margin = Cm(2.5)
section.right_margin = Cm(2.5)
section.top_margin = Cm(2.5)
section.bottom_margin = Cm(2.5)

# カスタムスタイルの作成
styles = doc.styles
heading_style = styles.add_style('CustomHeading', WD_STYLE_TYPE.PARAGRAPH)
heading_style.font.name = 'メイリオ'
heading_style.font.size = Pt(14)
heading_style.font.bold = True
heading_style.font.color.rgb = RGBColor(0, 51, 102)

# 表紙
title = doc.add_paragraph('技術仕様書', style='CustomHeading')
title.alignment = WD_ALIGN_PARAGRAPH.CENTER
title.runs[0].font.size = Pt(24)

subtitle = doc.add_paragraph('バージョン 1.0')
subtitle.alignment = WD_ALIGN_PARAGRAPH.CENTER

# 日付
date_p = doc.add_paragraph('2025年1月')
date_p.alignment = WD_ALIGN_PARAGRAPH.CENTER

# ロゴプレースホルダー
add_image_placeholder(
    doc, 'center', (3, 2),
    '【企業ロゴ】\ncompany_logo.png\n3×2 inches\n中央配置'
)

doc.add_page_break()

# 目次プレースホルダー
doc.add_heading('目次', level=1)
p = doc.add_paragraph('【目次プレースホルダー】')
p.add_run('\n※ Wordで「参考資料」→「目次」から自動生成してください')
p.add_run('\n※ 見出しスタイルを使用している箇所が自動的に目次に含まれます')

doc.add_page_break()

# セクション1
doc.add_heading('1. はじめに', level=1)
doc.add_paragraph('本書は製品の技術仕様について記載します。')

# 複雑な表
doc.add_heading('2. 仕様一覧', level=1)
table = doc.add_table(rows=5, cols=4)
table.style = 'Medium Grid 1 Accent 1'

# ヘッダー行をマージ
table.cell(0, 0).merge(table.cell(0, 3))
table.cell(0, 0).text = '製品仕様'

# サブヘッダー
headers = ['項目', '仕様', '備考', '参照']
for i, header in enumerate(headers):
    table.cell(1, i).text = header

# データ
specs = [
    ('サイズ', '100×50×30mm', '±2mm', '図1参照'),
    ('重量', '500g', '±10g', '-'),
    ('動作温度', '-10〜50℃', '結露なきこと', '試験データ参照')
]

for i, (item, spec, note, ref) in enumerate(specs, 2):
    table.cell(i, 0).text = item
    table.cell(i, 1).text = spec
    table.cell(i, 2).text = note
    table.cell(i, 3).text = ref

# 図表プレースホルダー
doc.add_paragraph()
add_image_placeholder(
    doc, 'center', (5, 4),
    '【図1: 製品寸法図】\ndimensions.png\n5×4 inches\nCADから出力した寸法図'
)

# セクション区切り
doc.add_section(WD_SECTION.NEW_PAGE)

# 付録
doc.add_heading('付録A: 試験データ', level=1)
add_image_placeholder(
    doc, 'left', (6, 4),
    '【試験データグラフ】\ntest_results.png\n6×4 inches\n温度試験の結果グラフ'
)

# フッター情報
doc.add_paragraph()
doc.add_paragraph()
p = doc.add_paragraph()
p.alignment = WD_ALIGN_PARAGRAPH.CENTER
run = p.add_run('機密情報 - 取扱注意')
run.font.size = Pt(9)
run.font.color.rgb = RGBColor(128, 128, 128)

# 保存
docx_io = BytesIO()
doc.save(docx_io)
docx_io.seek(0)
js.docx_base64_data = base64.b64encode(docx_io.read()).decode('utf-8')
```

---

## **📚 Word特有の機能**

### **ページ設定**
```python
from docx.shared import Cm
section = doc.sections[0]
section.page_height = Cm(29.7)  # A4縦
section.page_width = Cm(21.0)
section.left_margin = Cm(2.5)
section.right_margin = Cm(2.5)
section.top_margin = Cm(2.5)
section.bottom_margin = Cm(2.5)
section.orientation = WD_ORIENT.LANDSCAPE  # 横向き
```

### **スタイル管理**
```python
# カスタムスタイルの作成
styles = doc.styles
custom_style = styles.add_style('MyStyle', WD_STYLE_TYPE.PARAGRAPH)
custom_style.font.name = 'メイリオ'
custom_style.font.size = Pt(12)
custom_style.font.bold = True

# スタイルの適用
p = doc.add_paragraph('スタイル付きテキスト', style='MyStyle')
```

### **セクション区切り**
```python
from docx.enum.section import WD_SECTION
# 新しいページから始まるセクション
doc.add_section(WD_SECTION.NEW_PAGE)
# 連続セクション（同じページ内）
doc.add_section(WD_SECTION.CONTINUOUS)
```

### **ヘッダー・フッター**
```python
# ヘッダーの設定
header = doc.sections[0].header
header_p = header.paragraphs[0]
header_p.text = "文書タイトル"
header_p.alignment = WD_ALIGN_PARAGRAPH.CENTER

# フッターの設定
footer = doc.sections[0].footer
footer_p = footer.paragraphs[0]
footer_p.text = "ページ番号: "
footer_p.alignment = WD_ALIGN_PARAGRAPH.CENTER
```

### **高度な表操作**
```python
# セルの結合
table.cell(0, 0).merge(table.cell(0, 2))

# セル内の段落書式
cell = table.cell(0, 0)
cell.paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.CENTER

# 行の高さ設定
from docx.oxml import OxmlElement
from docx.oxml.ns import qn
tr = table.rows[0]._tr
trPr = tr.get_or_add_trPr()
trHeight = OxmlElement('w:trHeight')
trHeight.set(qn('w:val'), '1000')  # 高さ（twips単位）
trPr.append(trHeight)
```

---

## **✅ 実装チェックリスト**

### **必須確認項目**
- [ ] python-docxインストール処理が存在する
- [ ] Pythonコードが左端から開始されている
- [ ] `import js` が含まれている
- [ ] `js.docx_base64_data` への代入がある
- [ ] エラーオーバーレイのHTMLが完全である

### **プレースホルダー確認項目**
- [ ] 画像の直接埋め込みを行っていない
- [ ] プレースホルダーに必要な情報が含まれている
- [ ] 視覚的に分かりやすい表示（枠線や背景色）
- [ ] 推奨サイズが明記されている
- [ ] キャプション案が含まれている

### **動作確認項目**
- [ ] ページが正常に読み込まれる
- [ ] ダウンロードボタンが有効になる
- [ ] DOCXファイルがダウンロードできる
- [ ] エラー時にオーバーレイが表示される
- [ ] エラーコピーボタンが機能する

---

## **📊 python-docxリファレンス**

### **文書要素の追加**
| メソッド | 用途 | 例 |
|---------|------|-----|
| `add_heading(text, level)` | 見出し追加 | `doc.add_heading('タイトル', 0)` |
| `add_paragraph(text, style)` | 段落追加 | `doc.add_paragraph('本文')` |
| `add_table(rows, cols)` | 表追加 | `doc.add_table(3, 4)` |
| `add_page_break()` | 改ページ | `doc.add_page_break()` |
| `add_section(start_type)` | セクション区切り | `doc.add_section()` |

### **テキスト書式**
```python
# 段落の取得と書式設定
p = doc.add_paragraph()
run = p.add_run('テキスト')
run.bold = True
run.italic = True
run.underline = True
run.font.size = Pt(12)
run.font.name = 'メイリオ'
run.font.color.rgb = RGBColor(255, 0, 0)

# 段落の配置
p.alignment = WD_ALIGN_PARAGRAPH.CENTER  # 中央寄せ
p.alignment = WD_ALIGN_PARAGRAPH.RIGHT   # 右寄せ
p.alignment = WD_ALIGN_PARAGRAPH.JUSTIFY # 両端揃え
```

### **単位の扱い**
```python
from docx.shared import Inches, Pt, Cm, Mm

# インチ単位
width = Inches(3)  # 3インチ

# ポイント単位（フォントサイズ等）
size = Pt(12)  # 12ポイント

# センチメートル単位
margin = Cm(2.5)  # 2.5cm

# ミリメートル単位
spacing = Mm(10)  # 10mm
```

---

## **🚀 クイックスタート**

### **最速実装（3ステップ）**

1. **HTMLテンプレートをコピー**
2. **タイトルを変更**（`<title>`と`<h1>`の2箇所）
3. **Pythonコードで文書内容を定義**

```python
# 最小限の変更例
doc = Document()
doc.add_heading('あなたのタイトルをここに', 0)
doc.add_paragraph('本文をここに記述')
# ... 残りは保存処理（テンプレート通り）
```

### **画像が必要な場合**

1. **プレースホルダーを配置**
```python
table = doc.add_table(rows=1, cols=1)
table.style = 'Table Grid'
table.cell(0, 0).text = '【画像】\nここに画像を配置'
```

2. **ユーザーがWordで画像を追加**
   - ダウンロードしたDOCXファイルを開く
   - プレースホルダーを削除
   - 「挿入」→「画像」で実際の画像を挿入

---

## **📝 更新履歴**

- **v3.0** (2025-01-17)
  - 実装ルールを優先順位別に再構成
  - 画像処理方針を明確化（すべてプレースホルダー）
  - エラーUI機能を追加（フルスクリーンオーバーレイ）
  - 実装例を4段階に拡充
  - Word特有の高度な機能を追加
  - エラー対処手順を完備

- **v2.1** (2025-01-17)
  - 外部ライブラリインストールの重要性を強調
  - 画像プレースホルダー機能を追加

- **v2.0** (2025-01-17)
  - 画像配置のサジェスチョン機能
  - インデントエラー防止の明確化

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
   - 特にpython-docxのインストール処理

4. **実装例を参考に**
   - レベル1の最小限実装から始める
   - 段階的に機能を追加

### **よくある質問**

**Q: 画像を直接埋め込みたい**
A: base64エンコードによるコンテキストウィンドウ超過を防ぐため、プレースホルダー方式を採用しています。

**Q: プレースホルダーの見た目を変えたい**
A: 表のスタイルや背景色を`table.style`や`shading_elm`で自由にカスタマイズ可能です。

**Q: 縦書きの文書を作りたい**
A: python-docxの制限により、縦書きは直接サポートされていません。Wordで開いた後に設定してください。

**Q: 目次を自動生成したい**
A: プレースホルダーを配置し、Wordの目次機能を使用してください。見出しスタイルが自動的に認識されます。

---

**これでレシピv3.0の全文です。AIが正しく理解し、適切なDOCX生成ページを作成できるよう、詳細な説明と実例を含めました。**