# Anti-Productivity Simulator Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file `index.html` browser game that parodies corporate productivity tools with Zelda SNES aesthetics and 8 absurd mini-interactions.

**Architecture:** Everything lives in one `index.html` file — HTML structure, CSS (including all animations), and JS (modular functions per section). The game uses a simple global `GameState` object and function-per-interaction pattern. No frameworks, no build step, no external assets.

**Tech Stack:** HTML5, CSS3 (custom properties, animations, transforms), Vanilla JS (Web Audio API, requestAnimationFrame, DOM manipulation), Google Fonts CDN (Press Start 2P).

---

## File Structure

```
anti-productivity-simulator/
└── index.html   ← entire game: HTML skeleton + CSS + JS
```

All code sections are separated by clear comments:
```
<!-- ===== HTML ===== -->
<style>/* ===== CSS ===== */</style>
<script>/* ===== JS: STATE | SCREENS | HUD | DRAIN | ENGINE | INTERACTIONS | ACHIEVEMENTS | SOUND | MESSAGES */</script>
```

---

## Task 1: HTML Skeleton + CSS Foundation

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create index.html with DOCTYPE, Google Fonts import, CSS variables, and screen divs**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Anti-Productivity Simulator</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
  <style>
    /* ===== CSS VARIABLES ===== */
    :root {
      --green-dark: #1a3a1a;
      --gold: #c8a200;
      --beige: #f0e68c;
      --red-dark: #8b0000;
      --black: #0a0a0a;
      --corporate-gray: #f5f5f5;
      --corporate-blue: #2563eb;
    }

    * { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      width: 100vw;
      height: 100vh;
      overflow: hidden;
      background: var(--black);
    }

    /* ===== SCREEN CONTAINERS ===== */
    .screen {
      display: none;
      width: 100%;
      height: 100%;
      position: absolute;
      top: 0; left: 0;
    }
    .screen.active { display: flex; }

    /* ===== PIXEL FONT UTILITY ===== */
    .pixel { font-family: 'Press Start 2P', monospace; }
  </style>
</head>
<body>
  <!-- Corporate Screen -->
  <div id="screen-corporate" class="screen"></div>

  <!-- Loading Screen -->
  <div id="screen-loading" class="screen"></div>

  <!-- Game Screen -->
  <div id="screen-game" class="screen"></div>

  <!-- Game Over Screen -->
  <div id="screen-gameover" class="screen"></div>

  <!-- Achievement Overlay (always present, shown/hidden) -->
  <div id="achievement-overlay" style="display:none; position:fixed; top:0;left:0;width:100%;height:100%;pointer-events:none;z-index:100;"></div>

  <script>
    /* ===== JS PLACEHOLDER ===== */
    console.log('Anti-Productivity Simulator loaded');
  </script>
</body>
</html>
```

- [ ] **Step 2: Open index.html in browser, open DevTools console**

Expected: "Anti-Productivity Simulator loaded" in console. No errors. Blank black screen.

- [ ] **Step 3: Commit**

```bash
cd /home/mauricionunez/Desktop/anti-productivity-simulator
git add index.html
git commit -m "feat: html skeleton with CSS variables and screen containers"
```

---

## Task 2: Corporate Landing Screen

**Files:**
- Modify: `index.html` — add corporate screen CSS + HTML content + glitch keyframes

- [ ] **Step 1: Add CSS for corporate screen and glitch animation inside the `<style>` block**

```css
/* ===== CORPORATE SCREEN ===== */
#screen-corporate {
  background: var(--corporate-gray);
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

.corporate-logo {
  font-size: 14px;
  font-weight: 300;
  letter-spacing: 4px;
  color: #888;
  margin-bottom: 60px;
  text-transform: uppercase;
}

.corporate-logo span { color: var(--corporate-blue); font-weight: 600; }

.corporate-headline {
  font-size: 42px;
  font-weight: 700;
  color: #1a1a1a;
  margin-bottom: 16px;
  text-align: center;
}

.corporate-subtext {
  font-size: 18px;
  color: #666;
  margin-bottom: 48px;
}

.corporate-btn {
  background: var(--corporate-blue);
  color: white;
  border: none;
  padding: 16px 48px;
  font-size: 16px;
  font-weight: 600;
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.1s, box-shadow 0.1s;
  box-shadow: 0 4px 12px rgba(37,99,235,0.4);
}

.corporate-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(37,99,235,0.5);
}

/* Glitch animation */
@keyframes glitch {
  0%   { clip-path: inset(0 0 95% 0); transform: translateX(-5px); }
  10%  { clip-path: inset(30% 0 50% 0); transform: translateX(5px); }
  20%  { clip-path: inset(60% 0 20% 0); transform: translateX(-3px); }
  30%  { clip-path: inset(10% 0 80% 0); transform: translateX(3px); }
  40%  { clip-path: inset(50% 0 10% 0); transform: translateX(-5px); }
  50%  { clip-path: inset(0 0 0 0);     transform: translateX(0); opacity: 0.5; }
  60%  { clip-path: inset(20% 0 60% 0); transform: translateX(4px); }
  70%  { clip-path: inset(70% 0 5% 0);  transform: translateX(-4px); }
  80%  { clip-path: inset(5% 0 90% 0);  transform: translateX(2px); }
  90%  { clip-path: inset(40% 0 40% 0); transform: translateX(-2px); }
  100% { clip-path: inset(0 0 0 0);     transform: translateX(0); opacity: 0; }
}

.glitching { animation: glitch 0.5s steps(1) forwards; }
```

- [ ] **Step 2: Add corporate screen HTML content and initial `active` class**

Replace `<div id="screen-corporate" class="screen"></div>` with:
```html
<div id="screen-corporate" class="screen active">
  <div class="corporate-logo">Pro<span>Flow</span>™</div>
  <h1 class="corporate-headline">Optimize your productivity</h1>
  <p class="corporate-subtext">Start your focused work session</p>
  <button class="corporate-btn" id="btn-start">Start Working</button>
</div>
```

- [ ] **Step 3: Add JS in script block: showCorporateScreen(), showLoadingScreen() stub, and button listener**

```js
// ===== STATE =====
const GameState = {
  screen: 'corporate',
  focusPoints: 100,
  timeWasted: 0,
  interactionCount: 0,
  achievements: new Set(),
  currentInteraction: null,
  lastInteraction: null,
  passiveDrainPaused: false,
  cleanupFn: null,
  firstInteractionTime: null,
  _timerInterval: null,
  _drainInterval: null
};

function resetGameState() {
  GameState.screen = 'corporate';
  GameState.focusPoints = 100;
  GameState.timeWasted = 0;
  GameState.interactionCount = 0;
  GameState.achievements = new Set();
  GameState.currentInteraction = null;
  GameState.lastInteraction = null;
  GameState.passiveDrainPaused = false;
  GameState.cleanupFn = null;
  GameState.firstInteractionTime = null;
  if (GameState._timerInterval) clearInterval(GameState._timerInterval);
  if (GameState._drainInterval) clearInterval(GameState._drainInterval);
  GameState._timerInterval = null;
  GameState._drainInterval = null;
}

// ===== SCREEN HELPERS =====
function showScreen(id) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  document.getElementById('screen-' + id).classList.add('active');
  GameState.screen = id;
}

// ===== SCREENS =====
function showCorporateScreen() {
  resetGameState();
  showScreen('corporate');
}

function showLoadingScreen() {
  // Placeholder — implemented in Task 3
  showScreen('loading');
}

// ===== BUTTON WIRING =====
document.getElementById('btn-start').addEventListener('click', function() {
  const corp = document.getElementById('screen-corporate');
  corp.classList.add('glitching');
  setTimeout(() => showLoadingScreen(), 500);
});
```

- [ ] **Step 4: Open in browser, click "Start Working"**

Expected: corporate screen glitches for 500ms, then goes to blank loading screen. No errors.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: corporate landing screen with glitch transition"
```

---

## Task 3: Loading Screen

**Files:**
- Modify: `index.html` — loading screen CSS + HTML + JS sequence

- [ ] **Step 1: Add loading screen CSS**

