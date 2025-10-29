<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Ponto Eletrônico - Firebase com Pesquisa e Cargo</title>
<style>
:root{--blue:#003366;--green:#4CAF50;--yellow:#ff9800;--red:#f44336;}
body{font-family:Arial,Helvetica,sans-serif;background:#f7f9fc;margin:0}
header{background:var(--blue);color:#fff;padding:10px 16px;display:flex;align-items:center;justify-content:space-between;gap:12px;flex-wrap:wrap}
.logo{font-weight:700}
#clock{font-weight:700}
.controls{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
button{padding:8px 12px;border:none;border-radius:6px;cursor:pointer;font-weight:600}
.add{background:var(--green);color:#fff}
.secondary{background:#e0e0e0;color:#222}
.download{background:var(--yellow);color:#111}
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
    <h2>Login do Sistema</h2>
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
    <button class="secondary" id="limparTodosBtn">Limpar Pontos</button>
    <button class="secondary" id="logoutBtn">Sair</button>
  </div>
</header>

<main id="mainApp" class="hidden">
  <input id="search" class="search" placeholder="🔍 Pesquisar colaborador por nome, cargo, matrícula ou e-mail">

  <h3>Colaboradores</h3>
  <button class="add" id="addColabBtn">Adicionar Colaborador</button>

  <table id="colabTable">
    <thead>
      <tr>
        <th>#</th><th>ID</th><th>Nome</th><th>Cargo</th><th>Matrícula / E-mail</th><th>Turno</th><th>Ações</th>
      </tr>
    </thead>
    <tbody id="colabBody"></tbody>
  </table>

  <h3 style="margin-top:18px">Entradas Registradas</h3>
  <table id="entradasTable">
    <thead><tr><th>#</th><th>ID Colab</th><th>Nome</th><th>Data</th><th>Hora</th><th>Ações</th></tr></thead>
    <tbody id="entradasBody"></tbody>
  </table>

  <h3 style="margin-top:18px">Saídas Registradas</h3>
  <table id="saidasTable">
    <thead><tr><th>#</th><th>ID Colab</th><th>Nome</th><th>Data</th><th>Hora</th><th>Ações</th></tr></thead>
    <tbody id="saidasBody"></tbody>
  </table>

  <h3 style="margin-top:18px">Resumo de Horas Trabalhadas</h3>
  <table id="horasTable">
    <thead><tr><th>Funcionário</th><th>Data</th><th>Horas Trabalhadas</th></tr></thead>
    <tbody id="horasBody"></tbody>
    <tfoot><tr><td colspan="2"><b>Total Geral</b></td><td id="totalHoras">0</td></tr></tfoot>
  </table>
</main>

<!-- MODAL DE EDIÇÃO -->
<div id="editModal" class="modal hidden">
  <div class="modal-content">
    <h3>Editar Colaborador</h3>
    <input id="editNome" placeholder="Nome" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #ccc"><br>
    <input id="editCargo" placeholder="Cargo" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #ccc"><br>
    <input id="editMatricula" placeholder="Matrícula" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #ccc"><br>
    <input id="editEmail" placeholder="E-mail" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #ccc"><br>
    <input id="editTurno" placeholder="Turno" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #ccc"><br>
    <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:10px">
      <button class="secondary" id="cancelEdit">Cancelar</button>
      <button class="add" id="saveEdit">Salvar</button>
    </div>
  </div>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-app.js";
import { getFirestore, collection, getDocs, setDoc, doc, deleteDoc } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-firestore.js";

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

let colaboradores = [];
let pontos = [];
let colabEmEdicao = null;

/* LOGIN */
const loginScreen = document.getElementById('loginScreen');
const mainApp = document.getElementById('mainApp');
document.getElementById('loginBtn').onclick = async () => {
  const u = document.getElementById('user').value.trim();
  const p = document.getElementById('pass').value.trim();
  if (u === 'CLX' && p === '02072007') {
    loginScreen.style.display = 'none';
    mainApp.classList.remove('hidden');
    if (document.getElementById('remember').checked)
      localStorage.setItem('autenticado', '1');
    await carregarFirebase();
  } else {
    document.getElementById('loginMsg').textContent = 'Usuário ou senha incorretos.';
  }
};
if (localStorage.getItem('autenticado') === '1') {
  loginScreen.style.display = 'none';
  mainApp.classList.remove('hidden');
  carregarFirebase();
}
document.getElementById('logoutBtn').onclick = () => {
  localStorage.removeItem('autenticado');
  location.reload();
};

/* RELÓGIO */
setInterval(() => {
  document.getElementById('clock').textContent = new Date().toLocaleTimeString('pt-BR', { hour12: false });
}, 1000);

/* FIREBASE */
async function carregarFirebase() {
  const colabs = await getDocs(collection(db, "colaboradores"));
  colaboradores = colabs.docs.map(doc => ({ id: doc.id, ...doc.data() }));
  const pts = await getDocs(collection(db, "pontos"));
  pontos = pts.docs.map(doc => ({ id: doc.id, ...doc.data() }));
  document.getElementById('status').textContent = "Online • Firebase";
  renderAll();
}

function renderAll() {
  renderColaboradores();
  renderEntradasSaidas();
  calcularHoras();
}

/* PESQUISA */
const searchInput = document.getElementById('search');
searchInput.addEventListener('input', () => renderColaboradores(searchInput.value.toLowerCase()));

/* RENDERIZAÇÃO */
function renderColaboradores(filtro = '') {
  const body = document.getElementById('colabBody');
  body.innerHTML = '';
  colaboradores
    .filter(c =>
      c.nome?.toLowerCase().includes(filtro) ||
      c.cargo?.toLowerCase().includes(filtro) ||
      c.matricula?.toLowerCase().includes(filtro) ||
      c.email?.toLowerCase().includes(filtro)
    )
    .forEach((c, i) => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${i + 1}</td>
        <td>${c.id}</td>
        <td>${c.nome}</td>
        <td>${c.cargo || '—'}</td>
        <td>${c.matricula || ''} <span class="small">${c.email || ''}</span></td>
        <td>${c.turno || ''}</td>
        <td>
          <button class="add">Entrada</button>
          <button class="secondary">Saída</button>
          <button class="secondary editBtn">Editar</button>
          <button class="del">Excluir</button>
        </td>`;
      tr.querySelector('.add').onclick = () => registrarPonto(c.id, 'Entrada');
      tr.querySelector('.secondary').onclick = () => registrarPonto(c.id, 'Saída');
      tr.querySelector('.editBtn').onclick = () => abrirEdicao(c);
      tr.querySelector('.del').onclick = () => removerColab(c.id);
      body.appendChild(tr);
    });
}

/* MODAL DE EDIÇÃO */
const editModal = document.getElementById('editModal');
const editNome = document.getElementById('editNome');
const editCargo = document.getElementById('editCargo');
const editMatricula = document.getElementById('editMatricula');
const editEmail = document.getElementById('editEmail');
const editTurno = document.getElementById('editTurno');

function abrirEdicao(c) {
  colabEmEdicao = c;
  editNome.value = c.nome || '';
  editCargo.value = c.cargo || '';
  editMatricula.value = c.matricula || '';
  editEmail.value = c.email || '';
  editTurno.value = c.turno || '';
  editModal.classList.remove('hidden');
}

document.getElementById('cancelEdit').onclick = () => editModal.classList.add('hidden');
document.getElementById('saveEdit').onclick = async () => {
  if (!colabEmEdicao) return;
  colabEmEdicao.nome = editNome.value.trim();
  colabEmEdicao.cargo = editCargo.value.trim();
  colabEmEdicao.matricula = editMatricula.value.trim();
  colabEmEdicao.email = editEmail.value.trim();
  colabEmEdicao.turno = editTurno.value.trim();
  await setDoc(doc(db, "colaboradores", colabEmEdicao.id), colabEmEdicao);
  renderColaboradores();
  editModal.classList.add('hidden');
};

/* RESTANTE DO CÓDIGO IGUAL (entradas, saídas, horas, exclusões etc.) */
async function registrarPonto(idColab, tipo) {
  const c = colaboradores.find(x => x.id === idColab);
  if (!c) return alert("Colaborador não encontrado!");
  const now = new Date();
  const p = {
    id: Date.now().toString(),
    idColab,
    nome: c.nome,
    matricula: c.matricula,
    email: c.email,
    tipo,
    data: now.toLocaleDateString('pt-BR'),
    hora: now.toLocaleTimeString('pt-BR', { hour12: false }),
    horarioISO: now.toISOString()
  };
  pontos.push(p);
  renderEntradasSaidas();
  await setDoc(doc(db, "pontos", p.id), p);
}

function renderEntradasSaidas() {
  const entBody = document.getElementById('entradasBody');
  const saiBody = document.getElementById('saidasBody');
  entBody.innerHTML = '';
  saiBody.innerHTML = '';
  pontos.filter(p => p.tipo === 'Entrada').forEach((p, i) => {
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${i + 1}</td><td>${p.idColab}</td><td>${p.nome}</td><td>${p.data}</td><td>${p.hora}</td><td><button class="del">Excluir</button></td>`;
    tr.querySelector('.del').onclick = () => excluirPonto(p.id);
    entBody.appendChild(tr);
  });
  pontos.filter(p => p.tipo === 'Saída').forEach((p, i) => {
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${i + 1}</td><td>${p.idColab}</td><td>${p.nome}</td><td>${p.data}</td><td>${p.hora}</td><td><button class="del">Excluir</button></td>`;
    tr.querySelector('.del').onclick = () => excluirPonto(p.id);
    saiBody.appendChild(tr);
  });
  calcularHoras();
}

async function excluirPonto(id) {
  if (confirm("Excluir este ponto permanentemente?")) {
    pontos = pontos.filter(p => p.id !== id);
    renderEntradasSaidas();
    await deleteDoc(doc(db, "pontos", id));
  }
}

async function removerColab(id) {
  if (confirm("Excluir colaborador permanentemente?")) {
    colaboradores = colaboradores.filter(c => c.id !== id);
    pontos = pontos.filter(p => p.idColab !== id);
    renderAll();
    await deleteDoc(doc(db, "colaboradores", id));
  }
}

document.getElementById('limparTodosBtn').onclick = async () => {
  if (confirm("Deseja realmente excluir todos os pontos?")) {
    pontos = [];
    renderEntradasSaidas();
    const col = await getDocs(collection(db, "pontos"));
    for (let docSnap of col.docs) {
      await deleteDoc(doc(db, "pontos", docSnap.id));
    }
  }
};

function calcularHoras() {
  const horasBody = document.getElementById('horasBody');
  const totalHorasCell = document.getElementById('totalHoras');
  horasBody.innerHTML = '';
  let dados = {}, totalGeral = 0;
  pontos.forEach(p => {
    if (!dados[p.nome]) dados[p.nome] = {};
    if (!dados[p.nome][p.data]) dados[p.nome][p.data] = [];
    dados[p.nome][p.data].push(p);
  });
  Object.keys(dados).forEach(nome => {
    Object.keys(dados[nome]).forEach(data => {
      let reg = dados[nome][data].sort((a, b) => new Date(a.horarioISO) - new Date(b.horarioISO));
      let entrada = null, total = 0;
      reg.forEach(r => {
        const hora = new Date(r.horarioISO);
        if (r.tipo === 'Entrada') entrada = hora;
        if (r.tipo === 'Saída' && entrada) {
          total += (hora - entrada) / 3600000;
          entrada = null;
        }
      });
      totalGeral += total;
      const tr = document.createElement('tr');
      tr.innerHTML = `<td>${nome}</td><td>${data}</td><td>${total.toFixed(2)} h</td>`;
      horasBody.appendChild(tr);
    });
  });
  totalHorasCell.textContent = totalGeral.toFixed(2) + ' h';
}
</script>
</body>
</html>
