<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Sistema de Ponto Eletrônico</title>
<style>
  /* Mantive o layout visual bem próximo do seu original */
  body { font-family: Arial, sans-serif; background:#f5f5f5; margin:0; padding:20px; }
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
  /* Modal simple */
  .modal { display:none; position:fixed; z-index:2; left:0; top:0; width:100%; height:100%; background:rgba(0,0,0,0.4); align-items:center; justify-content:center;}
  .modal-content { background:white; padding:18px; border-radius:8px; width:90%; max-width:520px; box-shadow:0 8px 30px rgba(0,0,0,.12)}
  .close { float:right; font-size:22px; cursor:pointer; border:none;background:none}
  /* Login screen */
  #loginScreen { position:fixed; top:0; left:0; width:100%; height:100%; background:#222; color:white; display:flex; align-items:center; justify-content:center; flex-direction:column; gap:10px; z-index:9; }
  #loginScreen input { padding:10px; font-size:18px; border-radius:6px; border:none; text-align:center;}
  #loginScreen button { background:#4CAF50; color:white; padding:10px 20px; border:none; border-radius:6px; font-size:16px; cursor:pointer;}
  /* Mobile tweaks */
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
    <button class="reg" id="entradaBtn">Registrar Entrada</button>
    <button class="reg" id="saidaBtn">Registrar Saída</button>
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

  <h2 style="margin-top:18px">Registros Recentes</h2>
  <table id="regTable">
    <thead><tr><th>ID</th><th>Nome</th><th>Matrícula</th><th>Tipo</th><th>Data/Hora</th></tr></thead>
    <tbody id="regBody"></tbody>
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

<!-- Modal Registrar Ponto (pesquisa/gaveta) -->
<div id="pontoModal" class="modal">
  <div class="modal-content">
    <button class="close" data-target="pontoModal">&times;</button>
    <h3 id="pontoTitle">Registrar Ponto</h3>
    <p class="small">Pesquise o nome ou matrícula e selecione o colaborador:</p>
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
  getFirestore, collection, addDoc, onSnapshot, query, orderBy, serverTimestamp,
  getDocs, doc, getDoc, setDoc, updateDoc, deleteDoc, where, limit
} from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

/* Seu firebaseConfig */
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

/* ---------- Autenticação por senha simples (client-side) ---------- */
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
  const hora = new Date().getHours();
  let saudacao = "Olá";
  if (hora < 12) saudacao = "Bom dia";
  else if (hora < 18) saudacao = "Boa tarde";
  else saudacao = "Boa noite";
  document.getElementById('boasVindas').textContent = `${saudacao}, ${NOME_USUARIO}! 👋`;
}
window.onload = ()=>{
  if (localStorage.getItem('autenticado') === 'true') {
    loginScreen.style.display='none';
    conteudo.style.display='block';
    mostrarBoasVindas();
    startRealtime();
  }
};

/* ---------- Referências ---------- */
const colabBody = document.getElementById('colabBody');
const regBody = document.getElementById('regBody');
const totalCount = document.getElementById('totalCount');
const searchInput = document.getElementById('search');
const addColabBtn = document.getElementById('addColabBtn');
const editColabBtn = document.getElementById('editColabBtn');
const deleteColabBtn = document.getElementById('deleteColabBtn');
const exportExcelBtn = document.getElementById('exportExcelBtn');

const colabsRef = collection(db, 'colaboradores');
const pontosRef = collection(db, 'pontos');

/* ---------- Estado local (espelho) ---------- */
let colaboradores = []; // array de {id, ...}
let pontosMap = {};    // { colaboradorId: [pontoDoc,...] }

/* ---------- Realtime listeners ---------- */
function startRealtime(){
  // colaboradores em tempo real
  onSnapshot(colabsRef, (snap)=>{
    const list = [];
    snap.forEach(doc => list.push({ id: doc.id, ...doc.data() }));
    colaboradores = list.sort((a,b)=> (a.nome||'').localeCompare(b.nome||''));
    renderColaboradores(colaboradores);
    totalCount.textContent = 'Total: ' + colaboradores.length;
  });

  // pontos (últimos 200) em tempo real — preenche pontosMap
  const qPontos = query(pontosRef, orderBy('timestamp','desc'), limit(200));
  onSnapshot(qPontos, (snap)=>{
    pontosMap = {};
    snap.forEach(doc => {
      const p = { id: doc.id, ...doc.data() };
      const cid = p.colaboradorId;
      if (!pontosMap[cid]) pontosMap[cid] = [];
      pontosMap[cid].push(p);
    });
    renderRegistrosList(); // atualiza lista geral (registros recentes)
    // se algum detalhes estiver aberto, ele também puxará da pontosMap quando abrir
  });
}

