# **ASMSCRIPT-OPTIMIZER-RECIPE v1.4.1**
**AssemblyScript WebAssembly コンパイラ統合型単一HTMLアプリケーション生成レシピ**

---

## **📌 このレシピについて**

生成AIがユーザーの要求を解釈し、AssemblyScriptコードをブラウザ内でWebAssemblyにコンパイル・実行する**高性能単一HTMLファイル**を自動生成するための汎用指示書です。本レシピは特定のAI環境に依存せず、あらゆる生成AIシステムで利用可能な設計となっています。

---

### **基本機能**
- 🚀 **自動初期化**: ページロード時に自動的にコンパイル・実行開始
- 🔧 **15種類以上の最適化オプション**: 詳細なコンパイル制御
- 📦 **モジュール化アーキテクチャ**: 多様なユースケースに対応
- ⚡ **CDNベース**: 外部ツール不要、ブラウザのみで完結
- 📟 **コマンドラインプレビュー**: CLIコマンドをリアルタイム表示（v1.4.1新機能）

---

## **🎯 AI実装の手順**

### **ステップ0: ユーザー要求の確認（必須）**

AIは最初に以下のチェックを実行し、要求が不明確な場合は**必ず作業を中断**してユーザーに確認を求めます：

#### **要求明確性チェックリスト**
1. **具体的な処理内容が指定されているか？**
   - ❌ 不明確な例: 「AssemblyScriptでアプリを作って」
   - ✅ 明確な例: 「AssemblyScriptで素数を計算するアプリを作って」

2. **用途やゴールが明確か？**
   - ❌ 不明確な例: 「最適化したい」
   - ✅ 明確な例: 「画像のグレースケール変換を高速化したい」

3. **入出力の仕様が推測可能か？**
   - ❌ 不明確な例: 「データを処理して」
   - ✅ 明確な例: 「CSVファイルを読み込んでグラフを表示」

#### **作業中断と問い合わせのテンプレート**

```markdown
❓ **AssemblyScriptで実装する内容を教えてください**

申し訳ございません。具体的にどのような処理を実装したいか教えていただけますか？
以下の情報があると、最適なWebAssemblyアプリケーションを生成できます：

**1. 何を作りたいですか？**（例を選択または自由記述）
□ 計算処理（数値計算、統計、アルゴリズム実行など）
□ グラフィックス（Canvas描画、アニメーション、ゲームなど）  
□ データ処理（フィルタリング、変換、分析など）
□ インタラクティブツール（エディタ、シミュレーターなど）
□ その他: [具体的に記述してください]

**2. 具体的な処理内容**
例：
- 「1000個の素数を高速計算したい」
- 「ボールの物理シミュレーションを作りたい」
- 「画像にフィルタを適用したい」
- 「リアルタイムで音声を処理したい」

**3. 期待する最適化の方向性**（任意）
□ 実行速度を最優先
□ ファイルサイズを最小化
□ メモリ使用量を抑制
□ バランス重視

具体的な要望をお聞かせください。
```

### **ステップ0.5: AssemblyScriptバージョンの動的取得**

HTMLを生成する前に、AIは以下の手順で最新の安定版バージョンを取得します：

#### **動的バージョン取得の実装**

```javascript
// バージョン管理システム
const VersionManager = {
    // キャッシュ機構
    cache: {
        get() {
            try {
                const cached = sessionStorage.getItem('asc_version_cache');
                if (cached) {
                    const data = JSON.parse(cached);
                    // 1時間以内のキャッシュは有効
                    if (Date.now() - data.timestamp < 3600000) {
                        return data.version;
                    }
                }
            } catch (e) {}
            return null;
        },
        set(version) {
            try {
                sessionStorage.setItem('asc_version_cache', JSON.stringify({
                    version,
                    timestamp: Date.now()
                }));
            } catch (e) {}
        }
    },
    
    async getLatestVersion() {
        // キャッシュチェック
        const cached = this.cache.get();
        if (cached) {
            console.log(`[Version] Using cached: ${cached}`);
            return cached;
        }
        
        // jsDelivr優先の取得戦略
        const strategies = [
            { name: 'jsDelivr API', fn: () => this.fetchFromJsDelivr() },
            { name: 'jsDelivr Package', fn: () => this.fetchJsDelivrPackageJson() },
            { name: 'Unpkg', fn: () => this.fetchFromUnpkg() }
        ];
        
        for (const strategy of strategies) {
            try {
                console.log(`[Version] Trying ${strategy.name}...`);
                const version = await Promise.race([
                    strategy.fn(),
                    this.timeout(3000)
                ]);
                
                if (version && this.isValidStableVersion(version)) {
                    console.log(`[Version] Success: ${version}`);
                    this.cache.set(version);
                    return version;
                }
            } catch (e) {
                console.warn(`[Version] ${strategy.name} failed`);
            }
        }
        
        // フォールバック
        return "0.28.8";
    },
    
    async fetchFromJsDelivr() {
        const response = await fetch('https://data.jsdelivr.com/v1/package/npm/assemblyscript');
        const data = await response.json();
        const stable = data.versions.filter(v => this.isValidStableVersion(v));
        return stable[0];
    },
    
    async fetchJsDelivrPackageJson() {
        const response = await fetch('https://cdn.jsdelivr.net/npm/assemblyscript@latest/package.json');
        const data = await response.json();
        return data.version;
    },
    
    async fetchFromUnpkg() {
        const response = await fetch('https://unpkg.com/assemblyscript@latest/package.json');
        const data = await response.json();
        return data.version;
    },
    
    isValidStableVersion(version) {
        const semverRegex = /^(\d+)\.(\d+)\.(\d+)$/;
        if (!version.match(semverRegex)) return false;
        
        const preRelease = ['alpha', 'beta', 'rc', 'dev', '-'];
        return !preRelease.some(indicator => 
            version.toLowerCase().includes(indicator)
        );
    },
    
    timeout(ms) {
        return new Promise((_, reject) => 
            setTimeout(() => reject(new Error('Timeout')), ms)
        );
    }
};
```

### **ステップ1: ユーザー要求の分析**

AIは以下の手順でユーザーの要求を処理します：

1. **ユースケースの特定**
   ```
   - ビジュアル系（Canvas/WebGL使用） → RenderModule
   - 計算処理系（数値出力） → ComputeModule  
   - データ処理系（変換・分析） → DataModule
   - インタラクティブ系（ユーザー入力） → InteractiveModule
   ```

2. **AssemblyScriptコードの生成/選択**
   ```typescript
   // 例：物理シミュレーションの場合
   export function update(dt: f32): void {
     // 物理演算ロジック
   }
   ```

