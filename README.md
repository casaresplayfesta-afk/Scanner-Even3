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
    <div id="status" class="muted">Offline • Local Storage</div>
  </div>
  <div id="clock">--:--:--</div>
  <div class="controls top-right">
    <button class="download" id="baixarBtn">Baixar Planilha</button>
    <button class="secondary" id="limparTodosBtn">Limpar todos os pontos</button>
    <button class="secondary" id="logoutBtn">Sair</button>
  </div>
</header>

<main id="mainApp" class="hidden">
  <input id="search" class="search" placeholder="🔍 Pesquisar por nome, matrícula ou e-mail">
  <h3>Colaboradores</h3>
  <button class="add" id="addColabBtn">Adicionar Colaborador</button>
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

<!-- Modal Colaborador -->
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

<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-app.js";
import { getFirestore, collection, getDocs, setDoc, doc } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-firestore.js";

// Firebase config
const firebaseConfig = {
  apiKey: "AIzaSyCpBiFzqOod4K32cWMr5hfx13fw6LGcPVY",
  authDomain: "ponto-eletronico-f35f9.firebaseapp.com",
  projectId: "ponto-eletronico-f35f9",
  storageBucket: "ponto-eletronico-f35f9.firebasestorage.app",
  messagingSenderId: "208638350255",
  appId: "1:208638350255:web:63d016867a67575b5e155a"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

/* ---------------- DADOS ---------------- */
let colaboradores = [];
let pontos = [];

/* ---------------- ELEMENTOS ---------------- */
const loginBtn = document.getElementById('loginBtn');
const loginScreen = document.getElementById('loginScreen');
const mainApp = document.getElementById('mainApp');
const loginMsg = document.getElementById('loginMsg');
const rememberCheckbox = document.getElementById('remember');
const logoutBtn = document.getElementById('logoutBtn');

const addColabBtn = document.getElementById('addColabBtn');
const baixarBtn = document.getElementById('baixarBtn');
const limparTodosBtn = document.getElementById('limparTodosBtn');
const clockEl = document.getElementById('clock');

const colabBody = document.getElementById('colabBody');
const pontosBody = document.getElementById('pontosBody');
const searchInput = document.getElementById('search');

const colabModal = document.getElementById('colabModal');
const colabIdInput = document.getElementById('colabId');
const colabNomeInput = document.getElementById('colabNome');
const colabMatInput = document.getElementById('colabMatricula');
const colabEmailInput = document.getElementById('colabEmail');
const colabTurnoInput = document.getElementById('colabTurno');
const saveColabBtn = document.getElementById('saveColabBtn');
const cancelColabBtn = document.getElementById('cancelColabBtn');

/* ---------------- LOGIN ---------------- */
loginBtn.addEventListener('click', async () => {
  const u = document.getElementById('user').value.trim();
  const p = document.getElementById('pass').value.trim();
  if(u==='CLX' && p==='02072007'){  // <- login atualizado
    loginScreen.style.display='none';
    mainApp.classList.remove('hidden');
    if(rememberCheckbox.checked) localStorage.setItem('autenticado','1');
    await carregarColaboradoresFirebase();
  } else {
    loginMsg.textContent='Usuário ou senha incorretos.';
    setTimeout(()=> loginMsg.textContent='',3000);
  }
});
if(localStorage.getItem('autenticado')==='1'){
  loginScreen.style.display='none';
  mainApp.classList.remove('hidden');
  carregarColaboradoresFirebase();
}
logoutBtn.addEventListener('click', ()=> { localStorage.removeItem('autenticado'); location.reload(); });

/* ---------------- RELÓGIO ---------------- */
setInterval(()=> clockEl.textContent=new Date().toLocaleTimeString('pt-BR',{hour12:false}),1000);

/* ---------------- FIREBASE ---------------- */
async function carregarColaboradoresFirebase(){
  try{
    const snapshotColab = await getDocs(collection(db,"colaboradores"));
    colaboradores = snapshotColab.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    const snapshotPontos = await getDocs(collection(db,"pontos"));
    pontos = snapshotPontos.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    document.getElementById('status').textContent = "Online • Firebase";
    renderAll();
  } catch(e){
    console.error(e);
    document.getElementById('status').textContent = "Offline • Local Storage";
  }
}

async function salvarFirebase(){
  for(const c of colaboradores){
    await setDoc(doc(db,"colaboradores",c.id),c);
  }
  for(const p of pontos){
    await setDoc(doc(db,"pontos",p.id),p);
  }
}

/* ---------------- FUNÇÕES ---------------- */
function escapeHtml(s){ if(!s && s!==0) return ''; return String(s).replace(/[&<>"'`=\/]/g, ch=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;','/':'&#x2F;','`':'&#x60;','=':'&#x3D'}[ch])); }
function renderAll(){ renderColaboradores(); renderPontos(); }

function renderColaboradores() {
  const term = (searchInput.value || '').toLowerCase().trim();
  colabBody.innerHTML = '';

  colaboradores.forEach((c, idx) => {
    if (term && !(String(c.nome).toLowerCase().includes(term) ||
                  String(c.matricula).toLowerCase().includes(term) ||
                  String(c.email || '').toLowerCase().includes(term))) return;

    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${idx + 1}</td>
      <td>${escapeHtml(c.id)}</td>
      <td>${escapeHtml(c.nome)}</td>
      <td>${escapeHtml(c.matricula)} <span class="small">(${escapeHtml(c.email||'')})</span></td>
      <td>${escapeHtml(c.turno||'')}</td>
      <td class="flex-row">
        <button class="edit">Editar</button>
        <button class="del">Excluir</button>
        <button class="entrada" style="background:#e8f5e9;color:#111;margin-left:6px">Entrada</button>
        <button class="saida" style="background:#e3f2fd;color:#111">Saída</button>
      </td>
    `;
    colabBody.appendChild(tr);

    tr.querySelector('.edit').addEventListener('click', () => openColabModal(c.id));
    tr.querySelector('.del').addEventListener('click', () => removerColabPrompt(c.id));
    tr.querySelector('.entrada').addEventListener('click', () => registrarPontoPrompt(c.id, 'Entrada'));
    tr.querySelector('.saida').addEventListener('click', () => registrarPontoPrompt(c.id, 'Saída'));
  });
}

function renderPontos(){
  pontosBody.innerHTML = '';
  pontos.forEach((p, idx)=>{
    const tr = document.createElement('tr');
    tr.innerHTML=`
      <td>${idx+1}</td>
      <td>${escapeHtml(p.idColab)}</td>
      <td>${escapeHtml(p.nome)}</td>
      <td>${escapeHtml(p.matricula)}</td>
      <td>${escapeHtml(p.email||'')}</td>
      <td>${escapeHtml(p.tipo)}</td>
      <td>${escapeHtml(p.data)}</td>
      <td>${escapeHtml(p.hora)}</td>
      <td class="flex-row">
        <button class="del">Excluir</button>
      </td>
    `;
    pontosBody.appendChild(tr);
    tr.querySelector('.del').addEventListener('click', ()=>{ pontos.splice(idx,1); renderPontos(); salvarFirebase(); });
  });
}

/* ---------------- MODAL ---------------- */
function openColabModal(id){
  const c = colaboradores.find(x=>x.id===id);
  colabIdInput.value = c?.id||'';
  colabNomeInput.value = c?.nome||'';
  colabMatInput.value = c?.matricula||'';
  colabEmailInput.value = c?.email||'';
  colabTurnoInput.value = c?.turno||'';
  colabModal.classList.remove('hidden');
  document.getElementById('colabTitle').textContent = id?'Editar Colaborador':'Novo Colaborador';
}
cancelColabBtn.addEventListener('click', ()=> colabModal.classList.add('hidden'));
saveColabBtn.addEventListener('click', ()=>{
  const id = colabIdInput.value || Date.now().toString();
  const c = { id, nome: colabNomeInput.value, matricula: colabMatInput.value, email: colabEmailInput.value, turno: colabTurnoInput.value };
  const idx = colaboradores.findIndex(x=>x.id===id);
  if(idx>-1) colaboradores[idx]=c; else colaboradores.push(c);
  renderColaboradores();
  colabModal.classList.add('hidden');
  salvarFirebase();
});

/* ---------------- REGISTRAR PONTO ---------------- */
function registrarPontoPrompt(idColab, tipo){
  const c = colaboradores.find(x=>x.id===idColab);
  if(!c) return;
  const now = new Date();
  const p = { id: Date.now().toString(), idColab, nome:c.nome, matricula:c.matricula, email:c.email, tipo, data: now.toLocaleDateString('pt-BR'), hora: now.toLocaleTimeString('pt-BR',{hour12:false}) };
  pontos.push(p);
  renderPontos();
  salvarFirebase();
}

/* ---------------- EXCLUIR ---------------- */
function removerColabPrompt(id){
  if(confirm('Deseja realmente excluir este colaborador?')){
    colaboradores = colaboradores.filter(x=>x.id!==id);
    pontos = pontos.filter(p=>p.idColab!==id);
    renderAll();
    salvarFirebase();
  }
}

/* ---------------- PESQUISAR ---------------- */
searchInput.addEventListener('input', renderColaboradores);

/* ---------------- BAIXAR EXCEL ---------------- */
baixarBtn.addEventListener('click', ()=>{
  const wb = XLSX.utils.book_new();
  const wsColab = XLSX.utils.json_to_sheet(colaboradores);
  const wsPontos = XLSX.utils.json_to_sheet(pontos);
  XLSX.utils.book_append_sheet(wb, wsColab,'Colaboradores');
  XLSX.utils.book_append_sheet(wb, wsPontos,'Pontos');
  XLSX.writeFile(wb,'PontoEletronico.xlsx');
});

/* ---------------- LIMPAR TODOS ---------------- */
limparTodosBtn.addEventListener('click', ()=>{
  if(confirm('Deseja realmente limpar todos os pontos?')){
    pontos = [];
    renderPontos();
    salvarFirebase();
  }
});
</script>

</body>
</html>
