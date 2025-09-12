# Office文書自動生成システム - プロジェクトメモリ

## プロジェクト概要
このプロジェクトは、AIを使用してOffice文書（Word、Excel、PowerPoint）とPDFを自動生成するシステムです。レシピファイル（.md）をAIに渡すことで、ワンクリックでダウンロード可能なHTMLファイルを生成します。

## 重要な制約事項（必須遵守）

### 🔴 絶対に削除・変更してはいけない部分
1. **ライブラリインストール処理**
   ```javascript
   await micropip.install('python-docx')  # Word用
   await micropip.install('openpyxl')     # Excel用
   await micropip.install('python-pptx')  # PowerPoint用
   await micropip.install('reportlab')    # PDF用
   ```
   これらは削除するとModuleNotFoundErrorが発生します

2. **Pythonコードのインデント規則**
   - scriptタグ内のPythonコードは**必ず左端から開始**
   - HTMLのインデントに影響されてインデントを追加しない

3. **Base64データの受け渡し**
   - 必ず最終行で `js.xxx_base64_data = base64` を実行
   - Word: `js.docx_base64_data`
   - Excel: `js.xlsx_base64_data`
   - PowerPoint: `js.pptx_base64_data`
   - PDF: `js.pdf_base64_data`

## レシピファイルと用途

| ファイル形式 | レシピファイル | ライブラリ | 用途 |
|------------|--------------|-----------|------|
| Word (.docx) | DOCX-RECIPE.md | python-docx | 報告書、契約書、マニュアル |
| Excel (.xlsx) | XLSX-RECIPE.md | openpyxl | 売上表、予算管理、データ分析 |
| PowerPoint (.pptx) | PPTX-RECIPE.md | python-pptx | プレゼン、提案書 |
| PDF (.pdf) | PDF-RECIPE.md | reportlab | 請求書、証明書、印刷物 |

## よくあるワークフロー

### 新しい文書生成を依頼された場合
1. ユーザーが作りたいファイル形式を確認
2. 対応するレシピファイルの添付を確認
3. レシピファイルの指示に従ってHTMLを生成
4. **必須要件（ライブラリインストール、インデント、Base64）を必ず守る**

### CSVデータからExcel生成の場合
1. XLSX-RECIPE.mdとCSVファイルの両方が添付されているか確認
2. CSVデータを読み込んでDataFrameに変換
3. openpyxlでExcelファイルを生成
4. グラフや条件付き書式を適宜追加

## コード生成時のテンプレート

### 基本構造（全形式共通）
```python
# 1. 必須インポート（左端から開始）
from [ライブラリ] import [クラス]
from io import BytesIO
import base64
import js

# 2. 文書生成
[文書オブジェクト作成]
[コンテンツ追加]

# 3. 必須の終了処理
buffer = BytesIO()
[保存処理](buffer)
buffer.seek(0)

bytes_data = buffer.read()
base64_data = base64.b64encode(bytes_data).decode('utf-8')

# 4. JavaScriptへの受け渡し（必須）
js.[形式]_base64_data = base64_data
```

## よくあるエラーと対処法

| エラー | 原因 | 対処法 |
|--------|------|--------|
| ModuleNotFoundError | ライブラリインストール削除 | micropip.installを復元 |
| IndentationError | 不要なインデント | Pythonコードを左端から開始 |
| NameError: 'js' | import js忘れ | 必須インポートを確認 |
| No data produced | Base64代入忘れ | js.xxx_base64_data行を追加 |

## デバッグ用コマンド
- メモリファイルの確認: `/memory`
- エラー詳細の確認: ブラウザコンソールを参照するよう案内

## プロジェクト固有の注意事項

### 画像プレースホルダー機能
- v2.0以降のレシピは画像プレースホルダー機能対応
- 画像は埋め込まず、配置位置と推奨サイズを提案
- プレースホルダーには必要情報（ファイル名、用途、サイズ）を含める

### パフォーマンス考慮事項
- 初回ロード: 2-5秒（Pyodide + ライブラリ）
- 最大文書サイズ: ~50MB（ブラウザ制限）
- 大きなデータセットは分割処理を推奨

## よく使うコードスニペット

### Excelグラフ追加
```python
from openpyxl.chart import BarChart, Reference
chart = BarChart()
chart.title = "タイトル"
data = Reference(ws, min_col=2, min_row=1, max_row=5, max_col=3)
chart.add_data(data, titles_from_data=True)
ws.add_chart(chart, "E5")
```

### Word表追加
```python
table = doc.add_table(rows=2, cols=3)
table.style = 'Light Grid Accent 1'
table.cell(0, 0).text = 'ヘッダー'
```

### PowerPointスライド追加
```python
slide_layout = prs.slide_layouts[0]  # 0=タイトル, 1=タイトル+コンテンツ
slide = prs.slides.add_slide(slide_layout)
slide.shapes.title.text = "タイトル"
```

### PDF日本語フォント設定
```python
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.cidfonts import UnicodeCIDFont
pdfmetrics.registerFont(UnicodeCIDFont('HeiseiKakuGo-W5'))
styles['Normal'].fontName = 'HeiseiKakuGo-W5'
```

## 新しいファイル形式レシピの作成手順

### 📝 レシピ作成の基本手順

