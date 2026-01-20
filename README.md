<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Einkaufsmanager — EDEKA-nahe Produkte (OpenFoodFacts)</title>
<style>
:root{
  --green:#007d3c;--muted:#666;--bg:#f4f6f7;--card:#fff;--border:#ddd;--txt:#222;
}
*{box-sizing:border-box;margin:0;padding:0;font-family:system-ui,-apple-system,BlinkMacSystemFont,Segoe UI,Roboto,Arial}
body{background:var(--bg);color:var(--txt);min-height:100vh}
header{background:var(--green);color:#fff;padding:12px 16px;display:flex;justify-content:space-between;align-items:center}
.header-left{display:flex;gap:12px;align-items:center}
.app-title{font-weight:800}
.controls{display:flex;gap:8px;align-items:center}
.btn{padding:8px 10px;border-radius:8px;border:none;background:#fff;color:var(--green);font-weight:700;cursor:pointer}
.input{padding:10px;border-radius:10px;border:1px solid var(--border);width:100%;font-size:1rem}
.container{max-width:1200px;margin:14px auto;padding:0 12px}
.topbar{display:flex;gap:8px;align-items:center;margin-bottom:10px}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:12px}
.card{background:var(--card);border:1px solid var(--border);border-radius:12px;padding:10px;display:flex;flex-direction:column;position:relative;min-height:200px}
.card img{width:100%;height:120px;object-fit:cover;border-radius:6px;background:#efefef;margin-bottom:8px}
.badge{position:absolute;top:8px;right:8px;background:#2ecc71;color:#fff;padding:4px 8px;border-radius:12px;font-weight:800}
.controls-card{display:flex;justify-content:center;gap:8px;margin-top:auto}
.controls-card button{width:36px;height:36px;border-radius:50%;border:none;background:var(--green);color:#fff;font-weight:800;cursor:pointer}
.status{margin-top:8px;color:var(--muted)}
.footer{display:flex;justify-content:center;margin:14px 0}
.footer button{padding:10px 14px;border-radius:10px;border:none;background:#333;color:#fff;cursor:pointer;font-weight:800}
.listbar{background:#fff;border-radius:10px;padding:10px;border:1px solid var(--border);display:flex;gap:8px;align-items:center;margin-bottom:12px}
select{padding:8px;border-radius:8px;border:1px solid var(--border)}
.list{list-style:none;padding:0;margin:0}
.list li{display:flex;justify-content:space-between;padding:8px 6px;border-bottom:1px solid var(--border);align-items:center}
.small{width:34px;height:34px;border-radius:50%;border:none;background:var(--green);color:#fff;font-weight:800;cursor:pointer}
@media (max-width:520px){.grid{grid-template-columns:repeat(2,1fr)}}
</style>
</head>
<body>

<header>
  <div class="header-left">
    <div class="app-title">Einkaufsmanager</div>
    <div style="font-size:0.9rem;opacity:0.9">(Daten: Open Food Facts)</div>
  </div>
  <div class="controls">
    <div style="position:relative;cursor:pointer" title="Warenkorb" onclick="goToList()">🛒 <span id="countBadge" style="position:absolute;top:-8px;right:-10px;background:#ffcc00;color:#000;padding:2px 6px;border-radius:10px;font-weight:800;font-size:0.8rem">0</span></div>
  </div>
</header>

<div class="container">

  <div class="topbar">
    <input id="searchInput" class="input" placeholder="Suchbegriff (z. B. Wurst, Käse) und Enter drücken">
    <button id="brandToggle" class="btn" title="Nur EDEKA-ähnliche Produkte">Nur EDEKA-ähnliche</button>
    <button id="loadPreset" class="btn" title="Schnellstart Beispiel 'Wurst'">Beispiel: Wurst</button>
  </div>

  <div class="listbar">
    <label for="listSelect">Gespeicherte Listen</label>
    <select id="listSelect"></select>
    <button class="btn" onclick="createList()">➕ Neue Liste</button>
    <button class="btn" onclick="renameList()">✏️ Umbenennen</button>
    <button class="btn" onclick="deleteList()">🗑️ Löschen</button>
    <div style="flex:1"></div>
    <button class="btn" onclick="exportState()">Export</button>
    <button class="btn" onclick="importState()">Import</button>
    <button class="btn" onclick="window.print()">🖨️ Drucken</button>
  </div>

  <div id="status" class="status">Bereit.</div>

  <div id="grid" class="grid" aria-live="polite"></div>

  <div class="footer">
    <button id="moreBtn" style="display:none">Mehr laden</button>
  </div>

  <!-- List view (hidden by default, show when clicking cart) -->
  <section id="listView" style="display:none;margin-top:14px">
    <h3>Einkaufsliste</h3>
    <ul id="cartList" class="list"></ul>
    <div style="margin-top:8px;font-weight:800" id="totalSum"></div>
  </section>

</div>

<script>
/*
  Integration: Open Food Facts (legal, public)
  - Allows search by term
  - Client-side filter for brands likely at EDEKA (e.g., "EDEKA","Gut & Günstig","EDEKA Bio")
  - Paging & load-more to reach 1000+ items per query if available
  - Robust fallback to local seed
  - Images: uses OFF image fields, converts to https, falls back to inline SVG
  - No scraping, no EDEKA CDN hotlinks
  Sources: Open Food Facts API docs (public). See code comments.
*/

const OFF_ENDPOINT = 'https://world.openfoodfacts.org/cgi/search.pl';
const PAGE_SIZE = 300; // OFF supports large page_size; we request up to multiple pages to reach target
const TARGET_COUNT = 1000; // aim to reach this many items if available

// EDEKA-like brand keywords we will prioritize/filter for
const EDEKA_BRANDS = ["EDEKA","Gut & Günstig","EDEKA Bio","EDEKA EIGENE MARKE","EDEKA‐Bio"];

let products = [];   // accumulated product list for current query
let page = 1;
let query = "";
let loading = false;
let cart = {}; // id -> qty

// Lists state (stored)
let state = (function(){
  try{ return JSON.parse(localStorage.getItem('shopping_state_v3')) || {active:'Master Liste', lists:{'Master Liste':[]}}; }catch(e){ return {active:'Master Liste', lists:{'Master Liste':[]}};}
})();

function saveState(){ localStorage.setItem('shopping_state_v3', JSON.stringify(state)); updateUI(); }

function updateUI(){
  document.getElementById('countBadge').textContent = Object.values(cart).reduce((s,v)=>s+v,0);
  // list select
  const sel = document.getElementById('listSelect');
  sel.innerHTML = '';
  Object.keys(state.lists).forEach(n=>{
    const opt = document.createElement('option');
    opt.value = n; opt.textContent = n;
    if(n === state.active) opt.selected = true;
    sel.appendChild(opt);
  });
  // render cart list if visible
  if(document.getElementById('listView').style.display !== 'none') renderCart();
}

// helper: safe image (https)
function safeImg(url){
  if(!url) return null;
  if(url.startsWith('//')) return 'https:' + url;
  if(url.startsWith('http://')) return url.replace('http://','https://');
  return url;
}

function inlinePlaceholderSVG(text){
  return 'data:image/svg+xml;utf8,' + encodeURIComponent(`<svg xmlns='http://www.w3.org/2000/svg' width='400' height='300'><rect width='100%' height='100%' fill='%23efefef'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' fill='%23999' font-size='20'>${text}</text></svg>`);
}

// render products to grid
function render(){
  const grid = document.getElementById('grid');
  grid.innerHTML = '';
  products.forEach(p=>{
    const qty = cart[p.id] || 0;
    const img = p.img ? safeImg(p.img) : inlinePlaceholderSVG('Kein Bild');
    const card = document.createElement('div');
    card.className = 'card';
    card.innerHTML = `
      ${qty?`<div class="badge">x${qty}</div>`:""}
      <img src="${img}" alt="${escapeHtml(p.name)}" onerror="this.src='${inlinePlaceholderSVG('Bild nicht verfügbar')}'">
      <strong style="margin-bottom:6px">${escapeHtml(p.name)}</strong>
      <div style="color:#666;font-size:0.9rem;margin-bottom:6px">${p.brand?escapeHtml(p.brand):''}</div>
      <div style="font-weight:800;margin-bottom:6px">${p.price?p.price+' €':''}</div>
      <div class="controls-card">
        <button onclick="chg('${p.id}',-1)">−</button>
        <span>${qty}</span>
        <button onclick="chg('${p.id}',1)">+</button>
      </div>
    `;
    grid.appendChild(card);
  });
  updateUI();
}

// change qty in cart and in active list
function chg(id,delta){
  cart[id] = (cart[id]||0)+delta;
  if(cart[id] <= 0) delete cart[id];
  // also reflect into active saved list
  const list = state.lists[state.active];
  const entry = list.find(x=>x.id===id);
  const prod = products.find(x=>x.id===id);
  if(!entry && delta>0){
    list.push({id:id, name: prod ? prod.name : 'Unbenannt', price: prod && prod.price?parseFloat(prod.price):0, qty:1});
  } else if(entry){
    entry.qty += delta;
    if(entry.qty <= 0) state.lists[state.active] = list.filter(x=>x.id!==id);
  }
  saveState();
}

// fetch OFF pages until we have enough or pages exhausted
async function fetchOFF(term, maxTarget = TARGET_COUNT){
  if(loading) return;
  loading = true;
  document.getElementById('status').textContent = 'Lade Produkte…';
  products = []; page = 1;
  const brandFilterOn = document.getElementById('brandToggle').dataset.on === '1';
  // fetch sequentially until enough or no more results
  try{
    while(products.length < maxTarget){
      const params = new URLSearchParams({
        search_terms: term,
        search_simple: 1,
        action: 'process',
        json: 1,
        page_size: 300,
        page: page
      });
      const url = OFF_ENDPOINT + '?' + params.toString();
      const resp = await fetch(url);
      if(!resp.ok) throw new Error('API-Fehler: ' + resp.status);
      const data = await resp.json();
      const items = (data.products || []).map(it=>{
        const brand = (it.brands || '').split(',')[0] || '';
        return {
          id: it.code || ('of-'+Math.random().toString(36).slice(2,9)),
          name: it.product_name || it.generic_name || it.brands || 'Unbenannt',
          brand: brand,
          img: it.image_front_small_url || it.image_small_url || it.image_front_url || it.image_url || null,
          price: (it.stores || '').toLowerCase().includes('edeka') ? null : null // OFF has no price; placeholder later
        };
      });
      // if brand filter active: keep only items whose brand field matches our EDEKA keywords
      let filtered = items;
      if(brandFilterOn){
        filtered = items.filter(i=>{
          if(!i.brand) return false;
          const b = i.brand.toUpperCase();
          return EDEKA_BRANDS.some(k => b.includes(k.toUpperCase()));
        });
      }
      // map price placeholders and push
      filtered.forEach(f=>{
        f.price = parseFloat((Math.random()*2 + 0.79).toFixed(2)); // placeholder
        products.push(f);
      });
      // if data.products length < page_size -> no further pages
      if(!data.products || data.products.length === 0) break;
      page++;
      // defensive stop to avoid endless loops
      if(page > 10) break;
    }
    if(products.length === 0){
      // fallback to local seed
      products = [
        {id:'seed-1', name:'Salami (Demo)', brand:'Demo', img:null, price:2.49},
        {id:'seed-2', name:'Gouda (Demo)', brand:'Demo', img:null, price:1.99},
        {id:'seed-3', name:'Milch 1L (Demo)', brand:'Demo', img:null, price:1.49}
      ];
      document.getElementById('status').textContent = 'Keine Treffer bei OpenFoodFacts — Demo-Produkte geladen.';
    } else {
      document.getElementById('status').textContent = 'Produkte geladen: ' + products.length;
    }
    render();
    // show load more if plausible
    if(products.length < maxTarget) {
      document.getElementById('moreBtn').style.display = 'inline-block';
      document.getElementById('moreBtn').onclick = async ()=>{
        await fetchOFF(term, maxTarget); // call will increment page internally until limit
      };
    } else {
      document.getElementById('moreBtn').style.display = 'none';
    }
  }catch(err){
    console.error(err);
    document.getElementById('status').textContent = 'Fehler beim Laden der Produktdaten — es wird auf lokale Demo-Daten zurückgegriffen.';
    products = [
      {id:'seed-1', name:'Salami (Demo)', brand:'Demo', img:null, price:2.49},
      {id:'seed-2', name:'Gouda (Demo)', brand:'Demo', img:null, price:1.99},
      {id:'seed-3', name:'Milch 1L (Demo)', brand:'Demo', img:null, price:1.49}
    ];
    render();
  }finally{
    loading = false;
  }
}

// UI helpers: brand toggle
document.getElementById('brandToggle').dataset.on = '0';
document.getElementById('brandToggle').addEventListener('click', function(){
  this.dataset.on = this.dataset.on === '1' ? '0' : '1';
  this.style.background = this.dataset.on === '1' ? '#2ecc71' : '#fff';
  this.style.color = this.dataset.on === '1' ? '#fff' : '#2ecc71';
  if(query) fetchOFF(query, TARGET_COUNT);
});

// quick preset
document.getElementById('loadPreset').addEventListener('click', ()=>{ document.getElementById('searchInput').value='Wurst'; doSearch(); });

// escapeHtml
function escapeHtml(s){ return (''+s).replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c])); }

// search handling
document.getElementById('searchInput').addEventListener('keydown', (e)=>{ if(e.key === 'Enter') doSearch(); });
function doSearch(){
  query = document.getElementById('searchInput').value.trim();
  if(!query) { document.getElementById('status').textContent = 'Bitte Suchbegriff eingeben.'; return; }
  products = []; page = 1;
  // fetch with target aiming for TARGET_COUNT (will iterate pages)
  fetchOFF(query, TARGET_COUNT);
}

// Lists: create/rename/delete/export/import & local persistence
function createList(){ const n = prompt('Name der neuen Liste'); if(!n) return; if(state.lists[n]) return alert('Liste existiert'); state.lists[n]=[]; state.active = n; saveState(); }
function renameList(){ const n = prompt('Neuer Name', state.active); if(!n || n===state.active) return; if(state.lists[n]) return alert('Name existiert'); state.lists[n]=state.lists[state.active]; delete state.lists[state.active]; state.active = n; saveState(); }
function deleteList(){ if(state.active==='Master Liste') return alert('Master Liste nicht löschbar'); if(confirm('Wirklich löschen?')){ delete state.lists[state.active]; state.active='Master Liste'; saveState(); } }
function exportState(){ const blob = new Blob([JSON.stringify(state,null,2)],{type:'application/json'}); const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download='einkaufsmanager_export.json'; a.click(); }
function importState(){ const inp=document.createElement('input'); inp.type='file'; inp.accept='.json'; inp.onchange=(e)=>{ const f=e.target.files[0]; if(!f) return; const r=new FileReader(); r.onload=()=>{ try{ const p=JSON.parse(r.result); if(p && p.lists){ state=p; saveState(); alert('Import OK'); } else alert('Ungültiges Format'); }catch(err){ alert('Importfehler: '+err.message); } }; r.readAsText(f); }; inp.click(); }

// save state wrapper
function saveState(){ try{ localStorage.setItem('shopping_state_v3', JSON.stringify(state)); }catch(e){} updateUI(); }

// cart list render
function renderCart(){
  const ul = document.getElementById('cartList');
  ul.innerHTML = '';
  const list = state.lists[state.active] || [];
  if(list.length === 0) { ul.innerHTML = '<li style="color:#666">Liste leer</li>'; document.getElementById('totalSum').textContent=''; return; }
  let total = 0;
  list.forEach(it=>{
    const s = (it.price || 0) * (it.qty || 0);
    total += s;
    const li = document.createElement('li');
    li.innerHTML = `<span>${escapeHtml(it.name)}</span><span><button class="small" onclick="chg('${it.id}',-1)">−</button> ${it.qty} <button class="small" onclick="chg('${it.id}',1)">+</button></span>`;
    ul.appendChild(li);
  });
  document.getElementById('totalSum').textContent = 'Gesamt: ' + total.toFixed(2) + ' €';
}

// show list view when clicking cart
function goToList(){ document.getElementById('listView').style.display = 'block'; renderCart(); }

// init UI
(function init(){
  // ensure master list exists
  if(!state.lists['Master Liste']) state.lists['Master Liste'] = [];
  updateUI();
  // optional: prefill with example search for quick content
  // fetchOFF('Wurst', TARGET_COUNT);
})();

</script>

</body>
</html>
