# WAT_SingleHTML_WASI_Recipe_v3 - "AI->WAT->wasm32-wasi"を**単一HTML**でコンパイル＆実行するための完全テンプレート (COI不要 / CDN ESMのみ / 最小限WASI直接実行)

> このレシピは添付の単一HTML（最新版）を**フルテキストテンプレート**として採用し、**Math-RECIPEと同等の「テンプレートパワー」**を達成するためにv3で改良されました。  
> これによりPoCフローのシームレスな再現を可能にします：**ユーザーリクエスト -> AIがWATを生成 -> 単一HTMLが.wat->.wasmにアセンブル -> ブラウザがWASIで実行**。

---

## 1. 役割分担（AIと単一HTML）

- **AI（WAT生成エージェント）**
  - ユーザーリクエスト（自然言語）を**WAT（WebAssemblyテキスト）**に変換します。
  - 生成したWATコードを以下のテンプレートHTML内の`<script type="text/plain" id="wat-source">`コンテンツに**直接挿入（置換）します**。
  - 必要に応じて文字列長（iovec.lenなど）の**バイト長の一致**を確保します。

- **単一HTML（実行環境）**
  - ページ表示時にCDNから**wabt@1.0.37（ESM `index.js`）**を自動的にロードします。
  - **Assemble**がWAT->Wasmに変換、**Run**が最小限WASI（`fd_write`）で実行します。  
  - **セルフテスト**、**SHA-256**、**WASMダウンロード**、**既存WASMの直接実行**、**（オプション）自動実行トグル**、**進行状況に応じた自動スクロール**を含みます。

---

## 2. AI実装手順（**コピー＆ペースト操作用 / 変更不可ポイントを含む**）

**ステップA：WATを生成（SYSTEMテンプレート必須）**
- ターゲットは**wasm32-wasi**。  
- **`_start`をエクスポート**（`(start ...)`は使用しない）。
- stdout専用に`fd_write`をインポート。
- **(memory (export "memory") 1)**を定義、iovec(0..7) / メッセージ(8..) / nwritten(20..23)を使用。
- **文字列長は正確なバイト数でなければならない**（改行は`\0a`、終端NULは不要）。

**ステップB：テンプレートHTMLに挿入（ここが「固定位置」）**
- `<script type="text/plain" id="wat-source"> ... </script>`の**コンテンツ全体を置換**。
- **タグの属性、ID、`type`を変更してはならない**（削除も禁止）。

**ステップC：HTMLの他の部分には触れない**
- **CDN URLは`index.js`のみ**（`/wabt.js`や`/dist/wabt.js`は**使用しない**）。  
- **ロード順の変更禁止**（jsDelivr -> UNPKG -> esm.sh）。  
- UI IDの名前、ログフォーマット（`stdout equality - PASS`など）の変更禁止。

---

## 3. **変更不可チェックリスト**（AI/人間の両方が遵守すること）

- [ ] **`<script id="wat-source">`のタグ属性は不変**。コンテンツのみを置換。  
- [ ] **wabt@1.0.37の`index.js`のみをインポート**（UMD `/wabt.js`系列は**禁止**）。  
- [ ] ロード順：**jsDelivr -> UNPKG -> esm.sh**。  
- [ ] **`_start`を必ずエクスポート**。**`(start ...)`の使用は禁止**。  
- [ ] 文字列長（`iov.len`）は**正確**でなければならない（例：`"Hello, WAT!\n"`は**12**）。  
- [ ] セルフテストのログフォーマット（`stdout equality - ...` / `syntax error detection - ...`）は**変更不可**。

---

## 4. WAT生成テンプレート（AI用SYSTEMインストラクション）

````text
SYSTEM（不変ルール / WAT生成エージェント）：

**ターゲット**
- wasm32-wasi。単一（module）を出力。コードのみ。

**エクスポート**
- (func (export "_start") ...)を必ず定義。(start ...)は使用しない。

**インポート**
- stdout専用：
  (import "wasi_snapshot_preview1" "fd_write"
    (func $fd_write (param i32 i32 i32 i32) (result i32)))