```css
/* ===== LOADING SCREEN ===== */
#screen-loading {
  background: var(--black);
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 24px;
}

.loading-text {
  font-family: 'Press Start 2P', monospace;
  color: var(--beige);
  font-size: 14px;
  line-height: 2;
  text-align: center;
  min-height: 60px;
}

.loading-bar-container {
  width: 300px;
  height: 20px;
  border: 3px solid var(--gold);
  box-shadow: 4px 4px 0 #000, -2px -2px 0 #000;
  background: #0a0a0a;
}

.loading-bar-fill {
  height: 100%;
  width: 0%;
  background: var(--gold);
  transition: none;
}
```

- [ ] **Step 2: Add loading screen HTML**

Replace loading screen div with:
```html
<div id="screen-loading" class="screen">
  <div class="loading-text" id="loading-text">LOADING SAVE FILE...</div>
  <div class="loading-bar-container">
    <div class="loading-bar-fill" id="loading-bar"></div>
  </div>
</div>
```

- [ ] **Step 3: Implement showLoadingScreen() with the full timed sequence**

Replace the showLoadingScreen stub with:
```js
function showLoadingScreen() {
  showScreen('loading');
  const txt = document.getElementById('loading-text');
  const bar = document.getElementById('loading-bar');

  txt.textContent = 'LOADING SAVE FILE...';
  bar.style.width = '0%';

  // Fill bar unevenly over ~2000ms
  let progress = 0;
  const fillInterval = setInterval(() => {
    progress += Math.random() * 15 + 3;
    if (progress >= 100) {
      progress = 100;
      clearInterval(fillInterval);
      bar.style.width = '100%';

      // Reset and show CORRUPT
      setTimeout(() => {
        bar.style.width = '0%';
        txt.textContent = 'FILE CORRUPT...';

        setTimeout(() => {
          txt.textContent = 'LOADING ANYWAY';

          setTimeout(() => {
            showGameScreen();
          }, 400);
        }, 600);
      }, 800);
    } else {
      bar.style.width = progress + '%';
    }
  }, 100);
}

// stub for next task
function showGameScreen() {
  showScreen('game');
}
```

- [ ] **Step 4: Open in browser, click "Start Working"**

Expected: loading screen appears, bar fills unevenly, hits 100%, resets, shows "FILE CORRUPT...", then "LOADING ANYWAY", then blank game screen. Timing feels slightly broken/glitchy (intentional).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: RPG loading screen with fake corrupt save sequence"
```

---

## Task 4: Game Screen Layout + HUD

**Files:**
- Modify: `index.html` — game screen CSS + HTML + HUD JS functions

- [ ] **Step 1: Add game screen and HUD CSS**

```css
/* ===== GAME SCREEN ===== */
#screen-game {
  background: var(--green-dark);
  flex-direction: column;
  position: relative;
}

/* CRT scanlines overlay */
#screen-game::after {
  content: '';
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 2px,
    rgba(0,0,0,0.3) 2px,
    rgba(0,0,0,0.3) 4px
  );
  pointer-events: none;
  z-index: 10;
}

/* CRT flicker */
@keyframes crt-flicker {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.97; }
}
#screen-game { animation: crt-flicker 4s ease-in-out infinite; }

/* HUD */
.game-hud {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 16px;
  background: var(--black);
  border-bottom: 4px solid var(--gold);
  box-shadow: 0 4px 0 #000;
  font-family: 'Press Start 2P', monospace;
  font-size: 10px;
  color: var(--beige);
  z-index: 5;
  flex-shrink: 0;
}

.hud-title { color: var(--gold); font-size: 8px; }
.hud-focus { display: flex; align-items: center; gap: 6px; }
.hud-focus span { font-size: 16px; }
.hud-timer { color: var(--beige); }

/* Interaction area */
.game-interaction {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  position: relative;
}

/* Zelda dialog box */
.dialog-box {
  background: var(--green-dark);
  border: none;
  box-shadow:
    0 0 0 4px var(--gold),
    0 0 0 8px var(--black),
    0 0 0 12px var(--gold);
  padding: 24px 32px;
  max-width: 700px;
  width: 100%;
  min-height: 300px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  position: relative;
}

.dialog-title {
  font-family: 'Press Start 2P', monospace;
  font-size: 12px;
  color: var(--gold);
  text-align: center;
  border-bottom: 2px solid var(--gold);
  padding-bottom: 12px;
  width: 100%;
}

#interaction-area {
  width: 100%;
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  position: relative;
  overflow: hidden;
}

/* Message bar */
.game-message-bar {
  background: var(--black);
  border-top: 4px solid var(--gold);
  padding: 12px 24px;
  font-family: 'Press Start 2P', monospace;
  font-size: 10px;
  color: var(--beige);
  min-height: 48px;
  display: flex;
  align-items: center;
  flex-shrink: 0;
}

/* HUD flicker when focus <= 20 */
@keyframes hud-danger {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}
.hud-danger { animation: hud-danger 0.5s ease-in-out infinite; }
```

- [ ] **Step 2: Add game screen HTML**

Replace game screen div with:
```html
<div id="screen-game" class="screen">
  <!-- HUD -->
  <div class="game-hud">
    <div class="hud-title">ANTI-PRODUCTIVITY SIM</div>
    <div class="hud-focus" id="hud-focus">
      <span>🧠</span><span>🧠</span><span>🧠</span><span>🧠</span><span>🧠</span>
    </div>
    <div class="hud-timer" id="hud-timer">⏱ 00:00</div>
  </div>

  <!-- Interaction area -->
  <div class="game-interaction">
    <div class="dialog-box">
      <div class="dialog-title" id="interaction-title">LOADING...</div>
      <div id="interaction-area"></div>
    </div>
  </div>

  <!-- Message bar -->
  <div class="game-message-bar" id="message-bar"></div>
</div>
```

- [ ] **Step 3: Implement HUD functions in JS**

```js
// ===== HUD =====
function updateFocusDisplay() {
  const fp = GameState.focusPoints;
  const icons = document.querySelectorAll('#hud-focus span');
  const thresholds = [80, 60, 40, 20, 0]; // below this value = skull
  // icon 0 = rightmost (first to die)
  // Map: 5 icons, icon i dies when fp <= (5-i)*20
  for (let i = 0; i < 5; i++) {
    const threshold = (5 - i) * 20;
    icons[4 - i].textContent = fp <= threshold ? '💀' : '🧠';
  }
  const hud = document.querySelector('.game-hud');
  if (fp <= 20) hud.classList.add('hud-danger');
  else hud.classList.remove('hud-danger');
}

function formatTime(seconds) {
  const m = Math.floor(seconds / 60).toString().padStart(2, '0');
  const s = (seconds % 60).toString().padStart(2, '0');
  return m + ':' + s;
}

function updateHUD() {
  updateFocusDisplay();
  document.getElementById('hud-timer').textContent = '⏱ ' + formatTime(GameState.timeWasted);
}

function startTimerWasted() {
  if (GameState._timerInterval) clearInterval(GameState._timerInterval);
  GameState._timerInterval = setInterval(() => {
    GameState.timeWasted++;
    updateHUD();
  }, 1000);
}
```

- [ ] **Step 4: Update showGameScreen() to render HUD**

```js
function showGameScreen() {
  showScreen('game');
  updateHUD();
  startTimerWasted();
  // Drain and interactions wired in Task 5
}
```

- [ ] **Step 5: Open in browser, click through to game screen**

Expected: dark green game screen, gold HUD at top with 5 brains, timer counting up from 00:00, Zelda dialog box in center, message bar at bottom. Scanlines visible over the screen.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: game screen layout with HUD, dialog box, scanlines, CRT flicker"
```

---

## Task 5: Core Engine (State, Drain, Engine Functions)

**Files:**
- Modify: `index.html` — passive drain, applyFocusDelta, interaction engine, game over screen

- [ ] **Step 1: Add game over screen CSS and HTML**

CSS:
```css
/* ===== GAME OVER SCREEN ===== */
#screen-gameover {
  background: var(--black);
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 20px;
  font-family: 'Press Start 2P', monospace;
  text-align: center;
}

.gameover-title {
  font-size: 48px;
  color: var(--red-dark);
  text-shadow: 4px 4px 0 #000;
  letter-spacing: 4px;
}

.gameover-subtitle {
  font-size: 14px;
  color: var(--beige);
  margin-bottom: 20px;
}

.gameover-stats {
  color: var(--gold);
  font-size: 10px;
  line-height: 2.5;
}

.pixel-btn {
  font-family: 'Press Start 2P', monospace;
  font-size: 12px;
  background: var(--green-dark);
  color: var(--gold);
  border: none;
  padding: 14px 28px;
  cursor: pointer;
  box-shadow: 4px 4px 0 var(--gold), -2px -2px 0 var(--gold);
  margin-top: 20px;
}

.pixel-btn:hover { background: var(--gold); color: var(--black); }
```

