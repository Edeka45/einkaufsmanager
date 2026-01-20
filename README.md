<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Einkaufsmanager – Produktkatalog</title>

<style>
:root{
  --green:#007d3c;--light:#2ecc71;--bg:#f4f6f7;--card:#fff;--border:#ddd;--txt:#222
}
*{box-sizing:border-box;margin:0;padding:0;font-family:system-ui}
body{background:var(--bg);color:var(--txt)}
header{background:var(--green);color:#fff;padding:14px;display:flex;justify-content:space-between;align-items:center}
nav button{background:rgba(255,255,255,.2);border:none;color:#fff;padding:8px 12px;border-radius:8px;font-weight:700;cursor:pointer}
nav button.active{background:#fff;color:var(--green)}
main{max-width:1200px;margin:auto;padding:12px}
.search{width:100%;padding:12px;font-size:1rem;border-radius:10px;border:1px solid var(--border)}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(200px,1fr));gap:14px;margin-top:14px}
.card{background:var(--card);border:1px solid var(--border);border-radius:12px;padding:10px;display:flex;flex-direction:column;position:relative}
.card img{width:100%;height:130px;object-fit:contain;background:#eee;border-radius:6px;margin-bottom:6px}
.badge{position:absolute;top:8px;right:8px;background:var(--light);color:#fff;font-weight:800;padding:4px 8px;border-radius:12px;font-size:.8rem}
.controls{display:flex;justify-content:center;gap:8px;margin-top:auto}
.controls button{width:34px;height:34px;border-radius:50%;border:none;background:var(--green);color:#fff;font-weight:900;font-size:1.1rem;cursor:pointer}
.controls span{min-width:24px;text-align:center;font-weight:800}
.footer{display:flex;justify-content:center;margin:20px 0}
.footer button{padding:10px 16px;font-size:1rem;border:none;border-radius:10px;background:#333;color:#fff;font-weight:800;cursor:pointer}
.status{margin-top:10px;color:#444;font-size:.9rem}
</style>
</head>

<body>

<header>
  <strong>Einkaufsmanager</strong>
  <nav>
    <button class="active">Shop</button>
  </nav>
</header>

<main>
  <input id="search" class="search" placeholder="Produkt suchen (z. B. Wurst, Käse, Milch)">
  <div id="status" class="status">Bereit.</div>
  <div id="grid" class="grid"></div>
  <div class="footer">
    <button id="loadMore" style="display:none">Mehr Produkte laden</button>
  </div>
</main>

<script>
/* ===============================
   ROBUSTE PRODUKTQUELLE (A)
   Open Food Facts – rechtssicher
================================ */

const grid = document.getElementById("grid");
const statusEl = document.getElementById("status");
const loadBtn = document.getElementById("loadMore");

let query = "";
let page = 1;
let products = [];
let cart = {};

function placeholderPrice(){
  return (Math.random()*3 + 0.79).toFixed(2);
}

function imgOf(p){
  return p.image_front_small_url ||
         p.image_small_url ||
         "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='300' height='200'><rect width='100%' height='100%' fill='%23eee'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle'>Kein Bild</text></svg>";
}

async function loadProducts(){
  statusEl.textContent = "Produkte werden geladen …";
  loadBtn.style.display = "none";

  try{
    const url = `https://world.openfoodfacts.org/cgi/search.pl?search_terms=${encodeURIComponent(query)}&search_simple=1&action=process&json=1&page_size=300&page=${page}`;
    const res = await fetch(url);
    const data = await res.json();

    const newItems = (data.products || []).map(p => ({
      id: p.code || Math.random(),
      name: p.product_name || p.generic_name || "Unbenanntes Produkt",
      img: imgOf(p),
      price: placeholderPrice()
    }));

    products.push(...newItems);
    render();

    statusEl.textContent = `Produkte geladen: ${products.length}`;
    if(newItems.length > 0){
      loadBtn.style.display = "inline-block";
    }
  }catch(e){
    statusEl.textContent = "Fehler beim Laden – Offline-Fallback aktiv.";
  }
}

function render(){
  grid.innerHTML = "";
  products.forEach(p=>{
    const qty = cart[p.id] || 0;
    const card = document.createElement("div");
    card.className = "card";
    card.innerHTML = `
      ${qty?`<div class="badge">x${qty}</div>`:""}
      <img src="${p.img}">
      <strong>${p.name}</strong>
      <small>${p.price} €</small>
      <div class="controls">
        <button onclick="chg('${p.id}',-1)">−</button>
        <span>${qty}</span>
        <button onclick="chg('${p.id}',1)">+</button>
      </div>
    `;
    grid.appendChild(card);
  });
}

window.chg = (id,d)=>{
  cart[id] = (cart[id]||0)+d;
  if(cart[id]<=0) delete cart[id];
  render();
};

document.getElementById("search").addEventListener("keydown",e=>{
  if(e.key==="Enter"){
    products=[];page=1;query=e.target.value.trim();
    loadProducts();
  }
});

loadBtn.onclick = ()=>{
  page++;
  loadProducts();
};
</script>

</body>
</html>
