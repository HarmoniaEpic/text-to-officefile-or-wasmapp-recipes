# **XLSX自動生成ページ レシピ v1.1**

このドキュメントは、生成AIがユーザーの指示に基づき、「ワンクリック・ダウンロードページ」を自動生成するためのレシピ（指示書＋テンプレート）です。

AIはこのレシピに従い、最終的な成果物として単一のHTMLファイルを出力します。

## **🔴 必須要件 - 必ず最初に確認してください 🔴**

### **1. 外部ライブラリのインストール（絶対に省略禁止）**

**⚠️ 最重要事項：openpyxlライブラリのインストール処理は絶対に削除・省略してはいけません**

HTMLテンプレート内の以下のコードブロックは**必須**です：
```javascript
// このコードブロックは削除厳禁
await pyodide.loadPackage("micropip");
await pyodide.runPythonAsync(`
    import micropip
    await micropip.install('openpyxl')  # ← これがないとExcelファイルを生成できません
`);
```

**なぜ必要か？**
- openpyxlはExcelファイル（.xlsx）を生成するための外部ライブラリです
- ブラウザ環境では標準では利用できないため、micropipでインストールが必要です
- このインストール処理を削除すると、`ModuleNotFoundError: No module named 'openpyxl'`エラーが発生します

### **2. Pythonコードのインデント規則（エラー防止のため厳守）**

**⚠️ IndentationErrorを防ぐため、以下の規則を必ず守ってください**

- **scriptタグ内のPythonコードは、絶対に左端から開始すること**
- HTMLのインデントに影響されてPythonコードにインデントを追加してはいけない
- `<script type="text/python" id="python-code">` の直後の行から、インデントなしでPythonコードを記述する

**正しい例：**
```html
    <script type="text/python" id="python-code">
from openpyxl import Workbook  # ← 左端から開始（正しい）
import base64  # ← インデントなし（正しい）
```

**間違った例：**
```html
    <script type="text/python" id="python-code">
        from openpyxl import Workbook  # ← ❌ 不要なインデント（エラーの原因）
        import base64  # ← ❌ IndentationErrorが発生
```

## **📋 AIへの実装指示**

以下のHTML骨子テンプレートを使用し、指定された2箇所のみをユーザーの要求に応じて書き換えてください。**それ以外の部分（特にライブラリインストール部分）は一切変更してはいけません。**

### **書き換える箇所（2箇所のみ）**

1. **ページのタイトルと見出しを書き換える**  
   * `<!-- AIがここを書き換える: ページのタイトル -->` とコメントされている2箇所を、生成するExcelファイルの内容に合わせた具体的なタイトル（例: 売上レポートのダウンロード、在庫管理表のダウンロード）に書き換えてください。

2. **Pythonコードを書き換える**  
   * `<script type="text/python" id="python-code">` タグの内部を、要求されたExcelファイルを生成するための openpyxl コードに完全に置き換えてください。
   * **重要**: Pythonコードは必ず左端から開始し、HTMLのインデントを無視してください
   * このPythonコードは、必ず**最終行で `js.xlsx_base64_data` にBase64文字列を代入する**ルールに従う必要があります。

### **Pythonコード作成のルール**

1. **必須のインポート文**（左端から記述）
```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.chart import BarChart, LineChart, PieChart, Reference
from openpyxl.utils import get_column_letter
from openpyxl.formatting.rule import ColorScaleRule, DataBarRule
from io import BytesIO
import base64
import js
```

2. **Excelファイル作成の基本構造**
```python
# ワークブックの作成
wb = Workbook()
ws = wb.active
ws.title = "Sheet1"

# セルへの値の設定
ws['A1'] = 'タイトル'
ws['B1'] = 100
ws['C1'] = '=B1*2'  # 数式

# 複数セルへの値設定
for row in range(2, 5):
    ws[f'A{row}'] = f'データ{row-1}'
    ws[f'B{row}'] = row * 10

# スタイルの設定
ws['A1'].font = Font(bold=True, size=14, color="FF0000")
ws['A1'].fill = PatternFill(start_color="FFFF00", end_color="FFFF00", fill_type="solid")
ws['A1'].alignment = Alignment(horizontal="center", vertical="center")

# セルの結合
ws.merge_cells('A1:C1')

# 列幅の調整
ws.column_dimensions['A'].width = 20
ws.column_dimensions['B'].width = 15

# 新しいシートの追加
ws2 = wb.create_sheet("Sheet2")
```

