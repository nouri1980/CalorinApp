<!DOCTYPE html>
<html lang="de"><head>
<meta charset="UTF-8"><meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes"><meta name="apple-mobile-web-app-title" content="Kalorien">
<title>Kalorien</title>
<style>
*{box-sizing:border-box;font-family:-apple-system,sans-serif}body{margin:0;background:#f2f2f7;color:#111;padding-bottom:80px}
header{background:#1b9e5a;color:#fff;padding:50px 16px 16px}h1{margin:0;font-size:22px}
.card{background:#fff;border-radius:14px;margin:12px;padding:14px}
.ring{font-size:34px;font-weight:700;text-align:center}.bar{height:8px;background:#e5e5ea;border-radius:4px;overflow:hidden;margin:4px 0 10px}
.bar div{height:100%;background:#1b9e5a}.over div{background:#e5484d}
button{background:#1b9e5a;color:#fff;border:0;border-radius:10px;padding:10px 14px;font-size:15px}
button.s{background:#e5e5ea;color:#111}input,select{width:100%;padding:10px;border:1px solid #ccc;border-radius:10px;font-size:16px;margin:4px 0}
.row{display:flex;justify-content:space-between;align-items:center;padding:6px 0;border-bottom:1px solid #eee}
nav{position:fixed;bottom:0;left:0;right:0;display:flex;background:#fff;border-top:1px solid #ddd;padding-bottom:env(safe-area-inset-bottom)}
nav button{flex:1;background:none;color:#555;border-radius:0;padding:14px 0}nav button.a{color:#1b9e5a;font-weight:700}
.hide{display:none}small{color:#666}
</style></head><body>
<header><h1>Kalorien-Tagebuch</h1><small id="datum" style="color:#dff"></small></header>
<main id="v"></main>
<nav><button data-t="heute">Heute</button><button data-t="rezepte">Rezepte</button><button data-t="einkauf">Einkauf</button><button data-t="verlauf">Verlauf</button><button data-t="ziele">Ziele</button></nav>
<script>
const FOODS={"Hähnchenbrust":[165,31,0,3.6],"Rinderhack (halal)":[250,26,0,17],"Lammfleisch":[282,25,0,20],"Lachs":[208,20,0,13],"Thunfisch (Dose, Wasser)":[116,26,0,1],"Ei":[155,13,1.1,11],"Reis (gekocht)":[130,2.7,28,0.3],"Nudeln (gekocht)":[158,5.8,31,0.9],"Kartoffeln (gekocht)":[87,1.9,20,0.1],"Bulgur (gekocht)":[83,3.1,19,0.2],"Haferflocken":[372,13.5,59,7],"Vollkornbrot":[220,8,40,1.5],"Fladenbrot":[275,9,55,1.2],"Linsen (gekocht)":[116,9,20,0.4],"Kichererbsen (gekocht)":[164,8.9,27,2.6],"Hummus":[166,8,14,10],"Joghurt 1,5 %":[47,3.5,4.5,1.5],"Skyr":[63,11,4,0.2],"Milch 1,5 %":[47,3.4,4.9,1.5],"Feta":[264,14,4,21],"Banane":[89,1.1,23,0.3],"Apfel":[52,0.3,14,0.2],"Datteln":[282,2.5,75,0.4],"Tomate":[18,0.9,3.9,0.2],"Gurke":[15,0.7,3.6,0.1],"Paprika":[31,1,6,0.3],"Brokkoli":[34,2.8,7,0.4],"Olivenöl":[884,0,0,100],"Walnüsse":[654,15,14,65],"Mandeln":[579,21,22,50]};
const REZ=[
{n:"Hähnchen-Reis-Bowl",k:520,p:45,z:[["Hähnchenbrust",150],["Reis (gekocht)",150],["Brokkoli",150],["Olivenöl",5]],s:"Hähnchen würzen und anbraten, Brokkoli dämpfen, mit Reis servieren."},
{n:"Linsensuppe (Mercimek)",k:380,p:20,z:[["Linsen (gekocht)",250],["Tomate",100],["Kartoffeln (gekocht)",100],["Olivenöl",5]],s:"Rote Linsen mit Zwiebel, Karotte und Kreuzkümmel kochen, pürieren, Zitrone dazu."},
{n:"Menemen mit Fladenbrot",k:430,p:22,z:[["Ei",120],["Tomate",150],["Paprika",100],["Fladenbrot",60]],s:"Paprika und Tomaten anschmoren, Eier unterrühren, mit Brot essen."},
{n:"Lachs mit Kartoffeln",k:560,p:35,z:[["Lachs",150],["Kartoffeln (gekocht)",200],["Brokkoli",150]],s:"Lachs im Ofen 15 Min bei 200 °C, dazu Kartoffeln und Gemüse."},
{n:"Overnight Oats mit Datteln",k:390,p:20,z:[["Haferflocken",50],["Skyr",150],["Banane",60],["Datteln",15]],s:"Alles verrühren, über Nacht kühlen."},
{n:"Köfte mit Bulgur-Salat",k:610,p:36,z:[["Rinderhack (halal)",150],["Bulgur (gekocht)",150],["Gurke",100],["Tomate",100]],s:"Hack würzen, Köfte formen und braten. Bulgur mit Gemüse, Petersilie und Zitrone mischen."},
{n:"Thunfisch-Kichererbsen-Salat",k:420,p:38,z:[["Thunfisch (Dose, Wasser)",120],["Kichererbsen (gekocht)",120],["Gurke",100],["Tomate",100],["Olivenöl",5]],s:"Alles mischen, mit Zitrone, Salz und Pfeffer abschmecken."}];
const MZ=["Frühstück","Mittagessen","Abendessen","Snacks"];
const L=k=>JSON.parse(localStorage.getItem(k)||"null"),S=(k,v)=>localStorage.setItem(k,JSON.stringify(v));
let ziele=L("ziele")||{kcal:2000,p:120,kh:220,f:65,wasser:8};let eigene=L("eigene")||{};let tag=L("tage")||{};let gewicht=L("gewicht")||[];let einkauf=L("einkauf")||[];
const heute=()=>new Date().toISOString().slice(0,10);const T=()=>tag[heute()]||(tag[heute()]={e:[],w:0});
const alle=()=>({...FOODS,...eigene});const r=x=>Math.round(x);
function summe(d){let s={k:0,p:0,kh:0,f:0};(d?.e||[]).forEach(e=>{s.k+=e.k;s.p+=e.p;s.kh+=e.kh;s.f+=e.f});return s}
function save(){S("ziele",ziele);S("eigene",eigene);S("tage",tag);S("gewicht",gewicht);S("einkauf",einkauf)}
function bar(a,b,t){const o=a>b;return `<div class="row" style="border:0"><span>${t}</span><span>${r(a)} / ${b} g</span></div><div class="bar ${o?'over':''}"><div style="width:${Math.min(100,a/b*100)}%"></div></div>`}
const V={
heute(){const d=T(),s=summe(d),rest=ziele.kcal-s.k;let h=`<div class="card"><div class="ring" style="color:${rest<0?'#e5484d':'#1b9e5a'}">${r(rest)}</div><p style="text-align:center;margin:0">${rest<0?'kcal über dem Ziel!':'kcal übrig'} · gegessen ${r(s.k)} von ${ziele.kcal}</p><div class="bar ${rest<0?'over':''}"><div style="width:${Math.min(100,s.k/ziele.kcal*100)}%"></div></div>${bar(s.p,ziele.p,'Eiweiß')}${bar(s.kh,ziele.kh,'Kohlenhydrate')}${bar(s.f,ziele.f,'Fett')}</div>`;
h+=`<div class="card"><b>Eintragen</b><select id="mz">${MZ.map(m=>`<option>${m}</option>`).join('')}</select><input id="such" list="fl" placeholder="Lebensmittel suchen"><datalist id="fl">${Object.keys(alle()).map(n=>`<option value="${n}">`).join('')}</datalist><input id="gr" type="number" inputmode="decimal" placeholder="Menge in Gramm" value="100"><button onclick="add()">Hinzufügen</button></div>`;
MZ.forEach(m=>{const es=d.e.map((e,i)=>({...e,i})).filter(e=>e.m===m);const k=es.reduce((a,e)=>a+e.k,0);h+=`<div class="card"><b>${m}</b> <small>${r(k)} kcal</small>${es.map(e=>`<div class="row"><span>${e.n} <small>${e.g} g</small></span><span>${r(e.k)} kcal <button class="s" onclick="del(${e.i})">✕</button></span></div>`).join('')||'<br><small>Noch nichts eingetragen</small>'}</div>`});
h+=`<div class="card"><b>Wasser</b> ${d.w} / ${ziele.wasser} Gläser<br><br><button onclick="wasser(1)">+ Glas</button> <button class="s" onclick="wasser(-1)">− Glas</button></div>`;return h},
rezepte(){const rest=ziele.kcal-summe(T()).k;return `<div class="card">Noch <b>${r(rest)} kcal</b> übrig. <small>Rezepte, die noch passen, sind grün markiert. Alle Rezepte sind halal.</small></div>`+REZ.map((x,i)=>`<div class="card" style="border-left:5px solid ${x.k<=rest?'#1b9e5a':'#ccc'}"><b>${x.n}</b><br><small>ca. ${x.k} kcal · ${x.p} g Eiweiß</small><p>${x.s}</p><small>${x.z.map(z=>z[0]+' '+z[1]+' g').join(', ')}</small><br><br><button onclick="rezEss(${i})">Gegessen</button> <button class="s" onclick="rezKauf(${i})">Auf Einkaufsliste</button></div>`).join('')},
einkauf(){return `<div class="card"><b>Einkaufsliste (halal)</b><input id="neu" placeholder="Artikel hinzufügen"><button onclick="einkauf.push({n:neu.value,ok:false});save();zeig('einkauf')">Hinzufügen</button>${einkauf.map((e,i)=>`<div class="row" onclick="einkauf[${i}].ok=!einkauf[${i}].ok;save();zeig('einkauf')"><span style="${e.ok?'text-decoration:line-through;color:#999':''}">${e.ok?'☑':'☐'} ${e.n}</span></div>`).join('')}<br><button class="s" onclick="einkauf=einkauf.filter(e=>!e.ok);save();zeig('einkauf')">Erledigte löschen</button></div>`},
verlauf(){let h='<div class="card"><b>Letzte 7 Tage</b>';for(let i=6;i>=0;i--){const d=new Date(Date.now()-i*864e5).toISOString().slice(0,10),k=summe(tag[d]).k;h+=`<div class="row"><span>${d.slice(8)}.${d.slice(5,7)}.</span><span style="color:${k>ziele.kcal?'#e5484d':'#111'}">${r(k)} kcal</span></div>`}h+='</div>';h+=`<div class="card"><b>Gewicht</b><input id="kg" type="number" inputmode="decimal" placeholder="Gewicht in kg"><button onclick="if(kg.value){gewicht.push([heute(),+kg.value]);save();zeig('verlauf')}">Speichern</button>${gewicht.slice(-10).reverse().map(g=>`<div class="row"><span>${g[0]}</span><span>${g[1]} kg</span></div>`).join('')}</div>`;return h},
ziele(){return `<div class="card"><b>Meine Ziele</b><br><small>Tageskalorien</small><input id="zk" type="number" value="${ziele.kcal}"><small>Eiweiß (g)</small><input id="zp" type="number" value="${ziele.p}"><small>Kohlenhydrate (g)</small><input id="zh" type="number" value="${ziele.kh}"><small>Fett (g)</small><input id="zf" type="number" value="${ziele.f}"><small>Wasser (Gläser)</small><input id="zw" type="number" value="${ziele.wasser}"><button onclick="ziele={kcal:+zk.value,p:+zp.value,kh:+zh.value,f:+zf.value,wasser:+zw.value};save();alert('Gespeichert')">Speichern</button></div><div class="card"><b>Eigenes Lebensmittel</b> <small>Werte pro 100 g</small><input id="en" placeholder="Name"><input id="ek" type="number" placeholder="kcal"><input id="ep" type="number" placeholder="Eiweiß g"><input id="eh" type="number" placeholder="Kohlenhydrate g"><input id="ef" type="number" placeholder="Fett g"><button onclick="if(en.value){eigene[en.value]=[+ek.value,+ep.value,+eh.value,+ef.value];save();alert('Hinzugefügt')}">Speichern</button></div>`}};
function add(){const n=such.value,g=+gr.value,f=alle()[n];if(!f||!g){alert('Bitte Lebensmittel aus der Liste wählen und Menge eingeben');return}const x=g/100;T().e.push({m:mz.value,n,g,k:f[0]*x,p:f[1]*x,kh:f[2]*x,f:f[3]*x});save();zeig('heute')}
function del(i){T().e.splice(i,1);save();zeig('heute')}
function wasser(n){T().w=Math.max(0,T().w+n);save();zeig('heute')}
function rezEss(i){const m=prompt('Welche Mahlzeit? (Frühstück, Mittagessen, Abendessen, Snacks)','Mittagessen');if(!m)return;REZ[i].z.forEach(([n,g])=>{const f=FOODS[n],x=g/100;T().e.push({m:MZ.includes(m)?m:'Snacks',n,g,k:f[0]*x,p:f[1]*x,kh:f[2]*x,f:f[3]*x})});save();zeig('heute')}
function rezKauf(i){REZ[i].z.forEach(([n,g])=>einkauf.push({n:n+' ('+g+' g)',ok:false}));save();alert('Zur Einkaufsliste hinzugefügt')}
function zeig(t){document.getElementById('v').innerHTML=V[t]();document.querySelectorAll('nav button').forEach(b=>b.classList.toggle('a',b.dataset.t===t))}
document.querySelectorAll('nav button').forEach(b=>b.onclick=()=>zeig(b.dataset.t));
datum.textContent=new Date().toLocaleDateString('de-DE',{weekday:'long',day:'numeric',month:'long'});zeig('heute');
</script></body></html>
