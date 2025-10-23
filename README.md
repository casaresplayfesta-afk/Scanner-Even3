<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Ponto Eletrônico</title>
<style>
body { font-family: Arial; background:#f5f5f5; padding:20px; }
.container { max-width:1200px; margin:auto; background:white; padding:20px; border-radius:8px; box-shadow:0 0 10px rgba(0,0,0,0.1); }
h1 { text-align:center; color:#333; margin-bottom:30px; }
.button-group { display:flex; flex-wrap:wrap; gap:10px; margin-bottom:20px; }
button { padding:10px 15px; border:none; border-radius:4px; font-size:16px; cursor:pointer; transition:0.3s; }
button:hover { opacity:0.9; }
button.add { background:#4CAF50; color:white; }
button.facial { background:#ff9800; color:white; }
table { width:100%; border-collapse:collapse; margin-top:20px; }
th, td { border:1px solid #ddd; padding:12px; text-align:left; }
th { background:#f2f2f2; }
tr:nth-child(even) { background:#f9f9f9; }
.modal { display:none; position:fixed; z-index:1; left:0; top:0; width:100%; height:100%; background:rgba(0,0,0,0.4); overflow:auto; }
.modal-content { background:white; margin:8% auto; padding:20px; border-radius:8px; width:90%; max-width:700px; }
.close { float:right; font-size:28px; font-weight:bold; cursor:pointer; }
.close:hover { color:black; }
input, select { width:100%; padding:10px; margin:5px 0; border-radius:4px; border:1px solid #ccc; font-size:16px; }
.footer { text-align:center; margin-top:30px; padding-top:20px; border-top:1px solid #ddd; color:#666; }
.camera-container { display:flex; flex-direction:column; align-items:center; gap:12px; }
#video { background:#ddd; width:100%; max-width:500px; }
#canvas { display:none; }
</style>
</head>
<body>
<div class="container">
<h1>Sistema de Ponto Eletrônico</h1>

<div class="button-group">
  <button class="add" id="registrarPontoBtn">Registrar Ponto</button>
  <button class="facial" id="reconhecimentoBtn">Reconhecimento Facial</button>
</div>

<div id="registrosTableContainer">
  <h2>Registros de Ponto</h2>
  <table id="registrosTable">
    <thead>
      <tr><th>ID</th><th>Nome</th><th>Data/Hora</th><th>Tipo</th><th>Método</th></tr>
    </thead>
    <tbody id="registrosBody"></tbody>
  </table>
</div>

<div class="footer">
  <p>Sistema de Ponto Eletrônico v1.0</p>
</div>
</div>

<!-- Modal Registrar Ponto -->
<div id="registrarPontoModal" class="modal">
  <div class="modal-content">
    <span class="close" data-target="registrarPontoModal">&times;</span>
    <h2>Registrar Ponto</h2>
    <form id="registrarPontoForm">
      <select id="selectColaborador" required></select>
      <select id="pontoTipo" required>
        <option value="">Selecione o Tipo</option>
        <option value="Entrada">Entrada</option>
        <option value="Saída">Saída</option>
      </select>
      <button type="submit">Registrar</button>
    </form>
  </div>
</div>

<!-- Modal Reconhecimento Facial -->
<div id="facialModal" class="modal">
  <div class="modal-content">
    <span class="close" data-target="facialModal">&times;</span>
    <h2>Reconhecimento Facial</h2>
    <div class="camera-container">
      <video id="video" autoplay></video>
      <canvas id="canvas" width="640" height="480"></canvas>
      <button id="captureBtn">Capturar e Registrar Ponto</button>
      <div id="facialResult"></div>
    </div>
  </div>
</div>

<script>
let registrosPonto = JSON.parse(localStorage.getItem('registrosPonto')) || [];
let ultimoIdRegistro = registrosPonto.length>0?Math.max(...registrosPonto.map(r=>r.id)):0;

function carregarRegistros(){
  const tbody=document.getElementById('registrosBody');
  tbody.innerHTML='';
  registrosPonto.forEach(r=>{
    const row=document.createElement('tr');
    row.innerHTML=`<td>${r.id}</td><td>${r.nome}</td><td>${new Date(r.dataHora).toLocaleString()}</td><td>${r.tipo}</td><td>${r.metodo}</td>`;
    tbody.appendChild(row);
  });
}
carregarRegistros();

function carregarColaboradores(){
  const select=document.getElementById('selectColaborador');
  select.innerHTML='<option value="">Selecione o Colaborador</option>';
  let colaboradores=JSON.parse(localStorage.getItem('colaboradores'))||[];
  colaboradores.forEach(c=>{ if(!c.inativo) select.innerHTML+=`<option value="${c.nome}">${c.nome}</option>`; });
}
carregarColaboradores();

// Modal lógica
const registrarPontoBtn=document.getElementById('registrarPontoBtn');
const reconhecimentoBtn=document.getElementById('reconhecimentoBtn');
const registrarPontoModal=document.getElementById('registrarPontoModal');
const facialModal=document.getElementById('facialModal');
const closeButtons=document.querySelectorAll('.close');
registrarPontoBtn.onclick=()=>{ carregarColaboradores(); registrarPontoModal.style.display='block'; };
reconhecimentoBtn.onclick=()=>{ iniciarCamera(); facialModal.style.display='block'; };
closeButtons.forEach(btn=>btn.onclick=()=>{ const t=btn.dataset.target; if(t) document.getElementById(t).style.display='none'; pararCamera(); });
window.onclick=e=>{ if(e.target.classList.contains('modal')){ e.target.style.display='none'; pararCamera(); } };

// Registrar ponto manual
document.getElementById('registrarPontoForm').addEventListener('submit', e=>{
  e.preventDefault();
  const nome=document.getElementById('selectColaborador').value;
  const tipo=document.getElementById('pontoTipo').value;
  if(!nome){ alert('Selecione um colaborador'); return; }
  ultimoIdRegistro++;
  registrosPonto.push({id:ultimoIdRegistro,nome,tipo,dataHora:new Date().toISOString(),metodo:'Manual'});
  localStorage.setItem('registrosPonto',JSON.stringify(registrosPonto));
  carregarRegistros();
  registrarPontoModal.style.display='none';
});

// Reconhecimento facial (simulado)
let stream=null;
function iniciarCamera(){
  const video=document.getElementById('video');
  if(!navigator.mediaDevices||!navigator.mediaDevices.getUserMedia){ alert('Navegador não suporta câmera'); return; }
  navigator.mediaDevices.getUserMedia({video:true,audio:false}).then(s=>{stream=s; video.srcObject=stream;}).catch(err=>alert('Erro: '+err.message));
}
function pararCamera(){ if(stream){ stream.getTracks().forEach(t=>t.stop()); stream=null; } }
document.getElementById('captureBtn').onclick=()=>{
  const video=document.getElementById('video');
  const canvas=document.getElementById('canvas');
  canvas.getContext('2d').drawImage(video,0,0,canvas.width,canvas.height);
  let colaboradores=JSON.parse(localStorage.getItem('colaboradores'))||[];
  const ativos=colaboradores.filter(c=>!c.inativo);
  if(ativos.length===0){ alert('Nenhum colaborador ativo'); return; }
  const aleatorio=ativos[Math.floor(Math.random()*ativos.length)];
  const tipo=Math.random()>0.5?'Entrada':'Saída';
  ultimoIdRegistro++;
  registrosPonto.push({id:ultimoIdRegistro,nome:aleatorio.nome,tipo,dataHora:new Date().toISOString(),metodo:'Facial'});
  localStorage.setItem('registrosPonto',JSON.stringify(registrosPonto));
  carregarRegistros();
  document.getElementById('facialResult').textContent=`Ponto registrado para ${aleatorio.nome} (${tipo})`;
  setTimeout(()=>{ facialModal.style.display='none'; pararCamera(); document.getElementById('facialResult').textContent=''; },2000);
};
</script>
</body>
</html>
