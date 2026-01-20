<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Einkaufsmanager</title>

<style>
:root{
  --green:#007d3c;
  --green-light:#2ecc71;
  --bg:#f4f6f7;
  --card:#ffffff;
  --border:#dddddd;
  --text:#222222;
}

*{
  box-sizing:border-box;
  margin:0;
  padding:0;
  font-family:system-ui,-apple-system,BlinkMacSystemFont,sans-serif;
}

body{
  background:var(--bg);
  color:var(--text);
}

/* HEADER */
header{
  background:var(--green);
  color:#fff;
  padding:1rem;
  display:flex;
  justify-content:space-between;
  align-items:center;
}

nav{
  display:flex;
  align-items:center;
  gap:.5rem;
}

nav button{
  background:rgba(255,255,255,.2);
  border:none;
  color:#fff;
  padding:.6rem 1rem;
  border-radius:8px;
  font-weight:600;
  cursor:pointer;
}

nav button.active{
  background:#fff;
  color:var(--green);
}

.cart-icon{
  position:relative;
  font-size:1.4rem;
  cursor:pointer;
}

.cart-count{
  position:absolute;
  top:-6px;
  right:-10px;
  background:#ffcc00;
  color:#000;
  font-size:.7rem;
  font-weight:700;
  padding:2px 6px;
  border-radius:12px;
}

/* VIEWS */
.view{display:none;}
.view.active{display:block;}

.status{
  padding:.5rem 1rem;
  font-size:.85rem;
  color:#555;
}

/* SHOP */
.shop-toolbar{
  padding:1rem;
}

.shop-toolbar input{
  width:100%;
  padding:.7rem;
  font-size:1rem;
  border-radius:8px;
  border:1px solid var(--border);
}

.product-grid{
  display:grid;
  grid-template-columns:repeat(auto-fill,minmax(200px,1fr));
  gap:1rem;
  padding:1rem;
}

.product-card{
  background:var(--card);
  border:1px solid var(--border);
  border-radius:12px;
  padding:1rem;
  display:flex;
  flex-direction:column;
  position:relative;
}

.product-card.selected{
  outline:3px solid var(--green-light);
}

.product-card img{
  width:100%;
  height:140px;
  object-fit:contain;
  margin-bottom:.5rem;
}

.qty-badge{
  position:absolute;
  top:8px;
  right:8px;
  background:var(--green-light);
  color:#fff;
  font-size:.75rem;
  font-weight:700;
  padding:3px 7px;
  border-radius:12px;
}

.shop-controls{
  display:flex;
  justify-content:center;
  align-items:center;
  gap:.6rem;
  margin-top:auto;
}

.shop-controls button{
  width:36px;
  height:36px;
  border:none;
  border-radius:50%;
  background:var(--green);
  color:#fff;
  font-size:1.2rem;
  font-weight:700;
  cursor:pointer;
}

.shop-controls span{
  min-width:24px;
  text-align:center;
  font-weight:700;
}

/* LISTE */
.cart-list{
  list-style:none;
  padding:1rem;
}

.cart-list li{
  display:flex;
  justify-content:space-between;
  align-items:center;
  padding:.6rem 0;
  border-bottom:1px solid var(--border);
}

.cart-list button{
  width:32px;
  height:32px;
  border:none;
  border-radius:50%;
  background:var(--green);
  color:#fff;
  font-weight:700;
  cursor:pointer;
}

/* COSTS */
.cost-table{
  width:100%;
  border-collapse:collapse;
  margin:1rem;
}

.cost-table th,
.cost-table td{
  border-bottom:1px solid var(--border);
  padding:.6rem;
  text-align:left;
}

.cost-total{
  padding:1rem;
  font-weight:800;
}

@media print{
  header,nav,.shop-toolbar,.product-grid{display:none;}
  body{font-size:11px;}
}
</style>
</head>

<body>

<header>
  <h1>Einkaufsmanager</h1>
  <nav>
    <button data-view="shop" class="active">Shop</button>
    <button data-view="list">Einkaufsliste</button>
    <button data-view="costs">Kosten</button>
    <div class="cart-icon" onclick="goToList()">🛒
      <span class="cart-count" id="cartCount">0</span>
    </div>
  </nav>
</header>

<div class="status">
  Aktive Liste: <strong>Master Liste</strong>
</div>

<main>

