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
  --danger:#c0392b;
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
  background:rgba(255,255,255,.18);
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
  font-weight:800;
  padding:2px 6px;
  border-radius:12px;
}

/* STATUS */
.status{
  padding:.6rem 1rem;
  font-size:.85rem;
  color:#444;
}

/* VIEWS */
.view{display:none;}
.view.active{display:block;}

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
  height:130px;
  object-fit:contain;
  background:#f0f0f0;
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

/* LISTEN */
.list-manager{
  padding:1rem;
  background:#fff;
  border-bottom:1px solid var(--border);
  display:flex;
  gap:.6rem;
  flex-wrap:wrap;
  align-items:center;
}

.list-manager select{
  padding:.6rem;
  border-radius:8px;
  border:1px solid var(--border);
}

.list-btn{
  padding:.6rem .9rem;
  border:none;
  border-radius:8px;
  font-weight:600;
  cursor:pointer;
}

.list-btn.new{background:var(--green);color:#fff;}
.list-btn.rename{background:#2980b9;color:#fff;}
.list-btn.delete{background:var(--danger);color:#fff;}

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

/* PRINT */
@media print{
  header,nav,.shop-toolbar,.product-grid,.list-manager{display:none;}
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
  Aktive Liste: <strong id="activeListName"></strong>
</div>

<main>

<section id="shop-view" class="view active">
  <div class="shop-toolbar">
    <input type="search" placeholder="Produkt suchen (z. B. Wurst, Käse, Milch)" oninput="searchProducts(this.value)">
  </div>
  <div class="product-grid" id="productGrid"></div>
</section>

<section id="list-view" class="view">
  <div class="list-manager">
    <select id="listSelect" onchange="switchList(this.value)"></select>
    <button class="list-btn new" onclick="createList()">➕ Neue Liste</button>
    <button class="list-btn rename" onclick="renameList()">✏️ Umbenennen</button>
    <button class="list-btn delete" onclick="deleteList()">🗑️ Löschen</button>
  </div>
  <ul class="cart-list" id="cartList"></ul>
  <div style="padding:1rem">
    <button onclick="window.print()">🖨️ Einkaufsliste drucken</button>
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
/* STABILE PRODUKTDATEN + BILDER (öffentlich & zuverlässig) */
const products=[
 {id:1,name:"Salami",price:2.49,img:"https://upload.wikimedia.org/wikipedia/commons/3/3a/Salami_slices.jpg"},
 {id:2,name:"Gouda Käse",price:1.99,img:"https://upload.wikimedia.org/wikipedia/commons/8/89/Gouda_cheese.jpg"},
 {id:3,name:"Milch 1L",price:1.49,img:"https://upload.wikimedia.org/wikipedia/commons/a/a4/Milk_glass.jpg"},
 {id:4,name:"Frischkäse",price:0.89,img:"https://upload.wikimedia.org/wikipedia/commons/1/19/Cream_cheese.jpg"},
 {id:5,name:"Butter",price:1.79,img:"https://upload.wikimedia.org/wikipedia/commons/2/2d/Butter.jpg"},
 {id:6,name:"Brot",price:2.29,img:"https://upload.wikimedia.org/wikipedia/commons/0/0f/Bread_loaf.jpg"}
];

let lists = JSON.parse(localStorage.getItem("lists")) || {"Master Liste":[]};
let activeList = localStorage.getItem("activeList") || "Master Liste";

function save(){
  localStorage.setItem("lists",JSON.stringify(lists));
  localStorage.setItem("activeList",activeList);
  updateUI();
}

function updateUI(){
  document.getElementById("activeListName").textContent = activeList;
  updateListSelect();
  renderProducts(products);
  renderList();
  renderCosts();
  updateCartCount();
}

function updateCartCount(){
  document.getElementById("cartCount").textContent =
    lists[activeList].reduce((s,p)=>s+p.qty,0);
}

function updateListSelect(){
  const sel=document.getElementById("listSelect");
  sel.innerHTML="";
  Object.keys(lists).forEach(name=>{
    const o=document.createElement("option");
    o.value=name;o.textContent=name;
    if(name===activeList)o.selected=true;
    sel.appendChild(o);
  });
}

function switchList(name){activeList=name;save();}
function createList(){
  const n=prompt("Name der neuen Liste:");
  if(n && !lists[n]){lists[n]=[];activeList=n;save();}
}
function renameList(){
  const n=prompt("Neuer Name:",activeList);
  if(n && !lists[n]){
    lists[n]=lists[activeList];
    delete lists[activeList];
    activeList=n;
    save();
  }
}
function deleteList(){
  if(activeList==="Master Liste") return alert("Master Liste kann nicht gelöscht werden");
  if(confirm("Liste wirklich löschen?")){
    delete lists[activeList];
    activeList="Master Liste";
    save();
  }
}

function changeQty(id,delta){
  const list=lists[activeList];
  const f=list.find(p=>p.id===id);
  if(!f && delta>0){
    list.push({...products.find(p=>p.id===id),qty:1});
  }else if(f){
    f.qty+=delta;
    if(f.qty<=0){
      lists[activeList]=list.filter(p=>p.id!==id);
    }
  }
  save();
}

function renderProducts(list){
  const g=document.getElementById("productGrid");
  g.innerHTML="";
  const activeItems=lists[activeList];
  list.forEach(p=>{
    const f=activeItems.find(x=>x.id===p.id);
    g.innerHTML+=`
      <div class="product-card ${f?"selected":""}">
        ${f?`<div class="qty-badge">x${f.qty}</div>`:""}
        <img src="${p.img}" alt="${p.name}">
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
  const ul=document.getElementById("cartList");
  ul.innerHTML="";
  lists[activeList].forEach(p=>{
    ul.innerHTML+=`
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
  let total=0;
  const body=document.getElementById("costBody");
  body.innerHTML="";
  lists[activeList].forEach(p=>{
    const sum=p.qty*p.price;
    total+=sum;
    body.innerHTML+=`
      <tr>
        <td>${p.name}</td>
        <td>${p.qty}</td>
        <td>${p.price.toFixed(2)} €</td>
        <td>${sum.toFixed(2)} €</td>
      </tr>`;
  });
  document.getElementById("costTotal").textContent =
    "Gesamt: "+total.toFixed(2)+" €";
}

function searchProducts(t){
  t=t.toLowerCase();
  renderProducts(products.filter(p=>p.name.toLowerCase().includes(t)));
}

function goToList(){
  document.querySelector('[data-view="list"]').click();
}

document.querySelectorAll("nav button").forEach(b=>{
  b.onclick=()=>{
    document.querySelectorAll("nav button").forEach(x=>x.classList.remove("active"));
    b.classList.add("active");
    document.querySelectorAll(".view").forEach(v=>v.classList.remove("active"));
    document.getElementById(b.dataset.view+"-view").classList.add("active");
  };
});

updateUI();
</script>

</body>
</html>
