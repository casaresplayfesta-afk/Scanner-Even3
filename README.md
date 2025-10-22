<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Scanner Even3 - Ler Eventos</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f4f6f9;
      color: #333;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 30px;
    }
    h1 {
      color: #0070f3;
    }
    input {
      width: 90%;
      max-width: 600px;
      padding: 10px;
      font-size: 16px;
      border: 2px solid #0070f3;
      border-radius: 8px;
    }
    button {
      margin-top: 10px;
      padding: 10px 20px;
      font-size: 16px;
      border: none;
      background: #0070f3;
      color: white;
      border-radius: 8px;
      cursor: pointer;
    }
    button:hover {
      background: #0059b2;
    }
    .resultado {
      margin-top: 30px;
      width: 100%;
      max-width: 800px;
      background: white;
      border-radius: 10px;
      padding: 20px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    }
    .sessao {
      border-bottom: 1px solid #ddd;
      padding: 10px 0;
    }
    .sessao:last-child {
      border-bottom: none;
    }
    .erro {
      color: red;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <h1>Scanner Even3</h1>
  <p>Cole o link de sessões do Even3:</p>
  <input id="url" type="text" placeholder="https://www.even3.com.br/participante/sessions/">
  <button onclick="escanear()">🔍 Escanear Evento</button>
  <div id="resultado" class="resultado" style="display:none;"></div>
  <div id="erro" class="erro"></div>

  <script>
  async function escanear() {
    const url = document.getElementById('url').value.trim();
    const resultadoDiv = document.getElementById('resultado');
    const erroDiv = document.getElementById('erro');
    resultadoDiv.style.display = 'none';
    erroDiv.textContent = '';

    if (!url.startsWith('http')) {
      erroDiv.textContent = 'Por favor, insira um link válido do Even3.';
      return;
    }

    const proxyUrl = 'https://api.allorigins.win/get?url=' + encodeURIComponent(url);

    try {
      const res = await fetch(proxyUrl);
      if (!res.ok) throw new Error('Falha ao acessar o proxy.');
      const data = await res.json();
      const html = data.contents;

      const parser = new DOMParser();
      const doc = parser.parseFromString(html, 'text/html');

      // Pega título principal
      const titulo = doc.querySelector('h1')?.textContent?.trim() || 'Título não encontrado';

      // Tenta achar local e palestrante
      const texto = doc.body.innerText;
      const local = (texto.match(/(?:Local|Endereço|Localização)[:\s]*([^\n]+)/i)?.[1] || 'Local não informado').trim();
      const palestrante = (texto.match(/(?:Palestrante|Ministrante|Convidado)[:\s]*([^\n]+)/i)?.[1] || 'Palestrante não informado').trim();

      // Pega possíveis sessões (blocos com horário)
      const sessoes = [];
      const blocos = doc.querySelectorAll('div, section, article');
      blocos.forEach(b => {
        const t = b.innerText;
        if (t.match(/\d{1,2}:\d{2}/) && t.length < 400) {
          sessoes.push(t.replace(/\s+/g, ' ').trim());
        }
      });

      // Monta o HTML de resultado
      let htmlFinal = `<h2>${titulo}</h2>`;
      htmlFinal += `<p><b>Local:</b> ${local}</p>`;
      htmlFinal += `<p><b>Palestrante:</b> ${palestrante}</p>`;
      htmlFinal += `<h3>Sessões encontradas:</h3>`;

      if (sessoes.length === 0) {
        htmlFinal += `<p>Nenhuma sessão encontrada.</p>`;
      } else {
        sessoes.forEach(s => {
          htmlFinal += `<div class="sessao">${s}</div>`;
        });
      }

      resultadoDiv.innerHTML = htmlFinal;
      resultadoDiv.style.display = 'block';

    } catch (err) {
      erroDiv.textContent = 'Erro ao escanear o link: ' + err.message;
    }
  }
  </script>
</body>
</html>
