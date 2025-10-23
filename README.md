<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Ponto Eletrônico</title>
<style>
body {
  font-family: Arial, sans-serif;
  background: #f5f5f5;
  margin: 0;
  padding: 0;
}

/* --- Tela de Login --- */
#loginScreen {
  position: fixed;
  top: 0; left: 0; width: 100%; height: 100%;
  background: linear-gradient(135deg, #007bff, #4CAF50);
  display: flex; justify-content: center; align-items: center;
}

#loginBox {
  background: white;
  padding: 40px;
  border-radius: 12px;
  box-shadow: 0 0 15px rgba(0,0,0,0.2);
  text-align: center;
  width: 90%; max-width: 350px;
}

#loginBox h2 {
  margin-bottom: 20px;
  color: #333;
}

#loginBox input {
  width: 100%;
  padding: 10px;
  font-size: 16px;
  margin-bottom: 15px;
  border-radius: 6px;
  border: 1px solid #ccc;
  text-align: center;
}

#loginBox button {
  width: 100%;
  background: #007bff;
  color: white;
  border: none;
  padding: 10px;
  font-size: 16px;
  border-radius: 6px;
  cursor: pointer;
  transition: 0.3s;
}

#loginBox button:hover { background: #0056b3; }

.container { 
  display: none;
  max-width:1200px; 
  margin:auto; 
  background:white; 
  padding:20px; 
  border-radius:8px; 
  box-shadow:0 0 10px rgba(0,0,0,0.1);
}
h1 { text-align:center; color:#333; margin-bottom:20px;}
.button-group { display:flex; flex-wrap:wrap; gap:10px; margin-bottom:20px;}
button { padding:10px 15px; border:none; border-radius:4px; font-size:16px; cursor:pointer; transition:0.3s;}
button:hover { opacity:0.9;}
button.add { background:#4CAF50; color:white; }
button.edit { background:#FFC107; color:white;}
button.del { background:#F44336; color:white;}
button.reg { background:#2196F3; color:white;}
button.config { background:#9C27B0; color:white;}
table { width:100%; border-collapse:collapse; margin-top:20px;}
th, td { border:1px solid #ddd; padding:12px; text-align:left;}
th { background:#f2f2f2;}
tr:nth-child(even){background:#f9f9f9;}
.modal { display:none; position:fixed; z-index:1; left:0; top:0; width:100%; height:100%; background:rgba(0,0,0,0.4); overflow:auto;}
.modal-content { background:white; margin:5% auto; padding:20px; border-radius:8px; width:90%; max-width:600px;}
.close { float:right; font-size:28px; font-weight:bold; cursor:pointer;}
.close:hover { color:black;}
select,input,button { margin:5px 0; padding:10px; border-radius:4px; border:1px solid #ccc; font-size:16px; }
</style>
</head>
<body>

<!-- 🔐 Tela de Login -->
<div id="loginScreen">
  <div id="loginBox">
    <h2>🔒 Acesso Restrito</h2>
    <input type="password" id="senhaInput" placeholder="Digite a senha">
    <button id="entrarBtn">Entrar</button>
    <p id="erroMsg" style="color:red;display:none;margin-top:10px;">Senha incorreta!</p>
  </div>
</div>

<div class="container" id="appContainer">
<h1>Sistema de Ponto Eletrônico</h1>

<div class="button-group">
  <button class="add" id="addColabBtn">Adicionar Colaborador</button>
  <button class="edit" id="editColabBtn">Editar Colaborador</button>
  <button class="del" id="deleteColabBtn">Excluir Colaborador</button>
  <button class="reg" id="entradaBtn">Registrar Entrada</button>
  <button class="reg" id="saidaBtn">Registrar Saída</button>
  <button class="reg" id="exportExcelBtn">Exportar Excel</button>
  <button class="config" id="configBtn">⚙️ Configurações</button>
</div>

<!-- Tabelas -->
<h2>Colaboradores</h2>
<table id="colabTable">
<thead><tr><th>ID</th><th>Nome</th><th>Matrícula</th><th>Cargo</th><th>Turno</th><th>Ação</th></tr></thead>
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
      <select id="selectColab" style="display:none;"></select>
      <input type="text" id="colabNome" placeholder="Nome Completo" required>
      <input type="text" id="colabMatricula" placeholder="Matrícula (4-11 dígitos)" pattern="\d{4,11}" required>
      <input type="text" id="colabCargo" placeholder="Cargo" required>
      <select id="colabTurno" required>
        <option value="">Selecione o Turno</option>
        <option value="Manhã">Manhã</option>
        <option value="Tarde">Tarde</option>
        <option value="Noite">Noite</option>
        <option value="Madrugada">Madrugada</option>
      </select>
      <button type="submit" id="colabSubmit">Salvar</button>
    </form>
  </div>
</div>

<!-- Modal Registrar Ponto -->
<div id="pontoModal" class="modal">
  <div class="modal-content">
    <span class="close">&times;</span>
    <h2 id="pontoTitle">Registrar Ponto</h2>
    <form id="pontoForm">
      <select id="selectColaboradorPonto" required></select>
      <button type="submit">Registrar</button>
    </form>
  </div>
</div>

<!-- Modal Configurações -->
<div id="configModal" class="modal">
  <div class="modal-content">
    <span class="close">&times;</span>
    <h2>Alterar Senha</h2>
    <form id="senhaForm">
      <input type="password" id="senhaAtual" placeholder="Senha atual" required>
      <input type="password" id="novaSenha" placeholder="Nova senha" required>
      <input type="password" id="confirmaSenha" placeholder="Confirmar nova senha" required>
      <button type="submit">Salvar Nova Senha</button>
      <p id="senhaMsg" style="color:red;display:none;margin-top:10px;"></p>
    </form>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
<script>
async function gerarHash(texto) {
  const msgUint8 = new TextEncoder().encode(texto);
  const hashBuffer = await crypto.subtle.digest("SHA-256", msgUint8);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  return hashArray.map(b => b.toString(16).padStart(2, "0")).join("");
}

const hashPadrao = "fbe86c6922e1b4f2f06a50301a40113f5d5ea11aa7b9aaea418b9159b15e985e";
let senhaCorretaHash = localStorage.getItem("senhaSistema") || hashPadrao;

// Login
document.getElementById("entrarBtn").addEventListener("click", async ()=>{
  const senha = document.getElementById("senhaInput").value;
  const hash = await gerarHash(senha);
  if (hash === senhaCorretaHash) {
    document.getElementById("loginScreen").style.display = "none";
    document.getElementById("appContainer").style.display = "block";
  } else {
    document.getElementById("erroMsg").style.display = "block";
  }
});

// Alterar Senha
document.getElementById("configBtn").onclick = ()=> document.getElementById("configModal").style.display = "block";

document.getElementById("senhaForm").addEventListener("submit", async e=>{
  e.preventDefault();
  const senhaAtual = document.getElementById("senhaAtual").value;
  const nova = document.getElementById("novaSenha").value;
  const confirma = document.getElementById("confirmaSenha").value;
  const msg = document.getElementById("senhaMsg");
  const hashAtual = await gerarHash(senhaAtual);
  if (hashAtual !== senhaCorretaHash) {
    msg.textContent = "Senha atual incorreta!";
    msg.style.display = "block";
    return;
  }
  if (nova !== confirma) {
    msg.textContent = "As senhas não coincidem!";
    msg.style.display = "block";
    return;
  }
  const novoHash = await gerarHash(nova);
  localStorage.setItem("senhaSistema", novoHash);
  senhaCorretaHash = novoHash;
  msg.style.color = "green";
  msg.textContent = "Senha alterada com sucesso!";
  msg.style.display = "block";
  setTimeout(()=>document.getElementById("configModal").style.display="none",2000);
});

// fechar modais
document.querySelectorAll(".close").forEach(btn=>{
  btn.onclick = ()=> btn.closest(".modal").style.display="none";
});

// --- resto igual ---
let colaboradores = JSON.parse(localStorage.getItem('colaboradores')) || [];
let registros = JSON.parse(localStorage.getItem('registros')) || [];
function salvarLocal(){ localStorage.setItem('colaboradores',JSON.stringify(colaboradores)); localStorage.setItem('registros',JSON.stringify(registros)); }
function renderColaboradores(){ /* ... igual à versão anterior ... */ }
</script>
</body>
</html>
