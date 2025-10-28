<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Ponto Eletrônico Firebase - Entrada e Saída por Dia</title>
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
<style>
:root{--blue:#003366;--green:#4CAF50;--yellow:#ff9800;--red:#f44336;}
body{font-family:Arial,Helvetica,sans-serif;background:#f7f9fc;margin:0}
header{background:var(--blue);color:#fff;padding:10px 16px;display:flex;align-items:center;justify-content:space-between;gap:12px;flex-wrap:wrap}
.logo{font-weight:700}
#clock{font-weight:700}
main{padding:18px;max-width:1100px;margin:18px auto}
table{width:100%;border-collapse:collapse;background:#fff;border-radius:8px;overflow:hidden;box-shadow:0 4px 18px rgba(0,0,0,0.06);margin-bottom:18px;}
th,td{padding:10px;border-bottom:1px solid #eee;text-align:left;font-size:14px}
th{background:#fafafa;font-weight:700}
tr:hover td{background:#fbfbfb}
.add{background:#4CAF50;color:#fff;padding:6px 12px;border:none;border-radius:5px;cursor:pointer}
.edit{background:#2196F3;color:#fff;padding:4px 8px;border:none;border-radius:5px;cursor:pointer}
.del{background:#f44336;color:#fff;padding:4px 8px;border:none;border-radius:5px;cursor:pointer}
h4{margin-bottom:6px;margin-top:12px;}
.flex-row{display:flex;gap:8px;align-items:center}
</style>
</head>
<body>

<header>
  <div class="logo">Ponto Eletrônico</div>
  <div id="clock">--:--:--</div>
</header>

<main>
  <h3>Colaboradores</h3>
  <table id="colabTable">
    <thead>
      <tr><th>#</th><th>Nome</th><th>Matrícula</th><th>Turno/Cargo</th></tr>
    </thead>
    <tbody id="colabBody"></tbody>
  </table>

  <h3>Registrar Ponto</h3>
  <select id="colabSelect"></select>
  <select id="tipoSelect">
    <option value="Entrada">Entrada</option>
    <option value="Saída">Saída</option>
  </select>
  <select id="diaSelect">
    <option value="Segunda">Segunda</option>
    <option value="Terça">Terça</option>
    <option value="Quarta">Quarta</option>
    <option value="Quinta">Quinta</option>
    <option value="Sexta">Sexta</option>
    <option value="Sábado">Sábado</option>
  </select>
  <input type="time" id="horaInput">
  <button id="registrarBtn" class="add">Registrar Ponto</button>
  <button id="exportBtn" class="add">Exportar para Excel</button>

  <div id="tabelasDias"></div>
</main>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-app.js";
import { getFirestore, collection, getDocs, setDoc, doc, deleteDoc } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-firestore.js";

// Config Firebase
const firebaseConfig = {
  apiKey: "COLOQUE_SUA_APIKEY_AQUI",
  authDomain: "SEU_PROJETO.firebaseapp.com",
  projectId: "SEU_PROJETO",
  storageBucket: "SEU_PROJETO.appspot.com",
  messagingSenderId: "SEU_ID",
  appId: "SEU_APPID"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

// Dados colaboradores (você pode criar dinamicamente no Firebase)
const colaboradores=[
  {id:'1',nome:'Carlos',matricula:'001',turno:'Manhã',cargo:'Atendente'},
  {id:'2',nome:'Ana',matricula:'002',turno:'Tarde',cargo:'Caixa'}
];
let pontos=[]; 

const colabBody=document.getElementById('colabBody');
const colabSelect=document.getElementById('colabSelect');
const diaSelect=document.getElementById('diaSelect');
const tipoSelect=document.getElementById('tipoSelect');
const horaInput=document.getElementById('horaInput');
const tabelasDiasDiv=document.getElementById('tabelasDias');

function renderColaboradores(){
  colabBody.innerHTML='';
  colabSelect.innerHTML='';
  colaboradores.forEach((c,i)=>{
    colabBody.innerHTML+=`<tr><td>${i+1}</td><td>${c.nome}</td><td>${c.matricula}</td><td>${c.turno}/${c.cargo}</td></tr>`;
    colabSelect.innerHTML+=`<option value="${c.id}">${c.nome}</option>`;
  });
}

// Carregar pontos do Firebase
async function carregarPontos(){
  const pts = await getDocs(collection(db,"pontos"));
  pontos = pts.docs.map(d=>({id:d.id,...d.data()}));
  atualizarTabelasDias();
}

// Salvar ponto no Firebase
async function salvarPontoFirebase(ponto){
  await setDoc(doc(db,"pontos",ponto.id),ponto);
}

// Excluir ponto Firebase
async function excluirPontoFirebase(id){
  await deleteDoc(doc(db,"pontos",id));
}

function atualizarTabelasDias(){
  tabelasDiasDiv.innerHTML='';
  const diasSemana=['Segunda','Terça','Quarta','Quinta','Sexta','Sábado'];
  diasSemana.forEach(dia=>{
    const divDia=document.createElement('div');
    divDia.innerHTML=`<h4>${dia}</h4>`;
    
    // Tabela Entrada
    const tableEntrada=document.createElement('table');
    tableEntrada.innerHTML=`<thead><tr><th>#</th><th>Nome</th><th>Hora</th><th>Ações</th></tr></thead><tbody></tbody>`;
    const tbodyEntrada=tableEntrada.querySelector('tbody');
    pontos.filter(p=>p.dia===dia && p.tipo==='Entrada').forEach((p,i)=>{
      const tr=document.createElement('tr');
      tr.innerHTML=`<td>${i+1}</td><td>${p.nome}</td><td>${p.hora}</td><td><button class="edit">Editar</button> <button class="del">Excluir</button></td>`;
      tr.querySelector('.edit').onclick=()=>editarPonto(p);
      tr.querySelector('.del').onclick=async ()=>{
        pontos=pontos.filter(pt=>pt.id!==p.id);
        await excluirPontoFirebase(p.id);
        atualizarTabelasDias();
      };
      tbodyEntrada.appendChild(tr);
    });
    divDia.appendChild(document.createTextNode('Entrada'));
    divDia.appendChild(tableEntrada);

    // Tabela Saída
    const tableSaida=document.createElement('table');
    tableSaida.innerHTML=`<thead><tr><th>#</th><th>Nome</th><th>Hora</th><th>Ações</th></tr></thead><tbody></tbody>`;
    const tbodySaida=tableSaida.querySelector('tbody');
    pontos.filter(p=>p.dia===dia && p.tipo==='Saída').forEach((p,i)=>{
      const tr=document.createElement('tr');
      tr.innerHTML=`<td>${i+1}</td><td>${p.nome}</td><td>${p.hora}</td><td><button class="edit">Editar</button> <button class="del">Excluir</button></td>`;
      tr.querySelector('.edit').onclick=()=>editarPonto(p);
      tr.querySelector('.del').onclick=async ()=>{
        pontos=pontos.filter(pt=>pt.id!==p.id);
        await excluirPontoFirebase(p.id);
        atualizarTabelasDias();
      };
      tbodySaida.appendChild(tr);
    });
    divDia.appendChild(document.createTextNode('Saída'));
    divDia.appendChild(tableSaida);

    tabelasDiasDiv.appendChild(divDia);
  });
}

async function registrarPonto(){
  const colabId=colabSelect.value;
  const tipo=tipoSelect.value;
  const dia=diaSelect.value;
  const hora=horaInput.value || new Date().toLocaleTimeString('pt-BR',{hour12:false});
  const colaborador=colaboradores.find(c=>c.id===colabId);
  const ponto={id:Date.now().toString(),nome:colaborador.nome,tipo,dia,hora};
  pontos.push(ponto);
  await salvarPontoFirebase(ponto);
  atualizarTabelasDias();
}

function editarPonto(ponto){
  const novaHora=prompt("Digite nova hora (HH:MM)",ponto.hora);
  if(novaHora){
    ponto.hora=novaHora;
    salvarPontoFirebase(ponto);
    atualizarTabelasDias();
  }
}

// EXPORTAR PARA EXCEL
function exportarExcel(){
  const wb=XLSX.utils.book_new();
  const diasSemana=['Segunda','Terça','Quarta','Quinta','Sexta','Sábado'];
  diasSemana.forEach(dia=>{
    ['Entrada','Saída'].forEach(tipo=>{
      const dados=pontos.filter(p=>p.dia===dia && p.tipo===tipo).map(p=>({Nome:p.nome,Hora:p.hora}));
      XLSX.utils.book_append_sheet(wb,XLSX.utils.json_to_sheet(dados),`${dia} ${tipo}`);
    });
  });
  XLSX.writeFile(wb,'PontoEletronico.xlsx');
}

document.getElementById('registrarBtn').onclick=registrarPonto;
document.getElementById('exportBtn').onclick=exportarExcel;

renderColaboradores();
carregarPontos();
setInterval(()=>document.getElementById('clock').textContent=new Date().toLocaleTimeString('pt-BR',{hour12:false}),1000);
</script>
</body>
</html>
