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

<!-- Modal Ponto (pesquisa/gaveta) -->
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

<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
import {
  getFirestore, collection, doc, setDoc, getDoc, updateDoc, deleteDoc,
  onSnapshot, query, orderBy, addDoc, serverTimestamp, runTransaction,
  where, limit, getDocs
} from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

/* firebaseConfig usado nos exemplos anteriores */
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
function mostrarBoasVindas(){ const hora=new Date().getHours(); let saud='Olá'; if(hora<12) saud='Bom dia'; else if(hora<18) saud='Boa tarde'; else saud='Boa noite'; document.getElementById('boasVindas').textContent=`${saud}, ${NOME_USUARIO}! 👋`; }
window.onload = ()=>{ if (localStorage.getItem('autenticado')==='true'){ loginScreen.style.display='none'; conteudo.style.display='block'; mostrarBoasVindas(); startRealtime(); } };

/* refs */
const colabsColl = collection(db,'colaboradores');
const regsColl = collection(db,'registros');

const colabBody = document.getElementById('colabBody');
const entradaBody = document.getElementById('entradaBody');
const saidaBody = document.getElementById('saidaBody');
const totalCount = document.getElementById('totalCount');
const searchInput = document.getElementById('search');

let colaboradores = []; // cache ordenado alfabeticamente
let registrosCache = []; // cache de registros

