# **XLSX自動生成ページ レシピ v2.0**
**AIによるワンクリック・ダウンロードページ生成の完全ガイド**

---

## **📌 このレシピについて**

生成AIがユーザーの指示に基づき、Excelファイル（.xlsx）をブラウザ上で生成・ダウンロードできる**単一HTMLファイル**を自動作成するための完全な指示書です。

### **v2.0の特徴**
- 🎯 実装ルールを優先順位別に明確化
- 📊 CSVデータの自動処理に対応（オプション）
- 🔍 エラー対処の手順と実例を完備
- 💡 段階的な実装例で学習しやすい
- 📈 Excel特有の高度な機能を網羅

### **CSV処理について**
- CSVファイルの添付は**完全にオプション**です
- CSVがない場合は、通常のExcel生成が行われます
- CSVがある場合のみ、データの自動取り込みが実行されます
- どちらの場合でも正常に動作することを保証します

---

## **🚨 実装ルール（優先順位順）**

### **🔴 レベル1: 絶対禁止事項（変更・削除厳禁）**

以下の要素は**一切変更してはいけません**：

#### **1.1 openpyxlライブラリのインストール処理**
```javascript
await pyodide.loadPackage("micropip");
await pyodide.runPythonAsync(`
    import micropip
    await micropip.install('openpyxl')
`);
```

**なぜ絶対に必要か：**
- openpyxlはExcelファイル生成のための外部ライブラリ
- ブラウザのPyodide環境には標準で含まれていない
- micropipを使用してランタイムでインストールする必要がある
- **削除すると必ず発生するエラー**: `ModuleNotFoundError: No module named 'openpyxl'`

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
from openpyxl import Workbook
import base64

# ❌ 間違い（インデントあり）
    from openpyxl import Workbook  # IndentationError発生
    import base64
```

**理由**: HTMLのインデントに影響されてPythonコードにインデントを追加するとエラーになる

#### **2.2 必須のインポート文**
```python
from openpyxl import Workbook
from io import BytesIO
import base64
import js  # JavaScriptとの連携に必須
```

**省略した場合のエラー**: `NameError: name 'js' is not defined`

#### **2.3 Base64データの受け渡し**
Pythonコードの最終部で必ず実行：
```python
js.xlsx_base64_data = xlsx_base64
```

**理由**: JavaScriptがXLSXデータを受け取るための必須処理

### **🟢 レベル3: カスタマイズ可能要素**

以下は自由に変更できます：
- ページタイトル（`<title>`タグと`<h1>`タグ）
- Pythonコード内のExcel内容
- シートのレイアウト・デザイン
- ダウンロードファイル名
- グラフや集計の種類

---

## **📊 CSVデータ処理機能（オプション）**

### **基本方針**
CSVファイルが添付された場合、自動的にデータを読み込んでExcelに変換します。添付がない場合は、通常のExcel生成を行います。

### **CSV処理の流れ**
1. CSVデータの存在確認
2. 文字コードの自動判定（UTF-8、Shift-JIS対応）
3. ヘッダー行の認識
4. データ型の自動変換（数値、日付、文字列）
5. Excelシートへの展開

### **実装コード例**
```python
# CSVデータの確認（存在しない場合はNone）
csv_content = None
try:
    csv_content = js.csv_content if hasattr(js, 'csv_content') else None
except:
    csv_content = None

# CSVの有無で処理を分岐
if csv_content:
    # CSVデータを処理
    import csv
    import io
    
    # CSVを解析
    reader = csv.reader(io.StringIO(csv_content))
    headers = next(reader, None)
    
    if headers:
        # ヘッダーを設定
        for col, header in enumerate(headers, 1):
            ws.cell(row=1, column=col, value=header)
            # ヘッダーの書式設定
            cell = ws.cell(row=1, column=col)
            cell.font = Font(bold=True)
            cell.fill = PatternFill(start_color="366092", end_color="366092", fill_type="solid")
        
        # データを設定
        for row_idx, row_data in enumerate(reader, 2):
            for col_idx, value in enumerate(row_data, 1):
                # 数値変換を試みる
                try:
                    # 整数チェック
                    if '.' not in value:
                        numeric_value = int(value)
                    else:
                        numeric_value = float(value)
                    ws.cell(row=row_idx, column=col_idx, value=numeric_value)
                except ValueError:
                    # 文字列として設定
                    ws.cell(row=row_idx, column=col_idx, value=value)
