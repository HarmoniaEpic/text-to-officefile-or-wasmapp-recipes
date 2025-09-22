# WAT_SingleHTML_WASI_Recipe_v3 — 「AI→WAT→wasm32-wasi」を**単一HTMLで**コンパイル＆実行するための完全テンプレ（COI不要 / CDN ESMのみ / 最小WASI直実行）

> 本レシピは、添付の単一HTML（最新版）を**全文テンプレ**として採用し、**Math-RECIPE と同等の“テンプレート力”**に寄せて整備した v3 です。  
> これにより、**ユーザー要望 → AIがWATを生成 → 単一HTMLが .wat→.wasm にアセンブル → ブラウザ内でWASI実行**という PoC フローが迷いなく再現できます。

---

## 1. 役割分担（AI と 単一HTML）

- **AI（WAT生成エージェント）**
  - ユーザーの要望（自然言語）を **WAT（WebAssembly Text）** に変換。
  - 変換結果の **WATコードを、下記テンプレHTMLの `<script type="text/plain" id="wat-source">` の中身に**そのまま挿入（置換）。
  - 必要なら文字列長（iovec.len）などの**バイト長一致**を保証。

- **単一HTML（実行器）**
  - ページ表示時に **wabt@1.0.37（ESM `index.js`）** を CDN から自動ロード。
  - **Assemble** で WAT→Wasm、**Run** で最小WASI（`fd_write`）により実行。  
  - **Self-tests**、**SHA-256**、**WASMダウンロード**、**既存WASMの直接実行**、**（任意）自動実行トグル**、**進行に応じたオートスクロール**を備える。

---

## 2. AI 実装の手順（**コピペ運用向け／変更禁止点を含む**）

**ステップA：WATを生成する（SYSTEM テンプレ必須）**
- ターゲットは **wasm32-wasi**。  
- **`_start` を export**（`(start ...)` は使わない）。  
- 標準出力のみなら `fd_write` を import。  
- **(memory (export "memory") 1)** を定義し、iovec（0..7）/ メッセージ（8..）/書込数（20..23）を使う。  
- **文字列長は厳密なバイト数**（改行は `\0a`、終端NULは不要）。

**ステップB：テンプレHTMLへ差し込む（ここが“決め位置”）**
- `<script type="text/plain" id="wat-source"> ... </script>` の **中身全体を置換**する。  
- タグの **属性・ID・`type` を変更してはいけない**（削除も不可）。

**ステップC：HTMLは他を触らない**
- **CDN URL は `index.js` のみ**（`/wabt.js` や `/dist/wabt.js` を **使わない**）。  
- **ロード順**（jsDelivr → UNPKG → esm.sh）の変更禁止。  
- UI の ID 名、ログ書式（`stdout equality — PASS` など）は変更禁止。

---

## 3. **変更禁止チェックリスト**（AI/人間の双方が遵守）

- [ ] `<script id="wat-source">` の **タグ属性は不変**。中身だけ置換。  
- [ ] **wabt@1.0.37** の **`index.js` だけ**をインポート（UMDの `/wabt.js` 系は **禁止**）。  
- [ ] ロード順：**jsDelivr → UNPKG → esm.sh**。  
- [ ] `_start` を **必ず export**。`(start ...)` は **使用禁止**。  
- [ ] 文字列長（`iov.len`）は **厳密**（例：`"Hello, WAT!\n"` は **12**）。  
- [ ] 自己テストのログ書式（`stdout equality — ...` / `syntax error detection — ...`）は **変更禁止**。

---

## 4. WAT 生成テンプレ（AI用 SYSTEM 指示）

````text
SYSTEM（不変ルール / WAT生成エージェント）:

【ターゲット】
- wasm32-wasi。単一の (module) を出力。コードのみ。

【エクスポート】
- (func (export "_start") ...) を必ず定義。 (start ...) は使わない。

