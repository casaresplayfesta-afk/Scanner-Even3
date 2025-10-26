<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Ponto Eletrônico - Final</title>
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
  .del{background:var(--red);color:#fff}
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
    <button id="loginBtn" class="add" style="width:92%;margin-top:6px">Entrar</button>
    <p id="loginMsg" style="color:crimson;margin-top:8px;height:18px"></p>
    <p style="font-size:12px;color:#666;margin-top:6px">Usuário: <b>admin</b> / Senha: <b>1234</b></p>
  </div>
</div>

<header>
  <div style="display:flex;gap:12px;align-items:center">
    <div class="logo">Ponto Eletrônico</div>
    <div id="status" class="muted">Offline • Local Storage</div>
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

<!-- dependência XLSX -->
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

<script>
/* ---------------- CONFIG ---------------- */
const LOGIN_USER = 'CLX', LOGIN_PASS = '02072007';

/* ---------------- ESTADO (localStorage) ---------------- */
let colaboradores = JSON.parse(localStorage.getItem('colaboradores') || '[]');
let pontos = JSON.parse(localStorage.getItem('pontos') || '[]');

/* ---------------- ELEMENTOS ---------------- */
const loginScreen = document.getElementById('loginScreen');
const mainApp = document.getElementById('mainApp');
const loginBtn = document.getElementById('loginBtn');
const loginMsg = document.getElementById('loginMsg');
const logoutBtn = document.getElementById('logoutBtn');
const clockEl = document.getElementById('clock');
const statusEl = document.getElementById('status');

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

/* ---------------- LOGIN ---------------- */
loginBtn.addEventListener('click', ()=>{
  const u = document.getElementById('user').value.trim();
  const p = document.getElementById('pass').value.trim();
  if(u === LOGIN_USER && p === LOGIN_PASS){
    loginScreen.style.display = 'none';
    mainApp.classList.remove('hidden');
    persistAuth(true);
    renderAll();
  } else {
    loginMsg.textContent = 'Usuário ou senha incorretos.';
    setTimeout(()=> loginMsg.textContent = '', 3000);
  }
});
logoutBtn.addEventListener('click', ()=>{
  persistAuth(false);
  location.reload();
});
function persistAuth(val){ localStorage.setItem('autenticado', val ? '1' : '0'); }

/* auto-login se tinha sessão */
if (localStorage.getItem('autenticado') === '1') {
  loginScreen.style.display = 'none';
  mainApp.classList.remove('hidden');
  renderAll();
}

/* ---------------- RELÓGIO ---------------- */
function atualizarRelogio(){
  const now = new Date();
  clockEl.textContent = now.toLocaleTimeString('pt-BR', {hour12:false});
}
setInterval(atualizarRelogio, 1000);
atualizarRelogio();

/* ---------------- UTIL ---------------- */
function salvarLocal(){
  localStorage.setItem('colaboradores', JSON.stringify(colaboradores));
  localStorage.setItem('pontos', JSON.stringify(pontos));
}
function escapeHtml(s){ if(!s && s!==0) return ''; return String(s).replace(/[&<>"'`=\/]/g, ch=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;','/':'&#x2F;','`':'&#x60;','=':'&#x3D'}[ch])); }
function gerarIdColab(){ let max = 0; colaboradores.forEach(c=>{ if(Number(c.id)>max) max = Number(c.id); }); return max + 1; }

/* ---------------- RENDER ---------------- */
function renderColaboradores(){
  const term = (searchInput.value||'').toLowerCase().trim();
  colabBody.innerHTML = '';
  colaboradores.forEach((c, idx) => {
    if(term && !(String(c.nome).toLowerCase().includes(term) || String(c.matricula).toLowerCase().includes(term) || String(c.email||'').toLowerCase().includes(term))) return;
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${idx+1}</td>
      <td>${escapeHtml(c.id)}</td>
      <td>${escapeHtml(c.nome)}</td>
      <td>${escapeHtml(c.matricula)} <span class="small">(${escapeHtml(c.email||'')})</span></td>
      <td>${escapeHtml(c.turno||'')}</td>
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
  pontosBody.innerHTML = '';
  pontos.forEach((p, i) => {
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${i+1}</td>
      <td>${escapeHtml(p.id)}</td>
      <td>${escapeHtml(p.nome)}</td>
      <td>${escapeHtml(p.matricula)}</td>
      <td>${escapeHtml(p.email||'')}</td>
      <td>${escapeHtml(p.tipo)}</td>
      <td>${escapeHtml(p.data)}</td>
      <td>${escapeHtml(p.hora)}</td>
      <td>
        <button class="del" onclick="removerPonto(${i})">Excluir</button>
      </td>
    `;
    pontosBody.appendChild(tr);
  });
}

function renderAll(){
  renderColaboradores();
  renderPontos();
}

/* ---------------- COLABORADORES: CRUD ---------------- */
addColabBtn.addEventListener('click', ()=> openColabModal());
editColabBtn.addEventListener('click', ()=>{
  const id = prompt('Digite o ID do colaborador para editar:');
  if(id) openColabModal(id);
});
deleteColabBtn.addEventListener('click', ()=>{
  const id = prompt('Digite o ID do colaborador para excluir:');
  if(id) removerColabPrompt(id);
});

function openColabModal(id){
  if(id){
    const c = colaboradores.find(x=>String(x.id) === String(id));
    if(!c) return alert('Colaborador não encontrado.');
    colabIdInput.value = c.id;
    colabNomeInput.value = c.nome;
    colabMatInput.value = c.matricula;
    colabEmailInput.value = c.email || '';
    colabTurnoInput.value = c.turno || '';
    document.getElementById('colabTitle').textContent = 'Editar Colaborador';
  } else {
    colabIdInput.value = '';
    colabNomeInput.value = '';
    colabMatInput.value = '';
    colabEmailInput.value = '';
    colabTurnoInput.value = '';
    document.getElementById('colabTitle').textContent = 'Novo Colaborador';
  }
  colabModal.classList.remove('hidden');
}
cancelColabBtn.addEventListener('click', ()=> colabModal.classList.add('hidden'));
saveColabBtn.addEventListener('click', ()=>{
  const idVal = colabIdInput.value ? String(colabIdInput.value) : '';
  const nome = colabNomeInput.value.trim();
  const mat = colabMatInput.value.trim();
  const email = colabEmailInput.value.trim();
  const turno = colabTurnoInput.value.trim();
  if(!nome) return alert('Preencha o nome.');
  if(idVal){
    // editar
    const idx = colaboradores.findIndex(x=>String(x.id)===String(idVal));
    if(idx === -1) return alert('Colaborador não encontrado.');
    colaboradores[idx] = { id: idVal, nome, matricula: mat, email, turno };
  } else {
    // novo
    const novoId = gerarIdColab();
    colaboradores.push({ id: String(novoId), nome, matricula: mat, email, turno });
  }
  salvarLocal();
  renderAll();
  colabModal.classList.add('hidden');
});

window.editarColab = function(id){
  openColabModal(id);
};

function removerColabPrompt(id){
  if(!confirm('Confirma exclusão do colaborador ID ' + id + ' ?')) return;
  const idx = colaboradores.findIndex(x=>String(x.id) === String(id));
  if(idx === -1) return alert('Colaborador não encontrado.');
  colaboradores.splice(idx,1);
  salvarLocal();
  renderAll();
}

/* ---------------- PONTOS ---------------- */
entradaBtn.addEventListener('click', ()=> {
  const id = prompt('Digite o ID do colaborador para registrar Entrada:');
  if(id) registrarPontoPrompt(id,'Entrada');
});
saidaBtn.addEventListener('click', ()=> {
  const id = prompt('Digite o ID do colaborador para registrar Saída:');
  if(id) registrarPontoPrompt(id,'Saída');
});

function registrarPontoPrompt(colabId, tipo){
  const c = colaboradores.find(x=>String(x.id) === String(colabId));
  if(!c) return alert('Colaborador não encontrado.');
  registrarPonto(c, tipo);
}

function registrarPonto(colab, tipo){
  const now = new Date();
  const data = now.toLocaleDateString('pt-BR');
  const hora = now.toLocaleTimeString('pt-BR', {hour12:false});
  pontos.push({
    id: String(colab.id),
    nome: colab.nome,
    matricula: colab.matricula,
    email: colab.email || '',
    tipo, data, hora
  });
  salvarLocal();
  renderPontos();
  alert(`${colab.nome} registrou ${tipo} às ${hora}`);
}

window.removerPonto = function(index){
  if(!confirm('Excluir este registro?')) return;
  pontos.splice(index,1);
  salvarLocal();
  renderPontos();
};

/* limpar todos os pontos */
limparTodosBtn.addEventListener('click', ()=>{
  if(pontos.length === 0) return alert('Nenhum ponto para limpar.');
  if(confirm('Deseja apagar TODOS os pontos registrados?')){
    pontos = [];
    salvarLocal();
    renderPontos();
    alert('Todos os pontos foram removidos.');
  }
});

/* ---------------- EXPORTAR PLANILHA ---------------- */
baixarBtn.addEventListener('click', ()=>{
  if(pontos.length === 0) return alert('Nenhum ponto registrado ainda.');
  // criar cópia com cabeçalhos amigáveis
  const rows = pontos.map((p, i)=>({
    Numero: i+1,
    ID_Colaborador: p.id,
    Nome: p.nome,
    Matricula: p.matricula,
    Email: p.email,
    Tipo: p.tipo,
    Data: p.data,
    Hora: p.hora
  }));
  const ws = XLSX.utils.json_to_sheet(rows);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, 'Pontos');
  XLSX.writeFile(wb, 'registros_ponto.xlsx');
});

/* ---------------- PESQUISA ---------------- */
searchInput.addEventListener('input', renderAll);

/* ---------------- INICIALIZAR ---------------- */
renderAll();

/* Expor algumas funções para uso via onclick inline nas rows */
window.registrarPontoPrompt = registrarPontoPrompt;
window.removerColabPrompt = removerColabPrompt;
window.editarColab = window.editarColab;
</script>
</body>
</html>