/* ---------- Render colaborador list + detalhes recolhíveis ---------- */
function renderColaboradores(list){
  const term = searchInput.value.trim().toLowerCase();
  const filtered = term ? list.filter(c => (c.nome||'').toLowerCase().includes(term) || (c.matricula||'').toLowerCase().includes(term)) : list;
  colabBody.innerHTML = '';
  if (filtered.length === 0) {
    colabBody.innerHTML = '<tr><td colspan="7" style="text-align:center">Nenhum colaborador encontrado.</td></tr>';
    return;
  }

  filtered.forEach(c=>{
    // linha principal
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td style="vertical-align:middle">${escapeHtml(c.id)}</td>
      <td style="vertical-align:middle">${escapeHtml(c.nome)}</td>
      <td style="vertical-align:middle">${escapeHtml(c.email||'')}</td>
      <td style="vertical-align:middle">${escapeHtml(c.matricula||'')}</td>
      <td style="vertical-align:middle">${escapeHtml(c.cargo||'')}</td>
      <td style="vertical-align:middle">${escapeHtml(c.turno||'')}</td>
      <td style="vertical-align:middle">
        <button class="reg" onclick="openPontoModal('${c.id}')">Bater Ponto</button>
        <button class="edit" onclick="abrirEditarColab('${c.id}')">Editar</button>
        <button class="del" onclick="removerColab('${c.id}')">Excluir</button>
        <br/><button class="detailsToggle" onclick="toggleHistory('${c.id}', this)">📜 Ver histórico</button>
      </td>
    `;
    colabBody.appendChild(tr);

    // linha de detalhes (colspan 7)
    const trDetails = document.createElement('tr');
    trDetails.id = `histrow-${c.id}`;
    trDetails.style.display = 'none';
    trDetails.innerHTML = `<td colspan="7">
      <div id="history-${c.id}" class="historyList">Carregando histórico...</div>
    </td>`;
    colabBody.appendChild(trDetails);
  });
}

/* ---------- Toggle histórico (abre/fecha) ---------- */
window.toggleHistory = function(colabId, btn){
  const row = document.getElementById(`histrow-${colabId}`);
  const container = document.getElementById(`history-${colabId}`);
  if (row.style.display === 'none') {
    row.style.display = '';
    btn.textContent = '📜 Ocultar histórico';
    renderHistoryFor(colabId, container);
  } else {
    row.style.display = 'none';
    btn.textContent = '📜 Ver histórico';
  }
};

function renderHistoryFor(colabId, container){
  const items = pontosMap[colabId] || [];
  if (!items.length) {
    container.innerHTML = '<div class="small">Nenhum registro encontrado.</div>';
    return;
  }
  // mostrar tudo (já estão ordenados por timestamp desc)
  container.innerHTML = items.map(p=>`<div style="padding:6px;border-bottom:1px solid #eee">
      <strong>${escapeHtml(p.tipo)}</strong> — ${escapeHtml(p.nome)} — ${escapeHtml(p.matricula)} — ${p.hora || (p.timestamp && p.timestamp.toDate ? p.timestamp.toDate().toLocaleString() : '')}
    </div>`).join('');
}

/* ---------- Registros recentes (tabela topo) ---------- */
function renderRegistrosList(){
  const rows = [];
  // percorre pontosMap para criar lista mesclada (já ordenada por timestamp desc no snapshot)
  for (const cid in pontosMap) {
    pontosMap[cid].forEach(p => rows.push(p));
  }
  // ordenar por timestamp desc (fallback para campo hora)
  rows.sort((a,b)=>{
    const ta = a.timestamp && a.timestamp.toDate ? a.timestamp.toDate().getTime() : 0;
    const tb = b.timestamp && b.timestamp.toDate ? b.timestamp.toDate().getTime() : 0;
    return tb - ta;
  });
  const top = rows.slice(0, 80);
  regBody.innerHTML = '';
  top.forEach(r=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${escapeHtml(r.id)}</td>
      <td>${escapeHtml(r.nome || '')}</td>
      <td>${escapeHtml(r.matricula || '')}</td>
      <td>${escapeHtml(r.tipo || '')}</td>
      <td>${ r.hora || (r.timestamp && r.timestamp.toDate ? r.timestamp.toDate().toLocaleString() : '') }</td>`;
    regBody.appendChild(tr);
  });
}