3. **適切なモジュールタイプの決定**
   - 必要なWebAssemblyエクスポート関数
   - インポート要件（メモリ、環境変数等）
   - 実行頻度（アニメーションループ or オンデマンド）

### **ステップ2: AppModuleの生成**

選択されたユースケースに基づいて、AppModuleセクションを生成します：

```javascript
const AppModule = {
    name: "ユースケース名",
    sourceCode: `AssemblyScriptコード`,
    defaultOptions: { /* 最適なコンパイルオプション */ },
    execute: (exports) => { /* 実行ロジック */ },
    render: (data) => { /* 表示更新 */ }
};
```

### **ステップ3: HTMLファイルの組み立て**

1. コア構造テンプレートを読み込み
2. 動的バージョン取得機能を組み込み
3. AppModuleを埋め込み
4. 必要に応じてUIカスタマイズ部分を追加
5. 単一HTMLファイルとして出力

---

## **🚨 実装ルール（v1.4.1 コマンドラインプレビュー追加版）**

### **🔴 絶対禁止事項**

1. **直接importの使用禁止**
   ```javascript
   // ❌ 絶対に使用禁止（動作しない）
   import("https://cdn.jsdelivr.net/npm/assemblyscript@latest/dist/asc/index.js");
   
   // ✅ 必ず公式互換のweb.js方式を使用
   loadScriptTag("https://cdn.jsdelivr.net/npm/assemblyscript@0.28.8/dist/web.js");
   await import("assemblyscript/asc");
   ```

2. **@latestタグの使用禁止**
   ```javascript
   // ❌ 絶対に使用禁止
   const url = "assemblyscript@latest/dist/web.js";
   
   // ✅ 必ず具体的バージョンを指定
   const version = await VersionManager.getLatestVersion(); // 動的取得
   const url = `assemblyscript@${version}/dist/web.js`;
   ```

3. **単一CDNへの依存禁止**
   ```javascript
   // ❌ 単一CDNのみは禁止
   const url = "https://cdn.jsdelivr.net/...";
   
   // ✅ 必ず2つのCDNでフォールバック
   const cdnProviders = ['jsDelivr', 'UNPKG'];
   ```

### **🟡 必須ルール（v1.4.1）**

1. **公式互換のweb.js方式の使用**
   - web.jsスクリプトタグを動的に挿入
   - import map設定を待機
   - "assemblyscript/asc"をimport

2. **2段階CDNフォールバック**
   - 第1優先: jsDelivr（高速・公式推奨）
   - 第2優先: UNPKG（安定・npmミラー）

3. **タイムアウト保護**
   - 各CDN試行に10秒の上限
   - スクリプトタグのクリーンアップ

4. **import map待機処理**
   - グローバル変数の確認
   - 最大2秒の待機時間

5. **メタデータの記録**
   - 使用したCDNプロバイダー
   - 読み込み時間
   - 試行回数
   - バージョン取得元

6. **コマンドラインプレビュー表示**（v1.4.1新規）
   - WebAssemblyタブ最上部に配置
   - リアルタイム更新
   - 全オプション反映

### **🟢 推奨事項**

1. **デバッグ情報の出力**
   - 各CDN試行のログ
   - 成功/失敗の詳細
   - パフォーマンス測定

2. **エラーメッセージの明確化**
   - どのCDNが失敗したか
   - ネットワーク問題かバージョン問題か

3. **ユーザーへのフィードバック**
   - プログレスバーの更新
   - 現在試行中のCDN表示

---

## **📟 コマンドラインプレビュー機能（v1.4.1新機能）**

### **概要**

WebAssemblyタブの最上部に、現在のオプション設定に対応するCLIコマンドをリアルタイムで表示します。これにより、ブラウザ環境での設定をそのままローカル開発環境で再現できます。

### **機能仕様**

#### **表示位置**
- WebAssemblyタブの最上部（実行制御セクションの上）
- 常に可視領域に表示

#### **表示内容**
例）
```bash
asc main.ts -o main.wasm --runtime minimal --optimize --optimizeLevel 3
```

#### **更新タイミング**
- オプション変更時（チェックボックス、セレクトボックス）
- プリセット適用時
- 初期化完了時

#### **対応オプション**
すべてのUIオプションが自動的にコマンドライン引数に変換されます：
- 最適化オプション（--optimize, --optimizeLevel, --shrinkLevel）
- ランタイムオプション（--runtime, --exportRuntime, --importMemory等）
- デバッグオプション（--debug, --sourceMap, --noAssert等）
- 高度なオプション（--exportTable, --explicitStart等）

### **実装詳細**

```javascript
// コマンドラインプレビュー機能
function updateCmdPreview() {
    const options = OptionsManager.collectFromUI();
    const cmd = generateCmdLine(options);
    
    const preview = document.getElementById('cmd-preview');
    if (preview) {
        preview.textContent = cmd;
    }
}

function generateCmdLine(options) {
    const parts = ['asc', 'main.ts', '-o', 'main.wasm'];
    
    Object.entries(options).forEach(([key, value]) => {
        if (typeof value === 'boolean' && value) {
            parts.push(`--${key}`);
        } else if (typeof value !== 'boolean' && value !== '') {
            parts.push(`--${key}`, value.toString());
        }
    });
    
    return parts.join(' ');
}
```

---

## **📝 AssemblyScriptにおけるメモリ管理とデータ処理（v1.4.0）**

### **基本的なメモリアクセス方法**

AssemblyScriptでは、メモリへのアクセス方法が2つのレベルで提供されています：

#### **1. 高レベルアプローチ（推奨）**

TypedArrayを使用した管理されたメモリアクセス：

```typescript
// TypedArrayの使用（型安全で直感的）
let buffer = new Uint8ClampedArray(1024);
buffer[0] = 255;  // 自動的に境界チェック

// パフォーマンス最適化版
for (let i = 0; i < buffer.length; i++) {
    unchecked(buffer[i] = 128);  // 境界チェック省略
}
```

#### **2. 低レベルアプローチ（高速）**

組み込み関数による直接メモリアクセス：

```typescript
// load/store組み込み関数
store<u8>(ptr, value);         // 1バイト書き込み
let value = load<u8>(ptr);     // 1バイト読み込み

// 複数バイトの操作
store<i32>(ptr, 0x12345678);   // 4バイト書き込み
let int32 = load<i32>(ptr);    // 4バイト読み込み
```

### **メモリレイアウトとオブジェクト管理**

#### **管理オブジェクトのヘッダー構造**

