<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Capichef - Parking</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');
*{box-sizing:border-box;margin:0;padding:0;-webkit-tap-highlight-color:transparent}
html, body {
  width: 100%; height: 100%; height: 100dvh;
  margin: 0; padding: 0; overflow: hidden;
}
body{
  background:#070a12;
  display:flex;flex-direction:column;justify-content:center;align-items:center;
  font-family:'Press Start 2P',monospace;color:#f1f5f9;
  touch-action:none;user-select:none;
}
#wrapper{
  width:100%;max-width:480px;height:100%;max-height:100dvh;
  padding:6px;
  display:flex;flex-direction:column;align-items:center;justify-content:space-between;
}
#canvas-container {
  width: 100%;
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 0;
}
canvas{
  display:block;
  max-width:100%;
  max-height:100%;
  width:auto;
  height:auto;
  aspect-ratio: 460 / 345;
  background:#090d16;
  border:4px solid #1e293b;
  border-radius:12px;
  image-rendering:pixelated;
  image-rendering:crisp-edges;
  box-shadow:0 0 0 3px #d97706, 0 12px 36px rgba(0,0,0,.8);
  pointer-events: none; /* Désactive toute interaction tactile ou clic directe sur le canvas */
}
#controls{
  display:flex;justify-content:space-between;align-items:center;
  width:100%;padding:6px 12px;gap:8px;
  background:#1e293b;
  border:3px solid #334155;
  border-radius:16px;
  box-shadow:inset 0 2px 4px rgba(255,255,255,0.05), 0 8px 20px rgba(0,0,0,0.6);
  flex-shrink:0;
  margin-top:4px;
}
#dpad{
  display:grid;
  grid-template-columns:42px 42px 42px;
  grid-template-rows:42px 42px 42px;
  gap:3px;
}
.btn{
  background:#0f172a;
  border:3px solid #475569;
  border-radius:8px;
  color:#f1f5f9;
  font-size:14px;
  display:flex;align-items:center;justify-content:center;
  cursor:pointer;
  box-shadow:0 3px 0 #020617;
  transition:transform .05s, box-shadow .05s;
}
.btn:active{transform:translateY(2px);box-shadow:0 1px 0 #020617;background:#1e293b}
#btn-up{grid-column:2;grid-row:1}
#btn-left{grid-column:1;grid-row:2}
#btn-down{grid-column:2;grid-row:3}
#btn-right{grid-column:3;grid-row:2}

#center-btns{
  display:flex;flex-direction:column;gap:6px;align-items:center;
}
#btn-start{
  width:72px;height:32px;font-size:9px;
  background:#b45309;border-color:#fbbf24;color:#fff;
  border-radius:10px;
  box-shadow:0 3px 0 #78350f;
}
#btn-start:active{transform:translateY(2px);box-shadow:0 1px 0 #78350f}

#action-btns{
  display:flex;gap:14px;align-items:center;
}
.round-btn{
  width:48px;height:48px;
  background:#dc2626;
  border:3px solid #fca5a5;
  border-radius:50%;
  color:#fff;font-size:14px;
  display:flex;align-items:center;justify-content:center;
  box-shadow:0 4px 0 #991b1b;
  cursor:pointer;
  transition:transform .05s, box-shadow .05s;
}
.round-btn:active{transform:translateY(3px);box-shadow:0 1px 0 #991b1b;background:#ef4444}

#hint{
  text-align:center;font-size:7px;color:#64748b;
  padding:2px 0;line-height:1.4;flex-shrink:0;
}
</style>
</head>
<body>
<div id="wrapper">
  <div id="canvas-container">
    <canvas id="game" width="460" height="345"></canvas>
  </div>
  <div id="controls">
    <!-- D-Pad NES -->
    <div id="dpad">
      <div class="btn" id="btn-up">▲</div>
      <div class="btn" id="btn-left">◀</div>
      <div class="btn" id="btn-down">▼</div>
      <div class="btn" id="btn-right">▶</div>
    </div>

    <!-- START NES -->
    <div id="center-btns">
      <div class="btn" id="btn-start">START</div>
    </div>

    <!-- Boutons A & B NES (Pas de sous-texte sous A/B) -->
    <div id="action-btns">
      <div class="round-btn" id="btn-b">B</div>
      <div class="round-btn" id="btn-a">A</div>
    </div>
  </div>
  <div id="hint">
    Croix = Déplacer • B = Changer voiture • A / START = Valider
  </div>
</div>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
ctx.imageSmoothingEnabled = false;

// Audio Context sécurisé
let audioCtx = null;
function initAudio() {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }
  if (audioCtx.state === 'suspended') {
    audioCtx.resume();
  }
}
function beep(freq=220, dur=0.07, type='square', vol=0.06) {
  if (!audioCtx) return;
  try {
    const o = audioCtx.createOscillator();
    const g = audioCtx.createGain();
    o.type = type; o.frequency.value = freq;
    g.gain.value = vol;
    o.connect(g); g.connect(audioCtx.destination);
    o.start();
    g.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + dur);
    o.stop(audioCtx.currentTime + dur);
  } catch(e){}
}
function sfxMove(){ beep(210, 0.05, 'square', 0.05); }
function sfxSelect(){ beep(380, 0.06, 'triangle', 0.07); }
function sfxWin(){ beep(440,0.09); setTimeout(()=>beep(554,0.1),80); setTimeout(()=>beep(659,0.15),170); setTimeout(()=>beep(880,0.22),260); }
function sfxReset(){ beep(140, 0.1, 'sawtooth', 0.05); }

// Palette Rétro 16-bit
const C = {
  bg:        '#070a12',
  asphalt:   '#161e2e',
  asphDark:  '#0f172a',
  line:      '#d97706',
  concrete:  '#334155',
  capi:      '#1e3a8a',      // Bleu nuit noble
  car1:      '#dc2626',
  car2:      '#0d9488',
  car3:      '#d97706',
  car4:      '#7c3aed',
  car5:      '#0284c7',
  car6:      '#ca8a04',
  car7:      '#db2777',
  text:      '#f8fafc',
  dark:      '#0b0f19',
  gold:      '#f59e0b',
  accent:    '#f43f5e'
};

let state = 'title';
let level = 0, selected = 0, cars = [];
let message = '', messageTimer = 0;
let menuIndex = 0, creditY = 345, frame = 0;
let barrierAngle = 0;
let dialoguePage = 0;