#### 1. ファイル形式の調査
- ターゲットファイル形式のMIMEタイプを確認
- 必要なPythonライブラリを特定
- ライブラリがPyodideで利用可能か確認（[Pyodide packages](https://pyodide.org/en/stable/usage/packages-in-pyodide.html)）

#### 2. レシピファイル構造（必須セクション）
```markdown
# **[形式名]自動生成ページ レシピ v1.0**

## **🔴 必須要件 - 必ず最初に確認してください 🔴**
### **1. 外部ライブラリのインストール（絶対に省略禁止）**
### **2. Pythonコードのインデント規則（エラー防止のため厳守）**

## **📋 AIへの実装指示**
### **書き換える箇所**
### **Pythonコード作成のルール**

## **📄 HTML骨子テンプレート**

## **⚠️ よくあるエラーと対処法**

## **💡 実装例**

## **📚 [ライブラリ名]の主要機能リファレンス**

## **✅ 最終チェックリスト**

## **📝 更新履歴**

## **📄 ライセンス**
```

#### 3. HTMLテンプレートの作成
既存レシピのHTMLテンプレートを基に作成し、以下を変更：
- ページタイトル（2箇所）
- ライブラリインストール部分
- Pythonコード部分
- ファイル名とMIMEタイプ
- Base64変数名（js.xxx_base64_data）

#### 4. 必須実装パターン
```python
# 共通パターン（すべてのレシピで必須）
from io import BytesIO
import base64
import js

# 1. ライブラリ固有の処理
[ファイル生成処理]

# 2. BytesIOへの保存
buffer = BytesIO()
[保存処理]
buffer.seek(0)

# 3. Base64エンコード
bytes_data = buffer.read()
base64_data = base64.b64encode(bytes_data).decode('utf-8')

# 4. JavaScriptへの受け渡し（必須）
js.[形式]_base64_data = base64_data

print("[形式] file has been generated in memory.")
```

### ⚠️ レシピ作成時の重要な注意点

#### 削除厳禁箇所のマーキング
```html
<!-- 
  ⚠️⚠️⚠️ 警告: 以下のPyodideと[ライブラリ名]インストール部分は削除厳禁 ⚠️⚠️⚠️
  この部分を削除すると、ファイルの生成が不可能になります
-->
```
- 🔴🔴🔴 のマークで視覚的に強調
- HTMLコメントで警告を明記

#### インデントエラー防止の徹底
```html
<!-- 
  AIがここを書き換える: [ライブラリ名]のコード
  重要: Pythonコードは左端から開始（インデントなし）
-->
<script type="text/python" id="python-code">
from [ライブラリ] import [クラス]  # ← 左端から開始
import base64  # ← インデントなし
```

#### MIMEタイプの正確な設定
| ファイル形式 | MIMEタイプ |
|------------|-----------|
| CSV | text/csv |
| JSON | application/json |
| XML | application/xml |
| HTML | text/html |
| Markdown | text/markdown |
| YAML | application/x-yaml |
| SQLite | application/x-sqlite3 |

#### エラーハンドリングの実装
```javascript
if (!base64Data) {
    throw new Error("Python code did not produce any data.");
}
```

### 🧪 レシピのテスト手順

1. **基本動作テスト**
   - 最小限のコンテンツで動作確認
   - ダウンロードが正常に開始されるか
   - ファイルが正しく開けるか

2. **エラーケーステスト**
   - ライブラリ削除時のエラー確認
   - インデントエラーの発生確認
   - Base64変数忘れのエラー確認

3. **互換性テスト**
   - 主要ブラウザ（Chrome、Firefox、Edge、Safari）
   - 生成ファイルの対応アプリケーション確認

### 📋 新レシピ追加時のチェックリスト

- [ ] ライブラリがPyodideで利用可能か確認
- [ ] MIMEタイプが正しく設定されているか
- [ ] 必須セクションがすべて含まれているか
- [ ] 削除厳禁マーキングが適切に配置されているか
- [ ] インデント規則の説明が明確か
- [ ] Base64変数名が一意で分かりやすいか
- [ ] 実装例が実用的で理解しやすいか
- [ ] エラー対処法が網羅的か
- [ ] バージョン番号が v1.0 から開始されているか
- [ ] MITライセンス表記が含まれているか

### 🔄 既存レシピの更新手順

1. **バージョン番号の更新**
   - マイナー更新: v1.0 → v1.1（バグ修正、小改善）
   - メジャー更新: v1.x → v2.0（大きな機能追加）

2. **更新履歴の記録**
   ```markdown
   ## **📝 更新履歴**
   - **v1.1** (日付): 変更内容の概要
     - 具体的な変更点1
     - 具体的な変更点2
   - **v1.0** (日付): 初版リリース
   ```

3. **後方互換性の維持**
   - 既存の必須要素は削除しない
   - 新機能は追加として実装
   - 破壊的変更は避ける

### 💡 レシピ作成のベストプラクティス

1. **シンプルさを保つ**
   - 最小限の必須要件に集中
   - 複雑な機能は実装例セクションで提供

2. **視覚的な強調**
   - 🔴 赤丸で重要事項を強調
   - ⚠️ 警告マークでエラー防止
   - 📋 アイコンでセクションを分かりやすく

3. **実用的な例の提供**
   - ビジネスユースケース
   - 個人利用シナリオ
   - 段階的な複雑度の例

4. **エラーメッセージの充実**
   - 原因を明確に説明
   - 具体的な解決手順を提示
   - よくある間違いを事前に防ぐ

## 開発時の確認事項
- [ ] レシピファイルが添付されているか
- [ ] 必要なライブラリのインストール処理があるか
- [ ] Pythonコードは左端から開始しているか
- [ ] Base64データの受け渡し処理があるか
- [ ] ファイル名とMIMEタイプは適切か
- [ ] エラーハンドリングは実装されているか