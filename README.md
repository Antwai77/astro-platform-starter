<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Takeover — Refactored</title>
<style>
  /* NEW BRIGHT THEME COLORS */
  :root{
    --bg:#F0F4F8; /* Light blue/grey background */
    --panel:#FFFFFF; /* White panels */
    --muted:#5C7896; /* Muted blue text */
    --accent:#4CAF50; /* Vibrant Green for primary */
    --card:#E3EBF3; /* Lightest blue for elements */
    --ink:#1A3E5C; /* Dark blue for primary text/headers */
    --error-bg: #FEEAEA; /* Light red for errors */
  }

  html,body{height:100%;margin:0;font-family:Inter, system-ui, -apple-system, "Segoe UI", Roboto, Arial}
  body{background:var(--bg);color:var(--muted);-webkit-font-smoothing:antialiased; display: flex; flex-direction: column;}
  header{padding:12px 16px;display:flex;align-items:center;gap:12px; background-color: var(--panel); border-bottom: 1px solid #D5DBE1; box-shadow: 0 2px 4px rgba(0,0,0,0.05);}
  
  .brand-group { display: flex; align-items: center; gap: 8px; }
  .brand{font-weight:900;color:var(--ink);font-size:24px; letter-spacing: -0.5px;}
  .subtitle{font-size:12px;color:var(--muted); font-weight: 500;}

  /* Rotating Dice Animation (New) */
  .dice {
    width: 24px; height: 24px;
    fill: var(--accent);
    transform-origin: center center;
    animation: spinDice 2s linear infinite;
  }
  @keyframes spinDice {
    0% { transform: rotateY(0deg) rotateX(0deg); }
    100% { transform: rotateY(360deg) rotateX(360deg); }
  }

  .ticker-wrap{flex:1;background:var(--card);border-radius:6px;padding:8px 12px;border:1px solid rgba(0,0,0,0.05);overflow:hidden}
  .tape{display:inline-flex;white-space:nowrap;align-items:center;gap:18px;padding-left:6px;animation:scroll 20s linear infinite}
  @keyframes scroll{from{transform:translateX(0)}to{transform:translateX(-50%)}}
  .titem{display:flex;gap:8px;align-items:center}
  .tname{color:var(--ink);font-weight:700}
  .tval{color:var(--accent);font-weight:700}

  .controls{display:flex;gap:8px;align-items:center;margin-left:12px}
  button{background:var(--card);border:1px solid #D5DBE1;color:var(--muted);padding:8px 10px;border-radius:8px;cursor:pointer; transition: background-color 0.2s, box-shadow 0.1s}
  button:hover{background-color: #DDE5EC; box-shadow: 0 1px 3px rgba(0,0,0,0.1);}
  button.primary{background:var(--accent);color:white;border:0;font-weight:700; box-shadow: 0 4px 6px rgba(76, 175, 80, 0.3);}
  button.primary:hover{background-color: #43A047;}
  button.small{padding:6px 8px;font-size:13px}
  button:disabled{opacity: 0.5; cursor: not-allowed;}

  .main{display:flex;gap:16px;padding:16px;flex:1;min-height:0; overflow-y: hidden;}
  .sidebar{width:320px;background:var(--panel);padding:16px;border-radius:12px;box-shadow:0 8px 24px rgba(0,0,0,.1);overflow-y:auto; flex-shrink: 0;}
  .center{flex:1;background:var(--panel);padding:16px;border-radius:12px;overflow:hidden;display:flex;flex-direction:column;min-height:0; box-shadow:0 8px 24px rgba(0,0,0,.1);}
  h3{margin:0;color:var(--ink); font-size: 1.25rem;}
  .small{font-size:13px;color:var(--muted);margin-top:6px}

  .char{display:flex;justify-content:space-between;align-items:center;padding:10px;border-radius:8px;background:var(--card);margin-bottom:8px; border: 1px solid rgba(0,0,0,0.05);}
  .char .left{display:flex;flex-direction:column; min-width: 0; color: var(--ink);}
  .char .left > div { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
  .meta{font-size:13px;color:var(--muted)}
  .char button{margin-left:6px; background-color: var(--panel);}

  .log{background:#ffffff;color:var(--ink);padding:0;border-radius:8px;height:100%; overflow-y:auto;font-family:Georgia,serif; margin-top: 10px; flex: 1; border: 1px solid #D5DBE1;}
  .entry{padding:12px;border-bottom:1px solid #F0F4F8;line-height:1.5}
  .entry:last-child{border-bottom: none;}
  .entry time{display:block;font-size:12px;color:var(--muted);margin-bottom:8px}

  /* Modal Styles */
  .modal-back{position:fixed;inset:0;background:rgba(0,0,0,0.6);display:none;align-items:center;justify-content:center;z-index:9999}
  .modal{background:var(--panel);padding:20px;border-radius:12px;width:92%;max-width:520px;border:1px solid #D5DBE1; box-shadow: 0 10px 30px rgba(0,0,0,0.3);}
  label{display:block;font-size:14px;color:var(--ink);margin-top:10px; font-weight: 500;}
  input[type="text"], select{width:100%;padding:10px;border-radius:8px;border:1px solid #D5DBE1;background:var(--card);color:var(--ink); margin-top: 4px;}
  .programs{display:grid;grid-template-columns:repeat(2,1fr);gap:8px;margin-top:8px}
  .prog{display:flex;justify-content:space-between;align-items:center;padding:8px;border-radius:8px;background:var(--card); border: 1px solid #D5DBE1;}
  .modal .row{display:flex;gap:8px;margin-top:20px;justify-content:flex-end}
  .modal .row button.danger{background-color: var(--error-bg); color: #E53935; border-color: #F8BBD0;}
  .footer{font-size:12px;color:var(--muted);margin-top:8px; text-align: right;}

  /* Specific modal content styles */
  #genericDialog h4{margin-top: 0; color: var(--ink);}
  #genericDialog pre {
    background: var(--card); padding: 12px; border-radius: 8px;
    white-space: pre-wrap; word-break: break-all; font-size: 13px;
    color: var(--ink); margin-top: 10px; border: 1px solid #D5DBE1;
  }
  .detail-section-separator {
    display: block;
    margin: 10px 0;
    border-top: 1px solid #D5DBE1;
  }
  .owner-details-header {
    color:var(--ink); font-size: 15px; margin-top: 16px; margin-bottom: 0; font-weight: 600;
  }

  @media (max-width:900px){
    .main{flex-direction:column; overflow-y: auto;}
    .sidebar{width:auto; max-height: 40vh;}
    .center{flex: 0; min-height: 50vh;}
    .log{height: auto;}
    header{flex-wrap: wrap; justify-content: space-between;}
    .controls{margin-left: 0; margin-top: 8px;}
    .brand-group { flex-shrink: 0;}
  }
</style>
</head>
<body>
  <header>
    <div class="brand-group">
      <svg class="dice" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
          <path d="M5 3C3.89543 3 3 3.89543 3 5V19C3 20.1046 3.89543 21 5 21H19C20.1046 21 21 20.1046 21 19V5C21 3.89543 20.1046 3 19 3H5Z" stroke-width="2" stroke="currentColor" fill="none"/>
          <circle cx="8" cy="8" r="1.5" fill="currentColor"/>
          <circle cx="16" cy="16" r="1.5" fill="currentColor"/>
          <circle cx="16" cy="8" r="1.5" fill="currentColor"/>
          <circle cx="8" cy="16" r="1.5" fill="currentColor"/>
      </svg>
      <div>
        <div class="brand">Takeover</div>
        <div class="subtitle">Market-style story simulation — persistent local save</div>
      </div>
    </div>

    <div class="ticker-wrap" aria-hidden="false">
      <div id="tape" class="tape" aria-live="polite"></div>
    </div>

    <div class="controls" role="toolbar" aria-label="controls">
      <button id="btnAdd" class="primary">+ Add Character</button>
      <button id="btnPause" class="small">Pause</button>
      <button id="btnClear" class="small">Clear Log</button>
      <button id="btnExport" class="small">Export</button>
      <button id="btnImport" class="small">Import</button>
    </div>
  </header>

  <div class="main">
    <aside class="sidebar" aria-label="characters">
      <h3>Characters</h3>
      <div class="small">Tap View to inspect details.</div>
      <div id="characters" style="margin-top:12px"></div>
      <div class="footer">Local save (device)</div>
    </aside>

    <section class="center" aria-live="polite">
      <h3>Event Log</h3>
      <div class="small" id="countdownLabel">Next update in —</div>
      <div id="log" class="log" role="log"></div>
    </section>
  </div>

  <!-- Modal Container -->
  <div id="modalBack" class="modal-back" aria-hidden="true">
    <div class="modal" role="dialog" aria-modal="true" aria-labelledby="modalTitle">

      <!-- 1. Character Creation Form (Default) -->
      <div id="createCharForm">
        <h3 id="modalTitle">Create Character</h3>
        
        <label>Character Name
          <input id="inputName" type="text" placeholder="e.g., Marcus" />
        </label>
        
        <!-- NEW: Owner Fields -->
        <h4 class="owner-details-header">Owner Details (Record Keeping)</h4>
        <label>Owner Name
          <input id="inputOwnerName" type="text" placeholder="e.g., Jane Doe" />
        </label>
        <div style="display:flex;gap:8px;margin-top:8px">
          <label style="flex:1">Owner Email
            <input id="inputOwnerEmail" type="text" placeholder="e.g., jane@example.com" />
          </label>
          <label style="flex:1">Owner Phone
            <input id="inputOwnerPhone" type="text" placeholder="e.g., 555-123-4567" />
          </label>
        </div>
        <!-- END NEW -->

        <div style="display:flex;gap:8px;margin-top:8px">
          <label style="flex:1">Sex
            <select id="inputSex"><option value="male">male</option><option value="female">female</option></select>
          </label>
          <label style="flex:1">Race
            <select id="inputRace"><option value="white">white</option><option value="black">black</option><option value="asian">asian</option><option value="latino">latino</option></select>
          </label>
        </div>

        <label style="margin-top:10px">Programs (multi)</label>
        <div class="programs" id="programList"></div>

        <div class="row">
          <button id="saveBtn" class="primary">Save</button>
          <button id="cancelBtn">Cancel</button>
        </div>
      </div>

      <!-- 2. Generic Alert/Confirm Dialog (Hidden by default) -->
      <div id="genericDialog" style="display: none;">
        <h3 id="dialogTitle"></h3>
        <pre id="dialogMessage"></pre>
        <div class="row">
          <!-- Only for confirm -->
          <button id="dialogCancelBtn" style="display: none;">Cancel</button>
          <!-- For both alert/confirm -->
          <button id="dialogConfirmBtn" class="primary">OK</button>
        </div>
      </div>
    </div>
  </div>

<script>
/* ========== CONFIG ========== */
const IS_OWNER = true;                // set false to disable owner actions
const TURN_INTERVAL_MS = 60000;       // 1 minute default (use 5000 for testing)
const STORAGE_KEY = 'takeover_v2_chars';
const LOG_KEY = 'takeover_v2_logs';

const PROGRAMS = ["criminal","racist","giving","gay","jealous","church","rat","faithful","corrupt","prideful"];
const JOBS = {
  "President":10.00,"Governor":9.00,"Warden":8.00,"Doctor":7.00,"Chief of Police":6.00,"Detective":5.00,
  "Lawyer":5.00,"Counselor":4.00,"Sgt Officer":3.00,"Patrol Officer":3.00,"club dancer":2.00,"Pastor":3.00,
  "cartel1":2.00,"cartel2":2.00,"cartel3":2.00,"cartel":2.00,
  "welfare1":1.00,"welfare2":1.00,"welfare3":1.00,"welfare4":1.00
};

/* ========== STORAGE MODULE ========== */
const Storage = {
  load(key, fallback = []) {
    try { return JSON.parse(localStorage.getItem(key) || JSON.stringify(fallback)); }
    catch { return fallback; }
  },
  save(key, data) { localStorage.setItem(key, JSON.stringify(data)); }
};

/* ========== UTILITIES ========== */
const rand = (arr) => arr[Math.floor(Math.random() * arr.length)];
const rndId = () => Math.random().toString(36).slice(2,9);
const nowISO = () => new Date().toISOString();
const clamp = (v, min, max) => Math.max(min, Math.min(max, v));
const toMoney = (n) => +Number(n || 0).toFixed(2);
function escapeHtml(s){ if(!s) return ''; return String(s).replace(/[&<>"]/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c])); }
function formatDetail(key, value){ return `${key.padEnd(12)}: ${value}`; }

/* ========== APP STATE ========== */
let chars = Storage.load(STORAGE_KEY, []);
let logs = Storage.load(LOG_KEY, []);
let engineHandle = null, countdownHandle = null;
let paused = false, countdown = Math.round(TURN_INTERVAL_MS/1000);
let confirmCallback = null;

/* ========== DOM REFS ========== */
const tape = document.getElementById('tape');
const charactersEl = document.getElementById('characters');
const logEl = document.getElementById('log');
const modalBack = document.getElementById('modalBack');
const programListEl = document.getElementById('programList');
const countdownLabel = document.getElementById('countdownLabel');

// New Modal Refs
const createCharForm = document.getElementById('createCharForm');
const genericDialog = document.getElementById('genericDialog');
const dialogTitleEl = document.getElementById('dialogTitle');
const dialogMessageEl = document.getElementById('dialogMessage');
const dialogConfirmBtn = document.getElementById('dialogConfirmBtn');
const dialogCancelBtn = document.getElementById('dialogCancelBtn');

/* ========== CUSTOM ALERT/CONFIRM HANDLERS ========== */
function hideModal(){
  modalBack.style.display = 'none';
  modalBack.setAttribute('aria-hidden','true');
  // Reset visibility for next use
  createCharForm.style.display = 'none';
  genericDialog.style.display = 'none';
}

function showModal(contentToShow, titleText){
  createCharForm.style.display = 'none';
  genericDialog.style.display = 'none';

  if(contentToShow === 'create'){
    createCharForm.style.display = 'block';
  } else if(contentToShow === 'generic'){
    genericDialog.style.display = 'block';
    dialogTitleEl.textContent = titleText || 'Information';
  }

  modalBack.style.display = 'flex';
  modalBack.setAttribute('aria-hidden','false');
}

function showAlert(message, title = 'Notification'){
  dialogMessageEl.textContent = message;
  dialogCancelBtn.style.display = 'none';
  dialogConfirmBtn.textContent = 'OK';
  dialogConfirmBtn.onclick = hideModal;
  showModal('generic', title);
}

function showConfirm(message, callback, title = 'Confirm Action', confirmText = 'Confirm', cancelText = 'Cancel'){
  confirmCallback = callback;
  dialogMessageEl.textContent = message;
  dialogCancelBtn.style.display = 'block';
  dialogCancelBtn.textContent = cancelText;
  dialogConfirmBtn.textContent = confirmText;

  dialogConfirmBtn.onclick = () => {
    hideModal();
    if(confirmCallback) confirmCallback(true);
    confirmCallback = null;
  };

  dialogCancelBtn.onclick = () => {
    hideModal();
    if(confirmCallback) confirmCallback(false);
    confirmCallback = null;
  };

  showModal('generic', title);
}

/* ========== INITIAL UI BUILD ========== */
PROGRAMS.forEach(p=>{
  const d = document.createElement('div'); d.className='prog';
  d.innerHTML = `<span style="text-transform:capitalize">${escapeHtml(p)}</span><input type="checkbox" data-prog="${escapeHtml(p)}" />`;
  programListEl.appendChild(d);
});

/* ========== SMALL HELPERS ========== */
const updateStatus = c => c.status = Math.floor((c.wallet || 0) / 25);
function saveAll(){ Storage.save(STORAGE_KEY, chars); Storage.save(LOG_KEY, logs); }
function appendLog(text){
  logs.unshift({ ts: nowISO(), text: String(text) });
  logs = logs.slice(0,2000);
  Storage.save(LOG_KEY, logs);
  renderLogs();
}
function makeChar({name, sex, race, programs = [], ownerName, ownerEmail, ownerPhone}){
  return {
    id: rndId(), name, sex, race, programs,
    ownerName, ownerEmail, ownerPhone, // Owner details
    life:50, wallet:0, status:0, location:'apartment', con:0,
    friendList:[], enemyList:[], items:[], spouse:null, job:null, isDead:false,
    created: nowISO()
  };
}

/* ========== JOB HELPERS ========== */
function takenJobs(){ return chars.filter(c=>c.job).map(c=>c.job); }
function offerJob(){
  const all = Object.keys(JOBS);
  const taken = takenJobs();
  const avail = all.filter(j => !taken.includes(j));
  if(avail.length === 0) return null;
  avail.sort((a,b)=> JOBS[b]-JOBS[a]);
  const topHalf = avail.slice(0, Math.ceil(avail.length/2));
  const pool = topHalf.concat(avail);
  return rand(pool);
}
function assignJobToChar(char){
  const job = offerJob();
  if(!job) return null;
  char.job = job;
  char.wallet = toMoney((char.wallet||0) + (JOBS[job]||0));
  updateStatus(char);
  return job;
}
function assignWorkLocation(job){
  const mapping = {
    "President":"state department","Governor":"state department","Warden":"state department","Chief of Police":"state department","Detective":"state department",
    "Patrol Officer":"station","Sgt Officer":"station","Doctor":"hospital","Pastor":"church","club dancer":"nightclub",
    "Lawyer":"court","cartel1":"warehouse","cartel2":"warehouse","cartel3":"warehouse","cartel":"warehouse"
  };
  return mapping[job] || 'office';
}

/* ========== RENDERING ========== */
function renderTicker(){
  tape.innerHTML = '';
  if(!chars.length){
    const t = document.createElement('div'); t.className='titem';
    t.innerHTML = `<div class="tname">—</div><div class="tval">No characters</div>`;
    tape.appendChild(t); return;
  }
  const items = chars.map(c=>{
    const span = document.createElement('div'); span.className='titem';
    const prideTag = (c.programs||[]).includes('prideful') ? ' 👑' : '';
    span.innerHTML = `<div class="tname">${escapeHtml(c.name)}${prideTag}</div><div class="tval">$${toMoney(c.wallet).toFixed(2)}</div>`;
    span.title = `${c.name} — ${c.job || 'Unemployed'}`;
    return span;
  });
  // duplicate items for smooth scroll
  items.concat(items).forEach(it => tape.appendChild(it));
}

function renderChars(){
  charactersEl.innerHTML = '';
  const sorted = chars.slice().sort((a,b)=> (b.wallet||0)-(a.wallet||0));
  sorted.forEach(c=>{
    const el = document.createElement('div'); el.className='char';
    const left = document.createElement('div'); left.className='left';
    const prideLabel = (c.programs||[]).includes('prideful') ? ' (Prideful 👑)' : '';
    // NOTE: Removed ownerName from the main list display as per user request. It remains in the 'View' modal.

    left.innerHTML = `<div style="font-weight:700">${escapeHtml(c.name)} ${c.isDead?'<span style="color:#ff5252">(Deceased)</span>':''}${prideLabel}</div>
                      <div class="meta">${escapeHtml(c.job || 'Unemployed')} • ${escapeHtml(c.location||'apartment')} • Life:${c.life}</div>`;
    el.appendChild(left);

    const right = document.createElement('div');
    const view = document.createElement('button'); view.className='small'; view.textContent='View';
    view.addEventListener('click', ()=> showAlert(detailText(c), `Character Details: ${c.name}`));

    // --- Character deletion functionality removed per user request ---
    
    right.appendChild(view);
    el.appendChild(right);
    charactersEl.appendChild(el);
  });
}

function renderLogs(){
  if(!logs.length){
    logEl.innerHTML = `<div class="entry">The city is quiet. Add a character to begin their story.</div>`;
    return;
  }
  logEl.innerHTML = logs.map(l => `
    <div class="entry">
      <time>${new Date(l.ts).toLocaleString()}</time>
      <div>${escapeHtml(l.text)}</div>
    </div>`).join('');
}

function detailText(c){
  const details = [
    // Owner Details - KEPT IN VIEW MODAL
    formatDetail('Owner Name', c.ownerName || 'N/A'),
    formatDetail('Owner Email', c.ownerEmail || 'N/A'),
    formatDetail('Owner Phone', c.ownerPhone || 'N/A'),
    '---', // Separator for display
    // Character Details
    formatDetail('ID', c.id),
    formatDetail('Name', c.name),
    formatDetail('Sex', c.sex),
    formatDetail('Race', c.race),
    formatDetail('Life', c.life),
    formatDetail('Wallet', `$${toMoney(c.wallet).toFixed(2)}`),
    formatDetail('Status', c.status),
    formatDetail('Con', c.con),
    formatDetail('Location', c.location || 'apartment'),
    formatDetail('Job', c.job || 'Unemployed'),
    formatDetail('Spouse', c.spouse || 'none'),
    formatDetail('Programs', (c.programs||[]).join(', ') || 'none'),
    formatDetail('Friends', (c.friendList||[]).join(', ') || 'none'),
    formatDetail('Enemies', (c.enemyList||[]).join(', ') || 'none'),
    formatDetail('Items', (c.items||[]).join(', ') || 'none'),
    formatDetail('Created', new Date(c.created).toLocaleDateString()),
    c.isDead ? formatDetail('Deceased', c.timeOfDeath ? new Date(c.timeOfDeath).toLocaleString() : 'Yes') : null
  ].filter(Boolean).join('\n').replace('---\n', '\n\n'); // Replace my internal separator with two newlines
  return details;
}

function updateCountdownLabel(){
  countdownLabel.textContent = paused ? 'Paused' : (countdown > 0 ? `Next update in ${countdown}s` : 'Updating...');
}

function renderAll(){
  renderChars(); renderTicker(); renderLogs(); updateCountdownLabel();
}

/* ========== GAME RULES: TALK & ACTION ========== */
function doTalkRoll(char){
  const roll = Math.floor(Math.random()*6)+1 + Math.floor(Math.random()*6)+1;
  const texts = [];
  const others = chars.filter(c => c.id !== char.id && !c.isDead);
  const partner = others.length ? rand(others) : null;

  // helper checks
  const hasProg = (c, p) => (c.programs||[]).includes(p);

  // friendship (2-4)
  if([2,3,4].includes(roll) && partner){
    const cPrideful = hasProg(char, 'prideful');
    const pPrideful = hasProg(partner, 'prideful');
    if(!(hasProg(char,'racist') && partner.race !== char.race)){
      if(!char.friendList.includes(partner.name) && !char.enemyList.includes(partner.name)){
        if((cPrideful && (partner.status||0) < (char.status||0)) || (pPrideful && (char.status||0) < (partner.status||0))){
          if(!char.enemyList.includes(partner.name)) char.enemyList.push(partner.name);
          if(!partner.enemyList.includes(char.name)) partner.enemyList.push(char.name);
          texts.push(`${char.name} looked down on ${partner.name} and the meeting turned sour — rivals were born.`);
        } else {
          char.friendList.push(partner.name);
          texts.push(`${char.name} found a companion in ${partner.name}; a new friendship quietly began.`);
        }
      }
    }
  }

  // emotion (5)
  if(roll === 5){
    if(char.life >= 60) texts.push(`${char.name} felt light and content, as if the world had softened for a moment.`);
    else if(char.life >= 40) texts.push(`${char.name} carried a melancholy like a low cloud, steady and dull.`);
    else if(char.life >= 20) texts.push(`${char.name} walked under a heavy quiet that made choices slow and blunt.`);
    else texts.push(`${char.name} burned with a fierce anger that left them small and raw.`);
  }

  // offer job (6)
  if(roll === 6){
    const offered = offerJob();
    if(offered){
      const pay = JOBS[offered] || 0;
      const accept = Math.random() < 0.9;
      if(accept){
        char.job = offered; char.wallet = toMoney((char.wallet||0) + pay); updateStatus(char);
        texts.push(`${char.name} was offered the post of ${offered} (pay $${pay.toFixed(2)}) and accepted.`);
      } else texts.push(`${char.name} was offered ${offered} but declined for reasons they would not share.`);
    } else texts.push(`${char.name} hoped for an opportunity but found none available.`);
  }

  // marriage/spouse (7)
  if(roll === 7 && partner){
    const sameSexOK = hasProg(char,'gay') || hasProg(partner,'gay');
    const oppositeSex = char.sex !== partner.sex;
    const cStatus = char.status || 0;
    const pStatus = partner.status || 0;
    const cPrideful = hasProg(char,'prideful');
    const pPrideful = hasProg(partner,'prideful');

    if(!char.spouse && !partner.spouse && (oppositeSex || sameSexOK)){
      if(!(hasProg(char,'racist') && partner.race !== char.race)){
        let canMarry = true;
        if(cPrideful && pStatus < cStatus) canMarry = false;
        if(pPrideful && cStatus < pStatus) canMarry = false;
        if(canMarry){
          char.spouse = partner.name; partner.spouse = char.name;
          texts.push(`${char.name} and ${partner.name} pledged themselves to one another in a quiet promise.`);
        } else texts.push(`${char.name} considered marriage, but pride got in the way.`);
      } else texts.push(`${char.name} considered a closer bond but differences could not be bridged.`);
    } else if(char.spouse === partner.name){
      char.life = clamp(char.life + 10, 0, 100);
      texts.push(`${char.name} spent a night with ${partner.name}, gaining solace (+10 life).`);
    }
  }

  // workplace/spouse/officer/cartel logic (8,9)
  if([8,9].includes(roll) && partner){
    const stateJobs = ["President","Governor","Warden","Chief of Police","Detective","Counselor","Sgt Officer","Patrol Officer"];
    if(char.location === partner.location && char.location === 'state department' && char.job && partner.job){
      const myPay = JOBS[char.job] || 0; const otherPay = JOBS[partner.job] || 0;
      if(myPay > otherPay && partner.con > 0){
        partner.job = null; texts.push(`${char.name} used influence to remove ${partner.name} from their position.`);
      } else if(otherPay > myPay && char.con > 0){
        char.job = null; texts.push(`${partner.name} engineered ${char.name}'s dismissal.`);
      }
    }

    // spouse sharing
    if(char.spouse === partner.name){
      const share = toMoney((char.wallet||0) * 0.2);
      char.wallet = toMoney((char.wallet||0) - share);
      partner.wallet = toMoney((partner.wallet||0) + share);
      texts.push(`${char.name} gave $${share.toFixed(2)} to their spouse ${partner.name}.`);
    }

    // officers send to jail
    const officers = ['Patrol Officer','Sgt Officer','Detective','Chief of Police','Warden'];
    if(officers.includes(char.job) && partner.con > 0){
      partner.location = 'jail';
      texts.push(`${char.name} used authority to have ${partner.name} imprisoned.`);
    }

    // pastor tithe
    if(char.job === 'Pastor' && (partner.programs||[]).includes('church')){
      const tithe = toMoney((partner.wallet||0) * 0.1);
      partner.wallet = toMoney((partner.wallet||0) - tithe);
      char.wallet = toMoney((char.wallet||0) + tithe);
      texts.push(`${char.name} (Pastor) received a tithe of $${tithe.toFixed(2)} from ${partner.name}.`);
    }

    // cartel deal
    if((char.job || '').toLowerCase().includes('cartel')){
      char.wallet = toMoney((char.wallet||0) + 2.00);
      partner.life = clamp((partner.life||0) + 20, 0, 100);
      texts.push(`${char.name} completed a covert deal; money and relief changed hands.`);
    }
  }

  return texts;
}

function actionRoll(attacker, defender){
  const roll = Math.floor(Math.random()*6)+1 + Math.floor(Math.random()*6)+1;
  const out = [];
  attacker.enemyList = attacker.enemyList || []; defender.enemyList = defender.enemyList || [];
  if(!attacker.enemyList.includes(defender.name)) attacker.enemyList.push(defender.name);
  if(!defender.enemyList.includes(attacker.name)) defender.enemyList.push(attacker.name);

  if([2,3,4].includes(roll)){
    out.push(`${attacker.name}'s strike failed and the moment passed.`);
  } else if(roll === 5){
    out.push(`${defender.name} escaped into the crowd.`);
  } else if([6,7,8].includes(roll)){
    if((attacker.items||[]).includes('gun')){ defender.life = clamp(defender.life - 50, 0, 100); out.push(`${attacker.name} fired; ${defender.name} was badly wounded.`); }
    else if((attacker.items||[]).includes('knife')){ defender.life = clamp(defender.life - 20,0,100); out.push(`${attacker.name} stabbed ${defender.name}.`); }
    else { defender.life = clamp(defender.life - 10,0,100); out.push(`${attacker.name} struck ${defender.name} violently.`); }
  } else if([9,11].includes(roll)){
    attacker.location = 'jail'; attacker.con = (attacker.con||0) + 1; out.push(`Police arrived — ${attacker.name} was taken to jail.`);
  } else if([10,12].includes(roll)){
    const stolen = toMoney(defender.wallet || 0);
    const oldAtt = toMoney(attacker.wallet || 0);
    if(stolen > 0){
      attacker.wallet = toMoney(oldAtt + stolen);
      defender.wallet = 0;
      const oldStatus = attacker.status || 0;
      attacker.status = Math.floor(attacker.wallet / 25);
      let statusNote = '';
      if(attacker.status > oldStatus) statusNote = ` Status rose to ${attacker.status}.`;
      out.push(`${attacker.name} landed a decisive blow (roll ${roll}) and seized $${stolen.toFixed(2)} from ${defender.name} (new balances: ${attacker.name} $${attacker.wallet.toFixed(2)}, ${defender.name} $0.00).${statusNote}`);
    } else {
      out.push(`${attacker.name} struck decisively (roll ${roll}) but ${defender.name} had no money to take.`);
    }
  }
  return out;
}

/* ========== LIFE-ROLL TABLE (map roll -> effect) ========== */
const lifeRollEffects = {
  // Con -1 (Jail release logic)
  2: c => { if((c.con||0) > 0){ c.con = Math.max(0, c.con - 1); return `${c.name} felt a small easing of their record. (Con -1)`; } },
  3: c => { c.life = clamp(c.life - 10, 0, 100); return `${c.name} grew ill and lost -10 life.`; },
  4: c => { c.wallet = toMoney((c.wallet||0) - 2.50); return `${c.name} paid rent (-$2.50).`; },
  6: c => { if(c.job){ c.location = assignWorkLocation(c.job); return `${c.name} went to work at ${c.location}.`; } },
  7: c => { if(c.location === 'hospital'){ c.life = clamp(c.life + 30, 0, 100); return `${c.name} recovered in hospital (+30 life).`; } },
  8: c => { c.location = 'nightclub'; return `${c.name} drifted toward the nightclub as music rose.`; },
  9: c => { c.location = 'bank'; const pay = (c.job && JOBS[c.job]) ? JOBS[c.job] : 0; c.wallet = toMoney((c.wallet||0) + pay); return `${c.name} collected pay of $${pay.toFixed(2)} at the bank.`; },
  10: c => { if((c.programs||[]).includes('church')){ c.location = 'church'; return `${c.name} attended church.`; } else { c.location = 'apartment'; return `${c.name} returned home.`; } },
  11: c => { if(c.location === 'casino' && c.job){ const pay = JOBS[c.job]||0; c.wallet = toMoney((c.wallet||0) + pay*2); return `${c.name} doubled pay at the casino.`; } },
  12: c => { const tax = toMoney((c.wallet||0) * 0.1); c.wallet = toMoney((c.wallet||0) - tax); return `${c.name} paid taxes (-$${tax.toFixed(2)}).`; }
};

/* ========== ENGINE TICK ========== */
function engineTick(){
  const newLogs = [];
  for(const c of chars){
    if(c.isDead){
      if(c.location === 'casket'){ c.location = 'funeral'; newLogs.push(`${c.name} was carried from a casket to a small funeral.`); }
      continue;
    }

    // 1) liferoll
    const lr = Math.floor(Math.random()*6)+1 + Math.floor(Math.random()*6)+1;
    const lifeLog = lifeRollEffects[lr]?.(c);
    if(lifeLog) newLogs.push(lifeLog);

    if((c.life||0) < 30 && lr === 5){ c.location = 'hospital'; newLogs.push(`${c.name} was taken to the hospital — dangerously ill.`); }

    // 2) talkroll
    const talkLogItems = doTalkRoll(c);
    talkLogItems.forEach(t => newLogs.push(t));

    // 3) criminal action (if conditions)
    const tr = Math.floor(Math.random()*6)+1 + Math.floor(Math.random()*6)+1;
    if((c.programs||[]).includes('criminal') && c.life < 50 && tr >= 10){
      const targets = chars.filter(o => o.id !== c.id && !o.isDead);
      if(targets.length){
        const target = rand(targets);
        const actions = actionRoll(c, target);
        actions.forEach(a => newLogs.push(a));
      }
    }

    // 4) nightclub/dancer earnings
    if((c.job||'').toLowerCase().includes('club dancer') && c.location === 'nightclub' && (tr === 8 || tr === 9)){
      c.wallet = toMoney((c.wallet||0) + 1.00);
      newLogs.push(`${c.name} earned $1.00 dancing at the club.`);
    }

    // 5) lawyer fees
    if(c.job === 'Lawyer'){
      const othersHere = chars.filter(o=>o.id !== c.id && o.location === c.location);
      othersHere.forEach(p => {
        if((p.con||0) > 0){
          const fee = 1.00 * p.con;
          if((p.wallet||0) >= fee){
            p.wallet = toMoney((p.wallet||0) - fee);
            c.wallet = toMoney((c.wallet||0) + fee);
            newLogs.push(`${c.name} (Lawyer) dismissed ${p.con} con(s) for ${p.name} for $${fee.toFixed(2)}.`);
            p.con = 0;
          }
        }
      });
    }

    // update status after changes
    updateStatus(c);

    // death handling and inheritance
    if((c.life||0) <= 0 && !c.isDead){
      if(c.spouse){
        const spouse = chars.find(x => x.name === c.spouse && !x.isDead);
        if(spouse){
          if((c.programs||[]).includes('giving') && (c.wallet || 0) > 0){
            const amount = toMoney(c.wallet || 0);
            spouse.wallet = toMoney((spouse.wallet||0) + amount);
            newLogs.push(`${c.name} has died and left $${amount.toFixed(2)} to their spouse ${spouse.name}.`);
            c.wallet = 0;
          }
          spouse.spouse = null;
        }
      }
      c.isDead = true; c.location = 'casket'; c.timeOfDeath = nowISO();
      newLogs.push(`${c.name} died and was placed in a casket.`);
    }
  }

  // persist/log/render
  chars = chars; saveAll();
  newLogs.forEach(n => appendLog(n));
  renderAll();
}

/* ========== ENGINE CONTROL ========== */
function startEngine(){
  stopEngine();
  engineHandle = setInterval(()=>{ if(!paused) engineTick(); }, TURN_INTERVAL_MS);

  if(countdownHandle) clearInterval(countdownHandle);
  countdown = Math.round(TURN_INTERVAL_MS/1000);
  countdownHandle = setInterval(()=>{
    if(!paused) countdown--;
    if(countdown <= 0){
      if(!paused) engineTick();
      countdown = Math.round(TURN_INTERVAL_MS/1000);
    }
    updateCountdownLabel();
  }, 1000);
}
function stopEngine(){
  if(engineHandle) clearInterval(engineHandle);
  engineHandle = null;
  if(countdownHandle) clearInterval(countdownHandle);
  countdownHandle = null;
  countdown = -1;
  updateCountdownLabel();
}

/* ========== UI BINDINGS ========== */
document.getElementById('btnAdd').addEventListener('click', ()=> {
  if(!IS_OWNER){ showAlert('Only the owner may add characters.', 'Owner Restriction'); return; }
  showModal('create', 'Create Character');
  document.getElementById('inputName').focus();
});
document.getElementById('cancelBtn').addEventListener('click', hideModal);
document.getElementById('saveBtn').addEventListener('click', ()=> {
  const name = (document.getElementById('inputName').value || '').trim();
  const sex = document.getElementById('inputSex').value;
  const race = document.getElementById('inputRace').value;
  const progs = Array.from(document.querySelectorAll('#programList input[type=checkbox]:checked')).map(x=>x.getAttribute('data-prog'));

  // Retrieve owner details
  const ownerName = document.getElementById('inputOwnerName').value.trim();
  const ownerEmail = document.getElementById('inputOwnerEmail').value.trim();
  const ownerPhone = document.getElementById('inputOwnerPhone').value.trim();

  if(!name){ showAlert('Character Name required', 'Validation Error'); return; }

  const c = makeChar({ name, sex, race, programs: progs, ownerName, ownerEmail, ownerPhone }); // Pass owner details
  chars.push(c); saveAll();
  appendLog(`${c.name} stepped into the city with steady steps. A new story began.`);
  hideModal();
  if(!engineHandle) startEngine();

  // Clear all inputs after save
  document.getElementById('inputName').value='';
  document.getElementById('inputOwnerName').value='';
  document.getElementById('inputOwnerEmail').value='';
  document.getElementById('inputOwnerPhone').value='';
  document.querySelectorAll('#programList input[type=checkbox]').forEach(cb=>cb.checked=false);
  renderAll();
});

document.getElementById('btnPause').addEventListener('click', ()=>{
  paused = !paused;
  document.getElementById('btnPause').textContent = paused ? 'Resume' : 'Pause';
  updateCountdownLabel();
});
document.getElementById('btnClear').addEventListener('click', ()=>{
  if(!IS_OWNER){ showAlert('Only the owner may clear the log.', 'Owner Restriction'); return; }
  showConfirm('Clear the entire event log? This action is irreversible.', (result) => {
    if(!result) return;
    logs = []; Storage.save(LOG_KEY, logs); renderLogs(); appendLog('Event log was cleared.');
  }, 'Confirm Clear Log', 'Clear Log', 'Cancel');
});
document.getElementById('btnExport').addEventListener('click', ()=> {
  if(!IS_OWNER){ showAlert('Only the owner may export data.', 'Owner Restriction'); return; }
  exportData();
});
document.getElementById('btnImport').addEventListener('click', ()=> {
  if(!IS_OWNER){ showAlert('Only the owner may import data.', 'Owner Restriction'); return; }
  importData();
});

/* Export / Import */
function exportData(){
  // The chars object now includes ownerName, ownerEmail, and ownerPhone, so they are automatically included in the export.
  const payload = { chars, logs, exportedAt: nowISO() };
  const blob = new Blob([JSON.stringify(payload, null, 2)], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href = url; a.download='takeover-export.json'; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
  appendLog('Exported data to JSON.');
}
function importData(){
  const input = document.createElement('input'); input.type='file'; input.accept='.json,application/json';
  input.onchange = async (e) => {
    const f = e.target.files[0]; if(!f) return;
    const txt = await f.text();
    try{
      const parsed = JSON.parse(txt);
      if(Array.isArray(parsed.chars)) chars = parsed.chars;
      if(Array.isArray(parsed.logs)) logs = parsed.logs;
      saveAll(); renderAll(); appendLog('Imported data from JSON.');
    }catch(err){ showAlert('Import failed: ' + err.message, 'Import Error'); }
  };
  input.click();
}

/* ========== SEED WHEN EMPTY ========== */
function seedIfEmpty(){
  if(chars.length) return;
  const samples = ['Quiana','Dede','Maurie','Keke'];
  for(const n of samples){
    chars.push({
      id: rndId(), name:n, sex: Math.random() > .5 ? 'male' : 'female',
      race: ['white','black','asian','latino'][Math.floor(Math.random()*4)],
      programs: [], life: Math.floor(40 + Math.random()*60),
      location:'apartment', wallet: toMoney(Math.random()*6), status:0, con:0,
      friendList:[], enemyList:[], items:[], spouse:null, job:null, isDead:false, created: nowISO(),
      ownerName: 'System Admin', // Default owner name for seeded data
      ownerEmail: 'admin@system.io',
      ownerPhone: '555-000-0000'
    });
  }
  Storage.save(STORAGE_KEY, chars);
  appendLog('A few residents appeared to start the city.');
}

/* ========== OWNER RESTRICTIONS ========== */
function applyOwnerRestrictions(){
  if(IS_OWNER) return;
  ['btnAdd','btnClear','btnExport','btnImport'].forEach(id=>{
    const el = document.getElementById(id);
    if(el){ el.disabled = true; el.title = 'Read-only view: only owner may modify data'; }
  });
}
applyOwnerRestrictions();

/* ========== STARTUP ========== */
seedIfEmpty();
renderAll();
if(chars.length) startEngine();
</script>
</body>
</html>

