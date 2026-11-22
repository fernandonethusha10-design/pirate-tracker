<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pirate Token Tracker</title>
<style>
  * { margin:0; padding:0; box-sizing:border-box; }
  body { font-family:'Segoe UI',Arial,sans-serif; background:#0a1128; color:#e8e8f0; min-height:100vh; padding:20px; }
  .wrap { max-width:900px; margin:0 auto; }
  h1 { color:#ffd700; text-align:center; margin-bottom:6px; }
  .sub { text-align:center; color:#8899bb; margin-bottom:24px; font-size:.9rem; }
  .card { background:#131b3d; border:1px solid #253268; border-radius:12px; padding:20px; margin-bottom:20px; }
  .card h2 { color:#ffd700; font-size:1.1rem; margin-bottom:14px; }
  .grid { display:grid; grid-template-columns:repeat(auto-fit,minmax(140px,1fr)); gap:12px; }
  .stat { background:#0a1128; border-radius:10px; padding:14px; text-align:center; }
  .stat .num { font-size:1.6rem; font-weight:bold; color:#ffd700; }
  .stat .lbl { font-size:.75rem; color:#8899bb; margin-top:4px; }
  .bigbtns { display:flex; gap:10px; justify-content:center; flex-wrap:wrap; margin:16px 0 4px; }
  button { background:#ffd700; color:#0a1128; border:none; border-radius:8px; padding:14px 24px;
           font-size:1.1rem; font-weight:bold; cursor:pointer; }
  button:hover { background:#ffe75c; }
  button.ghost { background:#253268; color:#e8e8f0; }
  input, select { background:#0a1128; border:1px solid #253268; color:#e8e8f0;
           border-radius:8px; padding:10px; font-size:1rem; width:100%; }
  label { display:block; color:#8899bb; font-size:.8rem; margin:10px 0 4px; }
  .squadrow { display:flex; gap:8px; align-items:center; margin-bottom:8px; }
  .squadrow input { flex:1; }
  .squadrow button.ghost { padding:8px 14px; font-size:.85rem; }
  .log { font-size:.85rem; color:#aab4d4; max-height:180px; overflow-y:auto; }
  .log div { padding:4px 0; border-bottom:1px solid #1c2650; }
  .log span { color:#ffd700; }
  .bar { height:14px; background:#0a1128; border-radius:7px; overflow:hidden; margin-top:8px; }
  .bar > div { height:100%; background:linear-gradient(90deg,#ffd700,#ff9500); width:0%; transition:width .4s; }
  .note { color:#8899bb; font-size:.78rem; margin-top:10px; }
  .progress { font-size:.85rem; color:#8899bb; margin-top:6px; }
</style>
</head>
<body>
<div class="wrap">
  <h1>⚓ Pirate Token Tracker</h1>
  <p class="sub">Treasure Hunters · Chest drops end ~Dec 5, 2026 · Activate by Dec 15, 7 PM ET</p>

  <!-- CHEST COUNTER -->
  <div class="card">
    <h2>Chest-Run Tracker</h2>
    <div class="grid">
      <div class="stat"><div class="num" id="chests">0</div><div class="lbl">Chests opened</div></div>
      <div class="stat"><div class="num" id="chance">0.00%</div><div class="lbl">Chance so far</div></div>
      <div class="stat"><div class="num" id="matches">0</div><div class="lbl">Matches logged</div></div>
      <div class="stat"><div class="num" id="rate">1 / 137,500</div><div class="lbl">Current odds</div></div>
    </div>
    <div class="bigbtns">
      <button onclick="addChest(1)">+1 Chest</button>
      <button onclick="addChest(5)">+5</button>
      <button onclick="addChest(8)">+8 (avg match)</button>
      <button class="ghost" onclick="logMatch()">End match → log</button>
    </div>
    <label>Log match result</label>
    <div class="squadrow">
      <input id="matchChests" type="number" min="0" placeholder="Chests you opened this match">
      <button class="ghost" onclick="quickMatch()">Add</button>
    </div>
    <div class="log" id="log"></div>
    <p class="note">Check your real total in-game: Quests tab → Treasure Hunt section. Tap "Set total" to sync.</p>
    <div class="squadrow" style="margin-top:8px">
      <input id="setTotal" type="number" min="0" placeholder="Set total from Quests tab">
      <button class="ghost" onclick="setTotalFn()">Sync</button>
    </div>
  </div>

  <!-- SQUAD -->
  <div class="card">
    <h2>Squad Split (any teammate's find counts for everyone)</h2>
    <div id="squadList"></div>
    <div class="squadrow">
      <input id="mateName" placeholder="Teammate name">
      <button class="ghost" onclick="addMate()">+ Add teammate</button>
    </div>
    <div class="grid" style="margin-top:12px">
      <div class="stat"><div class="num" id="squadChests">0</div><div class="lbl">Team chests</div></div>
      <div class="stat"><div class="num" id="squadChance">0.00%</div><div class="lbl">Team chance so far</div></div>
    </div>
    <p class="note">Fill teammates count too — a random fill finding it means you get it.</p>
  </div>

  <!-- TREASURE HOURS -->
  <div class="card">
    <h2>Treasure Hours (Saturdays — token boosted 1.5×–2.0×)</h2>
    <label>Next Treasure Hour start (ET 24h, e.g. 14:00)</label>
    <div class="squadrow">
      <select id="boost"><option value="1.5">1.5× (~1 in 91,667)</option><option value="2">2.0× (~1 in 68,750)</option></select>
      <input id="hourDate" type="date">
      <input id="hourTime" type="time">
      <button class="ghost" onclick="setHour()">Set</button>
    </div>
    <div class="stat" style="margin-top:12px"><div class="num" id="countdown">—</div><div class="lbl">Until next Treasure Hour</div></div>
    <p class="note">Last confirmed: Sat Nov 21, 2 PM & 9 PM ET. Epic announces new times each week — update them here. Regular-day chests still count, so play when you can.</p>
  </div>
</div>

<script>
let state = JSON.parse(localStorage.getItem('pt_state') || '{}');
if (!state.squad) state = { chests:0, matches:[], squad:[], hour:null, boost:1.5 };
const BASE = 137500;

const save = () => localStorage.setItem('pt_state', JSON.stringify(state));
const $ = id => document.getElementById(id);
const chance = (chests, rate) => (1 - Math.pow(1 - 1/rate, chests)) * 100;

function render() {
  const rate = state.inHour ? Math.round(BASE / state.boost) : BASE;
  $('chests').textContent = state.chests;
  $('chance').textContent = chance(state.chests, rate).toFixed(2) + '%';
  $('matches').textContent = state.matches.length;
  $('rate').textContent = '1 / ' + rate.toLocaleString();
  const team = state.squad.reduce((s,m) => s + m.chests, 0) + state.chests;
  $('squadChests').textContent = team;
  $('squadChance').textContent = chance(team, rate).toFixed(2) + '%';
  $('log').innerHTML = state.matches.slice(-20).reverse()
    .map(m => `<div>${m.date}: <span>${m.chests}</span> chests</div>`).join('');
  $('squadList').innerHTML = state.squad.map((m,i) =>
    `<div class="squadrow"><input value="${m.name}" onchange="state.squad[${i}].name=this.value;save();render()">
     <input type="number" min="0" value="${m.chests}" onchange="state.squad[${i}].chests=+this.value||0;save();render()">
     <button class="ghost" onclick="state.squad.splice(${i},1);save();render()">✕</button></div>`).join('');
}

function addChest(n) { state.chests += n; save(); render(); }
function logMatch() {
  state.matches.push({ date: new Date().toLocaleString(), chests: 0 });
  save(); render();
}
function quickMatch() {
  const n = +$('matchChests').value || 0;
  state.chests += n;
  state.matches.push({ date: new Date().toLocaleString(), chests: n });
  $('matchChests').value = '';
  save(); render();
}
function setTotalFn() {
  const v = +$('setTotal').value;
  if (v >= 0) { state.chests = v; save(); render(); }
}
function addMate() {
  const n = $('mateName').value.trim();
  if (n) { state.squad.push({ name:n, chests:0 }); $('mateName').value=''; save(); render(); }
}
function setHour() {
  state.hour = $('hourDate').value + 'T' + ($('hourTime').value || '14:00') + ':00-05:00';
  state.boost = parseFloat($('boost').value);
  save(); tick();
}
function tick() {
  if (state.hour) {
    const ms = new Date(state.hour) - Date.now();
    state.inHour = ms <= 0 && ms > -2*3600e3;
    if (ms > 0) {
      const h = Math.floor(ms/3600e3), m = Math.floor(ms%3600e3/60e3), s = Math.floor(ms%60e3/1e3);
      $('countdown').textContent = `${h}h ${m}m ${s}s`;
    } else if (state.inHour) {
      $('countdown').textContent = 'LIVE — boosted rate active!';
    } else $('countdown').textContent = '— set next Saturday';
  }
  render(); setTimeout(tick, 1000);
}
$('hourDate').value = state.hour ? state.hour.split('T')[0] : '';
$('hourTime').value = state.hour ? state.hour.split('T')[1].slice(0,5) : '';
tick();
</script>
</body>
</html>