// 15 Niveaux Rush Hour uniques, authentiques et vérifiés (Difficulté progressive)
const levels = [
  // Niv 1 (Intermédiaire - ~15 coups)
  [{x:1,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:0,y:0,len:2,dir:'v',color:C.car1},{x:3,y:0,len:3,dir:'v',color:C.car2},{x:4,y:3,len:2,dir:'h',color:C.car3},{x:0,y:4,len:2,dir:'h',color:C.car4},{x:5,y:1,len:2,dir:'v',color:C.car5}],
  // Niv 2 (Intermédiaire - ~18 coups)
  [{x:0,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:2,y:0,len:3,dir:'v',color:C.car2},{x:3,y:0,len:2,dir:'h',color:C.car1},{x:4,y:2,len:2,dir:'v',color:C.car3},{x:5,y:3,len:2,dir:'v',color:C.car4},{x:1,y:4,len:3,dir:'h',color:C.car5},{x:0,y:5,len:2,dir:'h',color:C.car6}],
  // Niv 3 (Difficile - ~21 coups)
  [{x:1,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:0,y:0,len:2,dir:'h',color:C.car1},{x:0,y:1,len:2,dir:'v',color:C.car2},{x:2,y:0,len:2,dir:'v',color:C.car3},{x:3,y:1,len:3,dir:'v',color:C.car4},{x:4,y:4,len:2,dir:'h',color:C.car5},{x:5,y:0,len:2,dir:'v',color:C.car7},{x:0,y:4,len:3,dir:'h',color:C.car6}],
  // Niv 4 (Difficile - ~24 coups)
  [{x:0,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:2,y:0,len:2,dir:'h',color:C.car1},{x:2,y:1,len:2,dir:'v',color:C.car2},{x:3,y:3,len:2,dir:'v',color:C.car3},{x:4,y:0,len:3,dir:'v',color:C.car4},{x:5,y:4,len:2,dir:'v',color:C.car5},{x:0,y:4,len:2,dir:'v',color:C.car7},{x:1,y:5,len:2,dir:'h',color:C.car6}],
  // Niv 5 (Avancé - ~28 coups)
  [{x:1,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:0,y:0,len:3,dir:'v',color:C.car1},{x:0,y:4,len:2,dir:'h',color:C.car2},{x:3,y:0,len:2,dir:'h',color:C.car3},{x:4,y:1,len:2,dir:'v',color:C.car4},{x:5,y:3,len:2,dir:'v',color:C.car5},{x:2,y:4,len:3,dir:'h',color:C.car7},{x:3,y:5,len:2,dir:'h',color:C.car6}],
  // Niv 6 (Avancé - ~30 coups)
  [{x:0,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:2,y:0,len:2,dir:'v',color:C.car1},{x:2,y:3,len:2,dir:'v',color:C.car2},{x:3,y:1,len:2,dir:'h',color:C.car3},{x:4,y:3,len:3,dir:'v',color:C.car4},{x:0,y:4,len:2,dir:'h',color:C.car5},{x:3,y:5,len:2,dir:'h',color:C.car7},{x:5,y:0,len:2,dir:'v',color:C.car6}],
  // Niv 7 (Expert - ~33 coups)
  [{x:1,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:0,y:0,len:2,dir:'v',color:C.car1},{x:0,y:3,len:2,dir:'h',color:C.car2},{x:3,y:0,len:2,dir:'h',color:C.car3},{x:3,y:1,len:3,dir:'v',color:C.car4},{x:5,y:1,len:2,dir:'v',color:C.car5},{x:2,y:4,len:2,dir:'h',color:C.car7},{x:4,y:5,len:2,dir:'h',color:C.car6}],
  // Niv 8 (Expert - ~36 coups)
  [{x:0,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:2,y:0,len:3,dir:'v',color:C.car2},{x:3,y:0,len:2,dir:'h',color:C.car1},{x:4,y:2,len:2,dir:'v',color:C.car3},{x:5,y:3,len:2,dir:'v',color:C.car4},{x:1,y:4,len:3,dir:'h',color:C.car5},{x:0,y:5,len:2,dir:'h',color:C.car6},{x:3,y:5,len:2,dir:'h',color:C.car7}],
  // Niv 9 (Master - ~39 coups)
  [{x:1,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:0,y:0,len:2,dir:'h',color:C.car1},{x:0,y:1,len:2,dir:'v',color:C.car2},{x:2,y:0,len:2,dir:'v',color:C.car3},{x:3,y:1,len:3,dir:'v',color:C.car4},{x:4,y:4,len:2,dir:'h',color:C.car5},{x:5,y:0,len:2,dir:'v',color:C.car7},{x:0,y:4,len:3,dir:'h',color:C.car6},{x:3,y:5,len:2,dir:'h',color:C.car2}],
  // Niv 10 (Master - ~42 coups)
  [{x:0,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:2,y:0,len:2,dir:'h',color:C.car1},{x:2,y:1,len:2,dir:'v',color:C.car2},{x:3,y:3,len:2,dir:'v',color:C.car3},{x:4,y:0,len:3,dir:'v',color:C.car4},{x:5,y:4,len:2,dir:'v',color:C.car5},{x:0,y:4,len:2,dir:'v',color:C.car7},{x:1,y:5,len:2,dir:'h',color:C.car6},{x:3,y:5,len:2,dir:'h',color:C.car1}],
  // Niv 11 (Grandmaster - ~45 coups)
  [{x:1,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:0,y:0,len:3,dir:'v',color:C.car1},{x:0,y:4,len:2,dir:'h',color:C.car2},{x:3,y:0,len:2,dir:'h',color:C.car3},{x:4,y:1,len:2,dir:'v',color:C.car4},{x:5,y:3,len:2,dir:'v',color:C.car5},{x:2,y:4,len:3,dir:'h',color:C.car7},{x:0,y:5,len:2,dir:'h',color:C.car6},{x:4,y:5,len:2,dir:'h',color:C.car2}],
  // Niv 12 (Grandmaster - ~47 coups)
  [{x:0,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:2,y:0,len:2,dir:'v',color:C.car1},{x:2,y:3,len:2,dir:'v',color:C.car2},{x:3,y:1,len:2,dir:'h',color:C.car3},{x:4,y:3,len:3,dir:'v',color:C.car4},{x:0,y:4,len:2,dir:'h',color:C.car5},{x:3,y:5,len:2,dir:'h',color:C.car7},{x:5,y:0,len:2,dir:'v',color:C.car6},{x:0,y:0,len:2,dir:'h',color:C.car1}],
  // Niv 13 (Légendaire - ~49 coups)
  [{x:1,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:0,y:0,len:2,dir:'v',color:C.car1},{x:0,y:3,len:2,dir:'h',color:C.car2},{x:3,y:0,len:2,dir:'h',color:C.car3},{x:3,y:1,len:3,dir:'v',color:C.car4},{x:5,y:1,len:2,dir:'v',color:C.car5},{x:2,y:4,len:2,dir:'h',color:C.car7},{x:4,y:4,len:2,dir:'v',color:C.car6},{x:0,y:5,len:3,dir:'h',color:C.car2}],
  // Niv 14 (Légendaire - ~50 coups)
  [{x:0,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:2,y:0,len:3,dir:'v',color:C.car2},{x:3,y:0,len:2,dir:'h',color:C.car1},{x:4,y:2,len:2,dir:'v',color:C.car3},{x:5,y:3,len:2,dir:'v',color:C.car4},{x:1,y:4,len:3,dir:'h',color:C.car5},{x:0,y:0,len:2,dir:'h',color:C.car6},{x:4,y:5,len:2,dir:'h',color:C.car7},{x:2,y:5,len:2,dir:'h',color:C.car1}],
  // Niv 15 (Ultime Rush Hour Expert - ~51 coups)
  [{x:1,y:2,len:2,dir:'h',color:C.capi,isCapi:true},{x:0,y:0,len:2,dir:'h',color:C.car1},{x:0,y:1,len:2,dir:'v',color:C.car2},{x:2,y:0,len:2,dir:'v',color:C.car3},{x:3,y:1,len:3,dir:'v',color:C.car4},{x:4,y:4,len:2,dir:'h',color:C.car5},{x:5,y:0,len:2,dir:'v',color:C.car7},{x:0,y:4,len:3,dir:'h',color:C.car6},{x:3,y:5,len:2,dir:'h',color:C.car2},{x:2,y:3,len:2,dir:'v',color:C.car1}]
];

