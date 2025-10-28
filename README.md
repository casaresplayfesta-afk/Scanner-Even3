<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Ponto Eletrônico Corporativo</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body { font-family: Arial, sans-serif; background:#e0f7fa; margin:0; padding:20px; }
.container { background:white; padding:25px; border-radius:15px; box-shadow:0 6px 15px rgba(0,0,0,0.3); max-width:900px; margin:auto 0 20px auto; }
h2 { color:#00796b; text-align:center; margin-bottom:15px; }
button, select, input { padding:8px 12px; margin:5px; font-size:14px; border-radius:8px; border:none; cursor:pointer; }
button:hover { opacity:0.85; }
#entrada { background-color:#4CAF50; color:white; }
#saida { background-color:#f44336; color:white; }
#inicioIntervalo { background-color:#FF9800; color:white; }
#fimIntervalo { background-color:#9C27B0; color:white; }
#exportar { background-color:#2196F3; color:white; }
#limpar { background-color:#607D8B; color:white; }
table { width:100%; margin-top:15px; border-collapse: collapse; font-size:14px; }
th, td { border:1px solid #ddd; padding:8px; text-align:center; }
th { background-color:#b2dfdb; color:#004d40; }
td { font-weight:500; }
canvas { margin-top:25px; }
.flex { display:flex; flex-wrap:wrap; justify-content:center; }
</style>
</head>
<body>
<div class="container">
<h2>Ponto Eletrônico Corporativo</h2>
<div class="flex">
<select id="funcionario"></select>
<input type="text" id="novoFuncionario" placeholder="Novo funcionário">
<button id="adicionarFuncionario">Adicionar Funcionário</button>
</div>
<div class="flex">
<button id="entrada">Entrada</button>
<button id="saida">Saída</button>
<button id="inicioIntervalo">Início Intervalo</button>
<button id="fimIntervalo">Fim Intervalo</button>
<button id="exportar">Exportar CSV</button>
<button id="limpar">Limpar Registros</button>
</div>
<table>
<thead>
<tr>
<th>Funcionário</th>
<th>Data</th>
<th>Tipo</th>
<th>Horário</th>
<th>Horas Trabalhadas</th>
</tr>
</thead>
<tbody id="tabelaPonto"></tbody>
<tfoot>
<tr>
<td colspan="4"><strong>Total de Horas</strong></td>
<td id="totalHoras">0</td>
</tr>
</tfoot>
</table>
</div>
<div class="container">
<h2>Dashboard Corporativo</h2>
<canvas id="chartDias"></canvas>
<canvas id="chartSemana"></canvas>
</div>
<script>
const funcionarioSelect = document.getElementById('funcionario');
const tabela = document.getElementById('tabelaPonto');
const totalHorasCell = document.getElementById('totalHoras');

function carregarFuncionarios() {
let funcionarios = JSON.parse(localStorage.getItem('funcionarios')) || [];
funcionarioSelect.innerHTML = '';
funcionarios.forEach(f => {
const option = document.createElement('option');
option.value = f;
option.text = f;
funcionarioSelect.appendChild(option);
});
if(funcionarios.length === 0) {
funcionarioSelect.innerHTML = '<option value="">Sem funcionários</option>';
}
}

document.getElementById('adicionarFuncionario').addEventListener('click', () => {
const nome = document.getElementById('novoFuncionario').value.trim();
if(!nome) return alert('Digite um nome!');
let funcionarios = JSON.parse(localStorage.getItem('funcionarios')) || [];
if(!funcionarios.includes(nome)) {
funcionarios.push(nome);
localStorage.setItem('funcionarios', JSON.stringify(funcionarios));
document.getElementById('novoFuncionario').value = '';
carregarFuncionarios();
} else {
alert('Funcionário já existe!');
}
});

function registrarPonto(tipo) {
const funcionario = funcionarioSelect.value;
if(!funcionario) return alert('Selecione um funcionário!');
const agora = new Date();
let registros = JSON.parse(localStorage.getItem('registros')) || [];
registros.push({ funcionario, tipo, horario: agora.toLocaleTimeString(), data: agora.toLocaleDateString(), horarioISO: agora.toISOString() });
localStorage.setItem('registros', JSON.stringify(registros));
atualizarTabela();
}

function calcularHorasPorFuncionario() {
let registros = JSON.parse(localStorage.getItem('registros')) || [];
let dados = {};
registros.forEach(r => {
if(!dados[r.funcionario]) dados[r.funcionario] = {};
if(!dados[r.funcionario][r.data]) dados[r.funcionario][r.data] = [];
dados[r.funcionario][r.data].push(r);
});
let totalGeral = 0;
let horasFuncionario = {};
Object.keys(dados).forEach(f => {
horasFuncionario[f] = 0;
Object.keys(dados[f]).forEach(d => {
let diaRegistros = dados[f][d];
let entrada = null, inicioIntervalo = null, total = 0;
diaRegistros.forEach(r => {
let hora = new Date(r.horarioISO);
if(r.tipo==='Entrada') entrada=hora;
if(r.tipo==='Início Intervalo') inicioIntervalo=hora;
if(r.tipo==='Fim Intervalo'&&inicioIntervalo){
total+=(hora-inicioIntervalo)/3600000;
inicioIntervalo=null;
}
if(r.tipo==='Saída'&&entrada){
if(inicioIntervalo){
total+=(hora-inicioIntervalo)/3600000;
inicioIntervalo=null;
} else total+=(hora-entrada)/3600000;
entrada=null;
}
});
horasFuncionario[f]+=total;
totalGeral+=total;
});
});
return { totalGeral: totalGeral.toFixed(2), horasFuncionario };
}

function atualizarTabela() {
tabela.innerHTML = '';
let registros = JSON.parse(localStorage.getItem('registros')) || [];
let { totalGeral } = calcularHorasPorFuncionario();
registros.forEach(r => {
const linha = document.createElement('tr');
const horas = totalGeral;
linha.innerHTML = `<td>${r.funcionario}</td><td>${r.data}</td><td>${r.tipo}</td><td>${r.horario}</td><td>${horas}</td>`;
tabela.appendChild(linha);
});
totalHorasCell.innerText = totalGeral;
atualizarGraficos();
}

document.getElementById('entrada').addEventListener('click', ()=>registrarPonto('Entrada'));
document.getElementById('saida').addEventListener('click', ()=>registrarPonto('Saída'));
document.getElementById('inicioIntervalo').addEventListener('click', ()=>registrarPonto('Início Intervalo'));
document.getElementById('fimIntervalo').addEventListener('click', ()=>registrarPonto('Fim Intervalo'));
document.getElementById('exportar').addEventListener('click', ()=>{
let registros = JSON.parse(localStorage.getItem('registros')) || [];
if(registros.length===0){ alert('Nenhum registro para exportar!'); return; }
let csv='Funcionário,Data,Tipo,Horário,Horas Trabalhadas\n';
registros.forEach(r=>{
const { totalGeral } = calcularHorasPorFuncionario();
csv+=`${r.funcionario},${r.data},${r.tipo},${r.horario},${totalGeral}\n`;
});
const blob=new Blob([csv],{type:'text/csv'});
const url=URL.createObjectURL(blob);
const a=document.createElement('a');
a.href=url;
a.download='registros_ponto.csv';
a.click();
URL.revokeObjectURL(url);
});
document.getElementById('limpar').addEventListener('click', ()=>{
if(confirm('Deseja realmente limpar todos os registros?')){
localStorage.removeItem('registros');
atualizarTabela();
}
});

window.onload = ()=>{ carregarFuncionarios(); atualizarTabela(); };

// Gráficos corporativos
let chartDias=null;
let chartSemana=null;
function atualizarGraficos(){
let registros = JSON.parse(localStorage.getItem('registros')) || [];
let { horasFuncionario } = calcularHorasPorFuncionario();
const labels = Object.keys(horasFuncionario);
const data = Object.values(horasFuncionario);
if(chartDias) chartDias.destroy();
const ctxDias=document.getElementById('chartDias').getContext('2d');
chartDias = new Chart(ctxDias,{type:'bar',data:{labels, datasets:[{label:'Horas por Funcionário',data, backgroundColor:'#4CAF50'}]}, options:{responsive:true, plugins:{legend:{display:false}}}});
if(chartSemana) chartSemana.destroy();
const ctxSemana=document.getElementById('chartSemana').getContext('2d');
chartSemana = new Chart(ctxSemana,{type:'line', data:{labels, datasets:[{label:'Horas Semanais', data, borderColor:'#2196F3', fill:false} ] }, options:{responsive:true, plugins:{legend:{display:false}}}});
}
</script>
</body>
</html>
