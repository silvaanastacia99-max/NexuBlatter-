<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<meta name="theme-color" content="#101c25">
<title>NexusBlatter 3D - Anime Battle Royale</title>
<style>
*{box-sizing:border-box}
html,body{margin:0;width:100%;height:100%;overflow:hidden;background:#101c25;font-family:Arial,sans-serif;color:white;touch-action:none}
canvas{display:block;width:100%;height:100%}
#hud{position:fixed;top:10px;left:10px;right:10px;z-index:5;display:none;pointer-events:none}
#topo{display:flex;justify-content:space-between;gap:8px;align-items:center}
.painel{background:#07131ddd;border:1px solid #5ce5ff66;border-radius:10px;padding:8px 11px;font-size:13px;text-shadow:0 1px 2px #000}
#barraFundo{width:135px;height:12px;background:#472b32;border-radius:20px;overflow:hidden;margin-top:5px}
#barraVida{height:100%;width:100%;background:#35e46a;transition:width .2s}
#pontos{font-weight:bold;color:#ffe66d}
#ajuda{margin-top:7px;display:inline-block}
#controles{position:fixed;inset:0;z-index:6;display:none;pointer-events:none}
#joystick{position:absolute;left:22px;bottom:25px;width:125px;height:125px;border-radius:50%;background:#ffffff20;border:2px solid #ffffff66;pointer-events:auto;touch-action:none}
#pino{position:absolute;width:49px;height:49px;border-radius:50%;background:#65dfffbb;border:2px solid white;left:36px;top:36px}
.botoes{position:absolute;right:17px;bottom:25px;display:flex;flex-direction:column;align-items:center;gap:9px;pointer-events:auto}
.linhaBotoes{display:flex;gap:9px}
.btnJogo{width:65px;height:65px;border-radius:50%;border:2px solid #ffffffaa;background:#122e43dd;color:white;font-weight:bold;font-size:12px;touch-action:none}
.btnJogo:active{background:#1685a8}
#btnAtaque{width:76px;height:76px;background:#a52c45ee}
#btnEspecial{background:#6335a8ee}
#btnPulo{background:#237b9cee}
#mensagem{position:fixed;inset:0;z-index:10;display:flex;align-items:center;justify-content:center;background:#06101bdc;padding:20px}
#caixa{width:min(440px,100%);max-height:90vh;overflow:auto;text-align:center;background:linear-gradient(145deg,#18374b,#101725);border:2px solid #48dfff;border-radius:18px;padding:22px;box-shadow:0 0 35px #00cfff44}
h1{font-size:27px;margin:0 0 8px;color:#65eaff;text-shadow:0 0 12px #00bfff}
#caixa p{color:#dceaf2;line-height:1.5;font-size:14px}
.herois{display:grid;grid-template-columns:repeat(2,1fr);gap:9px;margin:16px 0}
.heroi{padding:12px 5px;border:2px solid #456477;border-radius:10px;background:#0c1e2c;color:white;font-weight:bold}
.heroi.selecionado{border-color:#65eaff;background:#14506a}
#jogar{width:100%;padding:15px;border:0;border-radius:12px;background:linear-gradient(90deg,#00bde8,#236be8);color:white;font-weight:bold;font-size:17px}
#minimapa{position:fixed;right:12px;top:90px;width:100px;height:100px;border:2px solid #7deaff;border-radius:9px;background:#173b2cdd;z-index:5;display:none;pointer-events:none}
@media(max-height:500px){#joystick{width:95px;height:95px;bottom:12px;left:12px}#pino{width:39px;height:39px;left:26px;top:26px}.btnJogo{width:54px;height:54px}.botoes{bottom:10px;right:10px}#btnAtaque{width:62px;height:62px}}
</style>
</head>
<body>

<div id="hud">
  <div id="topo">
    <div class="painel">
      <div>❤️ VIDA</div>
      <div id="barraFundo"><div id="barraVida"></div></div>
    </div>
    <div class="painel" id="pontos">Pontos: 0</div>
    <div class="painel" id="inimigosTexto">Inimigos: 8</div>
  </div>
  <div class="painel" id="ajuda">Derrote todos os adversários!</div>
</div>

<canvas id="minimapa" width="100" height="100"></canvas>

<div id="controles">
  <div id="joystick"><div id="pino"></div></div>
  <div class="botoes">
    <button class="btnJogo" id="btnPulo">PULAR</button>
    <div class="linhaBotoes">
      <button class="btnJogo" id="btnEspecial">PODER</button>
      <button class="btnJogo" id="btnAtaque">ATACAR</button>
    </div>
  </div>
</div>

<div id="mensagem">
  <div id="caixa">
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
  let z=deslocamento.y+(teclas.s?1:0)-(teclas.w?1:0);
  const v=new THREE.Vector3(x,0,z);
  if(v.length()>1)v.normalize();
  const frente=new THREE.Vector3(Math.sin(cameraAngulo),0,Math.cos(cameraAngulo));
  const lado=new THREE.Vector3(Math.cos(cameraAngulo),0,-Math.sin(cameraAngulo));
  const movimento=frente.multiplyScalar(v.y).add(lado.multiplyScalar(v.x));
  if(movimento.length()>.01){
   jogador.position.addScaledVector(movimento,7*dt);
   jogador.rotation.y=Math.atan2(movimento.x,movimento.z);
  }
  jogador.position.x=THREE.MathUtils.clamp(jogador.position.x,-78,78);
  jogador.position.z=THREE.MathUtils.clamp(jogador.position.z,-78,78);

  if(pulando){
   jogador.position.y+=velocidadePulo*dt;
   velocidadePulo-=18*dt;
   if(jogador.position.y<=0){jogador.position.y=0;pulando=false;}
  }

  inimigos.forEach(e=>{
   if(!e.vivo)return;
   const dx=jogador.position.x-e.obj.position.x;
   const dz=jogador.position.z-e.obj.position.z;
   const dist=Math.hypot(dx,dz);
   if(dist>1.65){
    e.obj.position.x+=(dx/dist)*e.vel*dt;
    e.obj.position.z+=(dz/dist)*e.vel*dt;
    e.obj.rotation.y=Math.atan2(dx,dz);
   }else if(performance.now()-e.ataque>1100){
    e.ataque=performance.now();
    vida=Math.max(0,vida-7);
    atualizarHUD();
    if(vida<=0)mostrarFim(false);
   }
  });

  const alvo=new THREE.Vector3(
   jogador.position.x-Math.sin(cameraAngulo)*8,
   jogador.position.y+5,
   jogador.position.z-Math.cos(cameraAngulo)*8
  );
  camera.position.lerp(alvo,1-Math.exp(-5*dt));
  camera.lookAt(jogador.position.x,jogador.position.y+1.4,jogador.position.z);
  atualizarMinimapa();
 }else if(!jogador){
  camera.position.set(0,12,18);camera.lookAt(0,0,0);
 }
 renderer.render(cena,camera);
}
animar();

window.addEventListener("resize",()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
</script>
</body>
</html>
