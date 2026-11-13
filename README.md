
<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>World Strategy</title>
<script src="https://cdn.jsdelivr.net/npm/d3@7"></script>
<script src="https://cdn.jsdelivr.net/npm/topojson-client@3"></script>
<style>
*{box-sizing:border-box}
body{margin:0;background:#0b1220;color:#edf3fb;font-family:Arial,sans-serif}
header{height:60px;background:#121d2e;display:flex;align-items:center;justify-content:space-between;padding:10px 16px}
header b{font-size:20px;color:#f4c95d}
button{background:#24364d;color:white;border:1px solid #405675;border-radius:8px;padding:10px;cursor:pointer}
main{display:grid;grid-template-columns:1fr 310px;height:calc(100vh - 60px)}
#map{width:100%;height:100%;background:#0d2940;touch-action:none}
aside{padding:15px;background:#121d2e;overflow:auto}
.card{background:#19263a;padding:13px;border-radius:10px;margin:10px 0}
h3{font-size:15px}
.stats{display:grid;grid-template-columns:1fr 1fr;gap:8px}
.stats div{background:#0b1728;padding:10px;border-radius:7px}
.stats small{display:block;color:#9aabc0}
.actions{display:grid;grid-template-columns:1fr 1fr;gap:8px}
.country{stroke:#101c2b;stroke-width:.7;cursor:pointer}
.country.selected{stroke:#ffe08a;stroke-width:2.5}
#log{font-size:12px;color:#b9c9dc;line-height:1.7}
@media(max-width:700px){
main{grid-template-columns:1fr;grid-template-rows:55% 45%;height:calc(100dvh - 60px)}
aside{padding:10px}
}
</style>
</head>
<body>
<header>
<b>WORLD STRATEGY</b>
<span>💰 <span id="topMoney">500</span></span>
<span>Tur <span id="topTurn">1</span></span>
<button onclick="endTurn()">Turu Bitir</button>
</header>

<main>
<section style="position:relative">
<svg id="map" viewBox="0 0 960 560"></svg>
<div style="position:absolute;top:10px;left:10px;display:flex;gap:5px">
<button onclick="zoom(1.4)">＋</button>
<button onclick="zoom(.7)">－</button>
<button onclick="resetMap()">⌂</button>
</div>
</section>

<aside>
<h2>Ülke Yönetimi</h2>
<div class="card">
<h2 id="countryName">Almanya</h2>
<p>Oyuncu ülkesi</p>
<div class="stats">
<div><small>Hazine</small><b id="money">500</b></div>
<div><small>Nüfus</small><b>83 M</b></div>
<div><small>Ekonomi</small><b id="economy">72</b></div>
<div><small>Ordu</small><b id="army">12</b></div>
</div>
</div>

<h3>İşlemler</h3>
<div class="actions">
<button onclick="develop()">🏭 Ekonomi geliştir (150)</button>
<button onclick="recruit()">🛡️ Ordu kur (100)</button>
<button onclick="diplomacy()">🤝 Diplomasi</button>
<button onclick="changeColor()">🎨 Renk değiştir</button>
</div>

<h3>Oyun günlüğü</h3>
<div class="card" id="log">Tur 1 başladı. Almanya yönetimi sende.</div>
</aside>
</main>

<script>
const svg=d3.select("#map");
const g=svg.append("g");
const projection=d3.geoNaturalEarth1()
.fitExtent([[10,10],[950,550]],{type:"Sphere"});
const path=d3.geoPath(projection);

let countries=[];
let selected="276";
let money=500, economy=72, army=12, turn=1;
let countryColor="#4b83c8";
let relations={};

const zoomBehavior=d3.zoom()
.scaleExtent([1,9])
.on("zoom",e=>g.attr("transform",e.transform));

svg.call(zoomBehavior);

async function loadMap(){
try{
const [world,names]=await Promise.all([
fetch("https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json").then(r=>r.json()),
fetch("https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.tsv").then(r=>r.text())
]);

let nameMap={};
names.trim().split(/\r?\n/).slice(1).forEach(line=>{
let p=line.split("\t");
nameMap[p[0]]=p[1];
});

countries=topojson.feature(world,world.objects.countries).features;

g.append("path").datum({type:"Sphere"})
.attr("d",path).attr("fill","#0d2940");

g.selectAll(".country")
.data(countries)
.join("path")
.attr("class",d=>"country "+(String(d.id)===selected?"selected":""))
.attr("d",path)
.attr("fill",d=>String(d.id)==="276"?countryColor:
["#bb7656","#4b9a79","#9875bf","#b58d42","#4c9ca8"][Number(d.id)%5])
.on("click",(event,d)=>{
selected=String(d.id);
document.getElementById("countryName").textContent=nameMap[selected]||"Ülke";
g.selectAll(".country").classed("selected",x=>String(x.id)===selected);
});

}catch(e){
document.getElementById("countryName").textContent="Harita yüklenemedi";
}
}

function update(){
document.getElementById("money").textContent=money;
document.getElementById("topMoney").textContent=money;
document.getElementById("economy").textContent=economy;
document.getElementById("army").textContent=army;
document.getElementById("topTurn").textContent=turn;
}

function message(t){
let log=document.getElementById("log");
log.innerHTML=t+"<br>"+log.innerHTML;
}

function develop(){
if(selected!=="276")return alert("Kendi ülkene geç.");
if(money<150)return alert("Yeterli para yok.");
money-=150; economy+=5;
message("Ekonomi geliştirildi.");
update();
}

function recruit(){
if(selected!=="276")return alert("Kendi ülkene geç.");
if(money<100)return alert("Yeterli para yok.");
money-=100;army+=3;
message("Ordu güçlendirildi.");
update();
}

function diplomacy(){
if(selected==="276")return alert("Önce başka ülke seç.");
if(money<50)return alert("Yeterli para yok.");
money-=50;
relations[selected]=(relations[selected]||0)+15;
message("Diplomatik ilişki geliştirildi.");
update();
alert("Diplomasi başarılı. İlişki: "+relations[selected]);
}

function changeColor(){
const colors=["#4b83c8","#c66c58","#4b9a79","#9b75c5","#c39b43"];
countryColor=colors[(colors.indexOf(countryColor)+1)%colors.length];
g.selectAll(".country").filter(d=>String(d.id)==="276").attr("fill",countryColor);
}

function endTurn(){
turn++;
money+=Math.floor(economy*.8);
message("Yeni tur başladı. Gelir: "+Math.floor(economy*.8));
update();
}

function zoom(value){svg.transition().call(zoomBehavior.scaleBy,value)}
function resetMap(){svg.transition().call(zoomBehavior.transform,d3.zoomIdentity)}

loadMap();
</script>
</body>
</html>
# world-strategy-game
multiplayer World strategy game
