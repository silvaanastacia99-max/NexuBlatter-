<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8"
    <h1>NEXUSBLATTER 3D</h1>
    <p>ANIME BATTLE ROYALE</p>
    <p>Escolha seu personagem:</p>
    <div class="herois">
      <button class="heroi selecionado" data-heroi="Nexus">⚡ NEXUS</button>
      <button class="heroi" data-heroi="Akira">🔥 AKIRA</button>
      <button class="heroi" data-heroi="Kai">❄️ KAI</button>
      <button class="heroi" data-heroi="Ren">🌟 REN</button>
    </div>
    <p>Explore o mapa, derrote os adversários e vença a batalha!</p>
    <button id="jogar">ENTRAR NO JOGO</button>
  </div>
</div>

<script type="module">
import * as THREE from "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js";

const canvas=document.createElement("canvas");
document.body.insertBefore(canvas,document.body.firstChild);

const cena=new THREE.Scene();
cena.background=new THREE.Color(0x91c9e8);
cena.fog=new THREE.Fog(0x91c9e8,45,115);

const camera=new THREE.PerspectiveCamera(65,innerWidth/innerHeight,.1,250);
const renderer=new THREE.WebGLRenderer({canvas,antialias:true});
renderer.setPixelRatio(Math.min(devicePixelRatio,1.5));
renderer.setSize(innerWidth,innerHeight);
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;
renderer.outputColorSpace=THREE.SRGBColorSpace;

cena.add(new THREE.HemisphereLight(0xdaf3ff,0x52613b,2));
const sol=new THREE.DirectionalLight(0xffffff,2.1);
sol.position.set(-25,45,20);
sol.castShadow=true;
sol.shadow.mapSize.set(1024,1024);
cena.add(sol);

const material=(cor,extra={})=>new THREE.MeshStandardMaterial({color:cor,roughness:.8,...extra});
const verde=material(0x438b43);
const tronco=material(0x70452b);
const folhas=material(0x207d45);
const parede=material(0xd1b68b);
const telhado=material(0x87483d);
const pedra=material(0x777d83);
const chao=new THREE.Mesh(new THREE.PlaneGeometry(180,180),verde);
chao.rotation.x=-Math.PI/2;
chao.receiveShadow=true;
cena.add(chao);

const grade=new THREE.GridHelper(180,45,0x315f36,0x57844a);
grade.position.y=.025;
cena.add(grade);

function caixa(w,h,d,mat,x,y,z){
 const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),mat);
 m.position.set(x,y,z);m.castShadow=true;m.receiveShadow=true;
 cena.add(m);return m;
}
function arvore(x,z){
 const g=new THREE.Group();
 const t=new THREE.Mesh(new THREE.CylinderGeometry(.25,.4,2.2,7),tronco);
 t.position.y=1.1;t.castShadow=true;g.add(t);
 const f=new THREE.Mesh(new THREE.ConeGeometry(1.5,3.5,8),folhas);
 f.position.y=3.3;f.castShadow=true;g.add(f);
 g.position.set(x,0,z);cena.add(g);
}
function casa(x,z){
 caixa(7,3.5,6,parede,x,1.75,z);
 const roof=caixa(8,1,7,telhado,x,4,z);
 roof.rotation.z=.08;
 caixa(1.4,2.2,.15,tronco,x,1.1,z-3.08);
 caixa(1.2,1.1,.2,material(0x7bd7ec),x-2,2,z-3.1);
 caixa(1.2,1.1,.2,material(0x7bd7ec),x+2,2,z-3.1);
}
for(let i=0;i<42;i++){
 let x=(Math.random()-.5)*145,z=(Math.random()-.5)*145;
 if(Math.abs(x)<13&&Math.abs(z)<13)continue;
 arvore(x,z);
}
casa(-23,-18);casa(24,-24);casa(-30,25);casa(31,26);
casa(5,39);casa(-48,-3);

const caminho=caixa(9,.08,150,material(0xb7a47a),0,.05,0);
caminho.position.y=.06;

const coresHerois={Nexus:0x16b9e9,Akira:0xe74c3c,Kai:0x55aaff,Ren:0xd8bd45};
let heroiEscolhido="Nexus";
let jogador,vida=100,pontos=0,acabou=false,iniciou=false;
let inimigos=[],tempoAtaque=0,tempoEspecial=0,tempoDano=0,pulando=false,velocidadePulo=0;
const teclas={};
const direcao=new THREE.Vector2();
let cameraAngulo=0;
let cameraArrastando=false;
let ultimoX=0;
let joystickAtivo=false;
let joystickId=null;
let deslocamento={x:0,y:0};