<section id="shop-view" class="view active">
  <div class="shop-toolbar">
    <input type="search" placeholder="Produkt suchen…" oninput="searchProducts(this.value)">
  </div>
  <div class="product-grid" id="productGrid"></div>
</section>

<section id="list-view" class="view">
  <ul class="cart-list" id="cartList"></ul>
  <div style="padding:1rem">
    <button onclick="window.print()">🖨️ Drucken</button>
  </div>
</section>

<section id="costs-view" class="view">
  <table class="cost-table">
    <thead>
      <tr><th>Produkt</th><th>Menge</th><th>Preis</th><th>Summe</th></tr>
    </thead>
    <tbody id="costBody"></tbody>
  </table>
  <div class="cost-total" id="costTotal"></div>
</section>

</main>

<script>
/* PRODUKTE */
const products = [
  {id:1,name:"EDEKA Bio Salami",price:2.49,img:"https://cdn.produkte.edeka/4311501758397.jpg"},
  {id:2,name:"EDEKA Gouda Scheiben",price:1.99,img:"https://cdn.produkte.edeka/4311501402375.jpg"},
  {id:3,name:"Frischkäse Natur",price:0.89,img:"https://cdn.produkte.edeka/4311501758397.jpg"},
  {id:4,name:"Bio Milch 3,5%",price:1.49,img:"https://cdn.produkte.edeka/4311501402375.jpg"}
];

let cart = [];

/* FUNKTIONEN */
function save(){
  renderProducts(products);
  renderList();
  renderCosts();
  updateCartCount();
}

function updateCartCount(){
  document.getElementById("cartCount").textContent =
    cart.reduce((s,p)=>s+p.qty,0);
}

function changeQty(id,delta){
  const p = products.find(x=>x.id===id);
  const f = cart.find(x=>x.id===id);
  if(!f && delta>0){
    cart.push({...p,qty:1});
  }else if(f){
    f.qty += delta;
    if(f.qty<=0){
      cart = cart.filter(x=>x.id!==id);
    }
  }
  save();
}

function renderProducts(list){
  const g = document.getElementById("productGrid");
  g.innerHTML = "";
  list.forEach(p=>{
    const f = cart.find(x=>x.id===p.id);
    g.innerHTML += `
      <div class="product-card ${f?"selected":""}">
        ${f?`<div class="qty-badge">x${f.qty}</div>`:""}
        <img src="${p.img}">
        <strong>${p.name}</strong>
        <div class="shop-controls">
          <button onclick="changeQty(${p.id},-1)">−</button>
          <span>${f?f.qty:0}</span>
          <button onclick="changeQty(${p.id},1)">+</button>
        </div>
      </div>`;
  });
}

function renderList(){
  const ul = document.getElementById("cartList");
  ul.innerHTML = "";
  cart.forEach(p=>{
    ul.innerHTML += `
      <li>
        <span>${p.name}</span>
        <span>
          <button onclick="changeQty(${p.id},-1)">−</button>
          ${p.qty}
          <button onclick="changeQty(${p.id},1)">+</button>
        </span>
      </li>`;
  });
}

function renderCosts(){
  let total = 0;
  const body = document.getElementById("costBody");
  body.innerHTML = "";
  cart.forEach(p=>{
    const sum = p.qty * p.price;
    total += sum;
    body.innerHTML += `
      <tr>
        <td>${p.name}</td>
        <td>${p.qty}</td>
        <td>${p.price.toFixed(2)} €</td>
        <td>${sum.toFixed(2)} €</td>
      </tr>`;
  });
  document.getElementById("costTotal").textContent =
    "Gesamt: " + total.toFixed(2) + " €";
}

function searchProducts(t){
  t = t.toLowerCase();
  renderProducts(products.filter(p=>p.name.toLowerCase().includes(t)));
}

function goToList(){
  document.querySelector('[data-view="list"]').click();
}

/* NAVIGATION */
document.querySelectorAll("nav button").forEach(b=>{
  b.onclick = ()=>{
    document.querySelectorAll("nav button").forEach(x=>x.classList.remove("active"));
    b.classList.add("active");
    document.querySelectorAll(".view").forEach(v=>v.classList.remove("active"));
    document.getElementById(b.dataset.view+"-view").classList.add("active");
  };
});

/* START */
renderProducts(products);
</script>

</body>
</html>
