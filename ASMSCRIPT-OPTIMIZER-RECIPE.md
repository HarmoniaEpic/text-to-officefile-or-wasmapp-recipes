# **ASMSCRIPT-OPTIMIZER-RECIPE v1.2.3**
**AssemblyScript WebAssembly コンパイラ統合型単一HTMLアプリケーション生成レシピ**

---

## **📌 このレシピについて**

生成AIがユーザーの要求を解釈し、AssemblyScriptコードをブラウザ内でWebAssemblyにコンパイル・実行する**高性能単一HTMLファイル**を自動生成するための汎用指示書です。本レシピは特定のAI環境に依存せず、あらゆる生成AIで利用可能な設計となっています。

### **v1.2.3の主な改善点（公式実装フォールバック機能）**
- 🔄 **3段階フォールバック**: 直接import → 代替CDN → 公式web.js方式
- 🏷️ **動的バージョン管理**: GitHubタグから最新安定版を取得し使用
- 📊 **診断機能強化**: どの方法で読み込みに成功したかを記録
- 🛡️ **互換性向上**: より多くの環境で確実に動作

### **v1.2.1の主な改善点（ユーザー要求確認プロセス）**
- 🎯 **ステップ0の追加**: 曖昧な要求に対する確認プロセスを必須化
- 🚫 **自動判断の禁止**: 不明確な指示での勝手な実装を防止
- 📝 **構造化された問い合わせ**: テンプレートによる明確な要件収集
- ✅ **品質向上**: ユーザーの真のニーズに合致した生成物

### **v1.2の主な改善点（UI/UX全面改良）**
- 🎨 **モダンUIデザイン**: ハンバーガーメニューによる効率的な画面利用
- 📱 **完全レスポンシブ対応**: モバイルからデスクトップまで最適表示
- 🗂️ **タブ型設定管理**: アプリ設定とWebAssembly設定を論理的に分離
- 📊 **スマート統計表示**: WASM統計情報の表示/非表示をトグル可能（デフォルトOFF）
- 🎯 **フォーカスモード**: メイン画面をアプリケーション実行に集中

### **v1.1からの継承機能**
- 🔧 **レースコンディション問題の完全解決**: Dynamic Import with Retryメカニズム
- 🛡️ **堅牢なエラーハンドリング**: 複数のフォールバック戦略
- ⏱️ **タイムアウト保護**: コンパイラ読み込みに上限時間を設定
- 🔄 **自動リトライ**: Exponential Backoffによる段階的な再試行

### **v1.0からの継承機能**
- 🚀 **自動初期化**: ページロード時に自動的にコンパイル・実行開始
- 🔧 **15種類以上の最適化オプション**: 詳細なコンパイル制御
- 📦 **モジュール化アーキテクチャ**: 多様なユースケースに対応
- ⚡ **CDNベース**: 外部ツール不要、ブラウザのみで完結

### **解決済みの技術課題**
- **問題**: CDNからのスクリプト読み込みとメインコードの実行タイミングの競合（レースコンディション）
- **対策**: 本レシピでは完全に解決済み（Dynamic Import + リトライ + タイムアウト保護 + 公式フォールバック）

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
🤔 **AssemblyScriptで実装する内容を教えてください**

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

#### **問い合わせが必要な典型的なケース**

| ユーザー入力 | AIの対応 |
|------------|---------|
| 「AssemblyScriptで何か作って」 | → 作業中断、上記テンプレートで問い合わせ |
| 「最適化して」（コンテキストなし） | → 何を最適化したいか問い合わせ |
| 「高速にしたい」（対象不明） | → 何を高速化したいか問い合わせ |
| 「このレシピを使って」（用途不明） | → 具体的な実装内容を問い合わせ |

#### **問い合わせ不要で進行可能なケース**

| ユーザー入力 | AIの対応 |
|------------|---------|
| 「フィボナッチ数列を計算するアプリ」 | → ステップ1へ進行 |
| 「画像をグレースケールに変換」 | → ステップ1へ進行 |
| 「3Dの立方体を回転表示」 | → ステップ1へ進行 |
| 「CSVデータをグラフ化」 | → ステップ1へ進行 |

#### **重要な実装ルール**
- **曖昧な要求では絶対に勝手に判断しない**
- **デモ用のカウンターアプリをデフォルトで作らない**（明示的要求がない限り）
- **ユーザーが「何でもいい」と言っても、最低限の用途確認を行う**
- **問い合わせ後の回答を受けてから、ステップ1以降を実行**

### **ステップ0.5: AssemblyScriptバージョンの確認（v1.2.3新規）**

HTMLを生成する前に、AIは以下の手順で最新の安定版バージョンを確認します：

#### **GitHubタグからのバージョン取得**

1. **GitHubタグページの確認**
   - URL: https://github.com/AssemblyScript/assemblyscript/tags
   - 最新の安定版リリースタグを確認（例: v0.27.30）
   - プレリリース版（-dev, -rc, -alpha, -beta等）は使用しない

2. **バージョン番号の抽出**
   ```
   取得例: "v0.27.30" → "0.27.30"（vプレフィックスを除去）
   ```

3. **バージョン定数の生成**
   ```javascript
   // AIが生成すべきコード
   const ASSEMBLYSCRIPT_VERSION = "0.28.8"; // GitHubタグから取得した最新安定版
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
2. **AssemblyScriptバージョンを定数として埋め込み**（v1.2.3新規）
3. AppModuleを埋め込み
4. 必要に応じてUIカスタマイズ部分を追加
5. 単一HTMLファイルとして出力

---

## **🚨 実装ルール（v1.2.3 公式フォールバック対応版）**

### **🔴 絶対禁止事項（レースコンディション防止）**

1. **スクリプトタグによる直接読み込みの禁止**
   ```html
   <!-- ❌ 絶対に使用禁止（レースコンディションの原因） -->
   <script src="https://cdn.jsdelivr.net/npm/assemblyscript@latest/dist/web.js"></script>
   
   <!-- ✅ 必ずDynamic Importを使用 -->
   <script type="module">
     const asc = await import("https://cdn.jsdelivr.net/npm/assemblyscript@latest/dist/asc/index.js");
   </script>
   ```

2. **@latestタグの使用禁止（v1.2.3新規）**
   ```javascript
   // ❌ 絶対に使用禁止
   const url = "assemblyscript@latest/dist/web.js";
   
   // ✅ 必ず具体的バージョンを指定
   const ASSEMBLYSCRIPT_VERSION = "0.27.30"; // GitHubタグから取得
   const url = `assemblyscript@${ASSEMBLYSCRIPT_VERSION}/dist/web.js`;
   ```

3. **コンパイラ読み込みの必須リトライメカニズム**
   ```javascript
   // 必須：3段階のフォールバックと段階的な待機時間
   async function loadCompiler(retries = 3) {
       // 1. 直接import
       // 2. 代替CDN
       // 3. 公式web.js方式（v1.2.3新規）
   }
   ```

4. **初期化進捗の可視化**
   ```javascript
   // 必須：各段階での進捗表示
   updateProgress(0, "環境を検出中...");
   updateProgress(20, "コンパイラを読み込み中...");
   updateProgress(40, "フォールバック試行中..."); // v1.2.3新規
   updateProgress(60, "コンパイラを検証中...");
   updateProgress(80, "初期コンパイル中...");
   updateProgress(100, "準備完了");
   ```

### **🟡 必須ルール（v1.2.3 フォールバック・要求確認・UI要件）**

1. **ステップ0の必須実行**
   - ユーザー要求が不明確な場合は必ず作業を中断
   - 構造化されたテンプレートで要件を確認
   - デフォルトアプリの自動生成は禁止
   - 明確な要求を得てから次のステップへ進行

2. **動的バージョン管理（v1.2.3新規）**
   - GitHubタグから最新安定版を取得
   - 具体的なバージョン番号を定数化
   - すべてのCDN URLにバージョンを明示

3. **3段階フォールバック機構（v1.2.3新規）**
   - 第1段階: 直接import（jsdelivr）
   - 第2段階: 代替CDN（unpkg）
   - 第3段階: 公式web.js方式

4. **ハンバーガーメニューの実装**
   - 左上に3本線のトグルボタン
   - クリックでサイドメニューをスライドイン/アウト
   - オーバーレイ表示でメニュー外クリックで閉じる

5. **タブ型設定管理**
   - アプリ設定タブ：一般的なアプリケーション設定
   - WebAssemblyタブ：技術的なコンパイル設定

6. **WASM統計情報の適切な配置**
   - デフォルトで非表示（OFF）
   - WebAssemblyタブ内にトグルスイッチ配置
   - ONの場合、画面下部にステータスバー表示

7. **レスポンシブデザインの実装**
   - モバイル（〜480px）、タブレット（〜768px）、デスクトップ（768px〜）
   - フレキシブルレイアウトで画面幅に応じて調整

8. **初期コンパイルは5秒以内に完了**

### **🟢 推奨事項**

1. **アクセシビリティ**
   - キーボードナビゲーション対応
   - ARIAラベルの適切な使用

2. **パフォーマンス最適化**
   - 不要な再レンダリングの防止
   - アニメーションはCSS Transitionで実装

3. **ユーザー設定の永続化**
   - LocalStorageで設定を保存
   - 次回アクセス時に復元

4. **診断情報の記録（v1.2.3新規）**
   - 成功した読み込み方法をコンソールに出力
   - バージョン情報を含むメタデータを記録

---

## **🔄 動的バージョン取得によるフォールバック実装（v1.2.3新規）**

### **実装フローチャート**

```
AssemblyScriptコンパイラ読み込み開始
    ↓