else:
    # CSVがない場合の通常処理
    ws['A1'] = 'データ'
    ws['B1'] = '値'
    # 通常のデータ設定を続ける
```

### **CSV処理のエラーハンドリング**
```python
def safe_csv_process(csv_content):
    """安全なCSV処理"""
    try:
        # CSV処理
        process_csv_data(csv_content)
    except UnicodeDecodeError:
        # 文字コードエラー
        try:
            # Shift-JISとして再試行
            csv_content = csv_content.encode('shift-jis').decode('utf-8')
            process_csv_data(csv_content)
        except:
            # エラーメッセージを設定
            ws['A1'] = 'CSVの読み込みに失敗しました'
    except Exception as e:
        # その他のエラー
        ws['A1'] = f'エラー: {str(e)}'
```

---

## **📝 HTMLテンプレート（完全版）**

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- カスタマイズ可能: ページタイトル -->
    <title>Excelファイルのダウンロード</title>

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
            Excelファイル
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
from openpyxl import Workbook
from io import BytesIO
import base64
import js

# ワークブックの作成
wb = Workbook()
ws = wb.active
ws.title = "データ"

# データの追加
ws['A1'] = 'AIによって自動生成'
ws['A2'] = 'このExcelファイルは動的に作成されました'

# BytesIOに保存
xlsx_io = BytesIO()
wb.save(xlsx_io)
xlsx_io.seek(0)

# Base64エンコード
xlsx_bytes = xlsx_io.read()
xlsx_base64 = base64.b64encode(xlsx_bytes).decode('utf-8')

# JavaScriptに渡す（必須）
js.xlsx_base64_data = xlsx_base64

print("XLSX file has been generated in memory.")
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

                // openpyxlのインストール（削除禁止）
                statusMessage.textContent = 'ライブラリをインストール中...';
                await pyodide.loadPackage("micropip");
                await pyodide.runPythonAsync(`
                    import micropip
                    await micropip.install('openpyxl')
                `);

                statusMessage.textContent = 'ファイルを生成しています...';
                const pythonCode = document.getElementById('python-code').textContent;
                window.xlsx_base64_data = null; 
                await pyodide.runPythonAsync(pythonCode);
                const base64Data = window.xlsx_base64_data;

                if (!base64Data) {
                    throw new Error("Python code did not produce any data.");
                }

                statusMessage.textContent = '準備が完了しました！';
                loadingSpinner.style.display = 'none';
                downloadButton.disabled = false;
                downloadButton.addEventListener('click', () => downloadFile(base64Data, 'spreadsheet.xlsx'));

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
                const blob = new Blob([byteArray], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });

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
ModuleNotFoundError: No module named 'openpyxl'
```
**原因**: openpyxlのインストール処理が削除されている
**解決**: `await micropip.install('openpyxl')` を復元

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

#### **4. ValueError**
```
ValueError: Cannot convert ... to Excel
```
**原因**: Excelで扱えないデータ型を設定
**解決**: 基本的なデータ型（文字列、数値、日付）を使用

#### **5. CSV関連のエラー**
```
UnicodeDecodeError: 'utf-8' codec can't decode
```
**原因**: CSVファイルの文字コードが異なる
**解決**: エラーハンドリングで自動的にShift-JISとして再試行

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
from openpyxl import Workbook
from io import BytesIO
import base64
import js

wb = Workbook()
ws = wb.active
ws['A1'] = 'シンプルな表'
ws['A2'] = 'データ1'
ws['B2'] = 100

