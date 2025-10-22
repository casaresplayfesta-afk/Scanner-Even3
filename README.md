<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Organizador de Eventos Even3</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f5f5f5;
      margin: 0;
      padding: 20px;
    }
    h1 {
      text-align: center;
      color: #333;
    }
    input {
      display: block;
      margin: 20px auto;
    }
    .evento {
      background: #fff;
      padding: 15px;
      margin: 10px 0;
      border-radius: 10px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.1);
    }
    .evento strong {
      color: #0077cc;
    }
  </style>
</head>
<body>
  <h1>Organizador de Eventos Even3</h1>
  <p style="text-align:center;">Escolha um arquivo HTML ou JSON exportado do Even3:</p>
  <input type="file" id="fileInput" accept=".html,.json">
  <div id="resultados"></div>

  <script>
    document.getElementById('fileInput').addEventListener('change', function (event) {
      const file = event.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = function (e) {
        const content = e.target.result;
        let eventos = [];

        if (file.name.endsWith('.json')) {
          try {
            eventos = JSON.parse(content);
          } catch {
            alert('Arquivo JSON inválido!');
            return;
          }
        } else {
          // Se for HTML — tenta extrair as sessões
          const parser = new DOMParser();
          const doc = parser.parseFromString(content, 'text/html');

          const items = doc.querySelectorAll('[data-testid*="session-card"], .session');
          items.forEach(item => {
            const titulo = item.querySelector('h3, h2, .title')?.innerText || 'Sem título';
            const data = item.querySelector('.date, [data-testid*="session-date"]')?.innerText || 'Data não informada';
            const hora = item.querySelector('.time, [data-testid*="session-time"]')?.innerText || 'Hora não informada';
            const local = item.querySelector('.location, [data-testid*="session-location"]')?.innerText || 'Local não informado';
            const palestrante = item.querySelector('.speaker, [data-testid*="session-speaker"]')?.innerText || 'Sem palestrante';

            eventos.push({ titulo, data, hora, local, palestrante });
          });
        }

        // Ordenar por data e hora (quando possível)
        eventos.sort((a, b) => (a.data + a.hora).localeCompare(b.data + b.hora));

        // Mostrar
        const container = document.getElementById('resultados');
        container.innerHTML = '';
        if (eventos.length === 0) {
          container.innerHTML = '<p style="text-align:center;color:red;">Nenhum evento encontrado!</p>';
          return;
        }

        eventos.forEach(ev => {
          const div = document.createElement('div');
          div.className = 'evento';
          div.innerHTML = `
            <strong>${ev.titulo}</strong><br>
            📅 ${ev.data} — 🕒 ${ev.hora}<br>
            📍 ${ev.local}<br>
            🎤 ${ev.palestrante}
          `;
          container.appendChild(div);
        });
      };

      reader.readAsText(file);
    });
  </script>
</body>
</html>
