<!doctype html><html lang="de"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover"><title>Stahlfront</title>
<link href="https://fonts.googleapis.com/css2?family=Saira+Condensed:wght@500;700&display=swap" rel="stylesheet">
<style>
:root{color-scheme:dark;--bg:#1d2219;--pn:#262d21;--ln:#4a5640;--tx:#e8e4d0;--me:#6bb6e8;--en:#e0653f;--ac:#d8b04a}
html,body{height:100%;margin:0;background:var(--bg);color:var(--tx);font:500 16px 'Saira Condensed','Arial Narrow',sans-serif;overflow:hidden;touch-action:none;user-select:none;-webkit-user-select:none}
canvas{position:fixed;inset:0;width:100%;height:100%;display:block}
#top,#bot{position:fixed;left:0;right:0;display:flex;flex-wrap:wrap;gap:6px;align-items:center;padding:6px 8px;background:rgba(29,34,25,.88)}
#top{top:0;padding-top:calc(6px + env(safe-area-inset-top,0px));justify-content:space-between}
#bot{bottom:0;padding-bottom:calc(6px + env(safe-area-inset-bottom,0px))}
#res{display:flex;gap:12px;font-weight:700;color:var(--ac)}
.b{background:var(--pn);border:1px solid var(--ln);color:var(--tx);font:inherit;padding:7px 10px;border-radius:4px;cursor:pointer}
.b:disabled{opacity:.4}#upn{position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);width:min(440px,94vw);max-height:70vh;overflow:auto;touch-action:pan-y;display:none;flex-direction:column;gap:6px;padding:12px;background:rgba(29,34,25,.97);border:1px solid var(--ln);border-radius:8px;z-index:5}.ur,.ut{display:flex;justify-content:space-between;align-items:center;gap:8px}.ur i{color:#d8b04a;font-style:normal}.b:focus-visible{outline:2px solid var(--ac)}
#ov{position:fixed;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:14px;background:rgba(20,24,17,.93);text-align:center;padding:24px}
#ov h1{font-size:clamp(44px,12vw,92px);margin:0;letter-spacing:.04em;color:var(--ac);line-height:1}
#ov p{max-width:440px;margin:0;line-height:1.35;color:#c9c5b0}
#ov .b{font-size:22px;padding:10px 28px;background:var(--ac);color:#1d2219;border:0;font-weight:700}
</style><script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script></head><body>
<canvas id="g"></canvas><canvas id="c"></canvas>
<div id="top"><div id="res"></div><button class="b" id="rs" onclick="research()"></button></div>
<div id="bot"></div><div id="upn"></div>
<div id="ov"><h1>Stahlfront</h1><p>Führe deine Armee über eine große Weltkarte. Sammler fördern Rohstoffe, Upgrades machen alles stärker (und jedes teurer). Drei Gegner bauen und verbessern sich ebenfalls, nur etwas langsamer als du. Zerstöre alle gegnerischen Hauptquartiere. Dein Spiel wird automatisch gespeichert.</p><p>Touch: Tippen wählt und befiehlt, Ziehen verschiebt, zwei Finger zoomen. Maus: Linke Taste wählt (Rahmen ziehen), rechte Taste befiehlt, Rad zoomt, WASD verschiebt.</p><button class="b" onclick="start()">Neues Spiel</button><button class="b" id="ld" style="display:none" onclick="load()">Spielstand laden</button></div>
<script>
const $=id=>document.getElementById(id),C=$('c'),X=C.getContext('2d'),W=4800,H=3200;
let ents=[],proj=[],fx=[],sel=[],bm=null,over=null,tm=0,run=0,z=.6,cam={x:0,y:0},dpr=1,mx=0,my=0,lastH='',msg='',msgT=0,aiT=2,wave=0;
const TY={hq:{b:1,hp:900,r:46,n:'Hauptquartier',g:'★'},
bar:{b:1,hp:420,r:32,n:'Kaserne',c:[150,50,0],g:'I'},fac:{b:1,hp:520,r:36,n:'Fabrik',c:[250,150,0],g:'⚙'},
tur:{b:1,hp:320,r:20,n:'Geschützturm',c:[120,80,0],rng:240,dmg:14,cd:.7,g:'◎'},
mine:{b:1,hp:300,r:24,n:'Mine',c:[120,60,0]},ref:{b:1,hp:380,r:30,n:'Raffinerie',c:[200,100,20]},
inf:{hp:60,r:8,sp:62,rng:95,dmg:6,cd:.6,c:[50,0,0],tt:4,n:'Infanterie',from:'bar'},
tnk:{hp:260,r:14,sp:82,rng:150,dmg:22,cd:1.1,c:[120,60,20],tt:8,n:'Panzer',from:'fac'},
art:{hp:110,r:12,sp:46,rng:340,dmg:34,cd:2.4,c:[150,80,30],tt:10,n:'Artillerie',from:'fac'},
hrv:{hp:220,r:12,sp:70,rng:0,dmg:0,cd:9,c:[90,20,0],tt:6,n:'Arbeiter',from:'hq'}};
const R=[0,1,2,3].map(()=>({m:500,s:200,f:100,r:0,dm:1,lv:0,u:{}}));
const S=[{x:900,y:2450,o:0,p:100,n:'Heimathafen'},{x:3900,y:750,o:1,p:-100,n:'Nordostfestung'},{x:3900,y:2450,o:2,p:-100,n:'Südostbastion'},{x:900,y:750,o:3,p:-100,n:'Nordwestwacht'},{x:2400,y:1600,o:-1,p:0,n:'Zentralpass'},{x:1600,y:1600,o:-1,p:0,n:'Ölfeld'},{x:3200,y:1600,o:-1,p:0,n:'Stahlwerk'},{x:2400,y:600,o:-1,p:0,n:'Nordgrat'},{x:2400,y:2600,o:-1,p:0,n:'Südhafen'},{x:1500,y:2200,o:-1,p:0,n:'Küstenlinie'},{x:3300,y:1000,o:-1,p:0,n:'Hochebene'}];
const COL=['#3fb4ff','#ff5a3c','#3fd36a','#e8b92a'],aff=(k,c)=>R[k].m>=c[0]&&R[k].s>=c[1]&&R[k].f>=c[2],pay=(k,c)=>{R[k].m-=c[0];R[k].s-=c[1];R[k].f-=c[2]};
function mk(t,team,x,y){const d=TY[t],hh=d.hp*(1+.15*(R[team].u[d.b?'bh':t+'_h']||0)),e={t,team,x,y,hp:hh,mx:hh,b:d.b,r:d.r,a:0,cd:0,q:[],pt:0};ents.push(e);return e}
function rs(){dpr=devicePixelRatio||1;C.width=innerWidth*dpr;C.height=innerHeight*dpr;if(window.REN){REN.setSize(innerWidth,innerHeight);CAM.aspect=innerWidth/innerHeight;CAM.updateProjectionMatrix()}}
addEventListener('resize',rs);rs();
function start(){$('ov').style.display='none';ents=[];proj=[];sel=[];tm=0;over=null;wave=0;R.forEach(r=>Object.assign(r,{m:500,s:200,f:100,r:0,dm:1,lv:0,u:{}}));
S.forEach((s,i)=>{s.o=i<4?i:-1;s.p=i==0?100:i<4?-100:0;s.lt=i<4?i:0});
NODES=genNodes();AIS=[0,1,2,3].map(()=>({t:2,w:0,u:60}));const h0=mk('hq',0,HQP[0][0],HQP[0][1]);HQP.forEach((p,k)=>{if(k)mk('hq',k,p[0],p[1]);const sx=k==0||k==3?1:-1,sy=k==0||k==2?-1:1;mk('bar',k,p[0]+sx*170,p[1]+sy*90);for(let i=0;i<3;i++)mk('inf',k,p[0]+sx*(100+i*24),p[1]+sy*140);mk('hrv',k,p[0]+sx*120,p[1]-sy*130);mk('hrv',k,p[0]+sx*60,p[1]-sy*170)});

cam.x=h0.x+200;cam.y=h0.y-150;cam.d=1500;if(!window.REN)init3();run=1;ui()}
function zoomAt(sx,sy,f){cam.d/=f}
function nearest(e,rng){let b=null,bd=rng*rng;for(const o of ents){if((o.team>0)==(e.team>0)||o.hp<=0)continue;const d=(o.x-e.x)**2+(o.y-e.y)**2;if(d<bd){bd=d;b=o}}return b}
function step(e,tx,ty,dt){const dx=tx-e.x,dy=ty-e.y,d=Math.hypot(dx,dy);if(d<2)return 0;const v=Math.min(d,TY[e.t].sp*dt*(1+.08*(R[e.team].u.mv||0)));e.x+=dx/d*v;e.y+=dy/d*v;e.a=Math.atan2(dy,dx);return d}
function fire(e,tg){const d=TY[e.t];e.a=Math.atan2(tg.y-e.y,tg.x-e.x);tg.hp-=d.dmg*R[e.team].dm*(1+.15*(R[e.team].u[e.t+'_a']||0));e.cd=d.cd;proj.push({x1:e.x,y1:e.y,x2:tg.x,y2:tg.y,l:.12,c:COL[e.team],w:e.t=='art'?4:2.5});if(e.t=='art')fx.push({x:tg.x,y:tg.y,l:.35})}
function update(dt){
tm+=dt;
for(let k=0;k<4;k++){const r=R[k],m=(k?.8+Math.min(.5,tm/1800):1)*(1+.15*(r.u.eco||0)),n=S.filter(s=>s.o==k).length;const mn=ents.filter(e=>e.team==k&&e.t=='mine').length;r.m+=(2+n*3+mn*3)*m*dt;r.s+=(.5+n+mn)*m*dt;r.f+=(.2+n*.6+mn*.5)*m*dt;r.r+=(.15+n*.12)*m*dt}
for(const s of S){let a=0,b=0;for(const e of ents)if(!e.b&&e.t!='hrv'&&(e.x-s.x)**2+(e.y-s.y)**2<22500){if(e.team){b++;s.lt=e.team}else a++}
if(a>b)s.p=Math.min(100,s.p+(a-b)*12*dt);else if(b>a)s.p=Math.max(-100,s.p-(b-a)*12*dt);
if(s.p>=100)s.o=0;else if(s.p<=-100)s.o=s.lt||1;else if((s.o==0&&s.p<=0)||(s.o>0&&s.p>=0))s.o=-1}
for(const e of ents){if(e.hp<=0)continue;const d=TY[e.t];e.cd-=dt;
if(e.b){if(e.t=='tur'&&e.cd<=0){const g=nearest(e,d.rng);if(g)fire(e,g)}
if(e.q.length){e.pt+=dt*(1+.12*(R[e.team].u.spd||0));const u=TY[e.q[0]];if(e.pt>=u.tt){e.pt=0;const a=Math.random()*6.28,n=mk(e.q.shift(),e.team,e.x+Math.cos(a)*(e.r+16),e.y+Math.sin(a)*(e.r+16));if(e.rally)n.mv={x:e.rally.x+(Math.random()-.5)*40,y:e.rally.y+(Math.random()-.5)*40}}}continue}
if(e.t=='hrv'){harv(e,dt);continue}if(e.tg&&e.tg.hp<=0)e.tg=null;
const tg=e.tg||nearest(e,d.rng*1.25);
if(tg){const dist=Math.hypot(tg.x-e.x,tg.y-e.y)-tg.r*.5;
if(dist<=d.rng){if(e.cd<=0)fire(e,tg);if(e.mv&&!e.tg&&!e.am&&step(e,e.mv.x,e.mv.y,dt)<8)e.mv=null}
else if(e.tg||e.am||!e.mv)step(e,tg.x,tg.y,dt);else if(step(e,e.mv.x,e.mv.y,dt)<8)e.mv=null}
else if(e.mv&&step(e,e.mv.x,e.mv.y,dt)<8){e.mv=null;e.am=false}}
const U=ents.filter(e=>!e.b&&e.hp>0);
for(let i=0;i<U.length;i++)for(let j=i+1;j<U.length;j++){const a=U[i],b=U[j],dx=b.x-a.x,dy=b.y-a.y,m=a.r+b.r,d2=dx*dx+dy*dy;if(d2<m*m&&d2>.01){const d=Math.sqrt(d2),p=(m-d)/2/d;a.x-=dx*p;a.y-=dy*p;b.x+=dx*p;b.y+=dy*p}}
ents.forEach(e=>{if(e.hp<=0)fx.push({x:e.x,y:e.y,l:.35})});ents=ents.filter(e=>e.hp>0);sel=sel.filter(e=>e.hp>0);
if(!over){if(!ents.some(e=>e.t=='hq'&&e.team>0))over='Sieg – feindliches Hauptquartier zerstört';else if(!ents.some(e=>e.t=='hq'&&e.team==0))over='Niederlage – dein Hauptquartier ist gefallen'}
proj.forEach(p=>p.l-=dt);proj=proj.filter(p=>p.l>0);fx.forEach(p=>p.l-=dt);fx=fx.filter(p=>p.l>0);
if(!over)ai(dt)}
function okp(x,y,r){if(x<80||y<80||x>W-80||y>H-80)return false;return ents.every(o=>!o.b||Math.hypot(o.x-x,o.y-y)>r+o.r+12)}
function ai(dt){for(let k=1;k<4;k++){const A=AIS[k];if(!A)continue;A.t-=dt;if(A.t>0)continue;A.t=2.6;const B=ents.filter(e=>e.team==k&&e.b),hq=B.find(e=>e.t=='hq');if(!hq)continue;
const c2=t=>B.filter(e=>e.t==t).length,E=R[k],un=ents.filter(e=>e.team==k&&!e.b),hv=un.filter(u=>u.t=='hrv'),ar=un.filter(u=>u.t!='hrv');
const tryB=t=>{const c=TY[t].c;if(!aff(k,c))return;for(let i=0;i<14;i++){const a=Math.random()*6.28,r=130+Math.random()*190,x=hq.x+Math.cos(a)*r,y=hq.y+Math.sin(a)*r;if(okp(x,y,TY[t].r)){pay(k,c);mk(t,k,x,y);return}}};
if(hv.length+hq.q.length<2+(tm>420?1:0)+(tm>900?1:0)&&hq.q.length<1&&aff(k,TY.hrv.c)){pay(k,TY.hrv.c);hq.q.push('hrv')}
else if(c2('bar')<1)tryB('bar');else if(c2('fac')<1)tryB('fac');else if(c2('tur')<2)tryB('tur');else if(c2('mine')<1+(tm>360?1:0))tryB('mine');else if(c2('ref')<1&&tm>300)tryB('ref');else if(c2('bar')<1+(tm>360?1:0))tryB('bar');else if(c2('fac')<1+(tm>600?1:0))tryB('fac');else if(c2('tur')<Math.min(8,2+tm/130|0))tryB('tur');
for(const b of B){if(b.q.length)continue;const t=b.t=='bar'?'inf':b.t=='fac'?(Math.random()<.6?'tnk':'art'):null;if(t&&aff(k,TY[t].c)){pay(k,TY[t].c);b.q.push(t)}}
A.u-=2.6;if(A.u<=0){A.u=40+Math.random()*25;const ok=UPG.filter(u=>{const c=ucost(k,u.id);return ulv(k,u.id)<10&&E.m>=c[0]&&E.s>=c[1]&&E.f>=c[2]&&E.r>=c[3]});if(ok.length)buyU(k,ok[Math.random()*ok.length|0].id)}
const thr=ents.find(e=>e.team==0&&!e.b&&Math.hypot(e.x-hq.x,e.y-hq.y)<500);if(thr){ar.forEach(u=>{if(!u.tg){u.mv={x:thr.x,y:thr.y};u.am=true}});continue}
const idle=ar.filter(u=>!u.mv);if(idle.length>=Math.min(26,5+A.w*2)){A.w++;const dh=(a,b)=>Math.hypot(a.x-hq.x,a.y-hq.y)-Math.hypot(b.x-hq.x,b.y-hq.y),fs=S.filter(s=>s.o<=0).sort(dh)[0],pb=ents.filter(e=>e.team==0&&e.b).sort(dh)[0],tg=fs&&Math.random()<.55?fs:(pb||fs);if(tg)idle.forEach(u=>{u.mv={x:tg.x+(Math.random()-.5)*100,y:tg.y+(Math.random()-.5)*100};u.am=true})}}}
function ai0(dt){aiT-=dt;if(aiT>0)return;aiT=2;const B=ents.filter(e=>e.team==1&&e.b),cn=t=>B.filter(e=>e.t==t).length,hq=B.find(e=>e.t=='hq');if(!hq)return;
const tryB=t=>{const c=TY[t].c;if(!aff(1,c))return;for(let i=0;i<14;i++){const a=Math.random()*6.28,r=110+Math.random()*170,x=hq.x+Math.cos(a)*r,y=hq.y+Math.sin(a)*r;if(okp(x,y,TY[t].r)){pay(1,c);mk(t,1,x,y);return}}};
if(cn('fac')<1)tryB('fac');else if(cn('tur')<2)tryB('tur');else if(cn('bar')<1+(tm>300?1:0))tryB('bar');else if(cn('fac')<1+(tm>480?1:0))tryB('fac');else if(cn('tur')<Math.min(7,2+tm/100|0))tryB('tur');
for(const b of B){if(b.q.length)continue;const t=b.t=='bar'?'inf':b.t=='fac'?(Math.random()<.6?'tnk':'art'):null;if(t&&aff(1,TY[t].c)){pay(1,TY[t].c);b.q.push(t)}}
const E=R[1],rc=20+E.lv*15;if(E.r>=rc){E.r-=rc;E.lv++;E.dm+=.2}
const un=ents.filter(e=>e.team==1&&!e.b),thr=ents.find(e=>e.team==0&&!e.b&&Math.hypot(e.x-hq.x,e.y-hq.y)<450);
if(thr){un.forEach(u=>{if(!u.tg){u.mv={x:thr.x,y:thr.y};u.am=true}});return}
const idle=un.filter(u=>!u.mv);
if(idle.length>=Math.min(30,6+wave*2)){wave++;const fs=S.filter(s=>s.o!=1).sort((a,b)=>Math.hypot(a.x-hq.x,a.y-hq.y)-Math.hypot(b.x-hq.x,b.y-hq.y))[0];
let tg=fs&&Math.random()<.6?fs:null;if(!tg){const pb=ents.filter(e=>e.team==0&&e.b).sort((a,b)=>a.hp/a.mx-b.hp/b.mx||Math.hypot(a.x-hq.x,a.y-hq.y)-Math.hypot(b.x-hq.x,b.y-hq.y))[0];tg=pb||fs}
if(tg)idle.forEach(u=>{u.mv={x:tg.x+(Math.random()-.5)*80,y:tg.y+(Math.random()-.5)*80};u.am=true})}}
function research(){togUp()}
function q(t){const b=sel[0];if(!b||b.q.length>=5||!aff(0,TY[t].c))return;pay(0,TY[t].c);b.q.push(t);ui()}
function cost(c){return c.map((v,i)=>v?v+['$','S','T'][i]:'').filter(Boolean).join(' ')}
function setB(t){bm=t;ui()}
function allU(){sel=ents.filter(e=>e.team==0&&!e.b);ui()}
function stop(){sel.forEach(u=>{u.mv=null;u.tg=null});}
function ui(){if(!run)return;const r=R[0];$('res').innerHTML=`<span style="color:#ffd24a">Geld ${r.m|0}</span><span style="color:#b8c4d0">Stahl ${r.s|0}</span><span style="color:#ff9a3c">Treibstoff ${r.f|0}</span><span style="color:#c78bff">FP ${r.r|0}</span>`;
$('rs').textContent='Upgrades';
let h='';const sb=sel.length==1&&sel[0].b&&sel[0].team==0?sel[0]:null,un=sel.filter(e=>!e.b&&e.team==0);
if(over)h=`<b>${over}</b><button class="b" onclick="start()">Neu starten</button>`;
else if(bm)h=`<span>${TY[bm].n} platzieren: auf die Karte tippen</span><button class="b" onclick="setB(null)">Abbrechen</button>`;
else{if(sb){h+=`<b>${TY[sb.t].n}</b>`;for(const t in TY)if(TY[t].from==sb.t)h+=`<button class="b" ${aff(0,TY[t].c)&&sb.q.length<5?'':'disabled'} onclick="q('${t}')">${TY[t].n} ${cost(TY[t].c)}</button>`;if(sb.q.length)h+=`<span>Warteschlange: ${sb.q.length}</span>`}
else if(un.length)h+=`<b>${un.length} Einheiten</b><button class="b" onclick="stop()">Stopp</button><button class="b" onclick="recall()">Zurück zur Basis</button><button class="b" onclick="amode=!amode;ui()">Angriffsmarsch: ${amode?'an':'aus'}</button><button class="b" onclick="sel=[];ui()">Abwählen</button>`;
for(const t of['bar','fac','tur','mine','ref'])h+=`<button class="b" ${aff(0,TY[t].c)?'':'disabled'} onclick="setB('${t}')">${TY[t].n} ${cost(TY[t].c)}</button>`;
h+=`<button class="b" onclick="allU()">Alle Einheiten</button><button class="b" onclick="save()">Speichern</button>`}
if(h!=lastH){lastH=h;$('bot').innerHTML=h}renderUp()}
function note(t){msg=t;msgT=2}
function place(w){const t=bm,d=TY[t];if(!aff(0,d.c))return note('Nicht genug Ressourcen');
if(!ents.some(e=>e.team==0&&e.b&&Math.hypot(e.x-w[0],e.y-w[1])<450))return note('Zu weit von deiner Basis entfernt');
if(!okp(w[0],w[1],d.r))return note('Hier ist kein Platz');pay(0,d.c);mk(t,0,w[0],w[1]);bm=null;ui()}
function pick(w){let b=null,bd=1e9;for(const e of ents){const d=Math.hypot(e.x-w[0],e.y-w[1]);if(d<e.r*1.2+cam.d*.012&&d<bd){bd=d;b=e}}return b}
function tap(sx,sy,p){if(MM&&sx>MM.x&&sx<MM.x+MM.w&&sy>MM.y&&sy<MM.y+MM.h){const wx=(sx-MM.x)/MM.k,wy=(sy-MM.y)/MM.k,us=sel.filter(e=>!e.b&&e.team==0);if(us.length){us.forEach(u=>{u.tg=null;u.am=true;u.mv={x:wx+(Math.random()-.5)*60,y:wy+(Math.random()-.5)*60}});fx.push({x:wx,y:wy,l:.5})}else{cam.x=wx;cam.y=wy}return}const w=s2w(sx,sy),right=p.t=='mouse'&&p.b==2;
if(bm){if(right){bm=null;ui()}else place(w);return}
const hit=pick(w),un=sel.filter(e=>!e.b&&e.team==0);const nd=NODES.find(n=>n.a>0&&Math.hypot(n.x-w[0],n.y-w[1])<50),wk=un.filter(u=>u.t=='hrv');if(nd&&wk.length&&!(hit&&hit.team>0)){wk.forEach(u=>{u.nd=nd;u.mv=null;u.man=1;if(u.cap>0&&u.ct!=nd.t)u.dl=1});note('Abbau: '+({m:'Geld',s:'Stahl',f:'Treibstoff'})[nd.t]);fx.push({x:nd.x,y:nd.y,l:.4});return}
if(hit&&hit.team>0&&un.length){un.forEach(u=>{u.tg=hit;u.mv=null});return}
if(!right&&hit&&hit.team==0){sel=[hit];ui();return}
if(un.length&&(right||p.t!='mouse')){un.forEach(u=>{u.tg=null;u.am=!!amode;u.mv={x:w[0]+(Math.random()-.5)*40,y:w[1]+(Math.random()-.5)*40}});fx.push({x:w[0],y:w[1],l:.4});return}
if(right&&sel.length==1&&sel[0].b&&sel[0].team==0){sel[0].rally={x:w[0],y:w[1]};fx.push({x:w[0],y:w[1],l:.4});return}
if(!right){sel=[];ui()}}
const P=new Map();let lp=null,box=null;
const pd=()=>{const a=[...P.values()];return{d:Math.hypot(a[0].x-a[1].x,a[0].y-a[1].y),x:(a[0].x+a[1].x)/2,y:(a[0].y+a[1].y)/2}};
C.addEventListener('contextmenu',e=>e.preventDefault());
C.addEventListener('pointerdown',e=>{C.setPointerCapture(e.pointerId);P.set(e.pointerId,{x:e.clientX,y:e.clientY,x0:e.clientX,y0:e.clientY,b:e.button,t:e.pointerType,mv:0});if(P.size==2)lp=pd()});
C.addEventListener('pointermove',e=>{mx=e.clientX;my=e.clientY;const p=P.get(e.pointerId);if(!p)return;const dx=e.clientX-p.x,dy=e.clientY-p.y;p.x=e.clientX;p.y=e.clientY;
if(Math.hypot(p.x-p.x0,p.y-p.y0)>6)p.mv=1;
if(P.size==2){P.forEach(q=>q.mv=1);const n=pd();zoomAt(n.x,n.y,n.d/lp.d);pan(n.x-lp.x,n.y-lp.y);lp=n;box=null}
else if(p.mv){if(p.t=='mouse'&&p.b==0&&!bm)box={x0:p.x0,y0:p.y0,x1:p.x,y1:p.y};else pan(dx,dy)}});
const up=(e,ok)=>{const p=P.get(e.pointerId);P.delete(e.pointerId);if(!p||!ok)return;
if(box){const b0=box;box=null;const s=ents.filter(o=>{if(o.team||o.b)return 0;const q=w2s(o.x,o.y,10);return q[0]>Math.min(b0.x0,b0.x1)&&q[0]<Math.max(b0.x0,b0.x1)&&q[1]>Math.min(b0.y0,b0.y1)&&q[1]<Math.max(b0.y0,b0.y1)});if(s.length){sel=s;ui()}return}
if(!p.mv&&P.size==0)tap(p.x,p.y,p)};
C.addEventListener('pointerup',e=>up(e,1));C.addEventListener('pointercancel',e=>{up(e,0);box=null});
C.addEventListener('wheel',e=>{e.preventDefault();zoomAt(e.clientX,e.clientY,e.deltaY<0?1.12:.89)},{passive:false});
const K={};addEventListener('keydown',e=>{K[e.key.toLowerCase()]=1;if(e.key=='Escape'){bm=null;sel=[];ui()}if(e.key=='a'&&run&&!e.ctrlKey)allU()});addEventListener('keyup',e=>K[e.key.toLowerCase()]=0);
function head(k,t){const hc=t?'#ff4f7a':'#4f8bff',ic=t?'#ffb02e':'#26d0ff';
X.fillStyle=hc;X.beginPath();X.arc(0,-k,7.6*k,Math.PI,0);X.lineTo(7.6*k,4*k);X.lineTo(-7.6*k,4*k);X.fill();
X.fillStyle='#ffe3c8';X.beginPath();X.arc(0,0,7*k,0,6.28);X.fill();
X.fillStyle=hc;X.beginPath();X.moveTo(-7.5*k,-k);X.lineTo(-4*k,-9*k);X.lineTo(-k,-3*k);X.lineTo(2*k,-10.5*k);X.lineTo(4*k,-3*k);X.lineTo(8*k,-6*k);X.lineTo(7.5*k,-k);X.fill();
for(const sx of[-3,3]){X.fillStyle='#fff';X.beginPath();X.ellipse(sx*k,k,2.4*k,3.1*k,0,0,6.28);X.fill();X.fillStyle=ic;X.beginPath();X.ellipse(sx*k,1.4*k,1.7*k,2.5*k,0,0,6.28);X.fill();X.fillStyle='#14203a';X.beginPath();X.ellipse(sx*k,1.7*k,.8*k,1.4*k,0,0,6.28);X.fill();X.fillStyle='#fff';X.beginPath();X.arc((sx-.6)*k,.2*k,.75*k,0,6.28);X.fill();X.fillStyle='#ff8fa866';X.beginPath();X.arc(sx*1.8*k,4.4*k,1.3*k,0,6.28);X.fill()}}
function bld(e,c,dk,lt,tt){const r=e.r,h=e.t=='hq'?r*1.1:r*.85,yf=e.y+r*.6,dd=r*.7,x=e.x;
if(e.t=='tur'){X.fillStyle=dk;X.fillRect(x-r,yf-h,2*r,h);X.beginPath();X.ellipse(x,yf,r,r*.45,0,0,6.28);X.fill();X.fillStyle=lt;X.beginPath();X.ellipse(x,yf-h,r,r*.5,0,0,6.28);X.fill();
X.save();X.translate(x,yf-h-3);X.rotate(e.a);X.scale(1,.6);X.fillStyle='#2a2a33';X.fillRect(0,-3.5,r*1.7,7);X.fillStyle=c;X.beginPath();X.arc(0,0,r*.55,0,6.28);X.fill();X.restore();if(e.cd>.55){X.fillStyle='#ffe27a';X.beginPath();X.arc(x+Math.cos(e.a)*r*1.7,yf-h-3+Math.sin(e.a)*r,5,0,6.28);X.fill()}return}
X.fillStyle=dk;X.fillRect(x-r,yf-h,2*r,h);X.fillStyle='rgba(0,0,0,.28)';X.beginPath();X.moveTo(x+r,yf-h);X.lineTo(x+r+dd*.5,yf-h-dd);X.lineTo(x+r+dd*.5,yf-dd);X.lineTo(x+r,yf);X.fill();
X.fillStyle='#ffe9a0';for(const i of[-1,1])X.fillRect(x+i*r*.6-r*.13,yf-h*.8,r*.26,h*.3);X.fillStyle='#10161f';X.fillRect(x-r*.22,yf-h*.5,r*.44,h*.5);
const g=X.createLinearGradient(0,yf-h-dd,0,yf-h);g.addColorStop(0,lt);g.addColorStop(1,c);X.fillStyle=g;X.beginPath();X.moveTo(x-r,yf-h);X.lineTo(x-r+dd*.5,yf-h-dd);X.lineTo(x+r+dd*.5,yf-h-dd);X.lineTo(x+r,yf-h);X.fill();
const ry=yf-h-dd*.5,rx=x+dd*.25;
if(e.t=='hq'){X.fillStyle=lt;X.fillRect(rx-r*.3,ry-r*.7,r*.6,r*.7);X.fillStyle=dk;X.fillRect(rx-r*.3,ry-r*.7,r*.6,r*.2);X.strokeStyle='#eee';X.lineWidth=2;X.beginPath();X.moveTo(rx,ry-r*.7);X.lineTo(rx,ry-r*1.6);X.stroke();X.fillStyle='#ffd24a';const w=Math.sin(tt*4)*3;X.beginPath();X.moveTo(rx,ry-r*1.6);X.lineTo(rx+r*.6,ry-r*1.4+w);X.lineTo(rx,ry-r*1.2);X.fill()}
else if(e.t=='fac'){X.fillStyle='#3a3f4a';X.fillRect(x+r*.4,ry-r*.8,r*.32,r*.8);for(let i=0;i<3;i++){const a=(tt*.6+i/3)%1;X.fillStyle='rgba(235,235,235,'+(1-a)*.6+')';X.beginPath();X.arc(x+r*.56+a*10,ry-r*.8-a*34,4+a*9,0,6.28);X.fill()}}
else{X.fillStyle='#ffffff55';for(const i of[-.5,.1])X.fillRect(rx+i*r,ry-dd*.45,r*.3,dd*.9)}}
function drawE(e){const d=TY[e.t],c=COL[e.team],dk=e.team?'#9a2e1c':'#16609e',lt=e.team?'#ffb08a':'#9fdcff',s=sel.includes(e),r=e.r,tt=performance.now()/1000;
X.fillStyle='#0006';X.beginPath();X.ellipse(e.x+r*.3,e.y+r*.55,r*1.15,r*.55,0,0,6.28);X.fill();
if(s){X.strokeStyle='#fff';X.lineWidth=3;X.beginPath();X.ellipse(e.x,e.y+r*.5,r*1.35,r*.7,0,0,6.28);X.stroke();if(e.mv&&e.team==0){X.setLineDash([6,6]);X.strokeStyle='#ffffff99';X.lineWidth=1.5;X.beginPath();X.moveTo(e.x,e.y);X.lineTo(e.mv.x,e.mv.y);X.stroke();X.setLineDash([])}}
let top=r*2.4;
if(e.b){bld(e,c,dk,lt,tt);top=r*2.6}
else if(e.t=='inf'){const k=r/8,f=Math.cos(e.a)<0?-1:1,mv=e.mv||e.tg?Math.sin(tt*12+e.x)*1.8*k:0;X.save();X.translate(e.x,e.y+r*.5);X.scale(f,1);
X.fillStyle='#2a2f3a';X.fillRect(-3.2*k,-5*k,2.6*k,5*k+mv);X.fillRect(.6*k,-5*k,2.6*k,5*k-mv);
X.fillStyle=dk;X.fillRect(-4*k,-13*k,8*k,9*k);X.fillStyle=c;X.fillRect(-4*k,-13*k,8*k,3*k);X.fillStyle='#ffd24a';X.fillRect(-4*k,-6*k,8*k,1.4*k);
X.fillStyle='#2a2a33';X.fillRect(2*k,-10.5*k,10*k,2.6*k);if(e.cd>TY.inf.cd-.1){X.fillStyle='#ffe27a';X.beginPath();X.arc(13*k,-9*k,2.5*k,0,6.28);X.fill()}
X.translate(0,-19*k+Math.abs(mv)*.4);head(k,e.team);X.restore();top=r*3.4}
else{const k=r/14,L=e.t=='art'?2.3:1.5,rise=e.t=='art'?r*.5:0,x=e.x,y=e.y+r*.5;
X.fillStyle='#262a33';X.beginPath();X.roundRect(x-r*1.2,y-6*k,r*2.4,7*k,3*k);X.fill();X.fillStyle='#555a66';for(let i=-3;i<=3;i++){X.beginPath();X.arc(x+i*r*.36,y-2.5*k,2.3*k,0,6.28);X.fill()}
const g=X.createLinearGradient(0,y-14*k,0,y-5*k);g.addColorStop(0,lt);g.addColorStop(1,c);X.fillStyle=g;X.beginPath();X.moveTo(x-r*1.05,y-5*k);X.lineTo(x-r*.8,y-13*k);X.lineTo(x+r*.9,y-13*k);X.lineTo(x+r*1.15,y-5*k);X.fill();
X.fillStyle=dk;X.beginPath();X.ellipse(x,y-14*k,r*.6,r*.42,0,0,6.28);X.fill();X.fillStyle=c;X.beginPath();X.ellipse(x,y-15.5*k,r*.55,r*.38,0,0,6.28);X.fill();
const bx=x+Math.cos(e.a)*r*L,by=y-15*k+Math.sin(e.a)*r*L*.55-rise;X.strokeStyle='#2a2a33';X.lineWidth=5*k;X.lineCap='round';X.beginPath();X.moveTo(x,y-15*k);X.lineTo(bx,by);X.stroke();X.lineCap='butt';
if(e.cd>TY[e.t].cd-.12){X.fillStyle='#ffe27a';X.beginPath();X.arc(bx,by,6*k,0,6.28);X.fill()}
X.save();X.translate(x,y-24*k);head(k*.75,e.team);X.restore();top=r*3.4}
if(e.hp<e.mx||s){const w=Math.max(26,r*2);X.fillStyle='#000a';X.fillRect(e.x-w/2-1,e.y-top-1,w+2,7);X.fillStyle=e.hp/e.mx>.4?'#7be04a':'#ffb02e';X.fillRect(e.x-w/2,e.y-top,w*e.hp/e.mx,5)}}
let BG=null;
function mkBg(){BG=document.createElement('canvas');BG.width=W/2;BG.height=H/2;const g=BG.getContext('2d');g.scale(.5,.5);let sd=7;const rn=()=>(sd=(sd*16807)%2147483647)/2147483647;
const gr=g.createLinearGradient(0,0,W,H);gr.addColorStop(0,'#3f8a3a');gr.addColorStop(.5,'#58a044');gr.addColorStop(1,'#7fa23c');g.fillStyle=gr;g.fillRect(0,0,W,H);
for(let i=0;i<80;i++){g.fillStyle=rn()<.5?'#2f7a36':'#79bb55';g.globalAlpha=.3;g.beginPath();g.ellipse(rn()*W,rn()*H,60+rn()*200,40+rn()*120,rn()*3,0,6.28);g.fill()}g.globalAlpha=1;
const sg=SG;
g.lineCap='round';for(const[c,w]of[['#a8925a',54],['#e3cf94',40]]){g.strokeStyle=c;g.lineWidth=w;g.beginPath();for(const s of sg){g.moveTo(s[0],s[1]);g.lineTo(s[2],s[3])}g.stroke()}
const ds=(px,py,s)=>{const dx=s[2]-s[0],dy=s[3]-s[1],t=Math.max(0,Math.min(1,((px-s[0])*dx+(py-s[1])*dy)/(dx*dx+dy*dy)));return Math.hypot(px-s[0]-t*dx,py-s[1]-t*dy)};
const bad=(x,y,m)=>HQP.some(p=>Math.hypot(p[0]-x,p[1]-y)<260)||S.some(s=>Math.hypot(s.x-x,s.y-y)<170)||sg.some(s=>ds(x,y,s)<m);
for(const[x,y,a,b]of[[1100,500,170,110],[1800,1700,200,100],[400,1000,130,170],[2650,950,120,160],[1500,330,190,90]]){g.fillStyle='#e8d9a0';g.beginPath();g.ellipse(x,y,a+14,b+14,.3,0,6.28);g.fill();g.fillStyle='#2f86c9';g.beginPath();g.ellipse(x,y,a,b,.3,0,6.28);g.fill();g.fillStyle='#55b4ee';g.beginPath();g.ellipse(x-a*.15,y-b*.2,a*.6,b*.5,.3,0,6.28);g.fill()}
const fc=['#ffd24a','#ff7ab0','#ffffff','#c78bff'];for(let i=0;i<1200;i++){const x=rn()*W,y=rn()*H;if(!bad(x,y,40)){g.fillStyle=fc[i%4];g.beginPath();g.arc(x,y,3.5,0,6.28);g.fill()}}
const tc=['#1f6a2a','#2e8b3a','#2e8b3a','#e0902a','#c9472b'];for(let i=0;i<1500;i++){const x=rn()*W,y=rn()*H,r=14+rn()*12;if(bad(x,y,60))continue;g.fillStyle='#0003';g.beginPath();g.ellipse(x+6,y+r*.6,r,r*.6,0,0,6.28);g.fill();g.fillStyle=tc[i%5];g.beginPath();g.arc(x,y,r,0,6.28);g.fill();g.fillStyle='#ffffff30';g.beginPath();g.arc(x-r*.3,y-r*.3,r*.45,0,6.28);g.fill()}
for(let i=0;i<60;i++){const x=rn()*W,y=rn()*H;if(bad(x,y,60))continue;g.fillStyle='#8b8f94';g.beginPath();g.arc(x,y,10+rn()*8,0,6.28);g.fill();g.fillStyle='#b9bec4';g.beginPath();g.arc(x-3,y-3,5,0,6.28);g.fill()}}
var NODES=[],AIS=[],NN=null,NG=null,MM=null,amode=0,upOpen=0,lastU='',sv=15;
const HQP=[[400,2800],[4400,400],[4400,2800],[400,400]],TCL=['#2f8ce0','#e0482e','#2faa55','#d6a21f'],TDK=['#17508a','#8f2a1a','#17653a','#8a6410'],HC=['#4f8bff','#ff4f7a','#58d67a','#ffd04a'];
const UPG=[{id:'inf_a',n:'Infanterie: Schaden',b:[60,20,0,10]},{id:'inf_h',n:'Infanterie: Panzerung',b:[60,30,0,10]},{id:'tnk_a',n:'Panzer: Kanone',b:[120,80,20,20]},{id:'tnk_h',n:'Panzer: Panzerung',b:[120,100,20,20]},{id:'art_a',n:'Artillerie: Sprengkraft',b:[140,80,30,20]},{id:'art_h',n:'Artillerie: Panzerung',b:[140,100,30,20]},{id:'tur_a',n:'Türme: Feuerkraft',b:[100,80,0,15]},{id:'bh',n:'Gebäude: Verstärkung',b:[150,120,0,15]},{id:'eco',n:'Wirtschaft: Förderrate',b:[120,60,0,10]},{id:'spd',n:'Ausbildung: Tempo',b:[100,60,20,15]},{id:'mv',n:'Fahrzeuge: Antrieb',b:[90,40,30,15]}];
const ulv=(k,id)=>R[k].u[id]||0,ucost=(k,id)=>{const f=Math.pow(1.7,ulv(k,id));return UPG.find(u=>u.id==id).b.map(v=>Math.ceil(v*f))};
function buyU(k,id){const l=ulv(k,id);if(l>=10)return 0;const c=ucost(k,id),r=R[k];if(r.m<c[0]||r.s<c[1]||r.f<c[2]||r.r<c[3])return 0;r.m-=c[0];r.s-=c[1];r.f-=c[2];r.r-=c[3];r.u[id]=l+1;const f=(1+.15*(l+1))/(1+.15*l);ents.forEach(e=>{if(e.team==k&&(id=='bh'?e.b:id==e.t+'_h')){e.hp*=f;e.mx*=f}});return 1}
function togUp(){upOpen=!upOpen;$('upn').style.display=upOpen?'flex':'none';lastU='';renderUp()}
function renderUp(){if(!upOpen)return;const r=R[0];let h='<div class=ut><b>Upgrades</b><button class=b data-c=1>Schließen</button></div>';for(const u of UPG){const l=ulv(0,u.id),c=ucost(0,u.id),ok=l<10&&r.m>=c[0]&&r.s>=c[1]&&r.f>=c[2]&&r.r>=c[3];h+=`<div class=ur><span>${u.n} <i>Stufe ${l}/10</i></span><button class=b data-u=${u.id} ${ok?'':'disabled'}>${l>=10?'Max':c[0]+'$ '+c[1]+'S '+c[2]+'T '+c[3]+'FP'}</button></div>`}if(h!=lastU){lastU=h;$('upn').innerHTML=h}}
$('upn').onclick=e=>{const d=e.target.dataset;if(!d)return;if(d.c)togUp();else if(d.u){buyU(0,d.u);lastU='';ui()}};
function recall(){const h=ents.find(e=>e.t=='hq'&&e.team==0);if(!h)return;sel.filter(e=>!e.b&&e.team==0).forEach(u=>{u.tg=null;u.am=false;u.mv={x:h.x+(Math.random()-.5)*160,y:h.y+(Math.random()-.5)*160}})}
function genNodes(){const n=[];HQP.forEach((p,k)=>{const sx=k==0||k==3?1:-1,sy=k==0||k==2?-1:1;[[270,-40,'m'],[340,130,'s'],[180,270,'f'],[430,-190,'m'],[120,-300,'s']].forEach(o=>n.push({x:p[0]+sx*o[0],y:p[1]+sy*o[1],t:o[2],a:3000,mx:3000}))});let sd=11;const rn=()=>(sd=sd*16807%2147483647)/2147483647,ty=['m','s','f'];for(let i=0;i<24;i++){let x,y,t=0;do{x=600+rn()*3600;y=500+rn()*2200;t++}while(HQP.some(p=>Math.hypot(p[0]-x,p[1]-y)<700)&&t<20);n.push({x,y,t:ty[i%3],a:2500,mx:2500})}return n}
function harv(e,dt){const hq=ents.find(o=>o.t=='hq'&&o.team==e.team);e.cap=e.cap||0;if(e.mv){if(step(e,e.mv.x,e.mv.y,dt)<10)e.mv=null;return}
if(e.cap>=60||e.dl||(e.cap>0&&(!e.nd||e.nd.a<=0))){let dp=null,db=1e13;for(const o of ents)if(o.team==e.team&&(o.t=='hq'||o.t=='ref')){const d=(o.x-e.x)**2+(o.y-e.y)**2;if(d<db){db=d;dp=o}}if(!dp)return;if(step(e,dp.x,dp.y,dt)<dp.r+24){const rf=ents.filter(o=>o.team==e.team&&o.t=='ref').length;if(e.cap>0)R[e.team][e.ct]+=e.cap*(e.team?.85:1)*(1+.15*(R[e.team].u.eco||0))*(1+.2*Math.min(3,rf));e.cap=0;e.dl=0;e.nd=e.man&&e.nd&&e.nd.a>0?e.nd:null}return}
if(!e.nd||e.nd.a<=0){let b=null,bd=1e13;for(const n of NODES){if(n.a<=0)continue;const d=(n.x-e.x)**2+(n.y-e.y)**2+(hq?((n.x-hq.x)**2+(n.y-hq.y)**2)*.7:0);if(d<bd){bd=d;b=n}}e.nd=b;if(!b)return}
if(step(e,e.nd.x,e.nd.y,dt)<34){const g=Math.min(12*dt,e.nd.a,60-e.cap);e.nd.a-=g;e.cap+=g;e.ct=e.nd.t}}
function save(q){try{const d={tm,R,S:S.map(s=>({o:s.o,p:s.p,lt:s.lt})),N:NODES.map(n=>n.a),AIS,ents:ents.map(e=>({t:e.t,team:e.team,x:e.x,y:e.y,hp:e.hp,mx:e.mx,a:e.a,q:e.q,pt:e.pt,mv:e.mv,am:e.am,rally:e.rally,cap:e.cap,ct:e.ct}))};localStorage.setItem('sf2',JSON.stringify(d));if(!q)note('Gespeichert')}catch(x){if(!q)note('Speichern nicht möglich')}}
function load(){try{const d=JSON.parse(localStorage.getItem('sf2'));start();ents=[];sel=[];tm=d.tm;d.R.forEach((r,i)=>Object.assign(R[i],r));S.forEach((s,i)=>Object.assign(s,d.S[i]));NODES.forEach((n,i)=>n.a=d.N[i]);AIS=d.AIS;for(const o of d.ents){const e=mk(o.t,o.team,o.x,o.y);Object.assign(e,o)}ui()}catch(x){note('Kein Spielstand gefunden')}}
try{if(localStorage.getItem('sf2'))$('ld').style.display=''}catch(x){}
var REN,CAM,SC,LT,GH,SM=[],MESH=new Map();
const SG=[[400,2800,900,2450],[900,2450,2400,1600],[2400,1600,3900,750],[3900,750,4400,400],[2400,1600,3900,2450],[3900,2450,4400,2800],[2400,1600,900,750],[900,750,400,400]];
const dsg=(px,py,s)=>{const dx=s[2]-s[0],dy=s[3]-s[1],t=Math.max(0,Math.min(1,((px-s[0])*dx+(py-s[1])*dy)/(dx*dx+dy*dy)));return Math.hypot(px-s[0]-t*dx,py-s[1]-t*dy)};
const MT={},mat=c=>MT[c]||(MT[c]=new THREE.MeshLambertMaterial({color:c}));
const ad=(g,m,x,y,z)=>{m.position.set(x,y,z);m.castShadow=m.receiveShadow=true;g.add(m);return m};
const bx=(g,w,h,d,c,x,y,z)=>ad(g,new THREE.Mesh(new THREE.BoxGeometry(w,h,d),mat(c)),x,y+h/2,z);
const cy=(g,a,b,h,c,x,y,z,n=14)=>ad(g,new THREE.Mesh(new THREE.CylinderGeometry(a,b,h,n),mat(c)),x,y+h/2,z);
const cn=(g,r,h,c,x,y,z,n=10)=>ad(g,new THREE.Mesh(new THREE.ConeGeometry(r,h,n),mat(c)),x,y+h/2,z);
const sp=(g,r,c,x,y,z,sx=1,sy=1,sz=1)=>{const m=ad(g,new THREE.Mesh(new THREE.SphereGeometry(r,14,10),mat(c)),x,y,z);m.scale.set(sx,sy,sz);return m};
function mkHead(t,hr){const g=new THREE.Group(),hc=HC[t],ic=t?'#ffb02e':'#26d0ff';sp(g,hr,'#ffe3c8',0,0,0);
const h=new THREE.Mesh(new THREE.SphereGeometry(hr*1.1,16,10,0,6.283,0,1.25),mat(hc));h.position.y=hr*.08;h.castShadow=true;g.add(h);sp(g,hr,hc,-hr*.3,0,0,.9,1,1);
for(const a of[-2.2,-1.1,0,1.1,2.2]){const m=cn(g,hr*.32,hr*.85,hc,Math.cos(a)*hr*.75,hr*.7,Math.sin(a)*hr*.75,6);m.rotation.set(Math.sin(a)*.6,0,-Math.cos(a)*.6)}
for(const z of[-.4,.4]){sp(g,hr*.3,'#ffffff',hr*.86,-hr*.02,z*hr,.5,1.15,.9);sp(g,hr*.22,ic,hr*.95,-hr*.05,z*hr,.4,1.1,.8);sp(g,hr*.11,'#14203a',hr*1.0,-hr*.06,z*hr,.4,1,.8);sp(g,hr*.07,'#ffffff',hr*1.04,hr*.08,z*hr-hr*.05,.4,1,1)}return g}
function build(e){const g=new THREE.Group(),b=new THREE.Group(),r=e.r,t=e.team,tc=TCL[t],td=TDK[t],wl='#efe7d2',ud={b,upd:null},T=e.t;g.add(b);g.userData=ud;
const rg=new THREE.Mesh(new THREE.RingGeometry(r*1.5,r*1.75,32),new THREE.MeshBasicMaterial({color:'#ffffff',side:THREE.DoubleSide}));rg.rotation.x=-Math.PI/2;rg.position.y=1.5;rg.visible=false;g.add(rg);ud.ring=rg;
if(T=='hq'){bx(g,2*r,r*1.1,2*r,wl,0,0,0);bx(g,2.1*r,r*.15,2.1*r,tc,0,r*1.1,0);bx(g,r*.9,r*.9,r*.9,wl,0,r*1.25,0);cn(g,r*.7,r*.5,td,0,r*2.15,0,4).rotation.y=.785;cy(g,1.2,1.2,r*1.1,'#dddddd',0,r*2.6,0,6);const fl=bx(g,r*.7,r*.4,2,'#ffd24a',r*.35,r*3.3,0);for(const s of[-1,1])bx(g,r*.3,r*.35,2,'#ffe27a',s*r*.55,r*.5,r);ud.upd=(e,tt)=>{fl.rotation.y=Math.sin(tt*4)*.3}}
else if(T=='bar'){bx(g,1.8*r,r*.8,1.6*r,wl,0,0,0);cn(g,r*1.45,r*.8,tc,0,r*.8,0,4).rotation.y=.785;bx(g,r*.45,r*.55,2,'#10161f',0,0,r*.8);bx(g,r*.3,r*.3,2,'#ffe27a',r*.55,r*.35,r*.8)}
else if(T=='fac'){bx(g,2.2*r,r*.9,1.8*r,wl,0,0,0);bx(g,2.3*r,r*.2,1.9*r,tc,0,r*.9,0);bx(g,r*.9,r*.6,2,'#10161f',0,0,.9*r);cy(g,r*.2,r*.25,r*1.2,'#555555',r*.6,r*.9,-r*.3,8);const pf=[0,1,2].map(()=>{const m=new THREE.Mesh(new THREE.SphereGeometry(r*.3,8,6),new THREE.MeshLambertMaterial({color:'#eeeeee',transparent:true}));g.add(m);return m});ud.upd=(e,tt)=>{pf.forEach((m,i)=>{const a=(tt*.5+i/3)%1;m.position.set(r*.6+a*r,r*2.1+a*r*1.8,-r*.3);m.scale.setScalar(.6+a*1.4);m.material.opacity=(1-a)*.7})}}
else if(T=='tur'){cy(g,r,r*1.15,r*.6,td,0,0,0);cy(b,r*.7,r*.8,r*.5,tc,0,r*.6,0);const br=new THREE.Mesh(new THREE.CylinderGeometry(r*.14,r*.16,r*1.7,8),mat('#2a2a33'));br.rotation.z=-Math.PI/2;ad(b,br,r*.95,r,0);const fl=sp(b,r*.3,'#ffe27a',r*1.9,r,0);fl.visible=false;ud.upd=e=>{fl.visible=e.cd>.55}}
else if(T=='mine'){bx(g,r*1.6,r*.8,r*1.4,'#6b5a45',0,0,0);cn(g,r*1.3,r*.7,tc,0,r*.8,0,4).rotation.y=.785;bx(g,r*.5,r*.6,2,'#10161f',0,0,r*.7);for(let i=0;i<3;i++)sp(g,r*.2,'#ffd24a',-r*.9+i*r*.25,r*.2,r*.9);const cr=cn(g,r*.3,r*.8,'#ffd24a',0,r*1.5,0,5);ud.upd=(e,tt)=>{cr.rotation.y=tt*2}}
else if(T=='ref'){bx(g,2*r,r*.35,1.6*r,'#555b66',0,0,0);for(const x of[-.55,.55])cy(g,r*.4,r*.4,r*1.1,'#c9d2dc',x*r,r*.35,-r*.2,16);cy(g,r*.15,r*.15,r*1.8,'#444444',0,r*.35,r*.5,8);const fl=sp(g,r*.2,'#ff9a2a',0,r*2.2,r*.5);bx(g,r*.9,r*.3,r*.5,tc,0,r*.35,r*.6);ud.upd=(e,tt)=>{fl.scale.setScalar(.8+Math.sin(tt*14)*.3)}}
else if(T=='hrv'){b.scale.setScalar(1.3);bx(b,r*1.8,r*.5,r*1.1,'#3a3f4a',0,r*.15,0);bx(b,r*1.1,r*.7,r*1.05,tc,-r*.3,r*.65,0);bx(b,r*.6,r*.6,r,wl,r*.55,r*.65,0);for(const x of[-.6,.6])for(const z of[-.6,.6]){const w=cy(b,r*.28,r*.28,r*.2,'#222222',x*r,r*.18,z*r,10);w.rotation.x=1.5708}const h=mkHead(t,r*.4);h.position.set(r*.55,r*1.55,0);b.add(h)}
else if(T=='inf'){b.scale.setScalar(1.5);const m2=z=>{const l=new THREE.Group();l.position.set(0,r*.8,z);bx(l,r*.4,r*.8,r*.4,'#2a2f3a',0,-r*.8,0);b.add(l);return l},L=m2(-r*.28),Rr=m2(r*.28);cy(b,r*.5,r*.58,r*.95,td,0,r*.8,0);cy(b,r*.52,r*.54,r*.3,tc,0,r*1.45,0);bx(b,r*1.1,r*.25,r*.25,'#2a2a33',r*.7,r*1.2,r*.45);const h=mkHead(t,r*.85);h.position.y=r*2.55;b.add(h);ud.upd=(e,tt)=>{const s=e.mv||e.tg?Math.sin(tt*12+e.x)*.7:0;L.rotation.z=s;Rr.rotation.z=-s;b.position.y=Math.abs(s)*1.5}}
else{b.scale.setScalar(1.3);const A=T=='art',len=A?r*2.4:r*1.5,bs=A?[r*.7,r*1.3]:[r*.5,r*1.1];for(const s of[-1,1])bx(b,r*2.4,r*.55,r*.5,'#2a2d36',0,0,s*r*.8);bx(b,r*2,r*.5,r*1.1,tc,0,r*.4,0);bx(b,r*1.2,r*.15,r*1.12,td,-r*.3,r*.9,0);cy(b,r*.5,r*.6,r*.4,td,-r*.1,r*.9,0);const th=A?-(Math.PI/2-.5):-Math.PI/2,dx=-Math.sin(th),dy=Math.cos(th),br=new THREE.Mesh(new THREE.CylinderGeometry(r*.09,r*.11,len,8),mat('#2a2a33'));br.rotation.z=th;ad(b,br,bs[0]+dx*len/2,bs[1]+dy*len/2,0);const fl=sp(b,r*.28,'#ffe27a',bs[0]+dx*len,bs[1]+dy*len,0);fl.visible=false;const h=mkHead(t,r*.5);h.position.set(-r*.15,r*1.75,0);b.add(h);const cd=TY[T].cd;ud.upd=e=>{fl.visible=e.cd>cd-.1}}
return g}
function init3(){REN=window.REN=new THREE.WebGLRenderer({canvas:$('g'),antialias:true});REN.setPixelRatio(Math.min(devicePixelRatio||1,2));REN.shadowMap.enabled=true;REN.shadowMap.type=THREE.PCFSoftShadowMap;
SC=new THREE.Scene();SC.background=new THREE.Color('#a8dcff');SC.fog=new THREE.Fog('#a8dcff',3500,8000);CAM=new THREE.PerspectiveCamera(45,1,20,12000);
SC.add(new THREE.HemisphereLight('#d8eeff','#56773f',.85));LT=new THREE.DirectionalLight('#fff1d0',.95);LT.castShadow=true;LT.shadow.mapSize.set(2048,2048);const sc=LT.shadow.camera;sc.left=-1100;sc.right=1100;sc.top=1100;sc.bottom=-1100;sc.near=100;sc.far=4000;SC.add(LT,LT.target);
if(!BG)mkBg();const tx=new THREE.CanvasTexture(BG);tx.anisotropy=8;const gd=new THREE.Mesh(new THREE.PlaneGeometry(W,H),new THREE.MeshLambertMaterial({map:tx}));gd.rotation.x=-Math.PI/2;gd.position.set(W/2,0,H/2);gd.receiveShadow=true;SC.add(gd);
{const N=650,d=new THREE.Object3D(),tg=new THREE.CylinderGeometry(3,4,14,6),cg=new THREE.ConeGeometry(16,34,8),a=new THREE.InstancedMesh(tg,mat('#6b4a2b'),N),m1=new THREE.MeshLambertMaterial({color:'#ffffff'}),b1=new THREE.InstancedMesh(cg,m1,N),b2=new THREE.InstancedMesh(cg,m1,N),pal=['#1f8a3a','#2fa04a','#2fa04a','#f0a030','#e0562f','#9c5bd6'];let sd=3,n=0;const rn=()=>(sd=sd*16807%2147483647)/2147483647;
while(n<N){const x=rn()*W,y=rn()*H;if(HQP.some(p=>Math.hypot(p[0]-x,p[1]-y)<300)||S.some(s=>Math.hypot(s.x-x,s.y-y)<190)||SG.some(s=>dsg(x,y,s)<70))continue;const k=.8+rn()*.9,c=new THREE.Color(pal[n%6]);d.scale.set(k,k,k);d.position.set(x,7*k,y);d.updateMatrix();a.setMatrixAt(n,d.matrix);d.position.y=27*k;d.updateMatrix();b1.setMatrixAt(n,d.matrix);b1.setColorAt(n,c);d.scale.set(.65*k,.65*k,.65*k);d.position.y=43*k;d.updateMatrix();b2.setMatrixAt(n,d.matrix);b2.setColorAt(n,c);n++}
for(const m of[a,b1,b2]){m.castShadow=true;SC.add(m)}}
for(let i=0;i<40;i++){const an=i/40*6.283,h=250+(i*97%250),r=180+(i*53%140),m=new THREE.Mesh(new THREE.ConeGeometry(r,h,6),mat(['#7a86c4','#8a7ab8','#6f95b8'][i%3]));m.position.set(W/2+Math.cos(an)*(W/2+300),h/2,H/2+Math.sin(an)*(H/2+300));SC.add(m)}
SM=S.map(s=>{const g=new THREE.Group();g.position.set(s.x,0,s.y);const rg=new THREE.Mesh(new THREE.RingGeometry(138,150,48),new THREE.MeshBasicMaterial({color:'#ffffff',side:THREE.DoubleSide}));rg.rotation.x=-Math.PI/2;rg.position.y=1.2;g.add(rg);const dc=new THREE.Mesh(new THREE.CircleGeometry(138,40),new THREE.MeshBasicMaterial({color:'#ffffff',transparent:true,opacity:.2,side:THREE.DoubleSide}));dc.rotation.x=-Math.PI/2;dc.position.y=1;g.add(dc);cy(g,2.5,2.5,95,'#eeeeee',0,0,0,6);const fl=new THREE.Mesh(new THREE.PlaneGeometry(52,30),new THREE.MeshBasicMaterial({color:'#ffffff',side:THREE.DoubleSide}));fl.position.set(26,78,0);g.add(fl);SC.add(g);return{rg,dc,fl}});
GH=new THREE.Mesh(new THREE.BoxGeometry(1,1,1),new THREE.MeshBasicMaterial({transparent:true,opacity:.55}));SC.add(GH);rs()}
const RC=new THREE.Raycaster(),V2=new THREE.Vector2();
function s2w(sx,sy){CAM.updateMatrixWorld();V2.set(sx/innerWidth*2-1,-(sy/innerHeight)*2+1);RC.setFromCamera(V2,CAM);const o=RC.ray.origin,d=RC.ray.direction;if(d.y>=-1e-4)return[cam.x,cam.y];const t=-o.y/d.y;return[o.x+d.x*t,o.z+d.z*t]}
function w2s(x,y,h=0){const v=new THREE.Vector3(x,h,y).project(CAM);return[(v.x+1)/2*innerWidth,(1-v.y)/2*innerHeight]}
function pan(dx,dy){const k=cam.d*.83/innerHeight;cam.x-=dx*k;cam.y-=dy*k*1.25}
function draw(){const tt=performance.now()/1000,live=new Set(ents);
for(const e of ents){let m=MESH.get(e);if(!m){m=build(e);SC.add(m);MESH.set(e,m)}const u=m.userData;m.position.set(e.x,0,e.y);if(!e.b||e.t=='tur')u.b.rotation.y=-e.a;u.ring.visible=sel.includes(e);if(u.upd)u.upd(e,tt)}
for(const[e,m]of MESH)if(!live.has(e)){SC.remove(m);MESH.delete(e)}
S.forEach((s,i)=>{const c=s.o<0?'#ffd24a':COL[s.o],m=SM[i];m.rg.material.color.set(c);m.dc.material.color.set(c);m.fl.material.color.set(c);m.fl.rotation.y=Math.sin(tt*3+i)*.4});
if(NODES!==NN){NN=NODES;if(NG)SC.remove(NG);NG=new THREE.Group();for(const n of NODES){const g=new THREE.Group();g.position.set(n.x,0,n.y);if(n.t=='m'){for(let i=0;i<4;i++)cn(g,9,26,'#ffd24a',Math.cos(i*1.6)*14,0,Math.sin(i*1.6)*14,5)}else if(n.t=='s'){for(let i=0;i<4;i++)bx(g,20,12+i*5,20,'#9fb4c8',Math.cos(i*1.6)*13,0,Math.sin(i*1.6)*13)}else{for(let i=0;i<3;i++){cy(g,9,9,22,'#2b2b33',Math.cos(i*2.1)*12,0,Math.sin(i*2.1)*12);cy(g,9.5,9.5,4,'#ff8a2a',Math.cos(i*2.1)*12,22,Math.sin(i*2.1)*12)}}n.g=g;NG.add(g)}SC.add(NG)}
for(const n of NODES)n.g.scale.setScalar((n.a>0?.5+.5*n.a/n.mx:.01)*1.8);
GH.visible=!!bm;if(bm){const d=TY[bm],w=s2w(mx,my),ok=okp(w[0],w[1],d.r)&&ents.some(e=>e.team==0&&e.b&&Math.hypot(e.x-w[0],e.y-w[1])<450);GH.scale.set(d.r*2,d.r*1.2,d.r*2);GH.position.set(w[0],d.r*.6,w[1]);GH.material.color.set(ok?'#4cff7a':'#ff4c3c')}
cam.x=Math.max(0,Math.min(W,cam.x));cam.y=Math.max(0,Math.min(H,cam.y));cam.d=Math.max(450,Math.min(5400,cam.d));
CAM.position.set(cam.x,cam.d*.82,cam.y+cam.d*.57);CAM.lookAt(cam.x,0,cam.y);LT.position.set(cam.x-500,1300,cam.y+500);LT.target.position.set(cam.x,0,cam.y);CAM.updateMatrixWorld();REN.render(SC,CAM);
X.setTransform(dpr,0,0,dpr,0,0);X.clearRect(0,0,innerWidth,innerHeight);const ppu=innerHeight/(.83*cam.d);
X.textAlign='center';X.font='700 16px "Saira Condensed",sans-serif';X.lineWidth=3;X.strokeStyle='#000a';for(const s of S){const q=w2s(s.x,s.y,110),c=s.o<0?'#ffd24a':COL[s.o];X.fillStyle=c;X.strokeText(s.n,q[0],q[1]);X.fillText(s.n,q[0],q[1])}
for(const p of proj){const a=w2s(p.x1,p.y1,20),b=w2s(p.x2,p.y2,14);X.strokeStyle=p.c;X.globalAlpha=.4;X.lineWidth=p.w*4;X.beginPath();X.moveTo(a[0],a[1]);X.lineTo(b[0],b[1]);X.stroke();X.globalAlpha=1;X.strokeStyle='#fff';X.lineWidth=p.w;X.beginPath();X.moveTo(a[0],a[1]);X.lineTo(b[0],b[1]);X.stroke()}
for(const f of fx){const q=w2s(f.x,f.y,10),rr=(40*(1-f.l*2)+6)*ppu,al=Math.min(1,f.l*2.5);X.fillStyle='rgba(255,150,30,'+al+')';X.beginPath();X.arc(q[0],q[1],rr,0,6.28);X.fill();X.fillStyle='rgba(255,240,140,'+al+')';X.beginPath();X.arc(q[0],q[1],rr*.5,0,6.28);X.fill()}
for(const e of ents){const s=sel.includes(e);if(e.t=='hrv'&&e.team==0&&e.cap>0){const q=w2s(e.x,e.y,e.r*3.4),w=30;X.fillStyle='#000a';X.fillRect(q[0]-w/2-1,q[1]-1,w+2,6);X.fillStyle=e.ct=='m'?'#ffd24a':e.ct=='s'?'#b8c4d0':'#ff9a3c';X.fillRect(q[0]-w/2,q[1],w*e.cap/60,4)}if(s&&e.mv&&e.team==0){const a=w2s(e.x,e.y,4),b=w2s(e.mv.x,e.mv.y,0);X.setLineDash([6,6]);X.strokeStyle='#ffffff99';X.lineWidth=1.5;X.beginPath();X.moveTo(a[0],a[1]);X.lineTo(b[0],b[1]);X.stroke();X.setLineDash([])}
if(e.hp<e.mx||s){const q=w2s(e.x,e.y,e.b?e.r*3.2:e.r*4.6),w=Math.max(26,e.r*2*ppu);X.fillStyle='#000a';X.fillRect(q[0]-w/2-1,q[1]-1,w+2,7);X.fillStyle=e.hp/e.mx>.4?'#7be04a':'#ffb02e';X.fillRect(q[0]-w/2,q[1],w*e.hp/e.mx,5)}}
if(box){X.strokeStyle='#d8b04a';X.lineWidth=1.5;X.strokeRect(box.x0,box.y0,box.x1-box.x0,box.y1-box.y0)}
if(msgT>0){X.fillStyle='#d8b04a';X.font='700 20px "Saira Condensed",sans-serif';X.fillText(msg,innerWidth/2,innerHeight-110)}
const mw=Math.min(170,innerWidth*.34),mh=mw*H/W,ox=innerWidth-mw-8,oy=innerHeight-mh-120,k=mw/W;MM={x:ox,y:oy,w:mw,h:mh,k};X.fillStyle='#1d2219cc';X.fillRect(ox,oy,mw,mh);
for(const n of NODES){if(n.a<=0)continue;X.fillStyle=n.t=='m'?'#ffd24a':n.t=='s'?'#b8c4d0':'#ff9a3c';X.fillRect(ox+n.x*k-1,oy+n.y*k-1,2,2)}
for(const s of S){X.fillStyle=s.o<0?'#ffd24a':COL[s.o];X.fillRect(ox+s.x*k-3,oy+s.y*k-3,6,6)}
for(const e of ents){X.fillStyle=COL[e.team];const q=e.b?3:1.5;X.fillRect(ox+e.x*k-q/2,oy+e.y*k-q/2,q,q)}
X.strokeStyle='#fff8';X.lineWidth=1;X.beginPath();[[0,0],[innerWidth,0],[innerWidth,innerHeight],[0,innerHeight]].map(p=>s2w(p[0],p[1])).forEach((p,i)=>X[i?'lineTo':'moveTo'](ox+Math.max(0,Math.min(W,p[0]))*k,oy+Math.max(0,Math.min(H,p[1]))*k));X.closePath();X.stroke()}
function draw2(){X.setTransform(1,0,0,1,0,0);X.fillStyle='#1d2219';X.fillRect(0,0,C.width,C.height);
X.setTransform(z*dpr,0,0,z*dpr,-cam.x*z*dpr,-cam.y*z*dpr);
if(!BG)mkBg();X.drawImage(BG,0,0,W,H);
X.strokeStyle='#5a6648';X.lineWidth=6;X.strokeRect(0,0,W,H);
X.textAlign='center';X.font='600 28px "Saira Condensed",sans-serif';
for(const s of S){const c=s.o<0?'#ffd24a':COL[s.o];X.fillStyle=c+'44';X.strokeStyle=c;X.lineWidth=4;X.setLineDash([14,10]);X.beginPath();X.arc(s.x,s.y,150,0,6.28);X.fill();X.stroke();X.setLineDash([]);
if(Math.abs(s.p)<100&&s.p){X.strokeStyle=COL[s.p>0?0:1];X.lineWidth=8;X.beginPath();X.arc(s.x,s.y,150,-1.57,-1.57+6.28*Math.abs(s.p)/100);X.stroke()}
X.fillStyle=c;X.fillText(s.n,s.x,s.y+60);X.strokeStyle='#eee';X.lineWidth=4;X.beginPath();X.moveTo(s.x,s.y+25);X.lineTo(s.x,s.y-50);X.stroke();const wv=Math.sin(performance.now()/300)*7;X.beginPath();X.moveTo(s.x,s.y-50);X.lineTo(s.x+46,s.y-40+wv);X.lineTo(s.x,s.y-26);X.fill()}
if(bm){X.strokeStyle='#ffffff22';X.lineWidth=3;for(const e of ents)if(e.team==0&&e.b){X.beginPath();X.arc(e.x,e.y,450,0,6.28);X.stroke()}}
for(const e of ents.slice().sort((a,b)=>a.y-b.y)){drawE(e);continue;const d=TY[e.t],c=COL[e.team],s=sel.includes(e);X.fillStyle='#0005';X.beginPath();X.ellipse(e.x+5,e.y+e.r*.7,e.r*1.05,e.r*.7,0,0,6.28);X.fill();
if(e.t=='tur'&&s){X.strokeStyle=c+'55';X.beginPath();X.arc(e.x,e.y,d.rng,0,6.28);X.stroke()}
if(e.b){X.fillStyle=e.team?'#8a3524':'#1f5f96';X.strokeStyle=c;X.lineWidth=s?6:3;const r=e.r;if(e.t=='tur'){X.beginPath();X.arc(e.x,e.y,r,0,6.28);X.fill();X.stroke()}else{X.fillRect(e.x-r,e.y-r,2*r,2*r);X.strokeRect(e.x-r,e.y-r,2*r,2*r)}
X.fillStyle=c;X.font='700 '+r+'px "Saira Condensed",sans-serif';X.fillText(d.g,e.x,e.y+r*.35)}
else{X.save();X.translate(e.x,e.y);X.rotate(e.a);X.fillStyle=c;X.strokeStyle='#fff';X.lineWidth=s?3:0;
if(e.t=='inf'){X.beginPath();X.arc(0,0,e.r,0,6.28);X.fill();if(s)X.stroke();X.fillRect(0,-1.5,e.r+5,3)}
else if(e.t=='tnk'){X.fillRect(-e.r,-e.r*.75,e.r*2,e.r*1.5);if(s)X.strokeRect(-e.r,-e.r*.75,e.r*2,e.r*1.5);X.fillStyle='#fff5';X.beginPath();X.arc(0,0,e.r*.5,0,6.28);X.fill();X.fillStyle='#222';X.fillRect(0,-2.5,e.r+10,5)}
else{X.fillRect(-e.r,-e.r*.7,e.r*1.7,e.r*1.4);if(s)X.strokeRect(-e.r,-e.r*.7,e.r*1.7,e.r*1.4);X.fillStyle='#222';X.fillRect(0,-3,e.r+22,6)}X.restore()}
if(e.hp<e.mx||s){const w=e.r*2;X.fillStyle='#000a';X.fillRect(e.x-w/2,e.y-e.r-10,w,5);X.fillStyle=e.hp/e.mx>.4?'#8bd35a':'#e0a03f';X.fillRect(e.x-w/2,e.y-e.r-10,w*e.hp/e.mx,5)}
if(e.mv&&s&&e.team==0){X.strokeStyle='#ffffff55';X.lineWidth=1.5;X.beginPath();X.moveTo(e.x,e.y);X.lineTo(e.mv.x,e.mv.y);X.stroke()}}
for(const p of proj){X.strokeStyle=p.c;X.lineWidth=p.w;X.beginPath();X.moveTo(p.x1,p.y1);X.lineTo(p.x2,p.y2);X.stroke()}
for(const f of fx){const rr=40*(1-f.l*2)+6,al=Math.min(1,f.l*2.5);X.fillStyle='rgba(255,150,30,'+al+')';X.beginPath();X.arc(f.x,f.y,rr,0,6.28);X.fill();X.fillStyle='rgba(255,240,140,'+al+')';X.beginPath();X.arc(f.x,f.y,rr*.5,0,6.28);X.fill()}
if(bm&&P.size<2){const d=TY[bm],x=mx/z+cam.x,y=my/z+cam.y,g=okp(x,y,d.r)&&ents.some(e=>e.team==0&&e.b&&Math.hypot(e.x-x,e.y-y)<450);X.fillStyle=g?'#6bd96b66':'#e0553f66';X.beginPath();X.arc(x,y,d.r,0,6.28);X.fill()}
X.setTransform(dpr,0,0,dpr,0,0);
if(box){X.strokeStyle='#d8b04a';X.lineWidth=1.5;X.strokeRect(box.x0,box.y0,box.x1-box.x0,box.y1-box.y0)}
if(msgT>0){X.fillStyle='#d8b04a';X.font='700 20px "Saira Condensed",sans-serif';X.textAlign='center';X.fillText(msg,innerWidth/2,innerHeight-110)}
const mw=Math.min(150,innerWidth*.3),mh=mw*H/W,ox=innerWidth-mw-8,oy=innerHeight-mh-110;
X.fillStyle='#1d2219cc';X.fillRect(ox,oy,mw,mh);const k=mw/W;
for(const s of S){X.fillStyle=s.o<0?'#c9b26a':COL[s.o];X.fillRect(ox+s.x*k-3,oy+s.y*k-3,6,6)}
for(const e of ents){X.fillStyle=COL[e.team];const q=e.b?3:1.5;X.fillRect(ox+e.x*k-q/2,oy+e.y*k-q/2,q,q)}
X.strokeStyle='#fff8';X.lineWidth=1;X.strokeRect(ox+cam.x*k,oy+cam.y*k,innerWidth/z*k,innerHeight/z*k)}
let last=performance.now(),ut=0;
function loop(n){const dt=Math.min(.05,(n-last)/1000);last=n;
if(run){const v=.5*cam.d*dt;if(K.w||K.arrowup)cam.y-=v;if(K.s||K.arrowdown)cam.y+=v;if(K.a&&0)0;if(K.d||K.arrowright)cam.x+=v;if(K.arrowleft)cam.x-=v;
if(!over){update(dt);sv-=dt;if(sv<0){sv=15;save(1)}}msgT-=dt;ut-=dt;if(ut<=0){ut=.25;ui()}draw()}
requestAnimationFrame(loop)}
requestAnimationFrame(loop);
</script></body></html>