```
オフセット  | サイズ | 内容
-----------|--------|------------------
-20        | usize  | メモリマネージャ情報
-16        | usize  | GC情報1
-12        | usize  | GC情報2
-8         | u32    | クラスID (rtId)
-4         | u32    | データサイズ (rtSize)
0          |        | ペイロード開始位置
```

#### **標準データ型のメモリ配置**

| 型 | クラスID | 特徴 |
|----|---------|------|
| ArrayBuffer | 1 | 生のバイト配列 |
| String | 2 | UTF-16文字列（JavaScriptと同じ） |
| TypedArray | 動的 | buffer + dataStart + byteLength |
| StaticArray | - | 固定サイズ、直接データ格納 |

### **実践的なメモリ管理パターン**

#### **パターン1: グローバルバッファ（再利用可能）**

```typescript
// グローバルバッファで頻繁な再割り当てを回避
let globalBuffer: Uint8ClampedArray | null = null;

export function initBuffer(size: i32): void {
    globalBuffer = new Uint8ClampedArray(size);
}

export function getBufferPtr(): usize {
    return globalBuffer ? globalBuffer!.dataStart : 0;
}
```

#### **パターン2: StaticArrayの活用（固定サイズ）**

```typescript
// StaticArrayは間接参照なし、高速アクセス
let lookupTable = new StaticArray<f32>(256);

// 初期化
for (let i = 0; i < 256; i++) {
    unchecked(lookupTable[i] = <f32>i / 255.0);
}
```

#### **パターン3: uncheckedによる最適化**

```typescript
// 安全性が保証される場合のみ使用
export function processArray(arr: Int32Array): i32 {
    let sum = 0;
    // lengthチェックを外側で行う
    let len = arr.length;
    for (let i = 0; i < len; i++) {
        sum += unchecked(arr[i]);  // 境界チェック省略
    }
    return sum;
}
```

### **型変換の重要性**

AssemblyScriptは暗黙的な型変換を許可しないため、明示的なキャストが必須です：

```typescript
// ❌ エラー：f64からf32への暗黙的変換
let value: f32 = 3.14;

// ✅ 正しい：明示的なキャスト（f32リテラル）
let value: f32 = <f32>3.14;
```

### **ホストとのデータ交換**

JavaScriptとWebAssemblyの間でデータを交換する際の戦略：

| データ型 | 戦略 | 説明 |
|---------|------|------|
| Number | 値渡し | 自動変換 |
| TypedArray | コピー | メモリ経由でコピー |
| String | コピー | UTF-16として転送 |
| Object（単純） | コピー | フィールド毎にコピー |
| Object（複雑） | 参照 | ポインタのみ渡す |

### **メモリ管理のベストプラクティス**

1. **TypedArrayを優先使用** - 型安全で管理された環境
2. **必要な場合のみload/store** - 低レベル制御が必要な場合
3. **uncheckedは計測後に適用** - まず動作確認、その後最適化
4. **StaticArrayで固定サイズデータ** - 間接参照を避ける
5. **グローバルバッファで再利用** - GC負荷を軽減

---

### 🟠 型変換に関する重要注意事項

AssemblyScriptは**暗黙的な型変換を一切許可しない**厳格な型システムを持っています。特に、JavaScriptの`number`に相当するデフォルトの浮動小数点型は`f64`であるため、パフォーマンス上の理由で`f32`を使用する際には、意識的な型変換が不可欠です。

#### 1. 基本ルール：明示的な型キャストを推奨

**現在のAssemblyScriptでは、`1.23f`のような`f`サフィックス記法はサポートされていません**。`f64`から`f32`への変換は、**`<f32>value`** という明示的な型キャストで行うのが最も安全で確実な方法です。

| 状況 | 対処法 | 記法（例） |
| :--- | :--- | :--- |
| **数値リテラル** | **明示的なキャスト**を推奨。 | `const gravity: f32 = <f32>9.8;` |
| **変数への代入** | 型推論が効く場合もあるが、キャストが安全。 | `let value: f32 = <f32>someF64Value;` |
| **`Math`関数** | 戻り値は常に`f64`のため、**キャストが必須**。 | `const sinValue: f32 = <f32>Math.sin(angle);` |
| **演算** | 演算子の両辺の型を`f32`に統一する。 | `const result: f32 = val1 * <f32>0.5;` |

#### 2. よくあるエラーと即座の対処

* **`Type 'f64' is not assignable to type 'f32'`**
    → `f64`の値の前に**`<f32>`キャストを追加**します。（例： `<f32>Math.PI`）

* **`Operator '*' cannot be applied to types 'f32' and 'f64'`**
    → `f64`側の値に**`<f32>`キャストを追加**し、両辺の型を`f32`に統一します。

#### 3. 型選択の指針（変更なし）

**f32を使用（パフォーマンス優先）**
* Canvas/WebGL描画
* ゲーム物理演算
* リアルタイム処理
* メモリ制約がある場合

**f64を使用（精度優先）**
* 科学計算・金融計算
* 累積誤差が問題となる場合
* JavaScriptとの互換性重視

#### 4. コード生成前チェック項目

* [ ] `f32`を期待する箇所で、`f64`となりうる値（数値リテラル、`Math.*`関数の戻り値など）に`<f32>`キャストが適用されているか？
* [ ] 異なる浮動小数点型同士の演算が行われていないか？
* [ ] 定数定義で型を明示しているか？ (`const G: f32 = <f32>9.8`)

**重要**: AssemblyScriptの数値リテラルのデフォルトは`f64`です。`f32`として扱いたい場合は、必ず型を意識したコーディングが必要です。

---

## **📄 公式推奨方式互換による実装（v1.3.1）**

### **実装フローチャート**

```
AssemblyScriptコンパイラ読み込み開始
    ↓
バージョン動的取得
    ├─ キャッシュ確認 → あれば使用
    └─ 新規取得
        ├─ jsDelivr API
        ├─ jsDelivr package.json
        └─ UNPKG package.json
    ↓
CDN選択ループ開始
    ↓
1. jsDelivr試行
    ├─ web.js読み込み → import map設定 → asc import
    ├─ 成功 → 使用開始
    └─ 失敗 → 次のCDNへ
    ↓
2. UNPKG試行
    ├─ web.js読み込み → import map設定 → asc import
    ├─ 成功 → 使用開始
    └─ 失敗 → エラー表示
```

### **CDNプロバイダー設定（v1.3.1）**

```javascript
const CDN_PROVIDERS = [
    {
        name: 'jsDelivr',
        baseUrl: 'https://cdn.jsdelivr.net/npm',
        characteristics: '高速・グローバル最適化・AssemblyScript公式推奨'
    },
    {
        name: 'UNPKG',
        baseUrl: 'https://unpkg.com',
        characteristics: 'npm公式ミラー・完全性保証・安定性重視'
    }
];
```

