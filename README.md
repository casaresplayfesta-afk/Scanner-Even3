<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Scanner Even3 — HTML único</title>
  <style>
    :root{--bg:#f7fafc;--card:#fff;--accent:#0ea5a4}
    body{font-family:Inter,system-ui,Segoe UI,Roboto,Arial,Helvetica,sans-serif;margin:0;background:var(--bg);color:#0f172a}
    .wrap{max-width:900px;margin:36px auto;padding:20px}
    .card{background:var(--card);border-radius:12px;box-shadow:0 6px 18px rgba(2,6,23,0.06);padding:18px}
    h1{margin:0 0 8px;font-size:20px}
    p.lead{margin:0 0 16px;color:#475569}
    form{display:flex;gap:8px}
    input{flex:1;padding:10px;border-radius:8px;border:1px solid #e2e8f0}
    button{padding:10px 14px;border-radius:8px;border:0;background:var(--accent);color:white;font-weight:600}
    .error{color:#dc2626;margin-top:12px}
    .result{margin-top:16px}
    .ev{border:1px solid #e6eef0;border-radius:10px;padding:12px;margin-bottom:10px;background:#fcffff}
    .meta{display:flex;justify-content:space-between;gap:12px}
    .muted{color:#64748b;font-size:13px}
    .prog{margin-top:8px}
    .smallbtn{font-size:12px;padding:6px 8px;border-radius:8px;border:1px solid #cbd5e1;background:white}
    footer{margin-top:12px;color:#94a3b8;font-size:13px}
    @media (max-width:600px){.meta{flex-direction:column;align-items:flex-start}}
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <h1>Scanner Even3 — HTML único</h1>
      <p class="lead">Cole o link de um evento Even3 e clique em "Escanear". O código tentará buscar a página (usando um proxy público para contornar CORS) e extrair dados via JSON-LD ou seletores simples. Resultado será ordenado por data/hora.</p>

      <form id="scanForm">
        <input id="url" placeholder="Cole o link do evento (ex: https://even3.com.br/...)" />
        <button id="scanBtn">Escanear</button>
      </form>

      <div id="note" class="muted" style="margin-top:10px">Observação: alguns sites bloqueiam requisições cross-origin. O exemplo usa o proxy <code>api.allorigins.win</code> para facilitar testes. Para produção, recomenda-se implementar um backend (forneço se quiser).</div>

      <div id="error" class="error" role="alert" style="display:none"></div>

      <div id="result" class="result"></div>

      <footer>
        Dica: se o evento não aparecer, pode ser por CORS ou por o Even3 usar renderização dinâmica. Quer que eu gere um backend em Node/Express para você deployar? (posso gerar os arquivos prontos)
      </footer>
    </div>
  </div>

  <script>
    // util: tentar buscar HTML via proxy público (AllOrigins). Troque se preferir outro proxy.
    async function fetchHtmlViaProxy(url){
      const proxy = 'https://api.allorigins.win/raw?url=' + encodeURIComponent(url);
      const r = await fetch(proxy);
      if (!r.ok) throw new Error('Falha ao buscar a página: ' + r.status);
      return await r.text();
    }

    // extrai scripts application/ld+json e tenta parsear eventos
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
        }catch(e){/* ignora */}
      }
      return events;
    }

    // fallback simples: buscar datas ISO na página e h1
    function fallbackParse(html, url){
      const parser = new DOMParser();
      const doc = parser.parseFromString(html, 'text/html');
      const title = doc.querySelector('h1')?.textContent?.trim() || doc.title || null;
      const body = doc.body.textContent || '';
      const iso = body.match(/\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:?\d{0,2}/g);
      const dates = iso ? Array.from(new Set(iso)) : [];
      return [{ name: title, url, start: dates[0] || null }];
    }

    function normalizeEvent(ev){
      return {
        name: ev.name || ev.title || ev.headline || '(sem título)',
        description: ev.description || ev.summary || null,
        start: ev.startDate || ev.start || ev.date || ev.start_date || null,
        end: ev.endDate || ev.end || null,
        location: ev.location?.name || ev.location?.address || (ev.venue && ev.venue.name) || ev.location || null,
        url: ev.url || null,
        schedule: ev.eventSchedule || ev.program || ev.schedule || null
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
        const div = document.createElement('div'); div.className = 'ev';
        const meta = document.createElement('div'); meta.className='meta';
        const left = document.createElement('div');
        left.innerHTML = `<div style="font-weight:700">${ev.name}</div><div class="muted">${ev.location || 'Local não informado'}</div>`;
        const right = document.createElement('div');
        right.style.textAlign='right';
        right.innerHTML = `<div style="font-weight:600">${ev.start ? new Date(ev.start).toLocaleString() : '-'}</div>${ev.end?'<div class="muted">até '+new Date(ev.end).toLocaleString()+'</div>':''}`;
        meta.appendChild(left); meta.appendChild(right);
        div.appendChild(meta);
        if (ev.description) { const p=document.createElement('p'); p.style.marginTop='8px'; p.textContent = ev.description; div.appendChild(p); }
        if (ev.schedule && Array.isArray(ev.schedule) && ev.schedule.length){
          const prog = document.createElement('div'); prog.className='prog'; prog.innerHTML='<div style="font-weight:600">Programação:</div>';
          const ul = document.createElement('ul'); ul.style.margin='6px 0 0 18px';
          ev.schedule.forEach(it=>{ const li=document.createElement('li'); li.textContent = (it.start||it.time? (it.start||it.time)+' — ':'') + (it.name||it.title||it.description||''); ul.appendChild(li); });
          prog.appendChild(ul); div.appendChild(prog);
        }
        const actions = document.createElement('div'); actions.style.marginTop='10px';
        if (ev.url) { const a=document.createElement('a'); a.href=ev.url; a.target='_blank'; a.className='smallbtn'; a.textContent='Abrir evento'; actions.appendChild(a); }
        const cb = document.createElement('button'); cb.className='smallbtn'; cb.style.marginLeft='8px'; cb.textContent='Copiar JSON'; cb.onclick = ()=>navigator.clipboard.writeText(JSON.stringify(ev, null, 2)); actions.appendChild(cb);
        div.appendChild(actions);
        container.appendChild(div);
      })
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
        // ordenar por data/hora (quando possível)
        normalized.sort((a,b)=>{
          if (a.start && b.start) return new Date(a.start) - new Date(b.start);
          if (a.start) return -1; if (b.start) return 1; return 0;
        });
        renderEvents(normalized);
      }catch(ex){
        console.error(ex); err.style.display='block'; err.textContent = 'Erro: ' + (ex.message || ex);
      }finally{
        btn.disabled = false; btn.textContent = 'Escanear';
      }
    });
  </script>
</body>
</html>
