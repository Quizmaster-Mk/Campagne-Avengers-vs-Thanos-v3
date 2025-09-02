<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Avengers Battle — Thanos</title>
<style>
:root { --bg:#0f0f12; --panel:#121216; --muted:#9aa0a6; --accent:#6ef08a; --fg:#eef3ea; --hit:#ff4d4d; --heal:#4dff88; }
body{ margin:0; font-family:Inter, Roboto, "Helvetica Neue", Arial; background:var(--bg); color:var(--fg); }
header{ padding:18px 12px; text-align:center; }
main{ max-width:600px; margin:0 auto; padding:12px; position:relative; }
.panel{ background:var(--panel); border-radius:10px; padding:14px; margin:10px 0; box-shadow:0 4px 18px rgba(0,0,0,0.6); position:relative; overflow:hidden;}
h1{ margin:0; font-size:20px; }
.muted{ color:var(--muted); font-size:13px; margin-top:6px; }
#stats{ display:flex; gap:10px; flex-wrap:wrap; margin-bottom:8px; }
.stat{ background:#0d0d0f; border:1px solid #232323; padding:8px 10px; border-radius:8px; font-weight:600; color:var(--fg); width:100%; }
.bar-container{ background:#222; border-radius:6px; width:100%; height:16px; margin-top:4px; }
.bar{ height:100%; border-radius:6px; transition: width 0.4s ease; }
#story{ white-space:pre-wrap; font-size:15px; line-height:1.4; margin-bottom:6px; }
#log{ color:var(--muted); font-size:13px; margin-top:8px; min-height:48px; white-space:pre-wrap; }
#choices{ display:grid; gap:8px; margin-top:12px; }
button.choice{ background:#141416; color:var(--fg); border:1px solid #2a2a2a; padding:10px 12px; border-radius:8px; text-align:center; cursor:pointer; font-weight:600; }
button.choice:hover{ border-color:var(--accent); color:var(--accent); transform:translateY(-1px); }
.avatar-container{ display:flex; justify-content:space-between; align-items:center; margin-bottom:8px; position:relative; }
.avatar{ width:80px; height:80px; border:2px solid #444; border-radius:8px; display:flex; align-items:center; justify-content:center; font-size:12px; text-align:center; line-height:1.2; background:#1a1a1a; position:relative; }
.hit{ animation:hitFlash 0.3s; }
@keyframes hitFlash{ 0%{background:#ff4d4d;}50%{background:#1a1a1a;}100%{background:#ff4d4d;} }

.projectile{
  width:12px; height:12px; background:var(--accent); border-radius:50%; position:absolute; top:50%; left:50%; transform:translate(-50%,-50%); z-index:100;
  animation:shoot 0.5s linear forwards;
}
@keyframes shoot{
  0%{ transform:translate(0,0);}
  100%{ transform:translate(200px,-20px);}
}

.shake{
  animation:shake 0.3s;
}
@keyframes shake{
  0%{transform:translate(0,0);}
  25%{transform:translate(4px,0);}
  50%{transform:translate(-4px,0);}
  75%{transform:translate(4px,0);}
  100%{transform:translate(0,0);}
}

footer{ text-align:center; padding:12px 8px; color:var(--muted); font-size:13px; }
</style>
</head>
<body>
<header>
  <h1>Avengers Battle — Stop Thanos</h1>
  <div class="muted">Tour par tour stratégique avec animations d’attaque !</div>
</header>

<main>
  <section class="panel">
    <div class="avatar-container">
      <div class="avatar" id="player-avatar">Héros</div>
      <div class="avatar" id="enemy-avatar">Thanos</div>
    </div>
    <div id="stats"></div>
    <div id="story" class="small"></div>
    <div id="log" class="small"></div>
    <div id="choices"></div>
  </section>
  <section style="text-align:center; margin-top:6px">
    <button id="restart" class="choice" style="max-width:260px;">🔁 Recommencer</button>
  </section>
</main>

<script>
// ---------- DONNÉES ----------
const HEROES = ["Iron Man","Thor","Black Widow","Captain America","Hulk","Doctor Strange"];
const POWERS = {
  "Iron Man":["Répulseurs","Surcharge d'arc"],
  "Thor":["Mjolnir","Éclair divin"],
  "Black Widow":["Arts martiaux","Infiltration"],
  "Captain America":["Bouclier ricochet","Charge tactique"],
  "Hulk":["Poing destructeur","Rage inarrêtable"],
  "Doctor Strange":["Portail dimensionnel","Sort mystique"]
};
const UNIQUE_POWERS = ["Éclair divin","Infiltration","Portail dimensionnel"];
const WEAPONS = ["Grenade EMP","Bouclier énergétique","Arc plasma","Lance-filet","Épée asgardienne","Neuro-stun"];

// ---------- ÉTAT ----------
let state = {
  phase:"pickHero",
  player:{ hero:null, power:null, weapon:null, pv:30, maxPv:30, usedUniquePowers:{} },
  enemy:{ name:"Thanos", pv:50, maxPv:50, rounds:5 },
  round:1,
  log:[]
};

// ---------- UTILITAIRES ----------
const $ = s=>document.querySelector(s);
function setChoices(list){ const c=$("#choices"); c.innerHTML=""; list.forEach(opt=>{ const b=document.createElement("button"); b.className="choice"; b.textContent=opt.label; b.onclick=opt.onClick; c.appendChild(b); }); }
function pushLog(t){ state.log.unshift(t); state.log=state.log.slice(0,6); render(); }
function render(){
  const p=state.player, e=state.enemy, stats=$("#stats"); stats.innerHTML="";
  stats.appendChild(el("div",{class:"stat"},`Tes PV: ${p.pv}/${p.maxPv}`));
  stats.appendChild(bar("player-bar",p.pv,p.maxPv,"--accent"));
  stats.appendChild(el("div",{class:"stat"},`${e.name} PV: ${e.pv}/${e.maxPv}`));
  stats.appendChild(bar("enemy-bar",e.pv,e.maxPv,"--hit"));
  $("#log").innerHTML = state.log.join("\n");
  $("#player-avatar").textContent = p.hero || "Héros";
}
function bar(id,current,max,colorVar){
  const cont = document.createElement("div"); cont.className="bar-container";
  const b = document.createElement("div"); b.className="bar"; b.id=id; b.style.width=`${(current/max)*100}%`; b.style.backgroundColor=`var(${colorVar})`;
  cont.appendChild(b); return cont;
}
function el(tag,attrs={},text=""){ const e=document.createElement(tag); for(const k in attrs) e.setAttribute(k,attrs[k]); e.textContent=text; return e; }
function rnd(min,max){ return Math.floor(Math.random()*(max-min+1))+min; }
function flashBar(id){ const b=$("#"+id); b.classList.add("hit"); setTimeout(()=>b.classList.remove("hit"),300); }

// ---------- FLOW ----------
function start(){
  state.phase="pickHero";
  state.player={ hero:null, power:null, weapon:null, pv:30, maxPv:30, usedUniquePowers:{} };
  state.enemy={ name:"Thanos", pv:50, maxPv:50, rounds:5 };
  state.round=1; state.log=[];
  $("#story").textContent="Choisis ton Avenger !";
  setChoices(HEROES.map(h=>({label:h,onClick:()=>chooseHero(h)})));
  render();
}
function chooseHero(h){ state.player.hero=h; $("#story").textContent=`Héros choisi: ${h}. Choisis un pouvoir :`; setChoices(POWERS[h].map(p=>({label:p,onClick:()=>choosePower(p)}))); render(); }
function choosePower(pw){ state.player.power=pw; $("#story").textContent=`Pouvoir choisi: ${pw}. Choisis une arme :`; setChoices(WEAPONS.map(w=>({label:w,onClick:()=>chooseWeapon(w)}))); render(); }
function chooseWeapon(w){ state.player.weapon=w; $("#story").textContent=`Arme choisie: ${w}. Prépare-toi pour le combat !`; playerAttackPrompt(); }

// ---------- COMBAT ----------
function playerAttackPrompt(){
  if(state.player.pv<=0 || state.enemy.pv<=0 || state.round>state.enemy.rounds) return;
  $("#story").textContent=`Round ${state.round}/${state.enemy.rounds} — Choisis ton attaque :`;
  setChoices([
    {label:"💨 Rapide (3-5 PV)", onClick:()=>playerAttack("rapide")},
    {label:"💥 Lourde (5-9 PV, peut rater)", onClick:()=>playerAttack("lourde")},
    {label:"✨ Pouvoir (usage unique)", onClick:()=>playerAttack("pouvoir")},
    {label:"⚔️ Arme (4-7 PV)", onClick:()=>playerAttack("arme")}
  ]);
}

function playerAttack(type){
  // Création du projectile
  const panel = document.querySelector(".panel");
  const proj = document.createElement("div");
  proj.className = "projectile";
  panel.appendChild(proj);

  proj.addEventListener("animationend",()=>{
    panel.removeChild(proj);
    applyPlayerDamage(type);
  });
}

function applyPlayerDamage(type){
  let dmg=0, miss=false;
  const p=state.player;
  if(type==="rapide"){ dmg=rnd(3,5); }
  else if(type==="lourde"){ dmg=rnd(5,9); if(Math.random()<0.2){ dmg=0; miss=true; } }
  else if(type==="pouvoir"){
    if(UNIQUE_POWERS.includes(p.power)){
      if(p.usedUniquePowers[p.power]){ pushLog("⚠️ Pouvoir déjà utilisé ! Attaque rapide à la place."); dmg=rnd(3,5);}
      else { dmg=rnd(8,12); p.usedUniquePowers[p.power]=true; pushLog(`💥 Pouvoir ${p.power} utilisé !`); }
    } else { dmg=rnd(6,10); }
  }
  else if(type==="arme"){ dmg=rnd(4,7); }

  state.enemy.pv=Math.max(0,state.enemy.pv-dmg);
  flashBar("enemy-bar");
  const enemyAvatar = $("#enemy-avatar");
  enemyAvatar.classList.add("shake");
  setTimeout(()=>enemyAvatar.classList.remove("shake"),300);

  pushLog(miss? "❌ Attaque ratée !" : `✅ Tu infliges ${dmg} PV à Thanos`);
  render();
  setTimeout(enemyTurn, 500);
}

function enemyTurn(){
  if(state.enemy.pv<=0 || state.player.pv<=0){ checkEnd(); return; }
  let dmg=rnd(4,9);
  state.player.pv=Math.max(0,state.player.pv-dmg);
  flashBar("player-bar");
  pushLog(`⚠️ Thanos riposte et inflige ${dmg} PV`);
  render();
  state.round++;
  checkEnd();
}

function checkEnd(){
  if(state.enemy.pv<=0){ pushLog("🎉 Tu as vaincu Thanos !"); setChoices([{label:"🔁 Rejouer", onClick:start}]); return; }
  if(state.player.pv<=0){ pushLog("💀 Tu as été vaincu par Thanos..."); setChoices([{label:"🔁 Rejouer", onClick:start}]); return; }
  if(state.round>state.enemy.rounds){ pushLog("⏰ Plus de rounds ! Thanos s'enfuit..."); setChoices([{label:"🔁 Rejouer", onClick:start}]); return; }
  playerAttackPrompt();
}

// ---------- INIT ----------
$("#restart").addEventListener("click",start);
start();
</script>
</body>
</html>