### **読み込み処理の詳細実装**

```javascript
// CDNフォールバック対応ローダー（v1.3.1簡素化版）
async function loadCompilerWithCDNFallback() {
    const version = await VersionManager.getLatestVersion();
    
    for (let i = 0; i < CDN_PROVIDERS.length; i++) {
        const cdn = CDN_PROVIDERS[i];
        const startTime = performance.now();
        
        try {
            updateProgress(30 + (i * 35), `コンパイラ読み込み中 (${cdn.name})...`);
            
            // web.jsのURLを構築
            const webJsUrl = `${cdn.baseUrl}/assemblyscript@${version}/dist/web.js`;
            console.log(`[Loader] Attempting ${cdn.name}: ${webJsUrl}`);
            
            // web.jsを読み込み
            await loadScriptTag(webJsUrl);
            
            // import mapの設定を待機
            await waitForImportMap();
            
            // assemblyscript/ascをimport
            const asc = await import("assemblyscript/asc");
            const compiler = asc.default || asc;
            
            // コンパイラの検証
            await validateCompiler(compiler);
            
            // メタデータを更新
            const loadTime = Math.round(performance.now() - startTime);
            COMPILER_METADATA.cdnProvider = cdn.name;
            COMPILER_METADATA.loadTime = loadTime;
            COMPILER_METADATA.attempts = i + 1;
            COMPILER_METADATA.versionSource = 'Dynamic fetch';
            
            console.log(`[Loader] Success with ${cdn.name} in ${loadTime}ms`);
            updateProgress(100, 'コンパイラ準備完了');
            
            return compiler;
            
        } catch (error) {
            console.error(`[Loader] ${cdn.name} failed:`, error.message);
            
            // スクリプトタグをクリーンアップ
            cleanupFailedScripts();
        }
    }
    
    throw new Error(`All CDN providers failed for AssemblyScript ${version}`);
}

// スクリプトタグ読み込み関数
function loadScriptTag(url) {
    return new Promise((resolve, reject) => {
        // 既存のAssemblyScriptチェック
        if (window.ASSEMBLYSCRIPT_VERSION) {
            console.log('[Loader] AssemblyScript already loaded');
            resolve();
            return;
        }
        
        const script = document.createElement('script');
        script.src = url;
        script.dataset.assemblyScript = 'loading';
        
        // タイムアウト設定（10秒）
        const timeout = setTimeout(() => {
            script.remove();
            reject(new Error(`Timeout loading: ${url}`));
        }, 10000);
        
        script.onload = () => {
            clearTimeout(timeout);
            script.dataset.assemblyScript = 'loaded';
            resolve();
        };
        
        script.onerror = () => {
            clearTimeout(timeout);
            script.remove();
            reject(new Error(`Failed to load: ${url}`));
        };
        
        document.head.appendChild(script);
    });
}

// import map待機関数
async function waitForImportMap() {
    const maxAttempts = 20;
    const waitTime = 100;
    
    for (let i = 0; i < maxAttempts; i++) {
        if (window.ASSEMBLYSCRIPT_VERSION && window.ASSEMBLYSCRIPT_IMPORTMAP) {
            console.log('[Loader] Import map ready');
            return;
        }
        await new Promise(resolve => setTimeout(resolve, waitTime));
    }
    
    throw new Error('Import map setup timeout');
}

// 失敗したスクリプトのクリーンアップ
function cleanupFailedScripts() {
    const scripts = document.querySelectorAll('script[data-assembly-script]');
    scripts.forEach(script => {
        if (script.dataset.assemblyScript !== 'loaded') {
            script.remove();
        }
    });
    
    // グローバル変数もクリア
    delete window.ASSEMBLYSCRIPT_VERSION;
    delete window.ASSEMBLYSCRIPT_IMPORTMAP;
}
```

---

## **🔧 コンパイルオプション仕様**

### **基本オプション**

| オプション | デフォルト | 説明 |
|-----------|----------|------|
| `--optimize` | false | Binaryen最適化の有効化 |
| `--optimizeLevel` | 3 | 最適化レベル (0-3) |
| `--shrinkLevel` | 0 | サイズ削減レベル (0-2) |
| `--runtime` | minimal | ランタイム選択 (stub/minimal/incremental) |
| `--debug` | false | デバッグ情報の生成 |
| `--sourceMap` | false | ソースマップ生成 |
| `--noAssert` | false | アサーションの無効化 |

### **高度なオプション**

| オプション | 説明 | 使用例 |
|-----------|------|--------|
| `--importMemory` | WebAssemblyメモリをインポート | 共有メモリ使用時 |
| `--sharedMemory` | SharedArrayBuffer使用 | マルチスレッド処理 |
| `--exportRuntime` | ランタイムAPIのエクスポート | 高度なメモリ管理 |
| `--explicitStart` | _start関数の明示的呼び出し | 初期化制御 |
| `--lowMemoryLimit` | 低メモリ環境向け最適化 | 組み込みブラウザ |

### **プリセット定義**

```javascript
const PRESETS = {
    simple: {
        optimize: false,
        runtime: 'minimal'
    },
    debug: {
        optimize: false,
        optimizeLevel: '0',
        debug: true,
        sourceMap: true,
        measure: true
    },
    release: {
        optimize: true,
        optimizeLevel: '3',
        shrinkLevel: '1',
        noAssert: true
    },
    minimal: {
        optimize: true,
        optimizeLevel: '3',
        shrinkLevel: '2',
        runtime: 'stub',
        noAssert: true
    }
};
```

---

## **📦 モジュールアーキテクチャ**

### **Layer 1: コアインフラ（v1.4.1 コマンドプレビュー対応版）**