3. **必須の終了処理**（必ず含める）
```python
# BytesIOに保存
xlsx_io = BytesIO()
wb.save(xlsx_io)
xlsx_io.seek(0)

# Base64エンコード
xlsx_bytes = xlsx_io.read()
xlsx_base64 = base64.b64encode(xlsx_bytes).decode('utf-8')

# JavaScriptに渡す（この行は必須）
js.xlsx_base64_data = xlsx_base64

print("XLSX file has been generated in memory.")
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
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen">

    <div class="bg-white rounded-2xl shadow-lg p-8 sm:p-12 text-center max-w-md w-full">
        <!-- AIがここを書き換える: ページのタイトル -->
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

    <!-- 
      AIがここを書き換える: openpyxlのコード
      重要: Pythonコードは左端から開始（インデントなし）
    -->
    <script type="text/python" id="python-code">
from openpyxl import Workbook
from io import BytesIO
import base64
import js

# ワークブックの作成
wb = Workbook()
ws = wb.active
ws.title = "サンプルデータ"

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

    <!-- 
      ⚠️⚠️⚠️ 警告: 以下のPyodideとopenpyxlインストール部分は削除厳禁 ⚠️⚠️⚠️
      この部分を削除すると、Excelファイルの生成が不可能になります
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

                // 🔴🔴🔴 削除厳禁: openpyxlライブラリのインストール 🔴🔴🔴
                // このコードブロックを削除するとModuleNotFoundErrorが発生します
                statusMessage.textContent = 'ライブラリをインストール中...';
                await pyodide.loadPackage("micropip");
                await pyodide.runPythonAsync(`
                    import micropip
                    await micropip.install('openpyxl')  # Excel操作に必須のライブラリ
                `);
                // 🔴🔴🔴 ここまで削除厳禁 🔴🔴🔴

                statusMessage.textContent = 'ファイルを生成しています...';
                const pythonCode = document.getElementById('python-code').textContent;
                // JavaScriptのグローバルスコープにデータを渡すための準備
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

## **⚠️ よくあるエラーと対処法**

### **ModuleNotFoundError: No module named 'openpyxl'**
**原因**: micropipでopenpyxlをインストールする処理が削除または省略されている  
**対処法**: HTMLテンプレートの中の `await micropip.install('openpyxl')` の部分を復元する

### **IndentationError: unexpected indent**
**原因**: Pythonコードに不要なインデントが含まれている  
**対処法**: すべてのPythonコードを左端から開始する

### **NameError: name 'js' is not defined**
**原因**: `import js` が抜けている  
**対処法**: 必須インポート文をすべて含める

### **ValueError: Cannot convert ... to Excel**
**原因**: Excelで扱えないデータ型を設定している  
**対処法**: 基本的なデータ型（文字列、数値、日付）を使用する

### **KeyError: 'Worksheet ... does not exist'**
**原因**: 存在しないシート名を参照している  
**対処法**: `wb.sheetnames` でシート名を確認

## **💡 実装例：売上レポートの作成**

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.chart import BarChart, Reference
from openpyxl.utils import get_column_letter
from io import BytesIO
import base64
import js
from datetime import date

wb = Workbook()
ws = wb.active
ws.title = "売上レポート"

# ヘッダーの設定
ws['A1'] = '2025年1月 売上レポート'
ws['A1'].font = Font(bold=True, size=16)
ws['A1'].alignment = Alignment(horizontal="center")
ws.merge_cells('A1:E1')

# 列ヘッダー
headers = ['日付', '商品名', '数量', '単価', '売上']
for idx, header in enumerate(headers, 1):
    cell = ws.cell(row=3, column=idx, value=header)
    cell.font = Font(bold=True, color="FFFFFF")
    cell.fill = PatternFill(start_color="366092", end_color="366092", fill_type="solid")
    cell.alignment = Alignment(horizontal="center")

# サンプルデータ
data = [
    ['2025/01/01', '商品A', 10, 1000, '=C4*D4'],
    ['2025/01/02', '商品B', 5, 2000, '=C5*D5'],
    ['2025/01/03', '商品C', 8, 1500, '=C6*D6'],
    ['2025/01/04', '商品A', 12, 1000, '=C7*D7'],
    ['2025/01/05', '商品B', 7, 2000, '=C8*D8'],
]

# データの入力
for row_idx, row_data in enumerate(data, 4):
    for col_idx, value in enumerate(row_data, 1):
        ws.cell(row=row_idx, column=col_idx, value=value)

# 罫線の設定
thin_border = Border(
    left=Side(style='thin'),
    right=Side(style='thin'),
    top=Side(style='thin'),
    bottom=Side(style='thin')
)

for row in ws.iter_rows(min_row=3, max_row=8, min_col=1, max_col=5):
    for cell in row:
        cell.border = thin_border

# 合計行の追加
ws['A10'] = '合計'
ws['A10'].font = Font(bold=True)
ws['E10'] = '=SUM(E4:E8)'
ws['E10'].font = Font(bold=True)
ws['E10'].fill = PatternFill(start_color="FFFF00", end_color="FFFF00", fill_type="solid")

# 列幅の調整
ws.column_dimensions['A'].width = 15
ws.column_dimensions['B'].width = 15
ws.column_dimensions['C'].width = 10
ws.column_dimensions['D'].width = 10
ws.column_dimensions['E'].width = 12

# グラフの追加
chart = BarChart()
chart.title = "日別売上"
chart.y_axis.title = '売上'
chart.x_axis.title = '日付'

# データ範囲の設定
data = Reference(ws, min_col=5, min_row=3, max_row=8, max_col=5)
categories = Reference(ws, min_col=1, min_row=4, max_row=8)
chart.add_data(data, titles_from_data=True)
chart.set_categories(categories)

# グラフの配置
ws.add_chart(chart, "G3")

# 2つ目のシート：集計
ws2 = wb.create_sheet("集計")
ws2['A1'] = '商品別集計'
ws2['A1'].font = Font(bold=True, size=14)

# 集計テーブル
ws2['A3'] = '商品名'
ws2['B3'] = '総数量'
ws2['C3'] = '総売上'

# 条件付き書式の例
ws2['A4'] = '商品A'
ws2['B4'] = 22
ws2['C4'] = 22000

ws2['A5'] = '商品B'
ws2['B5'] = 12
ws2['C5'] = 24000

ws2['A6'] = '商品C'
ws2['B6'] = 8
ws2['C6'] = 12000

# データバーの追加
from openpyxl.formatting.rule import DataBarRule
data_bar_rule = DataBarRule(
    start_type="num", start_value=0,
    end_type="max",
    color="638EC6"
)
ws2.conditional_formatting.add("C4:C6", data_bar_rule)

# 保存とBase64変換
xlsx_io = BytesIO()
wb.save(xlsx_io)
xlsx_io.seek(0)

xlsx_bytes = xlsx_io.read()
xlsx_base64 = base64.b64encode(xlsx_bytes).decode('utf-8')

js.xlsx_base64_data = xlsx_base64

print("XLSX file has been generated in memory.")
```

