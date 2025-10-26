<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Ponto Eletrônico - Firebase</title>
<style>
  :root{--blue:#003366;--green:#4CAF50;--yellow:#ff9800;--red:#f44336;}
  body{font-family:Arial,Helvetica,sans-serif;background:#f7f9fc;margin:0}
  header{background:var(--blue);color:#fff;padding:10px 16px;display:flex;align-items:center;justify-content:space-between;gap:12px;flex-wrap:wrap}
  .logo{font-weight:700}
  #clock{font-weight:700}
  .controls{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
  button{padding:8px 12px;border:none;border-radius:6px;cursor:pointer;font-weight:600}
  .add{background:var(--green);color:#fff}
  .edit{background:#2196F3;color:#fff}
  .del{background:#f44336;color:#fff}
  .download{background:var(--yellow);color:#111}
  .secondary{background:#e0e0e0;color:#222}
  main{padding:18px;max-width:1100px;margin:18px auto}
  .search{width:100%;padding:8px;border-radius:6px;border:1px solid #ccc;margin-bottom:12px}
  table{width:100%;border-collapse:collapse;background:#fff;border-radius:8px;overflow:hidden;box-shadow:0 4px 18px rgba(0,0,0,0.06)}
  th,td{padding:10px;border-bottom:1px solid #eee;text-align:left;font-size:14px}
  th{background:#fafafa;font-weight:700}
  tr:hover td{background:#fbfbfb}
  .small{font-size:13px;color:#666;margin-left:6px}
  .muted{color:#666;font-size:13px}
  .flex-row{display:flex;gap:8px;align-items:center}
  .modal{position:fixed;inset:0;background:rgba(0,0,0,.5);display:flex;align-items:center;justify-content:center;z-index:999}
  .modal-content{background:#fff;padding:20px;border-radius:10px;width:95%;max-width:420px}
  .hidden{display:none}
  .top-right{display:flex;gap:8px;align-items:center}
  @media(max-width:720px){ header{flex-direction:column;align-items:flex-start} .controls{width:100%;justify-content:space-between} }
</style>
</head>
<body>

<!-- LOGIN -->
<div id="loginScreen" style="position:fixed;inset:0;background:var(--blue);display:flex;align-items:center;justify-content:center;z-index:9999">
  <div style="background:#fff;padding:28px;border-radius:10px;width:92%;max-width:360px;text-align:center">
    <h2 style="margin:0 0 8px">Login do Sistema</h2>
    <input id="user" placeholder="Usuário" style="width:92%;padding:10px;margin:8px 0;border-radius:6px;border:1px solid #ccc"><br>
    <input id="pass" type="password" placeholder="Senha" style="width:92%;padding:10px;margin:8px 0;border-radius:6px;border:1px solid #ccc"><br>
    <label style="font-size:13px"><input type="checkbox" id="remember"> Lembrar login</label><br>
    <button id="loginBtn" class="add" style="width:92%;margin-top:6px">Entrar</button>
    <p id="loginMsg" style="color:crimson;margin-top:8px;height:18px"></p>
    <p style="font-size:12px;color:#666;margin-top:6px">Usuário: <b>CLX</b> / Senha: <b>02072007</b></p>
  </div>
</div>

<header>
  <div style="display:flex;gap:12px;align-items:center">
    <div class="logo">Ponto Eletrônico</div>
    <div id="status" class="muted">Online • Firebase</div>
  </div>

  <div id="clock">--:--:--</div>

  <div class="controls top-right">
    <button class="add" id="addColabBtn">Adicionar</button>
    <button class="edit" id="editColabBtn">Editar</button>
    <button class="del" id="deleteColabBtn">Excluir</button>
    <button class="add" id="entradaBtn">Bater Entrada</button>
    <button class="edit" id="saidaBtn">Bater Saída</button>
    <button class="download" id="baixarBtn">Baixar Planilha</button>
    <button class="secondary" id="limparTodosBtn">Limpar todos os pontos</button>
    <button class="secondary" id="logoutBtn">Sair</button>
  </div>
</header>

<main id="mainApp" class="hidden">
  <input id="search" class="search" placeholder="🔍 Pesquisar por nome, matrícula ou e-mail">

  <h3>Colaboradores</h3>
  <table id="colabTable">
    <thead>
      <tr>
        <th>#</th><th>ID</th><th>Nome</th><th>Matrícula / E-mail</th><th>Turno</th><th>Ações</th>
      </tr>
    </thead>
    <tbody id="colabBody"></tbody>
  </table>

  <h3 style="margin-top:18px">Pontos Registrados</h3>
  <table id="pontosTable">
    <thead>
      <tr>
        <th>#</th><th>ID Colab</th><th>Nome</th><th>Matrícula</th><th>E-mail</th><th>Tipo</th><th>Data</th><th>Hora</th><th>Ações</th>
      </tr>
    </thead>
    <tbody id="pontosBody"></tbody>
  </table>
</main>

<div id="colabModal" class="modal hidden">
  <div class="modal-content">
    <h3 id="colabTitle">Novo Colaborador</h3>
    <input type="hidden" id="colabId">
    <div style="margin-top:8px">
      <label>Nome</label><br><input id="colabNome" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #ccc">
      <label>Matrícula</label><br><input id="colabMatricula" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #ccc">
      <label>E-mail</label><br><input id="colabEmail" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #ccc">
      <label>Turno</label><br><input id="colabTurno" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #ccc">
    </div>
    <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:12px">
      <button id="saveColabBtn" class="add">Salvar</button>
      <button id="cancelColabBtn" class="secondary">Cancelar</button>
    </div>
  </div>
</div>

<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

<script>
// ---------------- CONFIG FIREBASE ----------------
const firebaseConfig = {
  apiKey: "AIzaSyCpBiFzqOod4K32cWMr5hfx13fw6LGcPVY",
  authDomain: "ponto-eletronico-f35f9.firebaseapp.com",
  projectId: "ponto-eletronico-f35f9",
  storageBucket: "ponto-eletronico-f35f9.appspot.com",
  messagingSenderId: "208638350255",
  appId: "1:208638350255:web:63d016867a67575b5e155a"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();

// ---------------- ESTADO ----------------
let colaboradores = [];
let pontos = [];

// ---------------- ELEMENTOS ----------------
const loginScreen = document.getElementById('loginScreen');
const mainApp = document.getElementById('mainApp');
const loginBtn = document.getElementById('loginBtn');
const loginMsg = document.getElementById('loginMsg');
const logoutBtn = document.getElementById('logoutBtn');
const rememberCheckbox = document.getElementById('remember');
const clockEl = document.getElementById('clock');

const addColabBtn = document.getElementById('addColabBtn');
const editColabBtn = document.getElementById('editColabBtn');
const deleteColabBtn = document.getElementById('deleteColabBtn');
const entradaBtn = document.getElementById('entradaBtn');
const saidaBtn = document.getElementById('saidaBtn');
const baixarBtn = document.getElementById('baixarBtn');
const limparTodosBtn = document.getElementById('limparTodosBtn');

const searchInput = document.getElementById('search');
const colabBody = document.getElementById('colabBody');
const pontosBody = document.getElementById('pontosBody');

const colabModal = document.getElementById('colabModal');
const colabIdInput = document.getElementById('colabId');
const colabNomeInput = document.getElementById('colabNome');
const colabMatInput = document.getElementById('colabMatricula');
const colabEmailInput = document.getElementById('colabEmail');
const colabTurnoInput = document.getElementById('colabTurno');
const saveColabBtn = document.getElementById('saveColabBtn');
const cancelColabBtn = document.getElementById('cancelColabBtn');

// ---------------- LOGIN ----------------
loginBtn.addEventListener('click',()=>{
  const u=document.getElementById('user').value.trim();
  const p=document.getElementById('pass').value.trim();
  if(u==='admin' && p==='1234'){
    loginScreen.style.display='none';
    mainApp.classList.remove('hidden');
    if(rememberCheckbox.checked) localStorage.setItem('autenticado','1');
    renderAll();
  } else {
    loginMsg.textContent='Usuário ou senha incorretos';
    setTimeout(()=>loginMsg.textContent='',3000);
  }
});
if(localStorage.getItem('autenticado')==='1'){
  loginScreen.style.display='none';
  mainApp.classList.remove('hidden');
}

// ---------------- RELÓGIO ----------------
function atualizarRelogio(){
  clockEl.textContent = new Date().toLocaleTimeString('pt-BR',{hour12:false});
}
setInterval(atualizarRelogio,1000);
atualizarRelogio();

// ---------------- FUNÇÕES ----------------
function salvarColaboradorFirebase(c){
  db.ref('colaboradores/'+c.id).set(c);
}
function removerColaboradorFirebase(id){
  db.ref('colaboradores/'+id).remove();
}
function salvarPontoFirebase(p){
  db.ref('pontos').push(p);
}

// ---------------- RENDER ----------------
function renderColaboradores(){
  const term=(searchInput.value||'').toLowerCase();
  colabBody.innerHTML='';
  colaboradores.forEach((c,idx)=>{
    if(term && !(c.nome.toLowerCase().includes(term) || c.matricula.toLowerCase().includes(term) || (c.email||'').toLowerCase().includes(term))) return;
    const tr=document.createElement('tr');
    tr.innerHTML=`
      <td>${idx+1}</td>
      <td>${c.id}</td>
      <td>${c.nome}</td>
      <td>${c.matricula} <span class="small">(${c.email||''})</span></td>
      <td>${c.turno||''}</td>
      <td class="flex-row">
        <button class="edit" onclick="editarColab('${c.id}')">Editar</button>
        <button class="del" onclick="removerColabPrompt('${c.id}')">Excluir</button>
        <button class="secondary" style="background:#e8f5e9;color:#111;margin-left:6px" onclick="registrarPontoPrompt('${c.id}','Entrada')">Entrada</button>
        <button class="secondary" style="background:#e3f2fd;color:#111" onclick="registrarPontoPrompt('${c.id}','Saída')">Saída</button>
      </td>
    `;
    colabBody.appendChild(tr);
  });
}

function renderPontos(){
  pontosBody.innerHTML='';
  pontos.forEach((p,i)=>{
    const tr=document.createElement('tr');
    tr.innerHTML=`
      <td>${i+1}</td>
      <td>${p.id}</td>
      <td>${p.nome}</td>
      <td>${p.matricula}</td>
      <td>${p.email||''}</td>
      <td>${p.tipo}</td>
      <td>${p.data}</td>
      <td>${p.hora}</td>
      <td><button class="del" onclick="removerPontoFirebase('${p.key}')">Excluir</button></td>
    `;
    pontosBody.appendChild(tr);
  });
}

function renderAll(){
  renderColaboradores();
  renderPontos();
}

// ---------------- FIREBASE LISTENERS ----------------
db.ref('colaboradores').on('value',snapshot=>{
  colaboradores=[];
  snapshot.forEach(snap=>{
    colaboradores.push(snap.val());
  });
  renderColaboradores();
});
db.ref('pontos').on('value',snapshot=>{
  pontos=[];
  snapshot.forEach(snap=>{
    let p=snap.val();
    p.key=snap.key;
    pontos.push(p);
  });
  renderPontos();
});

// ---------------- CRUD ----------------
addColabBtn.addEventListener('click',()=>openColabModal());
editColabBtn.addEventListener('click',()=>{
  const id=prompt('Digite o ID do colaborador para editar:');
  if(id) openColabModal(id);
});
deleteColabBtn.addEventListener('click',()=>{
  const id=prompt('Digite o ID do colaborador para excluir:');
  if(id) removerColabPrompt(id);
});

function openColabModal(id){
  if(id){
    const c=colaboradores.find(x=>x.id===id);
    if(!c) return alert('Colaborador não encontrado');
    colabIdInput.value=c.id;
    colabNomeInput.value=c.nome;
    colabMatInput.value=c.matricula;
    colabEmailInput.value=c.email||'';
    colabTurnoInput.value=c.turno||'';
    document.getElementById('colabTitle').textContent='Editar Colaborador';
  } else {
    colabIdInput.value='';
    colabNomeInput.value='';
    colabMatInput.value='';
    colabEmailInput.value='';
    colabTurnoInput.value='';
    document.getElementById('colabTitle').textContent='Novo Colaborador';
  }
  colabModal.classList.remove('hidden');
}
cancelColabBtn.addEventListener('click',()=>colabModal.classList.add('hidden'));
saveColabBtn.addEventListener('click',()=>{
  const idVal=colabIdInput.value.trim();
  const nome=colabNomeInput.value.trim();
  const mat=colabMatInput.value.trim();
  const email=colabEmailInput.value.trim();
  const turno=colabTurnoInput.value.trim();
  if(!nome) return alert('Preencha o nome');
  const c={id:idVal||String(Date.now()),nome,matricula:mat,email,turno};
  salvarColaboradorFirebase(c);
  colabModal.classList.add('hidden');
});

window.editarColab=openColabModal;
function removerColabPrompt(id){
  if(!confirm('Excluir colaborador '+id+'?')) return;
  removerColaboradorFirebase(id);
}

entradaBtn.addEventListener('click',()=>{const id=prompt('Digite o ID do colaborador para registrar Entrada:');if(id) registrarPontoPrompt(id,'Entrada');});
saidaBtn.addEventListener('click',()=>{const id=prompt('Digite o ID do colaborador para registrar Saída:');if(id) registrarPontoPrompt(id,'Saída');});

function registrarPontoPrompt(id,tipo){
  const c=colaboradores.find(x=>x.id===id);
  if(!c) return alert('Colaborador não encontrado');
  const now=new Date();
  const p={id:c.id,nome:c.nome,matricula:c.matricula,email:c.email||'',tipo,data:now.toLocaleDateString('pt-BR'),hora:now.toLocaleTimeString('pt-BR',{hour12:false})};
  salvarPontoFirebase(p);
  alert(`${c.nome} registrou ${tipo} às ${p.hora}`);
}
function removerPontoFirebase(key){
  if(!confirm('Excluir este ponto?')) return;
  db.ref('pontos/'+key).remove();
}

limparTodosBtn.addEventListener('click',()=>{
  if(confirm('Deseja apagar TODOS os pontos?')) db.ref('pontos').remove();
});

baixarBtn.addEventListener('click',()=>{
  if(pontos.length===0) return alert('Nenhum ponto registrado ainda');
  const rows=pontos.map((p,i)=>({Numero:i+1,ID_Colaborador:p.id,Nome:p.nome,Matricula:p.matricula,Email:p.email,Tipo:p.tipo,Data:p.data,Hora:p.hora}));
  const ws=XLSX.utils.json_to_sheet(rows);
  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,ws,'Pontos');
  XLSX.writeFile(wb,'registros_ponto.xlsx');
});

searchInput.addEventListener('input',renderAll);
</script>
</body>
</html>