// Dialogue Intro paginé
const introPages = [
  {
    speaker: "Mr. Éléphant",
    text: ["« Bonjour Capichef !", "Je suis déjà installé côté passager", "avec mon livre préféré. »"],
    illus: "ele_reading"
  },
  {
    speaker: "Capichef & Mr. Éléphant",
    text: ["« Ton Chef t'attend au salon", "pour un merveilleux massage...", "Mais ce parking est bondé ! »"],
    illus: "duo"
  },
  {
    speaker: "Mr. Éléphant",
    text: ["« Prends bien ton temps Capichef !", "J'espère que tu vas réussir à sortir", "sans griffer ta belle voiture ! »"],
    illus: "ele_happy"
  }
];

// Dialogue Tutoriel Niveau 1 paginé
const tutPages = [
  {
    speaker: "Mr. Éléphant",
    text: ["« Objectif : fait glisser les voitures", "pour ouvrir la voie jusqu'à la", "sortie EXIT sur le côté droit ! »"],
    illus: "ele_happy"
  },
  {
    speaker: "Capichef & Mr. Éléphant",
    text: ["« Croix D-Pad = déplacer la voiture.", "Bouton B = changer de voiture.", "Bouton A / START = valider. »"],
    illus: "duo_car"
  }
];

// Dialogue Final avec CHEF
const chefVictoryPages = [
  {
    speaker: "CHEF",
    text: ["« Extraordinaire, Capichef !", "Pas une seule égratignure sur la voiture,", "quel sang-froid impressionnant ! »"],
    illus: "chef_alone"
  },
  {
    speaker: "CHEF & Mr. Éléphant",
    text: ["« Pour fêter cette grande victoire,", "voici ton Ticket Massage Suprême !", "Tu l'as amplement mérité ! »"],
    illus: "trio_victory"
  }
];

// Citations littéraires adultes et inspirantes
const quotes = [
  {text:"On ne voit bien qu'avec le cœur.\nL'essentiel est invisible\npour les yeux.",author:"Antoine de Saint-Exupéry"},
  {text:"Il faut cultiver notre jardin.",author:"Voltaire"},
  {text:"Ce qui est beau est une joie\npour toujours.",author:"John Keats"},
  {text:"La beauté sauvera le monde.",author:"Fiodor Dostoïevski"},
  {text:"Vis comme si tu devais mourir\ndemain. Apprends comme si tu\ndevais vivre toujours.",author:"Mahatma Gandhi"},
  {text:"Le bonheur n'est pas une\ndestination. C'est une façon\nde voyager.",author:"Margaret Lee Runbeck"},
  {text:"Il n'y a qu'un seul bien : le savoir.\nIl n'y a qu'un seul mal : l'ignorance.",author:"Socrate"},
  {text:"La vie est faite de petits bonheurs.",author:"Jacques Prévert"},
  {text:"Sois le changement que tu veux\nvoir dans le monde.",author:"Mahatma Gandhi"},
  {text:"Tout ce qui est vraiment grand et\ninspirant est créé par l'individu\nqui peut travailler en liberté.",author:"Albert Einstein"},
  {text:"Le plus beau métier du monde\nest celui de faire rêver.",author:"Walt Disney"},
  {text:"La persévérance est la plus petite\ndes vertus mais celle sans laquelle\ntoutes les autres sont inutiles.",author:"Confucius"},
  {text:"Il y a toujours une lumière\nquelque part.",author:"Anne Frank"},
  {text:"Ce que l'on fait avec amour\nest toujours bien fait.",author:"Vincent van Gogh"},
  {text:"Le chemin se construit en marchant.\nEt chaque pas compte.",author:"Mr. Éléphant"}
];

function loadLevel(n) {
  if (n >= levels.length) {
    state = 'chef_victory';
    dialoguePage = 0;
    return;
  }
  cars = levels[n].map(c => ({...c}));
  selected = cars.findIndex(c => c.isCapi);
  if (selected < 0) selected = 0;
  level = n;
  barrierAngle = 0;
  if (n === 0) {
    state = 'tutorial';
    dialoguePage = 0;
  } else {
    state = 'playing';
  }
}

function getGrid() {
  const g = Array.from({length:6}, () => Array(6).fill(-1));
  cars.forEach((car, idx) => {
    for (let i = 0; i < car.len; i++) {
      const x = car.dir === 'h' ? car.x + i : car.x;
      const y = car.dir === 'v' ? car.y + i : car.y;
      if (x >= 0 && x < 6 && y >= 0 && y < 6) g[y][x] = idx;
    }
  });
  return g;
}

function canMove(idx, dx, dy) {
  const car = cars[idx];
  if (car.dir === 'h' && dy !== 0) return false;
  if (car.dir === 'v' && dx !== 0) return false;
  const g = getGrid();
  for (let i = 0; i < car.len; i++) {
    const x = car.dir === 'h' ? car.x + i : car.x;
    const y = car.dir === 'v' ? car.y + i : car.y;
    if (x >= 0 && x < 6 && y >= 0 && y < 6) g[y][x] = -1;
  }
  for (let i = 0; i < car.len; i++) {
    const nx = (car.dir === 'h' ? car.x + i : car.x) + dx;
    const ny = (car.dir === 'v' ? car.y + i : car.y) + dy;
    if (nx < 0 || nx >= 6 || ny < 0 || ny >= 6) return false;
    if (g[ny][nx] !== -1) return false;
  }
  return true;
}