```javascript
// ============================================
// バージョン管理とメタデータ
// ============================================
const COMPILER_METADATA = {
    version: null,          // 動的取得
    versionSource: null,    // 取得元（Dynamic/Fallback）
    fetchedFrom: "NPM Registry via CDN",
    generatedAt: new Date().toISOString(),
    cdnProvider: null,      // 成功したCDNプロバイダー
    loadTime: null,         // 読み込み時間（ms）
    attempts: 0,            // 試行回数
    
    async initialize() {
        const startTime = performance.now();
        try {
            this.version = await VersionManager.getLatestVersion();
            this.versionSource = 'Dynamic fetch';
            console.log(`[Version] Fetched: v${this.version}`);
        } catch (error) {
            this.version = "0.28.8";
            this.versionSource = 'Hardcoded fallback';
            console.warn('[Version] Using fallback version');
        }
        const fetchTime = Math.round(performance.now() - startTime);
        console.log(`[Version] Time: ${fetchTime}ms`);
    }
};

// ============================================
// CDNプロバイダー定義（v1.3.1簡素化版）
// ============================================
const CDN_PROVIDERS = [
    {
        name: 'jsDelivr',
        baseUrl: 'https://cdn.jsdelivr.net/npm',
        timeout: 10000,
        priority: 1
    },
    {
        name: 'UNPKG',
        baseUrl: 'https://unpkg.com',
        timeout: 10000,
        priority: 2
    }
];

// ============================================
// バージョン管理システム（v1.3.1新規）
// ============================================
const VersionManager = {
    // 実装は上記ステップ0.5を参照
};

// ============================================
// コンパイラローダー（公式互換方式統一版）
// ============================================
const CompilerLoader = {
    compiler: null,
    
    async load() {
        // バージョンを動的取得
        await COMPILER_METADATA.initialize();
        
        // CDNフォールバックでロード
        return await loadCompilerWithCDNFallback();
    }
};

// ============================================
// コンパイラコア
// ============================================
const CompilerCore = {
    compiler: null,
    
    init(ascInstance) {
        this.compiler = ascInstance;
        return true;
    },
    
    async compile(sourceCode, options) {
        if (!this.compiler) {
            throw new Error('Compiler not initialized');
        }
        
        const compileOptions = this.buildOptions(options);
        let wasmBinary = null;
        let compileStats = '';
        
        const { error, stdout, stderr } = await this.compiler.main(compileOptions, {
            readFile: (filename) => {
                if (filename === "main.ts") return sourceCode;
                return null;
            },
            writeFile: (filename, contents) => {
                if (filename === "main.wasm") {
                    wasmBinary = contents;
                }
            },
            listFiles: () => ["main.ts"]
        });
        
        if (error) throw new Error(stderr || error.toString());
        if (!wasmBinary) throw new Error('No WebAssembly binary generated');
        
        if (stdout) compileStats = stdout;
        
        return { wasmBinary, stats: compileStats };
    },
    
    buildOptions(userOptions) {
        const flags = ["main.ts", "-o", "main.wasm"];
        
        Object.entries(userOptions).forEach(([key, value]) => {
            if (typeof value === 'boolean' && value) {
                flags.push(`--${key}`);
            } else if (typeof value !== 'boolean' && value !== '') {
                flags.push(`--${key}`, value.toString());
            }
        });
        
        return flags;
    }
};

// ============================================
// オプション管理システム（v1.4.1 プレビュー対応）
// ============================================
const OptionsManager = {
    current: {},
    
    init() {
        this.current = { runtime: 'minimal' };
        this.loadFromStorage();
        this.updateUI();
    },
    
    apply(preset) {
        if (PRESETS[preset]) {
            this.current = { ...PRESETS[preset] };
            this.updateUI();
            this.saveToStorage();
        }
    },
    
    collectFromUI() {
        const options = {};
        
        // Boolean options
        ['optimize', 'exportRuntime', 'noAssert', 'debug', 'sourceMap',
         'measure', 'validate', 'importMemory', 'sharedMemory',
         'exportTable', 'explicitStart', 'lowMemoryLimit'].forEach(opt => {
            const el = document.getElementById(`opt-${opt}`);
            if (el && el.checked) options[opt] = true;
        });
        
        // Select options
        ['optimizeLevel', 'shrinkLevel', 'runtime'].forEach(opt => {
            const el = document.getElementById(`opt-${opt}`);
            if (el && el.value) options[opt] = el.value;
        });
        
        this.current = options;
        this.saveToStorage();
        
        // ⭐ コマンドラインプレビュー更新（v1.4.1）
        updateCmdPreview();
        
        return options;
    },
    
    updateUI() {
        Object.keys(this.current).forEach(key => {
            const el = document.getElementById(`opt-${key}`);
            if (el) {
                if (el.type === 'checkbox') {
                    el.checked = this.current[key] === true;
                } else {
                    el.value = this.current[key] || '';
                }
            }
        });
    },
    
    saveToStorage() {
        try {
            localStorage.setItem('asmOptions', JSON.stringify(this.current));
        } catch(e) {}
    },
    
    loadFromStorage() {
        try {
            const saved = localStorage.getItem('asmOptions');
            if (saved) this.current = JSON.parse(saved);
        } catch(e) {}
    }
};

// ============================================
// コマンドラインプレビュー機能（v1.4.1新規）
// ============================================
function updateCmdPreview() {
    const options = OptionsManager.collectFromUI();
    const cmd = generateCmdLine(options);
    
    const preview = document.getElementById('cmd-preview');
    if (preview) {
        preview.textContent = cmd;
    }
}

function generateCmdLine(options) {
    const parts = ['asc', 'main.ts', '-o', 'main.wasm'];
    
    Object.entries(options).forEach(([key, value]) => {
        if (typeof value === 'boolean' && value) {
            parts.push(`--${key}`);
        } else if (typeof value !== 'boolean' && value !== '') {
            parts.push(`--${key}`, value.toString());
        }
    });
    
    return parts.join(' ');
}

// ============================================
// WebAssembly実行管理
// ============================================
const WasmRunner = {
    instance: null,
    
    async instantiate(wasmBinary, imports = {}) {
        const defaultImports = {
            env: {
                abort: (msg, file, line, column) => {
                    console.error(`Abort: ${msg} at ${file}:${line}:${column}`);
                },
                trace: (msg, n, ...args) => {
                    console.log(`Trace: ${msg}`, ...args);
                },
                seed: () => Date.now(),
                memory: new WebAssembly.Memory({ initial: 1 })
            },
            Math: Math,
            Date: Date
        };
        
        const mergedImports = this.mergeImports(defaultImports, imports);
        
        let buffer;
        if (wasmBinary instanceof ArrayBuffer) {
            buffer = wasmBinary;
        } else if (wasmBinary instanceof Uint8Array) {
            buffer = wasmBinary.buffer.slice(wasmBinary.byteOffset, 
                                             wasmBinary.byteOffset + wasmBinary.byteLength);
        } else {
            throw new Error('Unexpected binary format');
        }
        
        const wasmModule = await WebAssembly.compile(buffer);
        this.instance = await WebAssembly.instantiate(wasmModule, mergedImports);
        
        return this.instance.exports;
    },
    
    mergeImports(defaults, custom) {
        const merged = { ...defaults };
        Object.keys(custom).forEach(key => {
            if (typeof custom[key] === 'object' && !Array.isArray(custom[key])) {
                merged[key] = { ...defaults[key], ...custom[key] };
            } else {
                merged[key] = custom[key];
            }
        });
        return merged;
    }
};

// ============================================
// プリセット定義
// ============================================
const PRESETS = {
    simple: {
        optimize: false,
        runtime: 'minimal'
    },
    debug: {
        optimize: false,
        optimizeLevel: '0',
        debug: true,
        sourceMap: true,
        measure: true
    },
    release: {
        optimize: true,
        optimizeLevel: '3',
        shrinkLevel: '1',
        noAssert: true
    },
    minimal: {
        optimize: true,
        optimizeLevel: '3',
        shrinkLevel: '2',
        runtime: 'stub',
        noAssert: true
    }
};
```