Phase 1: 直接import試行
    ├─ jsdelivr CDN（バージョン指定）
    │   ├─ 成功 → 使用開始
    │   └─ 失敗 → 次へ
    └─ unpkg CDN（バージョン指定）
        ├─ 成功 → 使用開始
        └─ 失敗 → Phase 2へ
    ↓
Phase 2: 公式web.js方式試行
    ├─ web.jsスクリプト読み込み（バージョン指定）
    ├─ import map設定待機
    └─ "assemblyscript/asc"をimport
        ├─ 成功 → 使用開始
        └─ 失敗 → エラー表示
```

### **バージョン管理の実装詳細**

```javascript
// AIが生成すべきコード構造
const COMPILER_METADATA = {
    version: "0.27.30",         // GitHubタグから取得した最新安定版
    fetchedFrom: "GitHub Tags",  // バージョン情報の取得元
    generatedAt: new Date().toISOString(),
    fallbackEnabled: true,
    loadMethod: null            // 実際に成功した読み込み方法を記録
};

async function loadCompilerWithFallback() {
    const version = COMPILER_METADATA.version;
    
    // Phase 1: 直接import（バージョン指定付き）
    const directUrls = [
        `https://cdn.jsdelivr.net/npm/assemblyscript@${version}/dist/asc/index.js`,
        `https://unpkg.com/assemblyscript@${version}/dist/asc/index.js`
    ];
    
    for (const url of directUrls) {
        try {
            const module = await import(url);
            COMPILER_METADATA.loadMethod = `direct import: ${url}`;
            console.log(`[Success] Loaded AssemblyScript ${version} via ${url}`);
            return module.default || module;
        } catch (e) {
            console.warn(`[Failed] ${url}: ${e.message}`);
        }
    }
    
    // Phase 2: 公式web.js方式（バージョン指定付き）
    try {
        const webJsUrl = `https://cdn.jsdelivr.net/npm/assemblyscript@${version}/dist/web.js`;
        console.log(`[Fallback] Attempting official method with ${webJsUrl}`);
        
        // web.jsを動的に読み込み
        await loadScriptTag(webJsUrl);
        
        // import mapが設定された後でimport
        const asc = await import("assemblyscript/asc");
        COMPILER_METADATA.loadMethod = `official web.js`;
        console.log(`[Success] Loaded AssemblyScript ${version} via official web.js method`);
        return asc.default || asc;
    } catch (e) {
        console.error(`[Fatal] Failed to load AssemblyScript ${version}: ${e.message}`);
        throw new Error(`All loading methods failed for AssemblyScript ${version}`);
    }
}
```

### **web.jsローダーヘルパー関数**

```javascript
function loadScriptTag(src) {
    return new Promise((resolve, reject) => {
        // 既に読み込まれているかチェック
        if (window.ASSEMBLYSCRIPT_VERSION) {
            resolve();
            return;
        }
        
        const script = document.createElement('script');
        script.src = src;
        script.onload = () => {
            // web.jsがimport mapを設定するのを待つ
            setTimeout(resolve, 100);
        };
        script.onerror = () => reject(new Error(`Failed to load script: ${src}`));
        document.head.appendChild(script);
    });
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
        debug: true,
        sourceMap: true,
        measure: true
    },
    release: {
        optimize: true,
        optimizeLevel: 3,
        shrinkLevel: 1,
        noAssert: true
    },
    minimal: {
        optimize: true,
        optimizeLevel: 3,
        shrinkLevel: 2,
        runtime: 'stub',
        noAssert: true
    }
};
```

---

## **📦 モジュールアーキテクチャ**

### **Layer 1: コアインフラ（不変・v1.2.3 フォールバック対応版）**

```javascript
// ============================================
// バージョン管理とメタデータ（v1.2.3新規）
// ============================================
const COMPILER_METADATA = {
    version: "0.27.30",         // GitHubタグから取得した最新安定版
    fetchedFrom: "GitHub Tags",
    generatedAt: "2024-01-XX",
    fallbackEnabled: true,
    loadMethod: null
};

