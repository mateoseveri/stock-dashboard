<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Dashboard de acciones (5 empresas)</title>
  <script src="https://cdn.plot.ly/plotly-2.30.0.min.js"></script>
  <style>
    :root{
      --bg:#0b1020; --panel:#111a33; --panel2:#0f1730; --text:#e8eefc; --muted:#a9b7e1;
      --border:#23305a; --accent:#6aa6ff; --danger:#ff6b6b; --good:#55efc4;
    }
    *{box-sizing:border-box}
    body{margin:0; font-family:system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Cantarell,"Noto Sans",sans-serif;
      background: radial-gradient(1200px 700px at 30% -10%, #1b2a6b 0%, rgba(27,42,107,0) 55%),
                  radial-gradient(900px 600px at 90% 10%, #3a1b6b 0%, rgba(58,27,107,0) 55%),
                  var(--bg);
      color:var(--text);
    }
    header{padding:22px 20px 12px; border-bottom:1px solid var(--border); background:rgba(11,16,32,0.65); backdrop-filter: blur(10px); position:sticky; top:0; z-index:5;}
    .wrap{max-width:1200px; margin:0 auto;}
    h1{font-size:18px; margin:0 0 6px; letter-spacing:.2px}
    .sub{color:var(--muted); font-size:13px; margin:0}

    .toolbar{display:flex; gap:12px; align-items:flex-end; flex-wrap:wrap; margin-top:14px;}
    .field{display:flex; flex-direction:column; gap:6px;}
    label{font-size:12px; color:var(--muted)}
    input[type="search"], select{background:var(--panel); border:1px solid var(--border); color:var(--text);
      padding:10px 10px; border-radius:10px; min-width:260px; outline:none;}
    input[type="search"]::placeholder{color:#7f92c8}
    .seg{display:flex; border:1px solid var(--border); background:var(--panel); border-radius:12px; overflow:hidden}
    .seg button{border:0; background:transparent; color:var(--muted); padding:10px 12px; cursor:pointer; font-weight:600; font-size:12px}
    .seg button.active{background:rgba(106,166,255,.16); color:var(--text)}

    .toggles{display:flex; gap:10px; align-items:center; flex-wrap:wrap}
    .chip{display:flex; gap:8px; align-items:center; padding:8px 10px; border-radius:12px; border:1px solid var(--border); background:rgba(17,26,51,.65)}
    .chip input{accent-color:var(--accent)}
    .chip span{font-size:12px; color:var(--text)}

    main{padding:18px 20px 30px;}
    .grid{display:grid; grid-template-columns: 1.4fr .6fr; gap:14px; align-items:start}
    .card{background:rgba(17,26,51,.72); border:1px solid var(--border); border-radius:16px; padding:14px; box-shadow: 0 10px 30px rgba(0,0,0,.25)}
    .card h2{font-size:13px; margin:0 0 10px; color:var(--muted); font-weight:650}

    #chart{height:520px;}

    .stats{display:grid; grid-template-columns: 1fr 1fr; gap:10px}
    .stat{background:rgba(15,23,48,.7); border:1px solid rgba(35,48,90,.8); border-radius:14px; padding:10px}
    .stat .k{color:var(--muted); font-size:11px; margin-bottom:6px}
    .stat .v{font-size:16px; font-weight:750}
    .stat .s{font-size:12px; color:var(--muted)}

    .hint{margin-top:10px; font-size:12px; color:var(--muted); line-height:1.35}
    .hint code{background:rgba(106,166,255,.14); border:1px solid rgba(106,166,255,.25); padding:2px 6px; border-radius:8px; color:var(--text)}

    .footer{margin-top:12px; font-size:12px; color:var(--muted)}

    /* Plotly tweaks */
    .js-plotly-plot .plotly .modebar{background:transparent !important}
  </style>
</head>
<body>
  <header>
    <div class="wrap">
      <h1>Dashboard de acciones - OHLC + volumen + indicadores</h1>
      <p class="sub">Buscador por empresa, filtros de período (D / 6M / M / Y / MAX) e indicadores (medias móviles y volumen promedio).</p>

      <div class="toolbar">
        <div class="field">
          <label for="companySearch">Empresa (buscador)</label>
          <input id="companySearch" type="search" list="companyList" placeholder="Ej: Apple o AAPL" />
          <datalist id="companyList"></datalist>
        </div>

        <div class="field">
          <label>Período</label>
          <div class="seg" role="tablist" aria-label="Período">
            <button data-range="D" class="active">D</button>
            <button data-range="6M">6M</button>
            <button data-range="M">M</button>
            <button data-range="Y">Y</button>
            <button data-range="MAX">MAX</button>
          </div>
        </div>

        <div class="field">
          <label>Indicadores</label>
          <div class="toggles">
            <label class="chip"><input id="ma20" type="checkbox" checked /><span>MA(20)</span></label>
            <label class="chip"><input id="ma50" type="checkbox" /><span>MA(50)</span></label>
            <label class="chip"><input id="vma20" type="checkbox" checked /><span>Vol. prom(20)</span></label>
          </div>
        </div>
      </div>
    </div>
  </header>

  <main>
    <div class="wrap grid">
      <section class="card">
        <div id="chart"></div>
        <div class="footer">Tip: podés hacer zoom con el mouse y doble click para resetear. En móvil, pellizcá para zoom.</div>
      </section>

      <aside class="card">
        <h2>Resumen rápido</h2>
        <div class="stats">
          <div class="stat"><div class="k">Empresa</div><div id="stName" class="v">-</div><div id="stTicker" class="s">-</div></div>
          <div class="stat"><div class="k">Último cierre</div><div id="stClose" class="v">-</div><div id="stChg" class="s">-</div></div>
          <div class="stat"><div class="k">Volatilidad (20)</div><div id="stVol" class="v">-</div><div class="s">std(retornos diarios) (aprox.)</div></div>
          <div class="stat"><div class="k">Volumen prom (20)</div><div id="stAvgVol" class="v">-</div><div class="s">rolling</div></div>
        </div>
        <div class="hint">
          <strong>Cómo correrlo:</strong> por seguridad del navegador, para leer los archivos locales usá un servidor.
          En la carpeta del proyecto corré <code>python -m http.server 8000</code> y abrí <code>http://localhost:8000</code>.
        </div>
      </aside>
    </div>
  </main>

<script>
  // === Config ===
  // Editá acá los nombres “amigables” si querés.
  const COMPANY_MAP = {
    "AAPL.US": { name: "Apple Inc." , file: "data/aapl.us.txt" },
    "AAON.US": { name: "AAON, Inc." , file: "data/aaon.us.txt" },
    "ABAT.US": { name: "ABAT (editar nombre)" , file: "data/abat.us.txt" },
    "AAPG.US": { name: "AAPG (editar nombre)" , file: "data/aapg.us.txt" },
    "AARD.US": { name: "AARD (editar nombre)" , file: "data/aard.us.txt" },
  };

  const RANGE_PRESETS = {
    // D: último tramo en granularidad diaria (por defecto ~ 60 ruedas)
    "D":   { mode: "daily",   windowDays: 90 },
    // 6M: diario, 6 meses
    "6M":  { mode: "daily",   windowDays: 183 },
    // M: mensual (OHLC resample), últimos 5 años
    "M":   { mode: "monthly", windowDays: 365 * 5 },
    // Y: anual (OHLC resample), últimos 20 años
    "Y":   { mode: "yearly",  windowDays: 365 * 20 },
    // MAX: todo, diario (para AAPL puede ser grande)
    "MAX": { mode: "daily",   windowDays: null },
  };

  // === State ===
  let currentTicker = "AAPL.US";
  let currentRange = "D";
  const cache = new Map(); // ticker -> parsed rows

  // === Utilities ===
  function parseYYYYMMDD(s){
    const y = +s.slice(0,4), m = +s.slice(4,6)-1, d = +s.slice(6,8);
    return new Date(Date.UTC(y,m,d));
  }
  function fmtDate(d){
    const yyyy = d.getUTCFullYear();
    const mm = String(d.getUTCMonth()+1).padStart(2,'0');
    const dd = String(d.getUTCDate()).padStart(2,'0');
    return `${yyyy}-${mm}-${dd}`;
  }
  function fmtNum(x, digits=2){
    if (x === null || x === undefined || Number.isNaN(x)) return "-";
    return Number(x).toLocaleString(undefined,{maximumFractionDigits:digits, minimumFractionDigits:digits});
  }
  function fmtInt(x){
    if (x === null || x === undefined || Number.isNaN(x)) return "-";
    return Math.round(x).toLocaleString();
  }

  function rollingMean(values, window){
    const out = new Array(values.length).fill(null);
    let sum = 0;
    let q = [];
    for (let i=0; i<values.length; i++){
      const v = values[i];
      q.push(v);
      sum += v;
      if (q.length > window) sum -= q.shift();
      if (q.length === window) out[i] = sum/window;
    }
    return out;
  }

  function rollingStd(values, window){
    const out = new Array(values.length).fill(null);
    let q=[];
    for (let i=0;i<values.length;i++){
      q.push(values[i]);
      if(q.length>window) q.shift();
      if(q.length===window){
        const mean = q.reduce((a,b)=>a+b,0)/window;
        const varr = q.reduce((a,b)=>a+(b-mean)*(b-mean),0)/window;
        out[i] = Math.sqrt(varr);
      }
    }
    return out;
  }

  function sliceByWindow(rows, windowDays){
    if (!windowDays) return rows;
    const last = rows[rows.length-1]?.date;
    if(!last) return rows;
    const cutoff = new Date(last.getTime() - windowDays*24*3600*1000);
    // keep rows where date >= cutoff
    let idx = 0;
    while(idx < rows.length && rows[idx].date < cutoff) idx++;
    return rows.slice(idx);
  }

  function groupKey(date, mode){
    const y = date.getUTCFullYear();
    const m = date.getUTCMonth();
    if(mode === 'monthly') return `${y}-${String(m+1).padStart(2,'0')}`;
    if(mode === 'yearly') return `${y}`;
    return fmtDate(date);
  }

  // Resample OHLCV to monthly/yearly
  function resampleOHLCV(rows, mode){
    if(mode === 'daily') return rows;
    const buckets = new Map();
    for(const r of rows){
      const k = groupKey(r.date, mode);
      if(!buckets.has(k)){
        buckets.set(k, {key:k, date:r.date, open:r.open, high:r.high, low:r.low, close:r.close, vol:r.vol});
      } else {
        const b = buckets.get(k);
        b.high = Math.max(b.high, r.high);
        b.low  = Math.min(b.low,  r.low);
        b.close = r.close; // last close
        b.vol += r.vol;
      }
    }
    // buckets insertion order follows encounter order (rows assumed sorted asc)
    return Array.from(buckets.values());
  }

  async function loadTicker(ticker){
    if (cache.has(ticker)) return cache.get(ticker);
    const meta = COMPANY_MAP[ticker];
    if(!meta) throw new Error(`Ticker no soportado: ${ticker}`);
    const resp = await fetch(meta.file);
    if(!resp.ok) throw new Error(`No pude leer ${meta.file} (HTTP ${resp.status}).`);
    const text = await resp.text();

    const lines = text.trim().split(/\r?\n/);
    const out = [];
    for(let i=1;i<lines.length;i++){
      const parts = lines[i].split(',');
      if(parts.length < 9) continue;
      const date = parseYYYYMMDD(parts[2]);
      const open = +parts[4];
      const high = +parts[5];
      const low  = +parts[6];
      const close= +parts[7];
      const vol  = +parts[8];
      if(!Number.isFinite(close)) continue;
      out.push({date, open, high, low, close, vol});
    }
    out.sort((a,b)=>a.date-b.date);
    cache.set(ticker, out);
    return out;
  }

  function computeReturns(closes){
    const ret = new Array(closes.length).fill(null);
    for(let i=1;i<closes.length;i++){
      const prev = closes[i-1];
      const cur = closes[i];
      if(prev && cur) ret[i] = (cur/prev) - 1;
    }
    return ret;
  }

  function setActiveRangeButton(range){
    document.querySelectorAll('.seg button').forEach(b=>{
      b.classList.toggle('active', b.dataset.range === range);
    })
  }

  function fillDatalist(){
    const dl = document.getElementById('companyList');
    dl.innerHTML = '';
    for(const [ticker, meta] of Object.entries(COMPANY_MAP)){
      const opt = document.createElement('option');
      opt.value = `${meta.name} (${ticker.replace('.US','')})`;
      opt.dataset.ticker = ticker;
      dl.appendChild(opt);

      // also add raw ticker
      const opt2 = document.createElement('option');
      opt2.value = ticker.replace('.US','');
      opt2.dataset.ticker = ticker;
      dl.appendChild(opt2);
    }
  }

  function resolveTickerFromSearch(value){
    const v = (value||'').trim();
    if(!v) return null;
    // if user typed raw ticker
    const normalized = v.toUpperCase().replace(/\s/g,'');
    for(const t of Object.keys(COMPANY_MAP)){
      if(normalized === t.replace('.US','')) return t;
      if(normalized === t) return t;
    }
    // match by name
    for(const [t, meta] of Object.entries(COMPANY_MAP)){
      if(v.toLowerCase().includes(meta.name.toLowerCase())) return t;
    }
    // match by pattern "(TICK)"
    const m = v.match(/\(([^)]+)\)/);
    if(m){
      const cand = m[1].toUpperCase();
      for(const t of Object.keys(COMPANY_MAP)){
        if(cand === t.replace('.US','')) return t;
      }
    }
    return null;
  }

  function updateStats(meta, rowsShown, rowsAll){
    const stName = document.getElementById('stName');
    const stTicker= document.getElementById('stTicker');
    const stClose = document.getElementById('stClose');
    const stChg   = document.getElementById('stChg');
    const stVol   = document.getElementById('stVol');
    const stAvgVol= document.getElementById('stAvgVol');

    stName.textContent = meta.name;
    stTicker.textContent = meta.file.replace('data/','') + ` · ${rowsAll.length.toLocaleString()} filas`;

    const last = rowsAll[rowsAll.length-1];
    const prev = rowsAll[rowsAll.length-2];
    if(last && prev){
      const chg = (last.close/prev.close - 1) * 100;
      stClose.textContent = fmtNum(last.close);
      stChg.textContent = `${chg>=0?'+':''}${fmtNum(chg,2)}% · ${fmtDate(last.date)} (UTC)`;
      stChg.style.color = chg>=0 ? 'var(--good)' : 'var(--danger)';
    } else {
      stClose.textContent = '-';
      stChg.textContent = '-';
    }

    // rolling vol & avg volume on daily data (last point)
    const closes = rowsAll.map(r=>r.close);
    const rets = computeReturns(closes).map(x=>x ?? 0);
    const vol20 = rollingStd(rets, 20);
    const vavg20 = rollingMean(rowsAll.map(r=>r.vol), 20);
    const lastVol = vol20[vol20.length-1];
    const lastVavg = vavg20[vavg20.length-1];
    stVol.textContent = lastVol ? fmtNum(lastVol*100,2) + '%' : '-';
    stAvgVol.textContent = lastVavg ? fmtInt(lastVavg) : '-';
  }

  async function render(){
    const meta = COMPANY_MAP[currentTicker];
    const allRows = await loadTicker(currentTicker);

    const preset = RANGE_PRESETS[currentRange];
    let rows = sliceByWindow(allRows, preset.windowDays);
    rows = resampleOHLCV(rows, preset.mode);

    // Build series
    const x = rows.map(r=>r.date);
    const open = rows.map(r=>r.open);
    const high = rows.map(r=>r.high);
    const low  = rows.map(r=>r.low);
    const close= rows.map(r=>r.close);
    const vol  = rows.map(r=>r.vol);

    const showMA20 = document.getElementById('ma20').checked;
    const showMA50 = document.getElementById('ma50').checked;
    const showVMA20= document.getElementById('vma20').checked;

    const traces = [];

    // Candlestick
    traces.push({
      type: 'candlestick',
      x, open, high, low, close,
      name: 'OHLC',
      yaxis: 'y',
      increasing: { line: { width: 1 } },
      decreasing: { line: { width: 1 } },
      hovertemplate:
        '%{x|%Y-%m-%d}<br>'+
        'O: %{open:.4g}  H: %{high:.4g}<br>'+
        'L: %{low:.4g}  C: %{close:.4g}<extra></extra>'
    });

    // Moving averages
    if(showMA20){
      const ma = rollingMean(close, 20);
      traces.push({
        type:'scatter', mode:'lines', x, y: ma,
        name:'MA(20)', yaxis:'y',
        line:{width:2},
        hovertemplate:'%{x|%Y-%m-%d}<br>MA20: %{y:.4g}<extra></extra>'
      });
    }
    if(showMA50){
      const ma = rollingMean(close, 50);
      traces.push({
        type:'scatter', mode:'lines', x, y: ma,
        name:'MA(50)', yaxis:'y',
        line:{width:2, dash:'dot'},
        hovertemplate:'%{x|%Y-%m-%d}<br>MA50: %{y:.4g}<extra></extra>'
      });
    }

    // Volume bars
    traces.push({
      type:'bar', x, y: vol,
      name:'Volumen', yaxis:'y2',
      opacity:0.75,
      hovertemplate:'%{x|%Y-%m-%d}<br>Vol: %{y:,}<extra></extra>'
    });

    // Volume average
    if(showVMA20){
      const vma = rollingMean(vol, 20);
      traces.push({
        type:'scatter', mode:'lines', x, y: vma,
        name:'Vol. prom(20)', yaxis:'y2',
        line:{width:2, dash:'dash'},
        hovertemplate:'%{x|%Y-%m-%d}<br>VMA20: %{y:,}<extra></extra>'
      });
    }

    const title = `${meta.name} · ${currentTicker.replace('.US','')} · ${currentRange}`;

    const layout = {
      paper_bgcolor:'rgba(0,0,0,0)',
      plot_bgcolor:'rgba(0,0,0,0)',
      margin:{l:50,r:18,t:40,b:45},
      title:{text:title, font:{size:14,color:'#e8eefc'}},
      xaxis:{
        domain:[0,1],
        showgrid:true,
        gridcolor:'rgba(35,48,90,.55)',
        zeroline:false,
        rangeslider:{visible:false},
        tickformat: preset.mode==='yearly' ? '%Y' : (preset.mode==='monthly' ? '%Y-%m' : '%Y-%m-%d'),
      },
      yaxis:{
        domain:[0.32, 1.0],
        showgrid:true,
        gridcolor:'rgba(35,48,90,.55)',
        zeroline:false,
        tickfont:{color:'#e8eefc'},
        title:{text:'Precio', font:{size:11,color:'#a9b7e1'}}
      },
      yaxis2:{
        domain:[0.0, 0.24],
        showgrid:true,
        gridcolor:'rgba(35,48,90,.35)',
        zeroline:false,
        tickfont:{color:'#e8eefc'},
        title:{text:'Volumen', font:{size:11,color:'#a9b7e1'}}
      },
      legend:{orientation:'h', y:1.08, x:0, font:{size:11,color:'#a9b7e1'}},
      hovermode:'x unified',
    };

    const config = {
      responsive:true,
      displaylogo:false,
      modeBarButtonsToRemove:['select2d','lasso2d'],
    };

    await Plotly.newPlot('chart', traces, layout, config);
    updateStats(meta, rows, allRows);
  }

  // === Wire UI ===
  function bind(){
    fillDatalist();

    // default search field
    document.getElementById('companySearch').value = `${COMPANY_MAP[currentTicker].name} (${currentTicker.replace('.US','')})`;

    document.querySelectorAll('.seg button').forEach(btn=>{
      btn.addEventListener('click', async ()=>{
        currentRange = btn.dataset.range;
        setActiveRangeButton(currentRange);
        await render();
      })
    });

    ['ma20','ma50','vma20'].forEach(id=>{
      document.getElementById(id).addEventListener('change', render);
    });

    const search = document.getElementById('companySearch');
    search.addEventListener('change', async ()=>{
      const t = resolveTickerFromSearch(search.value);
      if(t){
        currentTicker = t;
        // keep pretty label
        search.value = `${COMPANY_MAP[t].name} (${t.replace('.US','')})`;
        await render();
      }
    });

    // Enter key triggers change
    search.addEventListener('keydown', (e)=>{
      if(e.key === 'Enter') search.dispatchEvent(new Event('change'));
    })

    window.addEventListener('resize', ()=>Plotly.Plots.resize('chart'));
  }

  (async function init(){
    bind();
    try{
      await render();
    } catch (err){
      console.error(err);
      document.getElementById('chart').innerHTML = `<div style="padding:14px; color:#ffb3b3;">Error: ${err.message}<br><br>Probá correr el servidor local: <code>python -m http.server 8000</code></div>`;
    }
  })();
</script>
</body>
</html>
