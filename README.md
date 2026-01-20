<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Einkaufsmanager</title>

<style>
:root{
  --green:#007d3c;--green-light:#2ecc71;--bg:#f4f6f7;--card:#fff;--border:#ddd;--text:#222;--danger:#c0392b;
}
*{box-sizing:border-box;margin:0;padding:0;font-family:system-ui,-apple-system,BlinkMacSystemFont,sans-serif}
body{background:var(--bg);color:var(--text)}
header{background:var(--green);color:#fff;padding:12px 16px;display:flex;justify-content:space-between;align-items:center}
nav{display:flex;gap:8px;align-items:center}
nav button{background:rgba(255,255,255,.2);border:none;color:#fff;padding:8px 12px;border-radius:8px;font-weight:700;cursor:pointer}
nav button.active{background:#fff;color:var(--green)}
.cart{position:relative;font-size:1.3rem;cursor:pointer}
.cart span{position:absolute;top:-6px;right:-10px;background:#ffcc00;color:#000;padding:2px 6px;border-radius:12px;font-weight:800;font-size:.75rem}
.status{padding:8px 16px;font-size:.9rem;color:#444}

main{max-width:1100px;margin:12px auto;padding:0 12px}
.view{display:none}
.view.active{display:block}

/* SHOP */
.toolbar{margin-bottom:10px}
.search{width:100%;padding:10px;border-radius:10px;border:1px solid var(--border);font-size:1rem}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(210px,1fr));gap:12px}
.card{background:var(--card);border:1px solid var(--border);border-radius:12px;padding:12px;display:flex;flex-direction:column;position:relative}
.card.selected{outline:3px solid var(--green-light)}
.card img{width:100%;height:120px;object-fit:contain;background:#eee;border-radius:6px;margin-bottom:8px}
.badge{position:absolute;top:10px;right:10px;background:var(--green-light);color:#fff;padding:4px 8px;border-radius:14px;font-weight:800;font-size:.8rem}
.controls{display:flex;align-items:center;justify-content:center;gap:8px;margin-top:auto}
.controls button{width:36px;height:36px;border-radius:50%;border:none;background:var(--green);color:#fff;font-weight:800;font-size:1.1rem;cursor:pointer}
.controls span{min-width:28px;text-align:center;font-weight:800}

/* LISTEN */
.listbar{background:#fff;border:1px solid var(--border);border-radius:10px;padding:10px;display:flex;gap:8px;flex-wrap:wrap;align-items:center;margin-bottom:10px}
.listbar select{padding:8px;border-radius:8px;border:1px solid var(--border)}
.btn{padding:8px 10px;border-radius:8px;border:none;font-weight:800;cursor:pointer}
.btn.new{background:var(--green);color:#fff}
.btn.rename{background:#2980b9;color:#fff}
.btn.del{background:var(--danger);color:#fff}
.list{list-style:none}
.list li{display:flex;justify-content:space-between;align-items:center;border-bottom:1px solid var(--border);padding:10px 6px}
.small{width:32px;height:32px;border-radius:50%;border:none;background:var(--green);color:#fff;font-weight:800;cursor:pointer}

/* COSTS */
table{width:100%;border-collapse:collapse}
th,td{padding:8px;border-bottom:1px solid var(--border);text-align:left}
.total{font-weight:900;margin-top:8px}

/* PRINT */
@media print{
  header,.toolbar,.grid,.listbar,nav{display:none}
  body{font-size:11px}
}
</style>
</head>

<body>

<header>
  <strong>Einkaufsmanager</strong>
  <nav>
    <button data-v="shop" class="active">Shop</button>
    <button data-v="list">Einkaufsliste</button>
    <button data-v="costs">Kosten</button>
    <div class="cart" onclick="gotoList()">🛒<span id="cc">0</span></div>
  </nav>
</header>

<div class="status">Aktive Liste: <strong id="al">Master Liste</strong></div>

<main>

<!-- SHOP -->
<section id="shop" class="view active">
  <div class="toolbar">
    <input class="search" id="q" placeholder="Produkt suchen (z. B. Käse, Milch, Brot)">
  </div>

  <!-- Fallback-HTML: Produkte & Bilder IMMER sichtbar (ohne JS) -->
  <div class="grid" id="grid">
    <div class="card">
      <img alt="Salami" src="data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='400' height='300'><rect width='100%' height='100%' fill='%23eee'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-size='28'>Salami</text></svg>">
      <strong>Salami</strong>
      <div class="controls"><button>−</button><span>0</span><button>+</button></div>
    </div>
    <div class="card">
      <img alt="Käse" src="data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='400' height='300'><rect width='100%' height='100%' fill='%23eee'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-size='28'>Käse</text></svg>">
      <strong>Käse</strong>
      <div class="controls"><button>−</button><span>0</span><button>+</button></div>
    </div>
    <div class="card">
      <img alt="Milch" src="data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='400' height='300'><rect width='100%' height='100%' fill='%23eee'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-size='28'>Milch</text></svg>">
      <strong>Milch</strong>
      <div class="controls"><button>−</button><span>0</span><button>+</button></div>
    </div>
    <div class="card">
      <img alt="Brot" src="data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='400' height='300'><rect width='100%' height='100%' fill='%23eee'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-size='28'>Brot</text></svg>">
      <strong>Brot</strong>
      <div class="controls"><button>−</button><span>0</span><button>+</button></div>
    </div>
  </div>
</section>

<!-- LIST -->
<section id="list" class="view">
  <div class="listbar">
    <select id="ls"></select>
    <button class="btn new" onclick="nl()">➕ Neue Liste</button>
    <button class="btn rename" onclick="rl()">✏️ Umbenennen</button>
    <button class="btn del" onclick="dl()">🗑️ Löschen</button>
    <button class="btn" onclick="window.print()">🖨️ Drucken</button>
  </div>
  <ul class="list" id="ul"></ul>
</section>

<!-- COSTS -->
<section id="costs" class="view">
  <table>
    <thead><tr><th>Produkt</th><th>Menge</th><th>Preis</th><th>Summe</th></tr></thead>
    <tbody id="cb"></tbody>
  </table>
  <div class="total" id="ct"></div>
</section>

</main>

<script>
/* ====== ROBUSTE ENHANCEMENT-LOGIK (kann NICHT alles lahmlegen) ====== */
try{
  const PRODUCTS=[
    {id:1,n:"Salami",p:2.49,i:"data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='400' height='300'><rect width='100%' height='100%' fill='%23eee'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-size='28'>Salami</text></svg>"},
    {id:2,n:"Käse",p:1.99,i:"data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='400' height='300'><rect width='100%' height='100%' fill='%23eee'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-size='28'>Käse</text></svg>"},
    {id:3,n:"Milch",p:1.49,i:"data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='400' height='300'><rect width='100%' height='100%' fill='%23eee'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-size='28'>Milch</text></svg>"},
    {id:4,n:"Brot",p:2.29,i:"data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='400' height='300'><rect width='100%' height='100%' fill='%23eee'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-size='28'>Brot</text></svg>"}
  ];

  let S=JSON.parse(localStorage.getItem("S"))||{a:"Master Liste",l:{ "Master Liste":[] }};
  const $=q=>document.querySelector(q);
  const $$=q=>[...document.querySelectorAll(q)];

  function save(){localStorage.setItem("S",JSON.stringify(S));ui()}
  function ui(){
    $("#al").textContent=S.a;
    $("#ls").innerHTML=Object.keys(S.l).map(n=>`<option ${n===S.a?"selected":""}>${n}</option>`).join("");
    render(PRODUCTS);
    list();
    costs();
    $("#cc").textContent=S.l[S.a].reduce((s,x)=>s+x.q,0);
  }

  function render(arr){
    const g=$("#grid"); g.innerHTML="";
    const cur=S.l[S.a];
    arr.forEach(p=>{
      const f=cur.find(x=>x.id===p.id);
      g.insertAdjacentHTML("beforeend",`
        <div class="card ${f?"selected":""}">
          ${f?`<div class="badge">x${f.q}</div>`:""}
          <img src="${p.i}" alt="${p.n}">
          <strong>${p.n}</strong>
          <div class="controls">
            <button onclick="chg(${p.id},-1)">−</button>
            <span>${f?f.q:0}</span>
            <button onclick="chg(${p.id},1)">+</button>
          </div>
        </div>`);
    });
  }

  window.chg=(id,d)=>{
    const a=S.l[S.a];
    const f=a.find(x=>x.id===id);
    if(!f && d>0){ const p=PRODUCTS.find(x=>x.id===id); a.push({id,p:p.p,n:p.n,q:1}); }
    else if(f){ f.q+=d; if(f.q<=0) S.l[S.a]=a.filter(x=>x.id!==id); }
    save();
  };

  function list(){
    const u=$("#ul"); u.innerHTML="";
    S.l[S.a].forEach(x=>{
      u.insertAdjacentHTML("beforeend",`
        <li>
          <span>${x.n}</span>
          <span>
            <button class="small" onclick="chg(${x.id},-1)">−</button>
            ${x.q}
            <button class="small" onclick="chg(${x.id},1)">+</button>
          </span>
        </li>`);
    });
  }

  function costs(){
    let t=0, b=$("#cb"); b.innerHTML="";
    S.l[S.a].forEach(x=>{
      const s=x.q*x.p; t+=s;
      b.insertAdjacentHTML("beforeend",`<tr><td>${x.n}</td><td>${x.q}</td><td>${x.p.toFixed(2)} €</td><td>${s.toFixed(2)} €</td></tr>`);
    });
    $("#ct").textContent=t?`Gesamt: ${t.toFixed(2)} €`:"";
  }

  window.nl=()=>{const n=prompt("Name der neuen Liste"); if(n&&!S.l[n]){S.l[n]=[];S.a=n;save();}};
  window.rl=()=>{const n=prompt("Neuer Name",S.a); if(n&&!S.l[n]){S.l[n]=S.l[S.a]; delete S.l[S.a]; S.a=n;save();}};
  window.dl=()=>{ if(S.a==="Master Liste") return alert("Master Liste kann nicht gelöscht werden"); if(confirm("Liste löschen?")){ delete S.l[S.a]; S.a="Master Liste"; save(); }};

  $("#q").addEventListener("input",e=>{
    const q=e.target.value.toLowerCase();
    render(PRODUCTS.filter(p=>p.n.toLowerCase().includes(q)));
  });

  $$("nav button").forEach(b=>b.onclick=()=>{ $$("nav button").forEach(x=>x.classList.remove("active")); b.classList.add("active"); $$(".view").forEach(v=>v.classList.remove("active")); $("#"+b.dataset.v).classList.add("active"); if(b.dataset.v==="list") list(); if(b.dataset.v==="costs") costs(); });
  window.gotoList=()=>document.querySelector('[data-v="list"]').click();

  ui();
}catch(e){
  /* bewusst leer: Fallback-HTML bleibt sichtbar */
}
</script>

</body>
</html>
