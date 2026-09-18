<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>ALASTOR STRIKE</title>

<style>
*{
  box-sizing:border-box;
  margin:0;
  padding:0;
  -webkit-user-select:none;
  user-select:none;
}

html,body{
  width:100%;
  height:100%;
  overflow:hidden;
  background:#10151b;
  font-family:Arial,sans-serif;
  touch-action:none;
}

#game{
  position:fixed;
  inset:0;
}

canvas{
  display:block;
}

#hud{
  position:fixed;
  top:15px;
  left:15px;
  color:white;
  z-index:5;
  text-shadow:0 2px 4px #000;
  font-size:18px;
  line-height:1.45;
  pointer-events:none;
}

.title{
  font-size:25px;
  font-weight:bold;
  letter-spacing:3px;
  margin-bottom:5px;
}

#mission{
  position:fixed;
  top:18px;
  left:50%;
  transform:translateX(-50%);
  color:white;
  background:rgba(0,0,0,.45);
  padding:8px 18px;
  border-radius:20px;
  font-size:14px;
  z-index:5;
  text-align:center;
  pointer-events:none;
}

#crosshair{
  position:fixed;
  left:50%;
  top:50%;
  transform:translate(-50%,-50%);
  color:white;
  font-size:28px;
  z-index:4;
  text-shadow:0 2px 4px #000;
  pointer-events:none;
}

#joystick{
  position:fixed;
  width:145px;
  height:145px;
  left:28px;
  bottom:28px;
  border:3px solid rgba(255,255,255,.25);
  background:rgba(0,0,0,.20);
  border-radius:50%;
  z-index:10;
}

#stick{
  position:absolute;
  width:65px;
  height:65px;
  left:37px;
  top:37px;
  border-radius:50%;
  background:rgba(255,255,255,.25);
  border:2px solid rgba(255,255,255,.35);
}

.button{
  position:fixed;
  z-index:10;
  width:82px;
  height:82px;
  border-radius:50%;
  border:2px solid rgba(255,255,255,.3);
  background:rgba(20,25,30,.72);
  color:white;
  font-size:14px;
  font-weight:bold;
  display:flex;
  align-items:center;
  justify-content:center;
}

#jump{
  right:28px;
  bottom:125px;
}

#sprint{
  right:125px;
  bottom:28px;
}

#action{
  right:28px;
  bottom:28px;
}

#menu{
  position:fixed;
  inset:0;
  z-index:20;
  display:flex;
  align-items:center;
  justify-content:center;
  background:
    radial-gradient(circle at center,rgba(40,60,80,.4),rgba(0,0,0,.92));
  color:white;
  text-align:center;
}

.panel{
  width:min(90%,430px);
  padding:35px 25px;
  border:1px solid rgba(255,255,255,.18);
  border-radius:22px;
  background:rgba(10,15,20,.78);
  backdrop-filter:blur(10px);
  box-shadow:0 20px 60px rgba(0,0,0,.6);
}

.logo{
  font-size:38px;
  letter-spacing:5px;
  font-weight:900;
  margin-bottom:8px;
}

.sub{
  opacity:.7;
  margin-bottom:28px;
}

#play{
  border:0;
  border-radius:14px;
  padding:16px 45px;
  font-size:18px;
  font-weight:bold;
  background:white;
  color:#111;
}

#message{
  position:fixed;
  left:50%;
  bottom:190px;
  transform:translateX(-50%);
  z-index:8;
  color:white;
  background:rgba(0,0,0,.55);
  padding:10px 18px;
  border-radius:20px;
  opacity:0;
  transition:.3s;
  pointer-events:none;
}

#pause{
  position:fixed;
  right:15px;
  top:15px;
  z-index:10;
  width:45px;
  height:45px;
  border-radius:12px;
  border:1px solid rgba(255,255,255,.3);
  background:rgba(0,0,0,.45);
  color:white;
  font-size:20px;
}
</style>
</head>

<body>

<div id="game"></div>

<div id="hud">
  <div class="title">ALASTOR</div>
  <div>❤️ HP: <span id="hp">100</span></div>
  <div>⭐ SCORE: <span id="score">0</span></div>
  <div>🔷 ENERGY: <span id="energy">100</span></div>
</div>

<div id="mission">MISSION: Explore the city</div>
<div id="crosshair">+</div>
<div id="message"></div>

<button id="pause">Ⅱ</button>

<div id="joystick">
  <div id="stick"></div>
</div>

<div id="jump" class="button">JUMP</div>
<div id="sprint" class="button">RUN</div>
<div id="action" class="button">ACTION</div>