### **Layer 2: アプリケーションモジュール（カスタマイズ領域）**

```javascript
// AIが生成する部分のプレースホルダー
const AppModule = {
    // モジュール識別情報
    name: "UserDefinedApp",
    description: "ユーザー定義のアプリケーション",
    
    // AssemblyScriptソースコード
    sourceCode: `
        // ユーザーの要求に基づいてAIが生成
        export function main(): void {
            // アプリケーションロジック
        }
    `,
    
    // 推奨コンパイルオプション
    defaultOptions: {
        optimize: false,
        runtime: 'minimal'
    },
    
    // WebAssemblyインポート定義
    getImports() {
        return {
            // 必要に応じてカスタムインポート
        };
    },
    
    // 初期化処理
    async init(wasmExports) {
        // 初期化ロジック
    },
    
    // メインループ実行
    execute(wasmExports, deltaTime) {
        // フレーム毎の処理
    },
    
    // UI更新
    render(data) {
        // 描画・表示更新
    },
    
    // クリーンアップ
    cleanup() {
        // リソース解放
    }
};
```

---

## **📝 HTMLテンプレート（v1.4.1）**

HTMLテンプレートの完全なコードは、別ファイルで提供されています：

### **📄 ASMSCRIPT-OPTIMIZER-RECIPE-HTMLTEMPLATE.md**

このファイルには以下が含まれています：
- 完全なHTML構造（約1500行）
- デフォルトデモ（カウンターアプリ）
- すべてのUIコンポーネント
- CDNフォールバック機構
- 動的バージョン取得システム
- コマンドラインプレビュー機能（v1.4.1新機能）

実装時は、HTMLテンプレートファイルのコードをそのまま使用し、
AppModuleセクションをユースケースに応じて置き換えてください。

### ユーザーがテンプレートファイルを添付し忘れている等により「ASMSCRIPT-OPTIMIZER-RECIPE-HTMLTEMPLATE.md」を参照できない場合

1. **作業を中断し**、「ASMSCRIPT-OPTIMIZER-RECIPE-HTMLTEMPLATE.md」のファイル添付を促して下さい。
2. 「ASMSCRIPT-OPTIMIZER-RECIPE-HTMLTEMPLATE.md」のファイル添付を確認出来たら作業を再開して下さい。

---

## **🤖 汎用AI実装ガイド**

### **ステップ0実行の重要性**

生成AIがHTMLを生成する際、**必ず最初にステップ0を実行**してください：

1. **ユーザー要求の明確性を確認**
2. **不明確な場合は作業を中断して問い合わせ**
3. **明確な回答を得てから実装を開始**
4. **デフォルトアプリの自動生成は厳禁**

### **動的バージョン取得の実装**

生成AIがHTMLを生成する際、以下の手順で最新版を取得してください：

1. **VersionManagerを組み込む**
2. **jsDelivr API優先で試行**
3. **フォールバックとして0.28.8を使用**
4. **キャッシュ機構を活用**

### **公式推奨方式互換の遵守**

生成AIがHTMLを生成する際、以下の実装方針を厳守してください：

1. **直接importは絶対に使用しない**
2. **必ずweb.js経由でロード**
3. **CDNはjsDelivrとUNPKGの2つのみ**
4. **import mapの設定を待機**

### **コマンドラインプレビューの実装（v1.4.1）**

生成AIがHTMLを生成する際、以下を必ず実装してください：

1. **WebAssemblyタブ最上部にプレビューセクション追加**
2. **updateCmdPreview()関数の実装**
3. **各オプション要素にonchange属性追加**
4. **OptionsManager.collectFromUI()に更新呼び出し追加**

### **実装フローチャート**

```
0. ユーザー要求を受信
   ↓
1. 要求の明確性を確認（ステップ0）
   ├─ 不明確 → ユーザーに問い合わせ → 回答待ち
   └─ 明確 → 次へ進行
   ↓
2. 動的バージョン取得
   ↓
3. ユースケース判定
   ↓
4. AssemblyScriptコード生成
   ↓
5. AppModule生成
   ↓
6. HTMLテンプレートv1.4.1を使用
   ↓
7. 単一HTMLファイルとして出力
```

### **ユースケース別実装例**

#### **1. 物理シミュレーション（ボール）**

```javascript
const AppModule = {
    name: "Ball Physics Demo",
    description: "物理演算によるボールの動きシミュレーション",
    
    sourceCode: `
export class Ball {
    x: f32 = <f32>300.0;
    y: f32 = <f32>200.0;
    vx: f32 = <f32>3.0;
    vy: f32 = <f32>2.0;
    radius: f32 = <f32>15.0;
}

let ball = new Ball();

export function update(): void {
    ball.x += ball.vx;
    ball.y += ball.vy;
    
    if (ball.x - ball.radius <= 0 || ball.x + ball.radius >= 600) {
        ball.vx = -ball.vx;
    }
    if (ball.y - ball.radius <= 0 || ball.y + ball.radius >= 400) {
        ball.vy = -ball.vy;
    }
}

export function getX(): f32 { return ball.x; }
export function getY(): f32 { return ball.y; }
export function getRadius(): f32 { return ball.radius; }
    `.trim(),
    
    async init(wasmExports) {
        const viewport = document.getElementById('app-viewport');
        viewport.innerHTML = '<canvas id="canvas" width="600" height="400" style="border: 1px solid #e5e7eb; border-radius: 8px;"></canvas>';
    },
    
    execute(wasmExports) {
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        
        function animate() {
            wasmExports.update();
            
            ctx.fillStyle = '#f8f9fa';
            ctx.fillRect(0, 0, 600, 400);
            
            const x = wasmExports.getX();
            const y = wasmExports.getY();
            const r = wasmExports.getRadius();
            
            ctx.fillStyle = '#667eea';
            ctx.beginPath();
            ctx.arc(x, y, r, 0, Math.PI * 2);
            ctx.fill();
            
            requestAnimationFrame(animate);
        }
        animate();
    }
};
```

