<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Ponto Eletrônico - Firebase com Abas por Dia</title>
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
.del{background:#f44336;color:#fff}
.download{background:var(--yellow);color:#111}
.secondary{background:#e0e0e0;color:#222}
main{padding:18px;max-width:1100px;margin:18px auto}
.search{width:100%;padding:8px;border-radius:6px;border:1px solid #ccc;margin-bottom:12px}
table{width:100%;border-collapse:collapse;background:#fff;border-radius:8px;overflow:hidden;box-shadow:0 4px 18px rgba(0,0,0,0.06);margin-bottom:20px;}
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
.abas{display:flex;gap:6px;margin-bottom:12px;}
.abas button{padding:6px 10px;background:#ddd;border-radius:6px;}
.abas button.active{background:var(--blue);color:#fff;}
</style>
</head>
<body>

<!-- LOGIN -->
<div id="loginScreen" style="position:fixed;inset:0;background:var(--blue);display:flex;align-items:center;justify-content:center;z-index:9999">
  <div style="background:#fff;padding:28px;border-radius:10px;width:92%;max-width:360px;text-align:center">
    <h2 style="margin:0 0 8px">Login do Sistema</h2>
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
    <button class="secondary" id="limparTodosBtn">Limpar todos os pontos</button>
    <button class="secondary" id="logoutBtn">Sair</button>
  </div>
</header>

<main id="mainApp" class="hidden">
  <input id="search" class="search" placeholder="🔍 Pesquisar por nome, matrícula ou e-mail">
  <h3>Colaboradores</h3>
  <button class="add" id="addColabBtn">Adicionar Colaborador</button>
  <table id="colabTable">
    <thead>
      <tr>
        <th>#</th><th>ID</th><th>Nome</th><th>Matrícula / E-mail</th><th>Turno</th><th>Ações</th>
      </tr>
    </thead>
    <tbody id="colabBody"></tbody>
  </table>

  <!-- ABAS PARA DIAS DA SEMANA -->
  <div class="abas">
    <button class="active" data-dia="1">Segunda</button>
    <button data-dia="2">Terça</button>
    <button data-dia="3">Quarta</button>
    <button data-dia="4">Quinta</button>
    <button data-dia="5">Sexta</button>
    <button data-dia="6">Sábado</button>
  </div>

  <div id="tabelasDias"></div>

  <!-- RESUMO DE HORAS TRABALHADAS -->
  <h3>Resumo de Horas Trabalhadas</h3>
  <table id="horasTable">
    <thead>
      <tr>
        <th>Funcionário</th><th>Data</th><th>Horas Trabalhadas</th>
      </tr>
    </thead>
    <tbody id="horasBody"></tbody>
    <tfoot>
      <tr><td colspan="2"><b>Total Geral</b></td><td id="totalHoras">0</td></tr>
    </tfoot>
  </table>
</main>

<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
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

const loginBtn = document.getElementById('loginBtn');
const loginScreen = document.getElementById('loginScreen');
const mainApp = document.getElementById('mainApp');
const loginMsg = document.getElementById('loginMsg');
const rememberCheckbox = document.getElementById('remember');
const logoutBtn = document.getElementById('logoutBtn');
const colabBody = document.getElementById('colabBody');
const tabelasDias = document.getElementById('tabelasDias');
const horasBody = document.getElementById('horasBody');
const totalHorasCell = document.getElementById('totalHoras');

loginBtn.addEventListener('click', async () => {
  const u = document.getElementById('user').value.trim();
  const p = document.getElementById('pass').value.trim();
  if(u==='CLX' && p==='02072007'){
    loginScreen.style.display='none';
    mainApp.classList.remove('hidden');
    if(rememberCheckbox.checked) localStorage.setItem('autenticado','1');
    await carregarFirebase();
  } else {
    loginMsg.textContent='Usuário ou senha incorretos.';
    setTimeout(()=> loginMsg.textContent='',3000);
  }
});

if(localStorage.getItem('autenticado')==='1'){ 
  loginScreen.style.display='none'; 
  mainApp.classList.remove('hidden'); 
  carregarFirebase(); 
}
logoutBtn.addEventListener('click', ()=>{ 
  localStorage.removeItem('autenticado'); 
  location.reload(); 
});

setInterval(()=> document.getElementById('clock').textContent=new Date().toLocaleTimeString('pt-BR',{hour12:false}),1000);

async function carregarFirebase(){
  const colabs = await getDocs(collection(db,"colaboradores"));
  colaboradores = colabs.docs.map(doc=>({id:doc.id,...doc.data()}));
  const pts = await getDocs(collection(db,"pontos"));
  pontos = pts.docs.map(doc=>({id:doc.id,...doc.data()}));
  document.getElementById('status').textContent="Online • Firebase";
  renderAll();
}

function renderAll(){
  renderColaboradores();
  renderTabelasDias();
  calcularHoras();
}

function renderColaboradores(){
  colabBody.innerHTML='';
  colaboradores.forEach((c,i)=>{
    const tr=document.createElement('tr');
    tr.innerHTML=`
      <td>${i+1}</td><td>${c.id}</td><td>${c.nome}</td>
      <td>${c.matricula} <span class="small">(${c.email||''})</span></td>
      <td>${c.turno||''}</td>
      <td>
        <button class="add">Entrada</button>
        <button class="secondary">Saída</button>
        <button class="del">Excluir</button>
      </td>`;
    tr.querySelector('.add').onclick=()=>registrarPonto(c.id,'Entrada');
    tr.querySelector('.secondary').onclick=()=>registrarPonto(c.id,'Saída');
    tr.querySelector('.del').onclick=()=>removerColab(c.id);
    colabBody.appendChild(tr);
  });
}

function renderTabelasDias(){
  tabelasDias.innerHTML='';
  const diasSemana=['Segunda','Terça','Quarta','Quinta','Sexta','Sábado'];
  diasSemana.forEach((dia,i)=>{
    const table=document.createElement('table');
    table.id='tabelaDia'+(i+1);
    table.style.display=i===0?'table':'none';
    table.innerHTML=`
      <thead><tr>
        <th>#</th><th>ID Colab</th><th>Nome</th><th>Matrícula</th><th>E-mail</th><th>Data</th><th>Hora</th><th>Ações</th>
      </tr></thead>
      <tbody></tbody>`;
    tabelasDias.appendChild(table);
  });
  atualizarTabelasDias();
}

function atualizarTabelasDias(){
  const diasSemana=[1,2,3,4,5,6];
  diasSemana.forEach(dia=>{
    const tbody=document.querySelector('#tabelaDia'+dia+' tbody');
    tbody.innerHTML='';
    const diaPontos = pontos.filter(p=>{
      const d = new Date(p.horarioISO).getDay();
      // JS: domingo=0, segunda=1...
      return d===dia;
    });
    diaPontos.forEach((p,i)=>{
      const tr=document.createElement('tr');
      tr.innerHTML=`<td>${i+1}</td><td>${p.idColab}</td><td>${p.nome}</td><td>${p.matricula}</td><td>${p.email||''}</td><td>${p.data}</td><td>${p.hora}</td><td><button class="del">Excluir</button></td>`;
      tr.querySelector('.del').onclick=()=>excluirPonto(p.id);
      tbody.appendChild(tr);
    });
  });
}

async function registrarPonto(idColab,tipo){
  const c=colaboradores.find(x=>x.id===idColab);
  const now=new Date();
  const p={id:Date.now().toString(),idColab,nome:c.nome,matricula:c.matricula,email:c.email,tipo,data:now.toLocaleDateString('pt-BR'),hora:now.toLocaleTimeString('pt-BR',{hour12:false}), horarioISO:now.toISOString()};
  pontos.push(p); 
  atualizarTabelasDias(); 
  calcularHoras();
  await setDoc(doc(db,"pontos",p.id),p);
}

function calcularHoras(){
  horasBody.innerHTML='';
  let dados={};
  pontos.forEach(p=>{
    if(!dados[p.nome]) dados[p.nome]={};
    if(!dados[p.nome][p.data]) dados[p.nome][p.data]=[];
    dados[p.nome][p.data].push(p);
  });
  let totalGeral=0;
  Object.keys(dados).forEach(nome=>{
    Object.keys(dados[nome]).forEach(data=>{
      let reg=dados[nome][data].sort((a,b)=>new Date(a.horarioISO)-new Date(b.horarioISO));
      let entrada=null,total=0;
      reg.forEach(r=>{
        const hora=new Date(r.horarioISO);
        if(r.tipo==='Entrada') entrada=hora;
        if(r.tipo==='Saída' && entrada){
          total+=(hora-entrada)/3600000;
          entrada=null;
        }
      });
      totalGeral+=total;
      const tr=document.createElement('tr');
      tr.innerHTML=`<td>${nome}</td><td>${data}</td><td>${total.toFixed(2)} h</td>`;
      horasBody.appendChild(tr);
    });
  });
  totalHorasCell.textContent=totalGeral.toFixed(2)+" h";
}

async function excluirPonto(id){
  if(confirm("Excluir este ponto permanentemente?")){
    pontos=pontos.filter(p=>p.id!==id); 
    atualizarTabelasDias();
    calcularHoras();
    try { await deleteDoc(doc(db,"pontos",id)); } catch(err){ console.error(err); }
  }
}

async function removerColab(id){
  if(confirm("Excluir colaborador permanentemente?")){
    colaboradores=colaboradores.filter(c=>c.id!==id);
    pontos=pontos.filter(p=>p.idColab!==id);
    renderAll();
    try {
      await deleteDoc(doc(db,"colaboradores",id));
      const pts=await getDocs(collection(db,"pontos"));
      pts.docs.forEach(async p=>{ if(p.data().idColab===id) await deleteDoc(doc(db,"pontos",p.id)); });
    } catch(err){ console.error(err); }
  }
}

/* ABAS DIAS */
document.querySelectorAll('.abas button').forEach(btn=>{
  btn.addEventListener('click',()=>{
    document.querySelectorAll('.abas button').forEach(b=>b.classList.remove('active'));
    btn.classList.add('active');
    const dia=parseInt(btn.dataset.dia);
    for(let i=1;i<=6;i++){
      document.getElementById('tabelaDia'+i).style.display=i===dia?'table':'none';
    }
  });
});

/* EXPORTAR */
baixarBtn.onclick=()=>{
  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,XLSX.utils.json_to_sheet(colaboradores),'Colaboradores');
  XLSX.utils.book_append_sheet(wb,XLSX.utils.json_to_sheet(pontos),'Pontos');
  const resumo=[];
  horasBody.querySelectorAll('tr').forEach(r=>{
    const tds=r.querySelectorAll('td');
    resumo.push({Funcionário:tds[0].textContent,Data:tds[1].textContent,"Horas Trabalhadas":tds[2].textContent});
  });
  XLSX.utils.book_append_sheet(wb,XLSX.utils.json_to_sheet(resumo),'Resumo de Horas');
  XLSX.writeFile(wb,'PontoEletronico.xlsx');
};

/* LIMPAR */
limparTodosBtn.onclick=()=>{ 
  if(confirm("Limpar todos os pontos?")){ 
    pontos=[]; 
    atualizarTabelasDias(); 
    calcularHoras();
  } 
};
</script>
</body>
</html>
