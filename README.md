<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Ponto Eletrônico - CLX</title>
<style>
  body{font-family:Arial,sans-serif;background:#f5f5f5;margin:0;padding:20px;}
  .container{max-width:1200px;margin:auto;background:white;padding:20px;border-radius:8px;box-shadow:0 0 10px rgba(0,0,0,0.1);}
  h1{text-align:center;color:#333;margin-bottom:10px;}
  h3#boasVindas{text-align:center;color:#555;margin-top:0;margin-bottom:15px;}
  .button-group{display:flex;flex-wrap:wrap;gap:10px;margin-bottom:16px;}
  button{padding:10px 15px;border:none;border-radius:4px;font-size:15px;cursor:pointer;transition:0.3s;}
  button.add{background:#4CAF50;color:white;}
  button.edit{background:#FFC107;color:white;}
  button.del{background:#F44336;color:white;}
  button.reg{background:#2196F3;color:white;}
  button.export{background:#2E7D32;color:white;}
  button.logout{background:#9C27B0;color:white;}
  table{width:100%;border-collapse:collapse;margin-top:10px;}
  th,td{border:1px solid #ddd;padding:10px;text-align:left;vertical-align:middle;}
  th{background:#f2f2f2;}
  tr:nth-child(even){background:#f9f9f9;}
  .small{font-size:13px;color:#666;}
  input,select{padding:8px;border-radius:6px;border:1px solid #ccc;font-size:14px;}
  .modal{display:none;position:fixed;z-index:2;left:0;top:0;width:100%;height:100%;background:rgba(0,0,0,0.4);align-items:center;justify-content:center;}
  .modal-content{background:white;padding:18px;border-radius:8px;width:90%;max-width:520px;box-shadow:0 8px 30px rgba(0,0,0,.12);}
  .close{float:right;font-size:22px;cursor:pointer;border:none;background:none;}
  #loginScreen{position:fixed;top:0;left:0;width:100%;height:100%;background:#222;color:white;display:flex;align-items:center;justify-content:center;flex-direction:column;gap:10px;z-index:9;}
  #loginScreen input{padding:10px;font-size:18px;border-radius:6px;border:none;text-align:center;}
  #loginScreen button{background:#4CAF50;color:white;padding:10px 20px;border:none;border-radius:6px;font-size:16px;cursor:pointer;}
</style>
</head>
<body>

<!-- Tela de login -->
<div id="loginScreen">
  <h2>🔒 Login no Sistema</h2>
  <input type="text" id="loginUser" placeholder="Usuário">
  <input type="password" id="loginPass" placeholder="Senha">
  <button id="btnLogin">Entrar</button>
  <p id="erroLogin" style="color:red;display:none;">Usuário ou senha incorretos!</p>
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

  <input id="search" placeholder="🔍 Pesquise por nome ou matrícula" style="width:100%;margin-bottom:10px;">

  <table id="colabTable">
    <thead>
      <tr><th>#</th><th>ID</th><th>Nome</th><th>Matrícula / E-mail</th><th>Cargo</th><th>Turno</th><th>Ações</th></tr>
    </thead>
    <tbody id="colabBody"></tbody>
  </table>
</div>

<!-- Modal Colaborador -->
<div id="colabModal" class="modal">
  <div class="modal-content">
    <button class="close" onclick="fecharModal('colabModal')">&times;</button>
    <h3 id="modalTitle">Adicionar Colaborador</h3>
    <form id="colabForm">
      <input type="hidden" id="colabId">
      <label>Nome</label><input id="colabNome" required><br>
      <label>E-mail</label><input id="colabEmail" type="email" required><br>
      <label>Matrícula</label><input id="colabMatricula" required><br>
      <label>Cargo</label><input id="colabCargo" required><br>
      <label>Turno</label>
      <select id="colabTurno" required>
        <option>Manhã</option><option>Tarde</option><option>Noite</option>
      </select><br><br>
      <button type="submit" class="add">Salvar</button>
      <button type="button" onclick="fecharModal('colabModal')">Cancelar</button>
    </form>
  </div>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
import { getFirestore, collection, addDoc, updateDoc, deleteDoc, doc, onSnapshot, query, orderBy } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

// 🔥 Firebase Config
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
const colabsRef = collection(db, "colaboradores");

const conteudo = document.getElementById("conteudo");
const loginScreen = document.getElementById("loginScreen");

// 👤 Login
const USER = "CLX";
const PASS = "02072007";

document.getElementById("btnLogin").addEventListener("click",()=>{
  const u=document.getElementById("loginUser").value.trim();
  const p=document.getElementById("loginPass").value.trim();
  if(u===USER && p===PASS){
    loginScreen.style.display="none";
    conteudo.style.display="block";
  }else{
    document.getElementById("erroLogin").style.display="block";
    setTimeout(()=>document.getElementById("erroLogin").style.display="none",3000);
  }
});

document.getElementById("logoutBtn").addEventListener("click",()=>{
  conteudo.style.display="none";
  loginScreen.style.display="flex";
});

// 📋 Adicionar Colaborador
document.getElementById("addColabBtn").addEventListener("click",()=>{
  abrirModal("colabModal");
  document.getElementById("modalTitle").textContent="Adicionar Colaborador";
  document.getElementById("colabForm").reset();
  document.getElementById("colabId").value="";
});

document.getElementById("colabForm").addEventListener("submit", async(e)=>{
  e.preventDefault();
  const id=document.getElementById("colabId").value;
  const nome=document.getElementById("colabNome").value;
  const email=document.getElementById("colabEmail").value;
  const matricula=document.getElementById("colabMatricula").value;
  const cargo=document.getElementById("colabCargo").value;
  const turno=document.getElementById("colabTurno").value;

  if(id){
    await updateDoc(doc(db,"colaboradores",id),{nome,email,matricula,cargo,turno});
    alert("Colaborador atualizado!");
  }else{
    await addDoc(colabsRef,{nome,email,matricula,cargo,turno});
    alert("Colaborador adicionado!");
  }
  fecharModal("colabModal");
});

// 🧾 Editar
document.getElementById("editColabBtn").addEventListener("click",()=>{
  const id=prompt("Digite o ID do colaborador para editar:");
  if(!id)return;
  const colab=colabs.find(c=>c.id===id);
  if(!colab)return alert("ID não encontrado!");
  abrirModal("colabModal");
  document.getElementById("modalTitle").textContent="Editar Colaborador";
  document.getElementById("colabId").value=colab.id;
  document.getElementById("colabNome").value=colab.nome;
  document.getElementById("colabEmail").value=colab.email;
  document.getElementById("colabMatricula").value=colab.matricula;
  document.getElementById("colabCargo").value=colab.cargo;
  document.getElementById("colabTurno").value=colab.turno;
});

// ❌ Excluir
document.getElementById("deleteColabBtn").addEventListener("click",async()=>{
  const id=prompt("Digite o ID do colaborador para excluir:");
  if(!id)return;
  if(confirm("Deseja realmente excluir?")){
    await deleteDoc(doc(db,"colaboradores",id));
    alert("Colaborador removido!");
  }
});

// 🧠 Renderizar tabela
const colabBody=document.getElementById("colabBody");
const search=document.getElementById("search");
let colabs=[];

onSnapshot(query(colabsRef,orderBy("nome")),snap=>{
  colabs=[];
  snap.forEach(docu=>colabs.push({id:docu.id,...docu.data()}));
  renderColabs();
});

function renderColabs(){
  const termo=search.value.toLowerCase();
  const lista=colabs.filter(c=>c.nome.toLowerCase().includes(termo)||c.matricula.toLowerCase().includes(termo));
  colabBody.innerHTML="";
  lista.forEach((c,i)=>{
    const tr=document.createElement("tr");
    tr.innerHTML=`
      <td>${i+1}</td>
      <td>${c.id}</td>
      <td>${c.nome}</td>
      <td>${c.matricula} <span class="small">(${c.email})</span></td>
      <td>${c.cargo}</td>
      <td>${c.turno}</td>
      <td>
        <button class="edit" onclick="editarColab('${c.id}')">✏️</button>
        <button class="del" onclick="removerColab('${c.id}')">🗑️</button>
      </td>`;
    colabBody.appendChild(tr);
  });
}

search.addEventListener("input",renderColabs);

// editar/excluir direto na tabela
window.editarColab=(id)=>{
  const colab=colabs.find(c=>c.id===id);
  if(!colab)return;
  abrirModal("colabModal");
  document.getElementById("modalTitle").textContent="Editar Colaborador";
  document.getElementById("colabId").value=colab.id;
  document.getElementById("colabNome").value=colab.nome;
  document.getElementById("colabEmail").value=colab.email;
  document.getElementById("colabMatricula").value=colab.matricula;
  document.getElementById("colabCargo").value=colab.cargo;
  document.getElementById("colabTurno").value=colab.turno;
}

window.removerColab=async(id)=>{
  if(confirm("Tem certeza que deseja excluir este colaborador?")){
    await deleteDoc(doc(db,"colaboradores",id));
    alert("Removido!");
  }
}

// modal funções
window.abrirModal=(id)=>document.getElementById(id).style.display="flex";
window.fecharModal=(id)=>document.getElementById(id).style.display="none";
</script>
</body>
</html>
