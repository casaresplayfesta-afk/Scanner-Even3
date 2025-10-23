<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Ponto Eletrônico</title>
<style>
body { font-family: Arial,sans-serif; background:#f5f5f5; margin:0; padding:20px; }
.container { max-width:1200px; margin:auto; background:white; padding:20px; border-radius:8px; box-shadow:0 0 10px rgba(0,0,0,0.1);}
h1 { text-align:center; color:#333; margin-bottom:20px;}
.button-group { display:flex; flex-wrap:wrap; gap:10px; margin-bottom:20px;}
button { padding:10px 15px; border:none; border-radius:4px; font-size:16px; cursor:pointer; transition:0.3s;}
button:hover { opacity:0.9;}
button.add { background:#4CAF50; color:white; }
button.edit { background:#FFC107; color:white;}
button.del { background:#F44336; color:white;}
button.reg { background:#2196F3; color:white;}
table { width:100%; border-collapse:collapse; margin-top:20px;}
th, td { border:1px solid #ddd; padding:12px; text-align:left;}
th { background:#f2f2f2;}
tr:nth-child(even){background:#f9f9f9;}
.modal { display:none; position:fixed; z-index:1; left:0; top:0; width:100%; height:100%; background:rgba(0,0,0,0.4); overflow:auto;}
.modal-content { background:white; margin:5% auto; padding:20px; border-radius:8px; width:90%; max-width:600px;}
.close { float:right; font-size:28px; font-weight:bold; cursor:pointer;}
.close:hover { color:black;}
select,input,button { width:100%; margin:5px 0; padding:10px; border-radius:4px; border:1px solid #ccc; font-size:16px; }
</style>
</head>
<body>
<div class="container">
<h1>Sistema de Ponto Eletrônico</h1>

<div class="button-group">
  <button class="add" id="addColabBtn">Adicionar Colaborador</button>
  <button class="edit" id="editColabBtn">Editar Colaborador</button>
  <button class="del" id="deleteColabBtn">Excluir Colaborador</button>
  <button class="reg" id="entradaBtn">Registrar Entrada</button>
  <button class="reg" id="saidaBtn">Registrar Saída</button>
  <button class="reg" id="exportExcelBtn">Exportar Excel</button>
</div>

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
    <span class="close" data-target="colabModal">&times;</span>
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
    <span class="close" data-target="pontoModal">&times;</span>
    <h2 id="pontoTitle">Registrar Ponto</h2>
    <form id="pontoForm">
      <select id="selectColaboradorPonto" required></select>
      <button type="submit">Registrar</button>
    </form>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script>
<script>
// --- Dados ---
let colaboradores = JSON.parse(localStorage.getItem('colaboradores')) || [];
let registros = JSON.parse(localStorage.getItem('registros')) || [];
let ultimoIdColab = colaboradores.length>0 ? Math.max(...colaboradores.map(c=>c.id)) : 0;
let ultimoIdRegistro = registros.length>0 ? Math.max(...registros.map(r=>r.id)) : 0;

// --- DOM ---
const colabBody = document.getElementById('colabBody');
const entradaBody = document.getElementById('entradaBody');
const saidaBody = document.getElementById('saidaBody');
const colabModal = document.getElementById('colabModal');
const pontoModal = document.getElementById('pontoModal');
const selectColab = document.getElementById('selectColab');
const selectColabPonto = document.getElementById('selectColaboradorPonto');
const modalTitle = document.getElementById('modalTitle');
let tipoPonto = 'Entrada';

// --- Funções ---
function salvarLocal() {
  localStorage.setItem('colaboradores', JSON.stringify(colaboradores));
  localStorage.setItem('registros', JSON.stringify(registros));
}

function renderColaboradores() {
  colabBody.innerHTML = '';
  colaboradores.forEach(c => {
    if (!c.inativo) {
      colabBody.innerHTML += `
        <tr>
          <td>${c.id}</td>
          <td>${c.nome}</td>
          <td>${c.matricula}</td>
          <td>${c.cargo}</td>
          <td>${c.turno}</td>
          <td><button class="del" onclick="excluirColaborador(${c.id})">Excluir</button></td>
        </tr>`;
    }
  });
}

function excluirColaborador(id) {
  const colab = colaboradores.find(c => c.id === id && !c.inativo);
  if (!colab) return alert("Colaborador não encontrado.");
  if (!confirm(`Deseja realmente excluir ${colab.nome}?`)) return;
  colaboradores = colaboradores.filter(c => c.id !== id);
  salvarLocal();
  renderColaboradores();
  alert("Colaborador excluído com sucesso!");
}

function renderRegistros() {
  entradaBody.innerHTML=''; saidaBody.innerHTML='';
  registros.forEach(r=>{
    const row=`<tr><td>${r.id}</td><td>${r.nome}</td><td>${r.matricula}</td><td>${new Date(r.dataHora).toLocaleString()}</td></tr>`;
    if(r.tipo==='Entrada') entradaBody.innerHTML+=row;
    else saidaBody.innerHTML+=row;
  });
}

function carregarSelectColab(showAll=false) {
  selectColab.innerHTML='';
  selectColabPonto.innerHTML='';
  colaboradores.forEach(c=>{
    if(showAll || !c.inativo) {
      const option=`<option value="${c.id}">${c.nome} (${c.matricula})</option>`;
      selectColab.innerHTML+=option;
      selectColabPonto.innerHTML+=option;
    }
  });
}

// --- Botões ---
document.getElementById('addColabBtn').onclick = ()=>{
  modalTitle.textContent='Adicionar Colaborador';
  selectColab.style.display='none';
  colabModal.style.display='block';
  document.getElementById('colabForm').reset();
  document.getElementById('colabSubmit').dataset.action='add';
};

document.getElementById('editColabBtn').onclick = ()=>{
  modalTitle.textContent='Editar Colaborador';
  selectColab.style.display='block';
  carregarSelectColab(true);
  colabModal.style.display='block';
  document.getElementById('colabSubmit').dataset.action='edit';
};

document.getElementById('deleteColabBtn').onclick = ()=>{
  alert("Agora você pode excluir diretamente clicando em 'Excluir' na tabela de colaboradores.");
};

// Registrar ponto
document.getElementById('entradaBtn').onclick = ()=>{
  tipoPonto='Entrada';
  pontoModal.style.display='block';
  carregarSelectColab();
  document.getElementById('pontoTitle').textContent='Registrar Entrada';
};
document.getElementById('saidaBtn').onclick = ()=>{
  tipoPonto='Saída';
  pontoModal.style.display='block';
  carregarSelectColab();
  document.getElementById('pontoTitle').textContent='Registrar Saída';
};

// Exportar Excel
document.getElementById('exportExcelBtn').onclick = ()=>{
  const wb=XLSX.utils.book_new();
  const entradaWS=XLSX.utils.json_to_sheet(registros.filter(r=>r.tipo==='Entrada').map(r=>({ID:r.id,Nome:r.nome,Matrícula:r.matricula,'Data/Hora':new Date(r.dataHora).toLocaleString()})));
  const saidaWS=XLSX.utils.json_to_sheet(registros.filter(r=>r.tipo==='Saída').map(r=>({ID:r.id,Nome:r.nome,Matrícula:r.matricula,'Data/Hora':new Date(r.dataHora).toLocaleString()})));
  XLSX.utils.book_append_sheet(wb,entradaWS,'Entradas');
  XLSX.utils.book_append_sheet(wb,saidaWS,'Saídas');
  XLSX.writeFile(wb,'registros_ponto.xlsx');
};

// Fechar modal
document.querySelectorAll('.close').forEach(btn=>btn.onclick=()=>{ btn.closest('.modal').style.display='none'; });

// Formulário Colaborador
document.getElementById('colabForm').addEventListener('submit', e=>{
  e.preventDefault();
  const action=document.getElementById('colabSubmit').dataset.action;
  const id=parseInt(selectColab.value);
  const nome=document.getElementById('colabNome').value.trim();
  const matricula=document.getElementById('colabMatricula').value.trim();
  const cargo=document.getElementById('colabCargo').value.trim();
  const turno=document.getElementById('colabTurno').value;

  if(action==='add'){
    ultimoIdColab++;
    colaboradores.push({id:ultimoIdColab,nome,matricula,cargo,turno,inativo:false});
  } else if(action==='edit'){
    const colab=colaboradores.find(c=>c.id===id);
    if(colab){ colab.nome=nome; colab.matricula=matricula; colab.cargo=cargo; colab.turno=turno; }
  }
  salvarLocal();
  renderColaboradores();
  colabModal.style.display='none';
  document.getElementById('colabForm').reset();
});

// Formulário Ponto
document.getElementById('pontoForm').addEventListener('submit', e=>{
  e.preventDefault();
  const id=parseInt(selectColabPonto.value);
  const colab=colaboradores.find(c=>c.id===id);
  if(!colab) return;
  ultimoIdRegistro++;
  registros.push({id:ultimoIdRegistro,nome:colab.nome,matricula:colab.matricula,tipo:tipoPonto,dataHora:new Date().toISOString()});
  salvarLocal();
  renderRegistros();
  pontoModal.style.display='none';
});

// Inicial
renderColaboradores();
renderRegistros();

</script>
</body>
</html>
