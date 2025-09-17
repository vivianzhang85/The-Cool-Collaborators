---
layout: opencs
title: Connect 4
permalink: /c4
---

<!--
  CONNECT 4 — with challenge features integrated and FULLY COMMENTED

  Where to look:
  - (1) THEME SWITCHER: Start screen buttons ".btn.theme" + CSS :root[data-theme="..."]
  - (2) COLUMN HOVER HIGHLIGHT: CSS ".hole.preview" + JS highlightColumn/clearHighlight + mouseover/mouseleave listeners
  - (3) DROP SOUND: tiny WebAudio function playDropSound() called only after valid piece placement
  - (4) RESTART CONFIRM: confirm() in restart() before resetting
  - (5) WIN BANNER EMOJI + GLOW: GameUI.showWinMessage(message, color) adds emoji & glow class
  - BONUS) KEYBOARD CONTROLS: ←/→ to move preview/ghost coin; Enter to drop (selectedCol/previewCol state + ghost overlay)
-->

<div id="app" class="wrap">
  <!-- Start Screen (UNCHANGED UI + added Theme buttons) -->
  <section id="start" class="card center">
    <h1 class="title">Connect 4</h1>
    <p class="muted">Choose a timer per player</p>

    <!-- Timer preset buttons — these start the game with chosen time -->
    <div class="row">
      <button class="btn" data-time="180">3 Minutes</button>
      <button class="btn" data-time="300">5 Minutes</button>
      <button class="btn" data-time="600">10 Minutes</button>
    </div>

    <!-- (1) THEME SWITCHER — purely cosmetic, doesn’t affect layout/logic -->
    <div class="row" style="margin-top:10px">
      <span class="muted">Theme:</span>
      <button class="btn theme" data-theme="classic" title="Light classic">Classic</button>
      <button class="btn theme" data-theme="midnight" title="Dark midnight">Midnight</button>
    </div>

    <!-- Keyboard shortcut to start with the default time -->
    <p class="press">…or press <kbd>Enter</kbd> to start with 5:00</p>
  </section>

  <!-- Game Screen (overlay so it won’t disturb your homepage layout) -->
  <section id="game" class="hidden game-overlay">
    <div class="hud">
      <!-- Left player panel -->
      <div class="panel red-side">
        <h2>Red</h2>
        <div class="timer" id="tRed">05:00</div>
        <div class="stash">
          <div class="dot red"></div>
          <span>Coins: <b id="cRed">21</b></span>
        </div>
      </div>

      <!-- Board wrapper (blue frame) -->
      <div id="boardWrap">
        <!-- Grid holes are generated dynamically in JS -->
        <div id="board"></div>

        <!-- BONUS) Keyboard "ghost coin" overlay (visual column pointer on top) -->
        <div id="ghost" class="coin ghost hidden"></div>

        <!-- Falling coin overlay (re-usable DOM element for the drop animation) -->
        <div id="fall" class="coin hidden"></div>
      </div>

      <!-- Right player panel -->
      <div class="panel yellow-side">
        <h2>Yellow</h2>
        <div class="timer" id="tYellow">05:00</div>
        <div class="stash">
          <div class="dot yellow"></div>
          <span>Coins: <b id="cYellow">21</b></span>
        </div>
      </div>
    </div>

    <!-- Restart button (4) Wrapped with confirm() in JS -->
    <br><br><br>
    <div class="row center">
      <button id="restart" class="btn danger">Restart</button>
    </div>
  </section>
</div>

