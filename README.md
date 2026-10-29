# ZailEdge
my earning app <!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ZailEdge - AI Assistant</title>
<style>
:root{
  --bg:#0a1020; --card:#121c35; --input:#1a274d;
  --blue:#2563eb; --light-blue:#60a5fa; --text:#e6edff;
}
*{margin:0;padding:0;box-sizing:border-box;font-family:sans-serif}
body{background:var(--bg);color:var(--text);height:100vh;display:flex;flex-direction:column}
header{
  height:60px;background:var(--card);display:flex;align-items:center;
  justify-content:space-between;padding:0 15px;border-bottom:1px solid #1e2f5a
}
.logo-area{display:flex;align-items:center;gap:10px;font-weight:700;font-size:18px}
.rotating-logo{
  width:30px;height:30px;border-radius:50%;
  background:var(--blue);border:3px solid var(--light-blue);
  display:flex;align-items:center;justify-content:center;
  animation:spin 0.2s linear infinite;
  box-shadow:0 0 10px var(--light-blue), inset 0 0 8px #93c5fd;
  position:relative;
}
.rotating-logo::before{content:"⚡";font-size:16px;color:white}
@keyframes spin{100%{transform:rotate(360deg)}}
.header-btns button{background:var(--input);color:var(--text);border:1px solid #2a3d6b;padding:6px 10px;border-radius:8px;margin-left:5px}
#chat{flex:1;overflow-y:auto;padding:15px;display:flex;flex-direction:column;gap:12px}
.bubble{max-width:85%;padding:10px 14px;border-radius:16px;line-height:1.4;font-size:15px}
.user{align-self:flex-end;background:var(--blue);border-bottom-right-radius:4px}
.ai{align-self:flex-start;background:var(--card);border:1px solid #1e2f5a;border-bottom-left-radius:4px}
.bubble img{max-width:100%;border-radius:10px;margin-top:6px}
.bubble video{max-width:100%;border-radius:10px;margin-top:6px}
.audio-player{margin-top:6px;display:flex;align-items:center;gap:8px}
.typing{display:flex;gap:4px;padding:10px}
.typing span{width:6px;height:6px;background:var(--light-blue);border-radius:50%;animation:bounce 1s infinite}
.typing span:nth-child(2){animation-delay:.2s}.typing span:nth-child(3){animation-delay:.4s}
@keyframes bounce{0%,100%{transform:translateY(0)}50%{transform:translateY(-4px)}}
#preview-area{background:var(--card);padding:8px;display:none;align-items:center;gap:10px;border-top:1px solid #1e2f5a}
#preview-area img,#preview-area video{width:60px;height:60px;border-radius:8px;object-fit:cover}
.input-area{background:var(--card);padding:10px;display:flex;align-items:center;gap:8px;border-top:1px solid #1e2f5a}
.icon-btn{background:var(--input);border:none;width:40px;height:40px;border-radius:50%;color:var(--light-blue);font-size:20px;display:flex;align-items:center;justify-content:center}
#msgInput{flex:1;background:var(--input);border:1px solid #2a3d6b;border-radius:25px;padding:12px 15px;color:white;outline:none}
#sendBtn{background:var(--blue);border:none;width:44px;height:44px;border-radius:50%;color:white;font-size:18px}
#recording-ui{display:none;flex:1;background:#b91c1c;border-radius:25px;padding:10px 15px;align-items:center;gap:10px}
.wave{flex:1;height:20px;background:linear-gradient(90deg,var(--light-blue) 20%, transparent 20%);background-size:15px 100%;animation:waveMove .5s linear infinite}
@keyframes waveMove{0%{background-position:0}100%{background-position:15px}}
</style>
</head>
<body>
<header>
  <div class="logo-area">
    <div class="rotating-logo"></div>
    ZailEdge
  </div>
  <div class="header-btns">
    <button onclick="newChat()">+ New</button>
    <button onclick="clearChat()">Clear</button>
    <button onclick="showPolicy()">Privacy</button>
  </div>
</header>

<div id="chat">
  <div class="bubble ai">Assalam-o-Alaikum! I am ZailEdge AI. Photo, Video aur Voice bhej sakte ho. Kaise madad karun?</div>
</div>

<div id="preview-area">
  <div id="preview-content"></div>
  <button onclick="cancelPreview()" style="margin-left:auto;background:red;color:white;border:none;border-radius:50%;width:28px;height:28px">X</button>
</div>

<div class="input-area">
  <button class="icon-btn" onclick="document.getElementById('fileInput').click()">📎</button>
  <input type="file" id="fileInput" accept="image/*,video/*" hidden onchange="handleFile(this.files[0])">
  
  <button class="icon-btn" id="micBtn" ontouchstart="startRec()" ontouchend="stopRec()" onmousedown="startRec()" onmouseup="stopRec()">🎤</button>
  
  <div id="recording-ui">
    <span id="timer">00:00</span><div class="wave"></div><span>Recording...</span>
  </div>
  
  <input id="msgInput" placeholder="Ask anything..." onkeypress="if(event.key==='Enter')sendMsg()">
  <button id="sendBtn" onclick="sendMsg()">➤</button>
</div>

<script>
// --- CONFIG ---
const API_KEY = localStorage.getItem('zail_api_key') || "";
const SYSTEM_PROMPT = `You are ZailEdge AI. RULES: Never provide instructions for illegal activity, hacking, drugs, weapons, crime. Never provide hate, sexual, harmful content. If asked illegal, reply: 'Sorry, I cannot help with illegal or harmful requests. I can help you with legal and useful information about ZailEdge.' Always give safe legal answers.`;

let selectedFile = null, mediaRecorder, audioChunks=[], recStartTime, timerInterval;

function newChat(){ document.getElementById('chat').innerHTML='<div class="bubble ai">New chat started. How can I help?</div>' }
function clearChat(){ if(confirm('Clear chat?')) newChat() }
function showPolicy(){ alert(`Privacy Policy - ZailEdge\n\n1. We collect messages & media you send.\n2. Camera/Gallery only accessed when you tap attachment.\n3. Media stored securely.\n4. We never sell your data.\nContact: support@zailedge.com`) }

function handleFile(file){
  if(!file) return;
  selectedFile = file;
  const area = document.getElementById('preview-area');
  const content = document.getElementById('preview-content');
  area.style.display='flex';
  if(file.type.startsWith('image')){
    content.innerHTML=`<img src="${URL.createObjectURL(file)}"> <small>${file.name}</small>`;
  }else{
    content.innerHTML=`<video src="${URL.createObjectURL(file)}"></video> <small>${file.name}</small>`;
  }
}
function cancelPreview(){ selectedFile=null; document.getElementById('preview-area').style.display='none'; }

async function sendMsg(){
  const input=document.getElementById('msgInput');
  const text=input.value.trim();
  if(!text && !selectedFile) return;
  
  const chat=document.getElementById('chat');
  // Show user bubble
  let html = text ? `<div>${text}</div>` : "";
  if(selectedFile){
    const url=URL.createObjectURL(selectedFile);
    if(selectedFile.type.startsWith('image')) html+=`<img src="${url}">`;
    else html+=`<video src="${url}" controls></video>`;
  }
  chat.innerHTML+=`<div class="bubble user">${html}</div>`;
  input.value=""; cancelPreview();
  chat.scrollTop=chat.scrollHeight;

  // Typing indicator
  const typingId='typing-'+Date.now();
  chat.innerHTML+=`<div id="${typingId}" class="bubble ai typing"><span></span><span></span><span></span></div>`;
  chat.scrollTop=chat.scrollHeight;

  // --- AI CALL ---
  let aiResponse = "";
  try{
    if(!API_KEY){ throw "No API Key" }
    // Gemini API call (Secure: Key should be in backend, but for beginner we use direct)
    const res = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${API_KEY}`,{
      method:'POST',headers:{'Content-Type':'application/json'},
      body:JSON.stringify({contents:[{parts:[{text: SYSTEM_PROMPT + "\nUser: " + text}]}]})
    });
    const data=await res.json();
    aiResponse = data.candidates?.[0]?.content?.parts?.[0]?.text || "Sorry, AI response nahi mila.";
  }catch(e){
    aiResponse = `⚠️ API Key connect nahi hai. Free Gemini Key ke liye Google AI Studio me jao. Ya demo reply: "${text}" ka safe jawab ye hai ke mai ZailEdge hun aur legal madad kar sakta hun.`;
  }
  
  document.getElementById(typingId).remove();
  // Safety filter check
  if(/bomb|hack|drug|weapon|illegal/i.test(text.toLowerCase())){
    aiResponse="Sorry, I cannot help with illegal or harmful requests. I can help you with legal and useful information about ZailEdge.";
  }
  chat.innerHTML+=`<div class="bubble ai">${aiResponse}</div>`;
  chat.scrollTop=chat.scrollHeight;
}

// --- VOICE RECORDING ---
async function startRec(){
  try{
    const stream=await navigator.mediaDevices.getUserMedia({audio:true});
    mediaRecorder=new MediaRecorder(stream); audioChunks=[];
    mediaRecorder.ondataavailable=e=>audioChunks.push(e.data);
    mediaRecorder.onstop=()=>{
      const blob=new Blob(audioChunks,{type:'audio/webm'});
      const url=URL.createObjectURL(blob);
      const chat=document.getElementById('chat');
      chat.innerHTML+=`<div class="bubble user"><div class="audio-player"><button onclick="new Audio('${url}').play()">▶️</button> Voice Message</div><audio src="${url}" controls style="width:100%;margin-top:5px"></audio></div>`;
      chat.scrollTop=chat.scrollHeight;
    };
    mediaRecorder.start();
    document.getElementById('recording-ui').style.display='flex';
    document.getElementById('msgInput').style.display='none';
    recStartTime=Date.now();
    timerInterval=setInterval(()=>{
      const s=Math.floor((Date.now()-recStartTime)/1000);
      document.getElementById('timer').innerText=`00:${s<10?'0'+s:s}`;
    },1000);
  }catch{ alert('Mic permission do!') }
}
function stopRec(){
  if(mediaRecorder && mediaRecorder.state==='recording'){
    mediaRecorder.stop();
    clearInterval(timerInterval);
    document.getElementById('recording-ui').style.display='none';
    document.getElementById('msgInput').style.display='block';
  }
}

// Ask for API key first time
if(!API_KEY){
  const k=prompt("Apna FREE Gemini API Key dalo (Google AI Studio se milegi). Cancel karo to demo mode chalega:");
  if(k) localStorage.setItem('zail_api_key', k.trim());
}
</script>
</body>
</html>
