<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ponto Eletrônico Firebase</title>
<style>
  body { font-family: Arial, sans-serif; background: #eef2f5; margin: 0; }
  header { background: #007bff; color: white; padding: 10px; text-align: center; }
  .container { padding: 20px; max-width: 900px; margin: auto; }
  .card { background: white; padding: 15px; border-radius: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); margin-top: 10px; }
  button { background: #007bff; color: white; border: none; padding: 8px 15px; border-radius: 5px; cursor: pointer; }
  button:hover { background: #0056b3; }
  table { width: 100%; border-collapse: collapse; margin-top: 10px; }
  th, td { padding: 8px; border-bottom: 1px solid #ddd; text-align: left; }
  input { padding: 6px; border-radius: 5px; border: 1px solid #ccc; }
  #loginBox { text-align: center; margin-top: 100px; }
</style>
</head>
<body>

<header><h1>Ponto Eletrônico</h1></header>

<!-- LOGIN -->
<div id="loginBox">
  <h2>Login</h2>
  <input type="text" id="loginUser" placeholder="Usuário"><br><br>
  <input type="password" id="loginPass" placeholder="Senha"><br><br>
  <label><input type="checkbox" id="lembrarLogin"> Lembrar login</label><br><br>
  <button id="btnEntrar">Entrar</button>
  <p style="color:#555;font-size:13px;">Usuário: <b>CLX</b> / Senha: <b>02072007</b></p>
</div>

<!-- APP -->
<div class="container" id="app" style="display:none;">
  <div class="card">
    <h2>Cadastro de Colaborador</h2>
    <input type="text" id="nomeColab" placeholder="Nome do colaborador">
    <button id="btnAddColab">Adicionar</button>
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
    <button id="btnLimpar">Limpar Todos</button>
    <button id="btnExportar">Exportar Excel</button>
  </div>
</div>

<!-- XLSX -->
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

<!-- FIREBASE -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.14.0/firebase-app.js";
import { 
  getFirestore, collection, addDoc, getDocs, 
  deleteDoc, doc, query, where 
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

let colaboradores = [];

/* LOGIN */
document.getElementById("btnEntrar").addEventListener("click", () => {
  const user = document.getElementById("loginUser").value.trim();
  const pass = document.getElementById("loginPass").value.trim();
  const lembrar = document.getElementById("lembrarLogin").checked;

  if (user === "CLX" && pass === "02072007") {
    if (lembrar) localStorage.setItem("lembrarLogin", "1");
    document.getElementById("loginBox").style.display = "none";
    document.getElementById("app").style.display = "block";
    carregarDados();
  } else {
    alert("Usuário ou senha incorretos!");
  }
});

// Se login salvo
window.addEventListener("load", () => {
  if (localStorage.getItem("lembrarLogin") === "1") {
    document.getElementById("loginBox").style.display = "none";
    document.getElementById("app").style.display = "block";
    carregarDados();
  }
});

/* FUNÇÕES PRINCIPAIS */
async function addColaborador() {
  const nome = document.getElementById("nomeColab").value.trim();
  if (!nome) return alert("Digite o nome!");
  const docRef = await addDoc(collection(db, "colaboradores"), { nome });
  colaboradores.push({ id: docRef.id, nome });
  renderColab();
  document.getElementById("nomeColab").value = "";
}

function renderColab() {
  const tabela = document.getElementById("tabelaColab");
  tabela.innerHTML = "<tr><th>Nome</th><th>Ações</th></tr>";
  colaboradores.forEach(c => {
    const tr = tabela.insertRow();
    tr.insertCell(0).textContent = c.nome;
    tr.insertCell(1).innerHTML = `
      <button onclick="baterEntrada('${c.id}','${c.nome}')">Entrada</button>
      <button onclick="baterSaida('${c.id}','${c.nome}')">Saída</button>
      <button onclick="removerColab('${c.id}')">Excluir</button>
    `;
  });
}

window.baterEntrada = async (id, nome) => {
  const hora = new Date().toLocaleString();
  await addDoc(collection(db, "pontos"), { idColab: id, nome, entrada: hora, saida: "" });
  renderPontos();
};

window.baterSaida = async (id, nome) => {
  const hora = new Date().toLocaleString();
  await addDoc(collection(db, "pontos"), { idColab: id, nome, entrada: "", saida: hora });
  renderPontos();
};

window.removerColab = async (id) => {
  if (!confirm("Excluir colaborador e pontos?")) return;
  await deleteDoc(doc(db, "colaboradores", id));
  const q = query(collection(db, "pontos"), where("idColab", "==", id));
  const snap = await getDocs(q);
  snap.forEach(async d => await deleteDoc(doc(db, "pontos", d.id)));
  colaboradores = colaboradores.filter(c => c.id !== id);
  renderColab();
  renderPontos();
};

async function renderPontos() {
  const tabela = document.getElementById("tabelaPonto");
  tabela.innerHTML = "<tr><th>Colaborador</th><th>Entrada</th><th>Saída</th></tr>";
  const snap = await getDocs(collection(db, "pontos"));
  snap.forEach(d => {
    const p = d.data();
    const tr = tabela.insertRow();
    tr.insertCell(0).textContent = p.nome;
    tr.insertCell(1).textContent = p.entrada || "-";
    tr.insertCell(2).textContent = p.saida || "-";
  });
}

async function carregarDados() {
  const snapColab = await getDocs(collection(db, "colaboradores"));
  colaboradores = [];
  snapColab.forEach(d => colaboradores.push({ id: d.id, ...d.data() }));
  renderColab();
  renderPontos();
}

/* BOTÕES */
document.getElementById("btnAddColab").addEventListener("click", addColaborador);
document.getElementById("btnLimpar").addEventListener("click", async () => {
  if (!confirm("Apagar todos os pontos?")) return;
  const snap = await getDocs(collection(db, "pontos"));
  snap.forEach(async d => await deleteDoc(doc(db, "pontos", d.id)));
  renderPontos();
});
document.getElementById("btnExportar").addEventListener("click", async () => {
  const snap = await getDocs(collection(db, "pontos"));
  const dados = [];
  snap.forEach(d => dados.push(d.data()));
  if (dados.length === 0) return alert("Sem registros!");
  const wb = XLSX.utils.book_new();
  const ws = XLSX.utils.json_to_sheet(dados);
  XLSX.utils.book_append_sheet(wb, ws, "Pontos");
  XLSX.writeFile(wb, "pontos.xlsx");
});
</script>

</body>
</html>
