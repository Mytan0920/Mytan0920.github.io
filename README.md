
<!doctype html>

<html lang="ja">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>AI拓哉 音声ジェネレーター</title>
  <style>
    body{font-family:system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial;margin:20px;background:#f7f7fb;color:#111}
    .card{background:#fff;padding:18px;border-radius:12px;box-shadow:0 6px 18px rgba(20,20,50,0.06);max-width:820px;margin:0 auto}
    textarea{width:100%;min-height:100px;padding:10px;font-size:16px;border-radius:8px;border:1px solid #e0e6f0}
    input[type=text]{width:100%;padding:10px;border-radius:8px;border:1px solid #e0e6f0}
    button{padding:10px 14px;border-radius:8px;border:0;background:#2463ff;color:#fff;font-weight:600;cursor:pointer}
    .muted{color:#556}
    .row{display:flex;gap:10px;align-items:center}
    .small{font-size:13px}
    pre{white-space:pre-wrap;background:#f1f5ff;padding:10px;border-radius:8px}
    .controls{display:flex;gap:8px;flex-wrap:wrap}
  </style>
</head>
<body>
  <div class="card">
    <h1>AI拓哉 音声ジェネレーター</h1>
    <p class="muted">指定されたURLの <code>TXT=</code> の後ろの部分に任意の文字列を差し込み、音声ファイル（MP3）のURLを作成して再生・ダウンロードできます。</p>

    <label for="txt">テキスト（ここに入れたい文字を入力）</label>
    <textarea id="txt" placeholder="例: 拓哉の射精3000円">拓哉の射精3000円</textarea>

    <div style="height:8px"></div>

    <label for="base">ベースURL（必要なら編集）</label>
    <input id="base" type="text" value="https://cache-a.oddcast.com/tts/genC.php?EID=3&LID=12&VID=2&TXT=&EXT=mp3&FNAME=&ACC=9066743&SceneID=2770703&HTTP_ERR=" />

    <div style="height:12px"></div>

    <div class="controls">
      <button id="gen">URLを生成して再生</button>
      <button id="copy">URLをコピー</button>
      <button id="open">別タブで開く</button>
      <button id="clear">入力をクリア</button>
    </div>

    <div style="height:12px"></div>

    <div>
      <div class="small">生成されたURL（エンコード済み）:</div>
      <pre id="out">--</pre>
    </div>

    <div style="height:8px"></div>

    <audio id="player" controls preload="none"></audio>

    <div style="height:8px"></div>
    <div class="small muted">注意: サイト側のCORS設定などによりブラウザで直接再生・ダウンロードできない場合があります。その場合は「別タブで開く」を使って直接アクセスしてください。</div>
  </div>

  <script>
    const txtEl = document.getElementById('txt');
    const baseEl = document.getElementById('base');
    const outEl = document.getElementById('out');
    const player = document.getElementById('player');

    function buildUrl(base, text){
      // エンコードした値で TXT= の部分を置換する。
      // ベースURLに既に 'TXT=' が含まれていることを前提に動作します。
      const encoded = encodeURIComponent(text);
      // replace the first occurrence of TXT=... (even if it's TXT=【】)
      if(/TXT=[^&]*/.test(base)){
        return base.replace(/TXT=[^&]*/,'TXT=' + encoded);
      }
      // fallback: append
      return base + (base.includes('?') ? '&' : '?') + 'TXT=' + encoded;
    }

    function generate(play){
      const base = baseEl.value.trim();
      const text = txtEl.value || '';
      const url = buildUrl(base, text);
      outEl.textContent = url;
      // set audio src and try to play if requested
      player.pause();
      player.src = url;
      player.crossOrigin = 'anonymous';
      player.load();
      if(play){
        // attempt to play; browser may block autoplay until user interaction
        player.play().catch(e=>{
          console.warn('自動再生がブロックされました:', e);
        });
      }
    }

    document.getElementById('gen').addEventListener('click', ()=> generate(true));

    document.getElementById('copy').addEventListener('click', async ()=>{
      const url = outEl.textContent;
      try{
        await navigator.clipboard.writeText(url);
        alert('URLをコピーしました');
      }catch(e){
        prompt('コピーできませんでした。以下を手動でコピーしてください:', url);
      }
    });

    document.getElementById('open').addEventListener('click', ()=>{
      const url = outEl.textContent === '--' ? buildUrl(baseEl.value.trim(), txtEl.value) : outEl.textContent;
      window.open(url, '_blank');
    });

    document.getElementById('clear').addEventListener('click', ()=>{ txtEl.value = ''; outEl.textContent='--'; player.src=''; });

    // initial build
    generate(false);

  </script>
</body>
</html>
