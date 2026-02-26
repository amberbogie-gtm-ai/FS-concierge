<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Finite State · Security Concierge</title>

<!--
  ═══════════════════════════════════════════════════════════════════
  EMAILJS SETUP — DO THIS ONCE (5 minutes)
  ═══════════════════════════════════════════════════════════════════
  1. Go to https://www.emailjs.com and create a free account
  2. "Email Services" → Add New Service → connect your Gmail
     Copy the Service ID → paste into EMAILJS_SERVICE_ID below

  3. "Email Templates" → Create Two Templates:

     TEMPLATE 1 — Visitor Confirmation  (name it "visitor_confirm")
     Subject:  Your Finite State session is confirmed
     Body:
       Hi {{visitor_name}},
       Your session is confirmed for {{slot}}.
       You will be connected with a Finite State {{specialist}}.
       A calendar invite will follow shortly.
       — Finite State Security Concierge
     To email: {{visitor_email}}

     TEMPLATE 2 — Internal Notification  (name it "internal_notify")
     Subject:  New Concierge booking — {{role}} at {{company}}
     Body:
       New booking via Finite State Security Concierge

       Name:       {{visitor_name}}
       Email:      {{visitor_email}}
       Company:    {{company}}
       Role:       {{role}}
       Branch:     {{branch}}
       Focus:      {{focus}}
       Specialist: {{specialist}}
       Time:       {{slot}}

     To email: {{notify_email}}

  4. "Account" → "General" → copy your Public Key
     Paste into EMAILJS_PUBLIC_KEY below

  5. Both emails fire on booking confirmation.
  ═══════════════════════════════════════════════════════════════════

  ═══════════════════════════════════════════════════════════════════
  SALESFORCE WEB-TO-LEAD SETUP
  ═══════════════════════════════════════════════════════════════════
  1. Salesforce Setup → Web-to-Lead → Create Web-to-Lead Form
  2. Copy your Organisation ID → paste into SF_ORG_ID below
  3. Add two custom Lead fields in Salesforce:
       Concierge_Branch__c  (text)
       Concierge_Focus__c   (text)
     Copy their field IDs (format: 00N...) → paste into
     SF_BRANCH_FIELD and SF_FOCUS_FIELD below
  4. Posts fire silently on booking confirm alongside EmailJS.
  ═══════════════════════════════════════════════════════════════════
-->

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@300;400;500;600&family=IBM+Plex+Sans:ital,wght@0,300;0,400;0,500;0,600;1,400&display=swap" rel="stylesheet">

<!-- EmailJS SDK -->
<script src="https://cdn.jsdelivr.net/npm/@emailjs/browser@4/dist/email.min.js"></script>

