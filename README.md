<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Painel de Ponto</title>
<style>
  body{font-family:Arial,sans-serif;background:#f0f0f0;margin:0;padding:20px;}
  .login{max-width:400px;margin:100px auto;background:white;padding:20px;border-radius:10px;box-shadow:0 0 10px rgba(0,0,0,.1);}
  .login input,button{width:100%;padding:10px;margin:8px 0;border-radius:6px;border:1px solid #ccc;}
  button{background:#007bff;color:white;border:none;cursor:pointer;}
  button:hover{background:#0056b3;}
  .painel{display:none;}
  table{width:100%;border-collapse:collapse;background:white;}
  th,td{padding:10px;border:1px solid #ddd;text-align:left;}
  th{background:#007bff;color:white;}
  .btn-entrada{background:#28a745;color:white;border:none;padding:5px 10px;border-radius:5px;}
  .btn-saida{background:#dc3545;color:white;border:none;padding:5px 10px;border-radius:5px;}
</style>
</head>
<body>
  <div class="login" id="loginBox">
    <h2>Acesso Restrito</h2>
    <input type="password" id="senha" placeholder="Digite a senha" />
    <button id="entrar">Entrar</button>
    <p id="msg"></p>
  </div>

  <div class="painel" id="painel">
    <h1>Painel de Ponto</h1>
    <table id="tabelaMonitores">
      <thead>
        <tr><th>Nome</th><th>Matrícula</th><th>Cargo</th><th>Turno</th><th>Ações</th></tr>
      </thead>
      <tbody></tbody>
    </table>

    <h2>Últimos Registros</h2>
    <table id="tabelaRegistros">
      <thead>
        <tr><th>Nome</th><th>Tipo</th><th>Data/Hora</th></tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
import { getFirestore, collection, getDocs, addDoc, onSnapshot, orderBy, query, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

// Firebase Config
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

// Login simples
const senhaCorreta = "02072007";
document.getElementById("entrar").addEventListener("click", () => {
  const senha = document.getElementById("senha").value;
  const msg = document.getElementById("msg");
  if (senha === senhaCorreta) {
    document.getElementById("loginBox").style.display = "none";
    document.getElementById("painel").style.display = "block";
    carregarMonitores();
    ouvirRegistros();
  } else {
    msg.textContent = "Senha incorreta ❌";
    msg.style.color = "red";
  }
});

// Carregar monitores
async function carregarMonitores() {
  const tbody = document.querySelector("#tabelaMonitores tbody");
  tbody.innerHTML = "";
  const snap = await getDocs(collection(db, "monitores"));
  snap.forEach((doc) => {
    const m = doc.data();
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td>${m.nome}</td>
      <td>${m.matricula}</td>
      <td>${m.cargo}</td>
      <td>${m.turno}</td>
      <td>
        <button class="btn-entrada" onclick="registrar('${doc.id}','Entrada','${m.nome}')">Entrada</button>
        <button class="btn-saida" onclick="registrar('${doc.id}','Saída','${m.nome}')">Saída</button>
      </td>
    `;
    tbody.appendChild(tr);
  });
}

// Registrar ponto
window.registrar = async (id, tipo, nome) => {
  await addDoc(collection(db, "registros"), {
    monitorId: id,
    nome,
    tipo,
    dataHora: new Date().toLocaleString(),
    criadoEm: serverTimestamp()
  });
  alert(`Ponto de ${tipo} registrado para ${nome}! ✅`);
};

// Ouvir registros em tempo real
function ouvirRegistros() {
  const q = query(collection(db, "registros"), orderBy("criadoEm", "desc"));
  const tbody = document.querySelector("#tabelaRegistros tbody");
  onSnapshot(q, (snap) => {
    tbody.innerHTML = "";
    snap.forEach((doc) => {
      const r = doc.data();
      const tr = document.createElement("tr");
      tr.innerHTML = `<td>${r.nome}</td><td>${r.tipo}</td><td>${r.dataHora}</td>`;
      tbody.appendChild(tr);
    });
  });
}
</script>
</body>
</html>