/* ---------- Pesquisa (input) ---------- */
searchInput.addEventListener('input', ()=> renderColaboradores(colaboradores));

/* ---------- Abrir modal adicionar / editar ---------- */
addColabBtn.addEventListener('click', ()=> abrirNovoColab());
document.getElementById('cancelColab').addEventListener('click', ()=> closeModal('colabModal'));
document.querySelectorAll('.close').forEach(b=>b.addEventListener('click', (ev)=> closeModal(ev.target.dataset.target || ev.target.closest('.modal').id)));

function abrirNovoColab(){
  openColabModal();
}
function abrirEditarColab(id){
  openColabModal(id);
}

async function openColabModal(id){
  const modal = document.getElementById('colabModal');
  document.getElementById('delColab').style.display = id ? 'inline-block' : 'none';
  document.getElementById('modalTitle').textContent = id ? 'Editar Colaborador' : 'Adicionar Colaborador';
  document.getElementById('colabId').value = id || '';
  if (id) {
    // buscar documento
    const d = await getDoc(doc(db, 'colaboradores', id));
    const data = d.data() || {};
    document.getElementById('colabNome').value = data.nome || '';
    document.getElementById('colabEmail').value = data.email || '';
    document.getElementById('colabMatricula').value = data.matricula || '';
    document.getElementById('colabCargo').value = data.cargo || '';
    document.getElementById('colabTurno').value = data.turno || '';
  } else {
    document.getElementById('colabForm').reset();
  }
  modal.style.display = 'flex';
}

document.getElementById('colabForm').addEventListener('submit', async (e)=>{
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
      await updateDoc(doc(db, 'colaboradores', id), { nome, email, matricula, cargo, turno });
      alert('Colaborador atualizado');
    } else {
      await addDoc(colabsRef, { nome, email, matricula, cargo, turno, criadoEm: serverTimestamp() });
      alert('Colaborador adicionado');
    }
    closeModal('colabModal');
  } catch (err) {
    console.error(err); alert('Erro: ' + (err.message||err));
  }
});

document.getElementById('delColab').addEventListener('click', async ()=>{
  const id = document.getElementById('colabId').value;
  if (!id) return;
  if (!confirm('Confirma exclusão?')) return;
  try {
    await deleteDoc(doc(db,'colaboradores',id));
    closeModal('colabModal');
  } catch (err) {
    console.error(err); alert('Erro ao excluir: ' + (err.message||err));
  }
});

function closeModal(id){
  document.getElementById(id).style.display = 'none';
}

/* ---------- PONTO modal (pesquisa gaveta) ---------- */
const pontoModal = document.getElementById('pontoModal');
const pontoResults = document.getElementById('pontoResults');
const pontoSearch = document.getElementById('pontoSearch');
document.getElementById('cancelPonto').addEventListener('click', ()=> pontoModal.style.display='none');

window.openPontoModal = function(colabId=null){
  // abrir modal e opcionalmente focar num colaborador
  pontoModal.style.display='flex';
  pontoResults.innerHTML = '';
  pontoSearch.value = '';
  renderPontoResults(colaboradores);
  if (colabId) {
    // filtrar para mostrar primeiro
    const sel = colaboradores.find(c=>c.id===colabId);
    if (sel) {
      pontoResults.innerHTML = renderPontoCard(sel);
    }
  } else {
    renderPontoResults(colaboradores);
  }
};

// pesquisa na gaveta
pontoSearch.addEventListener('input', ()=> renderPontoResults(colaboradores));