<style>
  :root {
    --bg:     #07090d;
    --bg2:    #0c1018;
    --bg3:    #111720;
    --bg4:    #161e2a;
    --green:  #00d4aa;
    --gdim:   rgba(0,212,170,.13);
    --gline:  rgba(0,212,170,.27);
    --gglow:  rgba(0,212,170,.06);
    --amber:  #f0a500;
    --adim:   rgba(240,165,0,.13);
    --aline:  rgba(240,165,0,.28);
    --purple: #a78bfa;
    --pdim:   rgba(167,139,250,.13);
    --pline:  rgba(167,139,250,.28);
    --blue:   #4f8ef7;
    --text:   #dde3ed;
    --text2:  #7d8898;
    --text3:  #3f4a58;
    --line:   rgba(255,255,255,.055);
    --mono:   'IBM Plex Mono', monospace;
    --sans:   'IBM Plex Sans', sans-serif;
    --r:      6px;
  }
  *,*::before,*::after{margin:0;padding:0;box-sizing:border-box}
  body{background:var(--bg);color:var(--text);font-family:var(--sans);
    min-height:100vh;display:flex;flex-direction:column;overflow-x:hidden}
  body::before{content:'';position:fixed;inset:0;z-index:0;pointer-events:none;
    background-image:linear-gradient(rgba(0,212,170,.022) 1px,transparent 1px),
      linear-gradient(90deg,rgba(0,212,170,.022) 1px,transparent 1px);
    background-size:44px 44px}
  body::after{content:'';position:fixed;top:-30%;right:-15%;width:55%;height:55%;
    background:radial-gradient(ellipse,rgba(0,212,170,.032) 0%,transparent 68%);
    pointer-events:none;z-index:0}

  .hdr{position:relative;z-index:20;padding:14px 36px;border-bottom:1px solid var(--line);
    display:flex;align-items:center;gap:14px;
    background:rgba(7,9,13,.93);backdrop-filter:blur(14px)}
  .logo-img{height:23px;width:auto;filter:brightness(0) invert(1);opacity:.88}
  .logo-fb{display:none;font-family:var(--mono);font-size:14px;font-weight:600;
    letter-spacing:.08em;color:var(--text)}
  .hdr-badge{margin-left:auto;font-family:var(--mono);font-size:9px;font-weight:500;
    letter-spacing:.14em;text-transform:uppercase;color:var(--green);
    border:1px solid var(--gline);padding:3px 9px;border-radius:2px;background:var(--gdim)}

  .main{position:relative;z-index:1;flex:1;display:flex;flex-direction:column;
    max-width:900px;width:100%;margin:0 auto;padding:34px 18px 18px}

  .hero{text-align:center;padding-bottom:32px;animation:up .6s ease both}
  @keyframes up{from{opacity:0;transform:translateY(13px)}to{opacity:1;transform:translateY(0)}}
  .eyebrow{font-family:var(--mono);font-size:9px;letter-spacing:.22em;text-transform:uppercase;
    color:var(--green);margin-bottom:13px;display:flex;align-items:center;justify-content:center;gap:8px}
  .eyebrow::before,.eyebrow::after{content:'';width:34px;height:1px;background:var(--gline)}
  .hero h1{font-family:var(--mono);font-size:clamp(19px,3.4vw,27px);font-weight:600;
    letter-spacing:-.02em;color:var(--text);line-height:1.3;margin-bottom:9px}
  .hero h1 span{color:var(--green)}
  .hero p{font-size:13.5px;color:var(--text2);line-height:1.65;max-width:490px;margin:0 auto}

  .shell{background:var(--bg3);border:1px solid var(--line);border-radius:var(--r);
    overflow:hidden;display:flex;flex-direction:column;height:640px;
    box-shadow:0 0 0 1px rgba(0,212,170,.04),0 28px 56px rgba(0,0,0,.45);
    animation:up .6s .13s ease both}

  .topbar{display:flex;align-items:center;gap:10px;padding:11px 17px;
    background:var(--bg4);border-bottom:1px solid var(--line);flex-shrink:0}
  .dot{width:7px;height:7px;border-radius:50%;background:var(--green);
    box-shadow:0 0 7px var(--green);animation:blink 2.2s ease infinite;flex-shrink:0}
  @keyframes blink{0%,100%{opacity:1}50%{opacity:.35}}
  .tb-meta{display:flex;flex-direction:column;gap:1px}
  .tb-name{font-family:var(--mono);font-size:11px;font-weight:600;color:var(--text);letter-spacing:.04em}
  .tb-sub{font-family:var(--mono);font-size:9px;color:var(--green);letter-spacing:.07em;min-height:12px}
  .tb-right{margin-left:auto;display:flex;align-items:center;gap:10px}
  #restartBtn{background:transparent;border:1px solid var(--line);color:var(--text3);
    padding:4px 8px;border-radius:3px;font-family:var(--mono);font-size:9px;
    letter-spacing:.07em;text-transform:uppercase;cursor:pointer;transition:all .17s;display:none}
  #restartBtn:hover{color:var(--text2);border-color:var(--gline)}
  #restartBtn.on{display:block}
  #stepLbl{font-family:var(--mono);font-size:9px;color:var(--text3);letter-spacing:.07em;white-space:nowrap}

  .msgs{flex:1;overflow-y:auto;padding:20px 17px;
    display:flex;flex-direction:column;gap:16px;scroll-behavior:smooth}
  .msgs::-webkit-scrollbar{width:3px}
  .msgs::-webkit-scrollbar-track{background:transparent}
  .msgs::-webkit-scrollbar-thumb{background:var(--line);border-radius:2px}

  .msg{display:flex;gap:9px;animation:pop .3s ease both}
  @keyframes pop{from{opacity:0;transform:translateY(6px)}to{opacity:1;transform:translateY(0)}}
  .msg.usr{flex-direction:row-reverse}
  .av{width:28px;height:28px;border-radius:4px;flex-shrink:0;
    display:flex;align-items:center;justify-content:center;
    font-family:var(--mono);font-size:10px;font-weight:600}
  .msg.bot .av{background:var(--gdim);border:1px solid var(--gline);color:var(--green)}
  .msg.usr .av{background:rgba(79,142,247,.12);border:1px solid rgba(79,142,247,.25);
    color:var(--blue);font-size:8px;text-align:center;line-height:1.25}
  .bub{max-width:79%;padding:10px 14px;border-radius:var(--r);font-size:13.5px;line-height:1.67}
  .msg.bot .bub{background:var(--bg4);border:1px solid var(--line);color:var(--text);border-top-left-radius:2px}
  .msg.usr .bub{background:rgba(79,142,247,.11);border:1px solid rgba(79,142,247,.19);color:var(--text);border-top-right-radius:2px}

  #ia{flex-shrink:0}

  .opt-wrap{padding:12px 17px 15px;border-top:1px solid var(--line);
    background:var(--bg2);display:flex;flex-direction:column;gap:8px;animation:up .24s ease both}
  .opt-lbl{font-family:var(--mono);font-size:9px;letter-spacing:.12em;
    text-transform:uppercase;color:var(--text3);margin-bottom:1px}
  .opt-grid{display:flex;flex-wrap:wrap;gap:7px}
  .opt-btn{background:var(--bg4);border:1px solid var(--line);color:var(--text2);
    padding:8px 14px;border-radius:4px;font-family:var(--sans);font-size:13px;
    cursor:pointer;transition:all .16s ease;text-align:left;line-height:1.4}
  .opt-btn:hover{border-color:var(--gline);color:var(--green);background:var(--gglow);transform:translateY(-1px)}
  .opt-btn.sel{border-color:var(--green);color:var(--green);background:var(--gdim)}
  .opt-btn.am:hover{border-color:var(--aline);color:var(--amber);background:var(--adim)}
  .opt-btn.am.sel{border-color:var(--amber);color:var(--amber);background:var(--adim)}
  .opt-btn.pu:hover{border-color:var(--pline);color:var(--purple);background:var(--pdim)}
  .opt-btn.pu.sel{border-color:var(--purple);color:var(--purple);background:var(--pdim)}

  .inp-outer{padding:12px 17px;border-top:1px solid var(--line);background:var(--bg2);display:flex;flex-direction:column;gap:8px}
  .inp-row{display:flex;gap:8px}
  .inp-row input{flex:1;background:var(--bg4);border:1px solid var(--line);
    color:var(--text);padding:9px 12px;border-radius:4px;
    font-family:var(--sans);font-size:13px;outline:none;transition:border-color .17s}
  .inp-row input::placeholder{color:var(--text3)}
  .inp-row input:focus{border-color:var(--gline)}
  .send-btn{background:var(--green);border:none;color:var(--bg);
    padding:9px 15px;border-radius:4px;font-family:var(--mono);font-size:11px;
    font-weight:600;letter-spacing:.06em;cursor:pointer;transition:all .16s;white-space:nowrap}
  .send-btn:hover{background:#00eec0;transform:translateY(-1px)}
  .send-btn:disabled{opacity:.38;cursor:not-allowed;transform:none}
  .skip-lnk{background:transparent;border:none;color:var(--text3);
    font-family:var(--mono);font-size:10px;text-decoration:underline;
    text-underline-offset:3px;cursor:pointer;transition:color .17s;text-align:left}
  .skip-lnk:hover{color:var(--text2)}

  .inline-inp{display:flex;gap:8px;padding:12px 17px;
    border-top:1px solid var(--line);background:var(--bg2);flex-shrink:0}
  .inline-inp input{flex:1;background:var(--bg4);border:1px solid var(--line);
    color:var(--text);padding:9px 12px;border-radius:4px;
    font-family:var(--sans);font-size:13px;outline:none;transition:border-color .17s}
  .inline-inp input::placeholder{color:var(--text3)}
  .inline-inp input:focus{border-color:var(--gline)}

  .tdots{background:var(--bg4);border:1px solid var(--line);
    border-radius:var(--r);border-top-left-radius:2px;
    padding:12px 15px;display:flex;gap:4px;align-items:center}
  .td{width:5px;height:5px;border-radius:50%;background:var(--text3);animation:db 1.4s ease infinite}
  .td:nth-child(2){animation-delay:.22s}
  .td:nth-child(3){animation-delay:.44s}
  @keyframes db{0%,60%,100%{transform:translateY(0);opacity:.3}30%{transform:translateY(-5px);opacity:1}}

  .res-wrap{display:flex;flex-direction:column;gap:7px;max-width:79%}
  .res-section-lbl{font-family:var(--mono);font-size:9px;font-weight:600;
    letter-spacing:.12em;text-transform:uppercase;color:var(--text2);margin-bottom:2px}
  .res-card{background:var(--bg4);border:1px solid var(--line);border-radius:4px;
    padding:9px 12px;display:flex;align-items:flex-start;gap:9px;
    cursor:pointer;transition:all .16s;text-decoration:none;color:inherit}
  .res-card:hover{border-color:var(--gline);background:var(--gglow)}
  .res-icon{font-size:14px;flex-shrink:0;margin-top:1px}
  .res-type{font-family:var(--mono);font-size:9px;letter-spacing:.1em;
    text-transform:uppercase;color:var(--text3);margin-bottom:2px}
  .res-name{font-size:12.5px;color:var(--text);line-height:1.45}

  .book-card{background:var(--bg4);border:1px solid var(--gline);
    border-radius:var(--r);padding:15px;max-width:79%}
  .book-card.am{border-color:var(--aline)}
  .book-card.pu{border-color:var(--pline)}
  .book-title{font-family:var(--mono);font-size:10px;font-weight:600;
    letter-spacing:.1em;text-transform:uppercase;color:var(--green);margin-bottom:6px}
  .book-card.am .book-title{color:var(--amber)}
  .book-card.pu .book-title{color:var(--purple)}
  .book-who{font-size:12px;color:var(--text2);margin-bottom:11px;line-height:1.5}
  .book-fields{display:flex;flex-direction:column;gap:7px;margin-bottom:10px}
  .book-fields input{background:var(--bg3);border:1px solid var(--line);
    color:var(--text);padding:8px 11px;border-radius:3px;
    font-family:var(--sans);font-size:13px;outline:none;transition:border-color .17s;width:100%}
  .book-fields input::placeholder{color:var(--text3)}
  .book-fields input:focus{border-color:var(--gline)}
  .book-slots-lbl{font-family:var(--mono);font-size:9px;letter-spacing:.1em;
    text-transform:uppercase;color:var(--text3);margin-bottom:5px}
  .book-slots{display:flex;flex-wrap:wrap;gap:6px;margin-bottom:8px}
  .slot{background:var(--bg3);border:1px solid var(--line);color:var(--text2);
    padding:5px 10px;border-radius:3px;font-family:var(--mono);font-size:10px;
    cursor:pointer;transition:all .15s}
  .slot:hover{border-color:var(--gline);color:var(--green)}
  .book-card.am .slot:hover{border-color:var(--aline);color:var(--amber)}
  .book-card.pu .slot:hover{border-color:var(--pline);color:var(--purple)}
  .slot.picked{border-color:var(--green);color:var(--green);background:var(--gdim)}
  .book-card.am .slot.picked{border-color:var(--amber);color:var(--amber);background:var(--adim)}
  .book-card.pu .slot.picked{border-color:var(--purple);color:var(--purple);background:var(--pdim)}
  .book-note{font-family:var(--mono);font-size:9px;color:var(--text3);
    letter-spacing:.04em;margin-bottom:8px}
  .book-submit{background:var(--green);border:none;color:var(--bg);width:100%;
    padding:9px;border-radius:3px;font-family:var(--mono);font-size:11px;
    font-weight:600;letter-spacing:.08em;text-transform:uppercase;
    cursor:pointer;transition:all .16s}
  .book-submit:hover{background:#00eec0}
  .book-submit:disabled{opacity:.38;cursor:not-allowed}
  .book-card.am .book-submit{background:var(--amber);color:#000}
  .book-card.am .book-submit:hover{background:#ffc129}
  .book-card.pu .book-submit{background:var(--purple)}
  .book-card.pu .book-submit:hover{background:#c4b5fd}
  .book-error{font-family:var(--mono);font-size:10px;color:#f87171;
    margin-top:6px;display:none}

  .config-warn{background:rgba(240,165,0,.1);border:1px solid var(--aline);
    border-radius:4px;padding:10px 14px;margin-bottom:14px;
    font-family:var(--mono);font-size:11px;color:var(--amber);line-height:1.55;display:none}
  .config-warn.show{display:block}

  .foot{text-align:center;padding:16px 0 0;font-family:var(--mono);
    font-size:9px;color:var(--text3);letter-spacing:.07em;animation:up .6s .27s ease both}
</style>
</head>
<body>

<header class="hdr">
  <img class="logo-img"
    src="https://finitestate.io/hs-fs/hubfs/22144c83-c678-4b5f-83ae-51f23da9a089.webp?width=356"
    alt="Finite State"
    onerror="this.style.display='none';document.querySelector('.logo-fb').style.display='block'"/>
  <span class="logo-fb">FINITE STATE</span>
  <span class="hdr-badge">Security Concierge · Beta</span>
</header>

<main class="main">

  <div class="config-warn" id="configWarn">
    ⚠️  Setup required before sharing: add your Anthropic API key and EmailJS credentials at the top of the &lt;script&gt; block. See inline comments for instructions.
  </div>

  <div class="hero">
    <div class="eyebrow">Finite Concierge</div>
    <h1>Accelerate Your <span>Product Security</span></h1>
    <p>Three questions. A direct path to the right expert, the right content, or a booked call. No forms. No SDR queue.</p>
  </div>

  <div class="shell">
    <div class="topbar">
      <div class="dot"></div>
      <div class="tb-meta">
        <span class="tb-name">Finite Concierge</span>
        <span class="tb-sub" id="trackLbl"></span>
      </div>
      <div class="tb-right">
        <button id="restartBtn" onclick="restart()">Restart</button>
        <span id="stepLbl">Step 1 of 6</span>
      </div>
    </div>
    <div class="msgs" id="msgs"></div>
    <div id="ia"></div>
  </div>

  <div class="foot">Powered by Claude AI · Finite State Security Concierge</div>
</main>

<script>
/* ═══════════════════════════════════════════════════════════════════
   ANTHROPIC API CONFIG
   ═══════════════════════════════════════════════════════════════════
   NOTE: Key is visible in browser source. Fine for internal POC/demo.
   For production, proxy through a serverless function.
   ═══════════════════════════════════════════════════════════════════ */
const ANTHROPIC_API_KEY = "YOUR_ANTHROPIC_API_KEY";

/* ═══════════════════════════════════════════════════════════════════
   EMAILJS CONFIG
   ═══════════════════════════════════════════════════════════════════ */
const EMAILJS_PUBLIC_KEY        = "YOUR_PUBLIC_KEY";
const EMAILJS_SERVICE_ID        = "YOUR_SERVICE_ID";
const EMAILJS_TEMPLATE_VISITOR  = "YOUR_VISITOR_TEMPLATE";
const EMAILJS_TEMPLATE_INTERNAL = "YOUR_INTERNAL_TEMPLATE";
const NOTIFY_EMAIL              = "SalesRep@FiniteState.IO";  // ← internal booking notification

/* ═══════════════════════════════════════════════════════════════════
   SALESFORCE WEB-TO-LEAD CONFIG
   ═══════════════════════════════════════════════════════════════════
   To activate:
   1. Salesforce Setup → Web-to-Lead → Create Web-to-Lead Form
   2. Copy your Org ID → replace YOUR_SF_ORG_ID below
   3. Add custom Lead fields in Salesforce:
        Concierge_Branch__c  (text)
        Concierge_Focus__c   (text)
      Copy their field IDs (format: 00N...) → replace the
      SF_BRANCH_FIELD and SF_FOCUS_FIELD placeholders below
   4. Posts fire silently alongside EmailJS on every booking.
   ═══════════════════════════════════════════════════════════════════ */
const SF_ORG_ID       = "YOUR_SF_ORG_ID";          // ← paste Salesforce Org ID
const SF_ENDPOINT     = "https://webto.salesforce.com/servlet/servlet.WebToLead";
const SF_BRANCH_FIELD = "00N_BRANCH_FIELD_ID";      // ← custom field: Concierge_Branch__c
const SF_FOCUS_FIELD  = "00N_FOCUS_FIELD_ID";       // ← custom field: Concierge_Focus__c
const SF_RETURL       = "https://finitestate.io";

/* ═══════════════════════════════════════════════════════════════════
   INIT
   ═══════════════════════════════════════════════════════════════════ */
const CONFIGURED = EMAILJS_PUBLIC_KEY !== "YOUR_PUBLIC_KEY";
const API_READY  = ANTHROPIC_API_KEY  !== "YOUR_ANTHROPIC_API_KEY";
const SF_READY   = SF_ORG_ID          !== "YOUR_SF_ORG_ID";

if (CONFIGURED) emailjs.init({ publicKey: EMAILJS_PUBLIC_KEY });
if (!CONFIGURED || !API_READY) document.getElementById('configWarn').classList.add('show');

/* ═══════════════════════════════════════════════════════════════════
   STATE
   ═══════════════════════════════════════════════════════════════════ */
const API = "https://api.anthropic.com/v1/messages";

let S = {
  branch: null, sub: null, sub2: null,
  role: null, company: null, history: []
};

/* ═══════════════════════════════════════════════════════════════════
   UTILITIES
   ═══════════════════════════════════════════════════════════════════ */
const sleep = ms => new Promise(r => setTimeout(r, ms));
const mk = (tag, cls) => { const e = document.createElement(tag); if (cls) e.className = cls; return e; };

function setStep(n, total) {
  document.getElementById('stepLbl').textContent = `Step ${n} of ${total}`;
  if (n > 1) document.getElementById('restartBtn').classList.add('on');
}
function setTrack(t) { document.getElementById('trackLbl').textContent = t || ''; }
function clearIA()   { document.getElementById('ia').innerHTML = ''; }

async function type(text) {
  const C   = document.getElementById('msgs');
  const row = mk('div', 'msg bot');
  const av  = mk('div', 'av');  av.textContent = 'FC';
  const bub = mk('div', 'bub');
  row.appendChild(av); row.appendChild(bub); C.appendChild(row);
  const plain = text.replace(/<[^>]*>/g, '');
  for (let i = 0; i < plain.length; i++) {
    const ch = plain[i];
    bub.textContent = plain.slice(0, i + 1);
    let d = 14 + Math.random() * 18;
    if ('!?'.includes(ch))                d = 270 + Math.random() * 200;
    else if (ch === '.')                  d = 230 + Math.random() * 190;
    else if (ch === ',')                  d = 88  + Math.random() * 60;
    else if (ch === '\n')                 d = 180 + Math.random() * 120;
    else if (ch === ' ' && Math.random() < .07) d = 52 + Math.random() * 70;
    C.scrollTop = C.scrollHeight;
    await sleep(d);
  }
  bub.innerHTML = plain.replace(/\n/g, '<br>');
  C.scrollTop = C.scrollHeight;
}

function showTyping() {
  const C   = document.getElementById('msgs');
  const row = mk('div', 'msg bot'); row.id = 'typing';
  const av  = mk('div', 'av'); av.textContent = 'FC';
  const d   = mk('div', 'tdots');
  for (let i = 0; i < 3; i++) { const x = mk('div', 'td'); d.appendChild(x); }
  row.appendChild(av); row.appendChild(d); C.appendChild(row);
  setTimeout(() => C.scrollTop = C.scrollHeight, 40);
}
function removeTyping() { const e = document.getElementById('typing'); if (e) e.remove(); }

function userMsg(t) {
  const C   = document.getElementById('msgs');
  const row = mk('div', 'msg usr');
  const av  = mk('div', 'av'); av.textContent = 'YOU';
  const b   = mk('div', 'bub'); b.textContent = t;
  row.appendChild(av); row.appendChild(b); C.appendChild(row);
  setTimeout(() => C.scrollTop = C.scrollHeight, 40);
}

/* ═══════════════════════════════════════════════════════════════════
   INPUT COMPONENTS
   ═══════════════════════════════════════════════════════════════════ */
function showOpts(opts, onPick) {
  const wrap = mk('div', 'opt-wrap');
  const lbl  = mk('div', 'opt-lbl'); lbl.textContent = 'Select an option';
  const grid = mk('div', 'opt-grid');
  opts.forEach(o => {
    const b = mk('button', 'opt-btn' + (o.cls ? ' ' + o.cls : ''));
    b.textContent = o.label;
    b.onclick = () => {
      grid.querySelectorAll('.opt-btn').forEach(x => x.disabled = true);
      b.classList.add('sel');
      setTimeout(() => { clearIA(); onPick(o); }, 200);
    };
    grid.appendChild(b);
  });
  wrap.appendChild(lbl); wrap.appendChild(grid);
  clearIA(); document.getElementById('ia').appendChild(wrap);
}

function showTextInput(ph, skipLabel, onOK, onSkip) {
  const outer = mk('div', 'inp-outer');
  const row   = mk('div', 'inp-row');
  const inp   = mk('input'); inp.type = 'text'; inp.placeholder = ph;
  const btn   = mk('button', 'send-btn'); btn.textContent = 'Continue';
  const go = () => {
    const v = inp.value.trim();
    btn.disabled = true; inp.disabled = true; clearIA();
    if (v) onOK(v); else if (onSkip) onSkip();
  };
  inp.addEventListener('keydown', e => { if (e.key === 'Enter') go(); });
  btn.onclick = go;
  row.appendChild(inp); row.appendChild(btn); outer.appendChild(row);
  if (skipLabel && onSkip) {
    const sk = mk('button', 'skip-lnk'); sk.textContent = skipLabel;
    sk.onclick = () => { clearIA(); onSkip(); };
    outer.appendChild(sk);
  }
  clearIA(); document.getElementById('ia').appendChild(outer);
  setTimeout(() => inp.focus(), 100);
}

function showFollowUp(ph, onSend) {
  const wrap = mk('div', 'inline-inp');
  const inp  = mk('input'); inp.type = 'text'; inp.placeholder = ph;
  const btn  = mk('button', 'send-btn'); btn.textContent = 'Send';
  const go = () => {
    const v = inp.value.trim(); if (!v) return;
    btn.disabled = true; inp.disabled = true; onSend(v);
  };
  inp.addEventListener('keydown', e => { if (e.key === 'Enter') go(); });
  btn.onclick = go;
  wrap.appendChild(inp); wrap.appendChild(btn);
  clearIA(); document.getElementById('ia').appendChild(wrap);
  setTimeout(() => inp.focus(), 100);
}

/* ═══════════════════════════════════════════════════════════════════
   CLAUDE API
   ═══════════════════════════════════════════════════════════════════ */
async function callClaude(userText, extraSys) {
  if (!API_READY) throw new Error('Anthropic API key not configured.');
  const sys = `You are the Finite Concierge, the authoritative security advisor for Finite State.
Speak like a senior security engineer advising a peer. Be sharp, direct, technically credible. No fluffy marketing language, no em dashes, no bullet lists.
Visitor: ${S.role || 'security professional'}${S.company ? ' at ' + S.company : ''}. Branch: ${S.branch}. Focus: ${S.sub || 'general'}${S.sub2 ? ', ' + S.sub2 : ''}.
${extraSys || ''}`;
  const resp = await fetch(API, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': ANTHROPIC_API_KEY,
      'anthropic-version': '2023-06-01',
      'anthropic-dangerous-direct-browser-access': 'true'
    },
    body: JSON.stringify({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 1000,
      system: sys,
      messages: [...S.history, { role: 'user', content: userText }]
    })
  });
  if (!resp.ok) {
    const err = await resp.json().catch(() => ({}));
    throw new Error(err?.error?.message || `API error ${resp.status}`);
  }
  const d = await resp.json();
  let t = d.content?.[0]?.text || '';
  t = t.replace(/\u2014/g, ',').replace(/\u2013/g, ' to ');
  S.history.push({ role: 'user', content: userText });
  S.history.push({ role: 'assistant', content: t });
  return t;
}

/* ═══════════════════════════════════════════════════════════════════
   SALESFORCE WEB-TO-LEAD
   ═══════════════════════════════════════════════════════════════════ */
async function postToSalesforce(name, email, company, role, branch, focus) {
  if (!SF_READY) {
    console.warn('Salesforce: placeholder org ID — skipping post. Swap SF_ORG_ID to activate.');
    return { ok: false, reason: 'not_configured' };
  }
  const body = new URLSearchParams({
    oid:              SF_ORG_ID,
    retURL:           SF_RETURL,
    first_name:       name.split(' ')[0] || name,
    last_name:        name.split(' ').slice(1).join(' ') || '-',
    email:            email,
    company:          company    || 'Not provided',
    title:            role       || 'Not provided',
    lead_source:      'Finite Concierge',
    [SF_BRANCH_FIELD]: branch,
    [SF_FOCUS_FIELD]:  focus,
  });
  try {
    await fetch(SF_ENDPOINT, {
      method:  'POST',
      mode:    'no-cors',   // Web-to-Lead doesn't return CORS headers — silent post
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body:    body.toString()
    });
    console.log('Salesforce: Lead posted —', email, '|', branch, '|', role);
    return { ok: true };
  } catch (err) {
    console.error('Salesforce post error:', err);
    return { ok: false, reason: err.message };
  }
}

/* ═══════════════════════════════════════════════════════════════════
   EMAIL — dual fire via EmailJS + Salesforce post
   ═══════════════════════════════════════════════════════════════════ */
async function sendEmails(visitorName, visitorEmail, slot, specialist) {
  const focus = [S.sub, S.sub2].filter(Boolean).join(' / ');

  const sharedParams = {
    visitor_name:  visitorName,
    visitor_email: visitorEmail,
    company:       S.company || 'Not provided',
    role:          S.role    || 'Not provided',
    branch:        S.branch,
    focus:         focus,
    specialist,
    slot,
    notify_email:  NOTIFY_EMAIL,
  };

  // Fire EmailJS + Salesforce in parallel
  const results = await Promise.allSettled([
    CONFIGURED
      ? Promise.all([
          emailjs.send(EMAILJS_SERVICE_ID, EMAILJS_TEMPLATE_VISITOR,  { ...sharedParams, to_email: visitorEmail }),
          emailjs.send(EMAILJS_SERVICE_ID, EMAILJS_TEMPLATE_INTERNAL, { ...sharedParams, to_email: NOTIFY_EMAIL }),
        ])
      : Promise.resolve('not_configured'),
    postToSalesforce(visitorName, visitorEmail, S.company, S.role, S.branch, focus),
  ]);

  const emailOk = results[0].status === 'fulfilled';
  const sfOk    = results[1].status === 'fulfilled' && results[1].value?.ok;

  if (SF_READY) {
    console.log('Salesforce post:', sfOk ? 'success' : 'failed');
  }

  if (!CONFIGURED) return { ok: false, reason: 'not_configured' };
  if (!emailOk)    return { ok: false, reason: results[0].reason?.text || 'email_error' };
  return { ok: true };
}

/* ═══════════════════════════════════════════════════════════════════
   RESOURCES
   ═══════════════════════════════════════════════════════════════════ */
const RESOURCES = {
  builder: [
    { icon: '📄', type: 'Whitepaper', name: 'Deep Binary Analysis: Why Source-Focused SCA Falls Short for Embedded Systems', url: 'https://finitestate.io/resources' },
    { icon: '🎥', type: 'Webinar',    name: 'Securing IoT Products: Design, Verify, Prove', url: 'https://finitestate.io/security-shorts' },
    { icon: '📝', type: 'Blog',       name: 'Building Resilient IoT Products in an Era of Escalating Risk', url: 'https://finitestate.io/blog/secure-iot-product-development-strategies' },
  ],
  deployer: [
    { icon: '📄', type: 'Whitepaper', name: 'Managing Third-Party Device Risk Across the Enterprise', url: 'https://finitestate.io/resources' },
    { icon: '🎥', type: 'Webinar',    name: 'Vulnerability Monitoring for Deployed Connected Devices', url: 'https://finitestate.io/security-shorts' },
    { icon: '📝', type: 'Blog',       name: 'Software Supply Chain Security: What Asset Owners Need to Know', url: 'https://finitestate.io/blog' },
  ],
  compliance: [
    { icon: '📄', type: 'Datasheet',  name: 'Finite State Cybersecurity and Compliance Services', url: 'https://finitestate.io/resources/finite-state-cybersecurity-services' },
    { icon: '🎥', type: 'Webinar',    name: 'AI for Regulatory Demands: Bridging the Gap in Product Security', url: 'https://finitestate.io/security-shorts' },
    { icon: '📝', type: 'Blog',       name: 'Countdown to Compliance: Preparing for the EU CRA', url: 'https://finitestate.io/blog' },
  ],
};

function showResources(branch) {
  const C    = document.getElementById('msgs');
  const row  = mk('div', 'msg bot');
  const av   = mk('div', 'av'); av.textContent = 'FC';
  const wrap = mk('div', 'res-wrap');
  const lbl  = mk('div', 'res-section-lbl'); lbl.textContent = 'Relevant Resources';
  wrap.appendChild(lbl);
  RESOURCES[branch].forEach(r => {
    const a  = mk('a', 'res-card'); a.href = r.url; a.target = '_blank'; a.rel = 'noopener';
    const ic = mk('span', 'res-icon'); ic.textContent = r.icon;
    const info = mk('div', '');
    const tp = mk('div', 'res-type'); tp.textContent = r.type;
    const nm = mk('div', 'res-name'); nm.textContent = r.name;
    info.appendChild(tp); info.appendChild(nm);
    a.appendChild(ic); a.appendChild(info);
    wrap.appendChild(a);
  });
  row.appendChild(av); row.appendChild(wrap);
  C.appendChild(row);
  setTimeout(() => C.scrollTop = C.scrollHeight, 50);
}

/* ═══════════════════════════════════════════════════════════════════
   BOOKING CARD
   ═══════════════════════════════════════════════════════════════════ */
function timeSlots() {
  const days   = ['Sun','Mon','Tue','Wed','Thu','Fri','Sat'];
  const months = ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'];
  const now = new Date(); const slots = []; let offset = 1;
  while (slots.length < 4) {
    const d = new Date(now); d.setDate(d.getDate() + offset);
    if (d.getDay() !== 0 && d.getDay() !== 6) {
      const hours = slots.length % 2 === 0 ? 10 : 2;
      const ap    = slots.length % 2 === 0 ? 'AM' : 'PM';
      slots.push(`${days[d.getDay()]} ${months[d.getMonth()]} ${d.getDate()} · ${hours}:00 ${ap}`);
    }
    offset++;
  }
  return slots;
}

function showBookCard(specialist, colorCls) {
  const C    = document.getElementById('msgs');
  const row  = mk('div', 'msg bot');
  const av   = mk('div', 'av'); av.textContent = 'FC';
  const card = mk('div', `book-card${colorCls ? ' ' + colorCls : ''}`);

  card.innerHTML = `
    <div class="book-title">Book 30 · Show 15</div>
    <div class="book-who">Direct session with a <strong>${specialist}</strong>. No SDR. No discovery layer. Come ready to show your environment.</div>
    <div class="book-fields">
      <input type="text"  id="bkName"  placeholder="Your full name" />
      <input type="email" id="bkEmail" placeholder="Your work email" />
    </div>
    <div class="book-slots-lbl">Pick a time (your local timezone)</div>
    <div class="book-slots" id="slotGrid"></div>
    <div class="book-note">Calendar invite sent on confirmation.</div>
    <button class="book-submit" id="bkSubmit" disabled>Confirm Session</button>
    <div class="book-error" id="bkError">Something went wrong sending the confirmation email. Your session is still noted — we will follow up at the email you provided.</div>
  `;

  row.appendChild(av); row.appendChild(card);
  C.appendChild(row);
  setTimeout(() => C.scrollTop = C.scrollHeight, 50);

  const nameInp  = card.querySelector('#bkName');
  const emailInp = card.querySelector('#bkEmail');
  const grid     = card.querySelector('#slotGrid');
  const submitBtn= card.querySelector('#bkSubmit');
  const errEl    = card.querySelector('#bkError');
  let pickedSlot = null;

  const checkReady = () => {
    submitBtn.disabled = !(nameInp.value.trim() && emailInp.value.trim() && pickedSlot);
  };
  nameInp.addEventListener('input', checkReady);
  emailInp.addEventListener('input', checkReady);

  timeSlots().forEach(slot => {
    const s = mk('button', 'slot'); s.textContent = slot;
    s.onclick = () => {
      grid.querySelectorAll('.slot').forEach(x => x.classList.remove('picked'));
      s.classList.add('picked'); pickedSlot = slot; checkReady();
    };
    grid.appendChild(s);
  });

  submitBtn.onclick = async () => {
    const name  = nameInp.value.trim();
    const email = emailInp.value.trim();
    if (!name || !email || !pickedSlot) return;

    submitBtn.disabled    = true;
    submitBtn.textContent = 'Sending...';

    const result = await sendEmails(name, email, pickedSlot, specialist);

    if (result.ok || result.reason === 'not_configured') {
      submitBtn.textContent = 'Confirmed!';
      clearIA();
      userMsg(`${name} · ${pickedSlot}`);
      await sleep(400);
      showTyping(); await sleep(1700); removeTyping();

      const emailNote = result.reason === 'not_configured'
        ? `(Email service not yet active, but your booking is noted.)`
        : `A confirmation has been sent to ${email}.`;

      await type(`You are on the calendar. ${emailNote}\n\nYour ${specialist} will come prepared with context specific to ${S.company || 'your environment'}. If anything changes, reach us at security@finitestate.io.`);
      clearIA();
      document.getElementById('stepLbl').textContent = 'Complete';

    } else {
      submitBtn.textContent  = 'Confirmed';
      errEl.style.display    = 'block';
      clearIA();
      userMsg(`${name} · ${pickedSlot}`);
      await sleep(400);
      showTyping(); await sleep(1400); removeTyping();
      await type(`Your session is confirmed for ${pickedSlot}. We had a hiccup sending the confirmation email, so our team will follow up at ${email} manually. Apologies for the friction.`);
      clearIA();
      document.getElementById('stepLbl').textContent = 'Complete';
    }
  };
}

/* ═══════════════════════════════════════════════════════════════════
   FLOW
   ═══════════════════════════════════════════════════════════════════ */
async function start() {
  setStep(1, 6);
  await sleep(450);
  await type("Welcome to Finite State. How can we help you accelerate your product security today?");
  await sleep(140);
  showOpts([
    { label: '🔧  Secure devices we build',         value: 'builder',    cls: '' },
    { label: '🌐  Assess devices we deploy',         value: 'deployer',   cls: 'am' },
    { label: '📋  Streamline regulatory compliance', value: 'compliance', cls: 'pu' },
  ], async sel => {
    S.branch = sel.value;
    const labels = { builder: 'Device Builder', deployer: 'Asset Owner / Deployer', compliance: 'Compliance Officer' };
    setTrack(labels[sel.value]);
    userMsg(sel.label);
    await askRole();
  });
}

async function askRole() {
  setStep(2, 6);
  await sleep(700);
  await type("What is your role?");
  const roleOpts = {
    builder: [
      { label: 'Product Security Engineer / Manager', value: 'Product Security Engineer' },
      { label: 'CISO / VP of Security',               value: 'CISO' },
      { label: 'Firmware / Embedded Engineer',        value: 'Firmware Engineer' },
      { label: 'Engineering Leader / CTO',            value: 'Engineering Leader' },
    ],
    deployer: [
      { label: 'CISO / VP of Security',               value: 'CISO',                    cls: 'am' },
      { label: 'IT / Network Security Manager',       value: 'IT Security Manager',     cls: 'am' },
      { label: 'Risk and Compliance Lead',            value: 'Risk and Compliance Lead', cls: 'am' },
      { label: 'Engineering Leader / CTO',            value: 'Engineering Leader',       cls: 'am' },
    ],
    compliance: [
      { label: 'Regulatory Affairs Manager',          value: 'Regulatory Affairs Manager', cls: 'pu' },
      { label: 'CISO / VP of Security',               value: 'CISO',                       cls: 'pu' },
      { label: 'Product Security / Quality Lead',     value: 'Product Security Lead',      cls: 'pu' },
      { label: 'Legal / General Counsel',             value: 'Legal Counsel',              cls: 'pu' },
    ],
  };
  showOpts(roleOpts[S.branch], async sel => {
    S.role = sel.value;
    userMsg(sel.label);
    await askCompany();
  });
}

async function askCompany() {
  setStep(3, 6);
  await sleep(700);
  await type("And your company? It helps me pull the right context.");
  showTextInput('Company name...', 'Skip for now',
    async v => { S.company = v; userMsg(v); await branchSub(); },
    async () => { S.company = null; await branchSub(); }
  );
}

async function branchSub() {
  setStep(4, 6);
  await sleep(700);
  if (S.branch === 'builder') {
    await type("What is your biggest bottleneck right now?");
    showOpts([
      { label: 'SBOM Generation and Management',     value: 'SBOM generation' },
      { label: 'Deep Binary and Firmware Analysis',  value: 'binary analysis' },
      { label: 'CI/CD Pipeline Integration',         value: 'CI/CD integration' },
    ], async sel => { S.sub = sel.value; userMsg(sel.label); await painPointAI(); });
  } else if (S.branch === 'deployer') {
    await type("What is your primary concern right now?");
    showOpts([
      { label: 'Vendor Risk Management',                   value: 'vendor risk management',   cls: 'am' },
      { label: 'Vulnerability Monitoring and Mitigation',  value: 'vulnerability monitoring', cls: 'am' },
    ], async sel => { S.sub = sel.value; userMsg(sel.label); await painPointAI(); });
  } else if (S.branch === 'compliance') {
    await type("Which mandate are you solving for right now?");
    showOpts([
      { label: 'FDA Medical Device Regulations (524B)', value: 'FDA 524B',  cls: 'pu' },
      { label: 'EU Cyber Resilience Act (CRA)',          value: 'EU CRA',   cls: 'pu' },
      { label: 'US EO 14028 / NIST SSDF',               value: 'EO 14028', cls: 'pu' },
      { label: 'NIS 2 Directive',                        value: 'NIS 2',    cls: 'pu' },
    ], async sel => { S.sub = sel.value; userMsg(sel.label); await complianceDepth(); });
  }
}

async function complianceDepth() {
  await sleep(700);
  const msgs = {
    'FDA 524B': "FDA 524B requires manufacturers to submit and maintain an SBOM. Are you focused on pre-market submissions or post-market surveillance?",
    'EU CRA':   "The CRA requires continuous SBOMs and documented vulnerability handling to keep EU market access. Are you at the design stage or already shipping into the EU?",
    'EO 14028': "EO 14028 and the NIST SSDF require attestation of secure development practices including SBOM generation. Is your main challenge generating the SBOM or managing vulnerability disclosure?",
    'NIS 2':    "NIS 2 places strict incident reporting and supply chain risk requirements on critical infrastructure operators. Are you already a covered entity or preparing for potential scope expansion?",
  };
  const subOpts = {
    'FDA 524B': [{ label: 'Pre-market submissions',   value: 'pre-market submissions',  cls: 'pu' }, { label: 'Post-market surveillance', value: 'post-market surveillance', cls: 'pu' }],
    'EU CRA':   [{ label: 'Design stage',             value: 'design stage',             cls: 'pu' }, { label: 'Already shipping to EU',   value: 'shipping to EU',          cls: 'pu' }],
    'EO 14028': [{ label: 'Generating the SBOM',      value: 'SBOM generation',          cls: 'pu' }, { label: 'Vulnerability disclosure', value: 'vulnerability disclosure', cls: 'pu' }],
    'NIS 2':    [{ label: 'Already a covered entity', value: 'covered entity',            cls: 'pu' }, { label: 'Preparing for expansion',  value: 'preparing for expansion', cls: 'pu' }],
  };
  await type(msgs[S.sub]);
  showOpts(subOpts[S.sub], async sel => { S.sub2 = sel.value; userMsg(sel.label); await painPointAI(); });
}

async function painPointAI() {
  setStep(5, 6);
  await sleep(600);
  showTyping();
  let reply;
  try {
    reply = await callClaude(
      `In 2 short paragraphs, describe the specific risk this person is carrying right now based on their profile. Then in one sentence explain how Finite State resolves it. Be precise and technically grounded. Keep the total under 90 words. End with a new line then ask: "Does this capture what you are dealing with, or should we adjust?"`,
      `No bullet lists. No em dashes. No fluffy language. Write as a peer.`
    );
  } catch (e) { reply = fallbackPain(); }
  removeTyping();
  await type(reply);
  await sleep(200);
  showOpts([
    { label: 'Yes, that is accurate',     value: 'confirmed' },
    { label: 'Not quite, let me clarify', value: 'redirect'  },
  ], async sel => {
    userMsg(sel.label);
    if (sel.value === 'confirmed') await goToResources();
    else await askClarification();
  });
}

function fallbackPain() {
  const map = {
    builder:    "Without binary-level visibility across every component in your firmware, you are shipping unknowns into the field. Finite State scans at the binary level and generates SBOMs automatically so vulnerabilities are caught before they reach customers.\n\nDoes this capture what you are dealing with, or should we adjust?",
    deployer:   "Every device on your network carries software you did not write and cannot fully see. Finite State gives you continuous SBOM visibility across your device inventory so you know which vendors carry active CVEs before they become incidents.\n\nDoes this capture what you are dealing with, or should we adjust?",
    compliance: "Regulators are not asking whether you have a security program. They are asking whether you can prove it at the component level, continuously. Finite State automates the SBOM generation and vulnerability tracking that forms the evidentiary backbone for your mandate.\n\nDoes this capture what you are dealing with, or should we adjust?",
  };
  return map[S.branch] || map.builder;
}

async function askClarification() {
  await sleep(600);
  await type("No problem. Tell me more about what you are actually running into.");
  showFollowUp("Describe your situation...", async val => {
    userMsg(val);
    S.history.push({ role: 'user', content: `Visitor clarification: ${val}` });
    await sleep(600);
    showTyping();
    let reply;
    try {
      reply = await callClaude(
        `The visitor corrected the assessment. Their clarification: "${val}". Revise the pain point assessment in 2 short paragraphs based on this context. Keep it under 80 words. End with: "Does this better reflect what you are up against?"`,
        `Be precise and technical. No fluffy language.`
      );
    } catch (e) {
      reply = "Understood. " + fallbackPain().replace("Does this capture what you are dealing with, or should we adjust?", "Does this better reflect what you are up against?");
    }
    removeTyping();
    await type(reply);
    await sleep(200);
    showOpts([
      { label: 'Yes, that is right',     value: 'confirmed' },
      { label: 'Let me clarify further', value: 'redirect'  },
    ], async sel => {
      userMsg(sel.label);
      if (sel.value === 'confirmed') await goToResources();
      else await askClarification();
    });
  });
}

async function goToResources() {
  setStep(6, 6);
  await sleep(700);
  await type("Good. Before we get to next steps, here are a few resources directly relevant to your situation.");
  await sleep(300);
  showResources(S.branch);
  await sleep(700);
  await type("Want to go deeper? I can put you directly on the calendar with the right specialist. No SDR queue.");
  await sleep(200);
  const colorCls   = { builder: '', deployer: 'am', compliance: 'pu' };
  const specialist = { builder: 'Product Security Engineer', deployer: 'Asset Risk Specialist', compliance: 'Compliance Specialist' };
  showOpts([
    { label: 'Yes, book a session',               value: 'book', cls: S.branch === 'deployer' ? 'am' : S.branch === 'compliance' ? 'pu' : '' },
    { label: 'Not right now, I have what I need', value: 'done' },
  ], async sel => {
    userMsg(sel.label);
    clearIA();
    if (sel.value === 'book') {
      await sleep(400);
      showBookCard(specialist[S.branch], colorCls[S.branch]);
    } else {
      await sleep(400);
      await type("Understood. Everything you need is in the resources above. When you are ready, come back here and we will get you connected.\n\nYou can also reach us directly at security@finitestate.io.");
      document.getElementById('stepLbl').textContent = 'Complete';
    }
  });
}

function restart() {
  S = { branch: null, sub: null, sub2: null, role: null, company: null, history: [] };
  document.getElementById('msgs').innerHTML = '';
  clearIA();
  document.getElementById('restartBtn').classList.remove('on');
  setTrack('');
  start();
}

window.addEventListener('load', () => setTimeout(start, 400));
</script>
</body>
</html>
