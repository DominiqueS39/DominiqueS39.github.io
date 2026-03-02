<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>HDB BTO Simulator</title>
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=DM+Sans:wght@300;400;500;600&display=swap" rel="stylesheet">
<style>
  :root {
    --bg:      #0f1117;
    --surface: #181c27;
    --surf2:   #1f2435;
    --border:  #2a3045;
    --accent:  #4f8ef7;
    --accent2: #7c3aed;
    --text:    #e8eaf0;
    --muted:   #6b7280;

    --cell-w: 44px;
    --cell-h: 18px;
    --gap:    1px;

    --c-t1:        #ffe4e4;
    --c-t2:        #ffb3b3;
    --c-t3:        #bbf7d0;
    --c-t4:        #fef08a;
    --c-taken:     #374151;
    --c-unavail:   #161929;

    --c-sky-garden:  #a7f3d0;
    --c-sky-terrace: #6ee7b7;
    --c-link-bridge: #fed7aa;
    --c-residents:   #bfdbfe;
    --c-roof-garden: #d1fae5;
    --c-preschool:   #fde68a;
    --c-future:      #e9d5ff;
    --c-empty-block: #e5e7eb;
    --c-rental:      #e8d5a3;
  }

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body { background: var(--bg); color: var(--text); font-family: 'DM Sans', sans-serif; min-height: 100vh; }

  /* ── Header ── */
  header {
    background: linear-gradient(135deg,#0f1117,#1a1f35);
    border-bottom: 1px solid var(--border);
    padding: 16px 28px;
    display: flex; align-items: center; justify-content: space-between;
    position: sticky; top: 0; z-index: 200;
  }
  .logo { display:flex; align-items:center; gap:10px; }
  .logo-icon {
    width:34px; height:34px;
    background: linear-gradient(135deg,var(--accent),var(--accent2));
    border-radius:8px; display:flex; align-items:center; justify-content:center; font-size:17px;
  }
  .logo h1 { font-family:'Space Mono',monospace; font-size:14px; letter-spacing:.06em; }
  .logo p  { font-size:10px; color:var(--muted); letter-spacing:.08em; text-transform:uppercase; margin-top:1px; }
  .hdr-controls { display:flex; align-items:center; gap:10px; }
  .q-group {
    display:flex; align-items:center; gap:7px;
    background:var(--surf2); border:1px solid var(--border); border-radius:8px; padding:5px 12px;
  }
  .q-group label { font-size:11px; color:var(--muted); font-family:'Space Mono',monospace; }
  .q-group input {
    background:transparent; border:none; outline:none;
    color:var(--accent); font-family:'Space Mono',monospace; font-size:16px; font-weight:700;
    width:60px; text-align:center;
  }
  .btn {
    display:flex; align-items:center; gap:6px;
    padding:8px 15px; border-radius:8px;
    font-family:'DM Sans',sans-serif; font-size:13px; font-weight:600;
    cursor:pointer; border:none; transition:all .15s;
  }
  .btn-primary { background:linear-gradient(135deg,var(--accent),#3b6fd4); color:#fff; }
  .btn-primary:hover { filter:brightness(1.12); transform:translateY(-1px); }
  .btn-primary:disabled { opacity:.5; cursor:not-allowed; transform:none; }
  .btn-ghost { background:var(--surf2); color:var(--muted); border:1px solid var(--border); }
  .btn-ghost:hover { border-color:var(--accent); color:var(--text); }
  .type-select {
    background: var(--surf2); border: 1px solid var(--border); border-radius: 8px;
    padding: 5px 10px; color: var(--text);
    font-family: 'DM Sans', sans-serif; font-size: 13px; font-weight: 600;
    cursor: pointer; outline: none; height: 36px;
    transition: border-color .15s;
    appearance: none; -webkit-appearance: none;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='10' height='6' viewBox='0 0 10 6'%3E%3Cpath d='M1 1l4 4 4-4' stroke='%236b7280' stroke-width='1.5' fill='none' stroke-linecap='round'/%3E%3C/svg%3E");
    background-repeat: no-repeat;
    background-position: right 10px center;
    padding-right: 28px;
  }
  .type-select:hover { border-color: var(--accent); }
  .type-select option { background: #1a2235; color: var(--text); }

  /* ── Status bar ── */
  .status-bar {
    background:var(--surface); border-bottom:1px solid var(--border);
    padding:8px 28px; display:flex; align-items:center; gap:20px; font-size:12px; flex-wrap:wrap;
  }
  .stat { display:flex; align-items:center; gap:5px; color:var(--muted); }
  .stat b { font-family:'Space Mono',monospace; font-size:13px; font-weight:700; color:var(--text); }
  .stat b.red { color:#f87171; } .stat b.grn { color:#4ade80; }
  .prog-wrap { flex:1; max-width:180px; height:4px; background:var(--surf2); border-radius:2px; overflow:hidden; }
  .prog-fill { height:100%; background:linear-gradient(90deg,var(--accent),var(--accent2)); border-radius:2px; transition:width .3s; width:0; }
  #simStatus { font-family:'Space Mono',monospace; font-size:11px; color:var(--accent); min-width:110px; }

  /* ── Legend ── */
  .legend-bar {
    background:var(--surface); border-bottom:1px solid var(--border);
    padding:7px 28px; display:flex; align-items:center; gap:14px; flex-wrap:wrap;
  }
  .leg { display:flex; align-items:center; gap:5px; font-size:11px; color:var(--muted); }
  .leg-sw { width:11px; height:11px; border-radius:2px; border:1px solid rgba(255,255,255,.1); flex-shrink:0; }

  /* ── Grid wrapper ── */
  .grid-scroll { overflow:auto; padding:20px 28px 48px; }
  .grid { display:inline-flex; flex-direction:column; }

  /* Each row is a flex row of cells */
  .floor-row, .hdr-row { display:flex; align-items:stretch; }

  /* Floor label column */
  .fl-lbl {
    width:30px; min-width:30px; flex-shrink:0;
    display:flex; align-items:center; justify-content:center;
    font-family:'Space Mono',monospace; font-size:9px; color:var(--muted);
    border-right:1px solid var(--border);
    position:sticky; left:0; background:var(--bg); z-index:10;
  }

  /* Gap between blocks */
  .blk-gap { width:10px; min-width:10px; flex-shrink:0; }

  /* Block banner */
  .blk-banner {
    display:flex; align-items:center; justify-content:center;
    background:linear-gradient(135deg,#1e3a5f,#2d1b69);
    border:1px solid #3b5b8a; border-radius:5px 5px 0 0;
    font-family:'Space Mono',monospace; font-size:11px; font-weight:700;
    letter-spacing:.08em; color:#fff; height:24px;
  }

  /* Unit number header */
  .unit-hdr {
    flex-shrink:0;
    width:44px;
    display:flex; flex-direction:column; align-items:center; justify-content:center;
    background:#1a2a42; border:1px solid #243550; height:38px; gap:1px;
  }
  .unit-hdr .un { font-family:'Space Mono',monospace; font-size:8px; font-weight:700; color:#93b4e6; }
  .unit-hdr .ut { font-size:7px; color:var(--muted); }

  /* Generic cell */
  .cell {
    flex-shrink:0;
    width:44px; height:18px;
    display:flex; align-items:center; justify-content:center;
    border:1px solid rgba(0,0,0,.12);
    font-size:8px; font-weight:700; font-family:'Space Mono',monospace;
    transition:filter .1s; cursor:default; overflow:hidden;
  }
  .cell:hover { filter:brightness(1.18); position:relative; z-index:2; }

  .cell.t1 { background:var(--c-t1); color:#7f1d1d; }
  .cell.t2 { background:var(--c-t2); color:#7f1d1d; }
  .cell.t3 { background:var(--c-t3); color:#14532d; }
  .cell.t4 { background:var(--c-t4); color:#713f12; }
  .cell.unavail { background:var(--c-unavail); border-color:#1a1f30; }
  .cell.nounit  { background:#e5e7eb; border-color:#d1d5db; }

  .cell.taken {
    background:var(--c-taken) !important;
    color:#9ca3af !important; font-size:11px;
  }
  .cell.taken::after { content:'X'; }
  .cell.taken:hover  { filter:none; }

  /* Amenity span — width applied inline */
  .cell.amenity {
    font-family:'DM Sans',sans-serif;
    font-size:7px; font-weight:700;
    text-align:center; line-height:1.2; padding:0 3px;
    border-radius:0;
  }
  .cell.amenity:hover { filter:none; }
  .cell.amenity.nounit { background:#e5e7eb !important; border-color:#d1d5db !important; }

  .cell.sky-garden  { background:var(--c-sky-garden);  color:#065f46; border-color:#6ee7b7; }
  .cell.sky-terrace { background:var(--c-sky-terrace); color:#065f46; border-color:#34d399; }
  .cell.link-bridge { background:var(--c-link-bridge); color:#7c2d12; border-color:#fb923c; }
  .cell.residents   { background:var(--c-residents);   color:#1e3a5f; border-color:#93c5fd; }
  .cell.roof-garden { background:var(--c-roof-garden); color:#065f46; border-color:#86efac; }
  .cell.preschool   { background:var(--c-preschool);   color:#78350f; border-color:#fbbf24; }
  .cell.future      { background:var(--c-future);      color:#4c1d95; border-color:#c084fc; }
  .cell.empty-block { background:var(--c-empty-block); color:#6b7280; border-color:#d1d5db; }
  .cell.rental      { background:var(--c-rental);      color:#7c5c1e; border-color:#c9a85c; }

  /* ── Legend popup ── */
  .leg { position: relative; cursor: default; }
  .leg-popup {
    display: none;
    position: absolute;
    bottom: calc(100% + 8px);
    left: 50%;
    transform: translateX(-50%);
    background: #1a2235;
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 10px 14px;
    min-width: 170px;
    z-index: 300;
    box-shadow: 0 8px 24px rgba(0,0,0,0.5);
    pointer-events: none;
    white-space: nowrap;
  }
  .leg-popup::after {
    content: '';
    position: absolute;
    bottom: -5px; left: 50%;
    transform: translateX(-50%);
    width: 8px; height: 8px;
    background: #1a2235;
    border-right: 1px solid var(--border);
    border-bottom: 1px solid var(--border);
    rotate: 45deg;
  }
  .leg:hover .leg-popup { display: block; }
  .leg-popup-title {
    font-family: 'Space Mono', monospace;
    font-size: 9px; font-weight: 700;
    color: #93b4e6;
    margin-bottom: 7px;
    letter-spacing: 0.04em;
  }
  .leg-popup-row {
    display: flex; justify-content: space-between;
    align-items: center; gap: 16px;
    font-size: 10px; color: var(--muted);
    padding: 2px 0;
  }
  .leg-popup-row span:last-child {
    font-family: 'Space Mono', monospace;
    font-weight: 700; color: var(--text);
  }
  .leg-popup-row .avail  { color: #4ade80; }
  .leg-popup-row .taken-n { color: #f87171; }
  .leg-popup-divider { border: none; border-top: 1px solid var(--border); margin: 5px 0; }

  /* ── Unit popup ── */
  .unit-hdr { position: relative; cursor: pointer; }
  .unit-popup {
    display: none;
    position: absolute;
    top: calc(100% + 8px);
    left: 50%;
    transform: translateX(-50%);
    background: #1a2235;
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 10px 12px;
    min-width: 160px;
    z-index: 150;
    box-shadow: 0 8px 24px rgba(0,0,0,0.5);
    pointer-events: none;
  }
  .unit-popup::before {
    content: '';
    position: absolute;
    top: -5px; left: 50%;
    transform: translateX(-50%);
    width: 8px; height: 8px;
    background: #1a2235;
    border-left: 1px solid var(--border);
    border-top: 1px solid var(--border);
    rotate: 45deg;
  }
  .unit-hdr:hover .unit-popup { display: block; }
  .unit-popup-title {
    font-family: 'Space Mono', monospace;
    font-size: 9px; font-weight: 700;
    color: #93b4e6;
    margin-bottom: 8px;
    letter-spacing: 0.05em;
  }
  .unit-popup-row {
    display: flex; justify-content: space-between; align-items: center;
    font-size: 10px; color: var(--muted);
    padding: 2px 0;
    gap: 12px;
  }
  .unit-popup-row span:last-child {
    font-family: 'Space Mono', monospace;
    font-weight: 700; color: var(--text);
    white-space: nowrap;
  }
  .unit-popup-row span:last-child .avail { color: #4ade80; }
  .unit-popup-row span:last-child .taken-n { color: #f87171; }
  .unit-popup-divider {
    border: none; border-top: 1px solid var(--border);
    margin: 6px 0;
  }

  /* ── Tooltip ── */
  #tooltip {
    position:fixed; background:var(--surf2); border:1px solid var(--border);
    border-radius:6px; padding:5px 10px; font-size:11px; color:var(--text);
    pointer-events:none; z-index:300; white-space:nowrap; opacity:0; transition:opacity .1s;
  }
  #tooltip.vis { opacity:1; }

  /* ── Toast ── */
  #toast {
    position:fixed; bottom:28px; left:50%;
    transform:translateX(-50%) translateY(70px);
    background:var(--surf2); border:1px solid var(--border); border-radius:12px;
    padding:12px 20px; font-size:13px; color:var(--text);
    z-index:999; transition:transform .3s cubic-bezier(.34,1.56,.64,1);
    box-shadow:0 8px 32px rgba(0,0,0,.4); max-width:400px; text-align:center;
  }
  #toast.show { transform:translateX(-50%) translateY(0); }
  #toast.ok  { border-color:#4ade80; }
  #toast.err { border-color:#f87171; }

  /* ── Spinner ── */
  @keyframes spin { to { transform:rotate(360deg); } }
  .spinner {
    width:13px; height:13px;
    border:2px solid rgba(255,255,255,.3); border-top-color:#fff;
    border-radius:50%; animation:spin .6s linear infinite; display:none;
  }
  .running .spinner   { display:block; }
  .running .btn-label { display:none; }

  @keyframes popIn { from{opacity:0;transform:scale(.8)} to{opacity:1;transform:scale(1)} }
  .just-taken { animation:popIn .18s ease forwards; }
</style>
</head>
<body>

<header>
  <div class="logo">
    <div class="logo-icon">🏢</div>
    <div>
      <h1>BTO SIMULATOR</h1>
      <p>HDB Flat Selection · Queue Simulator</p>
    </div>
  </div>
  <div class="hdr-controls">
    <div class="q-group">
      <label>QUEUE #</label>
      <input type="number" id="queueInput" value="50" min="2" max="5000">
    </div>
    <select class="type-select" id="typeSelect" title="Select flat type to simulate">
      <option value="t4">4 Room</option>
      <option value="t1">2Rm Flexi T1</option>
      <option value="t2">2Rm Flexi T2</option>
      <option value="t3">3 Room</option>
    </select>
    <button class="btn btn-ghost" onclick="resetSim()">↺ Reset</button>
    <button class="btn btn-primary" id="runBtn" onclick="runSim()">
      <div class="spinner"></div>
      <span class="btn-label">▶ Run Simulation</span>
    </button>
  </div>
</header>

<div class="status-bar">
  <div class="stat">Total <b id="sTotal">—</b></div>
  <div class="stat">Taken <b id="sTaken" class="red">0</b></div>
  <div class="stat">Available <b id="sAvail" class="grn">—</b></div>
  <div class="prog-wrap"><div class="prog-fill" id="progFill"></div></div>
  <div id="simStatus">Ready</div>
</div>

<div class="legend-bar">
  <div class="leg" data-legtype="t1"><div class="leg-sw" style="background:var(--c-t1)"></div>2Rm Flexi T1<div class="leg-popup" id="legpop-t1"></div></div>
  <div class="leg" data-legtype="t2"><div class="leg-sw" style="background:var(--c-t2)"></div>2Rm Flexi T2<div class="leg-popup" id="legpop-t2"></div></div>
  <div class="leg" data-legtype="t3"><div class="leg-sw" style="background:var(--c-t3)"></div>3 Room<div class="leg-popup" id="legpop-t3"></div></div>
  <div class="leg" data-legtype="t4"><div class="leg-sw" style="background:var(--c-t4)"></div>4 Room<div class="leg-popup" id="legpop-t4"></div></div>
  <div class="leg"><div class="leg-sw" style="background:var(--c-taken)"></div>Taken (X)</div>
  <div class="leg"><div class="leg-sw" style="background:var(--c-sky-terrace)"></div>Sky Terrace</div>
  <div class="leg"><div class="leg-sw" style="background:var(--c-sky-garden)"></div>Sky Garden</div>
  <div class="leg"><div class="leg-sw" style="background:var(--c-preschool)"></div>Preschool</div>
  <div class="leg"><div class="leg-sw" style="background:var(--c-future)"></div>Future Amenities</div>
  <div class="leg"><div class="leg-sw" style="background:var(--c-residents)"></div>Residents Ctr</div>
  <div class="leg"><div class="leg-sw" style="background:var(--c-rental)"></div>Rental</div>
  <div class="leg"><div class="leg-sw" style="background:var(--c-unavail);border-color:#2a3045"></div>Not Available</div>
</div>

<div class="grid-scroll">
  <div class="grid" id="grid"></div>
</div>

<div id="tooltip"></div>
<div id="toast"></div>

<script>
// ════════════════════════════════════════════════════════════
// CONSTANTS
// ════════════════════════════════════════════════════════════
const MIN_FL  = 1;
const MAX_FL  = 46;
const CELL_W  = 44;   // px — matches CSS .cell width
const CELL_H  = 18;   // px
const GAP     = 1;    // px between cells
const BLK_GAP = 10;   // px between blocks

// ════════════════════════════════════════════════════════════
// BLOCK / STACK DATA
// ════════════════════════════════════════════════════════════
const BLOCKS = [
  { name:"100A", stacks:[
    {u:101,t:"t1",mf:35},{u:103,t:"t1",mf:35},{u:105,t:"t2",mf:35},{u:107,t:"t2",mf:35},
    {u:109,t:"t4",mf:35},{u:111,t:"t4",mf:35},
    {u:113,t:"t4",mf:46},{u:115,t:"t3",mf:46},{u:117,t:"t4",mf:46},{u:119,t:"t4",mf:46},
    {u:121,t:"t2",mf:46},{u:123,t:"t2",mf:46},{u:125,t:"t2",mf:46},{u:127,t:"t4",mf:46},
  ]},
  { name:"104A", stacks:[
    {u:147,t:"t1",mf:32},{u:149,t:"t1",mf:32},{u:151,t:"t2",mf:32},{u:153,t:"t2",mf:32},
    {u:155,t:"t4",mf:32},{u:157,t:"t4",mf:32},
    {u:159,t:"t4",mf:45},{u:161,t:"t3",mf:45},{u:163,t:"t4",mf:45},{u:165,t:"t4",mf:45},
    {u:167,t:"t2",mf:45},{u:169,t:"t2",mf:45},{u:171,t:"t4",mf:45},{u:173,t:"t4",mf:45},
  ]},
  { name:"105A", stacks:[
    {u:175,t:"t1",mf:15},{u:177,t:"t1",mf:15},{u:179,t:"t2",mf:15},{u:181,t:"t2",mf:15},
    {u:183,t:"t4",mf:15},{u:185,t:"t3",mf:15},
    {u:187,t:"t2",mf:40},{u:189,t:"t2",mf:40},{u:191,t:"t4",mf:40},{u:193,t:"t4",mf:40},
    {u:195,t:"t4",mf:40},{u:197,t:"t4",mf:40},{u:199,t:"t4",mf:40},
  ]},
];

// Flat list of all stacks in render order + which block they belong to
const ALL_STACKS = [];
BLOCKS.forEach(b => b.stacks.forEach(s => ALL_STACKS.push({b, s})));

// Quick unit→stack lookup
const S = {};
ALL_STACKS.forEach(({s}) => { S[s.u] = s; });

// ════════════════════════════════════════════════════════════
// AMENITY SPANS
// Per floor, list spans left→right.
// Each span: { from, to, label, css }
// Spans are rendered as a single wide cell replacing all cells in the range.
// Units NOT covered by any span on that floor get normal cell treatment.
// ════════════════════════════════════════════════════════════
const SPANS = {
  // Block 100A
  36: [ {from:101, to:111, label:"Sky Garden (Accessible)",  css:"sky-garden"} ],
  37: [ {from:113, to:127, label:"Sky Terrace (Accessible)", css:"sky-terrace"} ],
  17: [ {from:101, to:127, label:"Sky Terrace (Accessible)", css:"sky-terrace"},
        {from:147, to:173, label:"Sky Terrace (Accessible)", css:"sky-terrace"} ],
  33: [ {from:147, to:157, label:"Sky Garden (Accessible)",  css:"sky-garden"} ],
  34: [ {from:159, to:173, label:"Sky Terrace (Accessible)", css:"sky-terrace"} ],
   2: [ {from:147, to:173, label:"",                         css:"nounit"} ],
   1: [ {from:125, to:127, label:"Residents Network Centre", css:"residents"},
        {from:147, to:153, label:"Preschool",                css:"preschool"},
        {from:157, to:159, label:"Future Amenities/Facilities", css:"future"},
        {from:171, to:173, label:"Preschool",                css:"preschool"},
        {from:175, to:199, label:"",                         css:"empty-block"} ],
};

// Single-cell amenity overrides (floor → { unit → {label, css} })
const SINGLE = {
  2: { 109: {label:"Link Bridge to 104A Roof Garden", css:"link-bridge"} },
};

// Rental units: { unit: { minFloor, maxFloor } } — shown in sand, excluded from simulation
const RENTALS = {
  123: { min: 2,  max: 28 },  // Block 100A #02-123 to #28-123
  169: { min: 3,  max: 29 },  // Block 104A #03-169 to #29-169
};

// Units that have no flat on floor 1 (rendered as blank/transparent)
const NO_UNIT_FL1 = new Set([
  101,103,105,107,109,111,113,115,117,119,121,123,
  155,161,163,165,167,169,
]);

// ── Pre-compute span coverage sets ────────────────────────────
// spanCovered[fl][unit] = span descriptor (if this unit is covered on floor fl)
// spanOwner[fl][unit]   = true if this unit is the FIRST in a span (renders the cell)
const spanCovered = {};
const spanOwner   = {};

Object.entries(SPANS).forEach(([flStr, spans]) => {
  const fl = Number(flStr);
  spanCovered[fl] = spanCovered[fl] || {};
  spanOwner[fl]   = spanOwner[fl]   || {};

  spans.forEach(span => {
    let first = true;
    ALL_STACKS.forEach(({s}) => {
      if (s.u >= span.from && s.u <= span.to) {
        spanCovered[fl][s.u] = span;
        if (first) { spanOwner[fl][s.u] = span; first = false; }
      }
    });
  });
});

// ── Skip floors per unit for simulation ───────────────────────
const SKIP_FL = {};
ALL_STACKS.forEach(({s}) => {
  const skip = new Set();
  // All floors where this unit is covered by a span
  Object.keys(spanCovered).forEach(fl => {
    if (spanCovered[Number(fl)][s.u]) skip.add(Number(fl));
  });
  // Single amenity floors
  Object.keys(SINGLE).forEach(fl => {
    if (SINGLE[Number(fl)]?.[s.u]) skip.add(Number(fl));
  });
  // Floor 1 no-unit
  if (NO_UNIT_FL1.has(s.u)) skip.add(1);
  // Rental floors
  if (RENTALS[s.u]) { for (let f = RENTALS[s.u].min; f <= RENTALS[s.u].max; f++) skip.add(f); }
  SKIP_FL[s.u] = skip;
});

// ════════════════════════════════════════════════════════════
// SIMULATION DATA
// ════════════════════════════════════════════════════════════
// Probability configs per flat type
// t1, t2, t3 probabilities TBD — placeholders marked so they're easy to update
const TYPE_PROBS = {
  t4: {
    blockProb: {"100A":0.48,"104A":0.18,"105A":0.34},
    stackProb: {
      "100A":{109:0.30,111:0.12,113:0.13,117:0.10,119:0.22,127:0.13},
      "104A":{155:0.18,157:0.08,159:0.13,163:0.11,165:0.28,171:0.11,173:0.11},
      "105A":{183:0.28,191:0.08,193:0.26,195:0.11,197:0.09,199:0.18},
    },
  },
  t1: {
    blockProb: {"100A":0.33,"104A":0.33,"105A":0.34},  // placeholder — TBD
    stackProb: {
      "100A":{101:0.50,103:0.50},                       // placeholder — TBD
      "104A":{147:0.50,149:0.50},                       // placeholder — TBD
      "105A":{175:0.50,177:0.50},                       // placeholder — TBD
    },
  },
  t2: {
    blockProb: {"100A":0.33,"104A":0.33,"105A":0.34},  // placeholder — TBD
    stackProb: {
      "100A":{105:0.25,107:0.25,121:0.17,123:0.17,125:0.16},  // placeholder — TBD
      "104A":{151:0.34,153:0.33,167:0.33},                     // placeholder — TBD
      "105A":{179:0.25,181:0.25,187:0.25,189:0.25},            // placeholder — TBD
    },
  },
  t3: {
    blockProb: {"100A":0.34,"104A":0.33,"105A":0.33},  // placeholder — TBD
    stackProb: {
      "100A":{115:1.0},   // placeholder — TBD
      "104A":{161:1.0},   // placeholder — TBD
      "105A":{185:1.0},   // placeholder — TBD
    },
  },
};

// Active probabilities — set at runtime from dropdown
let BLOCK_PROB = TYPE_PROBS.t4.blockProb;
let STACK_PROB = TYPE_PROBS.t4.stackProb;
const BANDS = [
  {label:"highest",min:null,max:null,p:0.32},
  {label:"41-46",  min:41,  max:46,  p:0.11},
  {label:"36-40",  min:36,  max:40,  p:0.11},
  {label:"31-35",  min:31,  max:35,  p:0.14},
  {label:"26-30",  min:26,  max:30,  p:0.10},
  {label:"21-25",  min:21,  max:25,  p:0.16},
  {label:"16-20",  min:16,  max:20,  p:0.17},
  {label:"11-15",  min:11,  max:15,  p:0.11},
  {label:"6-10",   min:6,   max:10,  p:0.03},
  {label:"2-5",    min:2,   max:5,   p:0.04},
];

// ════════════════════════════════════════════════════════════
// STATE
// ════════════════════════════════════════════════════════════
const taken  = {};  // taken[unit][floor] = true
const cellEl = {};  // cellEl[unit][floor] = DOM element
ALL_STACKS.forEach(({s}) => { taken[s.u] = {}; cellEl[s.u] = {}; });
let totalUnits = 0;

// ════════════════════════════════════════════════════════════
// HELPER: pixel width of a span from unit A to unit B
// Accounts for gaps between cells AND the block-gap if the span crosses a block boundary
// ════════════════════════════════════════════════════════════
function spanPx(fromU, toU) {
  // Each cell is CELL_W px (border-box). No CSS gap on flex rows — span width = N * CELL_W only.
  // Block-gap divs are separate siblings and must NOT be included in the span width.
  let count = 0, inSpan = false;
  ALL_STACKS.forEach(({s}) => {
    if (s.u === fromU) inSpan = true;
    if (inSpan) count++;
    if (s.u === toU)   inSpan = false;
  });
  return count * CELL_W;
}

// ════════════════════════════════════════════════════════════
// UNIT POPUP — count available/total per stack
// ════════════════════════════════════════════════════════════
function getStackCounts(unit) {
  const s    = S[unit];
  const skip = SKIP_FL[unit] || new Set();
  const rental = RENTALS[unit];
  let total = 0, takenCount = 0;
  for (let fl = MIN_FL; fl <= s.mf; fl++) {
    if (skip.has(fl)) continue;
    if (rental && fl >= rental.min && fl <= rental.max) continue;
    total++;
    if (taken[unit][fl]) takenCount++;
  }
  return { total, taken: takenCount, avail: total - takenCount };
}

function refreshPopup(unit) {
  const pop = document.getElementById(`popup-${unit}`);
  if (!pop) return;
  const s = S[unit];
  const { total, taken: t, avail } = getStackCounts(unit);
  const typeNames = {t1:'2Rm Flexi T1',t2:'2Rm Flexi T2',t3:'3 Room',t4:'4 Room'};
  pop.innerHTML = `
    <div class="unit-popup-title">#${unit} · ${typeNames[s.t]}</div>
    <div class="unit-popup-row"><span>Available</span><span><span class="avail">${avail}</span></span></div>
    <div class="unit-popup-row"><span>Taken</span><span><span class="taken-n">${t}</span></span></div>
    <hr class="unit-popup-divider">
    <div class="unit-popup-row"><span>Total units</span><span>${total}</span></div>
  `;
}

function refreshAllPopups() {
  ALL_STACKS.forEach(({s}) => refreshPopup(s.u));
}

// ════════════════════════════════════════════════════════════
// LEGEND POPUP — count available/total per flat type
// ════════════════════════════════════════════════════════════
function getTypeCounts(type) {
  let total = 0, takenCount = 0;
  ALL_STACKS.forEach(({s}) => {
    if (s.t !== type) return;
    const skip   = SKIP_FL[s.u] || new Set();
    const rental = RENTALS[s.u];
    for (let fl = MIN_FL; fl <= s.mf; fl++) {
      if (skip.has(fl)) continue;
      if (rental && fl >= rental.min && fl <= rental.max) continue;
      total++;
      if (taken[s.u][fl]) takenCount++;
    }
  });
  return { total, taken: takenCount, avail: total - takenCount };
}

function refreshLegendPopups() {
  const TYPE_FULL_LONG = {t1:'2 Room Flexi Type 1', t2:'2 Room Flexi Type 2', t3:'3 Room', t4:'4 Room'};
  ['t1','t2','t3','t4'].forEach(type => {
    const pop = document.getElementById(`legpop-${type}`);
    if (!pop) return;
    const { total, taken: t, avail } = getTypeCounts(type);
    pop.innerHTML = `
      <div class="leg-popup-title">${TYPE_FULL_LONG[type]}</div>
      <div class="leg-popup-row"><span>Available</span><span class="avail">${avail}</span></div>
      <div class="leg-popup-row"><span>Taken</span><span class="taken-n">${t}</span></div>
      <hr class="leg-popup-divider">
      <div class="leg-popup-row"><span>Total units</span><span>${total}</span></div>
      <div class="leg-popup-row"><span>% remaining</span><span>${total ? Math.round(avail/total*100) : 0}%</span></div>
    `;
  });
}

// ════════════════════════════════════════════════════════════
// BUILD GRID
// ════════════════════════════════════════════════════════════
const TYPE_LABEL = {t1:"2Rm T1",t2:"2Rm T2",t3:"3Rm",t4:"4Rm"};
const TYPE_FULL  = {t1:"2 Room Flexi Type 1",t2:"2 Room Flexi Type 2",t3:"3 Room",t4:"4 Room"};

function makeFlLbl(txt) {
  const d = document.createElement('div');
  d.className = 'fl-lbl';
  d.textContent = txt;
  return d;
}
function makeGap(px) {
  const d = document.createElement('div');
  d.style.cssText = `width:${px}px;min-width:${px}px;flex-shrink:0;`;
  return d;
}

function buildGrid() {
  const grid = document.getElementById('grid');
  grid.innerHTML = '';
  totalUnits = 0;

  // ── Row 1: block banners ──────────────────────────────────
  const bannerRow = document.createElement('div');
  bannerRow.className = 'hdr-row';
  bannerRow.appendChild(makeFlLbl(''));

  BLOCKS.forEach((b, bi) => {
    if (bi > 0) bannerRow.appendChild(makeGap(BLK_GAP));
    const w = b.stacks.length * CELL_W;
    const el = document.createElement('div');
    el.className = 'blk-banner';
    el.style.width = w + 'px';
    el.textContent = `Block ${b.name}`;
    bannerRow.appendChild(el);
  });
  grid.appendChild(bannerRow);

  // ── Row 2: unit number headers ────────────────────────────
  const unitRow = document.createElement('div');
  unitRow.className = 'hdr-row';
  unitRow.appendChild(makeFlLbl(''));

  ALL_STACKS.forEach(({b, s}, idx) => {
    if (idx > 0 && ALL_STACKS[idx-1].b !== b) unitRow.appendChild(makeGap(BLK_GAP));
    const el = document.createElement('div');
    el.className = 'unit-hdr';
    el.dataset.unit = s.u;
    el.innerHTML = `<span class="un">#${s.u}</span><span class="ut">${TYPE_LABEL[s.t]}</span><div class="unit-popup" id="popup-${s.u}"></div>`;
    unitRow.appendChild(el);
  });
  grid.appendChild(unitRow);

  // ── Floor rows ────────────────────────────────────────────
  for (let fl = MAX_FL; fl >= MIN_FL; fl--) {
    const row = document.createElement('div');
    row.className = 'floor-row';
    row.appendChild(makeFlLbl(String(fl).padStart(2,'0')));

    // Track which units have already been absorbed into a rendered span
    const rendered = new Set();
    // Track the last block we actually rendered a cell for (for gap insertion)
    let lastBlock = null;

    ALL_STACKS.forEach(({b, s}) => {
      // Skip units already consumed by a span
      if (rendered.has(s.u)) return;

      // Determine what we're about to render
      const span   = spanOwner[fl]?.[s.u];
      const isMid  = !span && !!spanCovered[fl]?.[s.u];
      if (isMid) { rendered.add(s.u); return; } // orphaned mid-span unit (shouldn't happen, but safe)

      // Insert block gap only when we are about to render something from a new block
      if (lastBlock !== null && lastBlock !== b) row.appendChild(makeGap(BLK_GAP));
      lastBlock = b;

      // ── Span owner ─────────────────────────────────────────
      if (span) {
        const w = spanPx(span.from, span.to);
        const el = document.createElement('div');
        el.className = `cell amenity ${span.css}`;
        el.style.width    = w + 'px';
        el.style.minWidth = w + 'px';
        el.textContent = span.label;
        row.appendChild(el);
        ALL_STACKS.forEach(({s:s2}) => {
          if (s2.u >= span.from && s2.u <= span.to) rendered.add(s2.u);
        });
        return;
      }

      // ── Single-cell amenity ────────────────────────────────
      const single = SINGLE[fl]?.[s.u];
      if (single) {
        const el = document.createElement('div');
        el.className = `cell amenity ${single.css}`;
        el.textContent = single.label;
        row.appendChild(el);
        return;
      }

      // ── Floor 1 no-unit blank ──────────────────────────────
      if (fl === 1 && NO_UNIT_FL1.has(s.u)) {
        const el = document.createElement('div');
        el.className = 'cell nounit';
        row.appendChild(el);
        return;
      }

      // ── Normal cell ────────────────────────────────────────
      const el = document.createElement('div');
      const rental = RENTALS[s.u];
      if (rental && fl >= rental.min && fl <= rental.max) {
        el.className = 'cell rental';
        el.addEventListener('mouseenter', e => showTip(e, `#${String(fl).padStart(2,'0')}-${s.u}  ·  Rental  ·  Block ${b.name}`));
        el.addEventListener('mouseleave', hideTip);
        el.addEventListener('mousemove',  moveTip);
      } else if (fl > s.mf) {
        el.className = 'cell unavail';
      } else {
        el.className = `cell ${s.t}`;
        totalUnits++;
        cellEl[s.u][fl] = el;
        if (taken[s.u][fl]) el.classList.add('taken');
        el.addEventListener('mouseenter', e => showTip(e, `#${String(fl).padStart(2,'0')}-${s.u}  ·  ${TYPE_FULL[s.t]}  ·  Block ${b.name}`));
        el.addEventListener('mouseleave', hideTip);
        el.addEventListener('mousemove',  moveTip);
      }
      row.appendChild(el);
    });

    grid.appendChild(row);
  }

  updateStats();
  refreshAllPopups();
  refreshLegendPopups();
}

// ════════════════════════════════════════════════════════════
// STATS
// ════════════════════════════════════════════════════════════
function countTaken() {
  let n = 0;
  Object.values(taken).forEach(f => Object.values(f).forEach(v => { if(v) n++; }));
  return n;
}
function updateStats() {
  const t = countTaken(), a = totalUnits - t;
  document.getElementById('sTotal').textContent = totalUnits;
  document.getElementById('sTaken').textContent = t;
  document.getElementById('sAvail').textContent = a;
  document.getElementById('progFill').style.width = totalUnits ? (t/totalUnits*100)+'%' : '0%';
}

// ════════════════════════════════════════════════════════════
// SIMULATION
// ════════════════════════════════════════════════════════════
function wPick(arr) { // arr: [{key, p}]
  const r = Math.random(); let c = 0;
  for (const x of arr) { c += x.p; if (r < c) return x.key; }
  return arr[arr.length-1].key;
}

function pickFloor(band, u) {
  const mf = S[u].mf, skip = SKIP_FL[u];
  if (band.label === "highest") {
    for (let f = mf; f >= MIN_FL; f--) { if (!skip.has(f) && !taken[u][f]) return f; }
    return null;
  }
  if (band.min > mf) {
    for (let f = mf; f >= MIN_FL; f--) { if (!skip.has(f) && !taken[u][f]) return f; }
    return null;
  }
  const cands = [];
  for (let f = band.min; f <= Math.min(band.max, mf); f++) {
    if (!skip.has(f) && !taken[u][f]) cands.push(f);
  }
  return cands.length ? cands[Math.floor(Math.random()*cands.length)] : null;
}

function simOne() {
  const blk = wPick(Object.keys(BLOCK_PROB).map(k=>({key:k,p:BLOCK_PROB[k]})));
  const sp  = STACK_PROB[blk];
  const u   = parseInt(wPick(Object.keys(sp).map(k=>({key:parseInt(k),p:sp[k]}))));
  for (let i = 0; i < 100; i++) {
    const band  = wPick(BANDS.map(b=>({key:b,p:b.p})));
    const floor = pickFloor(band, u);
    if (floor !== null) return {u, floor};
  }
  return null;
}

async function runSim() {
  const q = parseInt(document.getElementById('queueInput').value);
  if (isNaN(q) || q < 2) { toast('Enter a queue number ≥ 2', 'err'); return; }

  // Set active probability tables from dropdown
  const selectedType = document.getElementById('typeSelect').value;
  const probs = TYPE_PROBS[selectedType];
  BLOCK_PROB = probs.blockProb;
  STACK_PROB = probs.stackProb;

  const btn = document.getElementById('runBtn');
  btn.disabled = true; btn.classList.add('running');

  const updates = [];
  const total = q - 1;
  const chunk = 30;
  const statusEl = document.getElementById('simStatus');

  for (let t = 0; t < total; t += chunk) {
    const end = Math.min(t + chunk, total);
    for (let i = t; i < end; i++) {
      const r = simOne();
      if (r) { taken[r.u][r.floor] = true; updates.push(r); }
    }
    statusEl.textContent = `Simulating ${end}/${total}…`;
    updateStats();
    await new Promise(r => setTimeout(r, 0));
  }

  updates.forEach(({u, floor}) => {
    const el = cellEl[u]?.[floor];
    if (el) { el.classList.add('taken','just-taken'); setTimeout(()=>el.classList.remove('just-taken'),250); }
  });

  updateStats();
  refreshAllPopups();
  refreshLegendPopups();
  statusEl.textContent = `Done — ${updates.length} taken`;
  btn.disabled = false; btn.classList.remove('running');

  const avail = totalUnits - countTaken();
  toast(avail > 0
    ? `✅ Queue #${q} — ${updates.length} units taken. ${avail} still available!`
    : `😔 All units taken before queue #${q}.`,
    avail > 0 ? 'ok' : 'err');
}

function resetSim() {
  ALL_STACKS.forEach(({s}) => {
    Object.keys(taken[s.u]).forEach(fl => {
      if (taken[s.u][fl]) {
        taken[s.u][fl] = false;
        cellEl[s.u]?.[Number(fl)]?.classList.remove('taken');
      }
    });
  });
  updateStats();
  refreshAllPopups();
  refreshLegendPopups();
  document.getElementById('simStatus').textContent = 'Ready';
  toast('↺ Simulation reset', '');
}

// ════════════════════════════════════════════════════════════
// TOOLTIP
// ════════════════════════════════════════════════════════════
const tipEl = document.getElementById('tooltip');
function showTip(e, txt) { tipEl.textContent = txt; tipEl.classList.add('vis'); moveTip(e); }
function hideTip()       { tipEl.classList.remove('vis'); }
function moveTip(e)      { tipEl.style.left=(e.clientX+12)+'px'; tipEl.style.top=(e.clientY-30)+'px'; }

// ════════════════════════════════════════════════════════════
// TOAST
// ════════════════════════════════════════════════════════════
let _tt;
function toast(msg, type='') {
  const el = document.getElementById('toast');
  el.textContent = msg; el.className = 'toast show ' + type;
  clearTimeout(_tt); _tt = setTimeout(()=>el.classList.remove('show'), 4500);
}

// ════════════════════════════════════════════════════════════
// INIT
// ════════════════════════════════════════════════════════════
buildGrid();
</script>
</body>
</html>