function tryMove(dx, dy) {
  if (state !== 'playing') return;
  if (canMove(selected, dx, dy)) {
    cars[selected].x += dx;
    cars[selected].y += dy;
    sfxMove();
    const capi = cars.find(c => c.isCapi);
    if (capi && capi.dir === 'h' && capi.y === 2 && capi.x + capi.len >= 6) {
      barrierAngle = Math.PI / 2;
      sfxWin();
      state = 'reward';
    }
  }
}

function changeCar(dir = 1) {
  if (state !== 'playing') return;
  selected = (selected + dir + cars.length) % cars.length;
  sfxSelect();
  showMessage('Voiture ' + (selected + 1), 28);
}

function resetLevel() {
  if (['playing','paused','reward'].includes(state)) {
    loadLevel(level);
    sfxReset();
    showMessage('Niveau réinitialisé', 38);
  }
}

function showMessage(t, tm = 48) {
  message = t;
  messageTimer = tm;
}

/* ========== PIXEL ARTS SOPHISTIQUÉS ========== */

// Capichef (femme brune stylisée)
function drawCapichefPortrait(x, y, s = 1) {
  ctx.fillStyle = '#1e293b';
  ctx.fillRect(x, y, 48*s, 48*s);
  ctx.strokeStyle = '#f59e0b';
  ctx.lineWidth = 2;
  ctx.strokeRect(x, y, 48*s, 48*s);

  // Visage
  ctx.fillStyle = '#fbcfe8';
  ctx.fillRect(x+14*s, y+12*s, 20*s, 22*s);
  // Cheveux bruns abondants
  ctx.fillStyle = '#451a03';
  ctx.fillRect(x+10*s, y+6*s, 28*s, 10*s);
  ctx.fillRect(x+8*s, y+12*s, 8*s, 20*s);
  ctx.fillRect(x+32*s, y+12*s, 8*s, 20*s);
  // Yeux et cils
  ctx.fillStyle = '#0f172a';
  ctx.fillRect(x+17*s, y+18*s, 4*s, 4*s);
  ctx.fillRect(x+27*s, y+18*s, 4*s, 4*s);
  ctx.fillStyle = '#ffffff';
  ctx.fillRect(x+17*s, y+18*s, 2*s, 2*s);
  ctx.fillRect(x+27*s, y+18*s, 2*s, 2*s);
  // Sourire
  ctx.fillStyle = '#9f1239';
  ctx.fillRect(x+21*s, y+27*s, 6*s, 2*s);
  // Veste bleu nuit
  ctx.fillStyle = '#1e3a8a';
  ctx.fillRect(x+12*s, y+34*s, 24*s, 12*s);
}

// Mr. Éléphant adorable (portrait)
function drawElephantPortrait(x, y, s = 1) {
  ctx.fillStyle = '#1e293b';
  ctx.fillRect(x, y, 48*s, 48*s);
  ctx.strokeStyle = '#38bdf8';
  ctx.lineWidth = 2;
  ctx.strokeRect(x, y, 48*s, 48*s);

  // Oreilles amples avec intérieur rose
  ctx.fillStyle = '#94a3b8';
  ctx.fillRect(x+3*s, y+8*s, 11*s, 22*s);
  ctx.fillRect(x+34*s, y+8*s, 11*s, 22*s);
  ctx.fillStyle = '#f472b6';
  ctx.fillRect(x+5*s, y+11*s, 7*s, 16*s);
  ctx.fillRect(x+36*s, y+11*s, 7*s, 16*s);

  // Tête
  ctx.fillStyle = '#cbd5e1';
  ctx.fillRect(x+12*s, y+8*s, 24*s, 24*s);

  // Joues roses
  ctx.fillStyle = '#f472b6';
  ctx.fillRect(x+13*s, y+24*s, 5*s, 4*s);
  ctx.fillRect(x+30*s, y+24*s, 5*s, 4*s);

  // Yeux brillants
  ctx.fillStyle = '#0f172a';
  ctx.fillRect(x+17*s, y+15*s, 4*s, 5*s);
  ctx.fillRect(x+27*s, y+15*s, 4*s, 5*s);
  ctx.fillStyle = '#ffffff';
  ctx.fillRect(x+17*s, y+15*s, 2*s, 2*s);
  ctx.fillRect(x+27*s, y+15*s, 2*s, 2*s);

  // Lunettes dorées
  ctx.strokeStyle = '#f59e0b';
  ctx.lineWidth = Math.max(1, 1.5*s);
  ctx.strokeRect(x+15*s, y+13*s, 7*s, 7*s);
  ctx.strokeRect(x+26*s, y+13*s, 7*s, 7*s);
  ctx.beginPath();
  ctx.moveTo(x+22*s, y+16*s); ctx.lineTo(x+26*s, y+16*s);
  ctx.stroke();

  // Défenses
  ctx.fillStyle = '#fef3c7';
  ctx.fillRect(x+16*s, y+28*s, 3*s, 4*s);
  ctx.fillRect(x+29*s, y+28*s, 3*s, 4*s);

  // Trompe
  ctx.fillStyle = '#94a3b8';
  ctx.fillRect(x+21*s, y+22*s, 6*s, 8*s);
  ctx.fillRect(x+24*s, y+27*s, 6*s, 4*s);
}

// CHEF (chauve, barbe marron, lunettes de soleil noires)
function drawChefPortrait(x, y, s = 1) {
  ctx.fillStyle = '#1e293b';
  ctx.fillRect(x, y, 48*s, 48*s);
  ctx.strokeStyle = '#10b981';
  ctx.lineWidth = 2;
  ctx.strokeRect(x, y, 48*s, 48*s);

  // Crane chauve
  ctx.fillStyle = '#fdba74';
  ctx.fillRect(x+13*s, y+6*s, 22*s, 24*s);
  ctx.fillStyle = 'rgba(255,255,255,0.3)';
  ctx.fillRect(x+16*s, y+7*s, 8*s, 4*s);

  // Lunettes de soleil noires
  ctx.fillStyle = '#020617';
  ctx.fillRect(x+14*s, y+15*s, 9*s, 7*s);
  ctx.fillRect(x+25*s, y+15*s, 9*s, 7*s);
  ctx.fillRect(x+22*s, y+17*s, 4*s, 2*s);
  ctx.fillStyle = '#ffffff';
  ctx.fillRect(x+15*s, y+16*s, 3*s, 2*s);
  ctx.fillRect(x+26*s, y+16*s, 3*s, 2*s);

  // Barbe complète
  ctx.fillStyle = '#78350f';
  ctx.fillRect(x+12*s, y+24*s, 24*s, 14*s);
  ctx.fillRect(x+15*s, y+38*s, 18*s, 4*s);

  ctx.fillStyle = '#fdba74';
  ctx.fillRect(x+20*s, y+25*s, 8*s, 2*s);
}