xlsx_io = BytesIO()
wb.save(xlsx_io)
xlsx_io.seek(0)
js.xlsx_base64_data = base64.b64encode(xlsx_io.read()).decode('utf-8')
```

### **レベル2: 基本的な表計算**
```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.utils import get_column_letter
from io import BytesIO
import base64
import js

wb = Workbook()
ws = wb.active
ws.title = "売上表"

# ヘッダー設定
headers = ['日付', '商品名', '数量', '単価', '売上']
for idx, header in enumerate(headers, 1):
    cell = ws.cell(row=1, column=idx, value=header)
    cell.font = Font(bold=True, color="FFFFFF")
    cell.fill = PatternFill(start_color="366092", end_color="366092", fill_type="solid")
    cell.alignment = Alignment(horizontal="center")

# データ入力
data = [
    ['2025/01/01', '商品A', 10, 1000],
    ['2025/01/02', '商品B', 5, 2000],
    ['2025/01/03', '商品C', 8, 1500],
]

for row_idx, row_data in enumerate(data, 2):
    for col_idx, value in enumerate(row_data, 1):
        ws.cell(row=row_idx, column=col_idx, value=value)
    # 売上計算式
    ws.cell(row=row_idx, column=5, value=f'=C{row_idx}*D{row_idx}')

# 合計行
last_row = len(data) + 2
ws[f'A{last_row}'] = '合計'
ws[f'A{last_row}'].font = Font(bold=True)
ws[f'E{last_row}'] = f'=SUM(E2:E{last_row-1})'
ws[f'E{last_row}'].font = Font(bold=True)
ws[f'E{last_row}'].fill = PatternFill(start_color="FFFF00", end_color="FFFF00", fill_type="solid")

# 罫線設定
thin_border = Border(
    left=Side(style='thin'),
    right=Side(style='thin'),
    top=Side(style='thin'),
    bottom=Side(style='thin')
)

for row in ws.iter_rows(min_row=1, max_row=last_row, min_col=1, max_col=5):
    for cell in row:
        cell.border = thin_border

# 列幅調整
for col in range(1, 6):
    ws.column_dimensions[get_column_letter(col)].width = 15

# 保存
xlsx_io = BytesIO()
wb.save(xlsx_io)
xlsx_io.seek(0)
js.xlsx_base64_data = base64.b64encode(xlsx_io.read()).decode('utf-8')
```

### **レベル3: CSVデータのインポートと処理**
```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment
from openpyxl.chart import BarChart, Reference
from io import BytesIO
import base64
import js
import csv
import io

wb = Workbook()
ws = wb.active
ws.title = "CSVデータ"

# CSVデータの確認（この例では存在すると仮定）
csv_content = None
try:
    csv_content = js.csv_content if hasattr(js, 'csv_content') else None
except:
    csv_content = None

