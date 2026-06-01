<!DOCTYPE html>
<html lang="pt-BR">
<head>
<!-- Playables SDK -->
<script>// Playables SDK v1.0.0
// Game lifecycle bridge: rAF-based game-ready detection + event communication
(function() {
  'use strict';

  // Idempotency: skip if already initialized (e.g., server-side injection
  // followed by client-side inject-javascript via the Bloks webview component).
  if (window.playablesSDK) return;

  var HANDLER_NAME = 'playablesGameEventHandler';
  var ANDROID_BRIDGE_NAME = '_MetaPlayablesBridge';
  var RAF_FRAME_THRESHOLD = 3;

  var gameReadySent = false;
  var firstInteractionSent = false;
  var errorSent = false;
  var frameCount = 0;
  var originalRAF = window.requestAnimationFrame;

  // --- Transport Layer ---

  function hasIOSBridge() {
    return !!(window.webkit &&
              window.webkit.messageHandlers &&
              window.webkit.messageHandlers[HANDLER_NAME]);
  }

  function hasAndroidBridge() {
    return !!(window[ANDROID_BRIDGE_NAME] &&
              typeof window[ANDROID_BRIDGE_NAME].postEvent === 'function');
  }

  function isInIframe() {
    return !!(window.parent && window.parent !== window);
  }

  function sendEvent(eventName, payload) {
    var message = {
      type: eventName,
      payload: payload || {},
      timestamp: Date.now()
    };

    if (hasIOSBridge()) {
      try {
        window.webkit.messageHandlers[HANDLER_NAME].postMessage(message);
      } catch (e) { /* ignore */ }
      return;
    }

    if (hasAndroidBridge()) {
    try {
      var p = payload || {};
      p.__secureToken = window.__fbAndroidBridgeAuthToken || '';
      p.timestamp = message.timestamp;
      window[ANDROID_BRIDGE_NAME].postEvent(
        eventName,
        JSON.stringify(p)
      );
    } catch (e) { /* ignore */ }
    return;
  }

    if (isInIframe()) {
      try {
        window.parent.postMessage(message, '*');
      } catch (e) { /* ignore */ }
      return;
    }
  }

  // --- rAF Game-Ready Detection ---

  function onFrame() {
    if (gameReadySent) return;

    frameCount++;
    if (frameCount >= RAF_FRAME_THRESHOLD) {
      gameReadySent = true;
      sendEvent('game_ready', {
        frame_count: frameCount,
        detected_at: Date.now()
      });
      return;
    }

    originalRAF.call(window, onFrame);
  }

  if (originalRAF) {
    window.requestAnimationFrame = function(callback) {
      if (!gameReadySent) {
        return originalRAF.call(window, function(timestamp) {
          frameCount++;
          if (frameCount >= RAF_FRAME_THRESHOLD && !gameReadySent) {
            gameReadySent = true;
            sendEvent('game_ready', {
              frame_count: frameCount,
              detected_at: Date.now()
            });
          }
          callback(timestamp);
        });
      }
      return originalRAF.call(window, callback);
    };
  }

  // --- First User Interaction Detection ---

  function setupFirstInteractionDetection() {
    var events = ['touchstart', 'mousedown', 'keydown'];

    function onFirstInteraction() {
      if (firstInteractionSent) return;
      firstInteractionSent = true;
      sendEvent('user_interaction_start', null);

      for (var i = 0; i < events.length; i++) {
        document.removeEventListener(events[i], onFirstInteraction, true);
      }
    }

    for (var i = 0; i < events.length; i++) {
      document.addEventListener(events[i], onFirstInteraction, true);
    }
  }

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', setupFirstInteractionDetection);
  } else {
    setupFirstInteractionDetection();
  }

  // --- Auto Error Capture ---

  window.addEventListener('error', function(event) {
    if (errorSent) return;
    errorSent = true;
    sendEvent('error', {
      message: event.message || 'Unknown error',
      source: event.filename || '',
      lineno: event.lineno || 0,
      colno: event.colno || 0,
      auto_captured: true
    });
  });

  window.addEventListener('unhandledrejection', function(event) {
    if (errorSent) return;
    errorSent = true;
    var reason = event.reason;
    sendEvent('error', {
      message: (reason instanceof Error) ? reason.message : String(reason),
      type: 'unhandled_promise_rejection',
      auto_captured: true
    });
  });

  // --- Public API ---

  window.playablesSDK = {
    complete: function(score) {
      sendEvent('game_ended', {
        score: score,
        completed: true
      });
    },

    error: function(message) {
      if (errorSent) return;
      errorSent = true;
      sendEvent('error', {
        message: message || 'Unknown error',
        auto_captured: false
      });
    },

    sendEvent: function(eventName, payload) {
      if (!eventName || typeof eventName !== 'string') return;
      sendEvent(eventName, payload);
    }
  };

  // Kick off rAF detection in case no game code calls rAF immediately
  if (originalRAF) {
    originalRAF.call(window, onFrame);
  }
})();</script>
<!-- PLAYABLE_TOUCH_PATCH_V1 --><script>
(function() {
  if (window.__playableTouchPatchInstalled) return;
  window.__playableTouchPatchInstalled = true;
  var origAdd = EventTarget.prototype.addEventListener;
  var blockedTypes = { touchstart: 1, touchmove: 1, wheel: 1 };
  EventTarget.prototype.addEventListener = function(type, listener, options) {
    if (blockedTypes[type]) {
      if (options === undefined || options === null) {
        options = { passive: true };
      } else if (typeof options === 'boolean') {
        options = { capture: options, passive: true };
      } else {
        options = Object.assign({}, options, { passive: true });
      }
    }
    return origAdd.call(this, type, listener, options);
  };
})();
</script><script>window.Intl=window.Intl||{};Intl.t=function(s){return(Intl._locale&&Intl._locale[s])||s;};</script>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Descaroçador de Guandu - Fazenda</title>
<style>
  * { margin:0; padding:0; box-sizing:border-box; }
  html, body { height:100%; overflow:hidden; }
  body {
    background:#3e2723;
    font-family:system-ui, -apple-system, Segoe UI, Roboto, sans-serif;
    display:flex;
    justify-content:center;
    align-items:center;
    touch-action:none;
    user-select:none;
    -webkit-user-select:none;
  }
  #gameContainer {
    position:relative;
    width:100vw;
    max-width:430px;
    height:100vh;
    max-height:930px;
    background:#000;
    border-radius:24px;
    overflow:hidden;
    box-shadow:0 20px 50px rgba(0,0,0,0.6), inset 0 0 0 4px #5d4037;
  }
  canvas {
    width:100%;
    height:100%;
    display:block;
    touch-action:none;
    cursor:grab;
  }
  canvas:active { cursor:grabbing; }
  #overlay {
    position:absolute;
    inset:0;
    background:rgba(30,20,10,0.82);
    display:none;
    place-items:center;
    z-index:20;
    backdrop-filter:blur(4px);
  }
  .panel {
    background:#fffaf0;
    padding:28px 26px;
    border-radius:22px;
    text-align:center;
    width:86%;
    max-width:320px;
    box-shadow:0 12px 30px rgba(0,0,0,0.35), inset 0 0 0 3px #8d6e63;
    color:#4e342e;
    animation:popIn .4s ease-out;
  }
  @keyframes popIn { from{transform:scale(.85);opacity:0} to{transform:scale(1);opacity:1} }
  .panel h1 { font-size:28px; margin-bottom:8px; color:#5d4037; }
  .panel p { font-size:18px; margin:6px 0; }
  .panel .score { font-size:26px; margin:14px 0; color:#2e7d32; }
  .panel button {
    margin-top:14px;
    background:linear-gradient(#66bb6a,#43a047);
    color:white;
    border:none;
    padding:14px 28px;
    font-size:18px;
    font-weight:700;
    border-radius:14px;
    box-shadow:0 4px 0 #2e7d32;
    cursor:pointer;
    width:100%;
  }
  .panel button:active { transform:translateY(2px); box-shadow:0 2px 0 #2e7d32; }
  
  .start-panel h1 { font-size:24px; line-height:1.15; margin-bottom:4px; color:#4e342e; letter-spacing:.3px; }
  .start-panel h2 { font-size:17px; margin-bottom:14px; color:#6d4c41; font-weight:600; }
  .pod-illustration { display:flex; justify-content:center; margin:8px 0 16px; }
  .start-panel .hint { font-size:14px; color:#795548; margin:8px 0 4px; line-height:1.3; }
  .start-panel .best { font-size:14px; color:#5d4037; margin-top:6px; }
  .start-panel button { font-size:20px; padding:16px 0; margin-top:6px; }
  .stats-grid { display:grid; grid-template-columns:repeat(3,1fr); gap:8px; margin:14px 0 4px; }
  .stat-box { background:#fff3e0; border:2px solid #a1887f; border-radius:12px; padding:8px 4px; box-shadow:inset 0 2px 0 rgba(255,255,255,0.6); }
  .stat-box .label { font-size:10px; color:#6d4c41; font-weight:700; text-transform:uppercase; letter-spacing:.3px; line-height:1.1; }
  .stat-box .value { font-size:22px; color:#4e342e; font-weight:800; margin-top:3px; }
</style>
</head>
<body>
<div id="gameContainer">
  <canvas id="game"></canvas>
  <div id="overlay"></div>
</div>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const container = document.getElementById('gameContainer');
const overlay = document.getElementById('overlay');

let W = 400, H = 700, dpr = 1;
let floorY = H - 30, tableY = 90;

const state = {
  level: 1,
  score: 0,
  lives: 3,
  pod: null,
  seeds: [],
  basket: { x: 0, y: 0, w: 110, h: 45 },
  playing: false,
  message: '',
  messageTimer: 0,
  roundLost: false,
  waitingNext: false,
  gameOver: false,
  guandusColhidos: 0
};

let pops = [];
let audioCtx = null;

// Resetar pontuação máxima ao carregar - inicia zerada
localStorage.setItem('guanduHighScore', '0');
localStorage.removeItem('guanduBestScore');
localStorage.removeItem('guanduBest');
localStorage.removeItem('guanduBestLevel');
localStorage.removeItem('guanduBestGuandus');

function initAudio(){
  try{
    if(!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    if(audioCtx.state === 'suspended') audioCtx.resume();
  }catch(e){}
}
function playOpen(){
  if(!audioCtx) return;
  const now = audioCtx.currentTime;
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.type = 'square';
  osc.frequency.setValueAtTime(1400, now);
  osc.frequency.exponentialRampToValueAtTime(320, now+0.05);
  gain.gain.setValueAtTime(0.22, now);
  gain.gain.exponentialRampToValueAtTime(0.001, now+0.06);
  osc.connect(gain).connect(audioCtx.destination);
  osc.start(now);
  osc.stop(now+0.07);
}
function playCatch(){
  if(!audioCtx) return;
  const now = audioCtx.currentTime;
  const o = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  o.type = 'sine';
  o.frequency.setValueAtTime(520, now);
  o.frequency.exponentialRampToValueAtTime(280, now+0.11);
  g.gain.setValueAtTime(0.26, now);
  g.gain.exponentialRampToValueAtTime(0.001, now+0.12);
  o.connect(g).connect(audioCtx.destination);
  o.start(now);
  o.stop(now+0.13);
}
function playMiss(){
  if(!audioCtx) return;
  const now = audioCtx.currentTime;
  const o = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  o.type = 'sawtooth';
  o.frequency.setValueAtTime(180, now);
  o.frequency.exponentialRampToValueAtTime(70, now+0.22);
  g.gain.setValueAtTime(0.3, now);
  g.gain.exponentialRampToValueAtTime(0.001, now+0.24);
  o.connect(g).connect(audioCtx.destination);
  o.start(now);
  o.stop(now+0.25);
}
function playBonus(){
  if(!audioCtx) return;
  const now = audioCtx.currentTime;
  [660, 880, 1100].forEach((f,i)=>{
    const o = audioCtx.createOscillator();
    const g = audioCtx.createGain();
    o.type = 'triangle';
    o.frequency.setValueAtTime(f, now + i*0.09);
    g.gain.setValueAtTime(0.2, now + i*0.09);
    g.gain.exponentialRampToValueAtTime(0.001, now + i*0.09 + 0.1);
    o.connect(g).connect(audioCtx.destination);
    o.start(now + i*0.09);
    o.stop(now + i*0.09 + 0.11);
  });
}

function resize() {
  W = Math.min(430, window.innerWidth);
  H = Math.min(window.innerHeight, 920);
  container.style.width = W + 'px';
  container.style.height = H + 'px';
  dpr = window.devicePixelRatio || 1;
  canvas.width = W * dpr;
  canvas.height = H * dpr;
  canvas.style.width = W + 'px';
  canvas.style.height = H + 'px';
  ctx.setTransform(1,0,0,1,0,0);
  ctx.scale(dpr, dpr);
  floorY = H - 24;
  tableY = 84;
  state.basket.y = H - 78;
  state.basket.x = Math.max(0, Math.min(state.basket.x, W - state.basket.w));
}
window.addEventListener('resize', resize);

function showStartScreen(){
  state.playing = false;
  state.gameOver = false;
  state.pod = null;
  state.seeds = [];
  overlay.style.display = 'grid';
  overlay.style.background = 'transparent';
  overlay.style.backdropFilter = 'none';
  overlay.innerHTML = `
    <div class="panel start-panel">
      <h1>DESCAROÇADOR DE GUANDU</h1>
      <h2>Na Fazenda</h2>
      <div class="pod-illustration">
        <svg viewBox="0 0 240 80" width="220" height="72">
          <defs><linearGradient id="g1" x1="0" x2="0" y1="0" y2="1"><stop offset="0%" stop-color="#81c784"/><stop offset="100%" stop-color="#4caf50"/></linearGradient></defs>
          <ellipse cx="120" cy="40" rx="95" ry="26" fill="url(#g1)" stroke="#2e7d32" stroke-width="4"/>
          <line x1="30" y1="40" x2="210" y2="40" stroke="#388e3c" stroke-width="2.5" stroke-linecap="round"/>
          <circle cx="72" cy="38" r="9" fill="#8bc34a" stroke="#558b2f" stroke-width="2"/>
          <circle cx="96" cy="40" r="9" fill="#7cb342" stroke="#558b2f" stroke-width="2"/>
          <circle cx="120" cy="41" r="9" fill="#689f38" stroke="#33691e" stroke-width="2"/>
          <circle cx="144" cy="40" r="9" fill="#7cb342" stroke="#558b2f" stroke-width="2"/>
          <circle cx="168" cy="38" r="9" fill="#8bc34a" stroke="#558b2f" stroke-width="2"/>
        </svg>
      </div>
      <button id="startBtn">COMEÇAR</button>
      <p class="hint">Arraste o cesto para pegar todos os caroços</p>
    </div>
  `;
  document.getElementById('startBtn').onclick = () => {
    initAudio();
    overlay.style.display = 'none';
    initGame();
  };
}

function initGame() {
  state.level = 1;
  state.score = 0;
  state.lives = 3;
  state.seeds = [];
  state.playing = true;
  state.gameOver = false;
  state.message = '';
  state.messageTimer = 0;
  state.roundLost = false;
  state.waitingNext = false;
  state.guandusColhidos = 0;
  state.basket.x = W/2 - state.basket.w/2;
  pops = [];
  spawnPod();
}

function spawnPod() {
  const count = Math.min(3 + state.level - 1, 7);
  state.pod = {
    x: W/2,
    y: tableY + 32,
    seedCount: count,
    opened: false,
    openAnim: 0
  };
  state.seeds = [];
  state.roundLost = false;
  state.waitingNext = false;
  // ABRE AUTOMATICAMENTE APÓS 0.8s
  setTimeout(()=>{
    if(state.playing && !state.gameOver && state.pod && !state.pod.opened){
      openPod();
    }
  }, 800);
}

function openPod() {
  const p = state.pod;
  if (!p || p.opened || !state.playing) return;
  p.opened = true;
  p.openAnim = 0;
  state.seeds = [];
  for (let i = 0; i < p.seedCount; i++) {
    const offset = (i - (p.seedCount-1)/2) * 16;
    state.seeds.push({
      x: p.x + offset + (Math.random()-0.5)*8,
      y: p.y + 4,
      vx: (Math.random() - 0.5) * (5 + state.level*0.2),
      vy: -2 - Math.random()*2.5 - state.level*0.1,
      r: 10,
      rot: Math.random()*Math.PI,
      hitFloor: false,
      floorTime: 0
    });
  }
  pops.push({x:p.x, y:p.y, vy:-1, life:20, text:'', color:''});
  playOpen();
}

function getCanvasX(clientX) {
  const rect = canvas.getBoundingClientRect();
  return (clientX - rect.left) * (W / rect.width);
}

function moveBasket(clientX) {
  const x = getCanvasX(clientX);
  state.basket.x = Math.max(4, Math.min(W - state.basket.w - 4, x - state.basket.w/2));
}

function handleStart(e) {
  e.preventDefault();
  initAudio();
  const point = e.touches ? e.touches[0] : e;
  moveBasket(point.clientX);
}
function handleMove(e) {
  e.preventDefault();
  const point = e.touches ? e.touches[0] : e;
  moveBasket(point.clientX);
}

canvas.addEventListener('pointerdown', handleStart);
canvas.addEventListener('pointermove', handleMove);
canvas.addEventListener('touchstart', handleStart, {passive:false});
canvas.addEventListener('touchmove', handleMove, {passive:false});
canvas.addEventListener('touchend', e=>e.preventDefault(), {passive:false});

function updateSeeds() {
  const grav = 0.4 + (state.level-1)*0.05;
  for (let i = state.seeds.length - 1; i >= 0; i--) {
    const s = state.seeds[i];
    s.vy += grav;
    s.vx *= 0.996;
    s.x += s.vx;
    s.y += s.vy;
    s.rot += s.vx * 0.04;

    if (s.x < s.r) { s.x = s.r; s.vx *= -0.7; }
    if (s.x > W - s.r) { s.x = W - s.r; s.vx *= -0.7; }

    // chão
    if (s.y + s.r > floorY) {
      s.y = floorY - s.r;
      s.vy = -s.vy * 0.3;
      s.vx *= 0.82;
      if (Math.abs(s.vy) < 0.8) s.vy = 0;
      if (!s.hitFloor) {
        s.hitFloor = true;
        s.floorTime = 0;
      }
    }
    if (s.hitFloor) s.floorTime++;

    // colisão com balde
    const b = state.basket;
    if (s.x > b.x + 6 && s.x < b.x + b.w - 6 && s.y + s.r > b.y + 8 && s.y < b.y + b.h) {
      state.seeds.splice(i, 1);
      state.score += 10;
      pops.push({x:s.x, y:b.y, vy:-2, life:32, text:'+10', color:'#2e7d32'});
      playCatch();
      checkRoundEnd();
      continue;
    }

    // perdeu no chão
    if (s.hitFloor && s.floorTime > 28) {
      state.seeds.splice(i, 1);
      state.lives--;
      state.roundLost = true;
      state.message = 'Perdeu!';
      state.messageTimer = 60;
      pops.push({x:s.x, y:s.y-8, vy:-1.5, life:38, text:'Perdeu!', color:'#c62828'});
      playMiss();
      if (state.lives <= 0) {
        gameOver();
      } else {
        checkRoundEnd();
      }
    }
  }
}

function checkRoundEnd() {
  if (!state.pod || !state.pod.opened || state.waitingNext) return;
  if (state.seeds.length === 0) {
    state.waitingNext = true;
    if (!state.roundLost) {
      state.score += 50;
      state.guandusColhidos++;
      state.level++;
      state.message = 'Boa! Próximo guandu!';
      state.messageTimer = 85;
      playBonus();
      setTimeout(()=>{ if(state.playing && !state.gameOver) spawnPod(); }, 1000);
    } else {
      state.message = 'Tente de novo!';
      state.messageTimer = 70;
      setTimeout(()=>{ if(state.playing && !state.gameOver) spawnPod(); }, 900);
    }
  }
}

function gameOver() {
  state.playing = false;
  state.gameOver = true;
  
  // Pontuação Máxima com chave 'guanduHighScore' - inicia em 0
  let highScore = parseInt(localStorage.getItem('guanduHighScore') || '0', 10);
  if (state.score > highScore) {
    highScore = state.score;
    localStorage.setItem('guanduHighScore', highScore);
  }
  
  const bestLevel = parseInt(localStorage.getItem('guanduBestLevel') || '0', 10);
  const bestGuandus = parseInt(localStorage.getItem('guanduBestGuandus') || '0', 10);
  
  if (state.level > bestLevel) {
    localStorage.setItem('guanduBestLevel', state.level);
  }
  if (state.guandusColhidos > bestGuandus) {
    localStorage.setItem('guanduBestGuandus', state.guandusColhidos);
  }
  
  overlay.style.display = 'grid';
  overlay.style.background = 'rgba(0,0,0,0.6)';
  overlay.style.backdropFilter = 'blur(4px)';
  overlay.innerHTML = `
    <div style="background:#FFFCF5; border-radius:24px; border:3px solid #A68A6D; padding:32px 28px; width:86%; max-width:320px; text-align:center; box-shadow:0 20px 50px rgba(0,0,0,0.5); animation:popIn .35s ease-out;">
      <div style="font-weight:700; color:#5D4037; font-size:28px; line-height:1.2;">Fim de Jogo!</div>
      <div style="color:#2E7D32; font-size:18px; font-weight:600; margin-top:12px;">Pontuação: ${state.score}</div>
      <div style="color:#5D4037; font-size:18px; margin-top:8px;">Nível alcançado: ${state.level}</div>
      <div style="color:#5D4037; font-size:18px; margin-top:4px;">Guandus colhidos: ${state.guandusColhidos}</div>
      <div style="color:#2E7D32; font-size:18px; margin-top:4px;">Pontuação Máxima: ${highScore}</div>
      <button id="restart" style="margin-top:20px; background:#4CAF50; color:white; border:none; width:90%; height:50px; border-radius:16px; font-weight:700; font-size:18px; box-shadow:0 4px 0 #388E3C; cursor:pointer; transition:transform 0.05s;">Jogar de novo</button>
    </div>
  `;
  document.getElementById('restart').onclick = () => {
    overlay.style.display = 'none';
    initGame();
  };
  const btn = document.getElementById('restart');
  btn.onpointerdown = ()=>{ btn.style.transform='translateY(2px)'; btn.style.boxShadow='0 2px 0 #388E3C'; };
  btn.onpointerup = ()=>{ btn.style.transform=''; btn.style.boxShadow='0 4px 0 #388E3C'; };
}

function drawBackground() {
  // céu
  const g = ctx.createLinearGradient(0,0,0,H);
  g.addColorStop(0, '#87CEEB');
  g.addColorStop(0.6, '#FFD7A5');
  g.addColorStop(1, '#ffbd7a');
  ctx.fillStyle = g;
  ctx.fillRect(0,0,W,H);

  // morros
  ctx.fillStyle = '#4caf50';
  ctx.beginPath();
  ctx.arc(-20, H*0.72, W*0.65, 0, Math.PI*2);
  ctx.fill();
  ctx.fillStyle = '#388e3c';
  ctx.beginPath();
  ctx.arc(W*0.5, H*0.78, W*0.7, 0, Math.PI*2);
  ctx.fill();
  ctx.fillStyle = '#2e7d32';
  ctx.beginPath();
  ctx.arc(W+30, H*0.74, W*0.6, 0, Math.PI*2);
  ctx.fill();
}

function drawTable() {
  ctx.fillStyle = '#8d6e63';
  ctx.fillRect(0, tableY-12, W, 28);
  ctx.fillStyle = '#6d4c41';
  ctx.fillRect(0, tableY-12, W, 6);
  for(let i=0;i<4;i++){
    ctx.strokeStyle = '#5d4037';
    ctx.lineWidth = 1;
    ctx.beginPath();
    ctx.moveTo(0, tableY -4 + i*5);
    ctx.lineTo(W, tableY -4 + i*5);
    ctx.stroke();
  }
  ctx.fillStyle = 'rgba(0,0,0,0.12)';
  ctx.fillRect(0, tableY+16, W, 12);
}

function drawPod() {
  const p = state.pod;
  if (!p) return;
  ctx.save();
  ctx.translate(p.x, p.y);
  
  if (!p.opened) {
    ctx.fillStyle = '#4caf50';
    ctx.strokeStyle = '#2e7d32';
    ctx.lineWidth = 3.5;
    ctx.beginPath();
    ctx.ellipse(0,0,72,20,0,0,Math.PI*2);
    ctx.fill();
    ctx.strokeStyle = '#388e3c';
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(-66,0); ctx.lineTo(66,0); ctx.stroke();
    for(let i=0;i<p.seedCount;i++){
      const ox = (i-(p.seedCount-1)/2)*18;
      ctx.fillStyle = '#7cb342';
      ctx.beginPath(); ctx.arc(ox,-1,7,0,Math.PI*2); ctx.fill();
      ctx.strokeStyle='#558b2f'; ctx.lineWidth=1.5; ctx.stroke();
    }
  } else {
    p.openAnim = Math.min(1, p.openAnim + 0.09);
    const off = p.openAnim * 38;
    ctx.save();
    ctx.translate(-off, -off*0.15);
    ctx.rotate(-0.18);
    ctx.fillStyle = '#66bb6a';
    ctx.strokeStyle = '#2e7d32';
    ctx.lineWidth = 3;
    ctx.beginPath();
    ctx.ellipse(-18,0,42,16,0,0,Math.PI*2);
    ctx.fill(); ctx.stroke();
    ctx.restore();
    ctx.save();
    ctx.translate(off, off*0.15);
    ctx.rotate(0.18);
    ctx.fillStyle = '#66bb6a';
    ctx.strokeStyle = '#2e7d32';
    ctx.lineWidth = 3;
    ctx.beginPath();
    ctx.ellipse(18,0,42,16,0,0,Math.PI*2);
    ctx.fill(); ctx.stroke();
    ctx.restore();
  }
  ctx.restore();
}

function drawSeeds() {
  state.seeds.forEach(s=>{
    ctx.save();
    ctx.translate(s.x, s.y);
    ctx.rotate(s.rot);
    ctx.fillStyle = 'rgba(0,0,0,0.15)';
    ctx.beginPath(); ctx.ellipse(2, s.r+3, s.r*0.9, 3, 0,0,Math.PI*2); ctx.fill();
    ctx.fillStyle = '#6B8E23';
    ctx.strokeStyle = '#334d14';
    ctx.lineWidth = 2.5;
    ctx.beginPath(); ctx.arc(0,0,s.r,0,Math.PI*2); ctx.fill(); ctx.stroke();
    ctx.fillStyle = 'rgba(255,255,255,0.28)';
    ctx.beginPath(); ctx.arc(-3,-3, s.r*0.38,0,Math.PI*2); ctx.fill();
    ctx.restore();
  });
}

function drawBasket() {
  const b = state.basket;
  ctx.save();
  ctx.translate(b.x, b.y);
  
  // Centralizar visual 120x50 sobre hitbox 110x45 para manter jogabilidade
  const Vw = 120, Vh = 50;
  const offsetX = (b.w - Vw) / 2;
  const offsetY = (b.h - Vh) / 2;
  ctx.translate(offsetX, offsetY);
  
  const cx = Vw/2;
  
  // 1. Sombra projetada #A0522D
  ctx.fillStyle = '#A0522D';
  ctx.globalAlpha = 0.26;
  ctx.beginPath();
  ctx.ellipse(cx, Vh + 8, Vw * 0.48, 9, 0, 0, Math.PI*2);
  ctx.fill();
  ctx.globalAlpha = 1;

  // 2. Alças simétricas em arco perfeito #654321
  ctx.strokeStyle = '#654321';
  ctx.lineWidth = 4.5;
  ctx.lineCap = 'round';
  ctx.beginPath();
  ctx.arc(18, 17, 12, Math.PI, 0, true);
  ctx.stroke();
  ctx.beginPath();
  ctx.arc(Vw - 18, 17, 12, Math.PI, 0, true);
  ctx.stroke();

  // 3. Corpo principal - U largo com Bézier
  ctx.beginPath();
  ctx.moveTo(8, 15);
  ctx.bezierCurveTo(1, 22, 0, 34, 11, 46);
  ctx.bezierCurveTo(26, 55, 94, 55, 109, 46);
  ctx.bezierCurveTo(120, 34, 119, 22, 112, 15);
  ctx.quadraticCurveTo(cx, 5, 8, 15);
  ctx.closePath();
  
  // Preenchimento gradiente #DEB887 -> #CD853F
  const grad = ctx.createLinearGradient(0, 12, 0, Vh + 2);
  grad.addColorStop(0, '#DEB887');
  grad.addColorStop(0.5, '#D2A679');
  grad.addColorStop(1, '#CD853F');
  ctx.fillStyle = grad;
  ctx.fill();

  // 4. Interior oco #8B5A2B
  ctx.fillStyle = '#8B5A2B';
  ctx.beginPath();
  ctx.ellipse(cx, 17.5, 47, 9.5, 0, 0, Math.PI*2);
  ctx.fill();
  ctx.fillStyle = 'rgba(0,0,0,0.22)';
  ctx.beginPath();
  ctx.ellipse(cx, 19, 39, 7, 0, 0, Math.PI*2);
  ctx.fill();

  // 5. Trama de vime - 7 linhas horizontais curvas
  ctx.strokeStyle = '#A06A3A';
  ctx.lineWidth = 1.7;
  ctx.globalAlpha = 0.88;
  const horizontais = [20, 25, 30, 35, 40, 44, 48];
  horizontais.forEach(y => {
    ctx.beginPath();
    ctx.moveTo(10, y);
    ctx.bezierCurveTo(cx-32, y+1.6, cx+32, y+1.6, Vw-10, y);
    ctx.stroke();
  });
  
  // 6. Trama de vime - 12 linhas verticais levemente curvas
  ctx.strokeStyle = '#8B5A2B';
  ctx.lineWidth = 1.4;
  for (let i = 0; i < 12; i++) {
    const t = i / 11;
    const xTop = 13 + t * 94;
    const xBot = 11 + t * 98;
    const curve = (t - 0.5) * 4;
    ctx.beginPath();
    ctx.moveTo(xTop, 16);
    ctx.bezierCurveTo(xTop + curve*0.3, 29, xBot + curve*0.5, 39, xBot, 49);
    ctx.stroke();
  }
  ctx.globalAlpha = 1;

  // 7. Borda superior grossa 5px #8B4513
  ctx.strokeStyle = '#8B4513';
  ctx.lineWidth = 5;
  ctx.lineCap = 'round';
  ctx.lineJoin = 'round';
  ctx.beginPath();
  ctx.moveTo(8, 15);
  ctx.quadraticCurveTo(cx, 4.5, Vw - 8, 15);
  ctx.stroke();
  
  // Contorno para definição
  ctx.strokeStyle = '#5D3A1A';
  ctx.globalAlpha = 0.45;
  ctx.lineWidth = 1.8;
  ctx.beginPath();
  ctx.moveTo(8, 15);
  ctx.bezierCurveTo(1, 22, 0, 34, 11, 46);
  ctx.bezierCurveTo(26, 55, 94, 55, 109, 46);
  ctx.bezierCurveTo(120, 34, 119, 22, 112, 15);
  ctx.stroke();
  ctx.globalAlpha = 1;

  ctx.restore();
}

function drawUI() {
  ctx.fillStyle = 'rgba(93,64,55,0.92)';
  ctx.beginPath();
  if (ctx.roundRect) {
    ctx.roundRect(10,10,W-20,54,14);
  } else {
    ctx.fillRect(10,10,W-20,54);
  }
  ctx.fill();

  ctx.fillStyle = '#ffecb3';
  ctx.font = '700 16px system-ui';
  ctx.textBaseline = 'middle';
  ctx.textAlign = 'left';
  ctx.fillText(`Nível ${state.level}`, 24, 37);
  ctx.textAlign = 'center';
  ctx.fillText(`Pontos ${state.score}`, W/2, 37);

  ctx.textAlign = 'right';
  for(let i=0;i<3;i++){
    const x = W - 22 - i*26;
    const y = 37;
    ctx.save();
    ctx.translate(x, y);
    ctx.fillStyle = i < state.lives ? '#6B8E23' : '#5d403755';
    ctx.strokeStyle = i < state.lives ? '#334d14' : '#3e272355';
    ctx.lineWidth = 2;
    ctx.beginPath(); ctx.arc(0,0,9,0,Math.PI*2); ctx.fill(); ctx.stroke();
    ctx.fillStyle = i < state.lives ? 'rgba(255,255,255,0.25)' : 'transparent';
    ctx.beginPath(); ctx.arc(-3,-3,3,0,Math.PI*2); ctx.fill();
    ctx.restore();
  }
}

function drawPops() {
  pops.forEach(p=>{
    p.y += p.vy;
    p.vy += 0.12;
    p.life--;
    if (p.text) {
      ctx.save();
      ctx.globalAlpha = Math.max(0, p.life/32);
      ctx.font = 'bold 22px system-ui';
      ctx.textAlign = 'center';
      ctx.fillStyle = p.color;
      ctx.strokeStyle = 'white';
      ctx.lineWidth = 4;
      ctx.strokeText(p.text, p.x, p.y);
      ctx.fillText(p.text, p.x, p.y);
      ctx.restore();
    }
  });
  pops = pops.filter(p=>p.life>0);
}

function drawMessage() {
  if (state.messageTimer > 0) {
    ctx.save();
    ctx.textAlign = 'center';
    ctx.font = '800 30px system-ui';
    const y = H*0.42;
    ctx.fillStyle = 'rgba(0,0,0,0.35)';
    ctx.fillText(state.message, W/2+2, y+3);
    ctx.fillStyle = '#fff8e1';
    ctx.strokeStyle = '#5d4037';
    ctx.lineWidth = 5;
    ctx.strokeText(state.message, W/2, y);
    ctx.fillText(state.message, W/2, y);
    ctx.restore();
    state.messageTimer--;
  }
}

function draw() {
  drawBackground();
  if (!state.playing) return;
  
  drawTable();
  drawPod();
  drawSeeds();
  drawBasket();
  drawPops();
  drawUI();
  drawMessage();

  // chão
  ctx.fillStyle = '#6d4c41';
  ctx.fillRect(0, floorY, W, H-floorY);
  ctx.fillStyle = '#5d4037';
  ctx.fillRect(0, floorY, W, 4);
}

function update() {
  if (!state.playing) return;
  updateSeeds();
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

resize();
showStartScreen();
requestAnimationFrame(loop);
</script>
<script>(function(){document.addEventListener("click",function(e){var a=e.target.closest("[data-product-id]");if(!a)return;e.preventDefault();var pid=a.getAttribute("data-product-id");if(pid)parent.postMessage({type:"ecto-artifact-link-click",productId:pid},"*")})})();</script>
</body>
</html>