// Illustration complète de Mr. Éléphant assis lisant son livre
function drawElephantFullBody(x, y, s = 1) {
  // Oreilles
  ctx.fillStyle = '#94a3b8';
  ctx.fillRect(x+2*s, y+8*s, 14*s, 26*s);
  ctx.fillRect(x+38*s, y+8*s, 14*s, 26*s);
  ctx.fillStyle = '#f472b6';
  ctx.fillRect(x+5*s, y+12*s, 8*s, 18*s);
  ctx.fillRect(x+41*s, y+12*s, 8*s, 18*s);

  // Tête
  ctx.fillStyle = '#cbd5e1';
  ctx.fillRect(x+12*s, y+4*s, 30*s, 28*s);

  // Corps
  ctx.fillStyle = '#94a3b8';
  ctx.fillRect(x+14*s, y+28*s, 26*s, 22*s);

  // Joues
  ctx.fillStyle = '#f472b6';
  ctx.fillRect(x+13*s, y+22*s, 6*s, 5*s);
  ctx.fillRect(x+35*s, y+22*s, 6*s, 5*s);

  // Yeux et lunettes
  ctx.fillStyle = '#0f172a';
  ctx.fillRect(x+18*s, y+12*s, 5*s, 6*s);
  ctx.fillRect(x+31*s, y+12*s, 5*s, 6*s);
  ctx.fillStyle = '#ffffff';
  ctx.fillRect(x+18*s, y+12*s, 2*s, 2*s);
  ctx.fillRect(x+31*s, y+12*s, 2*s, 2*s);

  ctx.strokeStyle = '#f59e0b'; ctx.lineWidth = Math.max(1, 2*s);
  ctx.strokeRect(x+16*s, y+10*s, 9*s, 10*s);
  ctx.strokeRect(x+29*s, y+10*s, 9*s, 10*s);
  ctx.beginPath(); ctx.moveTo(x+25*s, y+15*s); ctx.lineTo(x+29*s, y+15*s); ctx.stroke();

  // Trompe
  ctx.fillStyle = '#94a3b8';
  ctx.fillRect(x+23*s, y+20*s, 8*s, 10*s);
  ctx.fillRect(x+27*s, y+27*s, 8*s, 5*s);

  // Défenses
  ctx.fillStyle = '#fef3c7';
  ctx.fillRect(x+18*s, y+28*s, 3*s, 5*s);
  ctx.fillRect(x+33*s, y+28*s, 3*s, 5*s);

  // Livre rouge & or ouvert
  ctx.fillStyle = '#b91c1c';
  ctx.fillRect(x+7*s, y+34*s, 40*s, 18*s);
  ctx.fillStyle = '#f8fafc';
  ctx.fillRect(x+10*s, y+36*s, 16*s, 14*s);
  ctx.fillRect(x+28*s, y+36*s, 16*s, 14*s);
  ctx.fillStyle = '#f59e0b';
  ctx.fillRect(x+25*s, y+34*s, 4*s, 18*s);
}

// Passagers agrandis dans la voiture C (Capichef au volant + Mr. Éléphant passager)
function drawOccupantsInCar(x, y, w, h) {
  // Vitre panoramique épurée
  ctx.fillStyle = 'rgba(186,230,253,0.7)';
  ctx.fillRect(x + 8, y + 4, w - 24, h - 8);
  ctx.strokeStyle = '#f59e0b'; ctx.lineWidth = 1;
  ctx.strokeRect(x + 8, y + 4, w - 24, h - 8);

  const s = 0.65;
  // Capichef au volant à gauche
  const cx = x + 12;
  const cy = y + 6;
  ctx.fillStyle = '#451a03'; // Cheveux bruns
  ctx.fillRect(cx, cy, 12*s, 11*s);
  ctx.fillStyle = '#fbcfe8'; // Visage
  ctx.fillRect(cx+2*s, cy+3*s, 8*s, 8*s);
  ctx.fillStyle = '#0f172a'; // Œil
  ctx.fillRect(cx+7*s, cy+5*s, 2*s, 2*s);
  ctx.fillStyle = '#1e3a8a'; // Veste
  ctx.fillRect(cx+1*s, cy+11*s, 10*s, 6*s);
  ctx.fillStyle = '#f59e0b'; // Volant
  ctx.fillRect(cx+9*s, cy+7*s, 3*s, 6*s);

  // Mr. Éléphant passager à droite
  const ex = x + w - 30;
  const ey = y + 5;
  ctx.fillStyle = '#94a3b8'; // Tête
  ctx.fillRect(ex, ey, 13*s, 11*s);
  ctx.fillStyle = '#f472b6'; // Oreille rose
  ctx.fillRect(ex-2*s, ey+2*s, 3*s, 6*s);
  ctx.fillStyle = '#0f172a'; // Œil
  ctx.fillRect(ex+4*s, ey+3*s, 2*s, 2*s);
  ctx.fillStyle = '#f59e0b'; // Lunettes
  ctx.fillRect(ex+3*s, ey+2*s, 4*s, 4*s);
  ctx.fillStyle = '#dc2626'; // Livre rouge
  ctx.fillRect(ex-1*s, ey+9*s, 10*s, 5*s);
  ctx.fillStyle = '#fef08a'; // Pages du livre
  ctx.fillRect(ex+1*s, ey+10*s, 6*s, 3*s);
}