HTML — replace gameover div:
```html
<div id="screen-gameover" class="screen">
  <div class="gameover-title">GAME OVER</div>
  <div class="gameover-subtitle">YOU DID ACTUAL WORK</div>
  <div class="gameover-stats" id="gameover-stats"></div>
  <button class="pixel-btn" id="btn-play-again">[ PLAY AGAIN ]</button>
</div>
```

- [ ] **Step 2: Implement applyFocusDelta, showGameOverScreen, passive drain, and interaction engine stubs**

```js
// ===== DRAIN =====
function applyFocusDelta(n) {
  GameState.focusPoints = Math.max(0, GameState.focusPoints + n);
  updateFocusDisplay();
  checkAchievements();
  if (GameState.focusPoints === 0) {
    showGameOverScreen();
  }
}

function startPassiveDrain() {
  if (GameState._drainInterval) clearInterval(GameState._drainInterval);
  GameState._drainInterval = setInterval(() => {
    if (!GameState.passiveDrainPaused) {
      playSound('drain');
      applyFocusDelta(-1);
    }
  }, 3000);
}

// ===== GAME OVER =====
function showGameOverScreen() {
  teardownCurrentInteraction();
  if (GameState._timerInterval) clearInterval(GameState._timerInterval);
  if (GameState._drainInterval) clearInterval(GameState._drainInterval);
  showScreen('gameover');

  const stats = document.getElementById('gameover-stats');
  stats.textContent =
    'TIME WASTED: ' + formatTime(GameState.timeWasted) + '\n' +
    'INTERACTIONS SURVIVED: ' + GameState.interactionCount + '\n' +
    'ACHIEVEMENTS: ' + GameState.achievements.size + ' / 6';
  stats.style.whiteSpace = 'pre';
}

document.getElementById('btn-play-again').addEventListener('click', showCorporateScreen);

// ===== ENGINE =====
function teardownCurrentInteraction() {
  if (GameState.cleanupFn) {
    try { GameState.cleanupFn(); } catch(e) {}
    GameState.cleanupFn = null;
  }
}

const IRONIC_MESSAGES = [
  'PRODUCTIVITY DECREASED SUCCESSFULLY',
  'GREAT JOB AVOIDING WORK',
  'YOU ARE NOW 3% LESS EFFICIENT',
  'BOSS IMPRESSION: MAXIMUM',
  'THIS SESSION HAS BEEN LOGGED',
  'HR HAS BEEN NOTIFIED (JUST KIDDING)',
  'CONGRATULATIONS ON YOUR PROGRESS (THERE IS NONE)',
  'YOUR CONTRIBUTION HAS BEEN NOTED AND IGNORED',
  'SYNERGY LEVELS: CRITICAL',
  'PLEASE UPDATE YOUR STATUS IN JIRA'
];

function completeInteraction(penalty, msg) {
  const id = GameState.currentInteraction;
  playSound('complete');
  teardownCurrentInteraction();
  applyFocusDelta(penalty);
  GameState.interactionCount++;
  GameState.lastInteraction = id;
  if (GameState.firstInteractionTime === null) {
    GameState.firstInteractionTime = GameState.timeWasted;
  }
  checkAchievements();

  typewriterMessage(msg, () => {
    setTimeout(() => {
      const pool = IRONIC_MESSAGES;
      const ironic = pool[Math.floor(Math.random() * pool.length)];
      typewriterMessage(ironic, () => {
        if (GameState.screen === 'game') {
          setTimeout(() => loadRandomInteraction(), 1000);
        }
      });
    }, 500);
  });
}

const INTERACTIONS = []; // filled in Tasks 9-16

function loadRandomInteraction() {
  if (GameState.screen !== 'game') return;
  const available = INTERACTIONS.filter(i => i.id !== GameState.lastInteraction);
  const pick = available[Math.floor(Math.random() * available.length)];
  GameState.currentInteraction = pick.id;
  document.getElementById('interaction-area').textContent = '';
  pick.fn();
}
```

- [ ] **Step 3: Update showGameScreen() to start drain and load first interaction (after INTERACTIONS are filled)**

```js
function showGameScreen() {
  showScreen('game');
  updateHUD();
  startTimerWasted();
  startPassiveDrain();
  // Load first interaction after short delay so game screen is visible
  setTimeout(() => loadRandomInteraction(), 500);
}
```

- [ ] **Step 4: Add stub checkAchievements, typewriterMessage, playSound (full impl in later tasks)**

```js
function checkAchievements() {} // filled in Task 17
function showAchievement(id) {} // filled in Task 17
function playSound(type) {}     // filled in Task 6

function typewriterMessage(text, onDone) {
  const bar = document.getElementById('message-bar');
  bar.textContent = '';
  let i = 0;
  const interval = setInterval(() => {
    bar.textContent += text[i];
    i++;
    if (i >= text.length) {
      clearInterval(interval);
      if (onDone) onDone();
    }
  }, 50);
}
```

- [ ] **Step 5: Verify: open browser, navigate to game screen (no interactions loaded yet)**

Expected: HUD timer ticks, no drain errors in console, game over screen reachable via manual call `applyFocusDelta(-100)` in DevTools console. Stats display correctly.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: core engine - state, passive drain, completeInteraction, game over screen"
```

---

## Task 6: Sound System

**Files:**
- Modify: `index.html` — replace playSound stub with Web Audio API implementation

- [ ] **Step 1: Replace playSound stub with full Web Audio implementation**

```js
// ===== SOUND =====
let _audioCtx = null;
function getAudioCtx() {
  if (!_audioCtx) _audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  return _audioCtx;
}

function playTone(freq, type, duration, gainVal = 0.3) {
  try {
    const ctx = getAudioCtx();
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.connect(gain);
    gain.connect(ctx.destination);
    osc.type = type;
    osc.frequency.setValueAtTime(freq, ctx.currentTime);
    gain.gain.setValueAtTime(gainVal, ctx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + duration);
    osc.start(ctx.currentTime);
    osc.stop(ctx.currentTime + duration);
  } catch(e) {}
}

function playSound(type) {
  switch(type) {
    case 'click':
      playTone(880, 'square', 0.08);
      break;
    case 'drain':
      playTone(220, 'square', 0.15, 0.15);
      break;
    case 'complete':
      playTone(440, 'square', 0.1);
      setTimeout(() => playTone(330, 'square', 0.1), 120);
      break;
    case 'meeting':
      playTone(200, 'sawtooth', 0.15, 0.4);
      setTimeout(() => playTone(188, 'sawtooth', 0.15, 0.4), 100);
      break;
    case 'achievement':
      // C4-E4-G4-C5 arpeggio
      [261, 329, 392, 523].forEach((f, i) => {
        setTimeout(() => playTone(f, 'square', 0.2, 0.3), i * 80);
      });
      break;
  }
}
```

- [ ] **Step 2: Test sounds manually in DevTools console**

Run in console:
```js
playSound('click');
playSound('drain');
playSound('complete');
playSound('meeting');
playSound('achievement');
```

Expected: each produces a distinct 8-bit sound. No errors. (Note: browser may require user interaction before AudioContext starts — clicking anything on screen first satisfies this.)

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: 8-bit sound system via Web Audio API"
```

---

## Task 7: INT-01 — Click Quest

**Files:**
- Modify: `index.html` — add interactionClickQuest() and register in INTERACTIONS

- [ ] **Step 1: Add pixel button CSS and confetti CSS**

```css
/* ===== SHARED INTERACTION STYLES ===== */
.int-counter {
  font-family: 'Press Start 2P', monospace;
  font-size: 18px;
  color: var(--beige);
  margin-bottom: 16px;
}

.int-btn {
  font-family: 'Press Start 2P', monospace;
  font-size: 11px;
  padding: 12px 24px;
  background: var(--green-dark);
  color: var(--gold);
  border: 3px solid var(--gold);
  cursor: pointer;
  box-shadow: 4px 4px 0 #000;
  transition: transform 0.05s;
}
.int-btn:active { transform: translate(2px, 2px); box-shadow: 2px 2px 0 #000; }

/* Confetti particles */
.confetti-particle {
  position: absolute;
  width: 6px;
  height: 6px;
  pointer-events: none;
}

@keyframes confetti-fall {
  to { transform: translateY(200px) rotate(720deg); opacity: 0; }
}
```