if csv_content:
    # CSVデータが存在する場合
    try:
        # CSVを解析
        reader = csv.reader(io.StringIO(csv_content))
        headers = next(reader, None)
        
        if headers:
            # ヘッダーを設定
            for col, header in enumerate(headers, 1):
                cell = ws.cell(row=1, column=col, value=header)
                cell.font = Font(bold=True, color="FFFFFF")
                cell.fill = PatternFill(start_color="366092", end_color="366092", fill_type="solid")
                cell.alignment = Alignment(horizontal="center")
            
            # データを設定
            for row_idx, row_data in enumerate(reader, 2):
                for col_idx, value in enumerate(row_data, 1):
                    # 数値変換を試みる
                    try:
                        if '.' not in value:
                            numeric_value = int(value)
                        else:
                            numeric_value = float(value)
                        ws.cell(row=row_idx, column=col_idx, value=numeric_value)
                    except ValueError:
                        ws.cell(row=row_idx, column=col_idx, value=value)
            
            # データ範囲を取得
            max_row = ws.max_row
            max_col = ws.max_column
            
            # 合計行を追加（数値列のみ）
            ws.cell(row=max_row + 2, column=1, value="合計")
            ws.cell(row=max_row + 2, column=1).font = Font(bold=True)
            
            for col in range(2, max_col + 1):
                # 数値列かチェック
                is_numeric = True
                for row in range(2, max_row + 1):
                    cell_value = ws.cell(row=row, column=col).value
                    if cell_value is not None and not isinstance(cell_value, (int, float)):
                        is_numeric = False
                        break
                
                if is_numeric:
                    col_letter = get_column_letter(col)
                    sum_cell = ws.cell(row=max_row + 2, column=col)
                    sum_cell.value = f'=SUM({col_letter}2:{col_letter}{max_row})'
                    sum_cell.font = Font(bold=True)
                    sum_cell.fill = PatternFill(start_color="FFFF00", end_color="FFFF00", fill_type="solid")
            
            # グラフを追加（データが3列以上ある場合）
            if max_col >= 3 and max_row >= 3:
                chart = BarChart()
                chart.title = "データグラフ"
                chart.y_axis.title = headers[1] if len(headers) > 1 else 'Value'
                chart.x_axis.title = headers[0] if headers else 'Category'
                
                # データ範囲（2列目を使用）
                data = Reference(ws, min_col=2, min_row=1, max_row=max_row, max_col=2)
                categories = Reference(ws, min_col=1, min_row=2, max_row=max_row)
                chart.add_data(data, titles_from_data=True)
                chart.set_categories(categories)
                
                ws.add_chart(chart, f"{get_column_letter(max_col + 2)}3")
                
    except Exception as e:
        # エラー処理
        ws['A1'] = 'CSVデータの処理中にエラーが発生しました'
        ws['A2'] = str(e)
else:
    # CSVデータがない場合のデフォルト処理
    ws['A1'] = 'サンプルデータ'
    ws['A1'].font = Font(bold=True, size=14)
    
    # サンプルデータを作成
    headers = ['項目', '数値1', '数値2', '数値3']
    for idx, header in enumerate(headers, 1):
        cell = ws.cell(row=3, column=idx, value=header)
        cell.font = Font(bold=True)
        cell.fill = PatternFill(start_color="D9E1F2", end_color="D9E1F2", fill_type="solid")
    
    sample_data = [
        ['データA', 100, 150, 200],
        ['データB', 200, 250, 300],
        ['データC', 150, 200, 250],
    ]
    
    for row_idx, row_data in enumerate(sample_data, 4):
        for col_idx, value in enumerate(row_data, 1):
            ws.cell(row=row_idx, column=col_idx, value=value)

# 列幅を自動調整
for column in ws.columns:
    max_length = 0
    column_letter = get_column_letter(column[0].column)
    for cell in column:
        if cell.value:
            max_length = max(max_length, len(str(cell.value)))
    adjusted_width = min(max_length + 2, 50)
    ws.column_dimensions[column_letter].width = adjusted_width

# 保存
xlsx_io = BytesIO()
wb.save(xlsx_io)
xlsx_io.seek(0)
js.xlsx_base64_data = base64.b64encode(xlsx_io.read()).decode('utf-8')
```

### **レベル4: 高度な分析と複数シート**
```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.chart import BarChart, PieChart, LineChart, Reference
from openpyxl.formatting.rule import ColorScaleRule, DataBarRule
from openpyxl.utils import get_column_letter
from datetime import date, timedelta
from io import BytesIO
import base64
import js