<div id="menu">
  <div class="panel">
    <div class="logo">ALASTOR</div>
    <div class="sub">STRIKE // CITY OPERATIONS</div>
    <p style="margin-bottom:25px;opacity:.8">
      Explore the city, complete objectives and collect energy cores.
    </p>
    <button id="play">START GAME</button>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>

<script>
/* =========================
   ALASTOR STRIKE
   3D MOBILE ACTION GAME
   ========================= */

const scene = new THREE.Scene();

scene.background = new THREE.Color(0x91a9b5);

scene.fog = new THREE.Fog(0x91a9b5,45,180);

const camera = new THREE.PerspectiveCamera(
  72,
  innerWidth/innerHeight,
  .1,
  400
);

camera.position.set(0,3,8);

const renderer = new THREE.WebGLRenderer({
  antialias:true,
  powerPreference:"high-performance"
});

renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,1.7));
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;

document.getElementById("game").appendChild(renderer.domElement);

/* LIGHTING */

const hemi = new THREE.HemisphereLight(
  0xddeeff,
  0x34402f,
  2
);

scene.add(hemi);

const sun = new THREE.DirectionalLight(
  0xffffff,
  3
);

sun.position.set(-40,70,30);
sun.castShadow=true;

sun.shadow.mapSize.width=1024;
sun.shadow.mapSize.height=1024;

scene.add(sun);

/* GROUND */

const groundMat = new THREE.MeshStandardMaterial({
  color:0x4d5b50,
  roughness:1
});

const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(320,320),
  groundMat
);

ground.rotation.x=-Math.PI/2;
ground.receiveShadow=true;
scene.add(ground);

/* ROAD */

function box(w,h,d,color,x,y,z){

  const mesh = new THREE.Mesh(
    new THREE.BoxGeometry(w,h,d),
    new THREE.MeshStandardMaterial({
      color:color,
      roughness:.8
    })
  );

  mesh.position.set(x,y,z);
  mesh.castShadow=true;
  mesh.receiveShadow=true;

  scene.add(mesh);

  return mesh;
}

box(20,.08,320,0x292d31,0,.04,0);
box(320,.08,20,0x292d31,0,.05,0);

/* ROAD MARKINGS */

for(let z=-140;z<140;z+=12){
  box(.4,.04,6,0xd7d3a4,0,.1,z);
}

for(let x=-140;x<140;x+=12){
  box(6,.04,.4,0xd7d3a4,x,.1,0);
}

/* BUILDINGS */

const buildings=[];

function building(x,z,w,d,h,color){

  const b=box(w,h,d,color,x,h/2,z);

  buildings.push(b);

  /* windows */

  for(let yy=4;yy<h-1;yy+=5){

    for(let xx=-w/2+2;xx<w/2-1;xx+=4){

      const win=box(
        1.4,
        2,
        .08,
        0x8eb4c7,
        x+xx,
        yy,
        z-d/2-.05
      );

      win.material.metalness=.2;
    }
  }

  return b;
}

building(-35,-35,22,24,32,0x59616a);
building(38,-38,26,22,42,0x464d56);
building(-42,38,28,25,27,0x667078);
building(42,42,25,30,36,0x505861);

building(-75,-10,24,28,24,0x59636b);
building(75,12,30,25,31,0x4c565f);

building(-75,65,25,24,38,0x5a636b);
building(72,-70,28,28,34,0x4e575e);

/* SMALL PROPS */

for(let i=0;i<35;i++){

  const x=(Math.random()-.5)*260;
  const z=(Math.random()-.5)*260;

  if(Math.abs(x)<15 || Math.abs(z)<15) continue;

  box(
    1.2,
    2.5,
    1.2,
    0x20272b,
    x,
    1.25,
    z
  );
}

/* PLAYER */

const player = new THREE.Group();

const body = box(
  1.2,
  1.8,
  .7,
  0x315b75,
  0,
  1.9,
  0
);

body.position.set(0,1.9,0);
player.add(body);

const head = new THREE.Mesh(
  new THREE.SphereGeometry(.43,16,12),
  new THREE.MeshStandardMaterial({
    color:0xc49a78
  })
);

head.position.y=3.05;
head.castShadow=true;
player.add(head);

const backpack=box(
  .8,
  1,
  .35,
  0x202b31,
  0,
  2,
  .48
);

player.add(backpack);

player.position.set(0,0,8);
scene.add(player);

/* ROBOTS */

const robots=[];