【インポート】
- 標準出力のみなら:
  (import "wasi_snapshot_preview1" "fd_write"
    (func $fd_write (param i32 i32 i32 i32) (result i32)))

【メモリとレイアウト】
- (memory (export "memory") 1) を定義。
- iovec[0] = {{buf(0..3), len(4..7)}}。メッセージは 8..、nwritten は 20..23。
- 文字列長（len）は厳密なバイト数。終端NULは不要。改行は "\0a"。

【品質チェック】
- すべての i32.store/load は 4 バイト境界。
- fd_write の第4引数は書込済みバイト数を書き戻すアドレス。
- wabt(parseWat→toBinary) で構文/型エラーが出ない。

【禁止事項】
- (start ...) / 実験機能（threads, simd, ref.func など）禁止。
- 巨大データや無限ループの生成禁止。

【出力形式】
- 出力は WAT のみを ```wat で包む。前後に説明文を付けない。

USER 要望:
<<ここにユーザーの要望を入れる>>
````

---

## 5. 完成**単一HTMLテンプレ（全文）**
> 下記を **そのまま .html として保存**し、**`<script id="wat-source">` の中身だけを WAT で置換**して使います。  
> ベースは添付HTML（最新版）。一部コメントに **AI_INSERT マーカー**を付与しています。

```html
<!doctype html>
<html lang="ja">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>WAT → Wasm32-WASI 組み立て＆実行（COI不要・CDNロードのみ・既存WASM読込）</title>
  <style>
    :root { color-scheme: light dark; }
    body { margin: 0; font-family: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Noto Sans", Arial; line-height: 1.5; }
    header { padding: 12px 16px; background: #0001; }
    main { display: grid; grid-template-columns: 1fr; gap: 12px; padding: 12px; }
    @media (min-width: 1120px) { main { grid-template-columns: 1fr 1fr; } }
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
    <h1>WAT → Wasm32-WASI 組み立て＆実行 <span class="badge">COI不要 / CDNロードのみ（正規URL） / 既存WASM読込</span></h1>
  </header>

  <main>
    <section class="card" id="sec-load">
      <h2>1) WABT（アセンブラ）をCDNからロード</h2>
      <p class="muted">wabt は <b>正規の ESM エントリ（<code>index.js</code>）</b>のみを使用します。UMD の <code>/wabt.js</code> や <code>/dist/wabt.js</code> は使いません。</p>
      <div class="row" style="margin-top:6px">
        <button id="load-wabt-cdn" class="btn">CDN のリロード</button>
        <span id="wabt-cdn-status" class="muted">not loaded</span>
      </div>
      <details style="margin-top:6px">
        <summary>使用するURL（順番どおりにリトライ）</summary>
        <ul class="mono muted">
          <li>1) jsDelivr: https://cdn.jsdelivr.net/npm/wabt@1.0.37/index.js</li>
          <li>2) UNPKG: https://unpkg.com/wabt@1.0.37/index.js</li>
          <li>3) esm.sh: https://esm.sh/wabt@1.0.37</li>
        </ul>
      </details>
      <div id="cdn-warn" class="warn" style="margin-top:6px; display:none"></div>
    </section>

    <section class="card" id="sec-wat">
      <h2>2) WAT Source（AIが埋め込み／編集可）</h2>
      <p class="muted">既定は「Hello, WAT!」。<span class="tip">CDNロード後に Assemble → Run してください。</span></p>
      <textarea id="wat" class="mono"></textarea>
      <div class="row" style="margin-top:8px">
        <button id="assemble" class="btn" disabled>Assemble (WAT → Wasm)</button>
        <span id="asm-status" class="muted">-</span>
      </div>
      <div class="row" style="margin-top:4px">
        <label class="row" style="gap:6px; align-items:center">
          <input type="checkbox" id="auto-run" />
          <span class="muted">アセンブル成功の場合は自動実行</span>
        </label>
      </div>
    </section>

    <section class="card" id="sec-wasm">
      <h2>3) 既存 Wasm の読込（組み立て不要で直接実行）</h2>
      <p class="muted">作成済みの <code>.wasm</code> を読み込み、そのまま実行可能です（手順2不要の場合）。</p>
      <div class="row" style="margin-top:6px">
        <label class="btn" for="wasm-file">main.wasm を選択</label>
        <input id="wasm-file" type="file" accept=".wasm,application/wasm" style="display:none" />
        <span id="wasm-file-name" class="mono muted">(未選択)</span>
      </div>
    </section>

    <section class="card" id="sec-run">
      <h2>4) 実行とダウンロード</h2>
      <div class="row" style="margin-top:8px">
        <button id="run" class="btn" disabled>Run (WASI: fd_writeのみ)</button>
        <button id="tests" class="btn" disabled>Run self-tests</button>
        <span id="status" class="muted">idle</span>
      </div>
      <div class="kv" style="margin-top:8px">
        <div>WASM source:</div><div id="wasm-src" class="mono muted">(none)</div>
        <div>WASM size:</div><div id="wasm-size" class="mono muted">-</div>
        <div>SHA-256:</div><div id="sha256" class="mono muted">-</div>
        <div>Exit code:</div><div id="exit" class="mono muted">-</div>
      </div>
      <div class="row" style="margin-top:8px">
        <button id="dl-wasm" class="btn" disabled>Download wasm (main.wasm)</button>
      </div>
    </section>

    <section class="card" id="sec-log">
      <h2>5) Log / Output</h2>
      <div style="display:grid; grid-template-columns:1fr; gap:8px">
        <div>
          <div class="muted">Log</div>
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
      <div class="footer muted">wabt.js（CDNのみ）で WAT→Wasm を生成。最小WASI（fd_write）フックで実行。</div>
    </section>
      <section class="card" id="sec-links">
      <h2>参考リンク</h2>
      <ul>
        <li><a href="https://developer.mozilla.org/ja/docs/WebAssembly/Guides/Concepts" target="_blank" rel="noopener">WebAssembly の概要 - WebAssembly | MDN</a></li>
        <li><a href="https://developer.mozilla.org/ja/docs/WebAssembly/Guides/Text_format_to_Wasm" target="_blank" rel="noopener">WebAssembly テキスト形式から Wasm への変換 - WebAssembly | MDN</a></li>
      </ul>
    </section>
  </main>

  <!-- 既定の WAT ソース（Hello 例） -->
  <!-- AI_INSERT: Replace the innerText of this tag with generated WAT -->
  <script type="text/plain" id="wat-source">;; wasm32-wasi / _start で "Hello, WAT!