#### **2. 画像フィルタ処理（v1.4.0 正常動作版）**

```javascript
const AppModule = {
    name: "Image Filter Processor",
    description: "高速画像フィルタ処理",
    
    sourceCode: `
// グローバル画像バッファ（再利用可能）
let imageBuffer: Uint8ClampedArray | null = null;

// バッファの初期化または再割り当て
export function initBuffer(size: i32): void {
    imageBuffer = new Uint8ClampedArray(size);
}

// バッファのポインタ取得（JavaScript側からのデータコピー用）
export function getBufferPtr(): usize {
    if (!imageBuffer) return 0;
    return imageBuffer!.dataStart;
}

// グレースケール変換（TypedArrayベース）
export function applyGrayscale(): void {
    if (!imageBuffer) return;
    
    let data = imageBuffer!;
    let length = data.length;
    
    for (let i = 0; i < length; i += 4) {
        let r = unchecked(data[i]);
        let g = unchecked(data[i + 1]);
        let b = unchecked(data[i + 2]);
        
        // ITU-R BT.601規格のグレースケール計算
        let gray = <u8>(
            <f32>r * <f32>0.299 + 
            <f32>g * <f32>0.587 + 
            <f32>b * <f32>0.114
        );
        
        unchecked(data[i] = gray);
        unchecked(data[i + 1] = gray);
        unchecked(data[i + 2] = gray);
        // Alpha値（i + 3）は変更しない
    }
}

// セピアトーン効果
export function applySepia(): void {
    if (!imageBuffer) return;
    
    let data = imageBuffer!;
    let length = data.length;
    
    for (let i = 0; i < length; i += 4) {
        let r = unchecked(data[i]);
        let g = unchecked(data[i + 1]);
        let b = unchecked(data[i + 2]);
        
        let tr = <f32>r * <f32>0.393 + <f32>g * <f32>0.769 + <f32>b * <f32>0.189;
        let tg = <f32>r * <f32>0.349 + <f32>g * <f32>0.686 + <f32>b * <f32>0.168;
        let tb = <f32>r * <f32>0.272 + <f32>g * <f32>0.534 + <f32>b * <f32>0.131;
        
        unchecked(data[i] = <u8>Math.min(255, tr));
        unchecked(data[i + 1] = <u8>Math.min(255, tg));
        unchecked(data[i + 2] = <u8>Math.min(255, tb));
    }
}

// 反転効果
export function applyInvert(): void {
    if (!imageBuffer) return;
    
    let data = imageBuffer!;
    let length = data.length;
    
    for (let i = 0; i < length; i += 4) {
        unchecked(data[i] = 255 - data[i]);
        unchecked(data[i + 1] = 255 - data[i + 1]);
        unchecked(data[i + 2] = 255 - data[i + 2]);
    }
}

// 明度調整
export function adjustBrightness(factor: f32): void {
    if (!imageBuffer) return;
    
    let data = imageBuffer!;
    let length = data.length;
    
    for (let i = 0; i < length; i += 4) {
        let r = <f32>unchecked(data[i]) * factor;
        let g = <f32>unchecked(data[i + 1]) * factor;
        let b = <f32>unchecked(data[i + 2]) * factor;
        
        unchecked(data[i] = <u8>Math.max(0, Math.min(255, r)));
        unchecked(data[i + 1] = <u8>Math.max(0, Math.min(255, g)));
        unchecked(data[i + 2] = <u8>Math.max(0, Math.min(255, b)));
    }
}

// メモリ統計情報（デバッグ用）
export function getMemoryStats(): i32 {
    return memory.size(); // ページ数を返す
}

// バッファサイズ取得
export function getBufferSize(): i32 {
    return imageBuffer ? imageBuffer!.length : 0;
}
    `.trim(),
    
    defaultOptions: {
        optimize: true,
        optimizeLevel: '3',
        shrinkLevel: '1',
        noAssert: true,
        runtime: 'minimal',
        importMemory: false,  // 内部メモリを使用
        exportRuntime: false  // ランタイム関数は不要
    },
    
    // JavaScriptでのフィルタ適用処理
    applyFilter(wasmExports, filterType, imageData) {
        const dataLength = imageData.data.length;
        
        // バッファを初期化
        wasmExports.initBuffer(dataLength);
        const bufferPtr = wasmExports.getBufferPtr();
        
        if (bufferPtr === 0) {
            throw new Error('Failed to initialize buffer');
        }
        
        // WebAssemblyメモリにデータをコピー
        const wasmMemory = new Uint8ClampedArray(wasmExports.memory.buffer);
        for (let i = 0; i < dataLength; i++) {
            wasmMemory[bufferPtr + i] = imageData.data[i];
        }
        
        // フィルタ適用
        switch(filterType) {
            case 'grayscale':
                wasmExports.applyGrayscale();
                break;
            case 'sepia':
                wasmExports.applySepia();
                break;
            case 'invert':
                wasmExports.applyInvert();
                break;
            case 'bright':
                wasmExports.adjustBrightness(1.3);
                break;
            case 'dark':
                wasmExports.adjustBrightness(0.7);
                break;
        }
        
        // 処理済みデータを取得
        const updatedWasmMemory = new Uint8ClampedArray(wasmExports.memory.buffer);
        const updatedBufferPtr = wasmExports.getBufferPtr();
        
        for (let i = 0; i < dataLength; i++) {
            imageData.data[i] = updatedWasmMemory[updatedBufferPtr + i];
        }
        
        return imageData;
    }
};
```

---

## **📚 付録**

### **AssemblyScript型対応表**

| AssemblyScript | JavaScript | 用途 |
|---------------|------------|------|
| i32 | number | 整数演算 |
| f32 | number | 単精度浮動小数点 |
| f64 | number | 倍精度浮動小数点 |
| i64 | BigInt | 大きな整数 |
| bool | boolean | 真偽値 |

### **パフォーマンス指標**

- 初回コンパイル: < 5秒
- 再コンパイル: < 3秒
- WASMサイズ: 1KB～50KB（オプション依存）
- 実行速度: JavaScriptの2～10倍（処理内容依存）

### **トラブルシューティング**

| 問題 | 原因 | 解決策 |
|------|------|---------|
| "import map not found" | web.js読み込み失敗 | CDNフォールバックが自動対応 |
| コンパイラ読み込み失敗 | CDN障害/ネットワーク | 2つのCDNで自動リトライ |
| タイムアウト | 遅いネットワーク | 次のCDNに自動切り替え |
| バージョン不一致 | 破壊的変更 | 動的バージョン取得が対応 |
| 低パフォーマンス | 最適化なし | --optimizeオプションを有効化 |
| 大きなWASMサイズ | デバッグ情報 | --debugを無効化、--shrinkLevelを上げる |
| フィルタが適用されない | メモリ管理の誤り | TypedArrayとuncheckedを使用 |
| 型変換エラー | 暗黙的型変換 | 明示的な<f32>キャストを追加 |