## **📚 openpyxlの主要機能リファレンス**

### **基本操作**
- `Workbook()`: 新規ワークブックの作成
- `wb.active`: アクティブシートの取得
- `wb.create_sheet(title)`: 新規シートの作成
- `ws['A1']`: セルへのアクセス
- `ws.cell(row=1, column=1)`: 行列番号でのアクセス

### **セルの書式設定**
```python
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side

# フォント
cell.font = Font(bold=True, italic=True, size=12, color="FF0000")

# 背景色
cell.fill = PatternFill(start_color="FFFF00", end_color="FFFF00", fill_type="solid")

# 配置
cell.alignment = Alignment(horizontal="center", vertical="center", wrap_text=True)

# 罫線
border = Border(left=Side(style='thin'), right=Side(style='thin'),
                top=Side(style='thin'), bottom=Side(style='thin'))
cell.border = border
```

### **数式**
```python
ws['C1'] = '=A1+B1'
ws['D1'] = '=SUM(A1:A10)'
ws['E1'] = '=AVERAGE(B1:B10)'
ws['F1'] = '=IF(A1>10,"大","小")'
```

### **セルの結合と分割**
```python
ws.merge_cells('A1:D1')  # 結合
ws.unmerge_cells('A1:D1')  # 分割
```

### **グラフの作成**
```python
from openpyxl.chart import BarChart, LineChart, PieChart, Reference

chart = BarChart()
chart.title = "タイトル"
data = Reference(ws, min_col=2, min_row=1, max_row=5, max_col=3)
chart.add_data(data, titles_from_data=True)
ws.add_chart(chart, "E5")
```