function criarPersonagem(cor){
 const g=new THREE.Group();
 const corpo=new THREE.Mesh(new THREE.CapsuleGeometry(.55,.9,5,9),material(cor));
 corpo.position.y=1.25;corpo.castShadow=true;g.add(corpo);
 const cabeca=new THREE.Mesh(new THREE.SphereGeometry(.43,16,12),material(0xffd0a5));
 cabeca.position.y=2.15;cabeca.castShadow=true;g.add(cabeca);
 const cabelo=new THREE.Mesh(new THREE.SphereGeometry(.46,12,8),material(0x20202b));
 cabelo.scale.set(1, .65,1);cabelo.position.set(0,2.43,0);g.add(cabelo);
 const faixa=new THREE.Mesh(new THREE.BoxGeometry(.72,.12,.08),material(0xeeeeee));
 faixa.position.set(0,2.18,.39);g.add(faixa);
 const bracoMat=material(cor);
 for(const lado of [-1,1]){
  const b=new THREE.Mesh(new THREE.CapsuleGeometry(.16,.55,4,7),bracoMat);
  b.position.set(lado*.68,1.35,0);b.rotation.z=lado*.15;g.add(b);
 }
 cena.add(g);return g;
}

function criarInimigo(x,z){
 const g=new THREE.Group();
 const corpo=new THREE.Mesh(new THREE.CapsuleGeometry(.5,.85,5,8),material(0x9e3448));
 corpo.position.y=1.15;corpo.castShadow=true;g.add(corpo);
 const cabeca=new THREE.Mesh(new THREE.SphereGeometry(.4,12,10),material(0xd4a17e));
 cabeca.position.y=2.05;g.add(cabeca);
 const olho=material(0xff3030,{emissive:0x661111});
 for(const lado of [-1,1]){
  const e=new THREE.Mesh(new THREE.SphereGeometry(.06,8,8),olho);
  e.position.set(lado*.14,2.1,.35);g.add(e);
 }
 g.position.set(x,0,z);cena.add(g);
 return {obj:g,vida:3,vivo:true,vel:2.1,ataque:0};
}

function atualizarHUD(){
 document.getElementById("barraVida").style.width=Math.max(0,vida)+"%";
 document.getElementById("pontos").textContent="Pontos: "+pontos;
 document.getElementById("inimigosTexto").textContent="Inimigos: "+inimigos.filter(e=>e.vivo).length;
}

function mostrarFim(vitoria){
 acabou=true;
 const caixa=document.getElementById("caixa");
 document.getElementById("mensagem").style.display="flex";
 caixa.innerHTML=`
 <h1>${vitoria?"VOCÊ VENCEU!":"FIM DE JOGO"}</h1>
 <p>${vitoria?"Parabéns! Você derrotou todos os adversários!":"Sua aventura terminou. Tente novamente!"}</p>
 <p>Pontuação: <b>${pontos}</b></p>
 <button id="jogar">JOGAR NOVAMENTE</button>`;
 document.getElementById("jogar").addEventListener("click",reiniciar);
 document.getElementById("controles").style.display="none";
 document.getElementById("hud").style.display="none";
 document.getElementById("minimapa").style.display="none";
}

function reiniciar(){
 if(jogador)cena.remove(jogador);
 inimigos.forEach(e=>cena.remove(e.obj));
 inimigos=[];
 vida=100;pontos=0;acabou=false;iniciou=true;
 jogador=criarPersonagem(coresHerois[heroiEscolhido]);
 jogador.position.set(0,0,0);
 for(let i=0;i<8;i++){
  let ang=Math.random()*Math.PI*2;
  let dist=18+Math.random()*32;
  inimigos.push(criarInimigo(Math.cos(ang)*dist,Math.sin(ang)*dist));
 }
 document.getElementById("mensagem").style.display="none";
 document.getElementById("hud").style.display="block";
 document.getElementById("controles").style.display="block";
 document.getElementById("minimapa").style.display="block";
 document.getElementById("ajuda").textContent="Derrote todos os adversários!";
 atualizarHUD();
}

document.querySelectorAll(".heroi").forEach(b=>{
 b.addEventListener("click",()=>{
  heroiEscolhido=b.dataset.heroi;
  document.querySelectorAll(".heroi").forEach(x=>x.classList.remove("selecionado"));
  b.classList.add("selecionado");
 });
});
document.getElementById("jogar").addEventListener("click",reiniciar);