function createRobot(x,z){

  const r=new THREE.Group();

  const torso=box(
    1.3,
    1.8,
    .8,
    0x6c7278,
    0,
    1.5,
    0
  );

  r.add(torso);

  const headR=new THREE.Mesh(
    new THREE.BoxGeometry(.9,.7,.8),
    new THREE.MeshStandardMaterial({
      color:0x30383d
    })
  );

  headR.position.y=2.75;
  headR.castShadow=true;

  r.add(headR);

  const eye=box(
    .55,
    .08,
    .08,
    0x8bd5ff,
    0,
    2.75,
    -.42
  );

  r.add(eye);

  r.position.set(x,0,z);

  scene.add(r);

  robots.push({
    mesh:r,
    startX:x,
    startZ:z,
    phase:Math.random()*10
  });
}

createRobot(18,-20);
createRobot(-20,-25);
createRobot(25,25);
createRobot(-28,27);
createRobot(55,-5);
createRobot(-55,8);

/* ENERGY CORES */

const cores=[];

function createCore(x,z){

  const core=new THREE.Mesh(
    new THREE.OctahedronGeometry(.65),
    new THREE.MeshStandardMaterial({
      color:0x54d9ff,
      emissive:0x1a7890,
      emissiveIntensity:1.5,
      metalness:.5,
      roughness:.2
    })
  );

  core.position.set(x,1.2,z);
  core.castShadow=true;

  scene.add(core);

  cores.push(core);
}

createCore(28,-8);
createCore(-30,-5);
createCore(35,35);
createCore(-35,35);
createCore(60,50);
createCore(-65,-55);

/* CONTROLS */

let moveX=0;
let moveY=0;

let sprinting=false;
let jumping=false;
let velocityY=0;

const joystick=document.getElementById("joystick");
const stick=document.getElementById("stick");

let joyActive=false;

function joystickMove(clientX,clientY){

  const rect=joystick.getBoundingClientRect();

  let x=clientX-(rect.left+rect.width/2);
  let y=clientY-(rect.top+rect.height/2);

  const max=48;

  const len=Math.sqrt(x*x+y*y);

  if(len>max){

    x=x/len*max;
    y=y/len*max;

  }

  stick.style.transform=
    `translate(${x}px,${y}px)`;

  moveX=x/max;
  moveY=y/max;
}

joystick.addEventListener("touchstart",e=>{
  joyActive=true;
  joystickMove(
    e.touches[0].clientX,
    e.touches[0].clientY
  );
},{passive:false});

joystick.addEventListener("touchmove",e=>{
  if(joyActive){
    joystickMove(
      e.touches[0].clientX,
      e.touches[0].clientY
    );
  }
},{passive:false});

joystick.addEventListener("touchend",()=>{
  joyActive=false;
  moveX=0;
  moveY=0;
  stick.style.transform="translate(0,0)";
});

/* CAMERA LOOK */

let lookStartX=0;
let lookStartY=0;
let looking=false;

renderer.addEventListener("touchstart",e=>{

  if(e.touches.length!==1) return;

  const t=e.touches[0];

  if(t.clientX<innerWidth*.45) return;

  lookStartX=t.clientX;
  lookStartY=t.clientY;
  looking=true;

});

renderer.addEventListener("touchmove",e=>{

  if(!looking) return;

  const t=e.touches[0];

  const dx=t.clientX-lookStartX;
  const dy=t.clientY-lookStartY;

  player.rotation.y-=dx*.004;

  camera.rotation.x-=dy*.002;

  camera.rotation.x=
    Math.max(
      -0.8,
      Math.min(.8,camera.rotation.x)
    );

  lookStartX=t.clientX;
  lookStartY=t.clientY;

});

renderer.addEventListener("touchend",()=>{
  looking=false;
});

/* BUTTONS */

document.getElementById("sprint")
.addEventListener("touchstart",e=>{
  e.preventDefault();
  sprinting=true;
});

document.getElementById("sprint")
.addEventListener("touchend",()=>{
  sprinting=false;
});

document.getElementById("jump")
.addEventListener("touchstart",e=>{
  e.preventDefault();

  if(player.position.y<=.05){
    velocityY=8;
    jumping=true;
  }
});

let score=0;
let hp=100;
let energy=100;
let collected=0;
let gameStarted=false;
let paused=false;

/* ACTION */

document.getElementById("action")
.addEventListener("touchstart",e=>{

  e.preventDefault();

  energy=Math.max(0,energy-5);

  showMessage("ACTION COMPLETE");

  score+=10;

});

/* MESSAGE */

