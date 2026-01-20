<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Einkaufsmanager</title>

<style>
body{font-family:Arial,sans-serif;background:#f4f6f7;margin:0}
header{background:#007d3c;color:#fff;padding:15px}
input{width:100%;padding:10px;font-size:16px;margin:10px 0}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(200px,1fr));gap:15px;padding:15px}
.card{background:#fff;border-radius:10px;padding:10px;text-align:center}
.card img{width:100%;height:120px;object-fit:cover;background:#ddd}
button{padding:8px 12px;margin:5px;font-size:16px}
</style>
</head>

<body>

<header>
  <h1>Einkaufsmanager</h1>
</header>

<div style="padding:15px">
  <input id="search" placeholder="Produkt suchen (z. B. Käse)" oninput="filterProducts()">
</div>

<div class="grid" id="grid"></div>

<script>
/* 🔒 FEST DEFINIERTE PRODUKTE – KÖNNEN NICHT „VERSCHWINDEN“ */
const PRODUCTS = [
  {name:"Salami", img:"data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='300' height='200'><rect width='100%' height='100%' fill='%23ddd'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle'>Salami</text></svg>"},
  {name:"Käse", img:"data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='300' height='200'><rect width='100%' height='100%' fill='%23ddd'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle'>Käse</text></svg>"},
  {name:"Milch", img:"data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='300' height='200'><rect width='100%' height='100%' fill='%23ddd'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle'>Milch</text></svg>"},
  {name:"Brot", img:"data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='300' height='200'><rect width='100%' height='100%' fill='%23ddd'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle'>Brot</text></svg>"}
];

const grid = document.getElementById("grid");

/* ✅ RENDERT IMMER – KEIN LEERZUSTAND MÖGLICH */
function render(list){
  grid.innerHTML = "";
  list.forEach(p=>{
    grid.innerHTML += `
      <div class="card">
        <img src="${p.img}">
        <strong>${p.name}</strong><br>
        <button>➕</button>
      </div>
    `;
  });
}

/* 🔍 FILTERT NUR – LÖSCHT NIE */
function filterProducts(){
  const q = document.getElementById("search").value.toLowerCase();
  render(PRODUCTS.filter(p => p.name.toLowerCase().includes(q)));
}

/* 🚀 INITIALER START */
render(PRODUCTS);
</script>

</body>
</html>
