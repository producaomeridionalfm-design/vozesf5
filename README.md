<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Editor de Áudio IA — Meridional FM</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#0c1017;color:#eef2f7;font-family:Arial,Helvetica,sans-serif}
.app{max-width:1180px;margin:auto;padding:24px}
h1{margin:0 0 6px;font-size:30px}
.sub{color:#9ca8b8;margin-bottom:22px}
.grid{display:grid;grid-template-columns:1.15fr .85fr;gap:18px}
.card{background:#151b24;border:1px solid #273141;border-radius:16px;padding:20px;box-shadow:0 10px 30px #0005}
label{display:block;font-size:13px;color:#aeb8c6;margin:0 0 7px}
textarea,select,input[type=range]{width:100%}
textarea{height:310px;background:#0b0f15;color:#fff;border:1px solid #344052;border-radius:12px;padding:16px;font-size:19px;line-height:1.45;resize:vertical;outline:none}
textarea:focus{border-color:#6ea8ff}
.row{display:grid;grid-template-columns:1fr 1fr;gap:14px;margin-top:15px}
select{background:#0b0f15;color:#fff;border:1px solid #344052;border-radius:9px;padding:12px}
.stat{display:flex;justify-content:space-between;margin-top:9px;color:#aeb8c6;font-size:13px}
.stat strong{color:#fff}
.btns{display:flex;flex-wrap:wrap;gap:10px;margin-top:17px}
button{border:0;border-radius:10px;padding:12px 16px;font-weight:700;cursor:pointer;color:#fff;background:#263246}
button.primary{background:#2878ff}
button.green{background:#1e9e68}
button.danger{background:#b9384a}
button:hover{filter:brightness(1.12)}
.track{border:1px solid #303b4c;background:#0e141d;border-radius:12px;padding:14px;margin:10px 0;cursor:pointer}
.track.active{border-color:#4f94ff;background:#132039}
.track b{display:block;margin-bottom:5px}
.track span{font-size:12px;color:#929eaf}
.meter{height:12px;background:#080b10;border-radius:10px;overflow:hidden;margin:10px 0}
.meter i{display:block;height:100%;width:0;background:#39d98a}
.timeline{margin-top:18px;height:90px;background:#080b10;border-radius:12px;border:1px solid #283241;position:relative;overflow:hidden}
.wave{position:absolute;inset:0;opacity:.7;background:
repeating-linear-gradient(90deg,transparent 0 7px,#334154 8px 9px),
linear-gradient(90deg,#1b5d9b,#2c8b67,#1b5d9b)}
.badge{display:inline-block;padding:5px 9px;border-radius:20px;background:#20304a;color:#8fc0ff;font-size:11px;font-weight:700}
.small{font-size:12px;color:#8f9aaa;line-height:1.5}
.warning{background:#3a2711;border:1px solid #8a5b1c;color:#ffc76a;padding:10px;border-radius:10px;margin-top:12px;display:none}
.ok{background:#103523;border:1px solid #1d7c50;color:#7cf0b7;padding:10px;border-radius:10px;margin-top:12px;display:none}
@media(max-width:850px){.grid{grid-template-columns:1fr}.row{grid-template-columns:1fr}}
</style>
</head>
<body>
<div class="app">
  <h1>🎙️ Editor de Áudio IA</h1>
  <div class="sub">Texto → voz → trilha → mixagem. Pensado para locuções, comerciais e vinhetas de rádio.</div>

  <div class="grid">
    <section class="card">
      <span class="badge">1 · TEXTO</span>
      <h2>Seu roteiro</h2>
      <textarea id="text" placeholder="Cole aqui o texto da locução..."></textarea>
      <div class="stat">
        <span><strong id="words">0</strong> palavras</span>
        <span>tempo estimado: <strong id="time">0:00</strong></span>
      </div>

      <div class="row">
        <div>
          <label>Voz IA / voz disponível no computador</label>
          <select id="voice"><option>Carregando vozes...</option></select>
        </div>
        <div>
          <label>Velocidade</label>
          <input id="rate" type="range" min=".65" max="1.35" value="1" step=".01">
          <div class="stat"><span>lenta</span><strong id="rateVal">1.00x</strong><span>rápida</span></div>
        </div>
      </div>

      <div class="btns">
        <button class="primary" onclick="speak()">▶ Ouvir voz</button>
        <button onclick="pauseSpeech()">⏸ Pausar</button>
        <button class="danger" onclick="stopSpeech()">■ Parar</button>
      </div>

      <div id="browserNote" class="warning">
        A voz é fornecida pelo mecanismo de voz do navegador/sistema. Para uma voz neural comercial específica (ex.: ElevenLabs, Azure etc.), o arquivo pode ser adaptado para receber a API escolhida.
      </div>
    </section>

    <section class="card">
      <span class="badge">2 · TRILHA</span>
      <h2>Trilhas originais</h2>
      <p class="small">As trilhas abaixo são geradas pelo próprio editor por síntese sonora. Não são músicas comerciais e não dependem de catálogo de terceiros.</p>

      <div id="tracks">
        <div class="track active" data-track="ambient" onclick="selectTrack(this)">
          <b>🎹 Ambient Rádio</b><span>Clima moderno, leve e institucional</span>
        </div>
        <div class="track" data-track="sertanejo" onclick="selectTrack(this)">
          <b>🤠 Sertanejo Acústico</b><span>Base rural/sertaneja criada por síntese</span>
        </div>
        <div class="track" data-track="impact" onclick="selectTrack(this)">
          <b>⚡ Impacto News</b><span>Batida curta, tensão e energia</span>
        </div>
        <div class="track" data-track="romantic" onclick="selectTrack(this)">
          <b>❤️ Romântica</b><span>Pad suave para amor e carinho</span>
        </div>
      </div>

      <label>Volume da trilha</label>
      <input id="musicVol" type="range" min="0" max="1" value=".28" step=".01">
      <div class="stat"><span>0%</span><strong id="musicVal">28%</strong><span>100%</span></div>

      <div class="btns">
        <button class="green" onclick="previewMusic()">♫ Ouvir trilha</button>
        <button onclick="stopMusic()">■ Parar trilha</button>
      </div>
    </section>
  </div>

  <section class="card" style="margin-top:18px">
    <span class="badge">3 · MIXAGEM</span>
    <h2>Prévia</h2>
    <div class="row">
      <div>
        <label>Volume da voz</label>
        <input id="voiceVol" type="range" min="0" max="1" value="1" step=".01">
        <div class="stat"><span>0%</span><strong id="voiceVal">100%</strong><span>100%</span></div>
      </div>
      <div>
        <label>Duração aproximada</label>
        <div style="font-size:30px;font-weight:700" id="bigTime">00:00</div>
        <div class="small">A duração real da voz pode variar conforme a voz selecionada.</div>
      </div>
    </div>

    <div class="timeline"><div class="wave"></div></div>
    <div class="meter"><i id="meter"></i></div>

    <div id="status" class="small">Pronto. Cole seu texto para começar.</div>
    <div id="ok" class="ok">Trilha original selecionada. Ela é gerada localmente pelo editor.</div>

    <div class="btns">
      <button class="primary" onclick="previewAll()">▶ Prévia completa</button>
      <button onclick="stopAll()">■ Parar tudo</button>
      <button class="green" onclick="downloadProject()">⬇ Salvar projeto HTML</button>
    </div>

    <p class="small" style="margin-top:16px">
      Observação: o navegador não permite, de forma padronizada, gravar a saída do SpeechSynthesis diretamente em WAV.
      Por isso esta versão funciona imediatamente sem chave de API e oferece prévia da voz + trilhas originais.
      Para exportar um MP3/WAV final com voz neural, basta conectar um provedor TTS/API no próximo passo.
    </p>
  </section>
</div>

<script>
const textEl=document.getElementById('text');
const voiceEl=document.getElementById('voice');
const rateEl=document.getElementById('rate');
const musicVol=document.getElementById('musicVol');
const voiceVol=document.getElementById('voiceVol');
let voices=[], selectedTrack='ambient', audioCtx=null, musicNodes=[];

function updateText(){
  const t=textEl.value.trim();
  const words=t?t.split(/\\s+/).length:0;
  document.getElementById('words').textContent=words;
  const sec=words/2.5;
  const s=Math.round(sec);
  const out=`${Math.floor(s/60)}:${String(s%60).padStart(2,'0')}`;
  document.getElementById('time').textContent=out;
  document.getElementById('bigTime').textContent=out;
}
textEl.addEventListener('input',updateText);
rateEl.addEventListener('input',()=>document.getElementById('rateVal').textContent=Number(rateEl.value).toFixed(2)+'x');
musicVol.addEventListener('input',()=>document.getElementById('musicVal').textContent=Math.round(musicVol.value*100)+'%');
voiceVol.addEventListener('input',()=>document.getElementById('voiceVal').textContent=Math.round(voiceVol.value*100)+'%');

function loadVoices(){
  voices=speechSynthesis.getVoices();
  voiceEl.innerHTML='';
  const br=voices.filter(v=>/^pt-BR/i.test(v.lang));
  const list=br.length?br:voices;
  list.forEach((v,i)=>{
    const o=document.createElement('option');
    o.value=voices.indexOf(v);
    o.textContent=v.name+' — '+v.lang;
    voiceEl.appendChild(o);
  });
  if(!list.length) voiceEl.innerHTML='<option>Nenhuma voz encontrada</option>';
}
speechSynthesis.onvoiceschanged=loadVoices; loadVoices();

function speak(){
  const t=textEl.value.trim();
  if(!t)return alert('Cole um texto primeiro.');
  speechSynthesis.cancel();
  const u=new SpeechSynthesisUtterance(t);
  const v=voices[Number(voiceEl.value)];
  if(v) {u.voice=v;u.lang=v.lang;}
  u.rate=Number(rateEl.value);
  u.volume=Number(voiceVol.value);
  u.onstart=()=>{document.getElementById('status').textContent='Voz em reprodução...';document.getElementById('meter').style.width='70%';};
  u.onend=()=>{document.getElementById('status').textContent='Voz finalizada.';document.getElementById('meter').style.width='0';};
  speechSynthesis.speak(u);
}
function pauseSpeech(){speechSynthesis.pause()}
function stopSpeech(){speechSynthesis.cancel();document.getElementById('meter').style.width='0'}
function selectTrack(el){
  document.querySelectorAll('.track').forEach(x=>x.classList.remove('active'));
  el.classList.add('active');selectedTrack=el.dataset.track;
  document.getElementById('ok').style.display='block';
}

function ctx(){
  if(!audioCtx)audioCtx=new (window.AudioContext||window.webkitAudioContext)();
  if(audioCtx.state==='suspended')audioCtx.resume();
  return audioCtx;
}
function osc(c,f,type,g,dur,start){
  const o=c.createOscillator(), g=c.createGain();
  o.type=type;o.frequency.value=f;
  g.gain.setValueAtTime(0.0001,start);
  g.gain.exponentialRampToValueAtTime(Math.max(.0001,gainVal()),start+.02);
  g.gain.exponentialRampToValueAtTime(.0001,start+dur);
  o.connect(g).connect(c.destination);o.start(start);o.stop(start+dur+.03);
  musicNodes.push(o,g);
}
function gainVal(){return Number(musicVol.value)*.12}
function previewMusic(){
  stopMusic(); const c=ctx(), now=c.currentTime+.05;
  const bpm=selectedTrack==='impact'?110:82, beat=60/bpm;
  const total=18;
  if(selectedTrack==='ambient'){
    [261.63,329.63,392,523.25].forEach((f,i)=>osc(c,f,'sine',0,total,now+i*beat*2));
    for(let i=0;i<total/beat;i++)osc(c,[130.8,164.8][i%2],'triangle',0,beat*.7,now+i*beat);
  }else if(selectedTrack==='sertanejo'){
    const notes=[196,220,246.94,293.66,246.94,220];
    for(let i=0;i<24;i++)osc(c,notes[i%notes.length],'triangle',0,.35,now+i*.5);
    for(let i=0;i<18;i++)osc(c,98,'sine',0,.2,now+i*1);
  }else if(selectedTrack==='impact'){
    for(let i=0;i<36;i++)osc(c,i%3===0?55:220,'sawtooth',0,.08,now+i*.25);
    for(let i=0;i<8;i++)osc(c,440+i*55,'square',0,.12,now+i*.75);
  }else{
    const notes=[220,261.63,329.63,392,329.63,261.63];
    for(let i=0;i<30;i++)osc(c,notes[i%notes.length],'sine',0,.65,now+i*.5);
  }
  document.getElementById('status').textContent='Trilha original em reprodução...';
}
function stopMusic(){
  musicNodes.forEach(n=>{try{n.stop&&n.stop()}catch(e){}});musicNodes=[];
}
function previewAll(){
  stopAll(); previewMusic(); setTimeout(speak,100);
}
function stopAll(){stopMusic();stopSpeech();document.getElementById('status').textContent='Tudo parado.';document.getElementById('meter').style.width='0'}
function downloadProject(){
  const blob=new Blob([document.documentElement.outerHTML],{type:'text/html'});
  const a=document.createElement('a');a.href=URL.createObjectURL(blob);a.download='editor_audio_ia.html';a.click();
  setTimeout(()=>URL.revokeObjectURL(a.href),1000);
}
updateText();
</script>
</body>
</html>