def create_summary_sheet(wb, data_sheet):
    """集計シートを作成"""
    summary_ws = wb.create_sheet("集計")
    summary_ws['A1'] = '統計情報'
    summary_ws['A1'].font = Font(bold=True, size=14)
    
    # 統計情報の計算
    stats = [
        ['項目', '値'],
        ['データ数', f'=COUNTA({data_sheet.title}!B:B)-1'],
        ['合計', f'=SUM({data_sheet.title}!D:D)'],
        ['平均', f'=AVERAGE({data_sheet.title}!D:D)'],
        ['最大値', f'=MAX({data_sheet.title}!D:D)'],
        ['最小値', f'=MIN({data_sheet.title}!D:D)'],
    ]
    
    for row_idx, (label, formula) in enumerate(stats, 3):
        summary_ws.cell(row=row_idx, column=1, value=label)
        summary_ws.cell(row=row_idx, column=2, value=formula)
        
        if row_idx == 3:
            # ヘッダー行の書式
            summary_ws.cell(row=row_idx, column=1).font = Font(bold=True)
            summary_ws.cell(row=row_idx, column=2).font = Font(bold=True)
            summary_ws.cell(row=row_idx, column=1).fill = PatternFill(
                start_color="366092", end_color="366092", fill_type="solid"
            )
            summary_ws.cell(row=row_idx, column=2).fill = PatternFill(
                start_color="366092", end_color="366092", fill_type="solid"
            )
    
    # 条件付き書式（データバー）
    data_bar_rule = DataBarRule(
        start_type="num", start_value=0,
        end_type="max", color="638EC6"
    )
    summary_ws.conditional_formatting.add("B4:B8", data_bar_rule)
    
    return summary_ws

wb = Workbook()

# メインデータシート
ws = wb.active
ws.title = "売上データ"

# タイトル
ws['A1'] = '月次売上レポート'
ws['A1'].font = Font(bold=True, size=16)
ws.merge_cells('A1:E1')

# ヘッダー
headers = ['日付', '担当者', '商品', '数量', '売上']
for idx, header in enumerate(headers, 1):
    cell = ws.cell(row=3, column=idx, value=header)
    cell.font = Font(bold=True, color="FFFFFF")
    cell.fill = PatternFill(start_color="366092", end_color="366092", fill_type="solid")
    cell.alignment = Alignment(horizontal="center")

# サンプルデータ生成
base_date = date.today() - timedelta(days=30)
products = ['製品A', '製品B', '製品C', '製品D']
staff = ['田中', '佐藤', '鈴木', '高橋']

data = []
for i in range(30):
    import random
    data.append([
        (base_date + timedelta(days=i)).strftime('%Y/%m/%d'),
        random.choice(staff),
        random.choice(products),
        random.randint(5, 50),
        random.randint(10000, 100000)
    ])

# データ入力
for row_idx, row_data in enumerate(data, 4):
    for col_idx, value in enumerate(row_data, 1):
        ws.cell(row=row_idx, column=col_idx, value=value)

# 罫線設定
thin_border = Border(
    left=Side(style='thin'),
    right=Side(style='thin'),
    top=Side(style='thin'),
    bottom=Side(style='thin')
)

for row in ws.iter_rows(min_row=3, max_row=len(data)+3, min_col=1, max_col=5):
    for cell in row:
        cell.border = thin_border

# 条件付き書式（売上高に色スケール）
color_scale_rule = ColorScaleRule(
    start_type="min", start_color="63BE7B",
    mid_type="percentile", mid_value=50, mid_color="FFEB9C",
    end_type="max", end_color="F8696B"
)
ws.conditional_formatting.add(f"E4:E{len(data)+3}", color_scale_rule)

# 列幅調整
column_widths = [12, 10, 10, 8, 12]
for idx, width in enumerate(column_widths, 1):
    ws.column_dimensions[get_column_letter(idx)].width = width

# グラフシート
chart_ws = wb.create_sheet("グラフ")

# 売上推移グラフ
line_chart = LineChart()
line_chart.title = "日別売上推移"
line_chart.y_axis.title = "売上"
line_chart.x_axis.title = "日付"
line_chart.width = 20
line_chart.height = 10

data_ref = Reference(ws, min_col=5, min_row=3, max_row=len(data)+3)
dates_ref = Reference(ws, min_col=1, min_row=4, max_row=len(data)+3)
line_chart.add_data(data_ref, titles_from_data=True)
line_chart.set_categories(dates_ref)

chart_ws.add_chart(line_chart, "A1")

