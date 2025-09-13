# 🎉 Office文書自動生成レシピシステム

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/Version-1.1.1-blue.svg)](https://github.com/HarmoniaEpic/text-to-office-recipes)
[![Python](https://img.shields.io/badge/Python-3.x-green.svg)](https://www.python.org/)
[![Pyodide](https://img.shields.io/badge/Powered%20by-Pyodide-orange.svg)](https://pyodide.org/)

**AIとレシピファイルで、PowerPoint・Word・Excel・PDFを瞬時に自動生成！**

ローカルAI対応・MITライセンス・選択可能なセキュリティレベルを実現した文書生成レシピ

---

## 📚 目次

- [特徴](#-特徴)
- [デモ](#-デモ)
- [クイックスタート](#-クイックスタート)
- [作れるファイルの種類](#-作れるファイルの種類)
- [使い方](#-使い方)
- [技術仕様](#-技術仕様)
- [セキュリティとプライバシー](#-セキュリティとプライバシー)
- [よくある質問](#-よくある質問)
- [貢献する](#-貢献する)
- [更新履歴](#-更新履歴)
- [ライセンス](#-ライセンス)

---

## ✨ 特徴

### 🚀 **簡単・高速**
- **4ステップで完成** - レシピ選択 → AI添付 → 指示 → ダウンロード
- **処理時間約10秒** - 高速な文書生成
- **プログラミング知識不要** - 誰でも使える

### 🔒 **安全・プライベート**
- **ローカルAI対応** - Ollama/Jan使用で完全オフライン
- **文書生成はローカル処理** - WebAssemblyでブラウザ内実行
- **選択可能なセキュリティレベル** - 用途に応じてAIを選択

### 📄 **オープンソース**
- **MITライセンス** - 自由に使用・改変・配布可能
- **商用利用可能** - ビジネスでも制限なく使用
- **コミュニティ駆動** - 誰でも貢献できる

---

## 🎬 デモ

### 基本的な使い方
```
1. レシピファイルを選択（例：PPTX-RECIPE.md）
2. AIに添付
3. 「営業報告のプレゼンを5枚で作って」と依頼
4. 生成されたHTMLをダウンロード → 開く → PPTXゲット！
```

### 動作イメージ
- **PowerPoint生成**: プレゼンテーション、提案書、報告書
- **Word生成**: 契約書、マニュアル、レポート
- **Excel生成**: 売上表、予算管理、データ分析（CSV対応）
- **PDF生成**: 請求書、証明書、印刷物

---

## 🚀 クイックスタート

### 必要なもの
- 対応ブラウザ（Chrome、Firefox、Edge、Safari）
- レシピファイル（本プロジェクトに含まれる）
- AIアシスタント（以下から選択）：
  - **高セキュリティ**: [Ollama](https://ollama.ai/)、[Jan](https://jan.ai/)（ローカルAI）
  - **高利便性**: Claude、ChatGPT（クラウドAI）

### 30秒で始める（クラウドAI使用）

#### 1️⃣ **プロジェクトをダウンロード**
```bash
# GitHubからダウンロード
git clone https://github.com/HarmoniaEpic/text-to-office-recipes.git
```
または [ZIPファイルをダウンロード](https://github.com/HarmoniaEpic/text-to-office-recipes/archive/refs/heads/main.zip)

#### 2️⃣ **レシピを選んでAIに添付**
```
PowerPoint → PPTX-RECIPE.md
Word → DOCX-RECIPE.md
Excel → XLSX-RECIPE.md
PDF → PDF-RECIPE.md
```

#### 3️⃣ **AIに依頼（コピペOK）**
```
「新商品紹介のプレゼンを5枚で作成してください」
```

#### 4️⃣ **ダウンロード**
生成されたHTMLを保存 → ブラウザで開く → ダウンロードボタンをクリック！

### 完全プライバシー保護で始める（Ollama使用）

<details>
<summary>🔐 機密文書を扱う場合はこちら</summary>

#### 1️⃣ **Ollamaをインストール**
```bash
# Mac/Linux
curl -fsSL https://ollama.ai/install.sh | sh

# Windows
winget install Ollama.Ollama
# または、https://ollama.ai からインストーラーをダウンロード
```

#### 2️⃣ **モデルをダウンロード**
```bash
ollama pull gemma3  # または phi4, llama3, llava, codellama, mistral など
```

#### 3️⃣ **Ollamaを起動**
```bash
ollama serve
```

#### 4️⃣ **レシピファイルをOllamaに送信**
```bash
# レシピを読み込んで指示
cat PPTX-RECIPE.md | ollama run gemma3 "このレシピを使って営業報告のプレゼンを作成するHTMLを生成してください"
```

これで完全にオフラインで文書生成が可能です！
</details>

---

## 📦 最新バージョン

| コンポーネント | バージョン | 最終更新 | 状態 |
|-------------|----------|---------|------|
| **プロジェクト全体** | v1.1.1 | 2025-09-14 | ✅ 安定版 |
| PPTX-RECIPE | v4.0 | 2025-09-14 | ✅ 最新 |
| DOCX-RECIPE | v4.0 | 2025-09-14 | ✅ 最新 |
| XLSX-RECIPE | v2.0 | 2025-09-13 | ✅ 最新 |
| PDF-RECIPE | v2.0 | 2025-09-13 | ✅ 最新 |

---

## 📝 作れるファイルの種類

### 🎯 **PowerPoint（.pptx）** - v4.0
プレゼンテーション資料の自動生成
- ✅ 会議資料・提案書
- ✅ 商品紹介・発表スライド
- ✅ 画像プレースホルダー機能
- 📄 レシピ: `PPTX-RECIPE.md`

### 📄 **Word（.docx）** - v4.0
文書・レポートの自動生成
- ✅ 報告書・契約書
- ✅ マニュアル・お知らせ文書
- ✅ 表・画像プレースホルダー対応
- 📄 レシピ: `DOCX-RECIPE.md`

### 📊 **Excel（.xlsx）** - v2.0
表計算・データ分析の自動生成
- ✅ 売上表・予算管理
- ✅ グラフ・条件付き書式
- ✅ **CSVデータ自動インポート対応**
- 📄 レシピ: `XLSX-RECIPE.md`

### 📑 **PDF（.pdf）** - v2.0
印刷用文書の自動生成
- ✅ 請求書・領収書
- ✅ 証明書・賞状
- ✅ 日本語フォント完全対応
- 📄 レシピ: `PDF-RECIPE.md`

---

## 📖 使い方

### 基本的な流れ

```mermaid
graph LR
    A[レシピ選択] --> B[AIに添付]
    B --> C[要望を伝える]
    C --> D[HTML生成]
    D --> E[ダウンロード]
```

### レシピファイルの選び方

| 作りたいもの | 使うレシピ | ファイル名 | 追加ファイル |
|------------|-----------|-----------|------------|
| プレゼン・スライド | PowerPointレシピ | `PPTX-RECIPE.md` | 不要 |
| 文書・レポート | Wordレシピ | `DOCX-RECIPE.md` | 不要 |
| 表・グラフ | Excelレシピ | `XLSX-RECIPE.md` | CSVファイル（任意） |
| 請求書・証明書 | PDFレシピ | `PDF-RECIPE.md` | 不要 |

### 💬 AIへの依頼例（コピペOK）

#### PowerPointを作りたいとき
```
【手順】
1. PPTX-RECIPE.mdを添付
2. 以下のメッセージを送信

新商品の紹介プレゼンを作ってください。
以下の構成でお願いします：
1. タイトル「新商品のご案内」
2. 商品の特徴（3つ）
3. 価格とプラン
4. お問い合わせ先
```

#### Excelで既存データを使いたいとき
```
【手順】
1. XLSX-RECIPE.mdを添付
2. CSVファイルも添付
3. 以下のメッセージを送信

添付したCSVデータから月次レポートを作成してください。
・グラフを追加
・合計と平均を計算
・見やすく整形
```

### CSVファイルの準備（Excel用）

CSVファイルの例：
```csv
商品名,数量,単価,売上
ノートPC,5,80000,400000
マウス,20,3000,60000
キーボード,15,5000,75000
```

**ポイント**：
- 1行目に項目名を記載
- カンマ（,）で区切る
- UTF-8またはShift-JISで保存

---

## 🔧 技術仕様

### アーキテクチャ

```
┌─────────────────────────────────────┐
│           ユーザー                   │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│    AI選択（セキュリティレベル）       │
├─────────────────────────────────────┤
│ 🔐 ローカルAI     │ ☁️ クラウドAI   │
│ ・Ollama         │ ・Claude       │
│ ・Jan            │ ・ChatGPT      │
│ (完全オフライン)  │ (要ネット接続) │
└─────────────┬───────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│       生成されたHTML                 │
│   ┌───────────────────────┐        │
│   │    Pyodide (WASM)      │        │
│   │  - Python実行環境      │        │
│   │  - ライブラリ管理      │        │
│   └───────────────────────┘        │
│   ┌───────────────────────┐        │
│   │   文書生成ライブラリ    │        │
│   │  - python-docx         │        │
│   │  - openpyxl           │        │
│   │  - python-pptx        │        │
│   │  - reportlab          │        │
│   └───────────────────────┘        │
└─────────────────────────────────────┘
              │
              ▼
         📄 Office文書
        (ローカル保存)
```

### 主要技術

- **実行環境**: [Pyodide](https://pyodide.org/) (WebAssembly版Python)
- **UIフレームワーク**: Tailwind CSS
- **文書生成ライブラリ**:
  - `python-docx`: Word文書生成
  - `openpyxl`: Excel表計算生成
  - `python-pptx`: PowerPoint生成
  - `reportlab`: PDF生成

### システム要件

| 項目 | 最小要件 | 推奨要件 |
|-----|---------|---------|
| **ブラウザ** | Chrome 57+, Firefox 52+, Safari 11+, Edge 16+ | 最新版 |
| **メモリ** | 2GB RAM | 4GB RAM以上 |
| **ネットワーク** | 初回ロード時のみ必要 | ブロードバンド |
| **JavaScript** | 有効化必須 | - |
| **WebAssembly** | サポート必須 | - |

---

## 🔒 セキュリティとプライバシー

### AIモデルの選択によるセキュリティレベル

このシステムは使用するAIによって、セキュリティとプライバシーのレベルが異なります。

#### 🔐 **最高レベル：ローカルAI使用時（Ollama/Jan）**

**完全なプライバシー保護を実現**

| AI | 特徴 | プライバシーレベル |
|-----|------|------------------|
| **[Ollama](https://ollama.ai/)** | ローカルで動作するLLM | ⭐⭐⭐⭐⭐ 完全保護 |
| **[Jan](https://jan.ai/)** | オフライン対応AI | ⭐⭐⭐⭐⭐ 完全保護 |

**メリット：**
- ✅ **完全オフライン動作** - インターネット接続不要
- ✅ **データ漏洩リスクゼロ** - 一切の外部通信なし
- ✅ **機密文書の安全な処理** - 企業秘密・個人情報も安心
- ✅ **コンプライアンス完全準拠** - GDPR、個人情報保護法に完全対応

**デメリット：**
- ⚠️ 初期セットアップが必要
- ⚠️ ローカルマシンのスペックに依存
- ⚠️ モデルのダウンロードが必要（数GB）

#### ☁️ **標準レベル：クラウドAI使用時（Claude/ChatGPT）**

**利便性重視の選択肢**

| AI | 特徴 | プライバシーレベル |
|-----|------|------------------|
| **Claude** | Anthropic社のAI | ⭐⭐⭐ 標準保護 |
| **ChatGPT** | OpenAI社のAI | ⭐⭐⭐ 標準保護 |

**メリット：**
- ✅ **セットアップ不要** - すぐに使用開始
- ✅ **高性能なAIモデル** - 最新の言語モデル
- ✅ **常に最新版** - 自動アップデート

**注意点：**
- ⚠️ **レシピと要望がAIサービスに送信される**
- ⚠️ **生成されたHTMLコードがAI経由で返される**
- ⚠️ **AIサービスの利用規約に従う必要がある**

**ただし：**
- ✅ 生成後のOfficeファイル作成はブラウザ内で完結
- ✅ 最終的な文書データはローカル処理
- ✅ ダウンロードファイルは外部送信されない

### データフローの比較

#### 🔐 **Ollama/Jan使用時のデータフロー**
```
完全ローカル処理：
1. レシピファイル → ローカルAI（Ollama/Jan）✅ローカル
2. HTML生成 → ローカル完結 ✅ローカル
3. ブラウザで実行 → WebAssembly処理 ✅ローカル
4. ファイル生成 → ローカルダウンロード ✅ローカル
※ 一切の外部通信なし（CDN除く）
```

#### ☁️ **Claude/ChatGPT使用時のデータフロー**
```
ハイブリッド処理：
1. レシピファイル → クラウドAI ⚠️外部送信
2. HTML生成 → AIから受信 ⚠️外部通信
3. ブラウザで実行 → WebAssembly処理 ✅ローカル
4. ファイル生成 → ローカルダウンロード ✅ローカル
```

### 推奨される使用方法

| 用途 | 推奨AI | 理由 |
|-----|--------|------|
| **機密文書・個人情報** | Ollama/Jan | 完全なプライバシー保護 |
| **企業の内部文書** | Ollama/Jan | 情報漏洩リスクゼロ |
| **一般的な文書** | Claude/ChatGPT | 利便性と性能のバランス |
| **テスト・学習用** | Claude/ChatGPT | セットアップ不要 |

### セキュリティベストプラクティス

#### 1. **機密性の高い文書を扱う場合**
```bash
# Ollamaのインストール（Mac/Linux）
curl -fsSL https://ollama.ai/install.sh | sh

# モデルのダウンロード
ollama pull llama2

# 使用開始
ollama serve
```

#### 2. **一般的な文書を扱う場合**
- クラウドAIの利用規約を確認
- 個人情報や機密情報を含めない
- 生成されたHTMLを確認してから実行

#### 3. **共通の安全対策**
- ✅ ブラウザを最新版に保つ
- ✅ 信頼できるネットワークで使用
- ✅ 生成されたファイルをウイルススキャン
- ✅ 定期的にブラウザキャッシュをクリア

### プライバシーレベル評価

| 項目 | Ollama/Jan | Claude/ChatGPT |
|------|------------|----------------|
| レシピファイルの送信先 | ローカルのみ | AIサービス |
| HTML生成場所 | ローカル | AIサービス |
| 文書生成処理 | ブラウザ内 | ブラウザ内 |
| 最終ファイル | ローカル保存 | ローカル保存 |
| インターネット必要性 | 不要※ | 必要 |
| **総合評価** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |

※CDNからのライブラリ読み込みを除く

---

## ✨ 活用例

### 🏢 **ビジネスシーン**

| シチュエーション | 依頼例 | 使用レシピ | 推奨AI |
|----------------|--------|-----------|--------|
| 急な会議資料 | 「15分で説明できる事業計画プレゼンを作って」 | PPTX | Claude/ChatGPT |
| 月次報告（社内） | 「先月の売上データから分析レポートを作成」 | XLSX + CSV | Ollama/Jan |
| 見積書作成 | 「Webサイト制作の見積書を作って」 | PDF | Claude/ChatGPT |
| 機密契約書 | 「業務委託契約書のテンプレートを作成」 | DOCX | Ollama/Jan |
| 人事評価書 | 「従業員評価フォームを作成」 | XLSX | Ollama/Jan |

### 🎓 **教育・学習**

| シチュエーション | 依頼例 | 使用レシピ | 推奨AI |
|----------------|--------|-----------|--------|
| 授業資料 | 「歴史の授業用スライドを10枚作って」 | PPTX | Claude/ChatGPT |
| レポート作成 | 「環境問題についてのレポート構成を作って」 | DOCX | Claude/ChatGPT |
| 成績管理 | 「クラス30人分の成績管理表を作って」 | XLSX | Ollama/Jan |
| 修了証発行 | 「プログラミング講座の修了証を作成」 | PDF | Claude/ChatGPT |
| 個人情報含む名簿 | 「生徒の連絡先リストを作成」 | XLSX | Ollama/Jan |

### 🏠 **プライベート**

| シチュエーション | 依頼例 | 使用レシピ | 推奨AI |
|----------------|--------|-----------|--------|
| 家計簿 | 「月別収支が分かる家計簿を作って」 | XLSX | Ollama/Jan |
| 旅行計画 | 「3泊4日の京都旅行プランを作成」 | DOCX | Claude/ChatGPT |
| イベント案内 | 「誕生日パーティーの招待状を作って」 | PDF | Claude/ChatGPT |
| 医療記録管理 | 「家族の健康記録表を作成」 | XLSX | Ollama/Jan |

**🔐 セキュリティの目安**：
- 個人情報・機密情報を含む → **Ollama/Jan**
- 一般的な内容・テンプレート → **Claude/ChatGPT**

---

## ❓ よくある質問

### 基本的な質問

<details>
<summary>Q: このツール自体は無料ですか？</summary>

**A:** はい、このツール自体はMITライセンスのオープンソースソフトウェアです。自由に使用・改変・配布できます。ただし、以下の点にご注意ください：
- **Claude/ChatGPT使用時**: 各AIサービスの利用料金が適用される場合があります
- **Ollama/Jan使用時**: ソフトウェア自体は無料ですが、実行に必要なハードウェアやモデルのダウンロードに伴う通信費は発生します
</details>

<details>
<summary>Q: インターネット接続は必要ですか？</summary>

**A:** 使用するAIによって異なります：
- **Claude/ChatGPT使用時**: 常時インターネット接続が必要
- **Ollama/Jan使用時**: 完全オフライン動作可能（初回のライブラリダウンロード後）
</details>

<details>
<summary>Q: 機密文書を扱っても安全ですか？</summary>

**A:** AIの選択によってセキュリティレベルが異なります：
- **Ollama/Jan**: ⭐⭐⭐⭐⭐ 完全に安全（すべてローカル処理）
- **Claude/ChatGPT**: ⭐⭐⭐ 標準的な安全性（レシピと要望がAIサービスに送信される）

機密性の高い文書にはOllama/Janの使用を強く推奨します。
</details>

<details>
<summary>Q: スマートフォンでも使えますか？</summary>

**A:** 技術的には可能ですが、パソコンでの利用を推奨します。スマホではメモリ不足により動作が不安定になる場合があります。
</details>

### セキュリティに関する質問

<details>
<summary>Q: OllamaとClaude/ChatGPTの違いは何ですか？</summary>

**A:** 主な違いは以下の通りです：

| 項目 | Ollama/Jan | Claude/ChatGPT |
|------|------------|----------------|
| **動作場所** | 完全ローカル | クラウド |
| **プライバシー** | 完全保護 | 標準保護 |
| **セットアップ** | 必要（簡単） | 不要 |
| **性能** | PCスペック依存 | 常に高性能 |
| **オフライン** | 可能 | 不可能 |
</details>

<details>
<summary>Q: どのAIを選ぶべきですか？</summary>

**A:** 用途によって推奨が異なります：
- **企業の機密文書、個人情報**: Ollama/Jan（完全ローカル）
- **一般的な文書、テスト**: Claude/ChatGPT（高性能・便利）
- **プライバシー重視**: Ollama/Jan
- **利便性重視**: Claude/ChatGPT
</details>

### 技術的な質問

<details>
<summary>Q: なぜサーバーが不要なのですか？</summary>

**A:** Pyodide（WebAssembly版Python）を使用しているため、すべてのPython処理がブラウザ内で実行されます。ただし、AI処理は選択したAIによって異なります（Ollama/Janはローカル、Claude/ChatGPTはクラウド）。
</details>

<details>
<summary>Q: 生成できるファイルサイズに制限はありますか？</summary>

**A:** ブラウザのメモリ制限により、おおよそ50MBまでのファイルが生成可能です。
</details>

<details>
<summary>Q: CSVファイルの文字コードは？</summary>

**A:** UTF-8とShift-JISの両方に対応しています。自動判定により適切に処理されます。
</details>

### トラブルシューティング

<details>
<summary>Q: ファイルがダウンロードできません</summary>

**A:** ブラウザのポップアップブロックを解除してください。また、プライベートブラウジングモードでは正常に動作しない場合があります。
</details>

<details>
<summary>Q: エラーが表示されます</summary>

**A:** エラーオーバーレイの「エラー全文をコピー」ボタンでエラー内容をコピーし、AIに貼り付けて修正を依頼してください。
</details>

<details>
<summary>Q: 生成に時間がかかります</summary>

**A:** 初回は10-20秒かかる場合があります。2回目以降はキャッシュにより高速化されます。
</details>

---

## 🤝 フォーク歓迎

このプロジェクトのフォークを歓迎します！

---

## 📝 更新履歴

### [1.1.1] - 2025-01-18
#### 📝 ドキュメントの精度向上
- **修正**
  - コーディング時のAIによる誇張表現を修正
  - MITライセンスであることを適切に強調
  - AIサービス利用時のコストについて明確化

### [1.1.0] - 2025-01-18
#### 🔒 セキュリティドキュメントの改善
- **修正**
  - セキュリティとプライバシーの説明を正確に更新
  - Ollama/Janによる完全ローカル処理オプションを明確化
  - クラウドAI使用時のデータフローを正確に説明

- **追加**
  - ローカルAI（Ollama/Jan）のセットアップガイド
  - AIモデル選択によるセキュリティレベルの比較表
  - 用途別の推奨AI選択ガイド

### [1.0.0] - 2025-01-17
#### 🎉 初版リリース
- **新機能**
  - PowerPoint（PPTX）自動生成機能
  - Word（DOCX）自動生成機能
  - Excel（XLSX）自動生成機能（CSV対応）
  - PDF自動生成機能
  - エラーオーバーレイUI
  - 画像プレースホルダー機能

- **レシピバージョン**
  - PPTX-RECIPE v3.4
  - DOCX-RECIPE v3.0
  - XLSX-RECIPE v2.0
  - PDF-RECIPE v2.0

- **ドキュメント**
  - README.md作成
  - TECHNICAL_DOCUMENTATION.md作成
  - CLAUDE.md（プロジェクトメモリ）作成
  - 各レシピファイルの詳細ドキュメント

---

## 📱 対応環境

### ✅ 対応ブラウザ
- **Chrome** 57以降（推奨）
- **Firefox** 52以降
- **Safari** 11以降
- **Edge** 16以降

### ✅ 対応OS
- **Windows** 10以降
- **macOS** 10.15以降
- **Linux** Ubuntu 18.04以降
- **ChromeOS** 対応

### ⚠️ 非対応環境
- Internet Explorer（全バージョン）
- モバイルブラウザ（限定的なサポート）
- WebAssembly非対応ブラウザ

---

## 💡 トラブルシューティング

### よくある問題と解決策

| 問題 | 原因 | 解決策 |
|-----|------|--------|
| ダウンロードできない | ポップアップブロック | ブラウザ設定でポップアップを許可 |
| エラーが出る | ライブラリ読み込み失敗 | ページを再読み込み |
| 動作が遅い | メモリ不足 | 他のタブを閉じる |
| 文字化け | 文字コード不一致 | UTF-8で保存し直す |

### デバッグ方法

1. **ブラウザコンソールを開く**（F12キー）
2. **エラーメッセージを確認**
3. **エラーオーバーレイの内容をコピー**
4. **AIに貼り付けて解決策を求める**

---

## 👥 コミュニティ

- **GitHub Discussions**: [質問・議論](https://github.com/HarmoniaEpic/text-to-office-recipes/discussions)
- **Issues**: [バグ報告・機能要望](https://github.com/HarmoniaEpic/text-to-office-recipes/issues)
- **Wiki**: [詳細ドキュメント](https://github.com/HarmoniaEpic/text-to-office-recipes/wiki)

---

## 📄 ライセンス

このプロジェクトは[MITライセンス](LICENSE)の下で公開されています。

```
MIT License

Copyright (c) 2025 HarmoniaEpic

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### ライセンスの意味

**MITライセンスにより、あなたは以下のことができます：**
- ✅ **商用利用** - ビジネスで自由に使用
- ✅ **改変** - ニーズに合わせてカスタマイズ
- ✅ **配布** - 他の人と共有
- ✅ **私的利用** - 個人プロジェクトで使用

**条件：**
- 📋 著作権表示とライセンス表示を含める

---

## 🙏 謝辞

このプロジェクトは以下の素晴らしいオープンソースプロジェクトのおかげで実現しました：

- [Pyodide](https://pyodide.org/) - ブラウザでPythonを実行
- [python-docx](https://python-docx.readthedocs.io/) - Word文書生成
- [openpyxl](https://openpyxl.readthedocs.io/) - Excel表計算生成
- [python-pptx](https://python-pptx.readthedocs.io/) - PowerPoint生成
- [ReportLab](https://www.reportlab.com/) - PDF生成
- [Tailwind CSS](https://tailwindcss.com/) - UIフレームワーク

---

<div align="center">

**楽しいOffice文書作成ライフを！** 🎊

Made with Claude and ChatGPT

[⬆ トップに戻る](#-office文書自動生成システム)

</div>
