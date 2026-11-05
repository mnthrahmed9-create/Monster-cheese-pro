Monster-cheese-pro
│
├── index.html
│
└── models
    ├── king.glb
    ├── queen.glb
    ├── rook.glb
    ├── bishop.glb
    ├── knight.glb
    └── pawn.glb
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Monster Chess 3D – Jungle</title>
<style>
:root{color-scheme:dark;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
*{box-sizing:border-box}
html,body{margin:0;background:#12301a;color:#fff;font-family:system-ui,"Segoe UI",Tahoma,Arial,sans-serif;text-align:center}
header{padding:6px;background:#1c3f26}
h1{margin:2px;font-size:21px;color:#ffe08a;text-shadow:0 2px 0 #7a4a00}
#game{height:64vh;min-height:380px;width:100%;touch-action:pan-y}
.panel{padding:6px}
button,select{border:0;border-radius:12px;padding:10px 14px;margin:3px;background:#ffc857;color:#171717;font-weight:bold;font-size:15px}
#status{padding:6px;font-size:17px;min-height:34px}
#promotion{display:none}
#moves{font-size:12px;color:#ddd;max-height:44px;overflow:auto;direction:ltr}
</style>
</head>
<body>
<header><h1>🦁 MONSTER CHESS 3D</h1></header>
<div id="game"></div>
<div id="status"></div>
<div id="promotion">ترقية الجندي:
 <button onclick="promote('q')">ملكة</button><button onclick="promote('r')">ماموث</button><button onclick="promote('b')">تنين</button><button onclick="promote('n')">ذئب</button>
</div>
<div class="panel">
 <select id="level"><option value="easy">🟢 سهل</option><option value="hard">🔴 صعب</option></select>
 <button onclick="newGame()">لعبة جديدة</button><button onclick="undoMove()">تراجع</button>
</div>
<div id="moves"></div>

<script type="importmap">
{"imports":{
 "three":"https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js",
 "OrbitControls":"https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/controls/OrbitControls.js",
 "chess":"https://cdn.jsdelivr.net/npm/chess.js@1.4.0/dist/esm/chess.js"}}
</script>
<script type="module">
import * as T from "three";
import {OrbitControls} from "OrbitControls";
import {Chess} from "chess";

const chess=new Chess(),holder=document.getElementById("game"),files="abcdefgh";
const $=id=>document.getElementById(id);
const M=(c,r=.6,m=0)=>new T.MeshStandardMaterial({color:c,roughness:r,metalness:m});
const EM=c=>new T.MeshStandardMaterial({color:c,emissive:c,emissiveIntensity:1.6});

const scene=new T.Scene();
scene.background=new T.Color(0x8fc4e8);
scene.fog=new T.Fog(0x8fc4e8,24,60);
const camera=new T.PerspectiveCamera(42,1,.1,120);
camera.position.set(0,8.2,10.8);
const renderer=new T.WebGLRenderer({antialias:true});
renderer.setPixelRatio(Math.min(devicePixelRatio,1.7));
renderer.shadowMap.enabled=true;
holder.appendChild(renderer.domElement);
const controls=new OrbitControls(camera,renderer.domElement);
controls.target.set(0,.4,-.8);controls.enableDamping=true;controls.enablePan=false;
controls.minDistance=8;controls.maxDistance=18;controls.maxPolarAngle=Math.PI/2.2;

scene.add(new T.HemisphereLight(0xdff0ff,0x2c5a2c,1.7));
const sun=new T.DirectionalLight(0xfff2d0,2.4);sun.position.set(4,11,7);sun.castShadow=true;
Object.assign(sun.shadow.camera,{left:-7,right:7,top:7,bottom:-7});scene.add(sun);

// البيئة: أرض، أشجار، قلعة، شلالات، رايات
const add=(o,x,y,z,p=scene)=>{o.position.set(x,y,z);p.add(o);return o};
const ground=add(new T.Mesh(new T.PlaneGeometry(160,160),M(0x2f6b34,1)),0,-.5,0);ground.rotation.x=-Math.PI/2;ground.receiveShadow=true;
for(let i=0;i<34;i++){
 const a=Math.sin(i*12.9)*43758.5,b=Math.sin(i*78.2)*12345.6;
 const x=(a-Math.floor(a))*40-20,z=(b-Math.floor(b))*20-18;
 if(Math.abs(x)<6.5&&z>-7)continue;
 const t=new T.Group(),h=1+(i%3)*.3;
 add(new T.Mesh(new T.CylinderGeometry(.2,.28,1.4,8),M(0x5a3a1c)),0,.2,0,t);
 add(new T.Mesh(new T.ConeGeometry(1.1*h,2.4*h,10),M(0x1f7a35)),0,2,0,t);
 add(new T.Mesh(new T.ConeGeometry(.8*h,2*h,10),M(0x2b9444)),0,3.2*h,0,t);
 add(t,x,0,z);
}
const castle=new T.Group();
add(new T.Mesh(new T.BoxGeometry(14,3,2),M(0x44425f)),0,1,0,castle);
add(new T.Mesh(new T.BoxGeometry(4,5.5,3),M(0x44425f)),0,2.2,0,castle);
add(new T.Mesh(new T.ConeGeometry(2.4,2.4,4),M(0xb3262e)),0,6.4,0,castle).rotation.y=Math.PI/4;
for(const x of [-6.5,-3.6,3.6,6.5]){
 add(new T.Mesh(new T.CylinderGeometry(.8,.9,5,10),M(0x4d4b6b)),x,1.5,0,castle);
 add(new T.Mesh(new T.ConeGeometry(1.1,1.8,10),M(0xb3262e)),x,4.9,0,castle);
}
add(new T.Mesh(new T.BoxGeometry(1.2,1.8,.1),EM(0xffb347)),0,.4,1.55,castle);
for(const x of [-1,1])add(new T.Mesh(new T.BoxGeometry(.4,.8,.1),EM(0xffb347)),x,3,1.55,castle);
for(const x of [-2.6,2.6])add(new T.Mesh(new T.BoxGeometry(.9,1.8,.08),M(0xc8372d)),x,1.6,1.05,castle);
add(castle,0,0,-15);
for(const s of [-1,1]){
 add(new T.Mesh(new T.BoxGeometry(1.2,5,.4),EM(0x6ab8ff)),s*7.6,2,-4);
 add(new T.Mesh(new T.BoxGeometry(2.2,1.4,1.6),M(0x6b6b78,1)),s*7.6,.2,-3.6);
 const c=add(new T.Mesh(new T.ConeGeometry(.35,1.2,6),EM(s<0?0xff6a5a:0x66a8ff)),s*6.4,.1,3);
 // رايات
 add(new T.Mesh(new T.CylinderGeometry(.05,.05,3.4,6),M(0xddd)),s*5.4,1.2,-4.6);
 add(new T.Mesh(new T.BoxGeometry(.8,1.3,.05),M(s<0?0xb3262e:0x2a52c8)),s*5.4,2,-4.55);
 add(new T.Mesh(new T.SphereGeometry(.18,10,8),EM(0xffc83d)),s*5.4,2.1,-4.5);
}

// الرقعة
const frame=add(new T.Mesh(new T.BoxGeometry(8.9,.34,8.9),M(0xe8b53a,.3,.8)),0,-.2,0);frame.receiveShadow=true;
const LIGHT=0xe9dcc0,DARK=0x2a2a35;
const sqCol=s=>((Number(s[1])+files.indexOf(s[0]))%2?DARK:LIGHT);
const pos=s=>({x:files.indexOf(s[0])-3.5,z:4.5-Number(s[1])});
const squares=[];
for(let r=1;r<=8;r++)for(let f=0;f<8;f++){
 const s=files[f]+r,p=pos(s);
 const m=add(new T.Mesh(new T.BoxGeometry(.99,.1,.99),M(sqCol(s),.5,.1)),p.x,-.01,p.z);
 m.receiveShadow=true;m.userData.square=s;squares.push(m);
}
const pieces=new T.Group(),marks=new T.Group();scene.add(pieces,marks);

// القطع
const pal={w:{arm:0x2f5be0,eye:0x66d9ff,dr:0x3b9bff,wing:0x9fd0ff},b:{arm:0xc62828,eye:0xffa000,dr:0xd9482b,wing:0xff9a6a}};
function build(t,c){
 const g=new T.Group(),p=pal[c];
 const A=M(p.arm,.4,.4),G=M(0xffc83d,.25,.9),E=EM(p.eye);
 const fur=M(0x2b2b35,.9),white=M(0xffffff),black=M(0x111111),steel=M(0xdfe6ee,.2,.9);
 const mk=(geo,mat,x,y,z,rx=0,ry=0,rz=0)=>{const o=new T.Mesh(geo,mat);o.position.set(x,y,z);o.rotation.set(rx,ry,rz);o.castShadow=true;g.add(o);return o};
 const S=(a,b,d,mat,x,y,z,rx,ry,rz)=>{const o=mk(new T.SphereGeometry(1,14,10),mat,x,y,z,rx,ry,rz);o.scale.set(a,b,d);return o};
 const cyl=(a,b,h,mat,x,y,z,rx,ry,rz)=>mk(new T.CylinderGeometry(a,b,h,10),mat,x,y,z,rx,ry,rz);
 const cone=(r,h,mat,x,y,z,rx,ry,rz)=>mk(new T.ConeGeometry(r,h,10),mat,x,y,z,rx,ry,rz);
 const box=(w,h,d,mat,x,y,z,rx,ry,rz)=>mk(new T.BoxGeometry(w,h,d),mat,x,y,z,rx,ry,rz);
 const eyes=(y,z,s,r=.03)=>[-s,s].forEach(x=>S(r,r,r,E,x,y,z));
 const crown=(y,r,n)=>{mk(new T.TorusGeometry(r,.025,8,16),G,0,y,0,Math.PI/2);for(let i=0;i<n;i++){const a=i*Math.PI*2/n;cone(.035,.14,G,Math.sin(a)*r,y+.08,Math.cos(a)*r)}};
 cyl(.4,.44,.1,A,0,.05,0);mk(new T.TorusGeometry(.41,.03,8,24),G,0,.1,0,Math.PI/2);

 if(t==="p"){ // غوريلا
  const face=M(0x55525e);
  S(.26,.28,.2,fur,0,.42,0);S(.22,.2,.12,A,0,.45,.12);
  for(const x of [-1,1]){S(.11,.11,.11,fur,x*.27,.58,0);S(.1,.05,.1,A,x*.27,.66,0);cyl(.07,.06,.3,fur,x*.3,.35,.05);S(.08,.08,.08,fur,x*.3,.2,.1)}
  S(.15,.14,.14,fur,0,.78,.04);S(.09,.08,.05,face,0,.76,.16);eyes(.81,.17,.05,.022);
  box(.04,.42,.015,steel,.32,.3,.18,0,0,-.6);
 }else if(t==="r"){ // ماموث
  const br=M(0x7a5232,.9),tusk=M(0xf3ead2,.4);
  for(const x of [-.17,.17])for(const z of [-.22,.22])cyl(.09,.1,.3,br,x,.25,z);
  S(.36,.3,.5,br,0,.55,0);box(.5,.06,.5,A,0,.84,0);
  box(.34,.2,.34,A,0,.96,-.05);box(.2,.2,.03,G,0,.96,.13);
  for(const x of [-.14,.14])for(const z of [-.19,.09])box(.08,.08,.08,G,x,1.1,z);
  S(.24,.24,.24,br,0,.62,.46);for(const x of [-1,1])S(.04,.18,.14,br,x*.26,.66,.4);
  cyl(.07,.1,.4,br,0,.42,.66,-.2);
  for(const x of [-1,1])cone(.04,.5,tusk,x*.15,.5,.78,1.2);
  eyes(.68,.66,.1,.025);
 }else if(t==="n"){ // ذئب
  const wf=M(0x8a8fa3,.9),mane=M(0x5d6482,.9);
  S(.2,.22,.3,wf,0,.45,-.1);S(.22,.26,.2,wf,0,.58,.1);S(.2,.18,.12,A,0,.58,.2);
  for(const x of [-.12,.12])for(const z of [-.2,.15])cyl(.06,.07,.35,wf,x,.2,z);
  for(const x of [-1,1])S(.09,.05,.09,A,x*.18,.74,.1);
  S(.14,.13,.16,wf,0,.9,.25);cone(.07,.26,wf,0,.86,.46,Math.PI/2);S(.03,.03,.03,black,0,.86,.6);
  for(const x of [-1,1])cone(.05,.18,wf,x*.08,1.05,.2);
  for(let i=0;i<4;i++)cone(.05,.2,mane,0,.98-i*.1,.05-i*.1,-.6);
  cone(.06,.4,wf,0,.45,-.4,-1.2);eyes(.93,.36,.07,.025);
 }else if(t==="b"){ // تنين
  const dc=M(p.dr,.5,.2),wg=M(p.wing,.6),bone=M(0xf3ead2);
  S(.22,.2,.32,dc,0,.4,0);S(.16,.16,.26,M(0xeaf2ff),0,.36,.06);
  cyl(.07,.1,.5,dc,0,.78,.15,.35);S(.11,.09,.15,dc,0,1.04,.28);cone(.05,.18,dc,0,1,.45,Math.PI/2);
  for(const x of [-1,1]){cone(.03,.18,bone,x*.06,1.15,.2,-.5);const w=S(.02,.34,.3,wg,x*.3,.72,-.12,0,0,x*.5)}
  cone(.08,.55,dc,0,.25,-.42,-1.4);eyes(1.07,.34,.06,.022);
  for(let i=0;i<4;i++)cone(.03,.1,bone,0,.6+i*.1,-.05-i*.03,-.5);
 }else if(t==="q"){ // لبؤة/نمر
  const tg=M(0xe0903a,.8),cream=M(0xf3e0b8);
  cone(.34,.7,A,0,.45,0);cyl(.12,.16,.3,A,0,.85,0);mk(new T.TorusGeometry(.14,.03,8,16),G,0,.72,0,Math.PI/2);
  for(const x of [-1,1])cyl(.05,.05,.3,tg,x*.2,.82,.03,0,0,x*.3);
  S(.15,.15,.14,tg,0,1.1,0);S(.07,.05,.05,cream,0,1.06,.12);S(.02,.02,.02,black,0,1.09,.16);
  for(const x of [-1,1]){S(.05,.06,.03,tg,x*.12,1.24,0);for(const y of [1.14,1.19])box(.03,.012,.01,black,x*.1,y,.135)}
  eyes(1.13,.13,.06,.022);crown(1.3,.1,5);
 }else{ // أسد الملك
  const sk=M(0xc98a3a,.8),mane=M(0x8a4b1c,.95);
  box(.6,.7,.04,A,0,.7,-.2,.1);
  for(const x of [-.12,.12])cyl(.1,.12,.35,sk,x,.25,0);
  S(.27,.3,.2,sk,0,.6,0);S(.2,.12,.12,A,0,.66,.12);cyl(.27,.27,.08,G,0,.42,0);
  for(const x of [-1,1]){S(.11,.08,.11,A,x*.3,.82,0);cyl(.07,.08,.35,sk,x*.33,.6,0)}
  S(.24,.24,.16,mane,0,1.05,-.04);S(.14,.14,.14,sk,0,1.05,.08);S(.07,.05,.05,M(0xf3e0b8),0,1,.2);S(.025,.02,.02,black,0,1.03,.24);
  for(const x of [-1,1])S(.04,.04,.03,sk,x*.12,1.2,.05);
  eyes(1.08,.2,.06,.022);crown(1.27,.12,5);
  box(.05,.26,.05,G,0,1.55,0);box(.17,.05,.05,G,0,1.58,0);
 }

 // تفاصيل وحوش إضافية: عضلات، مخالب، دروع، قرون وصفائح جلدية
 const hide=M(0x40343a,.95),ivory=M(0xf1dfbd,.4),darkSteel=M(0x596574,.3,.8);
 if(t==="p"){
   S(.29,.22,.17,hide,0,.58,.12);
   for(const x of [-1,1]){
     S(.17,.24,.16,fur,x*.34,.52,0,0,0,x*.15);
     S(.10,.20,.10,fur,x*.39,.29,.04);
     S(.13,.08,.11,black,x*.39,.15,.07);
     box(.22,.18,.07,steel,x*.34,.70,.12,0,0,x*.15);
     for(let j=-1;j<=1;j++)cone(.025,.12,black,x*.39+j*.05,.10,.15,.15);
   }
   cyl(.035,.035,.48,steel,.49,.42,.18,0,0,-.9);
   cone(.07,.16,steel,.49,.16,.39);
 }else if(t==="r"){
   for(const x of [-1,1]){
     S(.12,.17,.11,fur,x*.23,.22,-.02);
     S(.13,.07,.12,black,x*.23,.10,.02);
     const tusk=mk(new T.TorusGeometry(.16,.045,8,18,Math.PI*.82),ivory,x*.19,.49,.63);
     tusk.rotation.z=x*.35;
   }
   S(.11,.16,.11,fur,0,.34,.69);
   cyl(.075,.10,.28,fur,0,.27,.74,-.35);
   S(.09,.11,.09,fur,0,.16,.80);
 }else if(t==="n"){
   const wolfFur=M(0x626b7d,.95);
   S(.19,.19,.17,wolfFur,0,.45,-.02);
   for(const x of [-1,1]){
     S(.10,.17,.11,wolfFur,x*.17,.30,-.03);
     S(.07,.06,.08,black,x*.17,.12,.04);
     cone(.03,.14,ivory,x*.10,.90,.30,Math.PI);
   }
   for(let j=0;j<5;j++)cone(.045,.18,wolfFur,(j-2)*.09,.77,-.12,-.65);
 }else if(t==="b"){
   for(const x of [-1,1]){
     cone(.045,.23,ivory,x*.10,1.20,.22,-.5);
     cyl(.018,.025,.50,steel,x*.39,.77,-.12,0,0,x*.65);
     for(let j=0;j<3;j++)cone(.025,.18,steel,x*(.38+j*.10),.70,-.25-j*.08,.8);
   }
   for(let j=0;j<5;j++)cone(.04,.16,ivory,0,.57+j*.10,-.20-j*.06,-.5);
 }else if(t==="q"){
   const tawny=M(0xb97832,.9);
   S(.19,.17,.15,tawny,0,.54,-.02);
   for(const x of [-1,1]){
     S(.07,.16,.08,tawny,x*.17,.35,0);
     S(.07,.06,.08,black,x*.17,.17,.05);
   }
   box(.36,.08,.30,G,0,.78,-.04);
   for(let j=0;j<5;j++)cone(.035,.13,G,(j-2)*.09,1.34,0);
 }else if(t==="k"){
   const mane2=M(0x754019,.98);
   S(.31,.25,.20,mane2,0,.92,-.03);
   for(const x of [-1,1]){
     S(.17,.23,.15,sk,x*.27,.56,0,0,0,x*.15);
     S(.12,.19,.11,sk,x*.31,.31,.02);
     S(.13,.07,.12,black,x*.31,.15,.04);
     box(.22,.20,.08,G,x*.29,.67,.12,0,0,x*.12);
   }
   box(.48,.09,.28,G,0,.84,-.02);
   for(let j=-1;j<=1;j++)cone(.05,.17,G,j*.13,1.62,0);
 }

 g.rotation.y=c==="w"?Math.PI:0;
 return g;
}
const tpl={};
const getPiece=(t,c)=>(tpl[t+c]||(tpl[t+c]=build(t,c))).clone(true);

let selected=null,thinking=false,pending=null;
function draw(){
 pieces.clear();
 chess.board().forEach((row,r)=>row.forEach((p,f)=>{
  if(!p)return;
  const s=files[f]+(8-r),o=getPiece(p.type,p.color),q=pos(s);
  o.position.set(q.x,.04,q.z);o.userData={square:s,color:p.color};pieces.add(o);
 }));
}
function highlight(){
 squares.forEach(s=>s.material.color.setHex(sqCol(s.userData.square)));
 marks.clear();
 if(!selected)return;
 squares.find(s=>s.userData.square===selected).material.color.setHex(0xffd166);
 chess.moves({square:selected,verbose:true}).forEach(m=>{
  const p=pos(m.to),cap=m.flags.includes("c")||m.flags.includes("e"),col=cap?0xff4040:0x39dc83;
  const o=new T.Mesh(cap?new T.TorusGeometry(.4,.05,8,24):new T.CylinderGeometry(.16,.16,.04,16),
   new T.MeshStandardMaterial({color:col,emissive:col,emissiveIntensity:1.5}));
  o.position.set(p.x,.07,p.z);if(cap)o.rotation.x=Math.PI/2;marks.add(o);
 });
}
function log(){$("moves").textContent=chess.history().join("  ")}
function status(){
 const s=$("status");
 if(chess.isCheckmate()){s.textContent="🏆 كش مات! "+(chess.turn()==="w"?"الجيش الأحمر":"الجيش الأزرق")+" فاز!";return true}
 if(chess.isDraw()){s.textContent="🤝 تعادل";return true}
 if(chess.isCheck()){s.textContent="⚠️ كش! الملك مهدد";return false}
 s.textContent=chess.turn()==="w"?"دورك - الجيش الأزرق":"الجيش الأحمر يفكر...";return false;
}

// الذكاء الاصطناعي
const val={p:100,n:320,b:330,r:500,q:900,k:20000};
function evaluate(){
 let sc=0;
 chess.board().forEach((row,r)=>row.forEach((p,f)=>{
  if(!p)return;
  let v=val[p.type];
  if(p.type==="p")v+=(p.color==="b"?r:7-r)*4;
  v+=(7-(Math.abs(3.5-f)+Math.abs(3.5-r)))*2;
  sc+=p.color==="b"?v:-v;
 }));
 return sc;
}
function minimax(d,a,b,max){
 if(d===0||chess.isGameOver())return evaluate();
 let best=max?-Infinity:Infinity;
 for(const m of chess.moves()){
  chess.move(m);const v=minimax(d-1,a,b,!max);chess.undo();
  if(max){best=Math.max(best,v);a=Math.max(a,best)}else{best=Math.min(best,v);b=Math.min(b,best)}
  if(b<=a)break;
 }
 return best;
}
function computer(){
 if(chess.isGameOver())return;
 const moves=chess.moves();let pick;
 if($("level").value==="easy"){
  const caps=moves.filter(m=>m.includes("x"));
  const pool=Math.random()<.5&&caps.length?caps:moves;
  pick=pool[Math.floor(Math.random()*pool.length)];
 }else{
  let best=-Infinity;
  for(const m of moves){
   chess.move(m);const v=minimax(2,-Infinity,Infinity,false)+Math.random();chess.undo();
   if(v>best){best=v;pick=m}
  }
 }
 if(pick){chess.move(pick);draw();log();status()}
}

// اللمس
let down=null;
renderer.domElement.addEventListener("pointerdown",e=>down=[e.clientX,e.clientY]);
renderer.domElement.addEventListener("pointerup",e=>{
 if(!down||Math.hypot(e.clientX-down[0],e.clientY-down[1])>6)return;
 tap(e);
});
function tap(e){
 if(thinking||pending||chess.isGameOver()||chess.turn()!=="w")return;
 const rc=renderer.domElement.getBoundingClientRect();
 const ray=new T.Raycaster();
 ray.setFromCamera(new T.Vector2((e.clientX-rc.left)/rc.width*2-1,-(e.clientY-rc.top)/rc.height*2+1),camera);
 let sq=null;
 for(const h of ray.intersectObjects([...pieces.children,...squares],true)){
  let o=h.object;while(o&&!o.userData.square)o=o.parent;
  if(o){sq=o.userData.square;break}
 }
 if(!sq)return;
 const p=chess.get(sq);
 if(p&&p.color==="w"){selected=sq;highlight();return}
 if(!selected)return;
 const mv=chess.moves({square:selected,verbose:true}).find(m=>m.to===sq);
 if(!mv)return;
 if(mv.promotion){pending={from:selected,to:sq};$("promotion").style.display="block";return}
 play(selected,sq,"q");
}
function play(from,to,pr){
 chess.move({from,to,promotion:pr});
 selected=null;highlight();draw();log();
 $("promotion").style.display="none";
 if(status())return;
 thinking=true;
 setTimeout(()=>{computer();thinking=false},350);
}
window.promote=t=>{if(!pending)return;const p=pending;pending=null;play(p.from,p.to,t)};
window.newGame=()=>{chess.reset();selected=null;pending=null;thinking=false;$("promotion").style.display="none";highlight();draw();log();status()};
window.undoMove=()=>{
 if(thinking||chess.history().length<2)return;
 chess.undo();chess.undo();selected=null;highlight();draw();log();status();
};

function resize(){
 const w=holder.clientWidth,h=holder.clientHeight;
 renderer.setSize(w,h);camera.aspect=w/h;camera.updateProjectionMatrix();
}
addEventListener("resize",resize);resize();
draw();status();
renderer.setAnimationLoop(t=>{
 pieces.children.forEach(o=>{o.position.y=(selected&&o.userData.square===selected)?.2+Math.sin(t/150)*.05:.04});
 controls.update();renderer.render(scene,camera);
});
</script>
</body>
</html>
