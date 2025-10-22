<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Organizador de Eventos Even3</title>
<style>
  body { font-family: Arial, sans-serif; background:#121212; color:#eaeaea; margin:0; padding:20px; }
  h1 { text-align:center; color:#00b4d8; margin-bottom:10px; }
  input, button { display:block; margin:10px auto; width:90%; max-width:700px; padding:10px; border-radius:6px; border:none; font-size:16px; }
  button { background:#00b4d8; color:#fff; cursor:pointer; }
  button:hover { background:#0096c7; }
  .evento { background:#1f1f1f; padding:15px; margin:10px auto; border-radius:10px; max-width:700px; }
  .evento strong { color:#00b4d8; font-size:18px; }
</style>
</head>
<body>

<h1>Organizador de Eventos Even3</h1>
<p>Escolha o arquivo JSON com os eventos:</p>
<input type="file" id="fileInput" accept=".json">
<div id="resultados"></div>

<script>
const resultados = document.getElementById('resultados');

document.getElementById('fileInput').addEventListener('change', function(e) {
  const file = e.target.files[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = function(e) {
    let eventos;
    try {
      eventos = JSON.parse(e.target.result);
    } catch {
      alert('Arquivo JSON inválido!');
      return;
    }

    // Ordena por data e hora de entrada
    eventos.sort((a,b) => (a.data + a.horaEntrada).localeCompare(b.data + b.horaEntrada));

    resultados.innerHTML = '';
    eventos.forEach(ev => {
      const div = document.createElement('div');
      div.className = 'evento';
      div.innerHTML = `
        <strong>${ev.titulo}</strong><br>
        📅 ${ev.data} — ⏰ ${ev.horaEntrada} - ${ev.horaSaida}<br>
        📍 ${ev.sala}<br>
        🎤 ${ev.palestrante}
      `;
      resultados.appendChild(div);
    });
  };
  reader.readAsText(file);
});
</script>

</body>
</html>
