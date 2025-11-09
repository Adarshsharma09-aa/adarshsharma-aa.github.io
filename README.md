<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Adarsh Alpha-Strategy — Final Simulator (INR, Stacked)</title>
<style>
  :root{
    --bg-1:#020617; --bg-2:#07122b;
    --muted:#98aebb; --card:rgba(255,255,255,0.03);
    --cyan:#06b6d4; --violet:#7c3aed;
    --green:#22c55e; --red:#ff5252;
    --glass:rgba(255,255,255,0.02);
  }
  *{box-sizing:border-box;font-family:Inter,ui-sans-serif,system-ui,Segoe UI,Roboto,"Helvetica Neue",Arial}
  html,body{height:100%;margin:0;background:linear-gradient(160deg,var(--bg-1),var(--bg-2));color:#e7f0fb}
  .app{max-width:960px;margin:14px auto;padding:14px}
  header{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:12px}
  .brand{display:flex;gap:12px;align-items:center}
  .logo{width:56px;height:56px;border-radius:12px;background:linear-gradient(135deg,var(--cyan),var(--violet));display:flex;align-items:center;justify-content:center;font-weight:900;color:#041022}
  .title{font-size:18px;font-weight:900}
  .subtitle{font-size:13px;color:var(--muted);margin-top:3px}
  .status-row{display:flex;gap:8px;align-items:center}
  .pill{font-size:12px;padding:6px 10px;border-radius:10px;background:var(--glass);color:var(--muted)}
  .sim-pill{background:linear-gradient(90deg,#b8f5c8,#86efac);color:#022;font-weight:800}

  /* stacked layout — mobile-first */
  .stack{display:flex;flex-direction:column;gap:14px}
  .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:14px;border-radius:12px;border:1px solid rgba(255,255,255,0.03);box-shadow:0 10px 34px rgba(2,6,23,0.6)}
  .balance{font-size:36px;font-weight:900;text-shadow:0 6px 18px rgba(0,0,0,0.35); /* color set dynamically */ }
  .small{font-size:13px;color:var(--muted)}
  .controls{display:flex;gap:10px;align-items:center;margin-top:12px;flex-wrap:wrap}
  button{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.04);padding:8px 12px;border-radius:10px;color:inherit;cursor:pointer}
  button.primary{background:linear-gradient(90deg,var(--cyan),var(--violet));border:none;color:#031125;font-weight:900}
  .spark-wrap{background:linear-gradient(180deg, rgba(6,182,212,0.02), rgba(124,58,237,0.02));padding:8px;border-radius:10px}
  canvas{width:100%;height:110px;display:block;border-radius:6px}
  .history{display:flex;flex-direction:column;gap:8px;max-height:260px;overflow:auto;margin-top:8px}
  .row{display:flex;justify-content:space-between;align-items:center;padding:6px 0;border-bottom:1px solid rgba(255,255,255,0.015);padding-top:4px} /* added padding-top */
  .metrics{display:flex;gap:10px;flex-wrap:wrap;margin-top:10px}
  .metric{min-width:120px;padding:10px;border-radius:10px;background:rgba(255,255,255,0.01);border:1px solid rgba(255,255,255,0.02)}
  .holdings-table{width:100%;border-collapse:collapse;margin-top:8px}
  .holdings-table th,.holdings-table td{padding:8px;text-align:left;font-size:13px;border-bottom:1px dashed rgba(255,255,255,0.03)}
  /* Normalized numeric inputs for holdings */
  .holdings-table input.qty, .holdings-table input.cost{
    width:80px; /* specific width for numeric fields */
    text-align:right; /* align numbers to the right */
    font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, "Roboto Mono", "Courier New", monospace;
    background: rgba(255,255,255,0.01);
    color:inherit;
    border:1px solid rgba(255,255,255,0.03);
    padding:6px;border-radius:6px;
  }
  .alloc-canvas{width:100%;height:220px}
  .settings-row{display:flex;gap:8px;align-items:center;margin-top:8px}
  label{font-size:13px;color:var(--muted);display:block;margin-top:8px}
  input[type="number"], select{width:100%;padding:8px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:inherit}
  input[type="range"]{width:100%}
  .footer-note{font-size:12px;color:var(--muted);margin-top:10px}
  @media(min-width:860px){
    /* wider screens: two-column stacked appearance but still vertical order preserved */
    .stack{max-width:900px;margin:0 auto}
  }
</style>
</head>
<body>
  <div class="app">
    <header>
      <div class="brand">
        <div class="logo">AA</div>
        <div>
          <div class="title">Adarsh Alpha-Strategy Fund</div>
          <div class="subtitle">Final Polished Simulator • IN₹ default • Stacked View</div>
        </div>
      </div>
      <div class="status-row">
        <div class="pill sim-pill">SIMULATION</div>
        <div class="pill" id="marketStatusPill">Market: Open</div>
        <div class="pill" id="userPill">Owner: Adarsh Sharma</div>
      </div>
    </header>

    <main class="stack">
      <!-- Chart Section (top) -->
      <section class="card" id="chartCard">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <div>
            <div class="small">Total Net Asset Value (NAV)</div>
            <div id="balance" class="balance">₹ 1,000,000</div>
            <div id="subline" class="small" style="margin-top:6px">Mark-to-Market • Live simulated feed</div>
          </div>
          <div style="text-align:right">
            <div class="small">Total Return (YTD)</div>
            <div id="pl" class="small" style="font-weight:900;color:var(--green)">+₹0.00 (0.00%)</div>
            <div class="small" style="margin-top:6px;color:var(--muted)">Reporting Period: YTD (sim)</div>
          </div>
        </div>

        <div class="spark-wrap" style="margin-top:10px;">
          <canvas id="spark"></canvas>
        </div>

        <div class="controls">
          <button id="pauseBtn">Pause</button>
          <button id="tradeBtn">Execute Test Trade</button>
          <button class="primary" id="resetBtn">Reset</button>
          <div style="margin-left:auto" class="small" id="statusText">Status: Simulation running</div>
        </div>
      </section>

      <!-- Holdings & Allocation -->
      <section class="card" id="holdingsCard">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <div class="small" style="font-weight:800">Holdings (editable)</div>
          <div class="small">Allocation & Exposure</div>
        </div>

        <div style="margin-top:8px;display:flex;gap:12px;flex-direction:column">
          <table class="holdings-table" id="holdingsTable">
            <thead><tr><th>Asset</th><th>Ticker</th><th>Qty</th><th>Cost (₹)</th><th>Value (₹)</th><th>%Port</th></tr></thead>
            <tbody></tbody>
          </table>

          <div>
            <canvas id="allocCanvas" class="alloc-canvas"></canvas>
          </div>
        </div>

        <div style="margin-top:12px">
          <div class="small">Recent Transaction Log</div>
          <div class="history" id="historyList" aria-live="polite"></div>
        </div>
      </section>

      <!-- Metrics & Settings -->
      <section class="card" id="metricsCard">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <div class="small">Strategy Metrics</div>
          <div class="pill">Simulated</div>
        </div>

        <div class="metrics" id="metricsWrap">
          <div class="metric">
            <div class="small">Sharpe Ratio (1Y)</div>
            <div id="sharpe" style="font-weight:900">1.42</div>
          </div>
          <div class="metric">
            <div class="small">Max Drawdown</div>
            <div id="mdd" style="font-weight:900;color:var(--red)">-5.60%</div>
          </div>
          <div class="metric">
            <div class="small">Alpha</div>
            <div id="alpha" style="font-weight:900;color:var(--green)">+0.84%</div>
          </div>
          <div class="metric">
            <div class="small">Beta</div>
            <div id="beta" style="font-weight:900">1.12</div>
          </div>
        </div>

        <!-- settings -->
        <div style="margin-top:14px">
          <div class="small" style="font-weight:800">Settings (persistent)</div>

          <label>Starting NAV (₹)</label>
          <input type="number" id="startNavInput" min="100" value="1000000" />

          <label>Update Frequency (ms)</label>
          <input type="range" id="freqRange" min="400" max="4000" step="50" value="950" />
          <div class="small" id="freqVal">950 ms</div>

          <label>Profit Bias (0-1)</label>
          <input type="range" id="biasRange" min="0" max="1" step="0.01" value="0.62" />
          <div class="small" id="biasVal">0.62</div>

          <label>Volatility (scale)</label>
          <input type="range" id="volRange" min="0.00005" max="0.003" step="0.00005" value="0.0006" />
          <div class="small" id="volVal">0.00060</div>

          <label>Currency</label>
          <select id="currencySelect">
            <option value="INR" selected>INR (₹)</option>
            <option value="USD">USD ($)</option>
            <option value="EUR">EUR (€)</option>
          </select>

          <label>Portfolio Owner</label>
          <input type="text" id="ownerInput" value="Adarsh Sharma" />

          <div class="settings-row">
            <button id="marketToggle" class="small-btn">Toggle Market Open/Closed</button>
            <div class="small" id="marketText" style="margin-left:auto">Open</div>
          </div>

          <div class="footer-note">Settings and holdings persist in your browser via <code>localStorage</code>.</div>
        </div>

      </section>
    </main>
  </div>

<script>
/* Final Polished Simulator
   - Default currency: INR (₹)
   - Stacked view (chart on top)
   - Persistent settings & holdings in localStorage
   - Holdings prices update only on heartbeat (no jitter)
   - Inputs in holdings are monospace and fixed width
   - Dynamic balance color matching daily P/L
   - Transaction logs mention asset tickers
*/

// ---------- Config & State ----------
const STORAGE_KEY = 'adarsh_alpha_sim_v2';
let state = {
  START_BALANCE: 1000000,
  initialBaseline: 888800,
  balance: 1000000,
  profitBias: 0.62,
  volatility: 0.0006,
  STEP_MS: 950,
  currency: 'INR',
  marketOpen: true,
  owner: 'Adarsh Sharma',
  holdings: [
    { name: 'Apple Inc.', ticker: 'AAPL', qty: 1200, cost: 120.00, class: 'Equity', currentPrice: 120.00 },
    { name: 'Microsoft', ticker: 'MSFT', qty: 700, cost: 210.00, class: 'Equity', currentPrice: 210.00 },
    { name: 'NVIDIA', ticker: 'NVDA', qty: 400, cost: 300.00, class: 'Equity', currentPrice: 300.00 },
    { name: 'Amazon', ticker: 'AMZN', qty: 180, cost: 160.00, class: 'Equity', currentPrice: 160.00 },
    { name: 'US Treasury', ticker: 'UST10Y', qty: 5000, cost: 1.00, class: 'Fixed Income', currentPrice: 1.00 },
    { name: 'Cash', ticker: 'CASH', qty: 1, cost: 50000.00, class: 'Cash', currentPrice: 50000.00 }
  ],
  history: [],
  lastTX: 9001
};

// ---------- DOM Refs ----------
const balanceEl = document.getElementById('balance');
const plEl = document.getElementById('pl');
const sublineEl = document.getElementById('subline');
const startNavInput = document.getElementById('startNavInput');
const freqRange = document.getElementById('freqRange');
const freqVal = document.getElementById('freqVal');
const biasRange = document.getElementById('biasRange');
const biasVal = document.getElementById('biasVal');
const volRange = document.getElementById('volRange');
const volVal = document.getElementById('volVal');
const currencySelect = document.getElementById('currencySelect');
const ownerInput = document.getElementById('ownerInput');
const marketToggle = document.getElementById('marketToggle');
const marketText = document.getElementById('marketText');
const marketStatusPill = document.getElementById('marketStatusPill');
const userPill = document.getElementById('userPill');

const spark = document.getElementById('spark'); const ctx = spark.getContext('2d');
const allocCanvas = document.getElementById('allocCanvas'); const allocCtx = allocCanvas.getContext('2d');

const holdingsTableBody = document.querySelector('#holdingsTable tbody');
const historyList = document.getElementById('historyList');

const pauseBtn = document.getElementById('pauseBtn');
const tradeBtn = document.getElementById('tradeBtn');
const resetBtn = document.getElementById('resetBtn');
const statusText = document.getElementById('statusText');

const sharpeEl = document.getElementById('sharpe');
const mddEl = document.getElementById('mdd');
const alphaEl = document.getElementById('alpha');
const betaEl = document.getElementById('beta');

let displayed = state.balance;
let paused = false;
let dpr = window.devicePixelRatio || 1;

// ---------- Persistence ----------
function saveState(){
  try { localStorage.setItem(STORAGE_KEY, JSON.stringify(state)); } catch(e){ console.warn('Save failed', e); }
}
function loadState(){
  try{
    const raw = localStorage.getItem(STORAGE_KEY);
    if(!raw) return;
    const s = JSON.parse(raw);
    // merge safely
    state = Object.assign(state, s);
    // ensure holdings have currentPrice
    state.holdings = (s.holdings || state.holdings).map(h => Object.assign({ currentPrice: h.cost }, h));
  } catch(e){ console.warn('Load failed', e); }
}

// ---------- Formatting ----------
function moneyFmt(v){
  if(isNaN(v)) v = 0;
  if(state.currency === 'INR') return new Intl.NumberFormat('en-IN',{style:'currency',currency:'INR',maximumFractionDigits:2}).format(v);
  if(state.currency === 'USD') return new Intl.NumberFormat('en-US',{style:'currency',currency:'USD',maximumFractionDigits:2}).format(v);
  if(state.currency === 'EUR') return new Intl.NumberFormat('de-DE',{style:'currency',currency:'EUR',maximumFractionDigits:2}).format(v);
  return v.toFixed(2);
}

// ---------- Canvas resizing ----------
function resizeCanvases(){
  const w = spark.clientWidth, h = spark.clientHeight;
  spark.width = Math.floor(w * dpr); spark.height = Math.floor(h * dpr); spark.style.width = w + 'px'; spark.style.height = h + 'px';
  ctx.setTransform(dpr,0,0,dpr,0,0);

  const aw = allocCanvas.clientWidth, ah = allocCanvas.clientHeight;
  allocCanvas.width = Math.floor(aw * dpr); allocCanvas.height = Math.floor(ah * dpr); allocCanvas.style.width = aw + 'px'; allocCanvas.style.height = ah + 'px';
  allocCtx.setTransform(dpr,0,0,dpr,0,0);
}
window.addEventListener('resize', ()=>{ dpr = window.devicePixelRatio || 1; resizeCanvases(); drawSpark(); drawAlloc(); });

// ---------- Seed history (on new run) ----------
function seedHistory(){
  if(state.history && state.history.length > 0) return;
  let v = state.START_BALANCE * 0.95;
  for(let i=0;i<120;i++){
    const trend = 1 + 0.00012 * (i/120) * state.profitBias;
    const noise = (Math.random()*2 -1) * state.volatility * state.START_BALANCE * 0.6;
    v = Math.max(100, v*trend + noise);
    state.history.push(v);
  }
}

// ---------- Holdings valuation utilities ----------
function calcHoldingsMarketValues(){
  // Uses stable currentPrice stored on each holding (updated only in stepSim)
  const values = []; let total = 0;
  for(const h of state.holdings){
    // Ensure qty/cost/currentPrice non-zero
    h.qty = Math.max(0, Math.floor(Number(h.qty) || 0));
    h.cost = Math.max(0.01, Number(h.cost) || 0.01);
    h.currentPrice = Math.max(0.01, Number(h.currentPrice) || h.cost);
    const mv = h.currentPrice * h.qty;
    values.push({ h, price: h.currentPrice, mv });
    total += mv;
  }
  return { values, total };
}

// ---------- Render holdings table (inputs aligned) ----------
function renderHoldings(){
  const { values, total } = calcHoldingsMarketValues();
  holdingsTableBody.innerHTML = '';
  for(const it of values){
    const h = it.h;
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${h.name}</td>
      <td>${h.ticker}</td>
      <td><input data-ticker="${h.ticker}" class="qty" type="number" min="0" value="${h.qty}"></td>
      <td><input data-ticker="${h.ticker}" class="cost" type="number" min="0.01" step="0.01" value="${h.cost.toFixed(2)}"></td>
      <td>${moneyFmt(it.mv)}</td>
      <td>${ total>0 ? ((it.mv/total)*100).toFixed(2) + '%' : '0.00%' }</td>
    `;
    holdingsTableBody.appendChild(tr);
  }

  // attach listeners
  holdingsTableBody.querySelectorAll('input.qty').forEach(inp=>{
    inp.addEventListener('input', e=>{
      const t = e.target.dataset.ticker;
      const h = state.holdings.find(x=>x.ticker === t);
      h.qty = Math.max(0, Math.floor(Number(e.target.value) || 0));
      saveState();
      // reflect changes immediately
      renderHoldings();
      drawAlloc();
      drawSpark();
    });
  });
  holdingsTableBody.querySelectorAll('input.cost').forEach(inp=>{
    inp.addEventListener('input', e=>{
      const t = e.target.dataset.ticker;
      const h = state.holdings.find(x=>x.ticker === t);
      h.cost = Math.max(0.01, Number(e.target.value) || 0.01);
      // optionally set currentPrice to cost if price was missing
      h.currentPrice = Math.max(0.01, h.currentPrice || h.cost);
      saveState();
      renderHoldings();
      drawAlloc();
      drawSpark();
    });
  });
}

// ---------- Allocation pie ----------
function drawAlloc(){
  const { values, total } = calcHoldingsMarketValues();
  const byClass = {};
  for(const it of values){
    const cls = it.h.class || 'Other';
    byClass[cls] = (byClass[cls]||0) + it.mv;
  }
  const classes = Object.keys(byClass);
  allocCtx.clearRect(0,0,allocCanvas.width,allocCanvas.height);
  if(classes.length === 0) return;
  const w = allocCanvas.clientWidth; const h = allocCanvas.clientHeight;
  const cx = w/2, cy = h/2, r = Math.min(w,h)/2 - 10;
  let startAng = -Math.PI/2;
  const palette = ['#22c55e','#06b6d4','#7c3aed','#f59e0b','#ef4444'];
  let i=0;
  const sumAll = classes.reduce((s,c)=>s+byClass[c],0);
  for(const c of classes){
    const val = byClass[c];
    const angle = (val / sumAll) * Math.PI*2;
    allocCtx.beginPath();
    allocCtx.moveTo(cx,cy);
    allocCtx.arc(cx,cy,r,startAng,startAng+angle);
    allocCtx.closePath();
    allocCtx.fillStyle = palette[i % palette.length];
    allocCtx.fill();
    // label
    const mid = startAng + angle/2;
    const lx = cx + Math.cos(mid)*(r*0.6);
    const ly = cy + Math.sin(mid)*(r*0.6);
    allocCtx.fillStyle = 'rgba(255,255,255,0.85)';
    allocCtx.font = '12px Inter';
    allocCtx.fillText(c, lx-10, ly);
    startAng += angle; i++;
  }
}

// ---------- Sparkline ----------
function drawSpark(){
  const w = spark.clientWidth; const h = spark.clientHeight;
  ctx.clearRect(0,0,w,h);
  if(state.history.length < 2) return;
  const values = state.history.slice(-120);
  const min = Math.min(...values);
  const max = Math.max(...values);
  const range = Math.max(1, max - min);

  const grad = ctx.createLinearGradient(0,0,w,0);
  grad.addColorStop(0,'#22c55e'); grad.addColorStop(0.5,'#06b6d4'); grad.addColorStop(1,'#7c3aed');

  ctx.lineWidth = 2.2; ctx.strokeStyle = grad; ctx.beginPath();
  values.forEach((v,i)=>{
    const x = (i/(values.length-1))*(w-12)+6;
    const y = (1 - (v - min)/range)*(h-12)+6;
    if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
  });
  ctx.stroke();
  ctx.lineTo(w-6,h-6); ctx.lineTo(6,h-6); ctx.closePath();
  ctx.fillStyle = 'rgba(6,182,212,0.04)'; ctx.fill();
  // end dot
  const last = values[values.length-1];
  const lx = w-6; const ly = (1 - (last - min)/range)*(h-12)+6;
  ctx.beginPath(); ctx.fillStyle = 'rgba(34,197,94,0.9)'; ctx.arc(lx,ly,4,0,Math.PI*2); ctx.fill();
}

// ---------- Metrics (update only in stepSim) ----------
function calcMetrics(){
  const vals = state.history.slice(-80);
  if(vals.length < 5) return;
  const returns = [];
  for(let i=1;i<vals.length;i++) returns.push((vals[i]-vals[i-1])/vals[i-1]);
  const mean = returns.reduce((a,b)=>a+b,0)/returns.length;
  const std = Math.sqrt(returns.reduce((a,b)=>a+(b-mean)**2,0)/returns.length);
  const sharpe = std === 0 ? 0 : (mean / std) * Math.sqrt(252);
  // MDD
  let peak = vals[0], mdd = 0;
  for(const v of vals){
    if(v > peak) peak = v;
    const dd = (v - peak) / peak;
    if(dd < mdd) mdd = dd;
  }
  const alpha = (mean * 252 * 100).toFixed(2);
  const beta = (std * Math.sqrt(252) * 1.05).toFixed(2);
  sharpeEl.textContent = sharpe ? sharpe.toFixed(2) : '0.00';
  mddEl.textContent = (mdd*100).toFixed(2) + '%';
  alphaEl.textContent = (alpha>0 ? '+'+alpha : alpha) + '%';
  betaEl.textContent = beta;
  saveState();
}

// ---------- Transaction log ----------
function logTx(text, amount, positive, ticker){
  const row = document.createElement('div'); row.className = 'row';
  const left = document.createElement('div');
  left.innerHTML = `<div style="font-weight:800;color:${positive?'var(--green)':'var(--red)'}">${text}${ticker?(' • '+ticker):''}</div>
                    <div class="small" style="color:var(--muted)">TX:${state.lastTX++} • ${new Date().toLocaleTimeString()}</div>`;
  const right = document.createElement('div');
  right.innerHTML = `<div style="font-weight:800;color:${positive?'var(--green)':'var(--red)'}">${positive?'+':'-'}${moneyFmt(Math.abs(amount))}</div>
                     <div class="small" style="color:var(--muted)">Impact</div>`;
  row.appendChild(left); row.appendChild(right);
  historyList.prepend(row);
  while(historyList.children.length > 12) historyList.removeChild(historyList.lastChild);
  saveState();
}

// ---------- Simulation step (heartbeat) ----------
// Important: update individual holdings' currentPrice ONLY here to avoid jitter
function randStdNormal(){
  let u=0,v=0; while(u===0) u=Math.random(); while(v===0) v=Math.random();
  return Math.sqrt(-2.0*Math.log(u)) * Math.cos(2*Math.PI*v);
}

function stepSim(){
  if(!state.marketOpen || paused) return;
  // drift and noise
  const drift = (state.profitBias - 0.5) * 0.0015;
  const rand = randStdNormal();
  const noise = rand * state.volatility * (1 + Math.random()*0.6);
  // occasional shock
  let shock = 0;
  if(Math.random() < 0.045) shock = -Math.abs(rand) * 0.007 * (Math.random()*1.2 + 0.2);
  const pct = drift + noise + shock;
  const prev = state.balance;
  state.balance = Math.max(100, state.balance * (1 + pct));

  // Update holdings prices once per step using small idiosyncratic moves plus global pct
  const globalPct = (state.balance / state.START_BALANCE) - 1;
  for(const h of state.holdings){
    // idiosyncratic change small, but stable across frames (applied per heartbeat)
    const idio = (Math.random() * 2 - 1) * 0.008; // up to ±0.8%
    const newPrice = Math.max(0.01, h.currentPrice * (1 + globalPct*0.03 + idio * 0.2)); // mild link to global pct
    // Smoothly move currentPrice toward newPrice to avoid huge jumps
    h.currentPrice = h.currentPrice + (newPrice - h.currentPrice) * 0.35;
  }

  // push to history
  state.history.push(state.balance); if(state.history.length > 400) state.history.shift();

  // log transaction for significant moves
  const impact = state.balance - prev;
  if(Math.abs(impact) > state.START_BALANCE * 0.003){
    // pick a random holding to attribute transaction
    const rnd = state.holdings[Math.floor(Math.random() * state.holdings.length)];
    const type = impact > 0 ? 'BUY' : 'SELL';
    logTx(`${type} simulated adjustment`, impact, impact > 0, rnd.ticker);
  }
  // occasional small loss log
  if(Math.random() < 0.02 && pct < 0) {
    const rnd = state.holdings[Math.floor(Math.random() * state.holdings.length)];
    logTx('Market micro-loss observed', impact, false, rnd.ticker);
  }

  // update metrics after the step
  calcMetrics();
  saveState();
}

// ---------- Main loop (rAF + accumulator to maintain STEP_MS cadence) ----------
let lastFrame = performance.now();
let accumulator = 0;
function loop(now){
  const delta = now - lastFrame; lastFrame = now;
  accumulator += delta;
  if(accumulator >= state.STEP_MS){
    const steps = Math.floor(accumulator / state.STEP_MS);
    for(let i=0;i<steps;i++) stepSim();
    accumulator -= steps * state.STEP_MS;
    drawSpark(); drawAlloc(); renderHoldings();
  }
  updateDOM(); // smooth animations & dynamic color
  requestAnimationFrame(loop);
}

// ---------- DOM update (smooth balance, dynamic color) ----------
let lastBalText = '', lastPlText = '';
function updateDOM(){
  // smooth displayed NAV number
  displayed += (state.balance - displayed) * 0.12;
  const balText = moneyFmt(displayed);
  if(balText !== lastBalText){
    balanceEl.textContent = balText;
    lastBalText = balText;
  }
  // P/L vs initialBaseline
  const change = displayed - state.initialBaseline;
  const pct = (change / state.initialBaseline) * 100;
  const sign = change >= 0 ? '+' : '';
  const plText = `${sign}${moneyFmt(change)} (${sign}${pct.toFixed(2)}%)`;
  if(plText !== lastPlText){
    plEl.textContent = plText;
    // dynamic coloring of both P/L and balance
    plEl.style.color = change >= 0 ? 'var(--green)' : 'var(--red)';
    balanceEl.style.color = change >= 0 ? 'var(--green)' : 'var(--red)';
    lastPlText = plText;
  }
  // update owner & market pill
  userPill.textContent = `Owner: ${state.owner}`;
  marketStatusPill.textContent = `Market: ${state.marketOpen ? 'Open' : 'Closed'}`;
}

// ---------- Controls wiring & persistence effects ----------
freqRange.addEventListener('input', e=>{
  state.STEP_MS = Number(e.target.value); freqVal.textContent = state.STEP_MS + ' ms'; saveState();
});
biasRange.addEventListener('input', e=>{ state.profitBias = Number(e.target.value); biasVal.textContent = state.profitBias.toFixed(2); saveState(); });
volRange.addEventListener('input', e=>{ state.volatility = Number(e.target.value); volVal.textContent = state.volatility.toFixed(5); saveState(); });

startNavInput.addEventListener('change', e=>{
  state.START_BALANCE = Math.max(100, Number(e.target.value) || 100);
  state.balance = state.START_BALANCE;
  state.initialBaseline = Math.round(state.START_BALANCE * 0.8888);
  displayed = state.balance;
  saveState();
  renderHoldings(); drawAlloc(); drawSpark();
});

currencySelect.addEventListener('change', e=>{ state.currency = e.target.value; saveState(); renderHoldings(); drawAlloc(); drawSpark(); updateDOM(); });
ownerInput.addEventListener('change', e=>{ state.owner = e.target.value || 'Adarsh Sharma'; saveState(); });

marketToggle.addEventListener('click', ()=>{
  state.marketOpen = !state.marketOpen;
  marketText.textContent = state.marketOpen ? 'Open' : 'Closed';
  statusText.textContent = state.marketOpen ? 'Status: Simulation running' : 'Status: Market Closed';
  saveState();
});

pauseBtn.addEventListener('click', ()=>{
  paused = !paused;
  pauseBtn.textContent = paused ? 'Resume' : 'Pause';
  statusText.textContent = paused ? 'Status: Paused' : 'Status: Simulation running';
});

tradeBtn.addEventListener('click', ()=>{
  if(!state.marketOpen || paused) return;
  // manual positive injection, attribute to a random holding
  const prev = state.balance;
  const gain = state.balance * (0.001 + Math.random()*0.002); // 0.1%-0.3%
  state.balance = Math.max(100, state.balance + gain);
  // attribute to random holding
  const rnd = state.holdings[Math.floor(Math.random() * state.holdings.length)];
  // also nudge that holding price
  rnd.currentPrice = Math.max(0.01, rnd.currentPrice * (1 + 0.001 + Math.random()*0.002));
  state.history.push(state.balance); if(state.history.length > 400) state.history.shift();
  logTx(`Manual test BUY executed`, gain, true, rnd.ticker);
  drawSpark(); drawAlloc(); renderHoldings(); calcMetrics(); saveState();
});

resetBtn.addEventListener('click', ()=>{
  if(!confirm('Reset to starting NAV and clear history & logs?')) return;
  state.balance = state.START_BALANCE;
  displayed = state.balance;
  state.initialBaseline = Math.round(state.START_BALANCE * 0.8888);
  state.history = [];
  state.lastTX = 9001;
  state.holdings.forEach(h => { h.currentPrice = h.cost; });
  historyList.innerHTML = '';
  seedHistory(); saveState(); renderHoldings(); drawAlloc(); drawSpark(); calcMetrics();
  statusText.textContent = 'Status: Reset';
});

// ---------- Load / Initial setup ----------
loadState();
seedHistory();
resizeCanvases();
renderHoldings();
drawAlloc();
drawSpark();
calcMetrics();

// Initialize UI elements from state
startNavInput.value = state.START_BALANCE;
freqRange.value = state.STEP_MS; freqVal.textContent = state.STEP_MS + ' ms';
biasRange.value = state.profitBias; biasVal.textContent = state.profitBias.toFixed(2);
volRange.value = state.volatility; volVal.textContent = state.volatility.toFixed(5);
currencySelect.value = state.currency;
ownerInput.value = state.owner;
marketText.textContent = state.marketOpen ? 'Open' : 'Closed';

// Start the loop
requestAnimationFrame(loop);

// Persist state periodically (in case)
setInterval(()=>saveState(), 5000);
</script>
</body>
</html>
