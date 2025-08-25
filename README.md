# HTML-ai-opensource
# This open-source kit provides a simple HTML + JavaScript interface for experimenting with AI. It’s customizable, and perfect for learning or building your own AI-powered web projects.​

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Clashnewbmes AI HTML Opensource!</title>
<style>
  :root {
    --bg: #0f1220; --card:#171a2b; --muted:#8a90b0; --text:#e7ebff; --accent:#6c8cff; --accent-2:#b27cff;
  }
  *{box-sizing:border-box}
  body{margin:0;background:radial-gradient(1200px 600px at 20% -10%, #1a1f3a, transparent 60%), var(--bg);
       color:var(--text);font-family:system-ui,-apple-system,Segoe UI,Roboto,Inter,Arial,sans-serif;min-height:100vh;display:flex}
  .wrap{max-width:900px;margin:0 auto;display:flex;flex-direction:column;gap:14px;flex:1;padding:18px}
  header{display:flex;align-items:center;gap:12px}
  .logo{width:38px;height:38px;border-radius:12px;background:
        conic-gradient(from 210deg at 50% 50%, var(--accent), var(--accent-2), #63e, var(--accent));
        box-shadow:0 6px 24px rgba(108,140,255,.3)}
  h1{font-size:20px;margin:0}
  .sub{color:var(--muted);font-size:13px;margin-top:2px}
  .card{background:rgba(23,26,43,.7);backdrop-filter: blur(10px);border:1px solid rgba(255,255,255,.06);
        border-radius:16px}
  .chat{flex:1;min-height:280px;padding:14px;overflow:auto;scroll-behavior:smooth}
  .row{display:flex;gap:10px;margin:8px 0;align-items:flex-end}
  .row.ai .bubble{background:#1e2238;border:1px solid rgba(255,255,255,.05)}
  .row.you{justify-content:flex-end}
  .avatar{width:30px;height:30px;border-radius:50%;background:#303859;flex:0 0 30px}
  .bubble{max-width:70%;padding:10px 12px;border-radius:14px 14px 14px 6px;background:#2a3052;
          line-height:1.4;font-size:14px;white-space:pre-wrap;word-wrap:break-word}
  .row.you .bubble{border-radius:14px 14px 6px 14px;background:linear-gradient(135deg,#5566ff,#9c6bff)}
  .controls{display:flex;gap:10px;padding:10px}
  input[type="text"]{flex:1;padding:12px 14px;border-radius:12px;border:1px solid rgba(255,255,255,.08);
                     background:#14182c;color:var(--text);outline:none}
  button{padding:10px 14px;border-radius:12px;border:1px solid rgba(255,255,255,.1);
         background:#1b2140;color:var(--text);cursor:pointer;transition:.2s}
  button:hover{transform:translateY(-1px)}
  .bar{display:flex;gap:8px;flex-wrap:wrap}
  .pill{font-size:12px;padding:6px 9px;border-radius:999px;border:1px dashed rgba(255,255,255,.15);color:var(--muted)}
  footer{display:flex;justify-content:space-between;align-items:center;color:var(--muted);font-size:12px;padding:0 6px 10px}
  .typing{display:inline-block;min-width:16px}
  .dot{display:inline-block;width:5px;height:5px;margin:0 1px;border-radius:50%;background:#9fb0ff;opacity:.4;animation:b 1.2s infinite}
  .dot:nth-child(2){animation-delay:.15s}.dot:nth-child(3){animation-delay:.3s}
  @keyframes b{0%,80%,100%{transform:translateY(0);opacity:.3}40%{transform:translateY(-4px);opacity:1}}
  .small{font-size:12px;color:var(--muted)}
  .split{display:flex;gap:10px;flex-wrap:wrap}
</style>
</head>
<body>
<div class="wrap">
  <header>
    <div class="logo" aria-hidden="true"></div>
    <div>
      <h1>Clashnewbmes AI HTML Opensource!</h1>
      <div class="sub">A cool AI created by clashnewbme</div>
    </div>
  </header>

  <div class="card bar" style="padding:10px">
    <span class="pill">Try: “what’s 14*37?”</span>
    <span class="pill">“remember: my name is Alex”</span>
    <span class="pill">“teach when I say ‘knock knock’ reply ‘who’s there?’”</span>
    <span class="pill">“flip a coin”</span>
    <span class="pill">“what time is it”</span>
  </div>

  <div id="chat" class="card chat" role="log" aria-live="polite"></div>

  <div class="card controls">
    <input id="input" type="text" placeholder="Type a message… (Enter to send)" autocomplete="off" />
    <button id="send">Send</button>
    <button id="export">Export Memory</button>
  </div>
  <footer>
    <span>Local memory: <span id="memCount">0</span> items</span>
    <label class="small"><input id="tts" type="checkbox" /> Speak replies</label>
  </footer>
</div>

<script>
/* -------- Created by clashnewbme! join my discord ------- */
const elChat = document.getElementById('chat');
const elInput = document.getElementById('input');
const elSend = document.getElementById('send');
const elExport = document.getElementById('export');
const elMemCount = document.getElementById('memCount');
const elTTS = document.getElementById('tts');

const store = {
  get() { return JSON.parse(localStorage.getItem('ai_memory') || '{"pairs":[],"profile":{}}'); },
  set(v) { localStorage.setItem('ai_memory', JSON.stringify(v)); refreshMemCount(); },
  addPair(trig, resp) {
    const m = store.get(); m.pairs.push({t: normalize(trig), r: resp}); store.set(m);
  },
  setProfile(k,v){ const m=store.get(); m.profile[k]=v; store.set(m); },
};
function refreshMemCount(){ elMemCount.textContent = store.get().pairs.length + Object.keys(store.get().profile).length; }

// Basic utilities
function normalize(s){ return (s||'').toLowerCase().replace(/[^a-z0-9\s\.\,\-\!\?]/g,' ').replace(/\s+/g,' ').trim(); }
function sleep(ms){ return new Promise(r=>setTimeout(r,ms)); }

// Very small fuzzy match (Levenshtein distance with early exit)
function distance(a,b,max=Math.ceil(Math.max(a.length,b.length)/3)){
  if(Math.abs(a.length-b.length)>max) return max+1;
  const dp=Array(b.length+1).fill(0).map((_,i)=>i);
  for(let i=1;i<=a.length;i++){
    let prev=dp[0]; dp[0]=i;
    for(let j=1;j<=b.length;j++){
      const tmp=dp[j];
      dp[j]=Math.min(dp[j]+1, dp[j-1]+1, prev + (a[i-1]===b[j-1]?0:1));
      prev=tmp;
    }
    if(Math.min(...dp)>max) return max+1;
  }
  return dp[b.length];
}

// Intent rules
const intents = [
  { name:'greet', test:q=>/\b(hi|hello|hey|yo|sup)\b/.test(q), fn:()=>pick([
      "Hey! What can I help with?",
      "Hello! Ask me anything.",
      "Yo! Need math, time, or a coin flip?"
  ])},
  { name:'name-remember', test:q=>/\b(my name is|remember: my name is)\b/.test(q), fn:q=>{
      const m=q.match(/\bname is ([a-z0-9\-\_ ]+)/);
      if(m){ const name=title(m[1]); store.setProfile('name', name); return `Nice to meet you, ${name}! I'll remember.`; }
      return "Tell me your name like: “my name is Alex”.";
  }},
  { name:'time', test:q=>/\b(time|date)\b/.test(q) && /\bwhat\b/.test(q), fn:()=>{
      const now=new Date(); return `It's ${now.toLocaleTimeString()} on ${now.toLocaleDateString()}.`;
  }},
  { name:'math', test:q=>/^[^a-z]*[\d\.\s\+\-\*\/\(\)\%]+[^a-z]*$/.test(q) || /\bcalculate|what'?s|solve\b/.test(q)&&/[\d\+\-\*\/\%]/.test(q), fn:q=>{
      try{
        const expr = (q.match(/([0-9\.\s\+\-\*\/\%\(\)]+)/)?.[1] || q).replace(/[^0-9\.\+\-\*\/\%\(\)\s]/g,'');
        const val = Function(`"use strict"; return (${expr})`)();
        if(typeof val==='number' && isFinite(val)) return `${expr} = ${val}`;
      }catch(e){}
      return "I couldn't parse that math. Try: 14*37 or (2+3)/5.";
  }},
  { name:'coin', test:q=>/\bflip (a )?coin\b/.test(q), fn:()=>Math.random()<0.5?'Heads!':'Tails!'},
  { name:'dice', test:q=>/\broll (a )?d(ice|[0-9]+)\b/.test(q), fn:q=>{
      const n=Number(q.match(/d([0-9]+)/)?.[1]||'6'); return `You rolled ${1+Math.floor(Math.random()*Math.max(2,n))} on a d${Math.max(2,n)}.`;
  }},
  { name:'teach', test:q=>/\bteach( me)? when I say\b/.test(q) || /\b^when I say\b/.test(q), fn:q=>{
      const m = q.match(/say (.+?) (?:reply|respond|answer|say back) (.+)$/);
      if(m){ store.addPair(m[1], m[2]); return `Got it. When you say “${m[1]}”, I will reply “${m[2]}”.`; }
      return `Format: teach when I say <trigger> reply <response>`;
  }},
  { name:'forget', test:q=>/\bforget (everything|my name|memory)\b/.test(q), fn:q=>{
      const m=store.get(); const was=m.pairs.length + Object.keys(m.profile).length;
      localStorage.removeItem('ai_memory'); refreshMemCount(); return `Memory cleared (${was} items).`;
  }},
  { name:'weather-demo', test:q=>/\bweather\b/.test(q), fn:()=>`I can't pull real weather yet, but here's a fake forecast: Sunny, 76°F high, 61°F low, light breeze.`},
  { name:'whoami', test:q=>/\bwho am i|my name\?\b/.test(q), fn:()=>{
      const name=store.get().profile.name; return name ? `You're ${name}!` : `I don't know yet. Tell me: “my name is …”`;
  }},
];

function pick(arr){ return arr[Math.floor(Math.random()*arr.length)]; }
function title(s){ return s.split(' ').map(w=>w[0]?w[0].toUpperCase()+w.slice(1):'').join(' '); }

// Memory lookup: exact + fuzzy
function memoryReply(q){
  const mem = store.get().pairs;
  const nq = normalize(q);
  let best = null;
  for(const p of mem){
    const d = distance(nq, p.t);
    if(d <= Math.ceil(p.t.length/4)){
      if(!best || d < best.d) best = {d, r:p.r, t:p.t};
    }
  }
  return best?.r || null;
}

// falback responses
const fallbacks = [
  "I'm not sure yet—teach me! Try: teach when I say X reply Y.",
  "Can you rephrase that?",
  "I can do math, time, coin flips, dice, and learn custom replies.",
  "Good question. What would you like me to answer to that next time?"
];

// Chat ui
function addMsg(text, who='ai'){
  const row = document.createElement('div');
  row.className = 'row ' + (who==='you'?'you':'ai');
  const avatar = document.createElement('div'); avatar.className='avatar';
  const bubble = document.createElement('div'); bubble.className='bubble'; bubble.textContent = text;
  if(who==='you'){ row.appendChild(bubble); row.appendChild(avatar); }
  else { row.appendChild(avatar); row.appendChild(bubble); }
  elChat.appendChild(row);
  elChat.scrollTop = elChat.scrollHeight;
  if(who==='ai' && elTTS.checked && 'speechSynthesis' in window){
    const u = new SpeechSynthesisUtterance(text); window.speechSynthesis.speak(u);
  }
}

function addTyping(){
  const row=document.createElement('div'); row.className='row ai'; row.id='typing';
  const avatar=document.createElement('div'); avatar.className='avatar';
  const bubble=document.createElement('div'); bubble.className='bubble';
  const wrap=document.createElement('span'); wrap.className='typing';
  wrap.innerHTML='<span class="dot"></span><span class="dot"></span><span class="dot"></span>';
  bubble.appendChild(wrap); row.appendChild(avatar); row.appendChild(bubble);
  elChat.appendChild(row); elChat.scrollTop=elChat.scrollHeight;
}
function removeTyping(){ const t=document.getElementById('typing'); if(t) t.remove(); }

// parse and then respond
async function respond(userText){
  const raw=userText;
  const q = normalize(raw);

  let ans = memoryReply(q);
  if(!ans){
    for(const it of intents){ try{ if(it.test(q)){ ans = await it.fn(q); break; } }catch(e){} }
  }
  const name = store.get().profile.name;
  if(!ans){
    const maybe = pick(fallbacks);
    ans = name ? maybe.replace('Good question','Good question, '+name) : maybe;
  }
  ans = ans.replace(/\bai\b/gi,'AI');
  return ans;
}

// handlers
async function onSend(){
  const text = elInput.value.trim();
  if(!text) return;
  addMsg(text, 'you');
  elInput.value='';
  addTyping();
  await sleep(300 + Math.random()*500);
  const reply = await respond(text);
  removeTyping();
  addMsg(reply, 'ai');
}
elSend.addEventListener('click', onSend);
elInput.addEventListener('keydown', e=>{ if(e.key==='Enter' && !e.shiftKey){ onSend(); }});
elExport.addEventListener('click', ()=>{
  const blob = new Blob([JSON.stringify(store.get(),null,2)], {type:'application/json'});
  const url = URL.createObjectURL(blob);
  const a = Object.assign(document.createElement('a'), {href:url, download:'ai-memory.json'});
  document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
});

// First run greeting
(function init(){
  refreshMemCount();
  if(!sessionStorage.getItem('greeted')){
    sessionStorage.setItem('greeted','1');
    addMsg("Hi! I'm your small, simple AI. I can do quick math, time, coin flips, dice, and you can TEACH me custom replies. I was created by clashnewbme.\n\nExamples:\n• my name is Alex\n• teach when I say knock knock reply who's there?\n• what’s 14*37?\n• flip a coin");
  }
})();
</script>
</body>
</html>