/* start realtime listeners */
function startRealtime(){
  // colaboradores ordenados por nome
  const qCol = query(colabsColl, orderBy('nome'));
  onSnapshot(qCol, (snap)=>{
    const list = [];
    snap.forEach(d=> list.push({ docId: d.id, ...d.data() }));
    colaboradores = list; // já ordenado por nome
    renderColaboradores();
    totalCount.textContent = 'Total: ' + colaboradores.length;
  });

  // registros (escuta últimos 1000 ordenados por timestamp desc)
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
  if (!filtered.length) { colabBody.innerHTML = '<tr><td colspan="7" style="text-align:center">Nenhum colaborador encontrado.</td></tr>'; return; }
  filtered.forEach(c=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${escapeHtml(c.id || c.idNum || c.docId)}</td>
      <td>${escapeHtml(c.nome)}</td>
      <td>${escapeHtml(c.email||'')}</td>
      <td>${escapeHtml(c.matricula||'')}</td>
      <td>${escapeHtml(c.cargo||'')}</td>
      <td>${escapeHtml(c.turno||'')}</td>
      <td>
        <button class="reg" onclick="openPontoModal('${c.docId || c.id || c.idNum}')">Bater Ponto</button>
        <button class="edit" onclick="abrirEditarColab('${c.docId || c.id || c.idNum}')">Editar</button>
        <button class="del" onclick="removerColab('${c.docId || c.id || c.idNum}')">Excluir</button>
        <br/><button class="detailsToggle" onclick="toggleHistory('${c.docId || c.id || c.idNum}', this)">📜 Ver histórico</button>
      </td>`;
    colabBody.appendChild(tr);

    const trDetails = document.createElement('tr');
    trDetails.id = `histrow-${c.docId || c.id || c.idNum}`;
    trDetails.style.display = 'none';
    trDetails.innerHTML = `<td colspan="7"><div id="history-${c.docId || c.id || c.idNum}" class="historyList">Carregando histórico...</div></td>`;
    colabBody.appendChild(trDetails);
  });
}

/* render registros separados em Entradas e Saídas */
function renderRegistrosSeparados(){
  // filtra registrosCache para entradas e saidas e exibe em cada tabela
  const entradas = registrosCache.filter(r=> (r.tipo||'').toLowerCase() === 'entrada');
  const saidas = registrosCache.filter(r=> (r.tipo||'').toLowerCase() === 'saída' || (r.tipo||'').toLowerCase() === 'saida' || (r.tipo||'').toLowerCase() === 'saida');
  // preencher entradas (limit 200)
  entradaBody.innerHTML = '';
  entradas.slice(0,200).forEach(r=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${escapeHtml(r.colaboradorId||r.colabId||'')}</td>
                    <td>${escapeHtml(r.nome||'')}</td>
                    <td>${escapeHtml(r.matricula||'')}</td>
                    <td>${ r.hora || (r.timestamp && r.timestamp.toDate ? r.timestamp.toDate().toLocaleString() : '') }</td>`;
    entradaBody.appendChild(tr);
  });
  // preencher saidas
  saidaBody.innerHTML = '';
  saidas.slice(0,200).forEach(r=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${escapeHtml(r.colaboradorId||r.colabId||'')}</td>
                    <td>${escapeHtml(r.nome||'')}</td>
                    <td>${escapeHtml(r.matricula||'')}</td>
                    <td>${ r.hora || (r.timestamp && r.timestamp.toDate ? r.timestamp.toDate().toLocaleString() : '') }</td>`;
    saidaBody.appendChild(tr);
  });
}

/* Toggle histórico por colaborador (recolhível) */
window.toggleHistory = function(colabId, btn){
  const row = document.getElementById(`histrow-${colabId}`);
  const container = document.getElementById(`history-${colabId}`);
  if (!row) return;
  if (row.style.display === 'none') {
    row.style.display = '';
    btn.textContent = '📜 Ocultar histórico';
    // preencher histórico por colaborador do cache
    const items = registrosCache.filter(r=> (r.colaboradorId||'') === colabId);
    if (!items.length) container.innerHTML = '<div class="small">Nenhum registro.</div>'; else {
      container.innerHTML = items.map(p=>`<div style="padding:6px;border-bottom:1px solid #eee"><strong>${escapeHtml(p.tipo)}</strong> — ${escapeHtml(p.hora || (p.timestamp && p.timestamp.toDate ? p.timestamp.toDate().toLocaleString() : ''))}</div>`).join('');
    }
  } else {
    row.style.display = 'none';
    btn.textContent = '📜 Ver histórico';
  }
};

/* ---------- Modal Colaborador: adicionar / editar ---------- */
const colabModal = document.getElementById('colabModal');
const colabForm = document.getElementById('colabForm');
document.getElementById('addColabBtn').addEventListener('click', ()=> openColabModal());
document.getElementById('cancelColab').addEventListener('click', ()=> closeModal('colabModal'));
document.querySelectorAll('.close').forEach(b=>b.addEventListener('click', (ev)=> closeModal(ev.target.dataset.target || ev.target.closest('.modal').id)));

async function openColabModal(id){
  const modal = document.getElementById('colabModal');
  document.getElementById('delColab').style.display = id ? 'inline-block' : 'none';
  document.getElementById('modalTitle').textContent = id ? 'Editar Colaborador' : 'Adicionar Colaborador';
  document.getElementById('colabId').value = id || '';
  if (id){
    const d = await getDoc(doc(db,'colaboradores',String(id)));
    const data = d.exists() ? d.data() : {};
    document.getElementById('colabNome').value = data.nome || '';
    document.getElementById('colabEmail').value = data.email || '';
    document.getElementById('colabMatricula').value = data.matricula || '';
    document.getElementById('colabCargo').value = data.cargo || '';
    document.getElementById('colabTurno').value = data.turno || '';
  } else {
    colabForm.reset();
  }
  modal.style.display = 'flex';
}
function closeModal(id){ document.getElementById(id).style.display='none'; }

/* salvar colaborador (novo ou editar) */
colabForm.addEventListener('submit', async (e)=>{
  e.preventDefault();
  const id = document.getElementById('colabId').value || null;
  const nome = document.getElementById('colabNome').value.trim();
  const email = document.getElementById('colabEmail').value.trim();
  const matricula = document.getElementById('colabMatricula').value.trim();
  const cargo = document.getElementById('colabCargo').value.trim();
  const turno = document.getElementById('colabTurno').value;
  if (!nome || !email || !matricula) { alert('Nome, e-mail e matrícula obrigatórios'); return; }

  try {
    if (id) {
      await updateDoc(doc(db,'colaboradores',String(id)), { nome, email, matricula, cargo, turno });
      alert('Colaborador atualizado');
    } else {
      // novo: buscar próximo id numérico via transação e criar doc com id numérico como chave
      const counterRef = doc(db,'counters','colaboradores');
      const newId = await runTransaction(db, async (t)=>{
        const snap = await t.get(counterRef);
        if (!snap.exists()){
          t.set(counterRef, { nextId: 2 });
          return 1;
        } else {
          const cur = snap.data().nextId;
          t.update(counterRef, { nextId: cur + 1 });
          return cur;
        }
      });
      await setDoc(doc(db,'colaboradores',String(newId)), { id: newId, nome, email, matricula, cargo, turno, criadoEm: serverTimestamp() });
      alert('Colaborador adicionado (ID: ' + newId + ')');
    }
    closeModal('colabModal');
  } catch (err) {
    console.error(err); alert('Erro: ' + (err.message||err));
  }
});

/* excluir colaborador */
document.getElementById('delColab').addEventListener('click', async ()=>{
  const id = document.getElementById('colabId').value;
  if (!id) return;
  if (!confirm('Confirma exclusão?')) return;
  try { await deleteDoc(doc(db,'colaboradores',String(id))); closeModal('colabModal'); alert('Excluído'); } catch (err){ console.error(err); alert('Erro: '+(err.message||err)); }
});

/* remover diretamente (botão na linha) */
window.removerColab = async function(id){
  if (!confirm('Deseja excluir este colaborador?')) return;
  try { await deleteDoc(doc(db,'colaboradores',String(id))); } catch (err){ console.error(err); alert('Erro: '+(err.message||err)); }
};

/* ---------- Modal Ponto (pesquisa/gaveta) ---------- */
const pontoModal = document.getElementById('pontoModal');
const pontoResults = document.getElementById('pontoResults');
const pontoSearch = document.getElementById('pontoSearch');
document.getElementById('cancelPonto').addEventListener('click', ()=> pontoModal.style.display='none');

document.getElementById('openPontoModalBtn').addEventListener('click', ()=> openPontoModal());
window.openPontoModal = function(id=null){ openPontoModal(id); };

function openPontoModal(highlightId=null){
  pontoModal.style.display='flex';
  pontoSearch.value = '';
  renderPontoResults(colaboradores, highlightId);
}
pontoSearch.addEventListener('input', ()=> renderPontoResults(colaboradores));

function renderPontoResults(list, highlightId=null){
  const term = (pontoSearch.value||'').toLowerCase().trim();
  const filtered = term ? list.filter(c => (c.nome||'').toLowerCase().includes(term) || (c.matricula||'').toLowerCase().includes(term)) : list;
  if (!filtered.length) { pontoResults.innerHTML = '<div class="small">Nenhum colaborador encontrado.</div>'; return; }
  pontoResults.innerHTML = filtered.map(c=> `
    <div style="padding:8px;border-bottom:1px solid #eee;display:flex;align-items:center;gap:10px">
      <div style="flex:1">
        <strong>${escapeHtml(c.nome)}</strong><div class="small">${escapeHtml(c.email||'')} • ${escapeHtml(c.matricula||'')}</div>
      </div>
      <div style="display:flex;gap:6px">
        <button class="reg" onclick='registrarPonto("${c.docId||c.id||c.idNum}","Entrada")'>Entrada</button>
        <button class="reg" onclick='registrarPonto("${c.docId||c.id||c.idNum}","Saída")'>Saída</button>
      </div>
    </div>
  `).join('');
}

/* ---------- Registrar ponto ---------- */
window.registrarPonto = async function(colabId, tipo){
  try {
    const cdoc = await getDoc(doc(db,'colaboradores',String(colabId)));
    const c = cdoc.exists() ? cdoc.data() : null;
    if (!c) { alert('Colaborador não encontrado'); return; }
    const payload = {
      colaboradorId: String(colabId),
      nome: c.nome || '',
      matricula: c.matricula || '',
      tipo,
      hora: new Date().toLocaleString('pt-BR'),
      timestamp: serverTimestamp()
    };
    await addDoc(regsColl, payload);
    alert('Ponto ' + tipo + ' registrado para ' + c.nome);
    pontoModal.style.display='none';
  } catch (err) {
    console.error(err); alert('Erro ao registrar: ' + (err.message||err));
  }
};

/* ---------- Exportar Excel (Entradas e Saídas) ---------- */
document.getElementById('exportExcelBtn').addEventListener('click', async ()=>{
  try {
    const snap = await getDocs(regsColl);
    const entradas = [], saidas = [];
    snap.forEach(d=>{
      const p = d.data();
      const row = {
        ID: d.id,
        Nome: p.nome || '',
        Matricula: p.matricula || '',
        Tipo: p.tipo || '',
        'Data/Hora': p.hora || (p.timestamp && p.timestamp.toDate ? p.timestamp.toDate().toLocaleString() : '')
      };
      if ((p.tipo||'').toLowerCase().includes('entrada')) entradas.push(row);
      else saidas.push(row);
    });
    const wb = XLSX.utils.book_new();
    const wsE = XLSX.utils.json_to_sheet(entradas);
    const wsS = XLSX.utils.json_to_sheet(saidas);
    XLSX.utils.book_append_sheet(wb, wsE, 'Entradas');
    XLSX.utils.book_append_sheet(wb, wsS, 'Saídas');
    XLSX.writeFile(wb, 'registros_ponto.xlsx');
  } catch (err) {
    console.error(err); alert('Erro ao exportar: ' + (err.message||err));
  }
});

/* ---------- util ---------- */
function escapeHtml(str){ if (str===undefined || str===null) return ''; return String(str).replace(/[&<>"'`=\/]/g, s=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;','/':'&#x2F;','`':'&#x60;','=':'&#x3D'}[s])); }

/* pesquisa em tempo real */
searchInput.addEventListener('input', ()=> renderColaboradores());

</script>
</body>
</html>
