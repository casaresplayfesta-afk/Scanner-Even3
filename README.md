<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>Ponto Eletrônico - CLX</title>
<style>
:root{--blue:#003366;--green:#4CAF50;--yellow:#ff9800;--red:#f44336;}
body{font-family:Arial,Helvetica,sans-serif;background:#f7f9fc;margin:0;}
header{background:var(--blue);color:#fff;padding:10px 16px;display:flex;justify-content:space-between;align-items:center;}
button{padding:6px 12px;border:none;border-radius:6px;cursor:pointer;font-weight:600;margin-right:6px;}
.add{background:var(--green);color:#fff;}
.edit{background:#2196F3;color:#fff;}
.del{background:var(--red);color:#fff;}
.secondary{background:#e0e0e0;color:#222;}
table{width:100%;border-collapse:collapse;margin-top:12px;}
th,td{padding:8px;border:1px solid #ccc;text-align:left;}
th{background:#fafafa;}
.modal{position:fixed;inset:0;background:rgba(0,0,0,0.5);display:flex;justify-content:center;align-items:center;z-index:999;}
.modal-content{background:#fff;padding:20px;border-radius:10px;width:90%;max-width:360px;}
.hidden{display:none;}
</style>
</head>
<body>

<!-- LOGIN -->
<div id="loginScreen" class="modal">
  <div class="modal-content">
    <h2>Login CLX</h2>
    <input id="user" placeholder="Usuário" style="width:100%;padding:8px;margin:6px 0;"><br>
    <input id="pass" type="password" placeholder="Senha" style="width:100%;padding:8px;margin:6px 0;"><br>
    <button id="loginBtn" class="add" style="width:100%;">Entrar</button>
    <p id="loginMsg" style="color:red;margin-top:8px;"></p>
  </div>
</div>

<header>
  <div>Ponto Eletrônico CLX</div>
  <div>
    <span id="clock">--:--:--</span>
    <button class="secondary" id="logoutBtn">Sair</button>
  </div>
</header>

<main id="mainApp" class="hidden" style="padding:16px;">
  <button class="add" id="entradaBtn">Registrar Entrada</button>
  <button class="edit" id="saidaBtn">Registrar Saída</button>

  <h3>Registros</h3>
  <table>
    <thead>
      <tr>
        <th>#</th>
        <th>Data</th>
        <th>Hora</th>
        <th>Tipo</th>
      </tr>
    </thead>
    <tbody id="registrosBody"></tbody>
  </table>
</main>

<script type="module">
// Firebase Config
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-app.js";
import { getFirestore, collection, addDoc, getDocs } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-firestore.js";

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

let registros = [];
const registrosBody = document.getElementById('registrosBody');

// ---------------- LOGIN ----------------
const loginBtn = document.getElementById('loginBtn');
const loginScreen = document.getElementById('loginScreen');
const mainApp = document.getElementById('mainApp');
const loginMsg = document.getElementById('loginMsg');
const logoutBtn = document.getElementById('logoutBtn');

loginBtn.addEventListener('click', async ()=>{
  const user = document.getElementById('user').value.trim();
  const pass = document.getElementById('pass').value.trim();
  if(user==='CLX' && pass==='02072007'){
    loginScreen.classList.add('hidden');
    mainApp.classList.remove('hidden');
    await carregarRegistrosFirebase();
  } else {
    loginMsg.textContent = 'Usuário ou senha incorretos';
    setTimeout(()=>loginMsg.textContent='',3000);
  }
});

logoutBtn.addEventListener('click', ()=> location.reload());

// ---------------- RELÓGIO ----------------
const clockEl = document.getElementById('clock');
setInterval(()=>clockEl.textContent = new Date().toLocaleTimeString('pt-BR',{hour12:false}),1000);

// ---------------- FUNÇÕES ----------------
function renderRegistros(){
  registrosBody.innerHTML='';
  registros.forEach((r,i)=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${i+1}</td><td>${r.data}</td><td>${r.hora}</td><td>${r.tipo}</td>`;
    registrosBody.appendChild(tr);
  });
}

async function registrar(tipo){
  const now = new Date();
  const data = now.toLocaleDateString('pt-BR');
  const hora = now.toLocaleTimeString('pt-BR',{hour12:false});
  const registro = {data,hora,tipo};
  registros.push(registro);
  renderRegistros();
  // Salvar no Firebase
  try{ await addDoc(collection(db,'registros'), registro); } catch(e){ console.error(e); }
}

// ---------------- BOTÕES ----------------
document.getElementById('entradaBtn').addEventListener('click', ()=> registrar('Entrada'));
document.getElementById('saidaBtn').addEventListener('click', ()=> registrar('Saída'));

// ---------------- CARREGAR FIREBASE ----------------
async function carregarRegistrosFirebase(){
  registros = [];
  const snapshot = await getDocs(collection(db,'registros'));
  snapshot.forEach(doc=>{
    registros.push(doc.data());
  });
  renderRegistros();
}
</script>
</body>
</html>
