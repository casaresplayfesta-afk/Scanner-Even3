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

<!-- Tela de login -->
<div id="loginScreen">
  <h2>🔒 Acesso Restrito</h2>
  <p>Digite a senha para entrar:</p>
  <input type="password" id="senhaInput" placeholder="Senha">
  <button id="btnEntrar">Entrar</button>
  <p id="erroSenha" style="color:red; display:none;">Senha incorreta!</p>
</div>

<div class="container" id="conteudo" style="display:none;">
  <h1>Sistema de Ponto Eletrônico</h1>
  <h3 id="boasVindas"></h3>

  <div class="button-group">
    <button class="add" id="addColabBtn">Adicionar Colaborador</button>
    <button class="export" id="exportExcelBtn">Exportar Excel</button>
    <button class="logout" id="logoutBtn">Sair</button>
  </div>

  <div style="display:flex;gap:10px;align-items:center;margin-bottom:8px">
    <input id="search" placeholder="🔍 Pesquise por nome, matrícula ou e-mail" style="flex:1" />
    <div class="small" id="totalCount">Total: 0</div>
  </div>

  <h2>Colaboradores</h2>
  <table id="colabTable">
    <thead>
      <tr><th>ID</th><th>Nome</th><th>E-mail</th><th>Matrícula</th><th>Cargo</th><th>Turno</th><th>Ações</th></tr>
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
    <p class="small">Pesquise o nome, matrícula ou e-mail:</p>
    <input id="pontoSearch" placeholder="Digite aqui..." />
    <div id="pontoResults" style="margin-top:12px;max-height:320px;overflow:auto"></div>
    <div style="display:flex;gap:8px;margin-top:12px">
      <button id="cancelPonto">Cancelar</button>
    </div>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
