# Verity- <!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>VERITY — 3D HORROR</title>
<style>
*{box-sizing:border-box}
html,body{margin:0;width:100%;height:100%;overflow:hidden;background:#080a0e;color:white;font-family:Arial;touch-action:none}
canvas{position:fixed;inset:0;display:block}
button{font-weight:bold;border:0;border-radius:9px;padding:12px;background:#ffdf35;color:#171717;touch-action:manipulation}
.screen{position:fixed;inset:0;z-index:20;display:flex;align-items:center;justify-content:center;background:radial-gradient(#27313b,#050609);padding:18px;text-align:center;overflow:auto}
.card{width:min(440px,100%);background:#090c11ee;padding:22px;border:1px solid #555;border-radius:16px}
h1{font-size:clamp(45px,12vw,72px);color:#ffdf35;letter-spacing:6px;margin:8px}
.menuBtn{display:block;width:100%;margin:9px 0}
.secondary{background:#29313a;color:white}
#hud{display:none;position:fixed;inset:0;z-index:5;pointer-events:none}
#top{display:flex;justify-content:space-between;padding:9px}
.panel,#msg{background:#10151ded;padding:10px;border-radius:9px}
#msg{position:absolute;top:12%;left:4%;width:92%;text-align:center;font-size:13px;line-height:1.5}
#cross{position:absolute;top:49%;left:49%;font-size:24px}
#move{position:absolute;bottom:105px;left:8px;display:grid;grid-template-columns:43px 43px 43px;gap:4px;pointer-events:auto}
#move button{width:43px;height:43px;background:#263442ee;color:white;font-size:21px;padding:0}
#look{position:absolute;right:15px;bottom:118px;width:80px;height:80px;border:3px solid #ffdf35;border-radius:50%;background:#263442aa;pointer-events:auto;touch-action:none}
#look:after{content:"ОБЗОР";position:absolute;top:29px;left:17px;font-size:11px}
#actions{position:absolute;bottom:7px;width:100%;display:flex;justify-content:center;flex-wrap:wrap;gap:5px;padding:4px;pointer-events:auto}
.action{font-size:11px;padding:11px 8px}
#questions{display:none;position:absolute;top:35%;width:100%;pointer-events:auto}
#questions button{margin:4px}
.small{font-size:12px;color:#bbb;line-height:1.5}
.item{padding:12px;background:#191e25;margin:8px 0;border-radius:8px;text-align:left}
</style>
</head>
<body>
<div id="menu" class="screen"><div class="card">
<h1>VERITY</h1><p>3D HORROR</p>
<p>Он всегда рядом. Даже когда ты смотришь в пустоту.</p>
<button class="menuBtn" id="start">НАЧАТЬ ИГРУ</button>
<button class="menuBtn secondary" id="ach">🏆 ДОСТИЖЕНИЯ</button>
<button class="menuBtn secondary" id="ends">🔒 КОНЦОВКИ</button>
<p class="small">Исследуй остров. Построй дом. Не зли Верити.</p>
</div></div>

<div id="library" class="screen" style="display:none"><div class="card">
<h2 id="libTitle">Достижения</h2><div id="items"></div>
<button class="menuBtn" id="back">НАЗАД</button>
</div></div>

<div id="ending" class="screen" style="display:none"><div class="card">
<h1 style="font-size:42px">VERITY</h1><h2 id="endTitle">КОНЕЦ</h2><p id="endText"></p>
<button class="menuBtn" id="endMenu">В МЕНЮ</button>
<button class="menuBtn secondary" id="retry">ИГРАТЬ СНОВА</button>
</div></div>

<div id="hud">
<div id="top"><div class="panel" id="day">ДЕНЬ 1</div><div class="panel">🪵 <span id="wood">0</span> | 🪨 <span id="stone">0</span></div></div>
<div id="cross">+</div><div id="msg">Открой коробку.</div>
<div id="questions"><button data-q="0">Кто такой Фалсити?</button><button data-q="1">2 + 2?</button><button data-q="2">Где мы?</button></div>
<div id="move"><span></span><button data-m="f">▲</button><span></span><button data-m="l">◀</button><button data-m="b">▼</button><button data-m="r">▶</button></div>
<div id="look"></div>
<div id="actions">
<button class="action" id="interact">✋ ДЕЙСТВИЕ</button>
<button class="action" id="chop">🪓 РУБИТЬ</button>
<button class="action" id="mine">⛏ ДОБЫВАТЬ</button>
<button class="action" id="build">🔨 СТРОИТЬ</button>
<button class="action" id="sleep">🛏 СПАТЬ</button>
</div></div>

<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.164.1/build/three.module.js';

const $=id=>document.getElementById(id);
const scene=new THREE.Scene();
scene.background=new THREE.Color(0x91a5b9);
scene.fog=new THREE.Fog(0x91a5b9,35,100);
const camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,.1,200);
camera.rotation.order='YXZ';
camera.position.set(0,1.7,4);
const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,1.5));
renderer.shadowMap.enabled=true;
renderer.outputColorSpace=THREE.SRGBColorSpace;
document.body.appendChild(renderer.domElement);
scene.add(new THREE.HemisphereLight(0xd9eaff,0x343020,2));
const sun=new THREE.DirectionalLight(0xffe8bd,2.3);
sun.position.set(-12,24,10);sun.castShadow=true;scene.add(sun);

const world=new THREE.Group();scene.add(world);
const mat=c=>new THREE.MeshStandardMaterial({color:c,roughness:.9});
const grass=mat(0x526d3d),woodMat=mat(0x60402b),leaves=mat(0x24482a);
const stoneMat=mat(0x85878a),yellow=mat(0xffdf16),dark=mat(0x171717);
function cube(x,y,z,sx,sy,sz,m,parent=world){
 const o=new THREE.Mesh(new THREE.BoxGeometry(sx,sy,sz),m);
 o.position.set(x,y,z);o.castShadow=true;o.receiveShadow=true;parent.add(o);return o;
}
function say(s){$('msg').textContent=s}
let floor,redPlate,islandSize=32;
function ground(size){
 islandSize=size;
 if(floor)world.remove(floor);
 if(redPlate)world.remove(redPlate);
 floor=cube(0,-.25,0,size,.5,size,grass);
 redPlate=cube(0,.015,0,1,.05,1,mat(0xc52c34));
}
ground(32);

let trees=[],rocks=[],wood=0,stone=0,day=1;
let opened=false,houseBuilt=false,wallsBuilt=false,towerBuilt=false;
let falls=0,sleptAfterFalls=false,secretTriggered=false;
let night=false,started=false,ended=false,falling=false,rescuing=false;
let fallVelocity=0,rescueStart=0;
let yaw=0,pitch=0,house=null,door=null,bed=null,tower=null;
let doorOpen=false,monsterAttack=false;
let unlocked={};
try{unlocked=JSON.parse(localStorage.getItem('verity_endings_v3')||'{}')}catch{}
let achievements={house:false,walls:false,tower:false,fall:false};
const keys={},touch={f:false,b:false,l:false,r:false};

function tree(x,z){
 const g=new THREE.Group();g.position.set(x,0,z);
 cube(0,1,0,.45,2,.45,woodMat,g);
 const crown=new THREE.Mesh(new THREE.ConeGeometry(1.25,2.5,7),leaves);
 crown.position.y=3; crown.castShadow=true;g.add(crown);
 world.add(g);trees.push({g,x,z,active:true});
}
function rock(x,z){
 const o=new THREE.Mesh(new THREE.DodecahedronGeometry(.65),stoneMat);
 o.position.set(x,.35,z);o.castShadow=true;world.add(o);
 rocks.push({o,x,z,active:true});
}
function plantTrees(){
 [[-4,-4],[4,4],[-7,3],[7,-4],[-10,-2],[2,-9],[10,4],[-11,8],[12,-9],[13,0],[-13,-5],[4,12],[-3,13],[10,13]].forEach(p=>tree(...p));
}
plantTrees();

const box=new THREE.Group();box.position.set(1,.4,-1);
cube(0,0,0,.9,.8,.8,woodMat,box);
const lid=cube(0,.43,0,.95,.12,.85,mat(0x33241a),box);world.add(box);

const verity=new THREE.Group();
const head=new THREE.Mesh(new THREE.SphereGeometry(.65,20,16),yellow);
head.castShadow=true;verity.add(head);
for(const x of [-.22,.22]){
 const eye=new THREE.Mesh(new THREE.SphereGeometry(.075,12,8),dark);
 eye.position.set(x,.12,.57);verity.add(eye);
}
verity.position.set(-1,1,-1);verity.scale.setScalar(.8);
verity.visible=false;world.add(verity);

function makeHouse(){
 const g=new THREE.Group();
 // Размер дома: 10 x 8. Передняя стена имеет настоящий дверной проём.
 cube(0,.12,0,10,.24,8,woodMat,g);
 cube(-4.75,2,0,.5,4,8,woodMat,g);
 cube(4.75,2,0,.5,4,8,woodMat,g);
 cube(0,2,-3.75,10,4,.5,woodMat,g);
 cube(-3.5,2,3.75,3,4,.5,woodMat,g);
 cube(3.5,2,3.75,3,4,.5,woodMat,g);
 // Двускатная крыша: две наклонные стороны, конёк вдоль дома.
 const roofMat=mat(0x493027);
 const left=cube(-2.45,4.45,0,5.7,.24,8.6,roofMat,g);
 left.rotation.z=-Math.atan2(2.0,2.5);
 const right=cube(2.45,4.45,0,5.7,.24,8.6,roofMat,g);
 right.rotation.z=Math.atan2(2.0,2.5);
 // Дверь открывается на петлях, не закрывая проход.
 const pivot=new THREE.Group();
 pivot.position.set(-1.5,0,4.02);
 cube(1.5,1.55,0,3,3.1,.16,woodMat,pivot);
 cube(2.55,1.55,.12,.12,.2,.08,yellow,pivot);
 g.add(pivot);
 g.position.set(-7,0,-7);world.add(g);
 door={pivot,open:false};
 // Кровать внутри дома.
 const b=new THREE.Group();
 cube(0,.32,0,2.4,.4,1.5,woodMat,b);
 cube(0,.58,0,2.3,.18,1.4,mat(0xd7d1c5),b);
 cube(0,.72,-.45,2.1,.15,.4,mat(0xf0e5d4),b);
 b.position.set(-8,0,-8);world.add(b);bed=b;
 return g;
}
function makeWalls(){
 const g=new THREE.Group();
 for(let i=-8;i<=8;i+=2){
  cube(i,.8,-8,1.8,1.6,.35,woodMat,g);
  cube(i,.8,8,1.8,1.6,.35,woodMat,g);
  cube(-8,.8,i,.35,1.6,1.8,woodMat,g);
  cube(8,.8,i,.35,1.6,1.8,woodMat,g);
 }
 world.add(g);return g;
}
function makeTower(){
 const g=new THREE.Group();
 for(let y=0;y<5;y++){
  for(const [x,z] of [[-1,-1],[1,-1],[-1,1],[1,1]])
   cube(x,y*1.4+.65,z,.18,1.4,.18,woodMat,g);
  cube(0,y*1.4+1.25,0,2.5,.18,2.5,woodMat,g);
 }
 cube(0,7.3,0,3,.2,3,woodMat,g);
 g.position.set(10,0,-10);world.add(g);return g;
}

// Огромная рука, которая поднимает игрока из пустоты.
const hand=new THREE.Group();
const skin=mat(0xffd21c);
cube(0,0,0,2.2,.65,2,skin,hand);
for(let i=0;i<5;i++){
 const x=(i-2)*.37;
 cube(x,.05,-1.4,.32,.4,1.4,skin,hand);
}
cube(-1.15,-.05,.1,.42,.4,1,skin,hand);
hand.visible=false;scene.add(hand);

function updateHUD(){
 $('day').textContent=night?'НОЧЬ '+day:'ДЕНЬ '+day;
 $('wood').textContent=wood;$('stone').textContent=stone;
}
function insideHouse(x,z){
 return houseBuilt&&x>-11.8&&x<-2.2&&z>-10.8&&z<-3.1;
}
function canMoveTo(nx,nz){
 if(!houseBuilt)return true;
 const was=insideHouse(camera.position.x,camera.position.z);
 const will=insideHouse(nx,nz);
 // Войти и выйти можно только через открытый дверной проём.
 if(!was&&will){
  return doorOpen&&Math.abs(nx+7)<1.65&&camera.position.z>=-3.2;
 }
 if(was&&!will){
  return doorOpen&&Math.abs(nx+7)<1.65&&nz>=-3.2;
 }
 // Не позволяем проходить сквозь внешние стены.
 if(!was&&!will){
  const crossesX=(camera.position.x<-11.8&&nx>=-11.8)||(camera.position.x>-2.2&&nx<=-2.2);
  const crossesZ=(camera.position.z<-10.8&&nz>=-10.8)||(camera.position.z>-3.1&&nz<=-3.1);
  if(crossesX&&nz>-10.8&&nz<-3.1)return false;
  if(crossesZ&&nx>-11.8&&nx<-2.2){
   if(doorOpen&&Math.abs(nx+7)<1.65&&nz>-3.2)return true;
   return false;
  }
 }
 return true;
}
function movePlayer(dx,dz){
 const nx=camera.position.x+dx,nz=camera.position.z+dz;
 if(canMoveTo(nx,camera.position.z))camera.position.x=nx;
 if(canMoveTo(camera.position.x,nz))camera.position.z=nz;
}
function nearest(){
 let best=null,dist=4;
 const list=[{type:'box',pos:box.position}];
 trees.forEach(t=>{if(t.active)list.push({type:'tree',obj:t,pos:t.g.position})});
 rocks.forEach(r=>{if(r.active)list.push({type:'rock',obj:r,pos:r.o.position})});
 if(verity.visible)list.push({type:'verity',pos:verity.position});
 if(houseBuilt)list.push({type:'door',pos:new THREE.Vector3(-7,1,-2.8)});
 if(houseBuilt)list.push({type:'bed',pos:bed.position});
 if(towerBuilt)list.push({type:'tower',pos:new THREE.Vector3(10,2,-10)});
 for(const n of list){
  const d=Math.hypot(camera.position.x-n.pos.x,camera.position.z-n.pos.z);
  if(d<dist){best=n;dist=d}
 }
 return best;
}
function saveEnding(id){
 unlocked[id]=true;
 try{localStorage.setItem('verity_endings_v3',JSON.stringify(unlocked))}catch{}
}
function finishEnding(title,text,id){
 if(ended)return;
 ended=true;if(id)saveEnding(id);
 $('endTitle').textContent=title;$('endText').textContent=text;
 $('ending').style.display='flex';$('questions').style.display='none';
}
function spawnSecretMonsters(){
 if(secretTriggered)return;
 secretTriggered=true;monsterAttack=true;
 say('Верити: «ТЫ УПАЛ ТРИЖДЫ! ТЫ ОБЕЩАЛ!»');
 for(let i=0;i<5;i++){
  const m=new THREE.Group();
  const h=new THREE.Mesh(new THREE.SphereGeometry(.7,16,12),yellow);
  h.castShadow=true;m.add(h);
  for(const x of [-.23,.23]){
   const e=new THREE.Mesh(new THREE.SphereGeometry(.08),dark);
   e.position.set(x,.15,.6);m.add(e);
  }
  m.position.set(-11+i*1.7,1,-2.2);world.add(m);
 }
 setTimeout(()=>finishEnding('РАЗОЗЛИТЬ ВЕРИТИ',
 'Ты трижды упал в первый день, поспал и открыл дверь. Верити больше не простит тебя.','angry'),2200);
}
function interact(){
 if(ended||falling||rescuing)return;
 const n=nearest();
 if(!n){say('Подойди ближе к объекту.');return}
 if(n.type==='box'){
  if(opened){say('Коробка пуста.');return}
  opened=true;verity.visible=true;lid.rotation.x=-1.5;
  say('Верити: «Собери дерево и построй дом на красной плите».');
 }else if(n.type==='verity'){
  $('questions').style.display='block';say('Верити ждёт твоего вопроса.');
 }else if(n.type==='door'){
  if(!doorOpen){
   door.pivot.rotation.y=-1.45;door.open=true;doorOpen=true;
   say('Дверь открыта. Пройди через проём в дом.');
   if(falls>=3&&sleptAfterFalls&&!secretTriggered)spawnSecretMonsters();
  }else{
   door.pivot.rotation.y=0;door.open=false;doorOpen=false;say('Дверь закрыта.');
  }
 }else if(n.type==='bed'){
  sleep();
 }else if(n.type==='tower'){
  if(night)finishEnding('СМЕРТЬ НА ВЫШКЕ','Монстр нашёл тебя на третью ночь.','tower');
  else say('Ночью на вышке будет опасно.');
 }
}
function chop(){
 if(!started||ended)return;
 const n=nearest();
 if(!n||n.type!=='tree'){say('Подойди к дереву.');return}
 n.obj.active=false;n.obj.g.visible=false;wood++;updateHUD();
 say('Получено дерево: '+wood);
 setTimeout(()=>{n.obj.active=true;n.obj.g.visible=true},12000);
}
function mine(){
 if(day<2){say('Камни появятся на второй день.');return}
 const n=nearest();
 if(!n||n.type!=='rock'){say('Подойди к камню.');return}
 n.obj.active=false;n.obj.o.visible=false;stone++;updateHUD();say('Получен камень: '+stone);
 setTimeout(()=>{n.obj.active=true;n.obj.o.visible=true},12000);
}
function build(){
 if(!opened){say('Сначала открой коробку.');return}
 if(!houseBuilt){
  if(wood<5){say('Для дома нужно 5 дерева. Собрано: '+wood);return}
  if(Math.hypot(camera.position.x,camera.position.z)>6){
   say('Подойди к красной плите в центре острова.');return
  }
  wood-=5;houseBuilt=true;house=makeHouse();achievements.house=true;
  say('Дом готов! Дверь спереди. Внутри кровать.');updateHUD();
 }else if(day===2&&!wallsBuilt){
  if(stone<5){say('Для защитных стен нужно 5 камней.');return}
  stone-=5;wallsBuilt=true;makeWalls();achievements.walls=true;
  say('Защитные стены построены.');updateHUD();
 }else if(day>=3&&!towerBuilt){
  if(wood<5||stone<5){say('Для вышки нужно 5 дерева и 5 камней.');return}
  wood-=5;stone-=5;towerBuilt=true;tower=makeTower();achievements.tower=true;
  say('Вышка построена.');updateHUD();
 }else say('Пока строить нечего.');
}
function sleep(){
 if(!houseBuilt){say('Сначала построй дом.');return}
 const d=Math.hypot(camera.position.x-bed.position.x,camera.position.z-bed.position.z);
 if(d>3){say('Зайди в дом и подойди к кровати.');return}
 if(day===1){
  if(falls>=3)sleptAfterFalls=true;
  day=2;ground(42);
  [[-15,-12],[14,-14],[-17,4],[17,7],[-10,16],[10,17],[0,13],[-16,-2],[16,-3],[-4,-16],[5,-17],[20,12],[-20,11],[0,-20],[22,-8],[-22,-8],[8,22],[-8,22]].forEach(p=>tree(...p));
  [[-3,-3],[3,3],[-5,0],[4,0],[0,5],[5,-4],[-4,5],[1,-5],[5,5],[-5,-5],[13,0],[-13,3],[12,13],[-14,-12],[18,4]].forEach(p=>rock(...p));
  say('День 2. Появились камни. Построй защитные стены.');
 }else if(day===2){
  if(!wallsBuilt){say('Сначала построй защитные стены.');return}
  day=3;say('День 3. Построй вышку.');
 }else if(day===3){
  if(!towerBuilt){say('Сначала построй вышку.');return}
  night=true;scene.background.set(0x05040d);scene.fog.color.set(0x080510);sun.intensity=.15;
  say('ТРЕТЬЯ НОЧЬ. Иди на вышку!');
 }else say('Ночь становится темнее...');
 updateHUD();
}
function startFall(){
 if(falling||rescuing||ended)return;
 falling=true;fallVelocity=0;achievements.fall=true;
 if(day===1&&!night)falls++;
 say('Ты падаешь в пустоту! Рука уже рядом...');
 $('questions').style.display='none';
}
function startRescue(){
 falling=false;rescuing=true;rescueStart=performance.now();hand.visible=true;
 hand.position.set(camera.position.x,-5,camera.position.z-1.5);
}
function finishRescue(){
 falling=false;rescuing=false;hand.visible=false;
 camera.position.set(0,1.7,4);yaw=0;pitch=0;
 say('Огромная жёлтая рука спасла тебя!');
}
$('start').onclick=()=>{
 $('menu').style.display='none';$('ending').style.display='none';$('hud').style.display='block';
 started=true;say('Открой коробку и начни исследование.');
};
function showLibrary(mode){
 $('menu').style.display='none';$('library').style.display='flex';
 $('libTitle').textContent=mode==='ends'?'Концовки':'Достижения';
 const list=$('items');list.innerHTML='';
 const arr=mode==='ends'?
  [{id:'angry',name:'Разозлить Верити',desc:'Три падения, сон и открытая дверь.'},{id:'tower',name:'Смерть на вышке',desc:'Доживи до третьей ночи и доберись до вышки.'}]:
  [{id:'house',name:'Первый дом',desc:'Построй дом.'},{id:'walls',name:'Защитник',desc:'Построй стены.'},{id:'tower',name:'Наблюдатель',desc:'Построй вышку.'},{id:'fall',name:'Падение',desc:'Упади с острова.'},{id:'angry',name:'Разозлить Верити',desc:'Открой секретную концовку.'}];
 arr.forEach(a=>{
  const done=mode==='ends'?!!unlocked[a.id]:!!achievements[a.id]||!!unlocked[a.id];
  const d=document.createElement('div');d.className='item';
  d.innerHTML='<b>'+(done?'✓ ':'🔒 ')+a.name+'</b><div class="small">'+a.desc+'</div>';
  list.appendChild(d);
 });
}
$('ach').onclick=()=>showLibrary('ach');$('ends').onclick=()=>showLibrary('ends');
$('back').onclick=()=>{$('library').style.display='none';$('menu').style.display='flex'};
$('endMenu').onclick=()=>{$('ending').style.display='none';$('hud').style.display='none';$('menu').style.display='flex'};
$('retry').onclick=()=>location.reload();
$('interact').onclick=interact;$('chop').onclick=chop;$('mine').onclick=mine;$('build').onclick=build;$('sleep').onclick=sleep;
document.querySelectorAll('[data-q]').forEach(b=>b.onclick=()=>{
 $('questions').style.display='none';
 const q=+b.dataset.q;
 say(['Верити: «Фалсити — мой клон».','Верити: «Четыре!»','Верити: «Мы в игре Verity».'][q]);
});
document.addEventListener('keydown',e=>keys[e.key.toLowerCase()]=true);
document.addEventListener('keyup',e=>keys[e.key.toLowerCase()]=false);
document.querySelectorAll('[data-m]').forEach(b=>{
 const k=b.dataset.m;
 b.addEventListener('pointerdown',e=>{e.preventDefault();touch[k]=true});
 for(const ev of ['pointerup','pointercancel','lostpointercapture'])
  b.addEventListener(ev,()=>touch[k]=false);
});
let lastLook=null;
$('look').addEventListener('pointerdown',e=>{
 e.preventDefault();lastLook={x:e.clientX,y:e.clientY};
 $('look').setPointerCapture(e.pointerId);
});
$('look').addEventListener('pointermove',e=>{
 if(!lastLook||falling||rescuing)return;
 yaw-=(e.clientX-lastLook.x)*.009;
 pitch=THREE.MathUtils.clamp(pitch-(e.clientY-lastLook.y)*.009,-1.3,1.3);
 lastLook={x:e.clientX,y:e.clientY};
});
$('look').addEventListener('pointerup',()=>lastLook=null);
$('look').addEventListener('pointercancel',()=>lastLook=null);

const clock=new THREE.Clock();
function animate(){
 requestAnimationFrame(animate);
 const dt=Math.min(clock.getDelta(),.05);
 if(started&&!ended&&!falling&&!rescuing){
  camera.rotation.set(pitch,yaw,0,'YXZ');
  const f=new THREE.Vector3(-Math.sin(yaw),0,-Math.cos(yaw));
  const r=new THREE.Vector3(Math.cos(yaw),0,-Math.sin(yaw));
  const v=new THREE.Vector3();
  if(keys.w||keys.arrowup||touch.f)v.add(f);
  if(keys.s||keys.arrowdown||touch.b)v.sub(f);
  if(keys.a||keys.arrowleft||touch.l)v.sub(r);
  if(keys.d||keys.arrowright||touch.r)v.add(r);
  if(v.lengthSq()){v.normalize().multiplyScalar(5*dt);movePlayer(v.x,v.z)}
  camera.position.y=1.7;
  if(Math.abs(camera.position.x)>islandSize/2||Math.abs(camera.position.z)>islandSize/2)startFall();
  if(night&&towerBuilt&&Math.hypot(camera.position.x-10,camera.position.z+10)<2.5)
   finishEnding('СМЕРТЬ НА ВЫШКЕ','Монстр настиг тебя на третью ночь.','tower');
 }
 if(falling){
  fallVelocity+=10*dt;camera.position.y-=fallVelocity*dt;
  if(camera.position.y< -3)startRescue();
 }
 if(rescuing){
  const t=Math.min((performance.now()-rescueStart)/1700,1);
  hand.position.set(camera.position.x,-5+t*7,camera.position.z-1.5);
  camera.position.y=-3.5+t*5.2;
  if(t>=1)finishRescue();
 }
 if(verity.visible)verity.position.y=1+Math.sin(performance.now()*.002)*.12;
 renderer.render(scene,camera);
}
animate();
addEventListener('resize',()=>{
 camera.aspect=innerWidth/innerHeight;camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
updateHUD();
</script>
</body>
</html>