**メモリとレイアウト**
- (memory (export "memory") 1)を定義。
- iovec[0] = {{buf(0..3), len(4..7)}}。メッセージは8..、nwrittenは20..23。
- 文字列長（len）は正確なバイト数。終端NULなし。改行は"\0a"。

**品質チェック**
- すべてのi32.store/loadは4バイト境界上。
- fd_writeの第4引数は書き込みバイト数を書き戻すアドレス。
- wabt(parseWat->toBinary)で構文/型エラーなし。

**禁止事項**
- (start ...) / 実験的機能（threads、simd、ref.funcなど）禁止。
- 巨大データや無限ループの生成禁止。

**出力フォーマット**
- ```watでラップしたWATのみを出力。前後に説明文なし。

USERリクエスト：
<<ここにユーザーのリクエストを挿入>>
````

---

## 5. 完全な**単一HTMLテンプレート（全文）**
> 以下を**そのまま.htmlとして保存**し、**`<script id="wat-source">`のコンテンツのみをWATで置換**して使用します。  
> 添付HTML（最新版）ベース。いくつかのコメントに**AI_INSERTマーカー**を追加。

```html
<!doctype html>
<html lang="ja">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>WAT -> Wasm32-WASI アセンブリ＆実行（COI不要・CDNロードのみ・既存WASMロード）</title>
  <style>
    :root { color-scheme: light dark; }
    body { margin: 0; font-family: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Noto Sans", Arial; line-height: 1.5; }
    header { padding: 12px 16px; background: #0001; }
    main { display: grid; grid-template-columns: 1fr; gap: 12px; padding: 12px; }
    /* @media (min-width: 1120px) { main { grid-template-columns: 1fr 1fr; } } */
    h1 { font-size: 1.05rem; margin: 0; }
    textarea, pre, button, input { font: inherit; }
    textarea { width: 100%; min-height: 260px; padding: 8px; border-radius: 8px; border: 1px solid #0002; font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; }
    .card { border: 1px solid #0002; border-radius: 12px; padding: 12px; background: #00000008; }
    .row { display: flex; gap: 8px; flex-wrap: wrap; align-items: center; }
    .mono { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; }
    .muted { opacity: .8; }
    .ok { color: #176b37; }
    .err { color: #b22222; }
    .out { height: 220px; overflow: auto; white-space: pre-wrap; background: #0000; padding: 8px; border-radius: 8px; border: 1px dashed #0003; }
    .kv { display: grid; grid-template-columns: max-content 1fr; gap: 4px 12px; }
    .kv div:nth-child(odd){ color: #000a; }
    .btn { padding: 8px 12px; border: 1px solid #0003; border-radius: 8px; background: #fff; cursor: pointer; }
    .btn[disabled]{ opacity: .5; cursor: not-allowed; }
    .footer { font-size: .9rem; padding-top: 4px; }
    .badge { padding: 2px 8px; border-radius: 999px; background: #9992; font-size: .85rem; }
    .tip { background: #ffeaa7; color: #4d3b00; padding: 6px 8px; border-radius: 8px; border: 1px solid #d8c16e; }
    .warn { background: #ffdddd; color: #6b1a1a; padding: 6px 8px; border-radius: 8px; border: 1px solid #e4a6a6; }
  </style>
</head>
<body>
  <header>
    <h1>WAT -> Wasm32-WASI アセンブリ＆実行 <span class="badge">COI不要 / CDNロードのみ（公式URL） / 既存WASMロード</span></h1>
  </header>

  <main>
    <section class="card" id="sec-load">
      <h2>1) CDNからWABT（アセンブラ）をロード</h2>
      <p class="muted">wabtは<b>公式ESMエントリ（<code>index.js</code>）</b>のみを使用。UMD <code>/wabt.js</code>や<code>/dist/wabt.js</code>は使用しません。</p>
      <div class="row" style="margin-top:6px">
        <button id="load-wabt-cdn" class="btn">CDNを再ロード</button>
        <span id="wabt-cdn-status" class="muted">ロードされていません</span>
      </div>
      <details style="margin-top:6px">
        <summary>使用するURL（順番に再試行）</summary>
        <ul class="mono muted">
          <li>1) jsDelivr: https://cdn.jsdelivr.net/npm/wabt@1.0.37/index.js</li>
          <li>2) UNPKG: https://unpkg.com/wabt@1.0.37/index.js</li>
          <li>3) esm.sh: https://esm.sh/wabt@1.0.37</li>
        </ul>
      </details>
      <div id="cdn-warn" class="warn" style="margin-top:6px; display:none"></div>
    </section>

    <section class="card" id="sec-wat">
      <h2>2) WATソース（AIが埋め込み / 編集可能）</h2>
      <p class="muted">デフォルトは「Hello, WAT!」。<span class="tip">CDNロード後にアセンブル -> 実行してください。</span></p>
      <textarea id="wat" class="mono"></textarea>
      <div class="row" style="margin-top:8px">
        <button id="assemble" class="btn" disabled>アセンブル（WAT -> Wasm）</button>
        <span id="asm-status" class="muted">-</span>
      </div>
      <div class="row" style="margin-top:4px">
        <label class="row" style="gap:6px; align-items:center">
          <input type="checkbox" id="auto-run" />
          <span class="muted">アセンブル成功時に自動実行</span>
        </label>
      </div>
    </section>

    <section class="card" id="sec-wasm">
      <h2>3) 既存Wasmをロード（アセンブルなしで直接実行）</h2>
      <p class="muted">事前ビルド済み<code>.wasm</code>をロードして直接実行（ステップ2が不要な場合）。</p>
      <div class="row" style="margin-top:6px">
        <label class="btn" for="wasm-file">main.wasmを選択</label>
        <input id="wasm-file" type="file" accept=".wasm,application/wasm" style="display:none" />
        <span id="wasm-file-name" class="mono muted">（未選択）</span>
      </div>
    </section>

    <section class="card" id="sec-run">
      <h2>4) 実行とダウンロード</h2>
      <div class="row" style="margin-top:8px">
        <button id="run" class="btn" disabled>実行（WASI: fd_writeのみ）</button>
        <button id="tests" class="btn" disabled>セルフテストを実行</button>
        <span id="status" class="muted">アイドル</span>
      </div>
      <div class="kv" style="margin-top:8px">
        <div>WASMソース:</div><div id="wasm-src" class="mono muted">（なし）</div>
        <div>WASMサイズ:</div><div id="wasm-size" class="mono muted">-</div>
        <div>SHA-256:</div><div id="sha256" class="mono muted">-</div>
        <div>終了コード:</div><div id="exit" class="mono muted">-</div>
      </div>
      <div class="row" style="margin-top:8px">
        <button id="dl-wasm" class="btn" disabled>wasmをダウンロード (main.wasm)</button>
      </div>
    </section>

    <section class="card" id="sec-log">
      <h2>5) ログ / 出力</h2>
      <div style="display:grid; grid-template-columns:1fr; gap:8px">
        <div>
          <div class="muted">ログ</div>
          <pre id="log" class="out mono"></pre>
        </div>
        <div>
          <div class="muted">stdout</div>
          <pre id="stdout" class="out mono"></pre>
        </div>
        <div>
          <div class="muted">stderr</div>
          <pre id="stderr" class="out mono"></pre>
        </div>
      </div>
      <div class="footer muted">wabt.js（CDNのみ）でWAT->Wasmを生成。最小限WASI（fd_write）フックで実行。</div>
    </section>
      <section class="card" id="sec-links">
      <h2>参考リンク</h2>
      <ul>
        <li><a href="https://developer.mozilla.org/ja/docs/WebAssembly/Guides/Concepts" target="_blank" rel="noopener">WebAssembly概要 - WebAssembly | MDN</a></li>
        <li><a href="https://developer.mozilla.org/ja/docs/WebAssembly/Guides/Text_format_to_Wasm" target="_blank" rel="noopener">WebAssemblyテキスト形式からWasmへの変換 - WebAssembly | MDN</a></li>
      </ul>
    </section>
  </main>

  <!-- デフォルトWATソース（Helloサンプル） -->
  <!-- AI_INSERT: このタグのinnerTextを生成したWATで置換 -->
  <script type="text/plain" id="wat-source">;; wasm32-wasi / _startで"Hello, WAT!
"を出力
(module
  (import "wasi_snapshot_preview1" "fd_write"
    (func $fd_write (param i32 i32 i32 i32) (result i32)))
  (memory (export "memory") 1)
  ;; iovec[0] は 0..7 (buf, len)、メッセージは 8..、nwrittenは 20..23
  (data (i32.const 8) "Hello, WAT!\0a")
  (func (export "_start")
    (i32.store (i32.const 0) (i32.const 8))   ;; iov.buf = 8
    (i32.store (i32.const 4) (i32.const 12))  ;; iov.len = 12
    (call $fd_write
      (i32.const 1) (i32.const 0) (i32.const 1) (i32.const 20))
    drop
  )
)</script>

  <script type="module">
    const $ = (s) => document.querySelector(s);
    const log = (m, cls="") => { const p = $("#log"); const t = document.createElement("div"); t.textContent = m; if(cls) t.className = cls; p.appendChild(t); p.scrollTop = p.scrollHeight; };
    const setStatus = (s) => $("#status").textContent = s;

    // 初期テキスト
    $("#wat").value = (document.getElementById("wat-source")?.textContent || "").trim();

    // ==== 状態管理 ====
    let wabt = null;           // WABTインスタンス
    let wabtReady = false;     // parseWat可能
    let wasmBytes = null;      // 現在選択/生成されたWasm
    let wasmName = "main.wasm";// ダウンロード名
    let wasmSource = "（なし）"; // アセンブル済み / アップロード済み

    function setWasm(bytes, name, sourceTag) {
      wasmBytes = bytes; wasmName = name || "main.wasm"; wasmSource = sourceTag || "（不明）";
      $("#wasm-src").textContent = wasmSource;
      $("#wasm-size").textContent = bytes ? String(bytes.length) + " バイト" : "-";
      $("#dl-wasm").disabled = !bytes;
      $("#run").disabled = !bytes;
      updateSha(bytes);
    }

    async function updateSha(bytes){
      if (!bytes) { $("#sha256").textContent = "-"; return; }
      const hash = await crypto.subtle.digest("SHA-256", bytes);
      const hex = [...new Uint8Array(hash)].map(b=>b.toString(16).padStart(2,"0")).join("");
      $("#sha256").textContent = hex;
    }

    function download(name, u8, mime="application/wasm"){
      const blob = new Blob([u8], { type: mime });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a"); a.href = url; a.download = name; a.click(); URL.revokeObjectURL(url);
    }

    // ---- 自動スクロールヘルパー（auto-runがチェックされている場合のみ） ----
    function autoScrollTo(selector){
      const auto = document.getElementById("auto-run");
      if (!auto || !auto.checked) return;
      const el = document.querySelector(selector);
      if (el) el.scrollIntoView({ behavior: 'smooth', block: 'start' });
    }

    // ==== CDNローダー（公式ESMエントリのみ） ====
    async function loadWabtFromCDN() {
      // 再ロードアクション：状態を初期化して再試行
      $("#cdn-warn").style.display = 'none';
      $("#cdn-warn").textContent = '';
      wabt = null; wabtReady = false;
      $("#assemble").disabled = true; $("#tests").disabled = true;
      $("#wabt-cdn-status").textContent = "ロード中 ...";

      const ts = Date.now();
      const addTs = (u) => u + (u.includes('?') ? '&' : '?') + 'ts=' + ts; // キャッシュ回避
      const tryList = [
        async () => { const m = await import(addTs("https://cdn.jsdelivr.net/npm/wabt@1.0.37/index.js")); return m.default(); },
        async () => { const m = await import(addTs("https://unpkg.com/wabt@1.0.37/index.js")); return m.default(); },
        async () => { const m = await import(addTs("https://esm.sh/wabt@1.0.37")); return (m && m.default) ? m.default() : (typeof m === 'function' ? m() : null); },
      ];
      for (const fn of tryList) {
        try {
          const inst = await fn();
          if (inst && typeof inst.parseWat === "function") {
            wabt = inst; wabtReady = true;
            $("#wabt-cdn-status").textContent = "ロード済み";
            $("#assemble").disabled = false; $("#tests").disabled = false;
            log("wabt.js準備完了（CDN）", "ok");
            return;
          }
        } catch (e) { console.warn("CDNロード失敗:", e); }
      }
      $("#wabt-cdn-status").textContent = "失敗";
      const warn = $("#cdn-warn");
      warn.textContent = "すべてのCDNがブロックされている場合は実行できません。";
      warn.style.display = 'block';
      log("すべてのCDNローダーが失敗しました。", "err");
    }

    $("#load-wabt-cdn").addEventListener("click", loadWabtFromCDN);

    // ==== アセンブル（WAT -> Wasm） ====
    async function assemble() {
      try {
        if (!wabtReady) throw new Error("最初にCDNからWABTをロードしてください。");
        setStatus("アセンブル中 ..."); $("#stdout").textContent = ""; $("#stderr").textContent = ""; $("#exit").textContent = "-"; $("#sha256").textContent = "-"; $("#wasm-size").textContent = "-"; $("#dl-wasm").disabled = true; $("#run").disabled = true; autoScrollTo('#sec-wat');
        const watText = $("#wat").value; log("[wabt] parse & toBinary ...");
        const mod = wabt.parseWat("inline", watText, {}); const bin = mod.toBinary({ log:false, write_debug_names:true }); mod.destroy();
        setWasm(bin.buffer, "main.wasm", "アセンブル済み"); $("#asm-status").textContent = "ok"; setStatus("アセンブル ok"); log("[wabt] ok", "ok");
        if (document.getElementById("auto-run")?.checked) {
          autoScrollTo('#sec-run');
          log("[auto] アセンブル後に実行");
          await runWasi();
        }
      } catch (e) {
        console.error(e); setStatus("アセンブルエラー");
        autoScrollTo('#sec-log'); $("#asm-status").textContent = "エラー"; log("アセンブルエラー: "+String(e?.message || e), "err");
      }
    }
    $("#assemble").addEventListener("click", assemble);

    // ==== 既存Wasmをロード ====
    $("#wasm-file").addEventListener("change", async (e)=>{
      const f = e.target.files?.[0] || null; $("#wasm-file-name").textContent = f ? f.name : "（未選択）";
      if (!f) return; const bytes = new Uint8Array(await f.arrayBuffer()); setWasm(bytes, f.name || "main.wasm", "アップロード済み"); log("wasmアップロード済み: "+(f.name||"（無名）"), "ok");
    });

    // ==== 実行（最小限WASI: fd_writeフック） ====
    async function runWasi() {
      if (!wasmBytes) return;
      try {
        setStatus("実行中 ..."); $("#stdout").textContent = ""; $("#stderr").textContent = ""; $("#exit").textContent = "-"; autoScrollTo('#sec-run');
        let memory = null; const out = { 1: [], 2: [] }; const decoder = new TextDecoder();
        function fd_write(fd, iovs_ptr, iovs_len, nwritten_ptr) {
          const mem = new DataView(memory.buffer); let written = 0;
          for (let i = 0; i < iovs_len; i++) { const base = iovs_ptr + i * 8; const ptr = mem.getUint32(base, true); const len = mem.getUint32(base + 4, true); const bytes = new Uint8Array(memory.buffer, ptr, len); out[fd]?.push(decoder.decode(bytes)); written += len; }
          if (nwritten_ptr) mem.setUint32(nwritten_ptr, written, true); return 0;
        }
        const imports = { wasi_snapshot_preview1: { fd_write } };
        const { instance } = await WebAssembly.instantiate(wasmBytes, imports);
        memory = instance.exports.memory;
        if (typeof instance.exports._start === "function") instance.exports._start(); else if (typeof instance.exports._main === "function") instance.exports._main(); else throw new Error("_startも_mainも見つかりません");
        $("#stdout").textContent = (out[1]||[]).join(""); $("#stderr").textContent = (out[2]||[]).join(""); $("#exit").textContent = "0"; setStatus("完了"); autoScrollTo('#sec-log');
      } catch (e) { console.error(e); log("実行エラー: "+String(e), "err"); autoScrollTo('#sec-log');
        setStatus("実行エラー"); }
    }
    $("#run").addEventListener("click", runWasi);

    // ==== セルフテスト（追加テスト） ====
    async function runTests() {
      if (!wabtReady) { log("[TEST] wabtがロードされていません", "err"); return; }
      log("[TEST] 開始 ...");
      // テスト1: Hello WAT -> アセンブル -> 実行 -> stdout一致
      try {
        const wat = document.getElementById("wat-source").textContent.trim();
        const m = wabt.parseWat("t1", wat, {}); const bin = m.toBinary({ log:false, write_debug_names:true }); m.destroy();
        const bytes = bin.buffer; const stdout = await (async () => {
          let memory=null; const acc={1:[],2:[]}; const dec=new TextDecoder();
          function fd_write(fd,iovs,iovlen,nw){ const dv=new DataView(memory.buffer); let w=0; for(let i=0;i<iovlen;i++){ const b=iovs+i*8; const p=dv.getUint32(b,true); const l=dv.getUint32(b+4,true); const byt=new Uint8Array(memory.buffer,p,l); acc[fd]?.push(dec.decode(byt)); w+=l;} if(nw) dv.setUint32(nw,w,true); return 0; }
          const { instance } = await WebAssembly.instantiate(bytes,{wasi_snapshot_preview1:{fd_write}}); memory=instance.exports.memory; instance.exports._start(); return (acc[1]||[]).join("");
        })();
        const expected = "Hello, WAT!\n"; if (stdout===expected) log("[TEST1] stdout equality - PASS", "ok"); else log(`[TEST1] stdout equality - FAIL (期待値 "${expected}"、実際 "${stdout}")`, "err");
      } catch (e) { log("[TEST1] stdout equality - ERROR: "+String(e), "err"); }
      // テスト2: 構文エラー検出
      try {
        const bad = "(module (func (export \"_start\") unreachable)"; let threw=false; try { wabt.parseWat("t2", bad, {}); } catch (_) { threw=true; }
        if (threw) log("[TEST2] syntax error detection - PASS", "ok"); else log("[TEST2] syntax error detection - FAIL (エラーが投げられませんでした)", "err");
      } catch (e) { log("[TEST2] syntax error detection - ERROR: "+String(e), "err"); }
      log("[TEST] 完了。");
    }
    $("#tests").addEventListener("click", runTests);

    // ダウンロード
    $("#dl-wasm").addEventListener("click", () => { if (wasmBytes) download(wasmName||"main.wasm", wasmBytes); });

    // 初期状態
    $("#asm-status").textContent = "-";
    // 起動時にCDNロードを自動実行
    loadWabtFromCDN();
  </script>
</body>
</html>

```

---

## 6. 動作確認（チェックリスト）
1) ページを開く -> 上部ステータスが**ロード済み**（CDN成功）。  
2) **アセンブル** -> **WASMサイズ / SHA-256**が更新、**ダウンロード**が利用可能に。  
3) **実行** -> `stdout`に期待する文字列が表示、終了コード=0。  
4) **セルフテストを実行** ->  
   - `[TEST1] stdout equality - PASS`  
   - `[TEST2] syntax error detection - PASS`  
5) 既存の`main.wasm`を選択して**実行**でも動作。

---

## 7. 既知の制限とトラブルシューティング
- **すべてのCDNがブロックされている場合は実行できません。**（このテンプレートはローカルロードがありません）  
- 文字列長のミス（`iov.len`）により**視覚的に一致してもFAIL**となることがあります（NUL挿入など）。  
- `"_startも_mainも見つかりません"`：WATで`_start`をエクスポートしていない可能性があります。  
- **HTTPS配信推奨**でブラウザの差異を軽減（`file://`でも動作することが多いが環境依存）。
