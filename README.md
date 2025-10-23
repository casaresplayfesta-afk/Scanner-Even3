<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Ponto Eletrônico</title>
<style>
body { font-family: Arial, sans-serif; background:#f5f5f5; margin:0; padding:20px; }
.container { max-width:1200px; margin:auto; background:white; padding:20px; border-radius:8px; box-shadow:0 0 10px rgba(0,0,0,0.1); }
h1 { text-align:center; color:#333; margin-bottom:20px; }
.button-group { display:flex; flex-wrap:wrap; gap:10px; margin-bottom:20px; }
button { padding:10px 15px; border:none; border-radius:4px; font-size:16px; cursor:pointer; transition:0.3s; }
button:hover { opacity:0.9; }
button.add { background:#4CAF50; color:white; }
button.exit { background:#f44336; color:white; }
button.export { background:#2196F3; color:white; }
table { width:100%; border-collapse:collapse; margin-top:20px; }
th, td { border:1px solid #ddd; padding:12px; text-align:left; }
th { background:#f2f2f2; }
tr:nth-child(even){background:#f9f9f9;}
.modal { display:none; position:fixed; z-index:1; left:0; top:0; width:100%; height:100%; background:rgba(0,0,0,0.4); overflow:auto; }
.modal-content { background:white; margin:8% auto; padding:20px; border-radius:8px; width:90%; max-width:600px; }
.close { float:right; font-size:28px; font-weight:bold; cursor:pointer; }
.close:hover { color:black; }
select, button { width:100%; margin:5px 0; padding:10px; border-radius:4px; border:1px solid #ccc; font-size:16px; }
</style>
</head>
<body>
<div class="container">
<h1>Sistema de Ponto Eletrônico</h1>

<div class="button-group">
  <button class="add" id="registrarEntradaBtn">Registrar Entrada</button>
  <button class="exit" id="registrarSaidaBtn">Registrar Saída</button>
  <button class="export" id="exportExcelBtn">Exportar Excel</button>
</div>

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

<!-- Modal Registrar -->
<div id="registrarModal" class="modal">
  <div class="modal-content">
    <span class="close" data-target="registrarModal">&times;</span>
    <h2 id="modalTitle">Registrar</h2>
    <form id="registrarForm">
      <select id="selectColaborador" required></select>
      <button type="submit">Registrar</button>
    </form>
  </div>
</div>

<!-- Biblioteca SheetJS para Excel -->
<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>

<script>
// --- Dados ---
let colaboradores = JSON.parse(localStorage.getItem('colaboradores')) || [];
let registros = JSON.parse(localStorage.getItem('registros')) || [];
let ultimoId = registros.length>0 ? Math.max(...registros.map(r=>r.id)) : 0;

// --- Elementos ---
const entradaBody = document.getElementById('entradaBody');
const saidaBody = document.getElementById('saidaBody');
const registrarModal = document.getElementById('registrarModal');
const selectColaborador = document.getElementById('selectColaborador');
const modalTitle = document.getElementById('modalTitle');

let tipoRegistro = 'Entrada';

// --- Funções ---
function carregarColaboradoresSelect() {
  selectColaborador.innerHTML='<option value="">Selecione o Colaborador</option>';
  colaboradores.forEach(c=>{ if(!c.inativo) selectColaborador.innerHTML+=`<option value="${c.matricula}">${c.nome} (${c.matricula})</option>`; });
}

function salvarRegistros() {
  localStorage.setItem('registros', JSON.stringify(registros));
}

function renderizarRegistros() {
  entradaBody.innerHTML='';
  saidaBody.innerHTML='';
  registros.forEach(r=>{
    const row=`<tr><td>${r.id}</td><td>${r.nome}</td><td>${r.matricula}</td><td>${new Date(r.dataHora).toLocaleString()}</td></tr>`;
    if(r.tipo==='Entrada') entradaBody.innerHTML+=row;
    else saidaBody.innerHTML+=row;
  });
}

// --- Botões ---
document.getElementById('registrarEntradaBtn').onclick = ()=>{
  tipoRegistro='Entrada';
  modalTitle.textContent='Registrar Entrada';
  carregarColaboradoresSelect();
  registrarModal.style.display='block';
};
document.getElementById('registrarSaidaBtn').onclick = ()=>{
  tipoRegistro='Saída';
  modalTitle.textContent='Registrar Saída';
  carregarColaboradoresSelect();
  registrarModal.style.display='block';
};

// Fechar modal
document.querySelectorAll('.close').forEach(btn=>btn.onclick=()=>{
  const t=btn.dataset.target;
  if(t) document.getElementById(t).style.display='none';
});

// Registrar ponto
document.getElementById('registrarForm').addEventListener('submit', e=>{
  e.preventDefault();
  const matricula = selectColaborador.value;
  const colaborador = colaboradores.find(c=>c.matricula===matricula);
  if(!colaborador) return alert('Selecione um colaborador válido');

  ultimoId++;
  registros.push({id:ultimoId,nome:colaborador.nome,matricula,colaborador:colaborador, tipo:tipoRegistro, dataHora:new Date().toISOString()});
  salvarRegistros();
  renderizarRegistros();
  registrarModal.style.display='none';
});

// Exportar Excel
document.getElementById('exportExcelBtn').onclick = ()=>{
  const wb=XLSX.utils.book_new();
  const entradaWS=XLSX.utils.json_to_sheet(registros.filter(r=>r.tipo==='Entrada').map(r=>({ID:r.id,Nome:r.nome,Matrícula:r.matricula,'Data/Hora':new Date(r.dataHora).toLocaleString()})));
  const saidaWS=XLSX.utils.json_to_sheet(registros.filter(r=>r.tipo==='Saída').map(r=>({ID:r.id,Nome:r.nome,Matrícula:r.matricula,'Data/Hora':new Date(r.dataHora).toLocaleString()})));
  XLSX.utils.book_append_sheet(wb,entradaWS,'Entradas');
  XLSX.utils.book_append_sheet(wb,saidaWS,'Saídas');
  XLSX.writeFile(wb,'registros_ponto.xlsx');
};

// --- Inicial ---
renderizarRegistros();

</script>
</body>
</html>
