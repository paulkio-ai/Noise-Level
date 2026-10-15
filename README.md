<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<title>Classroom Noise Monitor</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Atkinson+Hyperlegible:wght@400;700&family=Bricolage+Grotesque:opsz,wght@12..96,700;12..96,800&display=swap">
<style>
/* Layout: one big meter screen for the front of the room; settings tucked below. */
:root{
  --bg:#eef1f4; --panel:#ffffff; --fg:#14202b; --muted:#5a6876; --line:#d3dae1;
  --accent:#2457d6; --ok:#1f9d55; --warn:#e0a100; --hot:#d62839;
  --alert-bg:#d62839; --alert-fg:#ffffff;
  --display:"Bricolage Grotesque","Arial Black",system-ui,sans-serif;
  --body:"Atkinson Hyperlegible",system-ui,-apple-system,"Segoe UI",Roboto,sans-serif;
  color-scheme:light;
}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){
  --bg:#0f161d; --panel:#18222c; --fg:#e8eef4; --muted:#93a3b3; --line:#2a3744;
  --accent:#6d93ff; --ok:#3ccf7d; --warn:#ffc531; --hot:#ff4d5e; color-scheme:dark}}
:root[data-theme="dark"]{
  --bg:#0f161d; --panel:#18222c; --fg:#e8eef4; --muted:#93a3b3; --line:#2a3744;
  --accent:#6d93ff; --ok:#3ccf7d; --warn:#ffc531; --hot:#ff4d5e; color-scheme:dark}

*{box-sizing:border-box}
html,body{height:100%}
body{margin:0;background:var(--bg);color:var(--fg);font-family:var(--body);font-size:18px;line-height:1.45;
  padding-left:max(16px,env(safe-area-inset-left,0px));padding-right:max(16px,env(safe-area-inset-right,0px));
  padding-block:16px}
main{max-width:1200px;margin:0 auto;display:grid;gap:20px}
h1{font-family:var(--display);font-weight:800;font-size:clamp(1.5rem,3vw,2.2rem);margin:0;text-wrap:balance}
header{display:flex;flex-wrap:wrap;gap:12px;align-items:center;justify-content:space-between}
.controls{display:flex;flex-wrap:wrap;gap:10px}
button{font:inherit;font-weight:700;border-radius:12px;border:2px solid var(--line);background:var(--panel);color:var(--fg);
  padding:12px 18px;cursor:pointer;min-height:52px}
button:focus-visible,select:focus-visible,input:focus-visible{outline:3px solid var(--accent);outline-offset:2px}
button.primary{background:var(--accent);border-color:var(--accent);color:#fff}
button.voice-on{border-color:var(--ok)}
button.voice-off{border-color:var(--hot);color:var(--hot)}

.stage{background:var(--panel);border:2px solid var(--line);border-radius:20px;padding:clamp(16px,3vw,32px);display:grid;gap:18px}
.readout{display:flex;flex-wrap:wrap;align-items:baseline;gap:8px 24px;justify-content:space-between}
.big{font-family:var(--display);font-weight:800;font-size:clamp(4rem,14vw,10rem);line-height:.9;font-variant-numeric:tabular-nums}
.big small{font-size:.3em;color:var(--muted);margin-left:.15em}
.status{font-family:var(--display);font-weight:700;font-size:clamp(1.4rem,4vw,2.6rem);padding:.2em .7em;border-radius:999px;background:var(--bg)}
.status.ok{color:var(--ok)} .status.warn{color:var(--warn)} .status.hot{color:var(--hot)} .status.idle{color:var(--muted)}

.meter{position:relative;height:clamp(56px,9vw,96px);border-radius:14px;background:var(--bg);overflow:hidden;border:2px solid var(--line)}
.fill{position:absolute;inset:0 auto 0 0;width:0;background:var(--ok);transition:width .08s linear,background-color .2s}
.thresh{position:absolute;top:-4px;bottom:-4px;width:6px;margin-left:-3px;background:var(--fg);border-radius:3px}
.thresh span{position:absolute;top:6px;left:12px;white-space:nowrap;font-weight:700;font-size:.85rem;color:var(--fg)}
.scale{display:flex;justify-content:space-between;color:var(--muted);font-size:.85rem;font-variant-numeric:tabular-nums}
.hold{height:8px;border-radius:4px;background:var(--bg);overflow:hidden}
.hold div{height:100%;width:0;background:var(--hot)}
.meta{display:flex;flex-wrap:wrap;gap:8px 24px;color:var(--muted);font-size:.95rem}

details{background:var(--panel);border:2px solid var(--line);border-radius:16px;padding:12px 20px}
summary{font-weight:700;cursor:pointer;min-height:40px;display:flex;align-items:center}
.settings{display:grid;grid-template-columns:repeat(auto-fit,minmax(260px,1fr));gap:20px;padding-block:12px}
.field{display:grid;gap:6px;min-width:0}
.field label{font-weight:700}
.field .hint{color:var(--muted);font-size:.85rem}
.field output{font-variant-numeric:tabular-nums;color:var(--accent);font-weight:700}
input[type=range]{width:100%;accent-color:var(--accent);height:32px}
select{font:inherit;padding:10px;border-radius:10px;border:2px solid var(--line);background:var(--bg);color:var(--fg);width:100%}
.sub{font-family:var(--display);font-size:1.2rem;margin:12px 0 0}
.row{display:flex;flex-wrap:wrap;gap:8px}
.row button{min-height:44px;padding:8px 14px}
.row button:disabled{opacity:.45;cursor:default}
#recBtn.recording{background:var(--hot);border-color:var(--hot);color:#fff}
.upload{font-weight:400 !important;font-size:.9rem;color:var(--muted)}
input[type=file]{font:inherit;font-size:.9rem;max-width:100%}
.note{color:var(--muted);font-size:.9rem;max-width:70ch}
.error{color:var(--hot);font-weight:700}

/* Warning overlay */
.alert{position:fixed;inset:0;z-index:10;display:grid;place-items:center;text-align:center;
  background:var(--alert-bg);color:var(--alert-fg);
  padding:max(24px,env(safe-area-inset-top,0px)) max(24px,env(safe-area-inset-right,0px)) max(24px,env(safe-area-inset-bottom,0px)) max(24px,env(safe-area-inset-left,0px));
  animation:flash .6s steps(2,jump-none) infinite alternate}
.alert p{margin:0;font-family:var(--display);font-weight:800;font-size:clamp(2.6rem,8vw,7.5rem);line-height:1.02;text-wrap:balance;max-width:16ch;margin-inline:auto}
.alert p + p{margin:.4em auto 0;font-size:clamp(1.8rem,5vw,4.5rem)}
.alert .dismiss{margin-top:2rem;background:transparent;color:var(--alert-fg);border-color:var(--alert-fg)}
.alert[hidden]{display:none}
@keyframes flash{from{background:#d62839}to{background:#8f0f1d}}
@media (prefers-reduced-motion:reduce){.alert{animation:none}.fill{transition:none}}
</style>
</head>
<body>
<main>
  <header>
    <h1>Classroom Noise Monitor</h1>
    <div class="controls">
      <button id="startBtn" class="primary">Start listening</button>
      <button id="voiceBtn" class="voice-on" aria-pressed="true">🔊 Voice warning: ON</button>
      <button id="testBtn">Test warning</button>
      <button id="fsBtn">Full screen</button>
    </div>
  </header>

  <section class="stage" aria-live="polite">
    <div class="readout">
      <div class="big"><span id="dbNum">--</span><small>dB</small></div>
      <div id="status" class="status idle">Not listening</div>
    </div>
    <div>
      <div class="meter" aria-hidden="true">
        <div class="fill" id="fill"></div>
        <div class="thresh" id="thresh"><span id="threshLabel">Limit 75 dB</span></div>
      </div>
      <div class="scale" aria-hidden="true"><span>30</span><span>45</span><span>60</span><span>75</span><span>90</span><span>105</span></div>
    </div>
    <div>
      <div class="hold" title="Fills while the room stays above the limit"><div id="holdBar"></div></div>
    </div>
    <div class="meta">
      <span>Warnings this session: <strong id="count">0</strong></span>
      <span id="micName">Microphone: not started</span>
    </div>
    <p id="err" class="error" hidden></p>
  </section>

  <details id="settingsBox">
    <summary>Settings</summary>
    <div class="settings">
      <div class="field">
        <label for="threshold">Noise limit <output id="thresholdOut">75 dB</output></label>
        <input type="range" id="threshold" min="45" max="100" step="1" value="75">
        <span class="hint">Quiet work ≈ 50 · normal talking ≈ 60–65 · loud group ≈ 75+</span>
      </div>
      <div class="field">
        <label for="hold">Stay loud for <output id="holdOut">2.0 s</output> before warning</label>
        <input type="range" id="hold" min="0.5" max="8" step="0.5" value="2">
        <span class="hint">Stops a single clap or dropped book from triggering it.</span>
      </div>
      <div class="field">
        <label for="cooldown">Wait <output id="cooldownOut">15 s</output> between warnings</label>
        <input type="range" id="cooldown" min="5" max="120" step="5" value="15">
      </div>
      <div class="field">
        <label for="duration">Show warning for <output id="durationOut">6 s</output></label>
        <input type="range" id="duration" min="3" max="20" step="1" value="6">
      </div>
      <div class="field">
        <label for="calib">Calibration <output id="calibOut">+0 dB</output></label>
        <input type="range" id="calib" min="-30" max="30" step="1" value="0">
        <span class="hint">If you have a sound-meter app, match the reading here. Otherwise leave at 0.</span>
      </div>
      <div class="field">
        <label for="micSelect">Microphone</label>
        <select id="micSelect"><option value="">Default microphone</option></select>
        <span class="hint">Pick the Boxlight's built-in mic if several are listed. Names appear after you press Start.</span>
      </div>
    </div>
    <h2 class="sub">Warning voice</h2>
    <div class="settings">
      <div class="field">
        <label for="voiceMode">What plays when the warning appears</label>
        <select id="voiceMode">
          <option value="tts">Computer voice</option>
          <option value="clip">My recording</option>
        </select>
        <span class="hint">The Voice warning ON/OFF button still mutes either one.</span>
      </div>
      <div class="field">
        <label>Your own recording</label>
        <div class="row">
          <button id="recBtn" type="button">● Record</button>
          <button id="playBtn" type="button">▶ Play</button>
          <button id="delBtn" type="button">Delete</button>
        </div>
        <label class="upload" for="clipFile">or upload an audio file (MP3, WAV, M4A)</label>
        <input type="file" id="clipFile" accept="audio/*">
        <span class="hint" id="recStatus">No recording yet.</span>
      </div>
      <div class="field">
        <label for="voiceName">Computer voice</label>
        <select id="voiceName"><option value="">Default voice</option></select>
        <span class="hint">The voices available depend on the device.</span>
      </div>
      <div class="field">
        <label for="pitch">Pitch <output id="pitchOut">1.00×</output></label>
        <input type="range" id="pitch" min="0.1" max="1.6" step="0.05" value="1">
        <span class="hint">Lower is deeper.</span>
      </div>
      <div class="field">
        <label for="rate">Speed <output id="rateOut">1.00×</output></label>
        <input type="range" id="rate" min="0.6" max="1.4" step="0.05" value="1">
      </div>
    </div>
    <p class="note">Readings are approximate. Browsers report sound relative to the microphone, not a certified decibel meter, so set the limit by watching the meter during a normal class and choosing a number just above where it sits when the volume is fine.</p>
  </details>
</main>

<div class="alert" id="alert" hidden role="alert">
  <div>
    <p>Y'all are being too loud.</p>
    <p>Chill and bring it down.</p>
    <button class="dismiss" id="dismissBtn">Dismiss</button>
  </div>
</div>

<script>
(() => {
  const MESSAGE = "Y'all are being too loud. Chill and bring it down.";
  const BASE_OFFSET = 100;          // dBFS -> rough dB SPL
  const MIN_DB = 30, MAX_DB = 110;
  const $ = id => document.getElementById(id);

  const store = {
    get(k, d){ try { const v = localStorage.getItem('noise.' + k); return v === null ? d : JSON.parse(v); } catch { return d; } },
    set(k, v){ try { localStorage.setItem('noise.' + k, JSON.stringify(v)); } catch {} }
  };

  const s = {
    threshold: store.get('threshold', 75),
    hold: store.get('hold', 2),
    cooldown: store.get('cooldown', 15),
    duration: store.get('duration', 6),
    calib: store.get('calib', 0),
    voice: store.get('voice', true),
    mic: store.get('mic', ''),
    voiceMode: store.get('voiceMode', 'tts'),
    voiceName: store.get('voiceName', ''),
    pitch: store.get('pitch', 1),
    rate: store.get('rate', 1)
  };

  // ---------- settings UI ----------
  const fmt = {
    threshold: v => v + ' dB', hold: v => Number(v).toFixed(1) + ' s',
    cooldown: v => v + ' s', duration: v => v + ' s',
    calib: v => (v >= 0 ? '+' : '') + v + ' dB',
    pitch: v => Number(v).toFixed(2) + '×', rate: v => Number(v).toFixed(2) + '×'
  };
  for (const k of Object.keys(fmt)) {
    const input = $(k), out = $(k + 'Out');
    input.value = s[k]; out.textContent = fmt[k](s[k]);
    input.addEventListener('input', () => {
      s[k] = Number(input.value); out.textContent = fmt[k](s[k]); store.set(k, s[k]);
      if (k === 'threshold') placeThreshold();
    });
  }
  const pct = db => Math.max(0, Math.min(100, (db - MIN_DB) / (MAX_DB - MIN_DB) * 100));
  function placeThreshold(){
    $('thresh').style.left = pct(s.threshold) + '%';
    $('threshLabel').textContent = 'Limit ' + s.threshold + ' dB';
    // keep the label on-screen near the right edge
    $('threshLabel').style.left = pct(s.threshold) > 80 ? 'auto' : '12px';
    $('threshLabel').style.right = pct(s.threshold) > 80 ? '12px' : 'auto';
  }
  placeThreshold();

  function renderVoice(){
    const b = $('voiceBtn');
    b.textContent = s.voice ? '🔊 Voice warning: ON' : '🔇 Voice warning: OFF (visual only)';
    b.className = s.voice ? 'voice-on' : 'voice-off';
    b.setAttribute('aria-pressed', String(s.voice));
  }
  renderVoice();
  $('voiceBtn').addEventListener('click', () => {
    s.voice = !s.voice; store.set('voice', s.voice); renderVoice();
    if (!s.voice && 'speechSynthesis' in window) speechSynthesis.cancel();
  });

  // ---------- audio ----------
  let ctx, analyser, stream, buf, running = false;
  let level = null, loudMs = 0, lastT = 0;
  let alertUntil = 0, quietUntil = 0, count = 0, wakeLock = null;

  async function listMics(){
    try {
      const devs = (await navigator.mediaDevices.enumerateDevices()).filter(d => d.kind === 'audioinput');
      const sel = $('micSelect');
      sel.innerHTML = '<option value="">Default microphone</option>' +
        devs.map((d, i) => `<option value="${d.deviceId}">${(d.label || 'Microphone ' + (i + 1)).replace(/</g,'&lt;')}</option>`).join('');
      sel.value = devs.some(d => d.deviceId === s.mic) ? s.mic : '';
    } catch {}
  }
  $('micSelect').addEventListener('change', async e => {
    s.mic = e.target.value; store.set('mic', s.mic);
    if (running) { stop(); await start(); }
  });

  async function start(){
    $('err').hidden = true;
    if (!navigator.mediaDevices?.getUserMedia) {
      return showError('This browser cannot use the microphone here. Open this file in Chrome or Edge on the Boxlight (or the computer connected to it).');
    }
    try {
      stream = await navigator.mediaDevices.getUserMedia({ audio: {
        deviceId: s.mic ? { exact: s.mic } : undefined,
        echoCancellation: false, noiseSuppression: false, autoGainControl: false } });
    } catch (e) {
      return showError(e.name === 'NotAllowedError'
        ? 'Microphone access was blocked. Click the lock or mic icon in the address bar, allow the microphone, then press Start again.'
        : 'No microphone was found. Check that the Boxlight mic (or a USB mic) is connected and turned on.');
    }
    ctx = ctx || new (window.AudioContext || window.webkitAudioContext)();
    await ctx.resume();
    const src = ctx.createMediaStreamSource(stream);
    analyser = ctx.createAnalyser();
    analyser.fftSize = 2048;
    buf = new Float32Array(analyser.fftSize);
    src.connect(analyser);
    running = true; level = null; loudMs = 0; lastT = performance.now();
    const track = stream.getAudioTracks()[0];
    $('micName').textContent = 'Microphone: ' + (track?.label || 'default');
    $('startBtn').textContent = 'Stop listening';
    $('startBtn').classList.remove('primary');
    listMics();
    try { wakeLock = await navigator.wakeLock?.request('screen'); } catch {}
    requestAnimationFrame(tick);
  }

  function stop(){
    running = false;
    stream?.getTracks().forEach(t => t.stop());
    $('startBtn').textContent = 'Start listening';
    $('startBtn').classList.add('primary');
    $('dbNum').textContent = '--';
    setStatus('idle', 'Not listening');
    $('fill').style.width = '0';
    $('holdBar').style.width = '0';
    try { wakeLock?.release(); } catch {}
  }

  $('startBtn').addEventListener('click', () => running ? stop() : start());

  function showError(msg){ $('err').textContent = msg; $('err').hidden = false; }
  function setStatus(cls, text){ const el = $('status'); el.className = 'status ' + cls; el.textContent = text; }

  function tick(t){
    if (!running) return;
    const dt = Math.min(250, t - lastT); lastT = t;
    analyser.getFloatTimeDomainData(buf);
    let sum = 0; for (let i = 0; i < buf.length; i++) sum += buf[i] * buf[i];
    const rms = Math.sqrt(sum / buf.length) || 1e-8;
    let db = 20 * Math.log10(rms) + BASE_OFFSET + s.calib;
    db = Math.max(MIN_DB - 5, Math.min(MAX_DB + 5, db));
    // fast rise, slow fall so the number is readable
    level = level === null ? db : (db > level ? level + (db - level) * 0.35 : level + (db - level) * 0.06);

    const shown = Math.round(level);
    $('dbNum').textContent = shown;
    $('fill').style.width = pct(level) + '%';
    const over = level >= s.threshold;
    const near = level >= s.threshold - 6;
    $('fill').style.backgroundColor = over ? 'var(--hot)' : near ? 'var(--warn)' : 'var(--ok)';

    const now = Date.now();
    const paused = now < alertUntil || now < quietUntil;
    if (!paused) {
      if (over) loudMs += dt; else loudMs = Math.max(0, loudMs - dt * 0.5);
      if (loudMs >= s.hold * 1000) { loudMs = 0; warn(); }
    } else loudMs = 0;
    $('holdBar').style.width = Math.min(100, loudMs / (s.hold * 1000) * 100) + '%';

    if (now < alertUntil) setStatus('hot', 'Too loud!');
    else if (now < quietUntil) setStatus('idle', 'Cooling down…');
    else if (over) setStatus('hot', 'Getting loud…');
    else if (near) setStatus('warn', 'Watch it');
    else setStatus('ok', 'Good volume');

    requestAnimationFrame(tick);
  }

  // ---------- warning ----------
  let hideTimer;
  function warn(){
    count++; $('count').textContent = count;
    const ms = s.duration * 1000;
    alertUntil = Date.now() + ms;
    quietUntil = alertUntil + s.cooldown * 1000;
    $('alert').hidden = false;
    clearTimeout(hideTimer);
    hideTimer = setTimeout(hideAlert, ms);
    if (s.voice) speak();
  }
  function hideAlert(){
    $('alert').hidden = true;
    alertUntil = Math.min(alertUntil, Date.now());
    if (!quietUntil || quietUntil < Date.now()) quietUntil = Date.now() + s.cooldown * 1000;
  }
  $('dismissBtn').addEventListener('click', () => { clearTimeout(hideTimer); hideAlert(); stopVoice(); });
  $('testBtn').addEventListener('click', () => { quietUntil = 0; warn(); });

  function chime(){
    try {
      ctx = ctx || new (window.AudioContext || window.webkitAudioContext)();
      ctx.resume();
      [0, 0.22].forEach((delay, i) => {
        const o = ctx.createOscillator(), g = ctx.createGain();
        o.type = 'sine'; o.frequency.value = i ? 660 : 880;
        g.gain.setValueAtTime(0.0001, ctx.currentTime + delay);
        g.gain.exponentialRampToValueAtTime(0.4, ctx.currentTime + delay + 0.02);
        g.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + delay + 0.2);
        o.connect(g).connect(ctx.destination);
        o.start(ctx.currentTime + delay); o.stop(ctx.currentTime + delay + 0.22);
      });
    } catch {}
  }
  let clipAudio = null;
  function stopVoice(){
    if ('speechSynthesis' in window) speechSynthesis.cancel();
    if (clipAudio) { clipAudio.pause(); clipAudio.currentTime = 0; }
  }
  function playClip(){
    if (!clipUrl) return false;
    stopVoice();
    clipAudio = new Audio(clipUrl);
    clipAudio.play().catch(() => {});
    return true;
  }
  function speak(){
    if (s.voiceMode === 'clip' && playClip()) return;
    chime();
    if (!('speechSynthesis' in window)) return;
    speechSynthesis.cancel();
    setTimeout(() => {
      const u = new SpeechSynthesisUtterance(MESSAGE);
      const voices = speechSynthesis.getVoices();
      const v = voices.find(v => v.name === s.voiceName)
        || voices.find(v => /^en(-|_)US/i.test(v.lang)) || voices.find(v => /^en/i.test(v.lang));
      if (v) u.voice = v;
      u.rate = s.rate; u.pitch = s.pitch; u.volume = 1;
      speechSynthesis.speak(u);
    }, 450);
  }

  // ---------- computer voice list ----------
  function listVoices(){
    if (!('speechSynthesis' in window)) return;
    const voices = speechSynthesis.getVoices().filter(v => /^en/i.test(v.lang));
    const sel = $('voiceName');
    sel.innerHTML = '<option value="">Default voice</option>' +
      voices.map(v => `<option>${v.name.replace(/</g,'&lt;')}</option>`).join('');
    sel.value = voices.some(v => v.name === s.voiceName) ? s.voiceName : '';
  }
  if ('speechSynthesis' in window) { listVoices(); speechSynthesis.onvoiceschanged = listVoices; }
  $('voiceName').addEventListener('change', e => { s.voiceName = e.target.value; store.set('voiceName', s.voiceName); });
  $('voiceMode').value = s.voiceMode;
  $('voiceMode').addEventListener('change', e => { s.voiceMode = e.target.value; store.set('voiceMode', s.voiceMode); });

  // ---------- custom recording (kept in this browser) ----------
  const idb = {
    open(){ return new Promise((res, rej) => { const r = indexedDB.open('noise-monitor', 1);
      r.onupgradeneeded = () => r.result.createObjectStore('kv');
      r.onsuccess = () => res(r.result); r.onerror = () => rej(r.error); }); },
    async run(mode, fn){ const db = await this.open(); return new Promise((res, rej) => {
      const q = fn(db.transaction('kv', mode).objectStore('kv')); q.onsuccess = () => res(q.result); q.onerror = () => rej(q.error); }); },
    get(k){ return this.run('readonly', st => st.get(k)); },
    set(k, v){ return this.run('readwrite', st => st.put(v, k)); },
    del(k){ return this.run('readwrite', st => st.delete(k)); }
  };
  let clipUrl = null, recorder = null, recTimer;

  function renderClip(){
    const has = !!clipUrl;
    $('playBtn').disabled = !has; $('delBtn').disabled = !has;
    $('voiceMode').querySelector('[value=clip]').disabled = !has;
    if (!has && s.voiceMode === 'clip') { s.voiceMode = 'tts'; $('voiceMode').value = 'tts'; }
    $('recStatus').textContent = has ? 'Recording saved on this device. Press Test warning to hear it.' : 'No recording yet. Press Record, say the warning, then press Stop.';
  }
  async function saveClip(blob){
    if (clipUrl) URL.revokeObjectURL(clipUrl);
    clipUrl = URL.createObjectURL(blob);
    try { await idb.set('clip', blob); } catch {}
    s.voiceMode = 'clip'; store.set('voiceMode', 'clip');
    renderClip(); $('voiceMode').value = 'clip';
  }
  (async () => { try { const b = await idb.get('clip'); if (b) clipUrl = URL.createObjectURL(b); } catch {} renderClip(); })();

  $('recBtn').addEventListener('click', async () => {
    if (recorder && recorder.state === 'recording') { recorder.stop(); return; }
    if (!window.MediaRecorder) return ($('recStatus').textContent = 'This browser cannot record. Upload an audio file instead.');
    let st;
    try { st = await navigator.mediaDevices.getUserMedia({ audio: s.mic ? { deviceId: { exact: s.mic } } : true }); }
    catch { $('recStatus').textContent = 'Microphone access was blocked, so recording could not start.'; return; }
    quietUntil = Date.now() + 60000;            // don't trigger warnings while recording
    const chunks = [];
    recorder = new MediaRecorder(st);
    recorder.ondataavailable = e => e.data.size && chunks.push(e.data);
    recorder.onstop = () => {
      clearTimeout(recTimer);
      st.getTracks().forEach(t => t.stop());
      $('recBtn').textContent = '● Record'; $('recBtn').classList.remove('recording');
      quietUntil = Date.now() + 2000;
      if (chunks.length) saveClip(new Blob(chunks, { type: recorder.mimeType || 'audio/webm' }));
    };
    recorder.start();
    $('recBtn').textContent = '■ Stop'; $('recBtn').classList.add('recording');
    $('recStatus').textContent = 'Recording… say the warning now (stops by itself after 10 seconds).';
    recTimer = setTimeout(() => recorder.state === 'recording' && recorder.stop(), 10000);
  });
  $('playBtn').addEventListener('click', playClip);
  $('delBtn').addEventListener('click', async () => {
    stopVoice();
    if (clipUrl) URL.revokeObjectURL(clipUrl);
    clipUrl = null; try { await idb.del('clip'); } catch {}
    renderClip();
  });
  $('clipFile').addEventListener('change', e => { const f = e.target.files[0]; if (f) saveClip(f); e.target.value = ''; });

  // ---------- full screen & keys ----------
  $('fsBtn').addEventListener('click', async () => {
    try { document.fullscreenElement ? await document.exitFullscreen() : await document.documentElement.requestFullscreen(); } catch {}
  });
  document.addEventListener('fullscreenchange', () => { $('fsBtn').textContent = document.fullscreenElement ? 'Exit full screen' : 'Full screen'; });
  document.addEventListener('visibilitychange', async () => {
    if (running && document.visibilityState === 'visible') { try { wakeLock = await navigator.wakeLock?.request('screen'); } catch {} }
  });
  document.addEventListener('keydown', e => {
    if (e.target.matches('input,select')) return;
    if (e.key === 'm' || e.key === 'M') $('voiceBtn').click();
    if (e.key === 'Escape' && !$('alert').hidden) $('dismissBtn').click();
  });
})();
</script>
</body>
</html>