# 担当者別集計シート
staff_ws = wb.create_sheet("担当者別")
staff_ws['A1'] = '担当者別売上集計'
staff_ws['A1'].font = Font(bold=True, size=14)

# 担当者別の集計（簡易版）
staff_summary = {}
for row in data:
    staff_name = row[1]
    sales = row[4]
    if staff_name in staff_summary:
        staff_summary[staff_name] += sales
    else:
        staff_summary[staff_name] = sales

# 集計結果を入力
staff_ws['A3'] = '担当者'
staff_ws['B3'] = '売上合計'
staff_ws['A3'].font = Font(bold=True)
staff_ws['B3'].font = Font(bold=True)

for idx, (name, total) in enumerate(staff_summary.items(), 4):
    staff_ws.cell(row=idx, column=1, value=name)
    staff_ws.cell(row=idx, column=2, value=total)

# 円グラフ
pie_chart = PieChart()
pie_chart.title = "担当者別売上構成"
data_ref = Reference(staff_ws, min_col=2, min_row=3, max_row=3+len(staff_summary))
labels_ref = Reference(staff_ws, min_col=1, min_row=4, max_row=3+len(staff_summary))
pie_chart.add_data(data_ref, titles_from_data=True)
pie_chart.set_categories(labels_ref)

staff_ws.add_chart(pie_chart, "D3")

# 集計シートを作成
summary_sheet = create_summary_sheet(wb, ws)

# 保存
xlsx_io = BytesIO()
wb.save(xlsx_io)
xlsx_io.seek(0)
js.xlsx_base64_data = base64.b64encode(xlsx_io.read()).decode('utf-8')

print("Advanced XLSX file with multiple sheets has been generated.")
```

---

## **📚 Excel特有の機能**

### **数式と関数**
```python
# 基本的な数式
ws['C1'] = '=A1+B1'          # 加算
ws['C2'] = '=A2-B2'          # 減算
ws['C3'] = '=A3*B3'          # 乗算
ws['C4'] = '=A4/B4'          # 除算

# 集計関数
ws['D1'] = '=SUM(A1:A10)'    # 合計
ws['D2'] = '=AVERAGE(B1:B10)' # 平均
ws['D3'] = '=MAX(C1:C10)'    # 最大値
ws['D4'] = '=MIN(D1:D10)'    # 最小値
ws['D5'] = '=COUNT(E1:E10)'  # 個数

# 条件付き関数
ws['E1'] = '=IF(A1>100,"大","小")'              # IF文
ws['E2'] = '=SUMIF(A:A,">100",B:B)'            # 条件付き合計
ws['E3'] = '=COUNTIF(A:A,">=50")'              # 条件付きカウント
ws['E4'] = '=VLOOKUP(A1,F:G,2,FALSE)'          # VLOOKUP
```

### **データ検証（ドロップダウンリスト）**
```python
from openpyxl.worksheet.datavalidation import DataValidation

# ドロップダウンリストの作成
dv = DataValidation(
    type="list",
    formula1='"選択肢1,選択肢2,選択肢3"',
    allow_blank=True
)
dv.error = 'リストから選択してください'
dv.errorTitle = '入力エラー'

ws.add_data_validation(dv)
dv.add('A1:A10')  # A1からA10にドロップダウンを適用
```

### **条件付き書式**
```python
from openpyxl.formatting.rule import ColorScaleRule, DataBarRule, IconSetRule

# カラースケール（値に応じて色のグラデーション）
color_scale_rule = ColorScaleRule(
    start_type="min", start_color="00FF00",  # 最小値は緑
    end_type="max", end_color="FF0000"       # 最大値は赤
)
ws.conditional_formatting.add("A1:A10", color_scale_rule)

# データバー（セル内に棒グラフを表示）
data_bar_rule = DataBarRule(
    start_type="num", start_value=0,
    end_type="max", color="638EC6"
)
ws.conditional_formatting.add("B1:B10", data_bar_rule)