### **デバッグのヒント**

```javascript
// ローカルストレージでデバッグモード有効化
localStorage.setItem('debug', 'true');

// コンパイラ情報パネルが表示される（v1.3.1強化）
// - AssemblyScriptバージョン
// - バージョン取得元（Dynamic/Fallback）
// - 使用したCDNプロバイダー（色分け表示）
// - 読み込み時間
// - 試行回数

// コンソールで詳細ログ確認
// [Version] タグでバージョン取得状況
// [Loader] タグで読み込み進行状況
// [Init] タグで初期化状況
// [Progress] タグで進捗状況
// [Compile] タグでエクスポート関数確認
// [Filter] タグで画像処理の詳細
```

### **CDNプロバイダー詳細**

| CDN | URL | 特徴 | 推奨用途 |
|-----|-----|------|----------|
| **jsDelivr** | cdn.jsdelivr.net | 最速・グローバルCDN・公式推奨 | 第1選択 |
| **UNPKG** | unpkg.com | npm公式ミラー・完全性保証 | フォールバック |

---

## **📄 ライセンス**

MITライセンス - 自由に使用・改変・再配布・商用利用可能

---

## **📄 更新履歴**

### **v1.4.1** (2025-01) - コマンドラインプレビュー機能追加
**主な変更内容：**
- ✅ **コマンドラインプレビュー機能**: WebAssemblyタブ最上部に追加
- ✅ **リアルタイム更新**: オプション変更時に即座に反映
- ✅ **全オプション対応**: UIで設定可能なすべてのオプションを反映
- ✅ **プリセット連動**: プリセット適用時も自動更新
- ✅ **シンプル実装**: コピー機能を省略し必要最小限の実装

**技術的改善：**
- updateCmdPreview()関数による一元管理
- generateCmdLine()による柔軟なコマンド生成
- onchange属性による即時反応
- user-select: textによる選択可能なテキスト

### **v1.4.0** (2025-01) - メモリ管理とサンプル修正
**主な変更内容：**
- ✅ **画像フィルタサンプルの修正**: TypedArrayベースの正常動作版に更新
- ✅ **メモリ管理説明の追加**: 標準的なメモリアクセス方法の詳細説明
- ✅ **型変換の改善**: すべての数値リテラルに明示的キャスト
- ✅ **デバッグ機能強化**: エクスポート関数とメモリ状態の確認
- ✅ **ベストプラクティス追加**: unchecked最適化とグローバルバッファ
- ✅ **HTMLテンプレート分離**: 別ファイル化により保守性向上

**技術的改善：**
- TypedArrayによる型安全なメモリアクセス
- unchecked()による境界チェック省略
- グローバルバッファによるGC負荷軽減
- load/store組み込み関数の適切な使い分け

### **v1.3.1** (2025-01) - 安定性とシンプル化の向上
**主な変更内容：**
- ✅ **動的バージョン取得機能追加**: jsDelivr API優先での最新版取得
- ✅ **CDN最適化**: esm.shを除外し2プロバイダー体制へ
- ✅ **キャッシュ機構**: セッションストレージによる高速化
- ✅ **初期化時間短縮**: 最悪ケース30秒→20秒
- ✅ **デバッグ情報強化**: バージョン取得元の可視化

### **v1.3.0** (2025-09) - 公式推奨方式互換への完全移行
**主な変更内容：**
- ✅ **読み込み方式の統一**: 公式互換のweb.js方式のみを使用
- ✅ **直接importの削除**: 問題の多い直接import方式を完全削除
- ✅ **3つのCDNプロバイダー**: jsDelivr → UNPKG → esm.sh
- ✅ **実装の簡素化**: 単一パターンで保守性大幅向上

### **v1.2.2** (2025-09) - 公式互換実装フォールバック機能の追加
**主な変更内容：**
- ✅ **3段階フォールバック機構**: 直接import → 代替CDN → 公式web.js
- ✅ **動的バージョン管理**: GitHubタグから最新安定版を取得
- ✅ **@latest禁止ルール**: 具体的バージョン指定を必須化
- ✅ **診断機能強化**: 読み込み方法のロギングとメタデータ記録

### **v1.2.1** (2025-09) - ユーザー要求確認プロセスの追加
**主な変更内容：**
- ✅ **ステップ0追加**: ユーザー要求が不明確な場合の確認プロセスを必須化
- ✅ **作業中断ルール**: 曖昧な指示での自動判断を禁止
- ✅ **問い合わせテンプレート**: 構造化された質問フォーマットを提供

### **v1.2** (2025-09) - UI/UX全面改良
**主な変更内容：**
- ✅ **ハンバーガーメニュー採用**: サイドメニューによる効率的な画面利用
- ✅ **タブ型設定管理**: アプリ設定とWebAssembly設定を論理的に分離
- ✅ **WASM統計情報トグル**: デフォルトOFFで必要時のみ表示
- ✅ **レスポンシブ対応**: モバイルからデスクトップまで

### **v1.1** (2025-09) - レースコンディション問題の解決
**主な変更内容：**
- ✅ **Dynamic Import採用**: スクリプトタグによる直接読み込みを削除
- ✅ **リトライメカニズム追加**: 自動リトライとExponential Backoff
- ✅ **タイムアウト保護**: コンパイラ読み込みに上限設定

### **v1.0** (2025-09) - 初版リリース
- モジュール化アーキテクチャ採用
- 自動初期化機能
- 15種類のコンパイルオプション
- エラーオーバーレイ機能

---

## **📊 バージョン別機能比較**

| 機能 | v1.0 | v1.1 | v1.2 | v1.2.1 | v1.2.2 | v1.3.0 | v1.3.1 | v1.4.0 | v1.4.1 |
|------|------|------|------|--------|--------|--------|--------|--------|--------|
| 基本コンパイル | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| レースコンディション対策 | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| モダンUI | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| ユーザー要求確認 | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 公式互換フォールバック | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ |
| 動的バージョン管理 | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 公式方式互換 | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| jsDelivr優先 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| 2CDN体制 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| メモリ管理説明 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| 正常動作サンプル | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| HTMLテンプレート分離 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| コマンドラインプレビュー | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

---

**生成日**: 2025年1月  
**レシピバージョン**: 1.4.1  
**AssemblyScript対応バージョン**: 動的取得（フォールバック: 0.28.8）