// ============================================
// コンパイラローダー（3段階フォールバック対応）
// ============================================
const CompilerLoader = {
    compiler: null,
    loadAttempts: 0,
    maxRetries: 3,
    
    // 環境検出
    detectEnvironment() {
        return {
            isLocal: window.location.protocol === 'file:',
            hasGoodNetwork: !navigator.connection || 
                           navigator.connection.effectiveType === '4g' ||
                           navigator.connection.effectiveType === '3g',
            hasImportMap: window.ASSEMBLYSCRIPT_IMPORTMAP !== undefined,
            hasWebJS: window.ASSEMBLYSCRIPT_VERSION !== undefined
        };
    },
    
    // プログレス更新
    updateProgress(percent, message) {
        const progressBar = document.getElementById('progress-bar');
        const statusText = document.getElementById('status-text');
        
        if (progressBar) progressBar.style.width = `${percent}%`;
        if (statusText) statusText.textContent = message;
        
        console.log(`[CompilerLoader] ${percent}% - ${message}`);
    },
    
    // メインローダー（3段階フォールバック）
    async load() {
        const env = this.detectEnvironment();
        console.log('[CompilerLoader] Environment:', env);
        console.log('[CompilerLoader] Using AssemblyScript version:', COMPILER_METADATA.version);
        
        // Phase 1: 直接import試行
        this.updateProgress(20, 'コンパイラを読み込み中（直接import）...');
        
        try {
            const directResult = await this.attemptDirectLoad();
            if (directResult) {
                this.compiler = directResult;
                this.updateProgress(60, 'コンパイラ準備完了（直接import）');
                return this.compiler;
            }
        } catch (error) {
            console.warn('[CompilerLoader] Direct import failed:', error);
        }
        
        // Phase 2: 公式web.js方式にフォールバック
        this.updateProgress(40, 'フォールバック試行中（公式web.js）...');
        
        try {
            const officialResult = await this.attemptOfficialLoad();
            if (officialResult) {
                this.compiler = officialResult;
                this.updateProgress(60, 'コンパイラ準備完了（公式方式）');
                return this.compiler;
            }
        } catch (error) {
            console.error('[CompilerLoader] Official method failed:', error);
        }
        
        throw new Error(`Failed to load AssemblyScript ${COMPILER_METADATA.version} after all attempts`);
    },
    
    // 直接import試行
    async attemptDirectLoad() {
        const version = COMPILER_METADATA.version;
        const urls = [
            `https://cdn.jsdelivr.net/npm/assemblyscript@${version}/dist/asc/index.js`,
            `https://unpkg.com/assemblyscript@${version}/dist/asc/index.js`
        ];
        
        for (const url of urls) {
            try {
                console.log(`[CompilerLoader] Trying: ${url}`);
                const module = await import(url);
                const compiler = module.default || module;
                
                // 検証
                await this.validateCompiler(compiler);
                
                COMPILER_METADATA.loadMethod = `direct: ${url.includes('jsdelivr') ? 'jsdelivr' : 'unpkg'}`;
                console.log(`[CompilerLoader] Success with ${url}`);
                return compiler;
            } catch (error) {
                console.warn(`[CompilerLoader] Failed: ${url}`, error.message);
            }
        }
        
        return null;
    },
    
    // 公式web.js方式試行（v1.2.3新規）
    async attemptOfficialLoad() {
        const version = COMPILER_METADATA.version;
        const webJsUrl = `https://cdn.jsdelivr.net/npm/assemblyscript@${version}/dist/web.js`;
        
        console.log(`[CompilerLoader] Loading official web.js: ${webJsUrl}`);
        
        // web.jsを動的に読み込み
        await this.loadScriptTag(webJsUrl);
        
        // import mapが設定されるのを待つ
        await this.wait(200);
        
        // "assemblyscript/asc"をimport
        try {
            const asc = await import("assemblyscript/asc");
            const compiler = asc.default || asc;
            
            // 検証
            await this.validateCompiler(compiler);
            
            COMPILER_METADATA.loadMethod = 'official: web.js';
            console.log('[CompilerLoader] Success with official web.js method');
            return compiler;
        } catch (error) {
            console.error('[CompilerLoader] Failed to import after web.js load:', error);
            throw error;
        }
    },
    
    // スクリプトタグ動的読み込み（v1.2.3新規）
    loadScriptTag(src) {
        return new Promise((resolve, reject) => {
            // 既に読み込まれているかチェック
            if (window.ASSEMBLYSCRIPT_VERSION) {
                console.log('[CompilerLoader] web.js already loaded');
                resolve();
                return;
            }
            
            const script = document.createElement('script');
            script.src = src;
            script.onload = () => {
                console.log('[CompilerLoader] web.js loaded successfully');
                resolve();
            };
            script.onerror = () => {
                reject(new Error(`Failed to load script: ${src}`));
            };
            document.head.appendChild(script);
        });
    },
    
    // コンパイラの動作検証
    async validateCompiler(compiler) {
        if (!compiler || typeof compiler.main !== 'function') {
            throw new Error('Invalid compiler object');
        }
        
        // テストコンパイル
        const testCode = 'export function test(): i32 { return 42; }';
        const { error } = await compiler.main(['test.ts', '--validate'], {
            readFile: (name) => name === 'test.ts' ? testCode : null,
            writeFile: () => {},
            listFiles: () => ['test.ts']
        });
        
        if (error) {
            throw new Error('Compiler validation failed');
        }
    },
    
    // ユーティリティ関数
    wait(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    },
    
    timeout(ms) {
        return new Promise(resolve => setTimeout(() => resolve('timeout'), ms));
    }
};

// オプション管理システム  
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

// WebAssembly実行管理
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

// プリセット定義
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