# アイコンセット（値に応じてアイコンを表示）
from openpyxl.formatting.rule import Rule
from openpyxl.styles.differential import DifferentialStyle
from openpyxl.formatting import Rule

# 特定の値以上を強調
red_fill = PatternFill(start_color="FF0000", end_color="FF0000", fill_type="solid")
dxf = DifferentialStyle(fill=red_fill)
rule = Rule(type="expression", dxf=dxf)
rule.formula = ["$A1>100"]
ws.conditional_formatting.add("A1:A10", rule)
```

### **グラフの作成**
```python
from openpyxl.chart import BarChart, LineChart, PieChart, ScatterChart, Reference

# 棒グラフ
bar_chart = BarChart()
bar_chart.title = "売上グラフ"
bar_chart.y_axis.title = "売上"
bar_chart.x_axis.title = "月"

data = Reference(ws, min_col=2, min_row=1, max_row=10, max_col=3)
categories = Reference(ws, min_col=1, min_row=2, max_row=10)
bar_chart.add_data(data, titles_from_data=True)
bar_chart.set_categories(categories)

ws.add_chart(bar_chart, "E5")

# 折れ線グラフ
line_chart = LineChart()
line_chart.title = "推移グラフ"
line_chart.style = 13  # スタイル番号

# 円グラフ
pie_chart = PieChart()
pie_chart.title = "構成比"

# 散布図
scatter_chart = ScatterChart()
scatter_chart.title = "相関図"
scatter_chart.x_axis.title = "X軸"
scatter_chart.y_axis.title = "Y軸"
```

### **複数シート間の参照**
```python
# 別シートの値を参照
ws['A1'] = '=Sheet2!A1'           # Sheet2のA1セルを参照
ws['B1'] = '=SUM(Sheet2!A:A)'     # Sheet2のA列を合計
ws['C1'] = "='売上データ'!B2"     # シート名にスペースがある場合

# 3D参照（複数シートをまとめて参照）
ws['D1'] = '=SUM(Sheet1:Sheet3!A1)'  # Sheet1からSheet3のA1を合計
```

### **大量データの効率的な処理**
```python
# メモリ効率的な書き込み
def write_large_dataset(ws, data):
    """大量データを効率的に書き込む"""
    # append()を使用すると高速
    for row in data:
        ws.append(row)
    
    # または、範囲を指定して一括書き込み
    from openpyxl.utils import range_boundaries
    for row_idx, row_data in enumerate(data, 1):
        for col_idx, value in enumerate(row_data, 1):
            ws.cell(row=row_idx, column=col_idx, value=value)

# write_only モードで大量データ処理
wb = Workbook(write_only=True)
ws = wb.create_sheet()

# ストリーミング書き込み
for row in large_dataset:
    ws.append(row)

wb.save('large_file.xlsx')
```

---

## **✅ 実装チェックリスト**

### **必須確認項目**
- [ ] openpyxlインストール処理が存在する
- [ ] Pythonコードが左端から開始されている
- [ ] `import js` が含まれている
- [ ] `js.xlsx_base64_data` への代入がある
- [ ] エラーオーバーレイのHTMLが完全である

### **CSV処理確認項目**
- [ ] CSV処理はオプションとして実装されている
- [ ] CSVがない場合も正常に動作する
- [ ] 文字コードエラーの処理が含まれている
- [ ] データ型の自動変換が実装されている

### **動作確認項目**
- [ ] ページが正常に読み込まれる
- [ ] ダウンロードボタンが有効になる
- [ ] XLSXファイルがダウンロードできる
- [ ] エラー時にオーバーレイが表示される
- [ ] エラーコピーボタンが機能する

---

## **📊 openpyxlリファレンス**

### **基本操作**
| メソッド | 用途 | 例 |
|---------|------|-----|
| `Workbook()` | 新規ワークブック作成 | `wb = Workbook()` |
| `wb.active` | アクティブシート取得 | `ws = wb.active` |
| `wb.create_sheet(title)` | 新規シート作成 | `ws2 = wb.create_sheet("Sheet2")` |
| `ws['A1']` | セルアクセス | `ws['A1'] = 100` |
| `ws.cell(row, column)` | 行列指定アクセス | `ws.cell(1, 1, value=100)` |
| `ws.append(list)` | 行追加 | `ws.append([1, 2, 3])` |
| `ws.merge_cells()` | セル結合 | `ws.merge_cells('A1:D1')` |

### **書式設定**
```python
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side