function renderPontoResults(list){
  const term = (pontoSearch.value||'').toLowerCase().trim();
  const filtered = term ? list.filter(c => (c.nome||'').toLowerCase().includes(term) || (c.matricula||'').toLowerCase().includes(term)) : list;
  if (!filtered.length) { pontoResults.innerHTML = '<div class="small">Nenhum colaborador encontrado.</div>'; return; }
  pontoResults.innerHTML = filtered.map(c=> renderPontoCardHtml(c)).join('');
}
function renderPontoCardHtml(c){
  return `<div style="padding:8px;border-bottom:1px solid #eee;display:flex;align-items:center;gap:10px">
    <div style="flex:1">
      <strong>${escapeHtml(c.nome)}</strong><div class="small">${escapeHtml(c.email||'')} • ${escapeHtml(c.matricula||'')}</div>
    </div>
    <div style="display:flex;gap:6px">
      <button class="reg" onclick='registrarPonto("${c.id}","Entrada")'>Entrada</button>
      <button class="reg" onclick='registrarPonto("${c.id}","Saída")'>Saída</button>
    </div>
  </div>`;
}
function renderPontoCard(c){
  return renderPontoCardHtml(c);
}

/* ---------- Registrar ponto ---------- */
window.registrarPonto = async function(colabId, tipo){
  try {
    const d = await getDoc(doc(db,'colaboradores',colabId));
    const c = d.exists() ? d.data() : null;
    if (!c) { alert('Colaborador não encontrado'); return; }
    const payload = {
      colaboradorId: colabId,
      nome: c.nome || '',
      matricula: c.matricula || '',
      tipo,
      hora: new Date().toLocaleString('pt-BR'),
      timestamp: serverTimestamp()
    };
    await addDoc(pontosRef, payload);
    alert(`Ponto ${tipo} registrado para ${c.nome}`);
    // atualizar histórico exibido (pontosMap atualizará via onSnapshot)
    pontoModal.style.display='none';
  } catch (err) {
    console.error(err); alert('Erro ao registrar: ' + (err.message||err));
  }
};

/* ---------- Exportar Excel (Entradas / Saídas) ---------- */
exportExcelBtn.addEventListener('click', async ()=>{
  try {
    // buscar todos os pontos (ou os últimos 1000). Aqui pegamos todos para export
    const snap = await getDocs(pontosRef);
    const rowsEntrada = [], rowsSaida = [];
    snap.forEach(doc=>{
      const p = doc.data();
      const row = {
        ID: doc.id,
        Nome: p.nome || '',
        Matricula: p.matricula || '',
        Tipo: p.tipo || '',
        'Data/Hora': p.hora || (p.timestamp && p.timestamp.toDate ? p.timestamp.toDate().toLocaleString() : '')
      };
      if ((p.tipo||'').toLowerCase().includes('entrada')) rowsEntrada.push(row);
      else rowsSaida.push(row);
    });
    // construir workbook
    const wb = XLSX.utils.book_new();
    const wsE = XLSX.utils.json_to_sheet(rowsEntrada);
    const wsS = XLSX.utils.json_to_sheet(rowsSaida);
    XLSX.utils.book_append_sheet(wb, wsE, 'Entradas');
    XLSX.utils.book_append_sheet(wb, wsS, 'Saidas');
    XLSX.writeFile(wb, 'registros_ponto.xlsx');
  } catch (err) {
    console.error(err); alert('Erro ao exportar: ' + (err.message||err));
  }
});

/* ---------- Util helpers ---------- */
function escapeHtml(str){ if (str===undefined || str===null) return ''; return String(str).replace(/[&<>"'`=\/]/g, s=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;','/':'&#x2F;','`':'&#x60;','=':'&#x3D;'}[s])); }

async function removerColab(id){
  if (!confirm('Confirma exclusão?')) return;
  try { await deleteDoc(doc(db,'colaboradores',id)); alert('Excluído'); } catch (err){ console.error(err); alert('Erro ao excluir: '+(err.message||err)); }
}

/* ---------- Expor alguns métodos ao escopo window para uso em onClick gerado HTML ---------- */
window.abrirEditarColab = function(id){ openColabModal(id); }
window.openPontoModal = function(id){ pontoModal.style.display='flex'; renderPontoResults(colaboradores); if (id) pontoSearch.value = ''; }

/* ---------- Inicial (se autenticado, startRealtime() já é chamado no onload) ---------- */
</script>
</body>
</html>