### **条件付き書式**
```python
from openpyxl.formatting.rule import ColorScaleRule, DataBarRule, IconSetRule

# カラースケール
color_scale_rule = ColorScaleRule(start_type="min", start_color="00FF00",
                                   end_type="max", end_color="FF0000")
ws.conditional_formatting.add("A1:A10", color_scale_rule)

# データバー
data_bar_rule = DataBarRule(start_type="num", start_value=0,
                            end_type="max", color="638EC6")
ws.conditional_formatting.add("B1:B10", data_bar_rule)
```

### **データの検証（ドロップダウンリスト）**
```python
from openpyxl.worksheet.datavalidation import DataValidation

dv = DataValidation(type="list", formula1='"選択肢1,選択肢2,選択肢3"', allow_blank=True)
ws.add_data_validation(dv)
dv.add('A1:A10')
```

## **✅ 最終チェックリスト**

AIがHTMLを生成する際の確認事項：

- [ ] **openpyxlのインストール処理（`await micropip.install('openpyxl')`）が含まれているか**
- [ ] Pythonコードは左端から開始されているか
- [ ] 必須のimport文がすべて含まれているか
- [ ] `js.xlsx_base64_data = xlsx_base64` の行が存在するか
- [ ] ページタイトルは適切に変更されているか
- [ ] MIMEタイプが正しく設定されているか（application/vnd.openxmlformats-officedocument.spreadsheetml.sheet）
- [ ] エラーメッセージの表示処理が含まれているか
- [ ] Pyodideのロード処理が含まれているか

## **🎯 応用例**

### **在庫管理表**
```python
# 在庫管理用のシート
ws['A1'] = '在庫管理表'
ws['A3'] = '商品コード'
ws['B3'] = '商品名'
ws['C3'] = '在庫数'
ws['D3'] = '最小在庫'
ws['E3'] = 'ステータス'

# 条件付き書式で在庫不足を強調
ws['E4'] = '=IF(C4<D4,"要発注","OK")'
```

### **請求書テンプレート**
```python
# 請求書のヘッダー
ws['A1'] = '請求書'
ws['A3'] = '請求先：'
ws['A4'] = '〇〇株式会社 御中'
ws['E3'] = '請求日：'
ws['F3'] = date.today().strftime('%Y年%m月%d日')

# 明細テーブル
ws['A7'] = '品目'
ws['D7'] = '数量'
ws['E7'] = '単価'
ws['F7'] = '金額'
```

### **月次レポート**
```python
# 複数シートで月次データを管理
months = ['1月', '2月', '3月']
for month in months:
    ws = wb.create_sheet(month)
    ws['A1'] = f'{month}売上'
    # 各月のデータを設定
```

### **ピボットテーブル風の集計**
```python
# データの集計
from collections import defaultdict
summary = defaultdict(lambda: {'count': 0, 'total': 0})

# 集計処理
for row in ws.iter_rows(min_row=2, values_only=True):
    category = row[0]
    value = row[1]
    summary[category]['count'] += 1
    summary[category]['total'] += value

# 集計結果の出力
ws_summary = wb.create_sheet("集計")
for idx, (key, values) in enumerate(summary.items(), 2):
    ws_summary[f'A{idx}'] = key
    ws_summary[f'B{idx}'] = values['count']
    ws_summary[f'C{idx}'] = values['total']
```

## **📝 更新履歴**

- **v1.1** (2025-01-17): 外部ライブラリインストールの重要性を強調
  - 必須要件セクションを冒頭に追加
  - HTMLテンプレート内にコメントで警告を追加
  - チェックリストにopenpyxl確認項目を追加
  - エラー対処法にModuleNotFoundErrorを追加
- **v1.0** (2025-01-17): PPTX-RECIPE-v2.0から派生して初版リリース
  - openpyxlに対応
  - Excelファイル生成機能の実装
  - グラフ、条件付き書式、データ検証機能を追加
  - インデントエラー防止機能を継承

## **📄 ライセンス**

本レシピはMITライセンスの下で提供されています。自由に使用、改変、再配布、商用利用が可能です。

## **🆘 トラブルシューティング**

もしAIがこのレシピを正しく実行できない場合：

1. **最初に必須要件セクションを確認** - 特にopenpyxlのインストール処理
2. **HTMLテンプレートの警告コメントを確認** - 削除厳禁の箇所が残っているか
3. **チェックリストを使用して検証** - すべての項目が満たされているか
4. **エラーメッセージを確認** - ModuleNotFoundErrorの場合はライブラリインストールの問題

**それでも問題が解決しない場合は、このレシピの最新版を確認してください。**