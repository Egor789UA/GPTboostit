<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>Мульти-Аркада Локальна</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{margin:0; background:#111; overflow:hidden; font-family:sans-serif; color:white;}
canvas{display:block;}
#hud{position:absolute;top:5px; left:10px;}
#settings{position:absolute;top:70px; left:10px;}
#menu{position:absolute;top:140px; left:10px;}
#mobile-controls{position:absolute;bottom:10px;width:100%;display:flex; justify-content:space-around;}
button.control{width:60px;height:60px;font-size:24px;opacity:0.5;}
</style>
</head>
<body>

<canvas id="game"></canvas>

<div id="hud">
  FPS: <span id="fpsCounter">0</span> | 
  Score: <span id="scoreCounter">0</span> | 
  Health: <span id="healthCounter">3</span>
</div>

<div id="settings">
  <label>Max FPS: <input type="number" id="maxFPSInput" value="144" min="30" max="240"></label>
  <button onclick="togglePause()">Пауза</button>
  <button onclick="resetGame()">Скинути гру</button>
</div>

<div id="menu">
  <button onclick="switchGame('flappy')">Flappy Bird</button>
  <button onclick="switchGame('road')">Безкінечна дорога</button>
  <button onclick="switchGame('autogun')">Автострілялка</button>
  <button onclick="switchGame('blocks')">Падаючі блоки</button>
  <button onclick="switchGame('breakout')">Breakout</button>
  <button onclick="switchGame('sound')">Звуковий контроль</button>
</div>

<div id="mobile-controls">
  <button class="control" id="left">◀</button>
  <button class="control" id="jump">▲</button>
  <button class="control" id="right">▶</button>
</div>

<script>
const canvas=document.getElementById('game');
const ctx=canvas.getContext('2d');
canvas.width=window.innerWidth;
canvas.height=window.innerHeight;

let fps=0, frames=0, lastTime=performance.now(), lastFrameTime=performance.now();
let maxFPS=144, frameInterval=1000/maxFPS;
let score=0, health=3, paused=false, currentGame='flappy';
const keys={};

// ======= Клавіатура + сенсор =======
document.addEventListener('keydown', e=>keys[e.code]=true);
document.addEventListener('keyup', e=>keys[e.code]=false);
document.addEventListener('mousedown', ()=>{if(currentGame==='flappy') flappyJump(); if(currentGame==='sound') soundTrigger();});
document.addEventListener('touchstart', ()=>{if(currentGame==='flappy') flappyJump(); if(currentGame==='sound') soundTrigger();});
document.getElementById('left').addEventListener('touchstart', ()=>keys['ArrowLeft']=true);
document.getElementById('left').addEventListener('touchend', ()=>keys['ArrowLeft']=false);
document.getElementById('right').addEventListener('touchstart', ()=>keys['ArrowRight']=true);
document.getElementById('right').addEventListener('touchend', ()=>keys['ArrowRight']=false);
document.getElementById('jump').addEventListener('touchstart', ()=>{if(currentGame==='flappy') flappyJump();});

// ======= HUD =======
document.getElementById('maxFPSInput').addEventListener('change',(e)=>{
  let val=parseInt(e.target.value);
  if(val>=30 && val<=240){ maxFPS=val; frameInterval=1000/maxFPS; }
});
function updateHUD(){
  document.getElementById('fpsCounter').innerText=fps;
  document.getElementById('scoreCounter').innerText=score;
  document.getElementById('healthCounter').innerText=health;
}
function togglePause(){paused=!paused; alert(paused?"Гра на паузі":"Гра відновлена");}
function resetGame(){score=0; health=3; initGame(); updateHUD(); saveProgress();}

// ======= Збереження прогресу =======
function saveProgress(){
  localStorage.setItem('arcade_score',score);
  localStorage.setItem('arcade_health',health);
  localStorage.setItem('arcade_game',currentGame);
}
function loadProgress(){
  let s=localStorage.getItem('arcade_score');
  let h=localStorage.getItem('arcade_health');
  let g=localStorage.getItem('arcade_game');
  if(s!==null) score=parseInt(s);
  if(h!==null) health=parseInt(h);
  if(g!==null) currentGame=g;
}

// ======= Switch Game =======
function switchGame(name){ currentGame=name; score=0; health=3; initGame(); saveProgress();}

// ======= Ініціалізація ігор =======
function initGame(){
  if(currentGame==='flappy') initFlappy();
  else if(currentGame==='road') initRoad();
  else if(currentGame==='autogun') initAutoGun();
  else if(currentGame==='blocks') initBlocks();
  else if(currentGame==='breakout') initBreakout();
  else if(currentGame==='sound') initSound();
}

// ======= Головний цикл =======
function loop(now){
  requestAnimationFrame(loop);
  if(paused) return;
  const delta=now-lastFrameTime;
  if(delta<frameInterval) return;
  lastFrameTime=now;

  frames++;
  if(now-lastTime>=1000){fps=frames; frames=0; lastTime=now;}

  if(currentGame==='flappy'){updateFlappy(); drawFlappy();}
  else if(currentGame==='road'){updateRoad(); drawRoad();}
  else if(currentGame==='autogun'){updateAutoGun(); drawAutoGun();}
  else if(currentGame==='blocks'){updateBlocks(); drawBlocks();}
  else if(currentGame==='breakout'){updateBreakout(); drawBreakout();}
  else if(currentGame==='sound'){updateSound(); drawSound();}

  updateHUD();
  if(health<=0){alert(`Game Over! Score: ${score}`); resetGame();}
}

loadProgress();
initGame();
loop(performance.now());

// ======= ТУТ вставляєш функції ігор =======
// initFlappy, updateFlappy, drawFlappy, flappyJump
// initRoad, updateRoad, drawRoad
// initAutoGun, updateAutoGun, drawAutoGun
// initBlocks, updateBlocks, drawBlocks
// initBreakout, updateBreakout, drawBreakout
// initSound, updateSound, drawSound, soundTrigger

</script>
</body>
</html>