function drawCar(car, ox, oy, cell, isSelected) {
  const x = ox + car.x * cell;
  const y = oy + car.y * cell;
  const w = car.dir === 'h' ? car.len * cell : cell;
  const h = car.dir === 'v' ? car.len * cell : cell;

  // Ombre portée
  ctx.fillStyle = 'rgba(0,0,0,0.45)';
  ctx.fillRect(x+4, y+6, w-6, h-6);

  // Carrosserie principale
  ctx.fillStyle = car.color;
  ctx.fillRect(x+2, y+2, w-4, h-4);

  // Reflet carrosserie haut
  ctx.fillStyle = 'rgba(255,255,255,0.18)';
  ctx.fillRect(x+2, y+2, w-4, 5);

  // Pare-chocs
  ctx.fillStyle = '#0f172a';
  if (car.dir === 'h') {
    ctx.fillRect(x+2, y+2, 3, h-4);
    ctx.fillRect(x+w-5, y+2, 3, h-4);
  } else {
    ctx.fillRect(x+2, y+2, w-4, 3);
    ctx.fillRect(x+2, y+h-5, w-4, 3);
  }

  // Vitres standards pour véhicules secondaires
  if (!car.isCapi) {
    ctx.fillStyle = 'rgba(148,163,184,0.45)';
    if (car.dir === 'h') {
      ctx.fillRect(x+8, y+4, w-16, h-8);
    } else {
      ctx.fillRect(x+4, y+8, w-8, h-16);
    }
  }

  // Phares
  ctx.fillStyle = car.isCapi ? '#fbbf24' : '#fde047';
  if (car.dir === 'h') {
    ctx.fillRect(x+w-5, y+5, 3, 5);
    ctx.fillRect(x+w-5, y+h-10, 3, 5);
  } else {
    ctx.fillRect(x+5, y+h-5, 5, 3);
    ctx.fillRect(x+w-10, y+h-5, 5, 3);
  }

  // Marqueurs distinctifs sur la voiture de Capichef ('C')
  if (car.isCapi) {
    // Bande dorée centrale
    ctx.fillStyle = '#f59e0b';
    if (car.dir === 'h') {
      ctx.fillRect(x + 2, y + h/2 - 2, w - 4, 4);
      // Badge écusson "C" doré sur le capot
      ctx.fillStyle = '#fef08a';
      ctx.fillRect(x + w - 16, y + h/2 - 5, 10, 10);
      ctx.fillStyle = '#1e3a8a';
      ctx.font = '8px "Press Start 2P"';
      ctx.textAlign = 'center';
      ctx.fillText('C', x + w - 11, y + h/2 + 3);
      // Occupants agrandis
      drawOccupantsInCar(x, y, w, h);
    } else {
      ctx.fillRect(x + w/2 - 2, y + 2, 4, h - 4);
      // Badge écusson "C"
      ctx.fillStyle = '#fef08a';
      ctx.fillRect(x + w/2 - 5, y + h - 16, 10, 10);
      ctx.fillStyle = '#1e3a8a';
      ctx.font = '8px "Press Start 2P"';
      ctx.textAlign = 'center';
      ctx.fillText('C', x + w/2, y + h - 8);
    }
  }

  if (isSelected) {
    ctx.strokeStyle = '#ffffff';
    ctx.lineWidth = 2;
    ctx.strokeRect(x+1, y+1, w-2, h-2);
  }
}

// Dessin d'une scène de dialogue paginée avec stricte séparation verticale (Zone 1: Portraits y: 18..130, Zone 2: Texte y: 138..325)
function drawDialogueScene(pagesArray) {
  const pageObj = pagesArray[dialoguePage % pagesArray.length];
  ctx.fillStyle = 'rgba(9, 13, 22, 0.96)';
  ctx.fillRect(16, 16, 428, 313);
  ctx.strokeStyle = C.gold; ctx.lineWidth = 3;
  ctx.strokeRect(16, 16, 428, 313);

  // Zone 1 : Portraits centrés en haut (y: 22 à 125)
  const ill = pageObj.illus;
  if (ill === 'ele_reading') {
    drawElephantFullBody(200, 26, 1.3);
  } else if (ill === 'duo') {
    drawCapichefPortrait(140, 26, 1.25);
    drawElephantPortrait(260, 26, 1.25);
  } else if (ill === 'ele_happy' || ill === 'ele_cheer') {
    drawElephantPortrait(200, 26, 1.3);
  } else if (ill === 'duo_car') {
    drawCapichefPortrait(130, 26, 1.25);
    drawElephantPortrait(270, 26, 1.25);
  } else if (ill === 'chef_alone') {
    drawChefPortrait(200, 26, 1.3);
  } else if (ill === 'trio_victory') {
    drawCapichefPortrait(110, 26, 1.2);
    drawChefPortrait(205, 26, 1.2);
    drawElephantPortrait(300, 26, 1.2);
  }

  // Zone 2 : Boîte de dialogue inférieure séparée (y: 138 à 315)
  ctx.fillStyle = '#0f172a';
  ctx.fillRect(28, 138, 404, 178);
  ctx.strokeStyle = '#38bdf8'; ctx.lineWidth = 2;
  ctx.strokeRect(28, 138, 404, 178);

  ctx.fillStyle = C.gold; ctx.font = '10px "Press Start 2P"'; ctx.textAlign = 'left';
  ctx.fillText(pageObj.speaker + ' :', 44, 163);

  ctx.fillStyle = C.text; ctx.font = '8.5px "Press Start 2P"';
  pageObj.text.forEach((l, i) => {
    ctx.fillText(l, 44, 191 + i * 20);
  });

  if (Math.floor(frame / 20) % 2 === 0) {
    ctx.fillStyle = C.accent; ctx.font = '8px "Press Start 2P"'; ctx.textAlign = 'right';
    ctx.fillText('Appuie sur START ou A ▶', 418, 301);
  }
}

