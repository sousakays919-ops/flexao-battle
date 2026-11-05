<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Flexão Battle</title>
<style>
:root{--bg:#0b0f0c;--card:#151c17;--tx:#eafff0;--mut:#8aa093;--ac:#22e55f;--bad:#ff4d4d;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
*{box-sizing:border-box}
html,body{margin:0;background:var(--bg);color:var(--tx);font-family:system-ui,-apple-system,Arial,sans-serif;color-scheme:dark}
main{max-width:480px;margin:0 auto;padding:16px}
h1{font-size:22px;margin:0 0 4px}
p.s{color:var(--mut);margin:0 0 14px;font-size:14px}
.bots{display:grid;grid-template-columns:repeat(5,1fr);gap:6px;margin-bottom:12px}
.bots button{background:var(--card);color:var(--tx);border:2px solid transparent;border-radius:10px;padding:10px 2px;font-size:12px;cursor:pointer}
.bots button.on{border-color:var(--ac)}
.bots b{display:block;font-size:15px;color:var(--ac)}
.row{background:var(--card);border-radius:14px;padding:12px;margin-bottom:10px}
.row .t{display:flex;justify-content:space-between;font-size:14px;color:var(--mut)}
.row .n{font-size:44px;font-weight:800;line-height:1.1}
.bar{height:10px;background:#0b0f0c;border-radius:6px;overflow:hidden;margin-top:6px}
.bar i{display:block;height:100%;width:0;background:var(--ac);transition:width .3s}
.bar.b i{background:var(--bad)}
button.big{width:100%;border:0;border-radius:12px;padding:14px;font-size:17px;font-weight:700;background:var(--ac);color:#04200d;margin-top:4px;cursor:pointer}
button.sec{background:var(--card);color:var(--tx)}
.mode{display:grid;grid-template-columns:1fr 1fr;gap:6px;margin-bottom:12px}.mode button{background:var(--card);color:var(--tx);border:2px solid transparent;border-radius:10px;padding:10px;font-size:13px;cursor:pointer}.mode button.on{border-color:var(--ac)}video{width:100%;max-height:240px;object-fit:cover;border-radius:10px;background:#000;margin-top:8px;display:none}
#msg{text-align:center;font-size:18px;font-weight:700;margin:10px 0;min-height:26px}
.small{font-size:12px;color:var(--mut);text-align:center;margin-top:8px}
</style>
</head>
<body>
<main>
<h1>💪 Flexão Battle</h1>
<p class="s">Toque em Começar, permita a câmera e faça flexões. O app conta sozinho.</p>
<div class="bots" id="bots"></div>
<div class="mode"><button id="m1">📱 Celular na frente</button><button id="m2">⬆ Celular no chão</button></div>
<div class="row"><div class="t"><span>Você</span><span id="pl">meta: —</span></div><div class="n" id="pn">0</div><div class="bar"><i id="pb"></i></div></div>
<div class="row"><div class="t"><span id="bname">Bot</span><span id="bl"></span></div><div class="n" id="bn">0</div><div class="bar b"><i id="bb"></i></div></div>
<div id="perm" class="row" style="display:none;text-align:center"><div style="font-size:16px;margin-bottom:10px">📷 Permitir acesso à sua câmera para detectar as flexões?</div><button class="big" id="pyes">✅ Sim, permitir</button><button class="big sec" id="pno">Não, usar manual</button></div>
<div id="msg"></div>
<button class="big" id="go">▶ Começar</button>
<button class="big sec" id="tap">+1 flexão (manual)</button>
<video id="v" playsinline muted></video>
<canvas id="c" width="32" height="24" style="display:none"></canvas>
<p class="small" id="cam">A câmera frontal detecta quando seu corpo cobre a tela. Se não funcionar, use o botão manual.</p>
</main>
<script>
var BOTS=[{n:"Noob",s:4,m:10},{n:"Iniciante",s:2,m:28},{n:"Pro",s:2,m:68},{n:"Hacker",s:2,m:121},{n:"Deus",s:1,m:189}];
var sel=0,pc=0,bc=0,run=false,timer=null,stream=null,raf=null,sid=0,starting=false,camYes=null;
var hi=0,lo=0,down=false,first=true,lastAt=0,mode="frente",prv=new Float32Array(768),es=0,mx=0,burst=false,nb=0;
var $=function(i){return document.getElementById(i)};
function draw(){
  var h="";BOTS.forEach(function(b,i){h+='<button data-i="'+i+'" class="'+(i==sel?'on':'')+'">'+b.n+'<b>'+b.m+'</b>'+b.s+'s</button>'});
  $("bots").innerHTML=h;
  $("bots").querySelectorAll("button").forEach(function(b){b.onclick=function(){if(!run){sel=+b.dataset.i;draw();upd()}}});
}
function upd(){
  var b=BOTS[sel];
  $("pn").textContent=pc;$("bn").textContent=bc;
  $("pl").textContent="meta: "+(b.m+1);
  $("bname").textContent="🤖 "+b.n;$("bl").textContent="aguenta até "+b.m;
  $("pb").style.width=Math.min(100,pc/(b.m+1)*100)+"%";
  $("bb").style.width=Math.min(100,bc/b.m*100)+"%";
}
function beep(){try{var a=new (window.AudioContext||window.webkitAudioContext)(),o=a.createOscillator();o.connect(a.destination);o.frequency.value=700;o.start();o.stop(a.currentTime+.08)}catch(e){}}
function count(){
  if(!run)return;
  pc++;beep();upd();
  if(pc>BOTS[sel].m)end(true);
}
function end(win){
  run=false;sid++;starting=false;clearInterval(timer);cancelAnimationFrame(raf);
  $("msg").textContent=win?"🏆 Você venceu o "+BOTS[sel].n+"!":"💀 O "+BOTS[sel].n+" ganhou de você!";
  $("go").textContent="▶ Jogar de novo";
  if(stream){stream.getTracks().forEach(function(t){t.stop()});stream=null}
  $("v").style.display="none";
}
function loop(){
  if(!run)return;
  var v=$("v"),c=$("c"),x=c.getContext("2d");
  if(v.readyState>=2){
    x.drawImage(v,0,0,32,24);
    var d=x.getImageData(0,0,32,24).data,s=0,m=0;
    for(var i=0;i<d.length;i+=4){var g=d[i]*.3+d[i+1]*.59+d[i+2]*.11;s+=g;m+=Math.abs(g-prv[i/4]);prv[i/4]=g}
    var l=s/(d.length/4);
    if(mode=="frente"){
      var E=m/(d.length/4);es=es*.6+E*.4;mx=Math.max(es,mx*.995);var nw=Date.now();
      if(!burst&&es>mx*.5&&es>1.5){burst=true;nb++;if(nb%2==0&&nw-lastAt>500){lastAt=nw;count()}}
      else if(burst&&es<mx*.25){burst=false}
      raf=requestAnimationFrame(loop);return;
    }
    if(first){hi=lo=l;first=false}
    hi-=(hi-l)*.01;lo+=(l-lo)*.01;
    if(l>hi)hi=l;if(l<lo)lo=l;
    var r=hi-lo;
    if(r>15){
      var now=Date.now();
      if(!down&&l<lo+r*.3){down=true}
      else if(down&&l>lo+r*.7&&now-lastAt>600){down=false;lastAt=now;count()}
    }
  }
  raf=requestAnimationFrame(loop);
}
function start(){
  if(starting||run)return;starting=true;var my=++sid;clearInterval(timer);$("go").textContent="Cancelar";
  pc=0;bc=0;down=false;first=true;nb=0;burst=false;mx=0;es=0;$("msg").textContent="Prepare-se...";upd();
  var b=BOTS[sel];
  var go=function(){
    if(my!==sid)return;starting=false;run=true;$("msg").textContent="VAI!";$("go").textContent="⏹ Desistir";
    timer=setInterval(function(){if(bc<b.m){bc++;upd()}if(bc>=b.m){clearInterval(timer)}},b.s*1000);
    if(stream)loop();
  };
  var ask=function(){
  try{
    navigator.mediaDevices.getUserMedia({video:{facingMode:"user"}}).then(function(s){
      if(my!==sid){s.getTracks().forEach(function(t){t.stop()});return}
      stream=s;var v=$("v");v.srcObject=s;v.style.display="block";v.play();
      $("cam").textContent="Câmera ativa. Fique em cima do celular e desça até cobrir a tela.";
      setTimeout(go,3000);
    }).catch(function(){
      camYes=null;$("cam").textContent="A câmera foi bloqueada. Abra o link direto no Chrome ou Safari e toque em Permitir. Por enquanto, use o botão manual.";setTimeout(go,3000);
    });
  }catch(e){$("cam").textContent="Sem câmera neste navegador. Use o botão manual.";setTimeout(go,3000)}
  };
  if(camYes===true){ask()}
  else if(camYes===false){$("cam").textContent="Câmera desativada. Use o botão manual.";setTimeout(go,3000)}
  else{$("perm").style.display="block";
    $("pyes").onclick=function(){$("perm").style.display="none";camYes=true;ask()};
    $("pno").onclick=function(){$("perm").style.display="none";camYes=false;$("cam").textContent="Sem câmera. Use o botão manual.";setTimeout(go,3000)};
  }
}
$("go").onclick=function(){
  if(starting){sid++;starting=false;$("perm").style.display="none";if(stream){stream.getTracks().forEach(function(t){t.stop()});stream=null}$("v").style.display="none";$("msg").textContent="Cancelado";$("go").textContent="▶ Começar";return}
  if(run){end(false);$("msg").textContent="Você desistiu 🏳️";return}
  start();
};
$("tap").onclick=count;
function setMode(m){if(run)return;mode=m;$("m1").className=m=="frente"?"on":"";$("m2").className=m=="chao"?"on":""}
$("m1").onclick=function(){setMode("frente")};$("m2").onclick=function(){setMode("chao")};
$("m1").className="on";
draw();upd();
</script>
</body>
</html>
