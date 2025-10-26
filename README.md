<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Sistema de Ponto Eletrônico</title>
<style>
  body { font-family: Arial,sans-serif; background:#f5f5f5; margin:0; padding:20px; }
  .container { max-width:1200px; margin:auto; background:white; padding:20px; border-radius:8px; box-shadow:0 0 10px rgba(0,0,0,0.1);}
  h1 { text-align:center; color:#333; margin-bottom:10px;}
  h3#boasVindas { text-align:center; color:#555; margin-top:0; margin-bottom:15px; }
  .button-group { display:flex; flex-wrap:wrap; gap:10px; margin-bottom:16px;}
  button { padding:10px 15px; border:none; border-radius:4px; font-size:15px; cursor:pointer; transition:0.3s;}
  button.add { background:#4CAF50; color:white; }
  button.edit { background:#FFC107; color:white;}
  button.del { background:#F44336; color:white;}
  button.reg { background:#2196F3; color:white;}
  button.logout { background:#9C27B0; color:white;}
  button.export { background:#2E7D32; color:white;}
  input, select { padding:8px;border-radius:6px;border:1px solid #ccc;font-size:14px}
  table { width:100%; border-collapse:collapse; margin-top:10px;}
  th, td { border:1px solid #ddd; padding:10px; text-align:left; vertical-align:middle;}
  th { background:#f2f2f2;}
  tr:nth-child(even){background:#f9f9f9;}
  .small { font-size:13px;color:#666; }
  .historyList { margin:8px 0 12px 0; padding:8px; background:#fafafa; border-radius:6px; border:1px solid #eee; max-height:200px; overflow:auto; }
  .detailsToggle { cursor:pointer; color:#0b61a4; text-decoration:underline; background:none;border:none;padding:0;font-size:14px; }
  .modal { display:none; position:fixed; z-index:2; left:0; top:0; width:100%; height:100%; background:rgba(0,0,0,0.4); align-items:center; justify-content:center;}
  .modal-content { background:white; padding:18px; border-radius:8px; width:90%; max-width:520px; box-shadow:0 8px 30px rgba(0,0,0,.12)}
  .close { float:right; font-size:22px; cursor:pointer; border:none;background:none}
  #loginScreen { position:fixed; top:0; left:0; width:100%; height:100%; background:#222; color:white; display:flex; align-items:center; justify-content:center; flex-direction:column; gap:10px; z-index:9; }
  #loginScreen input { padding:10px; font-size:18px; border-radius:6px; border:none; text-align:center;}
  #loginScreen button { background:#4CAF50; color:white; padding:10px 20px; border:none; border-radius:6px; font-size:16px; cursor:pointer;}
  @media(max-width:720px){ .container{padding:12px} th,td{padding:8px} .button-group{gap:6px} }
</style>
</head>
<body>

<!-- Tela de senha -->
<div id="loginScreen">
  <h2>🔒 Acesso Restrito</h2>
  <p>Digite a senha para entrar:</p>
  <input type="password" id="senhaInput" placeholder="Senha">
  <label style="color:white"><input type="checkbox" id="lembrarSenha"> Lembrar login</label>
  <button id="btnEntrar">Entrar</button>
  <p id="erroSenha" style="color:red; display:none;">Senha incorreta!</p>
</div>

<div class="container" id="conteudo" style="display:none;">
  <h1>Sistema de Ponto Eletrônico</h1>
  <h3 id="boasVindas"></h3>

  <div class="button-group">
    <button class="add" id="addColabBtn">Adicionar Colaborador</button>
    <button class="edit" id="editColabBtn">Editar Colaborador</button>
    <button class="del" id="deleteColabBtn">Excluir Colaborador</button>
    <button class="reg" id="openPontoModalBtn">Registrar Ponto</button>
    <button class="export" id="exportExcelBtn">Exportar Excel</button>
    <button class="logout" id="logoutBtn">Sair</button>
  </div>

  <div style="display:flex;gap:10px;align-items:center;margin-bottom:8px">
    <input id="search" placeholder="🔍 Pesquise por nome ou matrícula" style="flex:1" />
    <div class="small" id="totalCount">Total: 0</div>
  </div>

  <h2>Colaboradores</h2>
  <table id="colabTable">
    <thead>
      <tr><th>ID</th><th>Nome</th><th>Matrícula / E-mail</th><th>Cargo</th><th>Turno</th><th>Ações</th></tr>
    </thead>
    <tbody id="colabBody"></tbody>
  </table>

  <h2 style="margin-top:18px">Entradas</h2>
  <table id="entradaTable">
    <thead><tr><th>ID</th><th>Nome</th><th>Matrícula</th><th>Data/Hora</th></tr></thead>
    <tbody id="entradaBody"></tbody>
  </table>

  <h2 style="margin-top:18px">Saídas</h2>
  <table id="saidaTable">
    <thead><tr><th>ID</th><th>Nome</th><th>Matrícula</th><th>Data/Hora</th></tr></thead>
    <tbody id="saidaBody"></tbody>
  </table>
</div>

<!-- Modal Colaborador -->
<div id="colabModal" class="modal">
  <div class="modal-content">
    <button class="close" data-target="colabModal">&times;</button>
    <h3 id="modalTitle">Adicionar Colaborador</h3>
    <form id="colabForm">
      <input type="hidden" id="colabId" value="">
      <label>Nome</label><input id="colabNome" required>
      <label>E-mail</label><input id="colabEmail" type="email" required>
      <label>Matrícula</label><input id="colabMatricula" pattern="\d{4,11}" required>
      <label>Cargo</label><input id="colabCargo" required>
      <label>Turno</label>
      <select id="colabTurno" required><option value="">Selecione</option><option>Manhã</option><option>Tarde</option><option>Noite</option><option>Madrugada</option></select>
      <div style="display:flex;gap:8px;margin-top:12px">
        <button type="submit" class="add">Salvar</button>
        <button type="button" id="cancelColab">Cancelar</button>
        <button type="button" id="delColab" class="del" style="margin-left:auto;display:none">Excluir</button>
      </div>
    </form>
  </div>
</div>

<!-- Modal Ponto -->
<div id="pontoModal" class="modal">
  <div class="modal-content">
    <button class="close" data-target="pontoModal">&times;</button>
    <h3 id="pontoTitle">Registrar Ponto</h3>
    <p class="small">Pesquise o nome ou matrícula e selecione:</p>
    <input id="pontoSearch" placeholder="Digite nome ou matrícula..." />
    <div id="pontoResults" style="margin-top:12px;max-height:320px;overflow:auto"></div>
    <div style="display:flex;gap:8px;margin-top:12px">
      <button id="cancelPonto">Cancelar</button>
    </div>
  </div>
</div>

<!-- ✅ Correção dos botões de topo -->
<script>
  document.getElementById('editColabBtn').addEventListener('click', ()=>{
    const id = prompt('Digite o ID do colaborador que deseja editar:');
    if (id) window.abrirEditarColab(id);
  });

  document.getElementById('deleteColabBtn').addEventListener('click', ()=>{
    const id = prompt('Digite o ID do colaborador que deseja excluir:');
    if (id) window.removerColab(id);
  });

  window.abrirEditarColab = function(id){
    if (!id) return alert('ID inválido');
    openColabModal(id);
  };
</script>

<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
import {
  getFirestore, collection, doc, setDoc, getDoc, updateDoc, deleteDoc,
  onSnapshot, query, orderBy, addDoc, serverTimestamp, runTransaction,
  getDocs
} from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

/* firebaseConfig */
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

/* login simples */
const SENHA_CORRETA = "02072007";
const NOME_USUARIO = "CLX";
const loginScreen = document.getElementById('loginScreen');
const conteudo = document.getElementById('conteudo');
const btnEntrar = document.getElementById('btnEntrar');
const erroSenha = document.getElementById('erroSenha');

btnEntrar.addEventListener('click', ()=>{
  const value = document.getElementById('senhaInput').value;
  const lembrar = document.getElementById('lembrarSenha').checked;
  if (value === SENHA_CORRETA) {
    if (lembrar) localStorage.setItem('autenticado','true');
    loginScreen.style.display='none';
    conteudo.style.display='block';
    mostrarBoasVindas();
    startRealtime();
  } else {
    erroSenha.style.display='block';
    setTimeout(()=> erroSenha.style.display='none',3000);
  }
});

document.getElementById('logoutBtn').onclick = ()=>{
  localStorage.removeItem('autenticado');
  conteudo.style.display='none';
  loginScreen.style.display='flex';
  document.getElementById('senhaInput').value='';
};

function mostrarBoasVindas(){
  const hora=new Date().getHours();
  let saud='Olá';
  if(hora<12) saud='Bom dia';
  else if(hora<18) saud='Boa tarde';
  else saud='Boa noite';
  document.getElementById('boasVindas').textContent=`${saud}, ${NOME_USUARIO}! 👋`;
}

window.onload = ()=>{
  if (localStorage.getItem('autenticado')==='true'){
    loginScreen.style.display='none';
    conteudo.style.display='block';
    mostrarBoasVindas();
    startRealtime();
  }
};

/* refs */
const colabsColl = collection(db,'colaboradores');
const regsColl = collection(db,'registros');
const colabBody = document.getElementById('colabBody');
const entradaBody = document.getElementById('entradaBody');
const saidaBody = document.getElementById('saidaBody');
const totalCount = document.getElementById('totalCount');
const searchInput = document.getElementById('search');

let colaboradores = [];
let registrosCache = [];

/* realtime */
function startRealtime(){
  const qCol = query(colabsColl, orderBy('nome'));
  onSnapshot(qCol, (snap)=>{
    const list = [];
    snap.forEach(d=> list.push({ docId: d.id, ...d.data() }));
    colaboradores = list;
    renderColaboradores();
    totalCount.textContent = 'Total: ' + colaboradores.length;
  });

  const qRegs = query(regsColl, orderBy('timestamp','desc'));
  onSnapshot(qRegs, (snap)=>{
    const arr = [];
    snap.forEach(d=> arr.push({ id: d.id, ...d.data() }));
    registrosCache = arr;
    renderRegistrosSeparados();
  });
}

/* render colaboradores */
function renderColaboradores(){
  const term = (searchInput.value||'').toLowerCase().trim();
  const filtered = term ? colaboradores.filter(c=> (c.nome||'').toLowerCase().includes(term) || (c.matricula||'').toLowerCase().includes(term)) : colaboradores;
  colabBody.innerHTML = '';
  if (!filtered.length) {
    colabBody.innerHTML = '<tr><td colspan="6" style="text-align:center">Nenhum colaborador encontrado.</td></tr>';
    return;
  }
  filtered.forEach(c=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${escapeHtml(c.id || c.docId)}</td>
      <td>${escapeHtml(c.nome)}</td>
      <td>${escapeHtml(c.matricula||'')} <span class="small">(${escapeHtml(c.email||'')})</span></td>
      <td>${escapeHtml(c.cargo||'')}</td>
      <td>${escapeHtml(c.turno||'')}</td>
      <td>
        <button class="reg" onclick="openPontoModal('${c.docId}')">Ponto</button>
        <button class="edit" onclick="abrirEditarColab('${c.docId}')">Editar</button>
        <button class="del" onclick="removerColab('${c.docId}')">Excluir</button>
      </td>`;
    colabBody.appendChild(tr);
  });
}

/* render entradas/saídas */
function renderRegistrosSeparados(){
  const entradas = registrosCache.filter(r=> (r.tipo||'').toLowerCase()==='entrada');
  const saidas = registrosCache.filter(r=> (r.tipo||'').toLowerCase().includes('saíd') || (r.tipo||'').toLowerCase().includes('saida'));
  entradaBody.innerHTML = '';
  entradas.slice(0,200).forEach(r=>{
    const tr=document.createElement('tr');
    tr.innerHTML=`<td>${escapeHtml(r.colaboradorId||'')}</td><td>${escapeHtml(r.nome||'')}</td><td>${escapeHtml(r.matricula||'')}</td><td>${r.hora||''}</td>`;
    entradaBody.appendChild(tr);
  });
  saidaBody.innerHTML='';
  saidas.slice(0,200).forEach(r=>{
    const tr=document.createElement('tr');
    tr.innerHTML=`<td>${escapeHtml(r.colaboradorId||'')}</td><td>${escapeHtml(r.nome||'')}</td><td>${escapeHtml(r.matricula||'')}</td><td>${r.hora||''}</td>`;
    saidaBody.appendChild(tr);
  });
}

/* funções auxiliares */
function escapeHtml(str){ if (str===undefined||str===null) return ''; return String(str).replace(/[&<>"'`=\/]/g, s=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;','/':'&#x2F;','`':'&#x60;','=':'&#x3D'}[s])); }
searchInput.addEventListener('input', ()=> renderColaboradores());
</script>
</body>
</html>