# フォント
cell.font = Font(
    bold=True,           # 太字
    italic=True,         # 斜体
    size=12,            # サイズ
    color="FF0000"      # 色（赤）
)

# 背景色
cell.fill = PatternFill(
    start_color="FFFF00",
    end_color="FFFF00",
    fill_type="solid"
)

# 配置
cell.alignment = Alignment(
    horizontal="center",  # 水平方向
    vertical="center",    # 垂直方向
    wrap_text=True       # 折り返し
)

# 罫線
border = Border(
    left=Side(style='thin'),
    right=Side(style='thin'),
    top=Side(style='thin'),
    bottom=Side(style='thin')
)
cell.border = border
```

### **数値フォーマット**
```python
# 通貨
cell.number_format = '#,##0'           # 1,000
cell.number_format = '¥#,##0'         # ¥1,000
cell.number_format = '#,##0.00'       # 1,000.00

# パーセント
cell.number_format = '0%'             # 10%
cell.number_format = '0.00%'          # 10.00%

# 日付
cell.number_format = 'yyyy/mm/dd'     # 2025/01/17
cell.number_format = 'yyyy年mm月dd日'  # 2025年01月17日
```

---

## **🚀 クイックスタート**

### **最速実装（3ステップ）**

1. **HTMLテンプレートをコピー**
2. **タイトルを変更**（`<title>`と`<h1>`の2箇所）
3. **Pythonコードで表の内容を定義**

```python
# 最小限の変更例
wb = Workbook()
ws = wb.active
ws['A1'] = 'あなたのデータをここに'
ws['B1'] = 100
# ... 残りは保存処理（テンプレート通り）
```

### **CSVデータを使用する場合**

1. **CSVファイルを準備**
```csv
商品名,数量,単価,売上
商品A,10,1000,10000
商品B,5,2000,10000
```

2. **AIに依頼**
```
CSVファイルを添付して、
「添付したCSVからグラフ付きレポートを作成してください」
```

---

## **📝 更新履歴**

- **v2.0** (2025-01-17)
  - 実装ルールを優先順位別に再構成
  - CSV処理機能を完全実装（オプション）
  - エラーUI機能を追加（フルスクリーンオーバーレイ）
  - 実装例を4段階に拡充
  - Excel特有の高度な機能を追加

- **v1.1** (2025-01-17)
  - 外部ライブラリインストールの重要性を強調
  - エラー対処法を追加

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
   - 特にopenpyxlのインストール処理

4. **実装例を参考に**
   - レベル1の最小限実装から始める
   - 段階的に機能を追加

### **よくある質問**

**Q: CSVファイルは必須ですか？**
A: いいえ、CSVファイルは完全にオプションです。CSVがなくても正常に動作します。

**Q: 大量のデータを処理できますか？**
A: はい、数万行程度のデータであれば処理可能です。それ以上の場合は、write_onlyモードの使用を検討してください。

**Q: 日本語のCSVが文字化けします**
A: 文字コードの自動判定機能により、UTF-8とShift-JISの両方に対応しています。それでも文字化けする場合は、CSVをUTF-8で保存し直してください。

**Q: 複雑な数式は使えますか？**
A: はい、ExcelのほとんどのVLOOKUP、SUMIF、INDEX/MATCH等が使用できます。

---

**これでレシピv2.0の全文です。AIが正しく理解し、CSVの有無に関わらず適切なXLSX生成ページを作成できるよう、詳細な説明と実例を含めました。**