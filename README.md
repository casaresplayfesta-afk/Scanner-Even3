<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ponto Eletrônico</title>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css">
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center">

<!-- LOGIN -->
<div id="loginScreen" class="bg-white p-6 rounded-2xl shadow-lg w-80">
  <h2 class="text-2xl font-bold text-center mb-4 text-blue-600">Login</h2>
  <input id="user" type="text" placeholder="Usuário" class="border rounded w-full p-2 mb-3">
  <input id="pass" type="password" placeholder="Senha" class="border rounded w-full p-2 mb-3">
  <button id="loginBtn" class="bg-blue-600 text-white w-full py-2 rounded hover:bg-blue-700">Entrar</button>
  <label class="flex items-center mt-3 text-sm text-gray-600">
    <input type="checkbox" id="lembrarLogin" class="mr-2"> Lembrar login
  </label>
  <p class="text-center text-sm text-gray-500 mt-3">
    Usuário: <b>CLX</b> / Senha: <b>02072007</b>
  </p>
</div>

<!-- SISTEMA -->
<div id="app" class="hidden container mx-auto p-4">
  <h1 class="text-3xl font-bold text-center mb-6 text-blue-600">Ponto Eletrônico</h1>

  <div class="flex justify-between mb-4">
    <button id="novoColabBtn" class="bg-green-600 text-white px-4 py-2 rounded hover:bg-green-700">+ Novo Colaborador</button>
    <div>
      <button id="baixarBtn" class="bg-yellow-500 text-white px-4 py-2 rounded hover:bg-yellow-600 mr-2">Baixar Excel</button>
      <button id="limparTodosBtn" class="bg-red-600 text-white px-4 py-2 rounded hover:bg-red-700">Limpar Todos</button>
    </div>
  </div>

  <input id="searchInput" type="text" placeholder="Pesquisar colaborador..." class="border rounded w-full p-2 mb-4">

  <div class="overflow-x-auto">
    <table class="min-w-full bg-white rounded shadow">
      <thead>
        <tr class="bg-blue-600 text-white">
          <th class="p-2">ID</th>
          <th class="p-2">Nome</th>
          <th class="p-2">Matrícula</th>
          <th class="p-2">Email</th>
          <th class="p-2">Ações</th>
        </tr>
      </thead>
      <tbody id="colabTable"></tbody>
    </table>
  </div>

  <h2 class="text-xl font-semibold mt-8 mb-2 text-gray-700">Registros de Ponto</h2>
  <div class="overflow-x-auto">
    <table class="min-w-full bg-white rounded shadow">
      <thead>
        <tr class="bg-gray-700 text-white">
          <th class="p-2">Nome</th>
          <th class="p-2">Tipo</th>
          <th class="p-2">Data</th>
          <th class="p-2">Hora</th>
        </tr>
      </thead>
      <tbody id="pontoTable"></tbody>
    </table>
  </div>
</div>

<script type="module">
// Firebase
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.14.0/firebase-app.js";
import { getFirestore, collection, addDoc, getDocs, deleteDoc, doc, query, where } from "https://www.gstatic.com/firebasejs/10.14.0/firebase-firestore.js";

// Configuração Firebase
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

const loginScreen = document.getElementById("loginScreen");
const appDiv = document.getElementById("app");
const lembrarLogin = document.getElementById("lembrarLogin");

// Login
const savedLogin = localStorage.getItem("loginLembrado");
if (savedLogin === "true") {
  appDiv.classList.remove("hidden");
  loginScreen.classList.add("hidden");
}

document.getElementById("loginBtn").addEventListener("click", () => {
  const u = document.getElementById("user").value;
  const p = document.getElementById("pass").value;
  if (u === "CLX" && p === "02072007") {
    if (lembrarLogin.checked) localStorage.setItem("loginLembrado", "true");
    appDiv.classList.remove("hidden");
    loginScreen.classList.add("hidden");
  } else {
    alert("Usuário ou senha incorretos!");
  }
});

let colaboradores = [];
let pontos = [];

// Renderizar colaboradores
async function renderColaboradores() {
  const tbody = document.getElementById("colabTable");
  tbody.innerHTML = "";
  const querySnapshot = await getDocs(collection(db, "colaboradores"));
  colaboradores = [];
  querySnapshot.forEach((docSnap) => {
    const c = { id: docSnap.id, ...docSnap.data() };
    colaboradores.push(c);
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td class="p-2 text-center">${c.id}</td>
      <td class="p-2">${c.nome}</td>
      <td class="p-2">${c.matricula}</td>
      <td class="p-2">${c.email}</td>
      <td class="p-2 flex flex-wrap gap-2 justify-center">
        <button class="bg-blue-500 text-white px-2 py-1 rounded" onclick="baterPonto('${c.id}','Entrada')">Entrada</button>
        <button class="bg-yellow-500 text-white px-2 py-1 rounded" onclick="baterPonto('${c.id}','Saída')">Saída</button>
        <button class="bg-red-600 text-white px-2 py-1 rounded" onclick="excluirColab('${c.id}')">Excluir</button>
      </td>`;
    tbody.appendChild(tr);
  });
  renderPontos();
}

// Bater ponto
window.baterPonto = async (idColab, tipo) => {
  const colab = colaboradores.find(c => c.id === idColab);
  if (!colab) return;
  const now = new Date();
  await addDoc(collection(db, "pontos"), {
    idColab,
    nome: colab.nome,
    tipo,
    data: now.toLocaleDateString('pt-BR'),
    hora: now.toLocaleTimeString('pt-BR', { hour12: false })
  });
  renderPontos();
};

// Renderizar pontos
async function renderPontos() {
  const tbody = document.getElementById("pontoTable");
  tbody.innerHTML = "";
  const querySnapshot = await getDocs(collection(db, "pontos"));
  pontos = [];
  querySnapshot.forEach((docSnap) => {
    const p = docSnap.data();
    pontos.push(p);
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td class="p-2">${p.nome}</td>
      <td class="p-2 text-center">${p.tipo}</td>
      <td class="p-2 text-center">${p.data}</td>
      <td class="p-2 text-center">${p.hora}</td>`;
    tbody.appendChild(tr);
  });
}

// Excluir colaborador
window.excluirColab = async (id) => {
  if (!confirm("Deseja realmente excluir este colaborador e seus pontos?")) return;
  await deleteDoc(doc(db, "colaboradores", id));
  const q = query(collection(db, "pontos"), where("idColab", "==", id));
  const snapshot = await getDocs(q);
  snapshot.forEach(async (docSnap) => {
    await deleteDoc(doc(db, "pontos", docSnap.id));
  });
  renderColaboradores();
};

// Limpar todos os pontos
document.getElementById("limparTodosBtn").addEventListener("click", async () => {
  if (!confirm("Deseja realmente apagar todos os pontos?")) return;
  const snapshot = await getDocs(collection(db, "pontos"));
  snapshot.forEach(async (docSnap) => {
    await deleteDoc(doc(db, "pontos", docSnap.id));
  });
  renderPontos();
});

// Baixar Excel
document.getElementById("baixarBtn").addEventListener("click", () => {
  const wb = XLSX.utils.book_new();
  const wsColab = XLSX.utils.json_to_sheet(colaboradores);
  const wsPontos = XLSX.utils.json_to_sheet(pontos);
  XLSX.utils.book_append_sheet(wb, wsColab, "Colaboradores");
  XLSX.utils.book_append_sheet(wb, wsPontos, "Pontos");
  XLSX.writeFile(wb, "PontoEletronico.xlsx");
});

renderColaboradores();
</script>
</body>
</html>