- [ ] **Step 2: Implement interactionClickQuest()**

```js
function interactionClickQuest() {
  GameState.currentInteraction = 'click_quest';
  const area = document.getElementById('interaction-area');
  const title = document.getElementById('interaction-title');
  title.textContent = 'UNLOCK MOTIVATION';

  let count = 0;
  const target = 37;
  const colors = ['#c8a200', '#f0e68c', '#8b0000', '#1a8b1a', '#ffffff'];

  area.innerHTML = '';
  const counter = document.createElement('div');
  counter.className = 'int-counter';
  counter.id = 'click-counter';
  counter.textContent = '0 / 37 CLICKS';

  const btn = document.createElement('button');
  btn.className = 'int-btn';
  btn.textContent = 'CLICK ME';

  area.appendChild(counter);
  area.appendChild(btn);

  function spawnConfetti() {
    for (let i = 0; i < 20; i++) {
      const p = document.createElement('div');
      p.className = 'confetti-particle';
      p.style.cssText = `
        left: ${Math.random() * 100}%;
        top: ${Math.random() * 50}%;
        background: ${colors[Math.floor(Math.random() * colors.length)]};
        animation: confetti-fall ${0.5 + Math.random() * 0.5}s ease-out forwards;
        animation-delay: ${Math.random() * 0.3}s;
      `;
      area.appendChild(p);
    }
  }

  function onClick() {
    playSound('click');
    count++;
    counter.textContent = count + ' / 37 CLICKS';
    if (count >= target) {
      btn.removeEventListener('click', onClick);
      spawnConfetti();
      setTimeout(() => {
        showAchievement('CLICKED_WITHOUT_PURPOSE');
        completeInteraction(-8, 'MOTIVATION NOT FOUND');
      }, 1000);
    }
  }

  btn.addEventListener('click', onClick);

  GameState.cleanupFn = () => {
    btn.removeEventListener('click', onClick);
    area.innerHTML = '';
    title.textContent = '';
  };
}
```

- [ ] **Step 3: Register in INTERACTIONS array**

```js
const INTERACTIONS = [
  { id: 'click_quest', fn: interactionClickQuest },
];
```

- [ ] **Step 4: Open in browser, play through INT-01**

Expected: Dialog shows "UNLOCK MOTIVATION", counter "0 / 37 CLICKS", clicking button increments counter and plays click sound, at 37 confetti explodes for 1s, then interaction ends and "MOTIVATION NOT FOUND" types out in message bar.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: INT-01 click quest interaction"
```

---

## Task 8: INT-02 — Elusive Button

**Files:**
- Modify: `index.html` — add interactionElusiveButton()

- [ ] **Step 1: Implement interactionElusiveButton()**

```js
function interactionElusiveButton() {
  GameState.currentInteraction = 'elusive_button';
  const area = document.getElementById('interaction-area');
  const title = document.getElementById('interaction-title');
  title.textContent = 'SUBMIT REPORT';

  area.innerHTML = '';
  area.style.position = 'relative';

  const attemptsLabel = document.createElement('div');
  attemptsLabel.className = 'int-counter';
  attemptsLabel.style.fontSize = '11px';
  attemptsLabel.textContent = 'ATTEMPTS REMAINING: 10';

  const btn = document.createElement('button');
  btn.className = 'int-btn';
  btn.textContent = 'SUBMIT';
  btn.style.position = 'absolute';
  btn.style.left = '50%';
  btn.style.top = '50%';
  btn.style.transform = 'translate(-50%, -50%)';

  area.appendChild(attemptsLabel);
  area.appendChild(btn);

  let teleports = 0;
  let frozen = false;
  let timeoutId = null;

  function teleport() {
    if (frozen) return;
    const areaRect = area.getBoundingClientRect();
    const btnRect = btn.getBoundingClientRect();
    let newLeft, newTop, dx, dy;
    do {
      newLeft = Math.random() * (areaRect.width - 120) + 60;
      newTop = Math.random() * (areaRect.height - 60) + 30;
      dx = newLeft - (btnRect.left - areaRect.left + btnRect.width / 2);
      dy = newTop - (btnRect.top - areaRect.top + btnRect.height / 2);
    } while (Math.sqrt(dx*dx + dy*dy) < 100);

    btn.style.left = newLeft + 'px';
    btn.style.top = newTop + 'px';
    btn.style.transform = 'none';

    teleports++;
    attemptsLabel.textContent = 'ATTEMPTS REMAINING: ' + Math.max(0, 10 - teleports);

    if (teleports >= 10) freeze();
  }

  function freeze() {
    if (frozen) return;
    frozen = true;
    if (timeoutId) clearTimeout(timeoutId);
    btn.removeEventListener('mouseover', teleport);
    btn.textContent = 'OK FINE, CLICK ME';
    btn.style.border = '3px solid var(--beige)';
    btn.addEventListener('click', onFrozenClick);
  }

  function onFrozenClick() {
    playSound('click');
    btn.removeEventListener('click', onFrozenClick);
    completeInteraction(-10, 'REPORT SUCCESSFULLY AVOIDED');
  }

  btn.addEventListener('mouseover', teleport);
  timeoutId = setTimeout(freeze, 15000);

  GameState.cleanupFn = () => {
    btn.removeEventListener('mouseover', teleport);
    btn.removeEventListener('click', onFrozenClick);
    if (timeoutId) clearTimeout(timeoutId);
    area.innerHTML = '';
    title.textContent = '';
  };
}
```

- [ ] **Step 2: Add to INTERACTIONS array**

```js
{ id: 'elusive_button', fn: interactionElusiveButton },
```

- [ ] **Step 3: Test in browser**

Expected: button teleports on hover, counter decrements from 10, after 10 escapes OR 15s button freezes with "OK FINE, CLICK ME" text and can be clicked to complete.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: INT-02 elusive button interaction"
```

---

## Task 9: INT-03 — Meeting Escape

**Files:**
- Modify: `index.html` — add interactionMeetingEscape()

- [ ] **Step 1: Add CSS for meeting enemy and survive bar**

```css
.meeting-enemy {
  position: absolute;
  width: 32px;
  height: 32px;
  background: #c8a200;
  box-shadow:
    8px 8px 0 #000,
    8px 12px 0 #000,
    16px 8px 0 #000,
    16px 12px 0 #000;
  z-index: 5;
  pointer-events: none;
}

.meeting-label {
  position: absolute;
  top: 36px;
  left: 50%;
  transform: translateX(-50%);
  font-family: 'Press Start 2P', monospace;
  font-size: 6px;
  color: var(--red-dark);
  white-space: nowrap;
}

.survive-bar-container {
  position: absolute;
  bottom: 10px;
  left: 50%;
  transform: translateX(-50%);
  width: 200px;
  height: 14px;
  border: 2px solid var(--gold);
  background: #000;
}

.survive-bar-fill {
  height: 100%;
  width: 0%;
  background: #00cc00;
  transition: width 0.1s linear;
}
```

- [ ] **Step 2: Implement interactionMeetingEscape()**

