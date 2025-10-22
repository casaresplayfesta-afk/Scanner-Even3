<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Scanner Even3 — com Palestrante e Local editável</title>
  <style>
    :root{--bg:#f7fafc;--card:#fff;--accent:#0ea5a4}
    body{font-family:Inter,system-ui,Segoe UI,Roboto,Arial,Helvetica,sans-serif;margin:0;background:var(--bg);color:#0f172a}
    .wrap{max-width:900px;margin:36px auto;padding:20px}
    .card{background:var(--card);border-radius:12px;box-shadow:0 6px 18px rgba(2,6,23,0.06);padding:18px}
    h1{margin:0 0 8px;font-size:20px}
    p.lead{margin:0 0 16px;color:#475569}
    form{display:flex;gap:8px}
    input{flex:1;padding:10px;border-radius:8px;border:1px solid #e2e8f0}
    button{padding:10px 14px;border-radius:8px;border:0;background:var(--accent);color:white;font-weight:600;cursor:pointer}
    .error{color:#dc2626;margin-top:12px}
    .result{margin-top:16px}
    .ev{border:1px solid #e6eef0;border-radius:10px;padding:12px;margin-bottom:10px;background:#fcffff}
    .meta{display:flex;justify-content:space-between;gap:12px}
    .muted{color:#64748b;font-size:13px}
    .prog{margin-top:8px}
    .smallbtn{font-size:12px;padding:6px 8px;border-radius:8px;border:1px solid #cbd5e1;background:white;cursor:pointer}
    footer{margin-top:12px;color:#94a3b8;font-size:13px}
    @media (max-width:600px){.meta{flex-direction:column;align-items:flex-start}}
    input.editable{border:1px dashed #0ea5a4;padding:6px;border-radius:6px;width:100%;margin-top:6px}
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <h1>Scanner Even3 — com Palestrante e Local editável</h1>
      <p class="lead">Cole o link do evento Even3 e veja as informações completas. Agora detecta <strong>palestrante</strong> e permite <strongeditar o local</strong>.</p>

      <form id="scanForm">
        <input id="url" placeholder="Cole o link do evento (ex: https://even3.com.br/...)" />
        <button id="scanBtn">Escanear</button>
      </form>

      <div id="error" class="error" role="alert" style="display:none"></div>
      <div id="result" class="result"></div>

      <footer>© 2025 Scanner Even3 — versão com palestrante e edição de local</footer>
    </div>
  </div>

  <script>
    async function fetchHtmlViaProxy(url){
      const proxy = 'https://api.allorigins.win/raw?url=' + encodeURIComponent(url);
      const r = await fetch(proxy);
      if (!r.ok) throw new Error('Falha ao buscar a página: ' + r.status);
      return await r.text();
    }

    function extractJsonLd(html){
      const parser = new DOMParser();
      const doc = parser.parseFromString(html, 'text/html');
      const scripts = Array.from(doc.querySelectorAll('script[type="application/ld+json"]'));
      const events = [];
      for(const s of scripts){
        try{
          const j = JSON.parse(s.textContent);
          if (Array.isArray(j)){
            j.forEach(item=>{ if(item['@type'] && String(item['@type']).toLowerCase().includes('event')) events.push(item) })
          } else if (j['@graph']){
            j['@graph'].forEach(item=>{ if(item['@type'] && String(item['@type']).toLowerCase().includes('event')) events.push(item) })
          } else if (j['@type'] && String(j['@type']).toLowerCase().includes('event')){
            events.push(j)
          }
        }catch(e){}
      }
      return events;
    }

    // 🔧 Fallback aprimorado — local + palestrante
    function fallbackParse(html, url){
      const parser = new DOMParser();
      const doc = parser.parseFromString(html, 'text/html');
      const title = doc.querySelector('h1')?.textContent?.trim() || doc.title || null;
      const body = doc.body.textContent || '';

      const iso = body.match(/\d{4}-\d{2}-\d{2}T\d{2}:\d{2}/g);
      const dates = iso ? Array.from(new Set(iso)) : [];

      // Buscar local
      const localMatch = body.match(/(?:Local|Endereço|Localização)[:\s]*([\wÀ-ÿ\s,\-\.\(\)]+)/i);
      const local = localMatch ? localMatch[1].trim() : "(local não informado)";

      // Buscar palestrante
      const palestranteMatch = body.match(/(?:Palestrante|Ministrante|Convidado)[:\s]*([\wÀ-ÿ\s,\.]+)/i);
      const palestrante = palestranteMatch ? palestranteMatch[1].trim() : "(palestrante não informado)";

      return [{
        name: title,
        url,
        start: dates[0] || null,
        location: local,
        speaker: palestrante
      }];
    }

    function normalizeEvent(ev){
      return {
        name: ev.name || ev.title || ev.headline || '(sem título)',
        description: ev.description || ev.summary || null,
        start: ev.startDate || ev.start || ev.date || ev.start_date || null,
        end: ev.endDate || ev.end || null,
        location: ev.location?.name || ev.location?.address || ev.venue?.name || ev.location || "(local não informado)",
        speaker: ev.performer?.name || ev.speaker?.name || ev.speaker || "(palestrante não informado)",
        url: ev.url || null
      }
    }

    function renderEvents(list){
      const container = document.getElementById('result');
      container.innerHTML = '';
      if (!list || list.length === 0){
        container.innerHTML = '<div class="muted">Nenhum evento encontrado.</div>';
        return;
      }

      list.forEach(ev=>{
        const div = document.createElement('div');
        div.className = 'ev';
        const meta = document.createElement('div'); meta.className='meta';

        const left = document.createElement('div');
        left.innerHTML = `<div style="font-weight:700">${ev.name}</div>
        <div class="muted">🗓 ${ev.start ? new Date(ev.start).toLocaleString() : '-'}</div>
        <div class="muted">🎤 ${ev.speaker}</div>
        <div class="muted">📍 <input class="editable" value="${ev.location}" onchange="this.dataset.value=this.value" />`;

        meta.appendChild(left);
        div.appendChild(meta);

        if (ev.description) {
          const p=document.createElement('p');
          p.style.marginTop='8px';
          p.textContent = ev.description;
          div.appendChild(p);
        }

        const actions = document.createElement('div');
        actions.style.marginTop='10px';

        if (ev.url) {
          const a=document.createElement('a');
          a.href=ev.url;
          a.target='_blank';
          a.className='smallbtn';
          a.textContent='Abrir evento';
          actions.appendChild(a);
        }

        const cb = document.createElement('button');
        cb.className='smallbtn';
        cb.style.marginLeft='8px';
        cb.textContent='Copiar JSON';
        cb.onclick = ()=>{
          const locInput = div.querySelector('input.editable');
          const localAtualizado = locInput?.dataset.value || ev.location;
          const obj = {...ev, location: localAtualizado};
          navigator.clipboard.writeText(JSON.stringify(obj, null, 2));
        };
        actions.appendChild(cb);

        div.appendChild(actions);
        container.appendChild(div);
      });
    }

    document.getElementById('scanForm').addEventListener('submit', async function(e){
      e.preventDefault();
      const url = document.getElementById('url').value.trim();
      const err = document.getElementById('error'); err.style.display='none'; err.textContent='';
      const res = document.getElementById('result'); res.innerHTML='';
      if(!url){ err.style.display='block'; err.textContent='Cole o link do evento.'; return; }
      const btn = document.getElementById('scanBtn'); btn.disabled = true; btn.textContent = 'Escaneando...';
      try{
        const html = await fetchHtmlViaProxy(url);
        let events = extractJsonLd(html);
        if (events.length === 0){
          events = fallbackParse(html, url);
        }
        const normalized = events.map(normalizeEvent);
        normalized.sort((a,b)=>{
          if (a.start && b.start) return new Date(a.start) - new Date(b.start);
          if (a.start) return -1; if (b.start) return 1; return 0;
        });
        renderEvents(normalized);
      }catch(ex){
        err.style.display='block';
        err.textContent = 'Erro: ' + (ex.message || ex);
      }finally{
        btn.disabled = false; btn.textContent = 'Escanear';
      }
    });
  </script>
</body>
</html>