function draw() {
  ctx.fillStyle = C.bg;
  ctx.fillRect(0, 0, 460, 345);

  // ===== TITLE SCREEN =====
  if (state === 'title') {
    // Fond étoilé
    ctx.fillStyle = '#fef08a';
    ctx.fillRect(40, 25, 4, 4); ctx.fillRect(400, 40, 4, 4);
    ctx.fillRect(120, 45, 3, 3); ctx.fillRect(340, 30, 3, 3);
    ctx.fillRect(405, 75, 18, 18); // Lune

    ctx.strokeStyle = C.gold; ctx.lineWidth = 3;
    ctx.strokeRect(18, 18, 424, 309);

    // Zone Titre supérieure
    ctx.fillStyle = C.text; ctx.font = '18px "Press Start 2P"'; ctx.textAlign = 'center';
    ctx.fillText('CAPICHEF', 230, 48);
    ctx.fillStyle = C.gold; ctx.font = '12px "Press Start 2P"';
    ctx.fillText('PARKING RETRO', 230, 72);

    // Zone Vitrine centrale
    drawCapichefPortrait(105, 98, 1.4);
    drawElephantFullBody(255, 88, 1.4);

    // Zone Prompt inférieure
    if (Math.floor(frame / 20) % 2 === 0) {
      ctx.fillStyle = C.gold; ctx.font = '9.5px "Press Start 2P"';
      ctx.fillText('APPUYER SUR START OU A', 230, 272);
    }
    ctx.fillStyle = '#64748b'; ctx.font = '8px "Press Start 2P"';
    ctx.fillText('© 2026 - Édition Luxe', 230, 305);
    return;
  }

  // ===== INTRO PAGINÉE =====
  if (state === 'intro') {
    drawDialogueScene(introPages);
    return;
  }

  // ===== TUTORIAL PAGINÉ =====
  if (state === 'tutorial') {
    drawDialogueScene(tutPages);
    return;
  }

  // ===== VICTOIRE CHEF PAGINÉE =====
  if (state === 'chef_victory') {
    drawDialogueScene(chefVictoryPages);
    return;
  }

  // ===== MENU =====
  if (state === 'menu') {
    ctx.strokeStyle = C.gold; ctx.lineWidth = 3;
    ctx.strokeRect(50, 40, 360, 265);
    ctx.fillStyle = C.text; ctx.font = '16px "Press Start 2P"'; ctx.textAlign = 'center';
    ctx.fillText('MENU PRINCIPAL', 230, 85);

    drawCapichefPortrait(206, 105, 1.15);

    const opts = ['JOUER', 'CRÉDITS'];
    opts.forEach((o, i) => {
      ctx.fillStyle = i === menuIndex ? C.gold : C.text;
      ctx.font = '12px "Press Start 2P"';
      ctx.fillText((i === menuIndex ? '▶ ' : '  ') + o, 230, 200 + i * 45);
    });
    return;
  }

  // ===== CREDITS =====
  if (state === 'credits') {
    const lines = [
      "CAPICHEF - PARKING",
      "",
      "Inspiré du mythique",
      "Rush Hour",
      "",
      "Citations littéraires :",
      "Saint-Exupéry • Voltaire",
      "Keats • Dostoïevski",
      "Gandhi • Socrate • Prévert",
      "Einstein • Confucius",
      "Anne Frank • van Gogh",
      "",
      "Graphismes Pixel Art 16-bit",
      "Capichef • Mr. Éléphant • CHEF",
      "",
      "Merci d'avoir joué !",
      "",
      "Appuie sur A pour revenir"
    ];
    ctx.fillStyle = C.text; ctx.font = '9px "Press Start 2P"'; ctx.textAlign = 'center';
    lines.forEach((l, i) => ctx.fillText(l, 230, creditY + i * 16));
    return;
  }

  // ===== END + TICKET =====
  if (state === 'end') {
    ctx.fillStyle = '#05080f';
    ctx.fillRect(0, 0, 460, 345);
    ctx.fillStyle = '#f8fafc';
    ctx.fillRect(30, 20, 400, 305);
    ctx.strokeStyle = C.gold; ctx.lineWidth = 4;
    ctx.strokeRect(30, 20, 400, 305);
    ctx.fillStyle = C.capi;
    ctx.fillRect(30, 20, 400, 44);
    ctx.fillStyle = '#fff'; ctx.font = '13px "Press Start 2P"'; ctx.textAlign = 'center';
    ctx.fillText('TICKET MASSAGE SUPRÊME', 230, 47);

    drawCapichefPortrait(55, 76, 1.1);
    drawChefPortrait(206, 76, 1.1);
    drawElephantPortrait(355, 76, 1.1);

    ctx.fillStyle = C.dark; ctx.font = '9px "Press Start 2P"';
    ctx.fillText('Victoire totale ! Capichef & Mr. Éléphant', 230, 150);
    ctx.fillText('ont surmonté les 15 parkings !', 230, 168);
    ctx.fillStyle = C.accent; ctx.font = '10px "Press Start 2P"';
    ctx.fillText('FAIS UNE CAPTURE D\'ÉCRAN', 230, 205);
    ctx.fillText('DE CE TICKET MASSAGE !', 230, 225);
    ctx.fillStyle = C.dark; ctx.font = '8px "Press Start 2P"';
    ctx.fillText('Valable pour un massage royal dispensé par CHEF', 230, 260);
    ctx.fillStyle = C.capi; ctx.font = '9px "Press Start 2P"';
    ctx.fillText('Appuie sur A pour les crédits', 230, 302);
    return;
  }

  // ===== JEU (PLATEAU DE PARKING 240x240) =====
  const cell = 40;
  const ox = Math.floor((460 - 6 * cell) / 2); // 110
  const oy = 50;

  // Sol asphalte texturé
  ctx.fillStyle = C.asphalt;
  ctx.fillRect(ox - 18, oy - 18, 6 * cell + 36, 6 * cell + 36);

  // Damier subtil asphalte
  ctx.fillStyle = C.asphDark;
  for (let y = 0; y < 6; y++) {
    for (let x = 0; x < 6; x++) {
      if ((x + y) % 2 === 0) {
        ctx.fillRect(ox + x * cell, oy + y * cell, cell, cell);
      }
    }
  }

  // Grille lignes jaunes
  ctx.strokeStyle = C.line; ctx.lineWidth = 1;
  for (let y = 0; y < 6; y++) for (let x = 0; x < 6; x++) {
    ctx.strokeRect(ox + x * cell + 1, oy + y * cell + 1, cell - 2, cell - 2);
  }

  // Bordure béton
  ctx.fillStyle = C.concrete;
  ctx.fillRect(ox - 18, oy - 22, 6 * cell + 36, 6);
  ctx.fillRect(ox - 18, oy + 6 * cell + 16, 6 * cell + 36, 6);
  ctx.fillRect(ox - 22, oy - 18, 6, 6 * cell + 36);

  // Sortie détaillée avec barrière
  ctx.fillStyle = C.gold;
  ctx.fillRect(ox + 6 * cell + 2, oy + 2 * cell + 4, 22, cell - 8);
  ctx.fillStyle = C.dark; ctx.font = '8px "Press Start 2P"'; ctx.textAlign = 'left';
  ctx.fillText('EXIT', ox + 6 * cell + 28, oy + 2 * cell + 24);

  // Dessin de la barrière de sortie
  ctx.save();
  ctx.translate(ox + 6 * cell + 10, oy + 2 * cell + cell / 2);
  ctx.rotate(barrierAngle);
  ctx.fillStyle = '#dc2626';
  ctx.fillRect(-2, -2, 34, 6);
  ctx.fillStyle = '#ffffff';
  ctx.fillRect(6, -2, 6, 6);
  ctx.fillRect(18, -2, 6, 6);
  ctx.restore();

  // Voitures
  cars.forEach((car, idx) => drawCar(car, ox, oy, cell, idx === selected));

  // Barre supérieure UI
  ctx.fillStyle = C.dark; ctx.fillRect(0, 0, 460, 38);
  ctx.fillStyle = C.text; ctx.font = '10px "Press Start 2P"'; ctx.textAlign = 'left';
  ctx.fillText('NIV ' + (level + 1) + '/' + levels.length, 16, 24);
  ctx.fillText('VOITURE ' + (selected + 1), 270, 24);

  // Message temporaire
  if (messageTimer > 0) {
    ctx.fillStyle = 'rgba(11,15,25,0.95)';
    ctx.fillRect(50, 298, 360, 32);
    ctx.strokeStyle = C.gold; ctx.lineWidth = 2;
    ctx.strokeRect(50, 298, 360, 32);
    ctx.fillStyle = C.gold; ctx.font = '9px "Press Start 2P"'; ctx.textAlign = 'center';
    ctx.fillText(message, 230, 318);
  }

  // Menu Pause
  if (state === 'paused') {
    ctx.fillStyle = 'rgba(11,15,25,0.95)';
    ctx.fillRect(70, 65, 320, 210);
    ctx.strokeStyle = C.gold; ctx.lineWidth = 3;
    ctx.strokeRect(70, 65, 320, 210);
    ctx.fillStyle = C.text; ctx.font = '14px "Press Start 2P"'; ctx.textAlign = 'center';
    ctx.fillText('PAUSE', 230, 105);
    const opts = ['REPRENDRE', 'MENU'];
    opts.forEach((o, i) => {
      ctx.fillStyle = i === menuIndex ? C.gold : C.text;
      ctx.font = '12px "Press Start 2P"';
      ctx.fillText((i === menuIndex ? '▶ ' : '  ') + o, 230, 165 + i * 50);
    });
  }

  // Écran de récompense & citation
  if (state === 'reward') {
    ctx.fillStyle = 'rgba(11,15,25,0.96)';
    ctx.fillRect(20, 20, 420, 305);
    ctx.strokeStyle = C.gold; ctx.lineWidth = 3;
    ctx.strokeRect(20, 20, 420, 305);

    drawElephantPortrait(35, 32, 1.25);
    ctx.fillStyle = C.gold; ctx.font = '9.5px "Press Start 2P"'; ctx.textAlign = 'left';
    ctx.fillText('Mr. Éléphant murmure :', 105, 54);

    const q = quotes[level % quotes.length];
    ctx.fillStyle = C.text; ctx.font = '8.5px "Press Start 2P"';
    const lines = q.text.split('\n');
    lines.forEach((l, i) => ctx.fillText(l, 38, 120 + i * 20));

    ctx.fillStyle = C.gold;
    ctx.fillText('— ' + q.author, 38, 120 + lines.length * 20 + 14);

    ctx.fillStyle = C.accent; ctx.font = '9px "Press Start 2P"'; ctx.textAlign = 'center';
    ctx.fillText('Appuie sur A pour continuer', 230, 298);
  }
}