```js
function interactionMeetingEscape() {
  GameState.currentInteraction = 'meeting_escape';
  const area = document.getElementById('interaction-area');
  const title = document.getElementById('interaction-title');
  title.textContent = '⚠ ENEMY ENCOUNTERED';
  playSound('meeting');

  area.innerHTML = '';
  area.style.position = 'relative';

  // Enemy element
  const enemy = document.createElement('div');
  enemy.className = 'meeting-enemy';
  const label = document.createElement('div');
  label.className = 'meeting-label';
  label.textContent = 'MEETING';
  enemy.appendChild(label);

  // Survive bar
  const barContainer = document.createElement('div');
  barContainer.className = 'survive-bar-container';
  const barFill = document.createElement('div');
  barFill.className = 'survive-bar-fill';
  barContainer.appendChild(barFill);

  const instruction = document.createElement('div');
  instruction.className = 'int-counter';
  instruction.style.fontSize = '9px';
  instruction.style.position = 'absolute';
  instruction.style.top = '8px';
  instruction.textContent = 'SURVIVE 10 SECONDS';

  area.appendChild(enemy);
  area.appendChild(barContainer);
  area.appendChild(instruction);

  const areaRect = area.getBoundingClientRect();
  let enemyX = areaRect.width / 2;
  let enemyY = areaRect.height / 2;
  let mouseX = 0, mouseY = 0;
  let surviveTime = 0;
  let contacts = 0;
  let speed = 1.5;
  let rafId = null;
  let lastTime = null;
  let done = false;

  function onMouseMove(e) {
    const rect = area.getBoundingClientRect();
    mouseX = e.clientX - rect.left;
    mouseY = e.clientY - rect.top;
  }

  area.addEventListener('mousemove', onMouseMove);

  function frame(timestamp) {
    if (done) return;
    if (!lastTime) lastTime = timestamp;
    const dt = (timestamp - lastTime) / 16.67; // normalize to 60fps
    lastTime = timestamp;

    // Move enemy toward cursor
    const dx = mouseX - enemyX;
    const dy = mouseY - enemyY;
    const dist = Math.sqrt(dx * dx + dy * dy);
    if (dist > 1) {
      enemyX += (dx / dist) * speed * dt;
      enemyY += (dy / dist) * speed * dt;
    }

    // Clamp to area
    const rect = area.getBoundingClientRect();
    enemyX = Math.max(0, Math.min(rect.width - 32, enemyX));
    enemyY = Math.max(0, Math.min(rect.height - 60, enemyY));

    enemy.style.left = enemyX + 'px';
    enemy.style.top = enemyY + 'px';

    // Contact check (< 40px from cursor to enemy center)
    const ecx = enemyX + 16;
    const ecy = enemyY + 16;
    const contactDist = Math.sqrt((mouseX - ecx) ** 2 + (mouseY - ecy) ** 2);

    if (contactDist < 40) {
      surviveTime = 0;
      contacts++;
      if (contacts >= 3) speed = 2.5;
      barFill.style.background = '#cc0000';
      setTimeout(() => { barFill.style.background = '#00cc00'; }, 200);
    } else {
      surviveTime += (timestamp - (lastTime - (timestamp - lastTime))) / 1000;
      // Simpler: increment per frame
    }

    // Simpler survive timer: count frames without contact
    barFill.style.width = Math.min(100, (surviveTime / 10) * 100) + '%';

    rafId = requestAnimationFrame(frame);
  }

  // Survive counter via interval (cleaner than rAF timing)
  let surviveInterval = setInterval(() => {
    if (done) return;
    const ecx = enemyX + 16;
    const ecy = enemyY + 16;
    const contactDist = Math.sqrt((mouseX - ecx) ** 2 + (mouseY - ecy) ** 2);
    if (contactDist >= 40) {
      surviveTime += 0.1;
      barFill.style.width = Math.min(100, (surviveTime / 10) * 100) + '%';
      if (surviveTime >= 10) {
        done = true;
        clearInterval(surviveInterval);
        cancelAnimationFrame(rafId);
        area.removeEventListener('mousemove', onMouseMove);
        showAchievement('MEETING_SURVIVOR');
        completeInteraction(-12, 'YOU SURVIVED: MEETING THAT COULD HAVE BEEN AN EMAIL');
      }
    }
  }, 100);

  rafId = requestAnimationFrame(frame);

  GameState.cleanupFn = () => {
    done = true;
    cancelAnimationFrame(rafId);
    clearInterval(surviveInterval);
    area.removeEventListener('mousemove', onMouseMove);
    area.innerHTML = '';
    title.textContent = '';
  };
}
```

- [ ] **Step 3: Add to INTERACTIONS**

```js
{ id: 'meeting_escape', fn: interactionMeetingEscape },
```

- [ ] **Step 4: Test in browser — enemy tracks cursor, survive bar fills, contact resets it**

