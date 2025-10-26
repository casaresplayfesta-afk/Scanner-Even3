<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Ponto Eletrônico</title>
<style>
body { font-family: Arial,sans-serif; background:#f5f5f5; margin:0; padding:20px; }
.container { max-width:1200px; margin:auto; background:white; padding:20px; border-radius:8px; box-shadow:0 0 10px rgba(0,0,0,0.1);}
h1 { text-align:center; color:#333; margin-bottom:10px;}
h3#boasVindas { text-align:center; color:#555; margin-top:0; margin-bottom:20px; }
.button-group { display:flex; flex-wrap:wrap; gap:10px; margin-bottom:20px; justify-content:center; }
button { padding:10px 15px; border:none; border-radius:4px; font-size:16px; cursor:pointer; transition:0.3s;}
button:hover { opacity:0.9;}
button.add { background:#4CAF50; color:white; }
button.edit { background:#FFC107; color:white;}
button.del { background:#F44336; color:white;}
button.reg { background:#2196F3; color:white;}
button.logout { background:#9C27B0; color:white;}
table { width:100%; border-collapse:collapse; margin-top:20px;}
th, td { border:1px solid #ddd; padding:12px; text-align:left;}
th { background:#f2f2f2;}
tr:nth-child(even){background:#f9f9f9;}
.modal { display:none; position:fixed; z-index:1; left:0; top:0; width:100%; height:100%; background:rgba(0,0,0,0.4); overflow:auto;}
.modal-content { background:white; margin:5% auto; padding:20px; border-radius:8px; width:90%; max-width:600px;}
.close { float:right; font-size:28px; font-weight:bold; cursor:pointer;}
.close:hover { color:black;}
select,input,button { width:100%; margin:5px 0; padding:10px; border-radius:4px; border:1px solid #ccc; font-size:16px; }
</style>
</head>
<body>

<div class="container">
<h1>Sistema de Ponto Eletrônico</h1>
<h3 id="boasVindas">Bem-vindo!</h3>

<div class="button-group">
  <button class="add" id="addColabBtn">Adicionar Colaborador</button>
  <button class="edit" id="editColabBtn">Editar Colaborador</button>
  <button class="reg" id="entradaBtn">Registrar Entrada</button>
  <button class="reg" id="saidaBtn">Registrar Saída</button>
  <button class="reg" id="exportExcelBtn">Exportar Excel</button>
</div>

<h2>Colaboradores</h2>
<table id="colabTable">
<thead><tr><th>ID</th><th>Nome</th><th>Matrícula</th><th>Cargo</th><th>Turno</th></tr></thead>
<tbody id="colabBody"></tbody>
</table>

<h2>Registros de Entrada</h2>
<table id="entradaTable">
<thead><tr><th>ID</th><th>Nome</th><th>Matrícula</th><th>Data/Hora</th></tr></thead>
<tbody id="entradaBody"></tbody>
</table>

<h2>Registros de Saída</h2>
<table id="saidaTable">
<thead><tr><th>ID</th><th>Nome</th><th>Matrícula</th><th>Data/Hora</th></tr></thead>
<tbody id="saidaBody"></tbody>
</table>
</div>

<!-- Modal Colaborador -->
<div id="colabModal" class="modal">
  <div class="modal-content">
    <span class="close">&times;</span>
    <h2 id="modalTitle">Adicionar Colaborador</h2>
    <form id="colabForm">
      <input type="hidden" id="colabId">
      <input type="text" id="colabNome" placeholder="Nome Completo" required>
      <input type="text" id="colabMatricula" placeholder="Matrícula" required>
      <input type="text" id="colabCargo" placeholder="Cargo" required>
      <select id="colabTurno" required>
        <option value="">Selecione o Turno</option>
        <option value="Manhã">Manhã</option>
        <option value="Tarde">Tarde</option>
        <option value="Noite">Noite</option>
      </select>
      <button type="submit">Salvar</button>
    </form>
  </div>
</div>

<!-- Modal Registrar Ponto -->
<div id="pontoModal" class="modal">
  <div class="modal-content">
    <span class="close">&times;</span>
    <h2 id="pontoTitle">Registrar Ponto</h2>
    <input type="text" id="pesquisarNome" placeholder="Pesquisar colaborador...">
    <select id="selectColaboradorPonto" required></select>
    <button id="registrarPontoBtn">Registrar</button>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
<script type="module">
// Firebase
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.13.0/firebase-app.js";
import { getFirestore, collection, addDoc, getDocs, updateDoc, doc, query, orderBy } from "https://www.gstatic.com/firebasejs/10.13.0/firebase-firestore.js";

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

// DOM
const colabModal = document.getElementById('colabModal');
const pontoModal = document.getElementById('pontoModal');
const colabBody = document.getElementById('colabBody');
const entradaBody = document.getElementById('entradaBody');
const saidaBody = document.getElementById('saidaBody');
const selectColaboradorPonto = document.getElementById('selectColaboradorPonto');
const pesquisarNome = document.getElementById('pesquisarNome');
let tipoPonto = 'Entrada';
let colaboradores = [];

// Fechar modal
document.querySelectorAll('.close').forEach(c => c.onclick = () => c.closest('.modal').style.display = 'none');

// Mostrar colaboradores
async function carregarColaboradores() {
  const q = query(collection(db, "colaboradores"), orderBy("nome", "asc"));
  const snapshot = await getDocs(q);
  colaboradores = [];
  colabBody.innerHTML = '';
  let contador = 1;

  snapshot.forEach(docSnap => {
    const c = docSnap.data();
    c.id = contador++;
    colaboradores.push(c);
    colabBody.innerHTML += `
      <tr><td>${c.id}</td><td>${c.nome}</td><td>${c.matricula}</td><td>${c.cargo}</td><td>${c.turno}</td></tr>`;
  });

  atualizarSelectColab();
}

// Atualizar select do ponto
function atualizarSelectColab() {
  selectColaboradorPonto.innerHTML = '';
  colaboradores.forEach(c => {
    selectColaboradorPonto.innerHTML += `<option value="${c.nome}">${c.nome} (${c.matricula})</option>`;
  });
}

// Pesquisar colaborador
pesquisarNome.addEventListener('input', () => {
  const termo = pesquisarNome.value.toLowerCase();
  selectColaboradorPonto.innerHTML = '';
  colaboradores
    .filter(c => c.nome.toLowerCase().includes(termo))
    .forEach(c => {
      selectColaboradorPonto.innerHTML += `<option value="${c.nome}">${c.nome} (${c.matricula})</option>`;
    });
});

// Adicionar colaborador
document.getElementById('addColabBtn').onclick = () => {
  document.getElementById('modalTitle').textContent = "Adicionar Colaborador";
  colabModal.style.display = 'block';
  document.getElementById('colabForm').reset();
};

// Registrar ponto
document.getElementById('entradaBtn').onclick = () => { tipoPonto = 'Entrada'; pontoModal.style.display = 'block'; };
document.getElementById('saidaBtn').onclick = () => { tipoPonto = 'Saída'; pontoModal.style.display = 'block'; };

document.getElementById('colabForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  const nome = document.getElementById('colabNome').value.trim();
  const matricula = document.getElementById('colabMatricula').value.trim();
  const cargo = document.getElementById('colabCargo').value.trim();
  const turno = document.getElementById('colabTurno').value;

  await addDoc(collection(db, "colaboradores"), { nome, matricula, cargo, turno });
  alert("✅ Colaborador adicionado com sucesso!");
  colabModal.style.display = 'none';
  carregarColaboradores();
});

// Registrar entrada/saída
document.getElementById('registrarPontoBtn').addEventListener('click', async () => {
  const nome = selectColaboradorPonto.value;
  const colaborador = colaboradores.find(c => c.nome === nome);
  if (!colaborador) return alert("Selecione um colaborador válido!");

  const dataHora = new Date().toISOString();
  await addDoc(collection(db, tipoPonto === 'Entrada' ? 'entradas' : 'saidas'), {
    nome: colaborador.nome,
    matricula: colaborador.matricula,
    dataHora
  });
  alert(`✅ ${tipoPonto} registrada para ${colaborador.nome}!`);
  pontoModal.style.display = 'none';
  carregarRegistros();
});

// Carregar registros
async function carregarRegistros() {
  entradaBody.innerHTML = '';
  saidaBody.innerHTML = '';

  const entradasSnap = await getDocs(collection(db, 'entradas'));
  const saidasSnap = await getDocs(collection(db, 'saidas'));

  let id = 1;
  entradasSnap.forEach(docSnap => {
    const e = docSnap.data();
    entradaBody.innerHTML += `<tr><td>${id++}</td><td>${e.nome}</td><td>${e.matricula}</td><td>${new Date(e.dataHora).toLocaleString()}</td></tr>`;
  });

  id = 1;
  saidasSnap.forEach(docSnap => {
    const s = docSnap.data();
    saidaBody.innerHTML += `<tr><td>${id++}</td><td>${s.nome}</td><td>${s.matricula}</td><td>${new Date(s.dataHora).toLocaleString()}</td></tr>`;
  });
}

// Exportar Excel
document.getElementById('exportExcelBtn').onclick = async () => {
  const entradasSnap = await getDocs(collection(db, 'entradas'));
  const saidasSnap = await getDocs(collection(db, 'saidas'));

  const entradaData = entradasSnap.docs.map(d => d.data());
  const saidaData = saidasSnap.docs.map(d => d.data());

  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, XLSX.utils.json_to_sheet(entradaData), 'Entradas');
  XLSX.utils.book_append_sheet(wb, XLSX.utils.json_to_sheet(saidaData), 'Saídas');
  XLSX.writeFile(wb, 'registros_ponto.xlsx');
};

carregarColaboradores();
carregarRegistros();
</script>
</body>
</html>