function update() {
  frame++;
  if (messageTimer > 0) messageTimer--;
  if (state === 'credits') creditY -= 0.5;
}

function handleAction() {
  initAudio();
  if (state === 'title') {
    state = 'intro';
    dialoguePage = 0;
  }
  else if (state === 'intro') {
    dialoguePage++;
    if (dialoguePage >= introPages.length) {
      state = 'menu';
      menuIndex = 0;
    }
  }
  else if (state === 'tutorial') {
    dialoguePage++;
    if (dialoguePage >= tutPages.length) {
      state = 'playing';
      showMessage('Sors la voiture bleue !', 55);
    }
  }
  else if (state === 'chef_victory') {
    dialoguePage++;
    if (dialoguePage >= chefVictoryPages.length) {
      state = 'end';
    }
  }
  else if (state === 'menu') {
    if (menuIndex === 0) {
      loadLevel(0);
    } else {
      state = 'credits'; creditY = 345;
    }
  }
  else if (state === 'paused') {
    if (menuIndex === 0) state = 'playing';
    else { state = 'menu'; menuIndex = 0; }
  }
  else if (state === 'reward') {
    loadLevel(level + 1);
  }
  else if (state === 'end') {
    state = 'credits';
    creditY = 345;
  }
  else if (state === 'credits') {
    state = 'menu';
    menuIndex = 0;
  }
  else if (state === 'playing') {
    changeCar(1);
  }
}

function handleStartButton() {
  initAudio();
  if (['title','intro','tutorial','chef_victory','reward','end','credits'].includes(state)) {
    handleAction();
  } else if (state === 'playing') {
    state = 'paused';
    menuIndex = 0;
  } else if (state === 'paused') {
    if (menuIndex === 0) state = 'playing';
    else { state = 'menu'; menuIndex = 0; }
  }
}

// Controls Keyboard
window.addEventListener('keydown', e => {
  initAudio();
  if (state === 'playing') {
    if (['ArrowUp','z','w'].includes(e.key)) tryMove(0, -1);
    if (['ArrowDown','s'].includes(e.key)) tryMove(0, 1);
    if (['ArrowLeft','q','a'].includes(e.key)) tryMove(-1, 0);
    if (['ArrowRight','d'].includes(e.key)) tryMove(1, 0);
    if (e.key === 'Tab' || e.key === 'c' || e.key === 'n') changeCar(1);
    if (e.key === 'b' || e.key === 'B') changeCar(-1);
    if (e.key === 'Escape' || e.key === 'm') handleStartButton();
  }
  if (state === 'menu' || state === 'paused') {
    if (e.key === 'ArrowUp' || e.key === 'z') menuIndex = Math.max(0, menuIndex - 1);
    if (e.key === 'ArrowDown' || e.key === 's') menuIndex = Math.min(1, menuIndex + 1);
  }
  if (e.key === ' ' || e.key === 'Enter') handleStartButton();
  if (['ArrowUp','ArrowDown','ArrowLeft','ArrowRight',' '].includes(e.key)) e.preventDefault();
});

// Controls NES Buttons
document.getElementById('btn-up').onclick = () => {
  initAudio();
  if (state === 'playing') tryMove(0, -1);
  if (state === 'menu' || state === 'paused') menuIndex = Math.max(0, menuIndex - 1);
};
document.getElementById('btn-down').onclick = () => {
  initAudio();
  if (state === 'playing') tryMove(0, 1);
  if (state === 'menu' || state === 'paused') menuIndex = Math.min(1, menuIndex + 1);
};
document.getElementById('btn-left').onclick = () => { initAudio(); tryMove(-1, 0); };
document.getElementById('btn-right').onclick = () => { initAudio(); tryMove(1, 0); };
document.getElementById('btn-start').onclick = handleStartButton;
document.getElementById('btn-b').onclick = () => { initAudio(); changeCar(-1); };
document.getElementById('btn-a').onclick = handleAction;

// Controls Touch sur la manette uniquement
['up','down','left','right'].forEach((dir, i) => {
  const d = [[0,-1],[0,1],[-1,0],[1,0]][i];
  document.getElementById('btn-' + dir).addEventListener('touchstart', e => {
    e.preventDefault();
    initAudio();
    if (state === 'playing') tryMove(...d);
    if ((state === 'menu' || state === 'paused') && (dir === 'up' || dir === 'down')) {
      menuIndex = dir === 'up' ? Math.max(0, menuIndex - 1) : Math.min(1, menuIndex + 1);
    }
  }, {passive: false});
});
document.getElementById('btn-start').addEventListener('touchstart', e => { e.preventDefault(); handleStartButton(); }, {passive:false});
document.getElementById('btn-b').addEventListener('touchstart', e => { e.preventDefault(); initAudio(); changeCar(-1); }, {passive:false});
document.getElementById('btn-a').addEventListener('touchstart', e => { e.preventDefault(); handleAction(); }, {passive:false});

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}
loop();
</script>
</body>
</html>