function atacar(){
 if(!iniciou||acabou||performance.now()-tempoAtaque<550)return;
 tempoAtaque=performance.now();
 let acertou=false;
 inimigos.forEach(e=>{
  if(!e.vivo)return;
  const d=jogador.position.distanceTo(e.obj.position);
  if(d<4.1){
   e.vida--;acertou=true;pontos+=10;
   if(e.vida<=0){
    e.vivo=false;e.obj.visible=false;pontos+=90;
    document.getElementById("ajuda").textContent="Adversário derrotado!";
   }
  }
 });
 atualizarHUD();
 if(inimigos.every(e=>!e.vivo))mostrarFim(true);
}

function especial(){
 if(!iniciou||acabou||performance.now()-tempoEspecial<4000)return;
 tempoEspecial=performance.now();
 inimigos.forEach(e=>{
  if(e.vivo&&jogador.position.distanceTo(e.obj.position)<9){
   e.vida-=2;pontos+=20;
   if(e.vida<=0){e.vivo=false;e.obj.visible=false;pontos+=80;}
  }
 });
 atualizarHUD();
 if(inimigos.every(e=>!e.vivo))mostrarFim(true);
 document.getElementById("ajuda").textContent="PODER ESPECIAL!";
}

function pular(){
 if(!iniciou||acabou||pulando)return;
 pulando=true;velocidadePulo=7;
}

document.getElementById("btnAtaque").addEventListener("pointerdown",e=>{e.preventDefault();atacar()});
document.getElementById("btnEspecial").addEventListener("pointerdown",e=>{e.preventDefault();especial()});
document.getElementById("btnPulo").addEventListener("pointerdown",e=>{e.preventDefault();pular()});

document.addEventListener("keydown",e=>{
 teclas[e.key.toLowerCase()]=true;
 if(e.key===" "){e.preventDefault();pular();}
 if(e.key.toLowerCase()==="j")atacar();
 if(e.key.toLowerCase()==="k")especial();
});
document.addEventListener("keyup",e=>teclas[e.key.toLowerCase()]=false);

const joystick=document.getElementById("joystick");
const pino=document.getElementById("pino");
joystick.addEventListener("pointerdown",e=>{
 e.preventDefault();joystickAtivo=true;joystickId=e.pointerId;
 joystick.setPointerCapture(e.pointerId);moverPino(e);
});
joystick.addEventListener("pointermove",e=>{
 if(joystickAtivo&&e.pointerId===joystickId)moverPino(e);
});
function soltarJoystick(){
 joystickAtivo=false;deslocamento.x=0;deslocamento.y=0;
 pino.style.left="36px";pino.style.top="36px";
}
joystick.addEventListener("pointerup",soltarJoystick);
joystick.addEventListener("pointercancel",soltarJoystick);
function moverPino(e){
 const r=joystick.getBoundingClientRect();
 let x=e.clientX-(r.left+r.width/2),y=e.clientY-(r.top+r.height/2);
 const limite=37,dist=Math.hypot(x,y);
 if(dist>limite){x=x/dist*limite;y=y/dist*limite;}
 deslocamento.x=x/limite;deslocamento.y=y/limite;
 pino.style.left=(36+x)+"px";pino.style.top=(36+y)+"px";
}

renderer.domElement.addEventListener("pointerdown",e=>{
 if(e.pointerType==="touch"||e.pointerType==="mouse"){
  cameraArrastando=true;ultimoX=e.clientX;
 }
});
renderer.domElement.addEventListener("pointermove",e=>{
 if(cameraArrastando){
  cameraAngulo-=(e.clientX-ultimoX)*.006;
  ultimoX=e.clientX;
 }
});
function pararCamera(){cameraArrastando=false;}
renderer.domElement.addEventListener("pointerup",pararCamera);
renderer.domElement.addEventListener("pointercancel",pararCamera);
renderer.domElement.addEventListener("contextmenu",e=>e.preventDefault());

function atualizarMinimapa(){
 const c=document.getElementById("minimapa");
 const ctx=c.getContext("2d");
 ctx.clearRect(0,0,100,100);
 ctx.fillStyle="#285b38";ctx.fillRect(0,0,100,100);
 ctx.fillStyle="#b6a47c";ctx.fillRect(47,0,6,100);
 ctx.fillStyle="#54dfff";
 ctx.beginPath();ctx.arc(50+jogador.position.x*.48,50+jogador.position.z*.48,4,0,Math.PI*2);ctx.fill();
 ctx.fillStyle="#ff5555";
 inimigos.forEach(e=>{
  if(!e.vivo)return;
  ctx.beginPath();ctx.arc(50+e.obj.position.x*.48,50+e.obj.position.z*.48,2.5,0,Math.PI*2);ctx.fill();
 });
}