Expected: yellow CSS enemy moves toward cursor. Keeping away fills green survive bar. Contact resets it. After 10 cumulative seconds: achievement + interaction ends.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: INT-03 meeting escape enemy interaction"
```

---

## Task 10: INT-04 — Rupia Rain

**Files:**
- Modify: `index.html` — add interactionRupiaRain()

- [ ] **Step 1: Add rupia CSS**

```css
.rupia {
  position: absolute;
  width: 20px;
  height: 20px;
  transform: rotate(45deg);
  cursor: pointer;
  pointer-events: auto;
  transition: top linear;
}
.rupia.gold { background: #c8a200; box-shadow: 2px 2px 0 #000; }
.rupia.green { background: #00a020; box-shadow: 2px 2px 0 #000; }

.rupia-counter { font-family: 'Press Start 2P', monospace; font-size: 11px; color: var(--gold); }
.rupia-timer { font-family: 'Press Start 2P', monospace; font-size: 9px; color: var(--beige); margin-top: 6px; }
```

- [ ] **Step 2: Implement interactionRupiaRain()**

```js
function interactionRupiaRain() {
  GameState.currentInteraction = 'rupia_rain';
  const area = document.getElementById('interaction-area');
  const title = document.getElementById('interaction-title');
  title.textContent = 'COLLECT RUPIAS';

  area.innerHTML = '';
  area.style.position = 'relative';
  area.style.overflow = 'hidden';

  const counter = document.createElement('div');
  counter.className = 'rupia-counter';
  counter.style.cssText = 'position:absolute;top:8px;left:50%;transform:translateX(-50%);';
  counter.textContent = 'GOLD: 0 / 10';

  const timerEl = document.createElement('div');
  timerEl.className = 'rupia-timer';
  timerEl.style.cssText = 'position:absolute;top:28px;left:50%;transform:translateX(-50%);';
  timerEl.textContent = 'TIME: 20';

  area.appendChild(counter);
  area.appendChild(timerEl);

  let goldCaught = 0;
  let timeLeft = 20;
  let done = false;
  const rupias = [];

  function spawnRupia() {
    if (done) return;
    const rect = area.getBoundingClientRect();
    const isGold = Math.random() > 0.4; // 60% gold, 40% green
    const r = document.createElement('div');
    r.className = 'rupia ' + (isGold ? 'gold' : 'green');
    const x = Math.random() * (rect.width - 30) + 5;
    r.style.left = x + 'px';
    r.style.top = '-20px';
    area.appendChild(r);
    rupias.push(r);

    const duration = 3000 + Math.random() * 2000;
    r.style.transition = 'top ' + (duration/1000) + 's linear';
    requestAnimationFrame(() => {
      r.style.top = (rect.height + 20) + 'px';
    });

    r.addEventListener('click', () => {
      if (done || !area.contains(r)) return;
      playSound('click');
      area.removeChild(r);
      if (isGold) {
        goldCaught++;
        counter.textContent = 'GOLD: ' + goldCaught + ' / 10';
        if (goldCaught >= 10) {
          finish('win');
        }
      } else {
        goldCaught = 0;
        counter.textContent = 'GOLD: 0 / 10 (RESET!)';
        setTimeout(() => {
          if (!done) counter.textContent = 'GOLD: 0 / 10';
        }, 800);
      }
    });

    // Remove rupia after it falls off screen
    setTimeout(() => {
      if (area.contains(r)) area.removeChild(r);
    }, duration + 100);
  }

  function finish(reason) {
    if (done) return;
    done = true;
    clearInterval(spawnInterval);
    clearInterval(timerInterval);
    completeInteraction(-7, reason === 'win' ? 'WALLET FULL, PRODUCTIVITY EMPTY' : 'DROPPED EVERYTHING');
  }

  const spawnInterval = setInterval(spawnRupia, 800);
  spawnRupia(); // spawn immediately

  const timerInterval = setInterval(() => {
    timeLeft--;
    timerEl.textContent = 'TIME: ' + timeLeft;
    if (timeLeft <= 0) finish('timeout');
  }, 1000);

  GameState.cleanupFn = () => {
    done = true;
    clearInterval(spawnInterval);
    clearInterval(timerInterval);
    area.innerHTML = '';
    area.style.overflow = '';
    title.textContent = '';
  };
}
```

- [ ] **Step 3: Add to INTERACTIONS**

```js
{ id: 'rupia_rain', fn: interactionRupiaRain },
```

- [ ] **Step 4: Test — rupias fall, gold increments counter, green resets, win at 10, timer ends at 0**

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: INT-04 rupia rain interaction"
```

---

## Task 11: INT-05 — Eternal Standup

**Files:**
- Modify: `index.html` — add interactionEternalStandup()

- [ ] **Step 1: Implement interactionEternalStandup()**

```js
function interactionEternalStandup() {
  GameState.currentInteraction = 'eternal_standup';
  GameState.passiveDrainPaused = true;
  const area = document.getElementById('interaction-area');
  const title = document.getElementById('interaction-title');
  title.textContent = 'DAILY STANDUP';

  area.innerHTML = '';

  const timerDisplay = document.createElement('div');
  timerDisplay.className = 'int-counter';
  timerDisplay.style.fontSize = '28px';
  timerDisplay.textContent = '0:30';

  const statusText = document.createElement('div');
  statusText.style.cssText = 'font-family:"Press Start 2P",monospace;font-size:9px;color:var(--beige);text-align:center;min-height:24px;';

  const leaveBtn = document.createElement('button');
  leaveBtn.className = 'int-btn';
  leaveBtn.textContent = 'LEAVE MEETING';
  leaveBtn.style.display = 'none';

  area.appendChild(timerDisplay);
  area.appendChild(statusText);
  area.appendChild(leaveBtn);

  let seconds = 30;
  let resets = 0;
  let done = false;

  const interval = setInterval(() => {
    if (done) return;
    seconds--;
    const m = Math.floor(seconds / 60);
    const s = seconds % 60;
    timerDisplay.textContent = m + ':' + s.toString().padStart(2, '0');

    if (seconds <= 0) {
      resets++;
      statusText.textContent = 'RESCHEDULED TO TOMORROW';
      timerDisplay.textContent = '⏰';
      setTimeout(() => {
        if (done) return;
        statusText.textContent = '';
        if (resets >= 3) {
          leaveBtn.style.display = 'inline-block';
          timerDisplay.textContent = 'WAITING...';
          clearInterval(interval);
        } else {
          seconds = 30;
          timerDisplay.textContent = '0:30';
        }
      }, 1000);
    }
  }, 1000);

  leaveBtn.addEventListener('click', () => {
    if (done) return;
    done = true;
    playSound('click');
    completeInteraction(-6, 'FREEDOM ACHIEVED (TEMPORARILY)');
  });

  GameState.cleanupFn = () => {
    done = true;
    clearInterval(interval);
    GameState.passiveDrainPaused = false;
    area.innerHTML = '';
    title.textContent = '';
  };
}
```

- [ ] **Step 2: Add to INTERACTIONS**

```js
{ id: 'eternal_standup', fn: interactionEternalStandup },
```

- [ ] **Step 3: Test — countdown resets 3 times, then LEAVE button appears and works**

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: INT-05 eternal standup interaction (passive drain paused)"
```

---

## Task 12: INT-06 — Progress Bar

**Files:**
- Modify: `index.html` — add interactionProgressBar()

- [ ] **Step 1: Implement interactionProgressBar()**

```js
function interactionProgressBar() {
  GameState.currentInteraction = 'progress_bar';
  const area = document.getElementById('interaction-area');
  const title = document.getElementById('interaction-title');
  title.textContent = 'UPLOADING TPS REPORT';

  area.innerHTML = '';

  const barLabel = document.createElement('div');
  barLabel.style.cssText = 'font-family:"Press Start 2P",monospace;font-size:9px;color:var(--beige);margin-bottom:8px;';
  barLabel.textContent = 'ATTEMPT 1 / 4';

  const barContainer = document.createElement('div');
  barContainer.style.cssText = 'width:280px;height:22px;border:3px solid var(--gold);background:#000;';

  const barFill = document.createElement('div');
  barFill.style.cssText = 'height:100%;width:0%;background:var(--gold);transition:none;';

  barContainer.appendChild(barFill);

  const statusText = document.createElement('div');
  statusText.style.cssText = 'font-family:"Press Start 2P",monospace;font-size:8px;color:var(--red-dark);margin-top:8px;min-height:20px;';

  const tooltip = document.createElement('div');
  tooltip.style.cssText = 'font-family:"Press Start 2P",monospace;font-size:8px;color:var(--gold);margin-top:12px;display:none;';
  tooltip.textContent = 'DO NOT MOVE THE MOUSE';

  area.appendChild(barLabel);
  area.appendChild(barContainer);
  area.appendChild(statusText);
  area.appendChild(tooltip);

  let attempt = 0;
  let progress = 0;
  let done = false;
  let fillInterval = null;
  let stillnessTimeout = null;
  let mouseStill = false;

  function startAttempt() {
    attempt++;
    progress = 0;
    barFill.style.width = '0%';
    barLabel.textContent = 'ATTEMPT ' + attempt + ' / 4';
    statusText.textContent = '';

    if (attempt <= 3) {
      // Regular fill: 25% per second, cap at 99%
      fillInterval = setInterval(() => {
        if (done) return;
        progress += 2.5; // 25% per second at 100ms intervals
        if (progress >= 99) {
          progress = 99;
          barFill.style.width = '99%';
          clearInterval(fillInterval);
          statusText.textContent = 'CONNECTION LOST';
          setTimeout(() => { if (!done) startAttempt(); }, 1000);
        } else {
          barFill.style.width = progress + '%';
        }
      }, 100);
    } else {
      // Attempt 4: mouse must be still
      tooltip.style.display = 'block';
      mouseStill = false;

      function onMouseMove() {
        progress = 0;
        barFill.style.width = '0%';
        mouseStill = false;
        clearInterval(fillInterval);
        fillInterval = null;
        if (stillnessTimeout) clearTimeout(stillnessTimeout);
        stillnessTimeout = setTimeout(() => {
          if (done) return;
          mouseStill = true;
          startFilling();
        }, 500);
      }

      function startFilling() {
        if (fillInterval) return;
        fillInterval = setInterval(() => {
          if (!mouseStill || done) return;
          progress += 1; // 10% per second at 100ms
          barFill.style.width = progress + '%';
          if (progress >= 100) {
            clearInterval(fillInterval);
            document.removeEventListener('mousemove', onMouseMove);
            done = true;
            completeInteraction(-9, "SCHRODINGER'S UPLOAD: COMPLETE");
          }
        }, 100);
      }

      document.addEventListener('mousemove', onMouseMove);

      // Override cleanupFn to also remove this listener
      const prevCleanup = GameState.cleanupFn;
      GameState.cleanupFn = () => {
        document.removeEventListener('mousemove', onMouseMove);
        if (stillnessTimeout) clearTimeout(stillnessTimeout);
        if (fillInterval) clearInterval(fillInterval);
        area.innerHTML = '';
        title.textContent = '';
      };
    }
  }

  startAttempt();

  GameState.cleanupFn = () => {
    done = true;
    if (fillInterval) clearInterval(fillInterval);
    if (stillnessTimeout) clearTimeout(stillnessTimeout);
    area.innerHTML = '';
    title.textContent = '';
  };
}
```

- [ ] **Step 2: Add to INTERACTIONS**

```js
{ id: 'progress_bar', fn: interactionProgressBar },
```

- [ ] **Step 3: Test — bar fills to 99% three times, 4th attempt requires mouse stillness**

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: INT-06 progress bar interaction with mouse-stillness mechanic"
```

---

## Task 13: INT-07 — Inbox Zero

**Files:**
- Modify: `index.html` — add interactionInboxZero()

- [ ] **Step 1: Implement interactionInboxZero()**

```js
function interactionInboxZero() {
  GameState.currentInteraction = 'inbox_zero';
  const area = document.getElementById('interaction-area');
  const title = document.getElementById('interaction-title');
  title.textContent = 'INBOX';

  area.innerHTML = '';
  area.style.position = 'relative';

  const counterEl = document.createElement('div');
  counterEl.style.cssText = 'font-family:"Press Start 2P",monospace;font-size:9px;color:var(--gold);position:absolute;top:4px;left:50%;transform:translateX(-50%);';
  counterEl.textContent = 'DELETED: 0';
  area.appendChild(counterEl);

  let totalDeleted = 0;
  let visibleEmails = [];
  let spawnQueue = 0;
  let done = false;
  let dequeueInterval = null;
  let forceTimeout = null;

  function createEmail() {
    if (done || visibleEmails.length >= 8) {
      spawnQueue++;
      return;
    }
    const email = document.createElement('div');
    email.style.cssText = `
      position:absolute;
      font-size:24px;
      cursor:pointer;
      user-select:none;
      left:${10 + Math.random() * 80}%;
      top:${20 + Math.random() * 60}%;
      transform:translate(-50%,-50%);
      font-family:monospace;
    `;
    email.textContent = '📧';
    area.appendChild(email);
    visibleEmails.push(email);

    email.addEventListener('click', () => {
      if (done || !area.contains(email)) return;
      playSound('click');
      area.removeChild(email);
      visibleEmails = visibleEmails.filter(e => e !== email);
      spawnQueue += 3;
      totalDeleted++;
      counterEl.textContent = 'DELETED: ' + totalDeleted;

      if (totalDeleted === 20 && !done) {
        showAchievement('INBOX_HERO');
      }
    });
  }

  // Seed with 5 emails
  for (let i = 0; i < 5; i++) createEmail();

  // Dequeue spawner
  dequeueInterval = setInterval(() => {
    if (done) return;
    while (spawnQueue > 0 && visibleEmails.length < 8) {
      spawnQueue--;
      createEmail();
    }
  }, 300);

  // Force-complete at 25s
  forceTimeout = setTimeout(() => {
    if (!done) {
      done = true;
      completeInteraction(-11, 'INBOX: INFINITY');
    }
  }, 25000);

  GameState.cleanupFn = () => {
    done = true;
    clearInterval(dequeueInterval);
    clearTimeout(forceTimeout);
    area.innerHTML = '';
    title.textContent = '';
    visibleEmails = [];
  };
}
```

- [ ] **Step 2: Add to INTERACTIONS**

```js
{ id: 'inbox_zero', fn: interactionInboxZero },
```

- [ ] **Step 3: Test — emails appear, clicking deletes and spawns 3 more, capped at 8, force-completes at 25s**

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: INT-07 inbox zero quest interaction"
```

---

## Task 14: INT-08 — Task Labyrinth

**Files:**
- Modify: `index.html` — add interactionTaskLabyrinth()

- [ ] **Step 1: Add maze CSS**

```css
.maze-grid {
  display: grid;
  grid-template-columns: repeat(5, 40px);
  grid-template-rows: repeat(5, 40px);
  gap: 2px;
  background: #000;
  border: 3px solid var(--gold);
}

.maze-cell {
  width: 40px;
  height: 40px;
  background: var(--green-dark);
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 10px;
}

.maze-cell.wall { background: #0a2a0a; }
.maze-cell.exit { background: #1a1a5a; }
.maze-cell.exit::after { content: '🚪'; font-size: 16px; }

.maze-player {
  width: 24px;
  height: 24px;
  background: #00cc00;
  border: 2px solid #00ff00;
  border-radius: 0;
  position: absolute;
  pointer-events: none;
}
```

- [ ] **Step 2: Implement interactionTaskLabyrinth()**

```js
function interactionTaskLabyrinth() {
  GameState.currentInteraction = 'task_labyrinth';
  const area = document.getElementById('interaction-area');
  const title = document.getElementById('interaction-title');
  title.textContent = 'NAVIGATE REQUIREMENTS';

  area.innerHTML = '';

  // 4 hardcoded 5x5 maze layouts: 1=wall, 0=open, E=exit position
  // Format: [row][col], walls as arrays of [r,c]
  const mazes = [
    { walls: [[0,1],[1,1],[1,3],[2,0],[2,3],[3,2],[3,3],[4,2]], exit: [4,4] },
    { walls: [[0,2],[0,3],[1,0],[1,2],[2,1],[2,4],[3,0],[3,3]], exit: [4,0] },
    { walls: [[0,4],[1,1],[1,4],[2,2],[2,3],[3,1],[4,1],[4,3]], exit: [0,4] },
    { walls: [[0,1],[1,3],[2,0],[2,4],[3,2],[3,4],[4,0],[4,2]], exit: [2,4] }
  ];

  let mazeIndex = 0;
  let exitCount = 0;
  let playerR = 0, playerC = 0;
  let done = false;
  let msgTimeout = null;

  function isWall(r, c, maze) {
    return maze.walls.some(w => w[0] === r && w[1] === c);
  }

  function renderMaze() {
    area.innerHTML = '';
    const maze = mazes[mazeIndex % 4];

    const progressEl = document.createElement('div');
    progressEl.style.cssText = 'font-family:"Press Start 2P",monospace;font-size:8px;color:var(--beige);margin-bottom:8px;';
    progressEl.textContent = 'WRONG EXITS FOUND: ' + exitCount + ' / 4';
    area.appendChild(progressEl);

    const grid = document.createElement('div');
    grid.className = 'maze-grid';

    for (let r = 0; r < 5; r++) {
      for (let c = 0; c < 5; c++) {
        const cell = document.createElement('div');
        const isW = isWall(r, c, maze);
        const isExit = maze.exit[0] === r && maze.exit[1] === c;
        cell.className = 'maze-cell' + (isW ? ' wall' : '') + (isExit ? ' exit' : '');
        cell.dataset.r = r;
        cell.dataset.c = c;

        if (r === playerR && c === playerC) {
          const player = document.createElement('div');
          player.className = 'maze-player';
          cell.appendChild(player);
        }
        grid.appendChild(cell);
      }
    }

    const hint = document.createElement('div');
    hint.style.cssText = 'font-family:"Press Start 2P",monospace;font-size:7px;color:#888;margin-top:8px;';
    hint.textContent = 'WASD / ARROWS TO MOVE';
    area.appendChild(grid);
    area.appendChild(hint);
  }

  function onKey(e) {
    if (done) return;
    const moves = {
      ArrowUp: [-1, 0], ArrowDown: [1, 0], ArrowLeft: [0, -1], ArrowRight: [0, 1],
      w: [-1, 0], s: [1, 0], a: [0, -1], d: [0, 1]
    };
    const move = moves[e.key];
    if (!move) return;
    e.preventDefault();

    const maze = mazes[mazeIndex % 4];
    const nr = playerR + move[0];
    const nc = playerC + move[1];

    if (nr < 0 || nr > 4 || nc < 0 || nc > 4) return;
    if (isWall(nr, nc, maze)) return;

    playerR = nr;
    playerC = nc;
    renderMaze();

    // Check exit
    if (maze.exit[0] === playerR && maze.exit[1] === playerC) {
      exitCount++;
      if (exitCount >= 4) {
        done = true;
        document.removeEventListener('keydown', onKey);
        completeInteraction(-13, 'REQUIREMENTS UPDATED. START OVER.');
        return;
      }
      // Show wrong exit message
      const msgEl = document.createElement('div');
      msgEl.style.cssText = 'position:absolute;top:0;left:0;width:100%;background:var(--red-dark);font-family:"Press Start 2P",monospace;font-size:8px;color:var(--beige);padding:8px;text-align:center;z-index:20;';
      msgEl.textContent = 'WRONG EXIT - SEE ATTACHED: requirements_FINAL_v3.docx';
      area.appendChild(msgEl);

      mazeIndex++;
      playerR = 0;
      playerC = 0;

      msgTimeout = setTimeout(() => {
        if (!done) renderMaze();
      }, 1000);
    }
  }

  document.addEventListener('keydown', onKey);
  renderMaze();

  GameState.cleanupFn = () => {
    done = true;
    document.removeEventListener('keydown', onKey);
    if (msgTimeout) clearTimeout(msgTimeout);
    area.innerHTML = '';
    title.textContent = '';
  };
}
```

- [ ] **Step 3: Add to INTERACTIONS**

```js
{ id: 'task_labyrinth', fn: interactionTaskLabyrinth },
```

- [ ] **Step 4: Test — maze renders, WASD moves player, exit shows wrong-exit message, 4 exits completes interaction**

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: INT-08 task labyrinth interaction"
```

---

## Task 15: Achievements System

**Files:**
- Modify: `index.html` — replace checkAchievements and showAchievement stubs with full implementation

- [ ] **Step 1: Add achievement overlay CSS**

```css
/* ===== ACHIEVEMENT OVERLAY ===== */
#achievement-overlay {
  display: flex;
  align-items: center;
  justify-content: center;
}

.achievement-popup {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  background: var(--black);
  border: 4px solid var(--gold);
  box-shadow: 0 0 0 8px #000, 0 0 0 12px var(--gold);
  padding: 20px 32px;
  font-family: 'Press Start 2P', monospace;
}

.chest-container {
  position: relative;
  width: 40px;
  height: 40px;
}

.chest-body {
  position: absolute;
  bottom: 0; left: 0;
  width: 40px; height: 22px;
  background: #8b4513;
  border: 3px solid var(--gold);
}

.chest-lid {
  position: absolute;
  top: 0; left: 0;
  width: 40px; height: 18px;
  background: #a0522d;
  border: 3px solid var(--gold);
  transform-origin: bottom center;
  transform: rotateX(0deg);
  transition: transform 0.4s ease-out;
}

.chest-lid.open { transform: rotateX(-60deg); }

.chest-star {
  position: absolute;
  top: -10px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 20px;
  opacity: 0;
}

@keyframes star-rise {
  0%   { opacity: 0; transform: translateX(-50%) translateY(0); }
  30%  { opacity: 1; }
  100% { opacity: 0; transform: translateX(-50%) translateY(-50px); }
}

.chest-star.rising { animation: star-rise 1.2s ease-out forwards; }

.achievement-label {
  font-size: 8px;
  color: #888;
  text-transform: uppercase;
}

.achievement-name {
  font-size: 10px;
  color: var(--gold);
  text-align: center;
}
```

- [ ] **Step 2: Replace checkAchievements and showAchievement stubs**

```js
const ACHIEVEMENT_DEFS = {
  CLICKED_WITHOUT_PURPOSE: 'CLICKED WITHOUT PURPOSE',
  MEETING_SURVIVOR: 'MEETING SURVIVOR',
  PROFESSIONAL_PROCRASTINATOR: 'PROFESSIONAL PROCRASTINATOR',
  THE_FLOOR_IS_DEADLINES: 'THE FLOOR IS DEADLINES',
  INBOX_HERO: 'INBOX HERO',
  SPEEDRUNNER: 'SPEEDRUNNER'
};

function checkAchievements() {
  const gs = GameState;
  if (!gs.achievements.has('PROFESSIONAL_PROCRASTINATOR') && gs.interactionCount >= 3) {
    showAchievement('PROFESSIONAL_PROCRASTINATOR');
  }
  if (!gs.achievements.has('THE_FLOOR_IS_DEADLINES') && gs.focusPoints < 50 && gs.focusPoints > 0) {
    showAchievement('THE_FLOOR_IS_DEADLINES');
  }
  if (!gs.achievements.has('SPEEDRUNNER') && gs.interactionCount >= 5 && gs.timeWasted < 180) {
    showAchievement('SPEEDRUNNER');
  }
}

let achievementQueue = [];
let achievementShowing = false;

function showAchievement(id) {
  if (GameState.achievements.has(id)) return;
  GameState.achievements.add(id);
  achievementQueue.push(id);
  if (!achievementShowing) processAchievementQueue();
}

function processAchievementQueue() {
  if (achievementQueue.length === 0) { achievementShowing = false; return; }
  achievementShowing = true;
  const id = achievementQueue.shift();
  const name = ACHIEVEMENT_DEFS[id] || id;

  const overlay = document.getElementById('achievement-overlay');
  overlay.style.display = 'flex';
  overlay.innerHTML = '';

  const popup = document.createElement('div');
  popup.className = 'achievement-popup';

  const chestContainer = document.createElement('div');
  chestContainer.className = 'chest-container';

  const body = document.createElement('div');
  body.className = 'chest-body';
  const lid = document.createElement('div');
  lid.className = 'chest-lid';
  const star = document.createElement('div');
  star.className = 'chest-star';
  star.textContent = '★';

  chestContainer.appendChild(body);
  chestContainer.appendChild(lid);
  chestContainer.appendChild(star);

  const label = document.createElement('div');
  label.className = 'achievement-label';
  label.textContent = 'YOU GOT:';

  const nameEl = document.createElement('div');
  nameEl.className = 'achievement-name';
  nameEl.textContent = name;

  popup.appendChild(chestContainer);
  popup.appendChild(label);
  popup.appendChild(nameEl);
  overlay.appendChild(popup);

  playSound('achievement');

  // Open lid + rise star
  setTimeout(() => { lid.classList.add('open'); }, 50);
  setTimeout(() => { star.classList.add('rising'); }, 200);

  // Auto-dismiss after 3s
  setTimeout(() => {
    overlay.style.display = 'none';
    overlay.innerHTML = '';
    setTimeout(processAchievementQueue, 300);
  }, 3000);
}
```

- [ ] **Step 3: Test achievements — manually call `showAchievement('SPEEDRUNNER')` in console**

Expected: achievement overlay appears with CSS chest, lid opens, star rises, auto-dismisses after 3s. Arpeggio sound plays.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: achievements system with Zelda chest CSS animation"
```

---

## Task 16: Visual Polish

**Files:**
- Modify: `index.html` — pixel border utilities, HUD tint states, interaction title styling

- [ ] **Step 1: Add HUD color tint transitions based on focus level**

Add to CSS:
```css
/* HUD tint based on focus */
.game-hud.focus-low   { border-bottom-color: #cc6600; }
.game-hud.focus-crit  { border-bottom-color: var(--red-dark); }
```

Add to updateFocusDisplay():
```js
const hud = document.querySelector('.game-hud');
hud.classList.remove('focus-low', 'focus-crit');
if (fp <= 20) hud.classList.add('focus-crit');
else if (fp <= 40) hud.classList.add('focus-low');
```

- [ ] **Step 2: Style dialog-title with text-shadow pixel effect**

Add to CSS:
```css
.dialog-title {
  text-shadow: 2px 2px 0 #000, -1px -1px 0 #000;
  letter-spacing: 2px;
}
```

- [ ] **Step 3: Add game screen body scroll prevention and cursor style**

```css
body { cursor: default; }
#screen-game { cursor: crosshair; }
```

- [ ] **Step 4: Full playthrough test**

Open in browser. Play through all 8 interactions by cycling through them. Verify:
- [ ] Corporate glitch transition works
- [ ] Loading sequence plays correctly
- [ ] HUD brain decay shows correctly as focus drops
- [ ] Timer counts up
- [ ] Message bar typewriter effect works after each interaction
- [ ] Achievements appear at correct triggers
- [ ] Game over screen shows correct stats when focus hits 0
- [ ] Play again resets to corporate screen

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: visual polish - HUD tints, pixel text shadows, cursor"
```

---

## Task 17: Final Integration + README

**Files:**
- Modify: `index.html` — any final bugs from playthrough
- Create: `README.md`

- [ ] **Step 1: Fix any bugs found during full playthrough**

Common things to check:
- INT-03 survive timer doesn't double-count on fast frames
- INT-06 cleanupFn properly reassigned on attempt 4 (mousemove listener removal)
- Achievement queue doesn't stack on rapid completions
- Game over properly stops all intervals when called mid-interaction

- [ ] **Step 2: Create minimal README**

```markdown
# Anti-Productivity Simulator

A browser game that pretends to be a productivity tool.

## Play

Open `index.html` in any modern browser. No server needed.

## Add a new interaction

1. Write `function interactionMyNew()` — set `GameState.cleanupFn` first
2. Call `completeInteraction(penalty, message)` when done
3. Add `{ id: 'my_id', fn: interactionMyNew }` to the `INTERACTIONS` array
```

- [ ] **Step 3: Final commit**

```bash
git add index.html README.md
git commit -m "feat: complete Anti-Productivity Simulator MVP"
```

---

## Summary

| Task | Component | Effort |
|------|-----------|--------|
| 1 | HTML Skeleton + CSS vars | ~15min |
| 2 | Corporate screen + glitch | ~15min |
| 3 | Loading screen sequence | ~15min |
| 4 | Game screen + HUD | ~20min |
| 5 | Core engine (state, drain, game over) | ~25min |
| 6 | Sound system | ~10min |
| 7 | INT-01 Click Quest | ~15min |
| 8 | INT-02 Elusive Button | ~15min |
| 9 | INT-03 Meeting Escape | ~20min |
| 10 | INT-04 Rupia Rain | ~15min |
| 11 | INT-05 Eternal Standup | ~10min |
| 12 | INT-06 Progress Bar | ~15min |
| 13 | INT-07 Inbox Zero | ~15min |
| 14 | INT-08 Task Labyrinth | ~20min |
| 15 | Achievements system | ~20min |
| 16 | Visual Polish | ~15min |
| 17 | Final integration + README | ~15min |