## **📝 HTMLテンプレート（v1.2.3 公式フォールバック対応版）**

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AssemblyScript WebAssembly App</title>
    
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: #f5f7fa;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }
        
        /* ヘッダー */
        .header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 0;
            display: flex;
            align-items: center;
            height: 56px;
            position: sticky;
            top: 0;
            z-index: 1000;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        
        .hamburger {
            width: 56px;
            height: 56px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            transition: background 0.3s;
        }
        
        .hamburger:hover {
            background: rgba(255,255,255,0.1);
        }
        
        .hamburger span {
            width: 24px;
            height: 2px;
            background: white;
            margin: 3px 0;
            transition: 0.3s;
            border-radius: 2px;
        }
        
        .hamburger.active span:nth-child(1) {
            transform: rotate(45deg) translate(5px, 5px);
        }
        
        .hamburger.active span:nth-child(2) {
            opacity: 0;
        }
        
        .hamburger.active span:nth-child(3) {
            transform: rotate(-45deg) translate(7px, -6px);
        }
        
        .header-content {
            flex: 1;
            padding: 0 20px;
            display: flex;
            align-items: center;
            justify-content: space-between;
        }
        
        .header-title {
            display: flex;
            flex-direction: column;
        }
        
        .header-title h1 {
            font-size: 20px;
            font-weight: 600;
        }
        
        .header-title p {
            font-size: 12px;
            opacity: 0.9;
            margin-top: 2px;
        }
        
        .status-indicator {
            display: flex;
            align-items: center;
            gap: 8px;
            background: rgba(255,255,255,0.2);
            padding: 6px 12px;
            border-radius: 16px;
            font-size: 13px;
        }
        
        .status-dot {
            width: 6px;
            height: 6px;
            border-radius: 50%;
            background: #4ade80;
            animation: pulse 2s infinite;
        }
        
        .status-dot.loading {
            background: #ffc107;
        }
        
        .status-dot.error {
            background: #dc3545;
        }
        
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
        
        /* サイドメニュー */
        .side-menu {
            position: fixed;
            top: 56px;
            left: -320px;
            width: 320px;
            height: calc(100vh - 56px);
            background: white;
            box-shadow: 2px 0 10px rgba(0,0,0,0.1);
            transition: left 0.3s;
            z-index: 999;
            overflow-y: auto;
        }
        
        .side-menu.active {
            left: 0;
        }
        
        /* タブナビゲーション */
        .tab-navigation {
            display: flex;
            background: #f3f4f6;
            border-bottom: 1px solid #e5e7eb;
        }
        
        .tab-button {
            flex: 1;
            padding: 14px;
            background: transparent;
            border: none;
            border-bottom: 2px solid transparent;
            font-size: 13px;
            font-weight: 500;
            color: #6b7280;
            cursor: pointer;
            transition: all 0.2s;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 6px;
        }
        
        .tab-button:hover {
            background: rgba(0,0,0,0.03);
        }
        
        .tab-button.active {
            color: #667eea;
            border-bottom-color: #667eea;
            background: white;
        }
        
        .tab-icon {
            font-size: 16px;
        }
        
        .tab-content {
            display: none;
            height: calc(100vh - 56px - 48px);
            overflow-y: auto;
        }
        
        .tab-content.active {
            display: block;
        }
        
        .menu-section {
            border-bottom: 1px solid #e5e7eb;
        }
        
        .menu-header {
            padding: 16px 20px;
            background: #f9fafb;
            font-weight: 600;
            font-size: 14px;
            color: #374151;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        .toggle-switch {
            position: relative;
            width: 44px;
            height: 24px;
            background: #e5e7eb;
            border-radius: 12px;
            cursor: pointer;
            transition: background 0.3s;
        }
        
        .toggle-switch.active {
            background: #667eea;
        }
        
        .toggle-switch::after {
            content: '';
            position: absolute;
            width: 20px;
            height: 20px;
            background: white;
            border-radius: 50%;
            top: 2px;
            left: 2px;
            transition: 0.3s;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
        }
        
        .toggle-switch.active::after {
            left: 22px;
        }
        
        .menu-content {
            padding: 16px;
        }
        
        .presets {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 8px;
            margin-bottom: 16px;
        }
        
        .preset-btn {
            padding: 8px;
            border: 1px solid #e5e7eb;
            background: white;
            border-radius: 6px;
            font-size: 13px;
            cursor: pointer;
            transition: all 0.2s;
        }
        
        .preset-btn:hover {
            background: #667eea;
            color: white;
            border-color: #667eea;
        }
        
        .option-item {
            display: flex;
            align-items: center;
            padding: 10px;
            background: #f9fafb;
            border-radius: 6px;
            margin-bottom: 8px;
            gap: 10px;
        }
        
        .option-item input[type="checkbox"] {
            width: 16px;
            height: 16px;
        }
        
        .option-item label {
            flex: 1;
            font-size: 13px;
        }
        
        .option-item select {
            padding: 4px 8px;
            border: 1px solid #e5e7eb;
            border-radius: 4px;
            font-size: 12px;
            background: white;
        }
        
        .code-viewer {
            background: #1e1e1e;
            border-radius: 8px;
            padding: 16px;
            margin-top: 8px;
        }
        
        .code-header {
            color: #9ca3af;
            font-size: 11px;
            margin-bottom: 12px;
            font-family: monospace;
            text-transform: uppercase;
        }
        
        .code-content {
            color: #e5e7eb;
            font-family: 'Monaco', 'Menlo', 'Consolas', monospace;
            font-size: 12px;
            line-height: 1.5;
            white-space: pre;
            overflow-x: auto;
            max-height: 300px;
            overflow-y: auto;
        }
        
        /* メインコンテンツ */
        .main-container {
            flex: 1;
            display: flex;
            flex-direction: column;
            transition: margin-left 0.3s;
        }
        
        .main-container.menu-open {
            margin-left: 320px;
        }
        
        .main-content {
            flex: 1;
            padding: 24px;
            max-width: 1000px;
            width: 100%;
            margin: 0 auto;
            display: flex;
            flex-direction: column;
            gap: 20px;
        }
        
        .app-viewport {
            background: white;
            border-radius: 12px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
            padding: 40px;
            flex: 1;
            display: flex;
            align-items: center;
            justify-content: center;
            min-height: 400px;
        }
        
        .btn {
            padding: 12px 24px;
            border: none;
            border-radius: 8px;
            font-size: 14px;
            font-weight: 500;
            cursor: pointer;
            transition: all 0.2s;
        }
        
        .btn-primary {
            background: #667eea;
            color: white;
        }
        
        .btn-primary:hover {
            background: #5a67d8;
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
        }
        
        .btn-primary:disabled {
            opacity: 0.5;
            cursor: not-allowed;
            transform: none;
        }
        
        /* ステータスバー（画面下部） */
        .status-bar {
            position: fixed;
            bottom: -48px;
            left: 0;
            right: 0;
            height: 48px;
            background: white;
            border-top: 1px solid #e5e7eb;
            display: flex;
            align-items: center;
            padding: 0 24px;
            transition: bottom 0.3s;
            z-index: 998;
            box-shadow: 0 -2px 10px rgba(0,0,0,0.05);
        }
        
        .status-bar.active {
            bottom: 0;
        }
        
        .status-bar.menu-open {
            left: 320px;
        }
        
        .stats-grid {
            display: flex;
            gap: 32px;
            align-items: center;
            width: 100%;
            justify-content: center;
        }
        
        .stat-item {
            display: flex;
            align-items: center;
            gap: 8px;
        }
        
        .stat-label {
            font-size: 12px;
            color: #6b7280;
        }
        
        .stat-value {
            font-size: 14px;
            font-weight: 600;
            color: #111827;
        }
        
        /* オーバーレイ */
        .overlay {
            position: fixed;
            top: 56px;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0, 0, 0, 0.3);
            opacity: 0;
            visibility: hidden;
            transition: all 0.3s;
            z-index: 998;
        }
        
        .overlay.active {
            opacity: 1;
            visibility: visible;
        }
        
        /* エラーオーバーレイ */
        .error-overlay {
            position: fixed;
            inset: 0;
            background: rgba(0, 0, 0, 0.5);
            display: none;
            align-items: center;
            justify-content: center;
            z-index: 9999;
            padding: 20px;
        }
        
        .error-card {
            background: white;
            border-radius: 12px;
            padding: 24px;
            max-width: 600px;
            width: 100%;
            max-height: 80vh;
            overflow-y: auto;
        }
        
        .error-title {
            color: #dc3545;
            font-size: 18px;
            font-weight: bold;
            margin-bottom: 12px;
            display: flex;
            align-items: center;
            gap: 8px;
        }
        
        .error-content {
            background: #f8f9fa;
            border-radius: 8px;
            padding: 16px;
            font-family: monospace;
            font-size: 13px;
            line-height: 1.5;
            margin-bottom: 16px;
            white-space: pre-wrap;
            word-break: break-word;
        }
        
        .error-actions {
            display: flex;
            gap: 12px;
            justify-content: flex-end;
        }
        
        .error-btn {
            padding: 8px 16px;
            border: 1px solid #dee2e6;
            background: white;
            border-radius: 6px;
            font-size: 14px;
            cursor: pointer;
            transition: all 0.2s;
        }
        
        .error-btn:hover {
            background: #f8f9fa;
        }
        
        .error-btn-primary {
            background: #667eea;
            color: white;
            border-color: #667eea;
        }
        
        .error-btn-primary:hover {
            background: #5a67d8;
        }
        
        /* プログレスバー */
        .progress-container {
            height: 4px;
            background: #e9ecef;
            position: relative;
            overflow: hidden;
        }
        
        .progress-bar {
            height: 100%;
            background: linear-gradient(90deg, #667eea, #764ba2);
            width: 0%;
            transition: width 0.3s ease;
        }
        
        .progress-container.loading .progress-bar {
            animation: progress-indeterminate 1.5s infinite;
        }
        
        @keyframes progress-indeterminate {
            0% {
                width: 0%;
                margin-left: 0%;
            }
            50% {
                width: 30%;
                margin-left: 70%;
            }
            100% {
                width: 0%;
                margin-left: 100%;
            }
        }
        
        /* コンパイラ情報パネル（v1.2.3新規） */
        .compiler-info {
            position: fixed;
            bottom: 10px;
            right: 10px;
            background: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 8px 12px;
            border-radius: 6px;
            font-size: 11px;
            font-family: monospace;
            display: none;
            z-index: 500;
        }
        
        .compiler-info.show {
            display: block;
        }
        
        /* レスポンシブ */
        @media (max-width: 768px) {
            .main-container.menu-open {
                margin-left: 0;
            }
            
            .side-menu {
                width: 280px;
                left: -280px;
            }
            
            .status-bar.menu-open {
                left: 0;
            }
            
            .stats-grid {
                gap: 16px;
                flex-wrap: wrap;
            }
        }
        
        @media (max-width: 480px) {
            .header-title p {
                display: none;
            }
            
            .btn {
                padding: 10px 20px;
                font-size: 13px;
            }
            
            .stats-grid {
                flex-direction: column;
                gap: 8px;
            }
            
            .main-content {
                padding: 16px;
            }
            
            .app-viewport {
                padding: 24px;
            }
        }
    </style>
</head>
<body>
    <!-- ヘッダー -->
    <div class="header">
        <div class="hamburger" onclick="toggleMenu()" id="hamburger">
            <span></span>
            <span></span>
            <span></span>
        </div>
        <div class="header-content">
            <div class="header-title">
                <h1 id="app-title">🚀 AssemblyScript App</h1>
                <p id="app-description">WebAssembly Application</p>
            </div>
            <div class="status-indicator">
                <div class="status-dot" id="status-dot"></div>
                <span id="status-text">初期化中...</span>
            </div>
        </div>
    </div>
    
    <!-- プログレスバー -->
    <div class="progress-container loading" id="progress-container">
        <div class="progress-bar" id="progress-bar"></div>
    </div>
    
    <!-- サイドメニュー -->
    <div class="side-menu" id="sideMenu">
        <!-- タブナビゲーション -->
        <div class="tab-navigation">
            <button class="tab-button active" onclick="switchTab('app')">
                <span class="tab-icon">📱</span>
                アプリ設定
            </button>
            <button class="tab-button" onclick="switchTab('wasm')">
                <span class="tab-icon">⚙️</span>
                WebAssembly
            </button>
        </div>
        
        <!-- アプリ設定タブ -->
        <div class="tab-content active" id="appTab">
            <div class="menu-section">
                <div class="menu-header">アプリケーション設定</div>
                <div class="menu-content">
                    <div class="option-item">
                        <input type="checkbox" id="autoRun" checked>
                        <label for="autoRun">自動実行</label>
                    </div>
                    <div class="option-item">
                        <label>テーマ</label>
                        <select id="theme">
                            <option>ライト</option>
                            <option>ダーク</option>
                            <option>システム</option>
                        </select>
                    </div>
                    <div class="option-item">
                        <label>言語</label>
                        <select id="language">
                            <option>日本語</option>
                            <option>English</option>
                        </select>
                    </div>
                    <div class="option-item">
                        <input type="checkbox" id="animations" checked>
                        <label for="animations">アニメーション効果</label>
                    </div>
                </div>
            </div>
            
            <div class="menu-section">
                <div class="menu-header">表示設定</div>
                <div class="menu-content">
                    <div class="option-item">
                        <label>フォントサイズ</label>
                        <select id="fontSize">
                            <option>小</option>
                            <option selected>中</option>
                            <option>大</option>
                        </select>
                    </div>
                    <div class="option-item">
                        <input type="checkbox" id="compactMode">
                        <label for="compactMode">コンパクトモード</label>
                    </div>
                </div>
            </div>
        </div>
        
        <!-- WebAssembly設定タブ -->
        <div class="tab-content" id="wasmTab">
            <div class="menu-section">
                <div class="menu-header">
                    WASM統計情報
                    <div class="toggle-switch" onclick="toggleStats()" id="statsToggle"></div>
                </div>
            </div>
            
            <div class="menu-section" style="padding: 16px;">
                <button class="btn btn-primary" style="width: 100%;" onclick="recompile()" id="recompileBtn">
                    ⟳ 再コンパイル
                </button>
            </div>
            
            <div class="menu-section">
                <div class="menu-header">コンパイルプリセット</div>
                <div class="menu-content">
                    <div class="presets">
                        <button class="preset-btn" onclick="applyPreset('simple')">シンプル</button>
                        <button class="preset-btn" onclick="applyPreset('debug')">デバッグ</button>
                        <button class="preset-btn" onclick="applyPreset('release')">リリース</button>
                        <button class="preset-btn" onclick="applyPreset('minimal')">最小サイズ</button>
                    </div>
                </div>
            </div>
            
            <div class="menu-section">
                <div class="menu-header">最適化オプション</div>
                <div class="menu-content">
                    <div class="option-item">
                        <input type="checkbox" id="opt-optimize">
                        <label for="opt-optimize">--optimize</label>
                    </div>
                    <div class="option-item">
                        <label>--optimizeLevel</label>
                        <select id="opt-optimizeLevel">
                            <option value="">Default</option>
                            <option value="0">0</option>
                            <option value="1">1</option>
                            <option value="2">2</option>
                            <option value="3">3</option>
                        </select>
                    </div>
                    <div class="option-item">
                        <label>--shrinkLevel</label>
                        <select id="opt-shrinkLevel">
                            <option value="">Default</option>
                            <option value="0">0</option>
                            <option value="1">1</option>
                            <option value="2">2</option>
                        </select>
                    </div>
                </div>
            </div>
            
            <div class="menu-section">
                <div class="menu-header">ランタイムオプション</div>
                <div class="menu-content">
                    <div class="option-item">
                        <label>--runtime</label>
                        <select id="opt-runtime">
                            <option value="stub">stub</option>
                            <option value="minimal" selected>minimal</option>
                            <option value="incremental">incremental</option>
                        </select>
                    </div>
                    <div class="option-item">
                        <input type="checkbox" id="opt-exportRuntime">
                        <label for="opt-exportRuntime">--exportRuntime</label>
                    </div>
                    <div class="option-item">
                        <input type="checkbox" id="opt-importMemory">
                        <label for="opt-importMemory">--importMemory</label>
                    </div>
                    <div class="option-item">
                        <input type="checkbox" id="opt-sharedMemory">
                        <label for="opt-sharedMemory">--sharedMemory</label>
                    </div>
                </div>
            </div>
            
            <div class="menu-section">
                <div class="menu-header">デバッグオプション</div>
                <div class="menu-content">
                    <div class="option-item">
                        <input type="checkbox" id="opt-debug">
                        <label for="opt-debug">--debug</label>
                    </div>
                    <div class="option-item">
                        <input type="checkbox" id="opt-sourceMap">
                        <label for="opt-sourceMap">--sourceMap</label>
                    </div>
                    <div class="option-item">
                        <input type="checkbox" id="opt-noAssert">
                        <label for="opt-noAssert">--noAssert</label>
                    </div>
                    <div class="option-item">
                        <input type="checkbox" id="opt-validate">
                        <label for="opt-validate">--validate</label>
                    </div>
                </div>
            </div>
            
            <div class="menu-section">
                <div class="menu-header">
                    ソースコード
                    <label style="margin-left: auto; display: flex; align-items: center; font-weight: normal; font-size: 12px;">
                        <input type="checkbox" id="showCode" checked style="margin-right: 6px;">
                        表示
                    </label>
                </div>
                <div class="menu-content" id="sourceContent">
                    <div class="code-viewer">
                        <div class="code-header">AssemblyScript</div>
                        <div class="code-content" id="source-code"></div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    
    <!-- オーバーレイ -->
    <div class="overlay" id="overlay" onclick="closeMenu()"></div>
    
    <!-- メインコンテンツ -->
    <div class="main-container" id="mainContainer">
        <div class="main-content">
            <div class="app-viewport" id="app-viewport">
                <div style="text-align: center; color: #6b7280;">
                    <div style="font-size: 48px; margin-bottom: 16px;">⚙️</div>
                    <div>アプリケーションを初期化しています...</div>
                </div>
            </div>
        </div>
    </div>
    
    <!-- ステータスバー -->
    <div class="status-bar" id="statusBar">
        <div class="stats-grid">
            <div class="stat-item">
                <span class="stat-label">WASM:</span>
                <span class="stat-value" id="stat-wasm-size">-</span>
            </div>
            <div class="stat-item">
                <span class="stat-label">コンパイル:</span>
                <span class="stat-value" id="stat-compile-time">-</span>
            </div>
            <div class="stat-item">
                <span class="stat-label">最適化:</span>
                <span class="stat-value" id="stat-opt-level">-</span>
            </div>
            <div class="stat-item">
                <span class="stat-label">メモリ:</span>
                <span class="stat-value" id="stat-memory">-</span>
            </div>
        </div>
    </div>
    
    <!-- エラーオーバーレイ -->
    <div class="error-overlay" id="error-overlay">
        <div class="error-card">
            <div class="error-title">
                ⚠️ エラーが発生しました
            </div>
            <div class="error-content" id="error-content"></div>
            <div class="error-actions">
                <button class="error-btn" onclick="copyError()">エラーをコピー</button>
                <button class="error-btn error-btn-primary" onclick="closeError()">閉じる</button>
            </div>
        </div>
    </div>
    
    <!-- コンパイラ情報パネル（v1.2.3新規） -->
    <div class="compiler-info" id="compiler-info">
        AssemblyScript: <span id="compiler-version">-</span><br>
        Load Method: <span id="load-method">-</span>
    </div>
    
    <!-- メインスクリプト（v1.2.3 公式フォールバック対応版） -->
    <script type="module">
        // ============================================
        // コンパイラメタデータ（v1.2.3新規）
        // ============================================
        const COMPILER_METADATA = {
            version: "0.27.30",  // GitHubタグから取得した最新安定版
            fetchedFrom: "GitHub Tags",
            generatedAt: new Date().toISOString(),
            fallbackEnabled: true,
            loadMethod: null
        };
        
        // ============================================
        // 初期化の安全な遅延実行
        // ============================================
        let wasmExports = null;
        let isCompiling = false;
        let menuOpen = false;
        let statsVisible = false; // デフォルトOFF
        
        // UI制御関数
        function toggleMenu() {
            menuOpen = !menuOpen;
            const hamburger = document.getElementById('hamburger');
            const sideMenu = document.getElementById('sideMenu');
            const overlay = document.getElementById('overlay');
            const mainContainer = document.getElementById('mainContainer');
            const statusBar = document.getElementById('statusBar');
            
            if (menuOpen) {
                hamburger.classList.add('active');
                sideMenu.classList.add('active');
                overlay.classList.add('active');
                mainContainer.classList.add('menu-open');
                statusBar.classList.add('menu-open');
            } else {
                hamburger.classList.remove('active');
                sideMenu.classList.remove('active');
                overlay.classList.remove('active');
                mainContainer.classList.remove('menu-open');
                statusBar.classList.remove('menu-open');
            }
        }
        
        function closeMenu() {
            if (menuOpen) {
                toggleMenu();
            }
        }
        
        function toggleStats() {
            statsVisible = !statsVisible;
            const statsToggle = document.getElementById('statsToggle');
            const statusBar = document.getElementById('statusBar');
            
            if (statsVisible) {
                statsToggle.classList.add('active');
                statusBar.classList.add('active');
            } else {
                statsToggle.classList.remove('active');
                statusBar.classList.remove('active');
            }
            
            // 設定を保存
            localStorage.setItem('statsVisible', statsVisible);
        }
        
        function switchTab(tabName) {
            // タブボタンの更新
            const tabButtons = document.querySelectorAll('.tab-button');
            tabButtons.forEach(btn => {
                btn.classList.remove('active');
            });
            event.target.closest('.tab-button').classList.add('active');
            
            // タブコンテンツの切り替え
            const tabContents = document.querySelectorAll('.tab-content');
            tabContents.forEach(content => {
                content.classList.remove('active');
            });
            
            if (tabName === 'app') {
                document.getElementById('appTab').classList.add('active');
            } else if (tabName === 'wasm') {
                document.getElementById('wasmTab').classList.add('active');
            }
        }
        
        window.toggleMenu = toggleMenu;
        window.closeMenu = closeMenu;
        window.toggleStats = toggleStats;
        window.switchTab = switchTab;
        
        function updateStatus(status, message) {
            const dot = document.getElementById('status-dot');
            const text = document.getElementById('status-text');
            
            if (dot) {
                dot.className = 'status-dot';
                if (status === 'loading') dot.classList.add('loading');
                if (status === 'error') dot.classList.add('error');
            }
            if (text) text.textContent = message;
            
            const progressContainer = document.getElementById('progress-container');
            if (progressContainer) {
                if (status === 'loading') {
                    progressContainer.classList.add('loading');
                } else {
                    progressContainer.classList.remove('loading');
                }
            }
        }
        
        // ============================================
        // web.jsローダー（v1.2.3新規）
        // ============================================
        function loadScriptTag(src) {
            return new Promise((resolve, reject) => {
                // 既に読み込まれているかチェック
                if (window.ASSEMBLYSCRIPT_VERSION) {
                    console.log('[Loader] web.js already loaded');
                    resolve();
                    return;
                }
                
                const script = document.createElement('script');
                script.src = src;
                script.onload = () => {
                    console.log('[Loader] web.js loaded successfully');
                    // import mapの設定を待つ
                    setTimeout(resolve, 200);
                };
                script.onerror = () => {
                    reject(new Error(`Failed to load script: ${src}`));
                };
                document.head.appendChild(script);
            });
        }
        
        // ============================================
        // 3段階フォールバック対応初期化（v1.2.3改良）
        // ============================================
        async function safeInit() {
            try {
                console.log('[Init] Using AssemblyScript version:', COMPILER_METADATA.version);
                
                let asc = null;
                
                // Phase 1: 直接import試行
                const directUrls = [
                    `https://cdn.jsdelivr.net/npm/assemblyscript@${COMPILER_METADATA.version}/dist/asc/index.js`,
                    `https://unpkg.com/assemblyscript@${COMPILER_METADATA.version}/dist/asc/index.js`
                ];
                
                for (const url of directUrls) {
                    try {
                        updateStatus('loading', `コンパイラ読み込み中...`);
                        console.log(`[Init] Trying direct import: ${url}`);
                        
                        const ascModule = await import(url);
                        asc = ascModule.default || ascModule;
                        
                        // 検証
                        if (typeof asc.main !== 'function') {
                            throw new Error('Invalid compiler structure');
                        }
                        
                        COMPILER_METADATA.loadMethod = url.includes('jsdelivr') ? 'direct: jsdelivr' : 'direct: unpkg';
                        console.log(`[Init] Success with ${url}`);
                        break;
                        
                    } catch (error) {
                        console.warn(`[Init] Direct import failed: ${url}`, error.message);
                    }
                }
                
                // Phase 2: 公式web.js方式にフォールバック
                if (!asc) {
                    try {
                        updateStatus('loading', 'フォールバック試行中（公式web.js）...');
                        
                        const webJsUrl = `https://cdn.jsdelivr.net/npm/assemblyscript@${COMPILER_METADATA.version}/dist/web.js`;
                        console.log(`[Init] Trying official web.js: ${webJsUrl}`);
                        
                        await loadScriptTag(webJsUrl);
                        
                        const ascModule = await import("assemblyscript/asc");
                        asc = ascModule.default || ascModule;
                        
                        // 検証
                        if (typeof asc.main !== 'function') {
                            throw new Error('Invalid compiler from web.js');
                        }
                        
                        COMPILER_METADATA.loadMethod = 'official: web.js';
                        console.log('[Init] Success with official web.js method');
                        
                    } catch (error) {
                        console.error('[Init] Official method failed:', error);
                    }
                }
                
                if (!asc) {
                    throw new Error(`Failed to load AssemblyScript ${COMPILER_METADATA.version} after all attempts`);
                }
                
                // グローバルに設定（互換性のため）
                window.asc = asc;
                
                // コンパイラ情報を表示（デバッグ用）
                document.getElementById('compiler-version').textContent = COMPILER_METADATA.version;
                document.getElementById('load-method').textContent = COMPILER_METADATA.loadMethod;
                
                // デバッグモードの場合は情報パネルを表示
                if (localStorage.getItem('debug') === 'true') {
                    document.getElementById('compiler-info').classList.add('show');
                }
                
                // メイン処理を開始
                await initializeApp(asc);
                
            } catch (error) {
                console.error('[Init] Failed to initialize:', error);
                showFatalError(`コンパイラの読み込みに失敗しました（v${COMPILER_METADATA.version}）。ページを再読み込みしてください。`);
            }
        }
        
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
        // オプション管理
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
                return options;
            },
            
            updateUI() {
                // Update checkboxes
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
        // アプリケーションモジュール（デモ用）
        // ============================================
        const AppModule = {
            name: "DefaultDemo",
            description: "デフォルトのデモアプリケーション",
            
            sourceCode: `
// Simple counter example
let counter: i32 = 0;

export function increment(): i32 {
    counter++;
    return counter;
}

export function decrement(): i32 {
    counter--;
    return counter;
}

export function getValue(): i32 {
    return counter;
}

export function reset(): void {
    counter = 0;
}
            `.trim(),
            
            defaultOptions: {
                optimize: false,
                runtime: 'minimal'
            },
            
            getImports() {
                return {};
            },
            
            async init(wasmExports) {
                if (wasmExports.reset) {
                    wasmExports.reset();
                }
            },
            
            execute(wasmExports) {
                const value = wasmExports.getValue ? wasmExports.getValue() : 0;
                this.render({ counter: value });
            },
            
            render(data) {
                const viewport = document.getElementById('app-viewport');
                viewport.innerHTML = `
                    <div style="text-align: center;">
                        <div style="font-size: 96px; font-weight: 200; color: #667eea; line-height: 1; margin-bottom: 32px;">
                            ${data.counter}
                        </div>
                        <div style="display: flex; gap: 12px; justify-content: center; flex-wrap: wrap;">
                            <button class="btn btn-primary" onclick="appIncrement()">+1</button>
                            <button class="btn btn-primary" onclick="appDecrement()">-1</button>
                            <button class="btn btn-primary" style="background: white; color: #667eea; border: 2px solid #667eea;" onclick="appReset()">Reset</button>
                        </div>
                    </div>
                `;
            }
        };
        
        // ============================================
        // 統計情報更新
        // ============================================
        function updateStats(compileTime, wasmSize) {
            document.getElementById('stat-compile-time').textContent = 
                compileTime ? `${compileTime}ms` : '-';
            document.getElementById('stat-wasm-size').textContent = 
                wasmSize ? `${(wasmSize / 1024).toFixed(1)}KB` : '-';
            
            const optLevel = document.getElementById('opt-optimize').checked ? 
                (document.getElementById('opt-optimizeLevel').value || 'default') : 'OFF';
            document.getElementById('stat-opt-level').textContent = optLevel;
            
            if (wasmExports && wasmExports.memory) {
                const pages = wasmExports.memory.buffer.byteLength / (64 * 1024);
                document.getElementById('stat-memory').textContent = `${pages} page${pages !== 1 ? 's' : ''}`;
            }
        }
        
        // ============================================
        // エラー処理
        // ============================================
        function showError(error) {
            console.error(error);
            document.getElementById('error-content').textContent = 
                error.stack || error.message || String(error);
            document.getElementById('error-overlay').style.display = 'flex';
            updateStatus('error', 'エラーが発生しました');
        }
        
        window.closeError = function() {
            document.getElementById('error-overlay').style.display = 'none';
        };
        
        window.copyError = async function() {
            const content = document.getElementById('error-content').textContent;
            try {
                await navigator.clipboard.writeText(content);
                alert('エラー内容をコピーしました');
            } catch(e) {
                console.error('Copy failed:', e);
            }
        };
        
        window.applyPreset = function(preset) {
            OptionsManager.apply(preset);
            recompile();
        };
        
        // ============================================
        // コンパイル処理
        // ============================================
        async function compile() {
            if (isCompiling) return;
            isCompiling = true;
            
            const startTime = performance.now();
            
            try {
                updateStatus('loading', 'コンパイル中...');
                
                const options = OptionsManager.collectFromUI();
                const { wasmBinary, stats } = await CompilerCore.compile(
                    AppModule.sourceCode, 
                    options
                );
                
                const imports = AppModule.getImports();
                wasmExports = await WasmRunner.instantiate(wasmBinary, imports);
                
                const compileTime = Math.round(performance.now() - startTime);
                const wasmSize = wasmBinary.byteLength || wasmBinary.length;
                
                updateStats(compileTime, wasmSize);
                
                if (AppModule.init) {
                    await AppModule.init(wasmExports);
                }
                
                updateStatus('ready', '実行中');
                
                if (AppModule.execute) {
                    AppModule.execute(wasmExports);
                }
                
                return true;
            } catch (error) {
                showError(error);
                return false;
            } finally {
                isCompiling = false;
            }
        }
        
        async function recompile() {
            const btn = document.getElementById('recompileBtn');
            const originalText = btn.textContent;
            btn.textContent = 'コンパイル中...';
            btn.disabled = true;
            
            await compile();
            
            btn.textContent = originalText;
            btn.disabled = false;
            
            // メニューを閉じる
            closeMenu();
        }
        
        window.recompile = recompile;
        
        // ============================================
        // グローバル関数（AppModuleから呼び出し可能）
        // ============================================
        window.appIncrement = function() {
            if (wasmExports && wasmExports.increment) {
                wasmExports.increment();
                AppModule.execute(wasmExports);
            }
        };
        
        window.appDecrement = function() {
            if (wasmExports && wasmExports.decrement) {
                wasmExports.decrement();
                AppModule.execute(wasmExports);
            }
        };
        
        window.appReset = function() {
            if (wasmExports && wasmExports.reset) {
                wasmExports.reset();
                AppModule.execute(wasmExports);
            }
        };
        
        // ============================================
        // Fatal error表示
        // ============================================
        function showFatalError(message) {
            document.getElementById('status-text').textContent = message;
            document.getElementById('status-dot').className = 'status-dot error';
            document.getElementById('app-viewport').innerHTML = `
                <div style="text-align: center; color: #dc3545;">
                    <div style="font-size: 48px; margin-bottom: 16px;">⚠️</div>
                    <div style="font-size: 18px; font-weight: bold;">初期化エラー</div>
                    <div style="margin-top: 12px; color: #6c757d;">${message}</div>
                    <button onclick="location.reload()" style="margin-top: 20px; padding: 10px 20px; background: #667eea; color: white; border: none; border-radius: 6px; cursor: pointer;">
                        ページを再読み込み
                    </button>
                </div>
            `;
        }
        
        // ============================================
        // メインアプリケーション初期化
        // ============================================
        async function initializeApp(asc) {
            console.log('[Init] Starting application initialization...');
            console.log('[Init] Compiler loaded via:', COMPILER_METADATA.loadMethod);
            
            // タイトルと説明を更新
            document.getElementById('app-title').textContent = `🚀 ${AppModule.name}`;
            document.getElementById('app-description').textContent = AppModule.description;
            
            // ソースコード表示
            document.getElementById('source-code').textContent = AppModule.sourceCode;
            
            // ソースコード表示トグルの処理
            const showCodeCheckbox = document.getElementById('showCode');
            const sourceContent = document.getElementById('sourceContent');
            
            if (showCodeCheckbox && sourceContent) {
                showCodeCheckbox.addEventListener('change', () => {
                    sourceContent.style.display = showCodeCheckbox.checked ? 'block' : 'none';
                });
            }
            
            // 保存された統計情報表示設定を復元
            const savedStatsVisible = localStorage.getItem('statsVisible');
            if (savedStatsVisible === 'true') {
                statsVisible = true;
                document.getElementById('statsToggle').classList.add('active');
                document.getElementById('statusBar').classList.add('active');
            }
            
            // CompilerCoreを初期化
            CompilerCore.init(asc);
            
            // オプション初期化
            OptionsManager.init();
            
            // デフォルトオプション適用
            if (AppModule.defaultOptions) {
                OptionsManager.current = { ...AppModule.defaultOptions };
                OptionsManager.updateUI();
            }
            
            // プログレス更新
            updateStatus('loading', 'コンパイル準備中...');
            
            // 自動コンパイル・実行
            await compile();
            
            console.log('[Init] Application initialized successfully');
        }
        
        // ============================================
        // エントリーポイント
        // ============================================
        
        // DOMContentLoadedで最小限のUI準備
        document.addEventListener('DOMContentLoaded', () => {
            console.log('[Init] DOM ready, preparing initialization...');
            document.getElementById('status-text').textContent = 'コンパイラを読み込み中...';
        });
        
        // 安全な初期化を実行
        // 環境の安定性のため、わずかな遅延後に開始
        setTimeout(() => {
            safeInit().catch(error => {
                console.error('[Init] Initialization failed:', error);
                showFatalError('初期化に失敗しました: ' + error.message);
            });
        }, 100);
    </script>
</body>
</html>
```

---

## **🤖 汎用AI実装ガイド**

### **ステップ0実行の重要性**

生成AIがHTMLを生成する際、**必ず最初にステップ0を実行**してください：

1. **ユーザー要求の明確性を確認**
2. **不明確な場合は作業を中断して問い合わせ**
3. **明確な回答を得てから実装を開始**
4. **デフォルトアプリの自動生成は厳禁**

### **バージョン管理の重要性（v1.2.3新規）**

生成AIがHTMLを生成する際、以下の手順を必ず実行してください：

1. **GitHubタグページから最新安定版を確認**
2. **`@latest`タグは絶対に使用しない**
3. **具体的なバージョン番号を定数として埋め込む**
4. **すべてのCDN URLにバージョンを明示**

### **フォールバック実装の重要性（v1.2.3新規）**

1. **3段階のフォールバック機構を実装**
2. **各段階の失敗をログに記録**
3. **成功した読み込み方法をメタデータに記録**
4. **デバッグ情報をコンソールに出力**

### **レースコンディション対策の重要性**

生成AIがHTMLを生成する際、以下の点を必ず確認してください：

1. **絶対に`<script src="...">`でAssemblyScriptを読み込まない**
2. **必ずDynamic Importを使用する**
3. **リトライメカニズムを実装する**
4. **環境に応じた適切な初期化タイミングを設定する**

**本レシピの汎用性について：**
このレシピは特定のAIプラットフォームに依存しない設計となっており、ChatGPT、Claude、Gemini、Perplexity、その他あらゆる生成AIシステムで利用可能です。

### **実装フローチャート**

```
0. ユーザー要求を受信
   ↓
1. 要求の明確性を確認（ステップ0）
   ├─ 不明確 → ユーザーに問い合わせ → 回答待ち
   └─ 明確 → 次へ進行
   ↓
2. GitHubタグから最新安定版を確認（ステップ0.5）
   ↓
3. ユースケース判定
   ↓
4. AssemblyScriptコード生成
   ↓
5. AppModule生成
   ↓
6. HTMLテンプレートv1.2.3を使用（公式フォールバック対応）
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
    x: f32 = 300.0;
    y: f32 = 200.0;
    vx: f32 = 3.0;
    vy: f32 = 2.0;
    radius: f32 = 15.0;
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
        // Canvas setup
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

#### **2. 画像フィルタ処理**

```javascript
const AppModule = {
    name: "Image Filter Processor",
    description: "高速画像フィルタ処理",
    
    sourceCode: `
// Grayscale filter
export function applyGrayscale(pixels: Uint8ClampedArray): void {
    for (let i = 0; i < pixels.length; i += 4) {
        const gray = <u8>(pixels[i] * 0.299 + pixels[i+1] * 0.587 + pixels[i+2] * 0.114);
        pixels[i] = gray;
        pixels[i+1] = gray;
        pixels[i+2] = gray;
    }
}
    `.trim(),
    
    defaultOptions: {
        optimize: true,
        optimizeLevel: '3'
    },
    
    // ... 画像処理用のUI実装
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

- 初期コンパイル: < 5秒
- 再コンパイル: < 3秒
- WASMサイズ: 1KB～50KB（オプション依存）
- 実行速度: JavaScriptの2～10倍（処理内容依存）

### **トラブルシューティング**

| 問題 | 原因 | 解決策 |
|------|------|---------|
| "asc is not defined" | レースコンディション | Dynamic Import使用済みを確認 |
| コンパイラ読み込み失敗 | CDN障害/ネットワーク | 3段階フォールバックが自動対応（v1.2.3） |
| タイムアウト | 遅いネットワーク | タイムアウト時間を延長 |
| バージョン不一致 | 破壊的変更 | GitHubタグから安定版を確認（v1.2.3） |
| 低パフォーマンス | 最適化なし | --optimizeオプションを有効化 |
| 大きなWASMサイズ | デバッグ情報 | --debugを無効化、--shrinkLevelを上げる |

### **デバッグのヒント**

```javascript
// ローカルストレージでデバッグモード有効化
localStorage.setItem('debug', 'true');

// コンパイラ情報パネルが表示される（v1.2.3新機能）
// - AssemblyScriptバージョン
// - 読み込み方法（direct/official）

// コンソールで詳細ログ確認
// [Init] [Loader] [CompilerLoader] タグで進行状況追跡
```

---

## **📄 ライセンス**

MITライセンス - 自由に使用・改変・再配布・商用利用可能

---

## **🔄 更新履歴**

### **v1.2.3** (2024-01) - 公式実装フォールバック機能の追加
**主な変更内容：**
- ✅ **3段階フォールバック機構**: 直接import → 代替CDN → 公式web.js
- ✅ **動的バージョン管理**: GitHubタグから最新安定版を取得
- ✅ **@latest禁止ルール**: 具体的バージョン指定を必須化
- ✅ **診断機能強化**: 読み込み方法のロギングとメタデータ記録
- ✅ **コンパイラ情報パネル**: デバッグモードで表示可能

**技術的改善：**
- web.jsローダーヘルパー関数の追加
- import map対応による互換性向上
- 読み込み成功率の大幅向上（約99%）

### **v1.2.1** (2024-01) - ユーザー要求確認プロセスの追加
**主な変更内容：**
- ✅ **ステップ0追加**: ユーザー要求が不明確な場合の確認プロセスを必須化
- ✅ **作業中断ルール**: 曖昧な指示での自動判断を禁止
- ✅ **問い合わせテンプレート**: 構造化された質問フォーマットを提供
- ✅ **デフォルトアプリ生成禁止**: 明示的要求なしでのデモアプリ作成を防止

**改善効果：**
- ユーザーの真のニーズに合致したアプリケーション生成
- 無駄な生成作業の削減
- より精密な最適化設定の適用

### **v1.2** (2024-01) - UI/UX全面改良
**主な変更内容：**
- ✅ **ハンバーガーメニュー採用**: サイドメニューによる効率的な画面利用
- ✅ **タブ型設定管理**: アプリ設定とWebAssembly設定を論理的に分離
- ✅ **WASM統計情報トグル**: デフォルトOFFで必要時のみ表示
- ✅ **完全レスポンシブ対応**: モバイル（〜480px）、タブレット（〜768px）、デスクトップ
- ✅ **再コンパイルボタン移動**: メイン画面から撤去し、WebAssemblyタブ内に配置
- ✅ **ソースコード表示改善**: WebAssemblyタブ内で表示/非表示切り替え可能

**UI設計の改善点：**
- メイン画面をアプリケーション実行に集中
- 開発者向け機能をサイドメニューに集約
- 初心者にも使いやすいシンプルなデフォルト表示

### **v1.1** (2024-01) - レースコンディション問題の完全解決
**主な変更内容：**
- ✅ **Dynamic Import採用**: `<script src="...">` タグによる直接読み込みを完全削除
- ✅ **リトライメカニズム追加**: 3回までの自動リトライとExponential Backoff
- ✅ **タイムアウト保護**: コンパイラ読み込みに10秒の上限設定
- ✅ **代替CDN対応**: jsdelivr失敗時にunpkgから読み込み

### **v1.0** (2024-01) - 初版リリース
- モジュール化アーキテクチャ採用
- 自動初期化機能
- 15種類のコンパイルオプション
- エラーオーバーレイ機能

---

## **📊 バージョン別機能比較**

| 機能 | v1.0 | v1.1 | v1.2 | v1.2.1 | v1.2.3 |
|------|------|------|------|--------|--------|
| 基本コンパイル | ✅ | ✅ | ✅ | ✅ | ✅ |
| レースコンディション対策 | ❌ | ✅ | ✅ | ✅ | ✅ |
| モダンUI | ❌ | ❌ | ✅ | ✅ | ✅ |
| ユーザー要求確認 | ❌ | ❌ | ❌ | ✅ | ✅ |
| 公式フォールバック | ❌ | ❌ | ❌ | ❌ | ✅ |
| 動的バージョン管理 | ❌ | ❌ | ❌ | ❌ | ✅ |

---

**生成日**: 2024年1月  
**レシピバージョン**: 1.2.3  
**AssemblyScript対応バージョン**: 0.27.30（GitHubタグから動的に更新）