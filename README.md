<!DOCTYPE html><html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Deobfuscator</title><script src="https://cdn.tailwindcss.com"></script><style>
body {
  margin: 0;
  background: radial-gradient(circle at top, #0b1220, #050814);
  color: white;
  font-family: ui-sans-serif, system-ui;
}

.glass {
  background: rgba(255,255,255,0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255,255,255,0.08);
}

textarea {
  font-family: monospace;
}

::-webkit-scrollbar {
  width: 6px;
}
::-webkit-scrollbar-thumb {
  background: #1f2937;
  border-radius: 10px;
}
</style></head><body class="min-h-screen p-6"><!-- HEADER --><div class="mb-6">
  <h1 class="text-2xl font-bold text-sky-400">Deobfuscator</h1>
  <p class="text-xs opacity-60">Multi-layer safe deobfuscation + formatter (no crash engine)</p>
</div><!-- GRID --><div class="grid grid-cols-1 md:grid-cols-2 gap-4">  <div class="glass p-4 rounded-xl">
    <div class="text-sm text-sky-300 mb-2">Input Code</div>
    <textarea id="input" class="w-full h-96 p-3 rounded bg-black/40 outline-none"></textarea>
  </div>  <div class="glass p-4 rounded-xl">
    <div class="text-sm text-green-300 mb-2">Output</div>
    <textarea id="output" class="w-full h-96 p-3 rounded bg-black/40 outline-none" readonly></textarea>
  </div></div><!-- BUTTONS --><div class="flex gap-2 mt-4 flex-wrap">
  <button onclick="deobfuscate()" class="px-4 py-2 rounded bg-sky-500 text-black font-bold">Deobfuscate</button>
  <button onclick="copyOutput()" class="px-4 py-2 rounded bg-green-500 text-black font-bold">Copy</button>
  <button onclick="clearAll()" class="px-4 py-2 rounded bg-red-500 text-black font-bold">Clear</button>
</div><!-- HISTORY --><div class="glass p-4 rounded-xl mt-6">
  <div class="text-lg mb-2">History</div>
  <div id="history" class="space-y-2 max-h-64 overflow-auto"></div>
</div><script>
let historyData = [];

// prevent full crash
window.onerror = function(){ return true; }

// safe base64
function safeBase64(str){
  try { return atob(str.trim()); } catch { return null; }
}

// safe uri decode
function safeURI(str){
  try { return decodeURIComponent(str); } catch { return null; }
}

// detect hex strings
function hexDecode(str){
  try {
    if(/^([0-9a-fA-F]{2})+$/.test(str.trim())){
      let out = '';
      for(let i=0;i<str.length;i+=2){
        out += String.fromCharCode(parseInt(str.substr(i,2),16));
      }
      return out;
    }
  } catch {}
  return null;
}

// multi-layer unpacker
function multiDecode(input){
  let current = input;

  for(let i=0;i<3;i++){

    let b64 = safeBase64(current);
    if(b64){ current = b64; continue; }

    let uri = safeURI(current);
    if(uri && uri !== current){ current = uri; continue; }

    let hex = hexDecode(current);
    if(hex){ current = hex; continue; }

    break;
  }

  return current;
}

// cleanup
function clean(code){
  return code
    .replace(/\\n/g,"\n")
    .replace(/\\t/g,"    ")
    .replace(/\beval\b/g,"/*eval*/")
    .replace(/\bloadstring\b/g,"/*loadstring*/");
}

// indentation
function indent(code){
  let lvl=0, out="";
  code.split(/\n/).forEach(l=>{
    l=l.trim(); if(!l) return;

    if(l.includes("end") || l.includes("}") ) lvl=Math.max(lvl-1,0);

    out += "  ".repeat(lvl)+l+"\n";

    if(l.includes("function") || l.includes("{") || l.includes("do")) lvl++;
  });
  return out;
}

function deobfuscate(){
  let code = document.getElementById("input").value;

  let decoded = multiDecode(code);

  let output = clean(decoded);

  try {
    output = indent(output);
  } catch {}

  document.getElementById("output").value = output;

  historyData.unshift({input: code, output});
  if(historyData.length > 25) historyData.pop();

  renderHistory();
}

function renderHistory(){
  let h = document.getElementById("history");
  h.innerHTML = "";

  historyData.forEach((x,i)=>{
    let div = document.createElement("div");
    div.className = "p-2 rounded bg-white/5 hover:bg-white/10 cursor-pointer";
    div.innerText = "History #" + (i+1);
    div.onclick = () => {
      document.getElementById("input").value = x.input;
      document.getElementById("output").value = x.output;
    };
    h.appendChild(div);
  });
}

function copyOutput(){
  navigator.clipboard.writeText(document.getElementById("output").value);
}

function clearAll(){
  document.getElementById("input").value="";
  document.getElementById("output").value="";
  historyData=[];
  renderHistory();
}
</script></body>
</html>
