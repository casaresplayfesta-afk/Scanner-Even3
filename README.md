<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Ponto Eletrônico</title>
<style>
  body { font-family: Arial, sans-serif; background: #eef2f5; margin: 0; }
  header { background: #007bff; color: white; padding: 10px; text-align: center; }
  .container { padding: 20px; max-width: 900px; margin: auto; }
  .card { background: white; padding: 15px; border-radius: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); margin-top: 10px; }
  button { background: #007bff; color: white; border: none; padding: 8px 15px; border-radius: 5px; cursor: pointer; }
  button:hover { background: #0056b3; }
  table { width: 100%; border-collapse: collapse; margin-top: 10px; }
  th, td { padding: 8px; border-bottom: 1px solid #ddd; text-align: left; }
  input, select { padding: 5px; border-radius: 5px; border: 1px solid #ccc; }
  #loginBox { text-align: center; margin-top: 100px; }
</style>
</head>
<body>

<header><h1>Ponto Eletrônico</h1></header>

<div id="loginBox">
  <h2>Login</h2>
  <input type="text" id="loginUser" placeholder="Usuário"><br><br>
  <input type="password" id="loginPass" placeholder="Senha"><br><br>
  <label><input type="checkbox" id="lembrarLogin"> Lembrar login</label><br><br>
  <button onclick="fazerLogin()">Entrar</button>
</div>

<div class="container" id="app" style="display:none;">
  <div class="card">
    <h2>Cadastro de Colaborador</h2>
    <input type="text" id="nomeColab" placeholder="Nome do colaborador">
    <button onclick="addColaborador()">Adicionar</button>
  </div>

  <div class="card">
    <h2>Colaboradores</h2>
    <table id="tabelaColab">
      <tr><th>Nome</th><th>Ações</th></tr>
    </table>
  </div>

  <div class="card">
    <h2>Registros de Ponto</h2>
    <table id="tabelaPonto">
      <tr><th>Colaborador</th><th>Entrada</th><th>Saída</th></tr>
    </table>
    <button onclick="limparTodosPontos()">Limpar Todos</button>
    <button onclick="exportarExcel()">Exportar Excel</button>
  </div>
</div>

<!-- Biblioteca para gerar Excel -->
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

<script type="module">
// -------------------- FIREBASE CONFIG --------------------
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.14.0/firebase-app.js";
import {
  getFirestore, collection, addDoc, getDocs,
  doc, deleteDoc, query, where
} from "https://www.gstatic.com/firebasejs/10.14.0/firebase-firestore.js";

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

// -------------------- LOGIN --------------------
function fazerLogin() {
  const user = document.getElementById('loginUser').value;
  const pass = document.getElementById('loginPass').value;
  const lembrar = document.getElementById('lembrarLogin').checked;

  if (user === "CLX" && pass === "02072007") {
    if (lembrar) localStorage.setItem("loginSalvo", "CLX");
    document.getElementById('loginBox').style.display = 'none';
    document.getElementById('app').style.display = 'block';
  } else {
    alert("Login ou senha incorretos!");
  }
}

window.onload = () => {
  const salvo = localStorage.getItem("loginSalvo");
  if (salvo === "CLX") {
    document.getElementById('loginBox').style.display = 'none';
    document.getElementById('app').style.display = 'block';
  }
};

// -------------------- VARIÁVEIS --------------------
let colaboradores = [];

// -------------------- CADASTRAR COLABORADOR --------------------
async function addColaborador() {
  const nome = document.getElementById('nomeColab').value.trim();
  if (!nome) return alert("Digite o nome do colaborador!");

  const docRef = await addDoc(collection(db, "colaboradores"), { nome });
  colaboradores.push({ id: docRef.id, nome });
  renderColab();
  document.getElementById('nomeColab').value = '';
}

// -------------------- RENDERIZAR COLABORADORES --------------------
function renderColab() {
  const tabela = document.getElementById('tabelaColab');
  tabela.innerHTML = "<tr><th>Nome</th><th>Ações</th></tr>";
  colaboradores.forEach(c => {
    const row = tabela.insertRow();
    row.insertCell(0).innerText = c.nome;
    const acoes = row.insertCell(1);
    acoes.innerHTML = `
      <button onclick="baterEntrada('${c.id}', '${c.nome}')">Entrada</button>
      <button onclick="baterSaida('${c.id}', '${c.nome}')">Saída</button>
      <button onclick="removerColabPrompt('${c.id}')">Excluir</button>
    `;
  });
}

// -------------------- BATER ENTRADA --------------------
async function baterEntrada(id, nome) {
  const hora = new Date().toLocaleString();
  await addDoc(collection(db, "pontos"), { idColab: id, nome, entrada: hora, saida: "" });
  renderPontos();
}

// -------------------- BATER SAÍDA --------------------
async function baterSaida(id, nome) {
  const hora = new Date().toLocaleString();
  await addDoc(collection(db, "pontos"), { idColab: id, nome, entrada: "", saida: hora });
  renderPontos();
}

// -------------------- LISTAR PONTOS --------------------
async function renderPontos() {
  const tabela = document.getElementById('tabelaPonto');
  tabela.innerHTML = "<tr><th>Colaborador</th><th>Entrada</th><th>Saída</th></tr>";
  const snap = await getDocs(collection(db, "pontos"));
  snap.forEach(d => {
    const p = d.data();
    const row = tabela.insertRow();
    row.insertCell(0).innerText = p.nome;
    row.insertCell(1).innerText = p.entrada || "-";
    row.insertCell(2).innerText = p.saida || "-";
  });
}

// -------------------- EXCLUIR COLABORADOR PERMANENTEMENTE --------------------
async function removerColabPrompt(id) {
  if (confirm("Deseja excluir este colaborador e seus pontos?")) {
    await deleteDoc(doc(db, "colaboradores", id));

    const q = query(collection(db, "pontos"), where("idColab", "==", id));
    const snap = await getDocs(q);
    snap.forEach(async (d) => {
      await deleteDoc(doc(db, "pontos", d.id));
    });

    colaboradores = colaboradores.filter(c => c.id !== id);
    renderColab();
    renderPontos();
  }
}

// -------------------- LIMPAR TODOS OS PONTOS --------------------
async function limparTodosPontos() {
  if (confirm("Deseja apagar todos os pontos?")) {
    const snap = await getDocs(collection(db, "pontos"));
    snap.forEach(async (d) => {
      await deleteDoc(doc(db, "pontos", d.id));
    });
    renderPontos();
  }
}

// -------------------- EXPORTAR PARA EXCEL --------------------
async function exportarExcel() {
  const snap = await getDocs(collection(db, "pontos"));
  const registros = [];
  snap.forEach(d => registros.push(d.data()));

  if (registros.length === 0) return alert("Nenhum ponto registrado.");

  const wb = XLSX.utils.book_new();
  const ws = XLSX.utils.json_to_sheet(registros);
  XLSX.utils.book_append_sheet(wb, ws, "Pontos");
  XLSX.writeFile(wb, "pontos.xlsx");
}

// -------------------- CARREGAR DADOS INICIAIS --------------------
async function carregarDados() {
  const snapColab = await getDocs(collection(db, "colaboradores"));
  colaboradores = [];
  snapColab.forEach(d => colaboradores.push({ id: d.id, ...d.data() }));
  renderColab();
  renderPontos();
}

carregarDados();

</script>
</body>
</html>