const relogio=new THREE.Clock();
function animar(){
 requestAnimationFrame(animar);
 const dt=Math.min(relogio.getDelta(),.04);
 if(iniciou&&!acabou&&jogador){
  let x=deslocamento.x+(teclas.d?1:0)-(teclas.a?1:0);
  let z=deslocamento
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<meta name="theme-color" content="#08152b">
<title>NexusBlatter 3D - Anime Battle Royale</title>
<style>
:root{--cyan:#27e8ff;--gold:#ffd52e;--panel:#071426eF;--line:#67dcff88}
*{box-sizing:border-box}
html,body{margin:0;width:100%;height:100%;overflow:hidden;background:#08152b;color:#fff;font-family:Arial,sans-serif;touch-action:none}
#world{position:fixed;inset:0;width:100%;height:100%;display:block}
#shade{position:fixed;inset:0;pointer-events:none;background:linear-gradient(90deg,#03101dbb,transparent 35%,transparent 70%,#03101d88),linear-gradient(0deg,#03101d88,transparent 30%,#03101d44)}
#topbar{position:fixed;top:10px;left:12px;right:12px;display:flex;align-items:center;justify-content:space-between;gap:8px;z-index:5}
.profile,.currency,.topicons{background:#061326d9;border:1px solid #ffffff30;border-radius:14px;padding:8px 11px;box-shadow:0 5px 20px #0005}
.profile{display:flex;align-items:center;gap:9px;min-width:145px}
.avatar{width:43px;height:43px;border-radius:12px;background:linear-gradient(135deg,#7146ce,#20cfff);display:grid;place-items:center;font-size:25px;border:2px solid #ffe36a}
.small{font-size:11px;color:#b9d5e7}.name{font-weight:900;font-size:14px}
.currency{display:flex;gap:15px;font-weight:bold;font-size:14px}
.topicons{display:flex;gap:14px;font-size:20px}
#brand{position:fixed;top:82px;left:20px;z-index:4;text-shadow:0 4px 5px #000}
#brand h1{margin:0;font-size:clamp(28px,5vw,55px);font-weight:1000;letter-spacing:-2px;font-style:italic;transform:skew(-7deg)}
#brand h1 span{color:var(--cyan)}
#brand p{margin:0 0 0 18px;font-size:13px;letter-spacing:5px;font-weight:bold}
#leftmenu{position:fixed;left:12px;top:37%;z-index:6;display:flex;flex-direction:column;gap:7px;width:158px}
.menuBtn{border:1px solid #ffffff35;background:#071629db;color:#fff;border-radius:8px;padding:11px 10px;text-align:left;font-size:14px;font-weight:900;box-shadow:0 4px 12px #0005}
.menuBtn.active,.menuBtn:active{background:linear-gradient(90deg,#ffcb20,#e78d00);color:#17202a;border-color:#fff2a4}
#rightpanel{position:fixed;right:12px;top:22%;width:min(265px,34vw);z-index:6;background:#061326df;border:1px solid #ffffff55;border-radius:13px;padding:10px;box-shadow:0 8px 25px #0007}
#rightpanel h3{margin:0 0 9px;font-size:14px;color:var(--gold);text-transform:uppercase}
#cards{display:grid;grid-template-columns:repeat(3,1fr);gap:6px}
.charCard{min-width:0;border:2px solid #71859a;background:#14263c;border-radius:8px;padding:4px;color:white;font-size:11px;font-weight:bold;text-align:center}
.charCard.selected{border:3px solid var(--gold);box-shadow:0 0 12px #ffdc39}
.portrait{height:62px;border-radius:5px;display:grid;place-items:center;font-size:34px;background:linear-gradient(145deg,#4566a8,#111c35)}
.charCard span{display:block;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;margin-top:3px}
#bottomnav{position:fixed;bottom:12px;left:12px;right:12px;z-index:7;display:flex;align-items:center;gap:5px}
.navBtn{flex:1;background:#071426e8;border:1px solid #ffffff38;border-radius:8px;color:white;padding:12px 4px;font-weight:bold;font-size:clamp(10px,1.5vw,14px)}
.navBtn.active{color:var(--gold);border-bottom:4px solid var(--gold)}
#start{background:linear-gradient(110deg,#ffe84b,#ff9f00);color:#191300;font-size:clamp(17px,2.3vw,24px);font-weight:1000;border:0;border-radius:10px;padding:13px 23px;box-shadow:0 0 20px #ffb00088;min-width:145px}
#modal{position:fixed;inset:0;z-index:20;display:none;align-items:center;justify-content:center;padding:16px;background:#020914b8}
#dialog{width:min(440px,95vw);max-height:82vh;overflow:auto;background:linear-gradient(145deg,#123653,#081321);border:2px solid #38dfff;border-radius:18px;padding:22px;text-align:center;box-shadow:0 0 35px #00bfff55}
#dialog h2{margin:0 0 10px;color:var(--cyan);font-size:27px}
#dialog p{line-height:1.45;color:#e2edf6}
.action{border:0;border-radius:10px;padding:13px 20px;background:linear-gradient(90deg,#25dfff,#32ffad);font-weight:900;color:#071625;font-size:16px;margin:5px}
.secondary{background:#203b55;color:#fff;border:1px solid #75cfff}
#gamehud{display:none;position:fixed;top:12px;left:12px;right:12px;z-index:8;justify-content:space-between;align-items:flex-start;gap:8px;pointer-events:none}
#minimap{width:clamp(88px,18vw,150px);height:clamp(88px,18vw,150px);border:2px solid #d8f6ff;border-radius:12px;background:#071426df;box-shadow:0 4px 18px #0008} 
#ammoText{display:inline-block;margin-top:4px;color:#ffe36a;font-size:13px}
.hudbox{background:#071426d9;padding:9px 12px;border:1px solid #ffffff55;border-radius:10px;font-weight:bold}
#healthbar{width:130px;height:10px;background:#555;border-radius:10px;overflow:hidden;margin-top:5px}
#healthfill{height:100%;width:100%;background:#2cff72}
#gamecontrols{display:none;position:fixed;bottom:18px;left:15px;right:15px;z-index:9;justify-content:space-between;align-items:end;pointer-events:none}
#stick{width:120px;height:120px;border-radius:50%;background:#ffffff24;border:2px solid #ffffff80;position:relative;pointer-events:auto}
#knob{position:absolute;width:48px;height:48px;left:34px;top:34px;border-radius:50%;background:#ffffffb8;border:2px solid white}
#fightbuttons{display:flex;gap:10px;align-items:end;pointer-events:auto}
.fight{width:67px;height:67px;border-radius:50%;border:2px solid #ffffffaa;background:#173451eF;color:white;font-weight:bold;font-size:13px}
#attack{width:78px;height:78px;background:#d84938}
#reload{background:#9b7621;font-size:10px}
#cameraHint{position:fixed;right:12px;bottom:110px;z-index:8;font-size:10px;color:white;display:none;text-shadow:0 1px 3px #000}
#toast{position:fixed;top:45%;left:50%;transform:translate(-50%,-50%);z-index:30;background:#061426ed;border:1px solid var(--cyan);border-radius:10px;padding:12px 18px;display:none;text-align:center}
@media(max-width:650px){
 #topbar{top:6px;left:6px;right:6px}.profile{min-width:0;padding:6px}.avatar{width:35px;height:35px}.currency{gap:7px;padding:7px;font-size:12px}.topicons{gap:8px;padding:8px;font-size:1
<canvas id="world"></canvas><div id="shade"></div>
<div id="topbar">
 <div class="profile"><div class="avatar" id="profileFace">⚡</div><div><div class="name">NexusBlatter</div><div class="small">Nv. 01 · Explorador</div></div></div>
 <div class="currency"><span>🪙 <b>0</b></span><span>💎 <b>0</b></span><span>➕</span></div>
 <div class="topicons"><span>♟</span><span>✉</span><span>⚙</span><span>📶</span></div>
</div>
<div id="brand"><h1>NEXUS<span>BLATTER</span></h1><p>ANIME BATTLE ROYALE · 3D</p></div>
<div id="leftmenu">
 <button class="menuBtn active" data-panel="LOJA">🛒 LOJA</button>
 <button class="menuBtn" data-panel="PERSONAGENS">👕 PERSONAGENS</button>
 <button class="menuBtn" data-panel="ARMAS">⚔ ARMAS</button>
 <button class="menuBtn" data-panel="MOCHILA">🎒 MOCHILA</button>
 <button class="menuBtn" data-panel="MISSÕES">☑ MISSÕES</button>
 <button class="menuBtn" data-panel="EVENTOS">🏆 EVENTOS</button