" を出力
(module
  (import "wasi_snapshot_preview1" "fd_write"
    (func $fd_write (param i32 i32 i32 i32) (result i32)))
  (memory (export "memory") 1)
  ;; iovec[0] は 0..7（buf, len）、メッセージは 8..、nwritten は 20..23 を使用
  (data (i32.const 8) "Hello, WAT! \0a")
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
    let wabt = null;           // WABT インスタンス
    let wabtReady = false;     // parseWat できるか
    let wasmBytes = null;      // 現在選択/生成されている Wasm
    let wasmName = "main.wasm";// ダウンロード名
    let wasmSource = "(none)"; // assembled / uploaded

    function setWasm(bytes, name, sourceTag) {
      wasmBytes = bytes; wasmName = name || "main.wasm"; wasmSource = sourceTag || "(unknown)";
      $("#wasm-src").textContent = wasmSource;
      $("#wasm-size").textContent = bytes ? String(bytes.length) + " bytes" : "-";
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

    // ---- Auto scroll helper (only when auto-run is checked) ----
    function autoScrollTo(selector){
      const auto = document.getElementById("auto-run");
      if (!auto || !auto.checked) return;
      const el = document.querySelector(selector);
      if (el) el.scrollIntoView({ behavior: 'smooth', block: 'start' });
    }

    // ==== CDN ローダ（正規 ESM エントリのみ）====
    async function loadWabtFromCDN() {
      // リロード動作：状態を初期化し、再試行する
      $("#cdn-warn").style.display = 'none';
      $("#cdn-warn").textContent = '';
      wabt = null; wabtReady = false;
      $("#assemble").disabled = true; $("#tests").disabled = true;
      $("#wabt-cdn-status").textContent = "loading ...";

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
            $("#wabt-cdn-status").textContent = "loaded";
            $("#assemble").disabled = false; $("#tests").disabled = false;
            log("wabt.js ready (CDN)", "ok");
            return;
          }
        } catch (e) { console.warn("CDN load failed:", e); }
      }
      $("#wabt-cdn-status").textContent = "failed";
      const warn = $("#cdn-warn");
      warn.textContent = "CDNがすべて塞がれている場合は実行できません。";
      warn.style.display = 'block';
      log("All CDN loaders failed.", "err");
    }

    $("#load-wabt-cdn").addEventListener("click", loadWabtFromCDN);

    // ==== Assemble（WAT → Wasm）====
    async function assemble() {
      try {
        if (!wabtReady) throw new Error("まず WABT をCDNから読み込んでください。");
        setStatus("assembling ..."); $("#stdout").textContent = ""; $("#stderr").textContent = ""; $("#exit").textContent = "-"; $("#sha256").textContent = "-"; $("#wasm-size").textContent = "-"; $("#dl-wasm").disabled = true; $("#run").disabled = true; autoScrollTo('#sec-wat');
        const watText = $("#wat").value; log("[wabt] parse & toBinary ...");
        const mod = wabt.parseWat("inline", watText, {}); const bin = mod.toBinary({ log:false, write_debug_names:true }); mod.destroy();
        setWasm(bin.buffer, "main.wasm", "assembled"); $("#asm-status").textContent = "ok"; setStatus("assemble ok"); log("[wabt] ok", "ok");
        if (document.getElementById("auto-run")?.checked) {
          autoScrollTo('#sec-run');
          log("[auto] run after assemble");
          await runWasi();
        }
      } catch (e) {
        console.error(e); setStatus("assemble error");
        autoScrollTo('#sec-log'); $("#asm-status").textContent = "error"; log("Assemble error: "+String(e?.message || e), "err");
      }
    }
    $("#assemble").addEventListener("click", assemble);

    // ==== 既存 Wasm 読込 ====
    $("#wasm-file").addEventListener("change", async (e)=>{
      const f = e.target.files?.[0] || null; $("#wasm-file-name").textContent = f ? f.name : "(未選択)";
      if (!f) return; const bytes = new Uint8Array(await f.arrayBuffer()); setWasm(bytes, f.name || "main.wasm", "uploaded"); log("wasm uploaded: "+(f.name||"(unnamed)"), "ok");
    });

    // ==== 実行（最小 WASI: fd_write フック）====
    async function runWasi() {
      if (!wasmBytes) return;
      try {
        setStatus("running ..."); $("#stdout").textContent = ""; $("#stderr").textContent = ""; $("#exit").textContent = "-"; autoScrollTo('#sec-run');
        let memory = null; const out = { 1: [], 2: [] }; const decoder = new TextDecoder();
        function fd_write(fd, iovs_ptr, iovs_len, nwritten_ptr) {
          const mem = new DataView(memory.buffer); let written = 0;
          for (let i = 0; i < iovs_len; i++) { const base = iovs_ptr + i * 8; const ptr = mem.getUint32(base, true); const len = mem.getUint32(base + 4, true); const bytes = new Uint8Array(memory.buffer, ptr, len); out[fd]?.push(decoder.decode(bytes)); written += len; }
          if (nwritten_ptr) mem.setUint32(nwritten_ptr, written, true); return 0;
        }
        const imports = { wasi_snapshot_preview1: { fd_write } };
        const { instance } = await WebAssembly.instantiate(wasmBytes, imports);
        memory = instance.exports.memory;
        if (typeof instance.exports._start === "function") instance.exports._start(); else if (typeof instance.exports._main === "function") instance.exports._main(); else throw new Error("_start も _main も見つかりません");
        $("#stdout").textContent = (out[1]||[]).join(""); $("#stderr").textContent = (out[2]||[]).join(""); $("#exit").textContent = "0"; setStatus("done"); autoScrollTo('#sec-log');
      } catch (e) { console.error(e); log("Run error: "+String(e), "err"); autoScrollTo('#sec-log');
        setStatus("run error"); }
    }
    $("#run").addEventListener("click", runWasi);

    // ==== Self-tests（追加テスト）====
    async function runTests() {
      if (!wabtReady) { log("[TEST] wabt not loaded", "err"); return; }
      log("[TEST] start ...");
      // Test 1: Hello WAT → assemble → run → stdout 一致
      try {
        const wat = document.getElementById("wat-source").textContent.trim();
        const m = wabt.parseWat("t1", wat, {}); const bin = m.toBinary({ log:false, write_debug_names:true }); m.destroy();
        const bytes = bin.buffer; const stdout = await (async () => {
          let memory=null; const acc={1:[],2:[]}; const dec=new TextDecoder();
          function fd_write(fd,iovs,iovlen,nw){ const dv=new DataView(memory.buffer); let w=0; for(let i=0;i<iovlen;i++){ const b=iovs+i*8; const p=dv.getUint32(b,true); const l=dv.getUint32(b+4,true); const byt=new Uint8Array(memory.buffer,p,l); acc[fd]?.push(dec.decode(byt)); w+=l;} if(nw) dv.setUint32(nw,w,true); return 0; }
          const { instance } = await WebAssembly.instantiate(bytes,{wasi_snapshot_preview1:{fd_write}}); memory=instance.exports.memory; instance.exports._start(); return (acc[1]||[]).join("");
        })();
        const expected = "Hello, WAT!\n"; if (stdout===expected) log("[TEST1] stdout equality — PASS", "ok"); else log(`[TEST1] stdout equality — FAIL (expected "${expected}", got "${stdout}")`, "err");
      } catch (e) { log("[TEST1] stdout equality — ERROR: "+String(e), "err"); }
      // Test 2: 構文エラー検知
      try {
        const bad = "(module (func (export \"_start\") unreachable)"; let threw=false; try { wabt.parseWat("t2", bad, {}); } catch (_) { threw=true; }
        if (threw) log("[TEST2] syntax error detection — PASS", "ok"); else log("[TEST2] syntax error detection — FAIL (no error thrown)", "err");
      } catch (e) { log("[TEST2] syntax error detection — ERROR: "+String(e), "err"); }
      log("[TEST] done.");
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
1) ページを開く → 上部ステータスが **loaded** になる（CDN成功）。  
2) **Assemble** → **WASM size / SHA-256** が更新され、**Download** が有効になる。  
3) **Run** → `stdout` に期待文字列が表示され、Exit code=0。  
4) **Run self-tests** →  
   - `[TEST1] stdout equality — PASS`  
   - `[TEST2] syntax error detection — PASS`  
5) 既存 `main.wasm` を選んで **Run** しても動く。

---

## 7. 既知の制約とトラブルシュート
- **CDNがすべて塞がれている場合は実行できません。**（本テンプレはローカル読込を持ちません）  
- 文字列長ミス（`iov.len`）で **見た目一致でも FAIL** になることがある（NUL混入など）。  
- `"_start も _main も見つかりません"`：WATで `_start` を export していない可能性。  
- ブラウザ差異を減らすため **HTTPS配信推奨**（`file://` でも動くことは多いが環境依存）。