<!-- ========= Styles ========= -->
<style>
  :root{
    /* Default to the Midnight theme colors */
    --bg:#0f0f10;
    --card:#17181c;
    --fg:#ffffff;
    --muted:#a9b0be;
    --blue:#1658e5;
    --red:#ef4444;
    --yellow:#facc15;

    /* Board dimensions + layout vars */
    --cell:76px;      /* size of a circular hole */
    --gap:12px;       /* spacing between holes */
    --radius:50%;
    --boardPad:18px;  /* blue frame padding */
    --boardCols:7;
    --boardRows:6;
  }

  /* (1) THEME SWITCHER — Classic theme overrides by attribute on <html> */
  :root[data-theme="classic"]{
    --bg:#f3f4f6;
    --card:#ffffff;
    --fg:#0f172a;
    --muted:#475569;
    --blue:#1d4ed8;
    --red:#dc2626;
    --yellow:#f59e0b;
  }

  /* Page container */
  *{box-sizing:border-box}
  .wrap{
    min-height:100vh; display:flex; align-items:center; justify-content:center;
    background:var(--bg); color:var(--fg); font-family:system-ui,Segoe UI,Roboto,Inter,Arial;
    padding:24px;
    position: relative; /* ensures overlay positions correctly but doesn’t affect homepage layout */
  }

  /* Utility classes */
  .hidden{display:none}
  .center{justify-content:center; text-align:center}
  .row{display:flex; gap:12px; align-items:center; flex-wrap:wrap}

  /* Card styling (start screen) */
  .card{
    background:var(--card); padding:28px; border-radius:18px; box-shadow:0 10px 30px #0006; width:min(720px,95vw);
  }
  .title{font-size:42px; margin:0 0 8px}
  .muted{color:var(--muted); margin:0 0 18px}
  .press{color:var(--muted); margin-top:14px; font-size:14px}
  kbd{background:#222; padding:2px 6px; border-radius:6px; border:1px solid #333; color:#fff}

  /* Buttons */
  .btn{
    background:#2b2f3a; border:1px solid #3a4151; color:#fff;
    padding:10px 16px; border-radius:12px; cursor:pointer; font-weight:600;
    transition:transform .06s ease, background .2s, border .2s, color .2s;
  }
  :root[data-theme="classic"] .btn{ background:#e5e7eb; border-color:#cbd5e1; color:#0f172a; }
  .btn:hover{transform:translateY(-1px); background:#323849}
  :root[data-theme="classic"] .btn:hover{ background:#dfe3ea; }
  .btn:active{transform:translateY(0)}
  .btn.danger{background:#b71c1c; border-color:#c72a2a}
  .btn.danger:hover{background:#d32626}

  /* GAME OVERLAY — fixed full-screen to avoid homepage shifts */
  .game-overlay{
    position: fixed; /* sits above homepage */
    top: 0; left: 0; width: 100vw; height: 100vh;
    background: var(--bg); z-index: 1000;
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    padding: 24px;
  }
  .game-overlay.hidden{ display: none; }

  /* Panels + HUD */
  .hud{display:grid; grid-template-columns:1fr auto 1fr; gap:18px; align-items:center}
  .panel{
    background:#14161b; border:1px solid #272c38; border-radius:16px; padding:16px;
    display:flex; flex-direction:column; align-items:center; gap:10px; min-width:160px;
  }
  :root[data-theme="classic"] .panel{ background:#f8fafc; border-color:#e5e7eb; }
  .panel h2{margin:0; letter-spacing:.5px}
  .red-side h2{color:var(--red)} .yellow-side h2{color:var(--yellow)}
  .timer{font-size:38px; font-variant-numeric:tabular-nums}
  .stash{display:flex; align-items:center; gap:10px; color:var(--muted)}
  .dot{width:18px; height:18px; border-radius:50%}
  .red{background:var(--red)} .yellow{background:var(--yellow)}

  /* Blue frame + board grid */
  #boardWrap{
    position:relative; padding:var(--boardPad); background:var(--blue);
    border-radius:22px; box-shadow:inset 0 0 0 6px #0e3ea3, 0 12px 24px #0007
  }
  #board{
    display:grid; gap:var(--gap);
    grid-template-columns:repeat(var(--boardCols), var(--cell));
    grid-template-rows:repeat(var(--boardRows), var(--cell));
  }

  /* Holes (empty) + filled states */
  #board .hole{
    width:var(--cell); height:var(--cell); border-radius:var(--radius);
    background:#f1f5f9; position:relative; overflow:hidden;
    box-shadow:inset 0 0 0 6px #0e3ea3;
    cursor:pointer; transition:filter .2s;
  }
  #board .hole:hover{filter:brightness(0.95)}
  #board .hole.filled::after{ content:""; position:absolute; inset:0; border-radius:var(--radius); }
  #board .hole.red::after{background:var(--red)}
  #board .hole.yellow::after{background:var(--yellow)}

  /* (2) COLUMN HOVER/KEYBOARD HIGHLIGHT — adds a subtle ring to all cells in a column */
  #board .hole.preview::before{
    content:""; position:absolute; inset:0; border-radius:var(--radius);
    box-shadow: inset 0 0 0 4px rgba(255,255,255,.65);
    pointer-events:none;
  }

  /* Reusable coin overlays: falling coin + ghost coin */
  .coin{
    position:absolute; width:var(--cell); height:var(--cell); border-radius:50%;
    left:0; top:0; transform:translate(0,-100%); /* start above the board */
    will-change:transform;
    box-shadow:0 4px 10px #0007, inset 0 -6px 0 #0003;
    transition:transform .28s cubic-bezier(.2,.9,.2,1);
  }
  .coin.red{background:var(--red)}
  .coin.yellow{background:var(--yellow)}

  /* BONUS) Ghost coin styling (semi-transparent, dashed) */
  .coin.ghost{
    opacity:.4; outline:2px dashed rgba(255,255,255,.6); outline-offset:-4px; pointer-events:none
  }

  /* (5) WIN OVERLAY with emoji + glow */
  .win-overlay{
    position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
    background: rgba(0, 0, 0, 0.8); z-index: 2000; display: flex; align-items: center; justify-content: center;
    backdrop-filter: blur(8px); animation: fadeIn 0.3s ease-out;
  }
  .win-card{
    background: var(--card); padding: 40px; border-radius: 20px; text-align: center;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5); transform: scale(0.9);
    animation: popIn 0.4s ease-out 0.1s forwards; border: 2px solid #3a4151;
  }
  .win-title{ font-size: 36px; margin: 0 0 24px; color: var(--fg); }
  .glow-red{ text-shadow: 0 0 14px rgba(239,68,68,.75), 0 0 28px rgba(239,68,68,.35); }
  .glow-yellow{ text-shadow: 0 0 14px rgba(250,204,21,.75), 0 0 28px rgba(250,204,21,.35); }
  .win-btn{ font-size: 18px; padding: 12px 24px; background: var(--blue); border-color: #1658e5; }
  .win-btn:hover{ background: #1d6bf0; }

  /* Small animations */
  @keyframes fadeIn{ from{ opacity: 0; } to{ opacity: 1; } }
  @keyframes popIn{ from{ transform: scale(0.8); opacity: 0; } to{ transform: scale(1); opacity: 1; } }
</style>

<!-- ========= OOP Logic ========= -->
<script>
/* -----------------------------------------------------------
   Utility helpers shared across UI logic
----------------------------------------------------------- */

/** Reads a numeric CSS custom property from :root (e.g., --cell) */
function getCSSNum(name){
  return parseInt(getComputedStyle(document.documentElement).getPropertyValue(name));
}

/** (3) Tiny synth "drop" sound — avoids external audio files.
    Called ONLY after a valid piece is placed (user gesture). */
function playDropSound() {
  try{
    const ctx = new (window.AudioContext || window.webkitAudioContext)();
    const o = ctx.createOscillator();
    const g = ctx.createGain();

    // Simple "plop" using a short down-sweep triangle wave
    o.type = 'triangle';
    o.frequency.value = 420;
    const now = ctx.currentTime;
    o.frequency.exponentialRampToValueAtTime(160, now + 0.12);

    // Quick volume envelope
    g.gain.setValueAtTime(0.001, now);
    g.gain.exponentialRampToValueAtTime(0.25, now + 0.02);
    g.gain.exponentialRampToValueAtTime(0.0001, now + 0.18);

    o.connect(g); g.connect(ctx.destination);
    o.start(); o.stop(now + 0.2);
  }catch(e){
    // If audio context fails (e.g., browser policy), silently ignore
  }
}

/* -----------------------------------------------------------
   Core classes: Player, GameBoard, GameTimer, GameUI
----------------------------------------------------------- */

class Player {
  constructor(name, color) {
    this.name = name;          // Display name
    this.color = color;        // 'red' | 'yellow' for CSS classes
    this.time = 300;           // Default 5 minutes
    this.coins = 21;           // Standard Connect 4 coin count per player
  }
  setTime(seconds) { this.time = seconds; }
  usesCoin() { if (this.coins > 0) { this.coins--; return true; } return false; }
  hasTimeLeft() { return this.time > 0; }
  decrementTime() { if (this.time > 0) this.time--; }
  reset(timeInSeconds) { this.time = timeInSeconds; this.coins = 21; }
}

class GameBoard {
  constructor(rows = 6, cols = 7) {
    this.rows = rows; this.cols = cols;
    this.grid = [];       // 2D array [row][col] storing null | 'red' | 'yellow'
    this.initialize();
  }
  initialize() { this.grid = Array.from({length: this.rows}, () => Array(this.cols).fill(null)); }
  isValidColumn(col) { return col >= 0 && col < this.cols && this.grid[0][col] === null; }
  getDropRow(col) {
    if (!this.isValidColumn(col)) return -1;
    for (let row = this.rows - 1; row >= 0; row--) if (this.grid[row][col] === null) return row;
    return -1;
  }
  placePiece(row, col, color) {
    if (row >= 0 && row < this.rows && col >= 0 && col < this.cols) { this.grid[row][col] = color; return true; }
    return false;
  }
  /* Checks 4-in-a-row from the last placed cell (row,col) in 4 directions */
  checkWin(row, col) {
    const color = this.grid[row][col]; if (!color) return false;
    const dirs = [[1,0],[0,1],[1,1],[1,-1]]; // vertical, horizontal, two diagonals
    for (const [dr,dc] of dirs) {
      let count = 1;
      for (const s of [-1,1]) { // extend both ways
        let r = row + dr*s, c = col + dc*s;
        while (r>=0 && r<this.rows && c>=0 && c<this.cols && this.grid[r][c] === color) {
          count++; r += dr*s; c += dc*s;
        }
      }
      if (count >= 4) return true;
    }
    return false;
  }
  isFull() { return this.grid.every(row => row.every(cell => cell !== null)); }
  reset() { this.initialize(); }
}

class GameTimer {
  constructor(){
    this.intervalId = null;    // setInterval handle
    this.onTick = null;        // callback each second
    this.onTimeUp = null;      // unused here, winner decided in handleTimerTick
  }
  start(tickCallback, timeUpCallback){
    this.onTick = tickCallback; this.onTimeUp = timeUpCallback;
    this.intervalId = setInterval(() => { if (this.onTick) this.onTick(); }, 1000);
  }
  stop(){ if (this.intervalId){ clearInterval(this.intervalId); this.intervalId = null; } }
  reset(){ this.stop(); }
}

class GameUI {
  constructor() {
    // Cache important elements for quick access
    this.elements = {
      start: document.getElementById('start'),
      game: document.getElementById('game'),
      boardWrap: document.getElementById('boardWrap'),
      board: document.getElementById('board'),
      fallCoin: document.getElementById('fall'),
      ghostCoin: document.getElementById('ghost'),
      redTimer: document.getElementById('tRed'),
      yellowTimer: document.getElementById('tYellow'),
      redCoins: document.getElementById('cRed'),
      yellowCoins: document.getElementById('cYellow'),
      restartBtn: document.getElementById('restart')
    };
  }

  /* Screen toggles (overlay is fixed so homepage layout remains unchanged) */
  showStartScreen(){ this.elements.start.classList.remove('hidden'); this.elements.game.classList.add('hidden'); }
  showGameScreen(){ this.elements.start.classList.add('hidden'); this.elements.game.classList.remove('hidden'); }

  /* Creates the 6x7 holes — each hole stores row/col as data-* for event delegation */
  createBoard(rows, cols) {
    this.elements.board.innerHTML = '';
    for (let r = 0; r < rows; r++) for (let c = 0; c < cols; c++) {
      const hole = document.createElement('div');
      hole.className = 'hole'; hole.dataset.row = r; hole.dataset.col = c;
      this.elements.board.appendChild(hole);
    }
  }

  /* Iterates grid to apply "filled red/yellow" classes */
  updateBoard(board) {
    const holes = [...this.elements.board.children];
    holes.forEach((hole, index) => {
      const row = Math.floor(index / board.cols);
      const col = index % board.cols;
      const piece = board.grid[row][col];
      hole.classList.remove('filled', 'red', 'yellow');
      if (piece) { hole.classList.add('filled', piece); }
    });
  }

  /* Timers + coin counts */
  updatePlayerInfo(redPlayer, yellowPlayer) {
    this.elements.redTimer.textContent = this.formatTime(redPlayer.time);
    this.elements.yellowTimer.textContent = this.formatTime(yellowPlayer.time);
    this.elements.redCoins.textContent = redPlayer.coins;
    this.elements.yellowCoins.textContent = yellowPlayer.coins;
  }

  /* MM:SS formatter (clamps at 00:00 visually) */
  formatTime(seconds) {
    const minutes = Math.floor(Math.max(0, seconds) / 60);
    const secs = Math.max(0, seconds) % 60;
    return `${String(minutes).padStart(2,'0')}:${String(secs).padStart(2,'0')}`;
  }

  /* Falling coin animation — moves reusable #fall to target cell location */
  async animateFallingCoin(col, row, color) {
    const cellSize = getCSSNum('--cell');
    const gap = getCSSNum('--gap');
    const padding = getCSSNum('--boardPad');
    const x = padding + col * (cellSize + gap);
    const y = padding + row * (cellSize + gap);

    this.elements.fallCoin.className = `coin ${color}`;
    this.elements.fallCoin.style.left = x + 'px';
    this.elements.fallCoin.style.transform = 'translate(0, -100%)';
    this.elements.fallCoin.classList.remove('hidden');

    // Force a reflow so the transition starts from above the board
    void this.elements.fallCoin.offsetWidth;

    const duration = Math.min(120 + row * 120, 520); // slightly longer for deeper drops
    this.elements.fallCoin.style.transitionDuration = `${duration}ms`;
    this.elements.fallCoin.style.transform = `translate(0, ${y}px)`;

    return new Promise(resolve => {
      const onEnd = () => {
        this.elements.fallCoin.removeEventListener('transitionend', onEnd);
        this.elements.fallCoin.classList.add('hidden');
        resolve();
      };
      this.elements.fallCoin.addEventListener('transitionend', onEnd, {once:true});
    });
  }

  /* (2) Column highlight helpers — add/remove .preview on all cells of a column */
  highlightColumn(col){
    const holes = this.elements.board.querySelectorAll(`.hole[data-col="${col}"]`);
    this.clearHighlight();
    holes.forEach(h => h.classList.add('preview'));
  }
  clearHighlight(){
    this.elements.board.querySelectorAll('.hole.preview').forEach(h => h.classList.remove('preview'));
  }

  /* BONUS) Keyboard ghost coin — visual pointer at current column (above the board) */
  setGhost(color, col){
    const cellSize = getCSSNum('--cell');
    const gap = getCSSNum('--gap');
    const pad = getCSSNum('--boardPad');
    const x = pad + col * (cellSize + gap);
    const ghost = this.elements.ghostCoin;
    ghost.className = `coin ghost ${color}`;
    ghost.style.left = x + 'px';
    ghost.style.transform = 'translate(0, -100%)'; // float above row 0
    ghost.classList.remove('hidden');
  }
  hideGhost(){ this.elements.ghostCoin.classList.add('hidden'); }

  /* (5) Win message — adds emoji + glow based on winner color */
  showWinMessage(message, color) {
    const emoji = color === 'red' ? '🔴🏆' : color === 'yellow' ? '🟡🏆' : '🤝';
    const glowClass = color === 'red' ? 'glow-red' : color === 'yellow' ? 'glow-yellow' : '';
    const winOverlay = document.createElement('div');
    winOverlay.className = 'win-overlay';
    winOverlay.innerHTML = `
      <div class="win-card">
        <h2 class="win-title ${glowClass}">${emoji} ${message}</h2>
        <button class="btn win-btn" onclick="this.closest('.win-overlay').remove()">Continue</button>
      </div>
    `;
    document.body.appendChild(winOverlay);
  }
}

/* -----------------------------------------------------------
   Main game controller — wires UI + state + inputs together
----------------------------------------------------------- */
class Connect4Game {
  constructor() {
    // Model
    this.board = new GameBoard(6, 7);
    this.redPlayer = new Player('Red', 'red');
    this.yellowPlayer = new Player('Yellow', 'yellow');
    this.currentPlayer = this.redPlayer;

    // Services
    this.timer = new GameTimer();
    this.ui = new GameUI();

    // Flags
    this.isRunning = false;   // true when a game is active
    this.isAnimating = false; // guard against double clicks during drop animation

    // Previews/keyboard state
    this.previewCol = null;   // last hovered/selected column
    this.selectedCol = 3;     // initial keyboard column (center)

    this.initializeEventListeners();
  }

  initializeEventListeners() {
    /* START SCREEN — only timer buttons start the game */
    this.ui.elements.start.querySelectorAll('.btn[data-time]').forEach(button => {
      button.addEventListener('click', () => {
        const timeInSeconds = parseInt(button.dataset.time);
        if (!Number.isFinite(timeInSeconds)) return;
        this.startGame(timeInSeconds);
      });
    });

    /* (1) THEME SWITCHER — applies data-theme on <html> */
    document.querySelectorAll('.btn.theme').forEach(btn => {
      btn.addEventListener('click', () => {
        const theme = btn.dataset.theme;
        document.documentElement.setAttribute('data-theme', theme);
      });
    });

    /* Enter to start with default 5:00 — only when game overlay is hidden */
    window.addEventListener('keydown', (e) => {
      if (!this.ui.elements.game.classList.contains('hidden') || e.key !== 'Enter') return;
      this.startGame(300);
    });

    /* Board click (mouse/tap) — uses event delegation */
    this.ui.elements.board.addEventListener('click', (e) => { this.handleBoardClick(e); });

    /* (2) HOVER HIGHLIGHT — when moving over any hole, highlight entire column */
    this.ui.elements.board.addEventListener('mouseover', (e) => {
      const hole = e.target.closest('.hole'); if (!hole) return;
      const col = parseInt(hole.dataset.col);
      this.previewCol = col;
      this.ui.highlightColumn(col);
      // Also align ghost with the hover column if game is running
      if (this.isRunning) this.ui.setGhost(this.currentPlayer.color, col);
    });
    this.ui.elements.board.addEventListener('mouseleave', () => {
      this.previewCol = null; this.ui.clearHighlight(); this.ui.hideGhost();
    });

    /* BONUS) KEYBOARD CONTROLS — ←/→ move selection, Enter drops coin */
    window.addEventListener('keydown', (e) => {
      // Ignore when start screen is visible or during animations
      if (this.ui.elements.game.classList.contains('hidden') || !this.isRunning || this.isAnimating) return;
      if (['ArrowLeft','ArrowRight','Enter'].includes(e.key)) e.preventDefault();

      if (e.key === 'ArrowLeft') {
        this.selectedCol = Math.max(0, (this.previewCol ?? this.selectedCol) - 1);
        this.previewCol = this.selectedCol;
        this.ui.highlightColumn(this.selectedCol);
        this.ui.setGhost(this.currentPlayer.color, this.selectedCol);
      } else if (e.key === 'ArrowRight') {
        this.selectedCol = Math.min(this.board.cols - 1, (this.previewCol ?? this.selectedCol) + 1);
        this.previewCol = this.selectedCol;
        this.ui.highlightColumn(this.selectedCol);
        this.ui.setGhost(this.currentPlayer.color, this.selectedCol);
      } else if (e.key === 'Enter') {
        // Synthesize a click on the column under preview/selection
        const col = (this.previewCol ?? this.selectedCol);
        const fakeTarget = this.ui.elements.board.querySelector(`.hole[data-col="${col}"]`);
        if (fakeTarget) this.handleBoardClick({ target: fakeTarget });
      }
    });

    /* (4) RESTART CONFIRM — protect against accidental resets */
    this.ui.elements.restartBtn.addEventListener('click', () => { this.restart(); });
  }

  /* Game setup/reset for a new match */
  startGame(timeInSeconds) {
    // Reset state
    this.board.reset();
    this.redPlayer.reset(timeInSeconds);
    this.yellowPlayer.reset(timeInSeconds);
    this.currentPlayer = this.redPlayer;
    this.isRunning = true;
    this.isAnimating = false;
    this.previewCol = null;
    this.selectedCol = 3; // center

    // UI init
    this.ui.showGameScreen();
    this.ui.createBoard(this.board.rows, this.board.cols);
    this.ui.updateBoard(this.board);
    this.ui.updatePlayerInfo(this.redPlayer, this.yellowPlayer);

    // Show ghost centered initially
    this.ui.setGhost(this.currentPlayer.color, this.selectedCol);

    // Start ticking timers (per-second)
    this.timer.start(
      () => this.handleTimerTick(),
      (player) => this.handleTimeUp(player) // not used explicitly here
    );
  }

  /* Handles board interaction for both real mouse clicks and keyboard "Enter" */
  async handleBoardClick(event) {
    const hole = event.target.closest('.hole');
    if (!hole || !this.isRunning || this.isAnimating) return;

    const col = parseInt(hole.dataset.col);
    const row = this.board.getDropRow(col);

    // Safety checks: full column or no coins
    if (row < 0) return;
    if (!this.currentPlayer.usesCoin()) return;

    // Prevent re-entrancy during animation
    this.isAnimating = true;

    // Animate falling coin above the board into place
    await this.ui.animateFallingCoin(col, row, this.currentPlayer.color);

    // Commit to board
    this.board.placePiece(row, col, this.currentPlayer.color);

    // (3) Play drop sound AFTER a valid placement (user-gesture safe)
    playDropSound();

    // Reflect board + HUD
    this.ui.updateBoard(this.board);
    this.ui.updatePlayerInfo(this.redPlayer, this.yellowPlayer);

    // Allow interactions again
    this.isAnimating = false;

    // End conditions
    if (this.board.checkWin(row, col)) {
      this.endGame(`${this.currentPlayer.name} wins!`, this.currentPlayer.color); // (5) color drives emoji/glow
      return;
    }
    if (this.board.isFull()) {
      this.endGame('Draw!', null);
      return;
    }

    // Next player
    this.switchPlayer();

    // Keep the preview/ghost on the last used column for continuity
    this.selectedCol = col;
    this.previewCol = col;
    this.ui.highlightColumn(col);
    this.ui.setGhost(this.currentPlayer.color, col);
  }

  /* Per-second timer update: when current player's time hits 0, opponent wins */
  handleTimerTick() {
    if (!this.isRunning) return;
    this.currentPlayer.decrementTime();
    if (!this.currentPlayer.hasTimeLeft()) {
      const winner = this.currentPlayer === this.redPlayer ? this.yellowPlayer : this.redPlayer;
      this.endGame(`${winner.name} wins on time!`, winner.color);
      return;
    }
    this.ui.updatePlayerInfo(this.redPlayer, this.yellowPlayer);
  }

  switchPlayer() { this.currentPlayer = this.currentPlayer === this.redPlayer ? this.yellowPlayer : this.redPlayer; }

  /* Show win banner + stop timer; also clear previews/ghost */
  endGame(message, color) {
    this.isRunning = false;
    this.timer.stop();
    this.ui.clearHighlight(); this.ui.hideGhost();
    this.ui.showWinMessage(message, color);
  }

  /* (4) Confirm before leaving the game back to start screen */
  restart() {
    if (!confirm('Start a new game?')) return;
    this.timer.stop();
    this.ui.showStartScreen();
    this.isRunning = false;
    this.ui.clearHighlight(); this.ui.hideGhost();
  }
}

/* -----------------------------------------------------------
   Bootstrapping — set theme and create a game instance
----------------------------------------------------------- */
(() => {
  // Default to Midnight to preserve existing look-and-feel
  document.documentElement.setAttribute('data-theme', 'midnight');

  // Create controller; listeners attach inside constructor
  const game = new Connect4Game();
})();
</script>
