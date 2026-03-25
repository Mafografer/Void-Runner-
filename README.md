<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>VOID RUNNER</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
html, body {
  width: 100%; height: 100%;
  background: #06060f;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  font-family: 'Courier New', monospace;
  user-select: none;
  -webkit-user-select: none;
}
canvas {
  display: block;
  cursor: pointer;
  image-rendering: pixelated;
}
</style>
</head>
<body>
<canvas id="c"></canvas>
<script>
(function(){

var canvas = document.getElementById(‘c’);
var ctx    = canvas.getContext(‘2d’);

var W = 360, H = 560;
canvas.width  = W;
canvas.height = H;

function resize(){
var scl = Math.min(window.innerWidth/W, window.innerHeight/H, 1.6);
canvas.style.width  = Math.floor(W*scl)+‘px’;
canvas.style.height = Math.floor(H*scl)+‘px’;
}
resize();
window.addEventListener(‘resize’, resize);

// save
var best=0, totalGames=0;
try{ var sv=JSON.parse(localStorage.getItem(‘vr3’)||’{}’); best=sv.b||0; totalGames=sv.g||0; }catch(e){}
function doSave(){ try{ localStorage.setItem(‘vr3’,JSON.stringify({b:best,g:totalGames})); }catch(e){} }

// constants
var GRAV=0.40, FLAP=-7.5, PW=50, GAP0=165, GAPMIN=115;
var SPD0=2.5, SPDMAX=5.0, PIPE_EVERY=85;

// game vars
var STATE=‘start’;
var bx,by,bvy,btilt,btrail;
var pipes,parts,frame,spd;
var score,streak,lives,iframes,shake;
var stars=[];
var msgT=0,msgS=’’,msgC=’#ffbe0b’;
var bestRun=0;

function rnd(n){ return Math.random()*n; }

function initStars(){
stars=[];
for(var i=0;i<100;i++) stars.push({x:rnd(W),y:rnd(H),r:rnd(1)+0.3,s:rnd(0.25)+0.05,a:rnd(0.5)+0.2});
}
initStars();

function reset(){
bx=80; by=H/2; bvy=0; btilt=0; btrail=[];
pipes=[]; parts=[];
frame=0; spd=SPD0;
score=0; streak=0; lives=3; iframes=0; shake=0;
bestRun=best;
msgT=0;
}

function showMsg(t,c){ msgS=t; msgC=c||’#ffbe0b’; msgT=55; }

function spawnPipe(){
var gap=Math.max(GAPMIN, GAP0-score*0.4);
var topH=50+rnd(H-100-gap);
var mv=score>12&&Math.random()<0.35;
pipes.push({x:W+PW, topH:topH, gap:gap, scored:false, mv:mv, md:1, ms:0.7+rnd(0.5)});
}

function burst(x,y,col,n){
for(var i=0;i<n;i++){
var a=(Math.PI*2*i/n)+rnd(0.8);
var s=rnd(3)+1.5;
parts.push({x:x,y:y,vx:Math.cos(a)*s,vy:Math.sin(a)*s,life:1,dc:rnd(0.04)+0.025,r:rnd(2)+1.2,col:col});
}
}

function flap(){ bvy=FLAP; btilt=-0.45; burst(bx-8,by,’#00f5ff55’,4); }

var DMSGS=[‘ONE MORE RUN’,‘SO CLOSE’,‘ALMOST!’,‘DON'T STOP’,‘KEEP GOING’,
‘NEXT ONE'S YOURS’,‘YOU GOT THIS’,‘TRY AGAIN’,‘NOT THIS TIME’,‘ALMOST PERFECT’];
function dmsg(s){
if(s===0) return ‘YOU CAN DO BETTER’;
if(s>=best*0.9&&s<best) return ‘SO CLOSE TO YOUR BEST!’;
return DMSGS[Math.floor(rnd(DMSGS.length))];
}

var lastDmsg=’’;
function setDead(){
lastDmsg=dmsg(score);
totalGames++;
if(score>best) best=score;
doSave();
STATE=‘dead’;
burst(bx,by,’#ff006e’,20);
}

function hitPipe(){
if(iframes>0) return;
lives–;
iframes=80; shake=14;
streak=0;
burst(bx,by,’#ff006e’,14);
if(lives<=0){ setDead(); }
}

function update(){
if(STATE!==‘play’) return;
frame++;
spd=Math.min(SPDMAX, SPD0+frame*0.0008);

bvy+=GRAV; by+=bvy;
btilt=Math.max(-0.5,Math.min(1.3, btilt+(bvy*0.045-btilt)*0.18));
btrail.push({x:bx,y:by});
if(btrail.length>9) btrail.shift();

if(iframes>0) iframes–;
if(shake>0) shake–;

if(by-10<0){ by=10; bvy=Math.abs(bvy)*0.4; hitPipe(); }
if(by+10>H){ by=H-10; bvy=-Math.abs(bvy)*0.4; hitPipe(); }

if(frame%PIPE_EVERY===0) spawnPipe();

var nm=false;
for(var i=0;i<pipes.length;i++){
var p=pipes[i];
p.x-=spd;
if(p.mv){ p.topH+=p.md*p.ms; if(p.topH<40||p.topH>H-40-p.gap) p.md*=-1; }

```
if(!p.scored&&p.x+PW<bx){
  p.scored=true; score++;
  streak=Math.min(10,streak+1);
  burst(bx+14,by,'#00f5ff44',5);
  if(score===10) showMsg('STREAK x10!','#00f5ff');
  else if(score===25) showMsg('ZONE SHIFT','#c46fff');
  else if(score%10===0) showMsg('SCORE '+score+'!','#ffbe0b');
}

var br=8;
var inX=bx+br>p.x&&bx-br<p.x+PW;
if(inX){
  if(by-br<p.topH){ hitPipe(); bvy=3; }
  else if(by+br>p.topH+p.gap){ hitPipe(); bvy=-3; }
}

var mg=14;
if(bx+br>p.x-mg&&bx-br<p.x+PW+mg){
  var ct=by-br-p.topH, cb=p.topH+p.gap-(by+br);
  if(ct>0&&cb>0&&(ct<mg||cb<mg)) nm=true;
}
```

}
if(nm) showMsg(‘NEAR MISS!’,’#ff006e’);
pipes=pipes.filter(function(p){ return p.x+PW>-5; });

for(var j=0;j<parts.length;j++){
var pt=parts[j];
pt.x+=pt.vx; pt.y+=pt.vy; pt.vy+=0.15; pt.life-=pt.dc;
}
parts=parts.filter(function(p){ return p.life>0; });

for(var k=0;k<stars.length;k++){
stars[k].x-=stars[k].s;
if(stars[k].x<0){ stars[k].x=W; stars[k].y=rnd(H); }
}

if(msgT>0) msgT–;
}

function zoneColor(){
var z=Math.floor(score/10);
var c=[’#00f5ff’,’#a855f7’,’#ff006e’,’#ffbe0b’,’#39ff14’];
return c[z%c.length];
}

function rrect(x,y,w,h,r){
r=Math.min(r,w/2,h/2);
ctx.beginPath();
ctx.moveTo(x+r,y); ctx.lineTo(x+w-r,y); ctx.quadraticCurveTo(x+w,y,x+w,y+r);
ctx.lineTo(x+w,y+h-r); ctx.quadraticCurveTo(x+w,y+h,x+w-r,y+h);
ctx.lineTo(x+r,y+h); ctx.quadraticCurveTo(x,y+h,x,y+h-r);
ctx.lineTo(x,y+r); ctx.quadraticCurveTo(x,y,x+r,y);
ctx.closePath();
}

function draw(){
var sx=shake>0?(rnd(shake)-shake/2)*0.7:0;
var sy=shake>0?(rnd(shake)-shake/2)*0.4:0;
ctx.save();
ctx.translate(sx,sy);

// bg
var bg=ctx.createLinearGradient(0,0,0,H);
bg.addColorStop(0,’#06060f’); bg.addColorStop(1,’#0c0920’);
ctx.fillStyle=bg; ctx.fillRect(0,0,W,H);

// grid
ctx.strokeStyle=’#ffffff07’; ctx.lineWidth=1;
for(var gx=0;gx<W;gx+=36){ ctx.beginPath(); ctx.moveTo(gx,0); ctx.lineTo(gx,H); ctx.stroke(); }
for(var gy=0;gy<H;gy+=36){ ctx.beginPath(); ctx.moveTo(0,gy); ctx.lineTo(W,gy); ctx.stroke(); }

// stars
for(var si=0;si<stars.length;si++){
var s=stars[si];
ctx.globalAlpha=s.a*(0.5+0.5*Math.sin(Date.now()*0.001+si));
ctx.fillStyle=’#fff’;
ctx.beginPath(); ctx.arc(s.x,s.y,s.r,0,Math.PI*2); ctx.fill();
}
ctx.globalAlpha=1;

// pipes
var pc=zoneColor();
for(var pi=0;pi<pipes.length;pi++){
var p=pipes[pi];
ctx.save();
ctx.shadowColor=pc; ctx.shadowBlur=14;
var g1=ctx.createLinearGradient(p.x,0,p.x+PW,0);
g1.addColorStop(0,pc+‘88’); g1.addColorStop(0.5,pc); g1.addColorStop(1,pc+‘55’);
ctx.fillStyle=g1;
ctx.fillRect(p.x,0,PW,p.topH);
ctx.fillRect(p.x-5,p.topH-12,PW+10,12);
var by2=p.topH+p.gap;
ctx.fillRect(p.x,by2,PW,H-by2);
ctx.fillRect(p.x-5,by2,PW+10,12);
ctx.restore();
}

// trail
for(var ti=0;ti<btrail.length;ti++){
var tr=btrail[ti];
var a=(ti/btrail.length)*0.22;
var r=(ti/btrail.length)*10*0.55;
ctx.globalAlpha=a;
ctx.fillStyle=’#00f5ff’;
ctx.beginPath(); ctx.arc(tr.x,tr.y,r,0,Math.PI*2); ctx.fill();
}
ctx.globalAlpha=1;

// bird
var blink=iframes>0&&Math.floor(iframes/6)%2===0;
if(!blink){
ctx.save();
ctx.translate(bx,by); ctx.rotate(btilt);
ctx.shadowColor=’#00f5ff’; ctx.shadowBlur=20;
var bgr=ctx.createRadialGradient(0,0,0,0,0,10);
bgr.addColorStop(0,’#ffffff’); bgr.addColorStop(0.4,’#00f5ff’); bgr.addColorStop(1,’#004488’);
ctx.fillStyle=bgr;
ctx.beginPath(); ctx.arc(0,0,10,0,Math.PI*2); ctx.fill();
ctx.fillStyle=‘rgba(255,255,255,0.3)’;
ctx.beginPath(); ctx.moveTo(0,-9); ctx.lineTo(4.5,0); ctx.lineTo(0,9); ctx.lineTo(-4.5,0); ctx.closePath(); ctx.fill();
ctx.restore();
}

// particles
for(var xi=0;xi<parts.length;xi++){
var pt=parts[xi];
ctx.globalAlpha=pt.life;
ctx.fillStyle=pt.col;
ctx.beginPath(); ctx.arc(pt.x,pt.y,pt.r*pt.life,0,Math.PI*2); ctx.fill();
}
ctx.globalAlpha=1;

// HUD (during play or dead overlay)
if(STATE===‘play’||STATE===‘dead’){
// score
ctx.textAlign=‘center’;
ctx.font=‘bold 30px “Courier New”’;
ctx.fillStyle=’#00f5ff’;
ctx.shadowColor=’#00f5ff’; ctx.shadowBlur=12;
ctx.fillText(score, W/2, 42);
ctx.shadowBlur=0;

```
// best
ctx.font='11px "Courier New"';
ctx.fillStyle='#ffffff44';
ctx.textAlign='right';
ctx.fillText('BEST '+bestRun, W-12, 20);

// lives (diamonds)
for(var li=0;li<3;li++){
  var alive=li<lives;
  ctx.fillStyle=alive?'#ff006e':'#ffffff15';
  ctx.shadowColor=alive?'#ff006e':'transparent';
  ctx.shadowBlur=alive?8:0;
  var dx=13+li*17, dy=13;
  ctx.beginPath(); ctx.moveTo(dx,dy-5); ctx.lineTo(dx+5,dy); ctx.lineTo(dx,dy+5); ctx.lineTo(dx-5,dy); ctx.closePath(); ctx.fill();
}
ctx.shadowBlur=0;

// streak bar
ctx.fillStyle='#ffffff0a';
ctx.fillRect(14,H-8,W-28,3);
if(streak>0){
  var sg=ctx.createLinearGradient(14,0,W-14,0);
  sg.addColorStop(0,'#00f5ff'); sg.addColorStop(1,'#ff006e');
  ctx.fillStyle=sg;
  ctx.shadowColor='#00f5ff'; ctx.shadowBlur=6;
  ctx.fillRect(14,H-8,(W-28)*(streak/10),3);
  ctx.shadowBlur=0;
}

// combo msg
if(msgT>0){
  ctx.globalAlpha=Math.min(1,msgT/12);
  ctx.font='bold 15px "Courier New"';
  ctx.fillStyle=msgC;
  ctx.shadowColor=msgC; ctx.shadowBlur=12;
  ctx.textAlign='center';
  ctx.fillText(msgS, W/2, H/2-30);
  ctx.shadowBlur=0;
  ctx.globalAlpha=1;
}
```

}

// START SCREEN
if(STATE===‘start’){
ctx.fillStyle=‘rgba(6,6,15,0.75)’;
ctx.fillRect(0,0,W,H);

```
ctx.textAlign='center';
ctx.font='bold 52px "Courier New"';
ctx.fillStyle='#00f5ff';
ctx.shadowColor='#00f5ff'; ctx.shadowBlur=28;
ctx.fillText('VOID', W/2, H/2-60);
ctx.fillText('RUNNER', W/2, H/2-4);
ctx.shadowBlur=0;

ctx.font='10px "Courier New"';
ctx.fillStyle='#ffffff33';
ctx.fillText('∞  ENDLESS  ∞', W/2, H/2+18);

var p2=0.35+0.65*Math.abs(Math.sin(Date.now()*0.0025));
ctx.globalAlpha=p2;
ctx.font='bold 13px "Courier New"';
ctx.fillStyle='#ffffff';
ctx.fillText('TAP  /  CLICK  /  SPACE', W/2, H/2+58);
ctx.globalAlpha=1;

if(totalGames>0){
  ctx.font='10px "Courier New"';
  ctx.fillStyle='#ffffff25';
  ctx.fillText('BEST: '+best+'   RUNS: '+totalGames, W/2, H/2+88);
}
```

}

// DEAD SCREEN
if(STATE===‘dead’){
ctx.fillStyle=‘rgba(6,6,15,0.82)’;
ctx.fillRect(0,0,W,H);
ctx.textAlign=‘center’;

```
ctx.font='10px "Courier New"';
ctx.fillStyle='#ffffff2a';
ctx.fillText('SIGNAL LOST', W/2, H/2-96);

ctx.font='bold 76px "Courier New"';
ctx.fillStyle='#ff006e';
ctx.shadowColor='#ff006e'; ctx.shadowBlur=28;
ctx.fillText(score, W/2, H/2-22);
ctx.shadowBlur=0;

ctx.font='9px "Courier New"';
ctx.fillStyle='#ffffff33';
ctx.fillText('S C O R E', W/2, H/2-4);

if(score>0&&score>=best){
  var gb=Math.floor(Date.now()/320)%2===0;
  ctx.globalAlpha=gb?1:0.45;
  ctx.font='bold 11px "Courier New"';
  ctx.fillStyle='#ffbe0b';
  ctx.shadowColor='#ffbe0b'; ctx.shadowBlur=12;
  ctx.fillText('✦  NEW RECORD  ✦', W/2, H/2+22);
  ctx.shadowBlur=0;
  ctx.globalAlpha=1;
}

ctx.font='11px "Courier New"';
ctx.fillStyle='#ffbe0b';
ctx.fillText(lastDmsg, W/2, H/2+46);

// retry button
var rbx=W/2-56, rby=H/2+62, rbw=112, rbh=36;
ctx.strokeStyle='#00f5ff'; ctx.lineWidth=1.5;
ctx.shadowColor='#00f5ff'; ctx.shadowBlur=10;
rrect(rbx,rby,rbw,rbh,4); ctx.stroke();
ctx.shadowBlur=0;
ctx.font='bold 13px "Courier New"';
ctx.fillStyle='#00f5ff';
ctx.fillText('RETRY', W/2, rby+rbh/2+5);

ctx.font='9px "Courier New"';
ctx.fillStyle='#ffffff18';
ctx.fillText('RUN #'+(totalGames+1), W/2, H-14);
```

}

ctx.restore();
}

function action(){
if(STATE===‘start’){ STATE=‘play’; reset(); flap(); }
else if(STATE===‘play’){ flap(); }
else if(STATE===‘dead’){ STATE=‘play’; reset(); flap(); }
}

canvas.addEventListener(‘pointerdown’, function(e){ e.preventDefault(); action(); });
document.addEventListener(‘keydown’, function(e){
if(e.code===‘Space’||e.code===‘ArrowUp’){ e.preventDefault(); action(); }
});

STATE=‘start’;
reset();

function loop(){
update();
draw();
requestAnimationFrame(loop);
}
loop();

})();
</script>

</body>
</html>
