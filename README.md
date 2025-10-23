<!doctype html>
<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Ponto Eletrônico Local</title>
<style>
  body {
    font-family: Arial, sans-serif;
    background: url('fundo.jpg') no-repeat center center fixed;
    background-size: cover;
    color: #fff;
    margin: 0;
    padding: 0;
  }
  h1 {
    text-align: center;
    background: rgba(0,0,0,0.6);
    padding: 20px;
    margin: 0;
  }
  .container {
    max-width: 900px;
    margin: 20px auto;
    background: rgba(0,0,0,0.7);
    padding: 20px;
    border-radius: 10px;
  }
  input, select, button {
    margin: 5px;
    padding: 8px;
    border-radius: 5px;
    border: none;
  }
  button {
    background: #00c853;
    color: white;
    cursor: pointer;
  }
  button:hover {
    background: #009624;
  }
  table {
    width: 100%;
    border-collapse: collapse;
    background: rgba(255,255,255,0.1);
  }
  th, td {
    border: 1px solid rgba(255,255,255,0.3);
    padding: 8px;
    text-align: center;
  }
  th {
    background: rgba(0,0,0,0.5);
  }
</style>
</head>
<body>
  <h1>Sistema de Ponto Eletrônico Local</h1>
  <div class="container">
    <input type="text" id="nome" placeholder="Nome do Colaborador">
    <button onclick="adicionarColaborador()">Adicionar</button>
    <button onclick="exportarExcel()">Exportar Excel</button>

    <table>
      <thead>
        <tr>
          <th>Nome</th>
          <th>Entrada</th>
          <th>Saída</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody id="lista"></tbody>
    </table>
  </div>

<script>
let colaboradores = JSON.parse(localStorage.getItem('colaboradores')) || [];

function salvarLocal() {
  localStorage.setItem('colaboradores', JSON.stringify(colaboradores));
}

function renderColaboradores() {
  const lista = document.getElementById('lista');
  lista.innerHTML = '';
  colaboradores.forEach(c => {
    lista.innerHTML += `
      <tr>
        <td>${c.nome}</td>
        <td>${c.entrada || '-'}</td>
        <td>${c.saida || '-'}</td>
        <td>
          <button onclick="registrarAcao('${c.id}', 'entrada')">Entrada</button>
          <button onclick="registrarAcao('${c.id}', 'saida')">Saída</button>
          <button style="background:#ff4444;" onclick="registrarAcao('${c.id}', 'delete')">Excluir</button>
        </td>
      </tr>
    `;
  });
}

function adicionarColaborador() {
  const nome = document.getElementById('nome').value.trim();
  if (!nome) return alert('Digite um nome!');
  const novo = { id: Date.now().toString(), nome, entrada: '', saida: '' };
  colaboradores.push(novo);
  salvarLocal();
  renderColaboradores();
  document.getElementById('nome').value = '';
}

function registrarAcao(id, action) {
  const c = colaboradores.find(x => x.id === id);
  if (!c) return;

  if (action === 'entrada') {
    c.entrada = new Date().toLocaleString();
  } else if (action === 'saida') {
    c.saida = new Date().toLocaleString();
  } else if (action === 'delete') {
    // ⚠️ Confirma antes de excluir
    if (confirm(`Tem certeza que deseja excluir ${c.nome}?`)) {
      const index = colaboradores.findIndex(x => x.id === id);
      if (index !== -1) {
        colaboradores.splice(index, 1);
      }
    } else {
      return; // se cancelar, não faz nada
    }
  }

  // 💾 Sempre salva e recarrega
  salvarLocal();
  renderColaboradores();
}

function exportarExcel() {
  let csv = "Nome,Entrada,Saída\n";
  colaboradores.forEach(c => {
    csv += `${c.nome},${c.entrada},${c.saida}\n`;
  });
  const blob = new Blob([csv], { type: "text/csv" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "ponto-eletronico.csv";
  a.click();
}

renderColaboradores();
</script>
</body>
</html>
