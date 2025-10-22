<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Organizador de Eventos Even3</title>
<style>
  body { font-family: Arial, sans-serif; background:#121212; color:#eaeaea; margin:0; padding:20px; }
  h1 { text-align:center; color:#00b4d8; margin-bottom:10px; }
  textarea, input, button { display:block; margin:10px auto; width:90%; max-width:700px; padding:10px; border-radius:6px; border:none; font-size:16px; }
  textarea { height:200px; background:#1e1e1e; color:#fff; resize:vertical; }
  button { background:#00b4d8; color:#fff; cursor:pointer; }
  button:hover { background:#0096c7; }
  .evento { background:#1f1f1f; padding:15px; margin:10px auto; border-radius:10px; max-width:700px; }
  .evento strong { color:#00b4d8; font-size:18px; }
</style>
</head>
<body>

<h1>Organizador de Eventos Even3</h1>
<p>Cole o HTML do evento do Even3 abaixo:</p>
<textarea id="htmlInput" placeholder="Cole o HTML do site do Even3 aqui"></textarea>
<button onclick="processarHTML()">Organizar Eventos</button>
<div id="resultados"></div>

<script>
function processarHTML() {
  const html = document.getElementById('htmlInput').value;
  const resultados = document.getElementById('resultados');
  resultados.innerHTML = '';

  if (!html.trim()) return alert('Cole algum HTML primeiro!');

  const parser = new DOMParser();
  const doc = parser.parseFromString(html, 'text/html');

  const panels = doc.querySelectorAll('.panel-body');
  const eventos = [];

  panels.forEach(panel => {
    // Data (se tiver, ou você pode preencher manualmente)
    const data = panel.querySelector('.data, .date')?.innerText || 'Data não informada';

    // Hora de entrada e saída
    const horario = panel.querySelector('.horario')?.innerText || '';
    let [horaEntrada, horaSaida] = horario.split('-').map(h => h.trim());
    if (!horaSaida) horaSaida = '';

    // Título
    const titulo = panel.querySelector('.atividade')?.innerText || 'Sem título';

    // Palestrante(s)
    const palestrantes = Array.from(panel.querySelectorAll('.desc-atividade[ng-repeat]'))
                              .map(s => s.innerText)
                              .join(', ') || 'Sem palestrante';

    // Sala / local
    const sala = panel.querySelector('.local, .sala, .location')?.innerText || 'Sala não informada';

    eventos.push({ data, horaEntrada, horaSaida, titulo, palestrantes, sala });
  });

  if (eventos.length === 0) {
    resultados.innerHTML = '<p style="text-align:center;color:#ccc;">Nenhum evento encontrado!</p>';
    return;
  }

  // Ordena por data e hora de entrada
  eventos.sort((a, b) => (a.data + a.horaEntrada).localeCompare(b.data + b.horaEntrada));

  // Renderiza
  eventos.forEach(ev => {
    const div = document.createElement('div');
    div.className = 'evento';
    div.innerHTML = `
      <strong>${ev.titulo}</strong><br>
      📅 ${ev.data} — ⏰ ${ev.horaEntrada} - ${ev.horaSaida}<br>
      📍 ${ev.sala}<br>
      🎤 ${ev.palestrantes}
    `;
    resultados.appendChild(div);
  });
}
</script>

</body>
</html>