import { getFirestore, collection, doc, setDoc, getDoc, updateDoc, deleteDoc, addDoc, onSnapshot, query, orderBy, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

/* Firebase config */
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
  if (value === SENHA_CORRETA) {
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

function startRealtime(){
  const qCol = query(colabsColl, orderBy('nome'));
  onSnapshot(qCol, snap=>{
    colaboradores = snap.docs.map(d=>({ docId:d.id, ...d.data() }));
    renderColaboradores();
    totalCount.textContent = 'Total: ' + colaboradores.length;
  });

  const qRegs = query(regsColl, orderBy('timestamp','desc'));
  onSnapshot(qRegs, snap=>{
    registrosCache = snap.docs.map(d=>({ id:d.id, ...d.data() }));
    renderRegistrosSeparados();
  });
}

/* render colaboradores */
function renderColaboradores(){
  const term = (searchInput.value||'').toLowerCase().trim();
  const filtered = term ? colaboradores.filter(c=> 
    (c.nome||'').toLowerCase().includes(term) ||
    (c.matricula||'').toLowerCase().includes(term) ||
    (c.email||'').toLowerCase().includes(term)
  ) : colaboradores;

  colabBody.innerHTML = '';
  if(!filtered.length){ colabBody.innerHTML = '<tr><td colspan="7" style="text-align:center">Nenhum colaborador encontrado.</td></tr>'; return; }
  
  filtered.forEach(c=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${c.docId}</td>
      <td>${c.nome}</td>
      <td>${c.email||''}</td>
      <td>${c.matricula||''}</td>
      <td>${c.cargo||''}</td>
      <td>${c.turno||''}</td>
      <td>
        <button class="reg" onclick="openPontoModal('${c.docId}')">Bater Ponto</button>
        <button class="edit" onclick="abrirEditarColab('${c.docId}')">Editar</button>
        <button class="del" onclick="removerColab('${c.docId}')">Excluir</button>
        <br/><button class="detailsToggle" onclick="toggleHistory('${c.docId}', this)">📜 Ver histórico</button>
      </td>`;
    colabBody.appendChild(tr);

    const trDetails = document.createElement('tr');
    trDetails.id = `histrow-${c.docId}`;
    trDetails.style.display = 'none';
    trDetails.innerHTML = `<td colspan="7"><div id="history-${c.docId}" class="historyList">Carregando histórico...</div></td>`;
    colabBody.appendChild(trDetails);
  });
}

/* render registros */
function renderRegistrosSeparados(){
  const entradas = registrosCache.filter(r=> (r.tipo||'').toLowerCase() === 'entrada');
  const saidas = registrosCache.filter(r=> (r.tipo||'').toLowerCase() === 'saída' || (r.tipo||'').toLowerCase() === 'saida');

  entradaBody.innerHTML=''; saidasBody.innerHTML='';
  entradas.slice(0,200).forEach(r=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${r.colaboradorId||''}</td><td>${r.nome||''}</td><td>${r.matricula||''}</td><td>${r.hora||''}</td>`;
    entradaBody.appendChild(tr);
  });
  saidas.slice(0,200).forEach(r=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${r.colaboradorId||''}</td><td>${r.nome||''}</td><td>${r.matricula||''}</td><td>${r.hora||''}</td>`;
    saidaBody.appendChild(tr);
  });
}

/* histórico */
window.toggleHistory = function(colabId, btn){
  const row = document.getElementById(`histrow-${colabId}`);
  const container = document.getElementById(`history-${colabId}`);
  if(!row) return;
  if(row.style.display==='none'){
    row.style.display='';
    btn.textContent='📜 Ocultar histórico';
    const items = registrosCache.filter(r=> (r.colaboradorId||'')===colabId);
    container.innerHTML = items.length ? items.map(p=>`<div style="padding:6px;border-bottom:1px solid #eee"><strong>${p.tipo}</strong> — ${p.hora||''}</div>`).join('') : '<div class="small">Nenhum registro.</div>';
  } else {
    row.style.display='none';
    btn.textContent='📜 Ver histórico';
  }
};

/* modal colaboradores */
const colabModal = document.getElementById('colabModal');
const colabForm = document.getElementById('colabForm');
document.getElementById('addColabBtn').addEventListener('click', ()=> openColabModal());
document.getElementById('cancelColab').addEventListener('click', ()=> colabModal.style.display='none');
document.querySelectorAll('.close').forEach(b=>b.addEventListener('click', e=> colabModal.style.display='none'));

async function openColabModal(id=null){
  document.getElementById('delColab').style.display = id ? 'inline-block':'none';
  document.getElementById('modalTitle').textContent = id?'Editar Colaborador':'Adicionar Colaborador';
  document.getElementById('colabId').value = id||'';
  if(id){
    const d = await getDoc(doc(db,'colaboradores',id));
    if(d.exists()){
      const c = d.data();
      document.getElementById('colabNome').value=c.nome||'';
      document.getElementById('colabEmail').value=c.email||'';
      document.getElementById('colabMatricula').value=c.matricula||'';
      document.getElementById('colabCargo').value=c.cargo||'';
      document.getElementById('colabTurno').value=c.turno||'';
    }
  } else { colabForm.reset(); }
  colabModal.style.display='flex';
}

/* salvar colaborador */
colabForm.addEventListener('submit', async e=>{
  e.preventDefault();
  const id = document.getElementById('colabId').value||null;
  const nome = document.getElementById('colabNome').value.trim();
  const email = document.getElementById('colabEmail').value.trim();
  const matricula = document.getElementById('colabMatricula').value.trim();
  const cargo = document.getElementById('colabCargo').value.trim();
  const turno = document.getElementById('colabTurno').value;
  if(!nome||!email||!matricula){ alert('Nome, e-mail e matrícula obrigatórios'); return; }

  if(id){
    await updateDoc(doc(db,'colaboradores',id), {nome,email,matricula,cargo,turno});
    alert('Colaborador atualizado');
  } else {
    await addDoc(colabsColl,{nome,email,matricula,cargo,turno,criadoEm:serverTimestamp()});
    alert('Colaborador adicionado');
  }
  colabModal.style.display='none';
});

/* excluir colaborador */
document.getElementById('delColab').addEventListener('click', async ()=>{
  const id = document.getElementById('colabId').value;
  if(id && confirm('Deseja realmente excluir este colaborador?')){
    await deleteDoc(doc(db,'colaboradores',id));
    colabModal.style.display='none';
  }
});
window.abrirEditarColab = openColabModal;

/* modal ponto */
const pontoModal = document.getElementById('pontoModal');
document.getElementById('cancelPonto').addEventListener('click', ()=> pontoModal.style.display='none');

window.openPontoModal = function(colabId){
  pontoModal.style.display='flex';
  const resultsDiv = document.getElementById('pontoResults');
  resultsDiv.innerHTML = '';
  const c = colaboradores.find(c=>c.docId===colabId);
  if(c){
    const div = document.createElement('div');
    div.style.marginBottom='6px';
    div.innerHTML = `<strong>${c.nome}</strong> — ${c.matricula} 
      <button style="margin-left:8px;padding:4px 8px;background:#4CAF50;color:white;border:none;border-radius:4px;cursor:pointer" 
      onclick="baterPonto('${c.docId}','Entrada')">Entrada</button>
      <button style="margin-left:4px;padding:4px 8px;background:#F44336;color:white;border:none;border-radius:4px;cursor:pointer"
      onclick="baterPonto('${c.docId}','Saída')">Saída</button>`;
    resultsDiv.appendChild(div);
  }
};

window.baterPonto = async function(colabId,tipo){
  const c = colaboradores.find(c=>c.docId===colabId);
  if(!c) return;
  await addDoc(regsColl,{
    colaboradorId: colabId,
    nome: c.nome,
    matricula: c.matricula,
    tipo: tipo,
    hora: new Date().toLocaleString('pt-BR'),
    timestamp: serverTimestamp()
  });
  alert(`Ponto ${tipo} registrado!`);
};

/* filtro de busca */
searchInput.addEventListener('input', renderColaboradores);

/* export Excel */
document.getElementById('exportExcelBtn').addEventListener('click', ()=>{
  const wb = XLSX.utils.book_new();
  const data = colaboradores.map(c=>({
    ID:c.docId, Nome:c.nome, Email:c.email, Matricula:c.matricula, Cargo:c.cargo, Turno:c.turno
  }));
  const ws = XLSX.utils.json_to_sheet(data);
  XLSX.utils.book_append_sheet(wb,ws,"Colaboradores");
  XLSX.writeFile(wb,"Colaboradores.xlsx");
});
</script>
</body>
</html>