function showMessage(text){

  const m=document.getElementById("message");

  m.textContent=text;
  m.style.opacity=1;

  clearTimeout(window.messageTimer);

  window.messageTimer=setTimeout(()=>{
    m.style.opacity=0;
  },1600);
}

/* UPDATE ROBOTS */

function updateRobots(time){

  robots.forEach(r=>{

    r.mesh.position.x=
      r.startX+
      Math.sin(time*.0007+r.phase)*4;

    r.mesh.position.z=
      r.startZ+
      Math.cos(time*.0006+r.phase)*4;

    r.mesh.rotation.y=
      Math.sin(time*.0005+r.phase);

  });

}

/* COLLECT CORES */

function checkCores(){

  cores.forEach((core,index)=>{

    if(!core.visible) return;

    const d=core.position.distanceTo(player.position);

    if(d<2.5){

      core.visible=false;

      collected++;
      score+=100;
      energy=Math.min(100,energy+15);

      showMessage("ENERGY CORE COLLECTED!");

      if(collected===cores.length){

        document.getElementById("mission").textContent=
          "MISSION COMPLETE — CITY SECURED";

        score+=500;

      }

    }

  });

}

/* PLAYER MOVEMENT */

function updatePlayer(delta){

  const speed=
    sprinting && energy>0 ? 12 : 7;

  if(sprinting && (moveX!==0 || moveY!==0)){
    energy=Math.max(0,energy-delta*10);
  }else{
    energy=Math.min(100,energy+delta*5);
  }

  const forward=new THREE.Vector3(
    0,
    0,
    -1
  );

  const right=new THREE.Vector3(
    1,
    0,
    0
  );

  forward.applyQuaternion(player.quaternion);
  right.applyQuaternion(player.quaternion);

  const direction=new THREE.Vector3();

  direction.addScaledVector(
    right,
    moveX
  );

  direction.addScaledVector(
    forward,
    -moveY
  );

  if(direction.lengthSq()>0){
    direction.normalize();

    player.position.addScaledVector(
      direction,
      speed*delta
    );
  }

  /* jump physics */

  velocityY-=20*delta;

  player.position.y+=velocityY*delta;

  if(player.position.y<0){

    player.position.y=0;
    velocityY=0;
    jumping=false;

  }

  /* world boundaries */

  player.position.x=
    THREE.MathUtils.clamp(
      player.position.x,
      -145,145
    );

  player.position.z=
    THREE.MathUtils.clamp(
      player.position.z,
      -145,145
    );

}

/* CAMERA */

function updateCamera(){

  const offset=new THREE.Vector3(
    0,
    4.2,
    7
  );

  offset.applyQuaternion(player.quaternion);

  const target=
    player.position.clone().add(offset);

  camera.position.lerp(
    target,
    .12
  );

  const look=
    player.position.clone();

  look.y+=2;

  camera.lookAt(look);

  /* preserve touch vertical camera rotation */

  camera.rotation.x+=
    THREE.MathUtils.clamp(
      camera.rotation.x,
      -.7,.7
    )*.03;
}

/* HUD */

function updateHUD(){

  document.getElementById("hp").textContent=
    Math.round(hp);

  document.getElementById("score").textContent=
    score;

  document.getElementById("energy").textContent=
    Math.round(energy);

}

/* PAUSE */

document.getElementById("pause")
.addEventListener("click",()=>{

  paused=!paused;

  document.getElementById("pause")
    .textContent=paused?"▶":"Ⅱ";

});

/* START */

document.getElementById("play")
.addEventListener("click",()=>{

  gameStarted=true;

  document.getElementById("menu")
    .style.display="none";

  showMessage("MISSION STARTED");

});

/* RESIZE */

addEventListener("resize",()=>{

  camera.aspect=
    innerWidth/innerHeight;

  camera.updateProjectionMatrix();

  renderer.setSize(
    innerWidth,
    innerHeight
  );

});

/* ANIMATION */

let last=performance.now();

function animate(time){

  requestAnimationFrame(animate);

  const delta=
    Math.min(
      (time-last)/1000,
      .05
    );

  last=time;

  if(gameStarted && !paused){

    updatePlayer(delta);
    updateRobots(time);
    checkCores();
    updateHUD();

  }

  /* floating energy cores */

  cores.forEach((c,i)=>{

    if(c.visible){

      c.rotation.y+=delta*2;
      c.rotation.x+=delta;

      c.position.y=
        1.2+
        Math.sin(time*.003+i)*.25;

    }

  });

  updateCamera();

  renderer.render(
    scene,
    camera
  );

}

animate(performance.now());

</script>

</body>
</html>
