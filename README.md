<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<title>Мульти-Аркада ПК+Мобільні</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{margin:0; background:#111; overflow:hidden; font-family:sans-serif; color:white;}
canvas{display:block;}
#hud{position:absolute;top:5px; left:10px;}
#settings{position:absolute;top:70px; left:10px;}
#menu{position:absolute;top:140px; left:10px;}
#auth{position:absolute;top:100px; left:50%; transform:translateX(-50%); background:#222; padding:20px; border-radius:10px;}
#auth input{display:block; margin:10px 0; padding:5px; width:200px;}
#mobile-controls{position:absolute;bottom:10px;width:100%;display:flex; justify-content:space-around;}
button.control{width:60px;height:60px;font-size:24px;opacity:0.5;}
#leaderboard{position:absolute;top:300px; left:10px; background:#222; padding:10px; border-radius:5px; max-width:300px;}
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
  <button onclick="showLeaderboard()">Показати лідерів</button>
</div>

<div id="auth">
  <input id="email" placeholder="Email">
  <input id="password" type="password" placeholder="Пароль">
  <button onclick="signup()">Реєстрація</button>
  <button onclick="login()">Вхід</button>
</div>

<div id="mobile-controls">
  <button class="control" id="left">◀</button>
  <button class="control" id="jump">▲</button>
  <button class="control" id="right">▶</button>
</div>

<div id="leaderboard"><h3>Лідери</h3><ul id="leaderList"></ul></div>

<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>

<script>
// ================= Firebase =================
const firebaseConfig = {
  apiKey: "ВАШ_API_KEY",
  authDomain: "ВАШ_PROJECT.firebaseapp.com",
  projectId: "ВАШ_PROJECT",
  storageBucket: "ВАШ_PROJECT.appspot.com",
  messagingSenderId: "ВАШ_ID",
  appId: "ВАШ_APP_ID"
};
firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const db = firebase.firestore();
let currentUser = null;

// ================= Авторизація =================
function signup() {
  const email = document.getElementById('email').value.trim();
  const password = document.getElementById('password').value;
  if (!email || !password) { alert("Введіть email та пароль!"); return; }
  auth.createUserWithEmailAndPassword(email, password)
    .then((userCredential) => {
      currentUser = userCredential.user;
      alert("Реєстрація успішна! Ви увійшли як " + currentUser.email);
      loadProgress(); // Завантажити прогрес після реєстрації
    })
    .catch((error) => { alert("Помилка реєстрації: " + error.message); });
}

function login() {
  const email = document.getElementById('email').value.trim();
  const password = document.getElementById('password').value;
  if (!email || !password) { alert("Введіть email та пароль!"); return; }
  auth.signInWithEmailAndPassword(email, password)
    .then((userCredential) => {
      currentUser = userCredential.user;
      alert("Вхід успішний! Ви увійшли як " + currentUser.email);
      loadProgress(); // Завантажити прогрес після входу
    })
    .catch((error) => { alert("Помилка входу: " + error.message); });
}

auth.onAuthStateChanged((user) => { if (user) { currentUser = user; loadProgress(); } else currentUser = null; });

// ================= Canvas =================
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

let fps = 0, lastTime = performance.now(), frames = 0, lastFrameTime = performance.now();
let maxFPS = 144, frameInterval = 1000/maxFPS;
let score = 0, health = 3;
let paused = false;
let currentGame = 'flappy';
const keys = {};

// ================= Клавіатура + сенсор =================
document.addEventListener('keydown', e => keys[e.code]=true);
document.addEventListener('keyup', e => keys[e.code]=false);
document.addEventListener('mousedown', () => { if(currentGame==='flappy') flappyJump(); if(currentGame==='sound') soundTrigger(); });
document.addEventListener('touchstart', () => { if(currentGame==='flappy') flappyJump(); if(currentGame==='sound') soundTrigger(); });
document.getElementById('left').addEventListener('touchstart', ()=>keys['ArrowLeft']=true);
document.getElementById('left').addEventListener('touchend', ()=>keys['ArrowLeft']=false);
document.getElementById('right').addEventListener('touchstart', ()=>keys['ArrowRight']=true);
document.getElementById('right').addEventListener('touchend', ()=>keys['ArrowRight']=false);
document.getElementById('jump').addEventListener('touchstart', ()=>{ if(currentGame==='flappy') flappyJump(); });

// ================= FPS та HUD =================
document.getElementById('maxFPSInput').addEventListener('change', (e)=> {
  let val=parseInt(e.target.value);
  if(val>=30 && val<=240){ maxFPS=val; frameInterval=1000/maxFPS; }
});

function updateHUD(){
  document.getElementById('fpsCounter').innerText = fps;
  document.getElementById('scoreCounter').innerText = score;
  document.getElementById('healthCounter').innerText = health;
}

function togglePause(){ paused=!paused; alert(paused ? "Гра на паузі" : "Гра відновлена"); }
function resetGame(){ score=0; health=3; initGame(); updateHUD(); saveProgress(); }

// ================= Switch Game =================
function switchGame(name){ currentGame=name; score=0; health=3; initGame(); saveProgress(); }

// ================= Збереження прогресу =================
function getDeviceType(){ return /Mobi|Android/i.test(navigator.userAgent)?'mobile':'pc'; }

function saveProgress(){
  if(!currentUser) return;
  let collection = getDeviceType()==='pc'?'progress_pc':'progress_mobile';
  db.collection(collection).doc(currentUser.uid).set({
    score: score,
    health: health,
    game: currentGame,
    timestamp: Date.now()
  }, {merge:true});
}

async function loadProgress(){
  if(!currentUser) return;
  let collection = getDeviceType()==='pc'?'progress_pc':'progress_mobile';
  let docSnap = await db.collection(collection).doc(currentUser.uid).get();
  if(docSnap.exists){
    let data = docSnap.data();
    score = data.score || 0;
    health = data.health || 3;
    currentGame = data.game || 'flappy';
    initGame();
    updateHUD();
  }
}

// ================= Лідерборд =================
async function loadLeaderboard(collection){
  let snapshot = await db.collection(collection).orderBy('score','desc').limit(10).get();
  let leaders = [];
  snapshot.forEach(doc=>leaders.push(doc.data()));
  return leaders;
}

async function showLeaderboard(){
  let isMobile = getDeviceType()==='mobile';
  let ownLeaders = await loadLeaderboard(isMobile?'scores_mobile':'scores_pc');
  let otherLeaders = await loadLeaderboard(isMobile?'scores_pc':'scores_mobile');
  let listEl=document.getElementById('leaderList');
  listEl.innerHTML='<li><b>Ваша платформа:</b></li>';
  ownLeaders.forEach(l=>listEl.innerHTML+=`<li>${l.game}: ${l.score}</li>`);
  listEl.innerHTML+='<li><b>Інша платформа:</b></li>';
  otherLeaders.forEach(l=>listEl.innerHTML+=`<li>${l.game}: ${l.score}</li>`);
}

// ================= Ініціалізація ігор =================
function initGame(){ if(currentGame==='flappy') initFlappy(); else if(currentGame==='road') initRoad(); else if(currentGame==='autogun') initAutoGun();
else if(currentGame==='blocks') initBlocks(); else if(currentGame==='breakout') initBreakout(); else if(currentGame==='sound') initSound(); }

// ================= Головний цикл =================
function loop(now){
  requestAnimationFrame(loop);
  if(paused) return;
  const delta = now - lastFrameTime;
  if(delta < frameInterval) return;
  lastFrameTime
