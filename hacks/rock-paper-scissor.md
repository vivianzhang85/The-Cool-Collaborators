---
title: Rock paper Scissors
comments: true
hide: true
layout: base
description: Learn how to experiment with the console, elements, and see OOP in action while playing Rock paper Scissors!
permalink: /rock-paper-scissor/
---


<div id="mainGameBox" style="max-width:700px;margin:64px auto 48px auto;position:relative;z-index:2;">
  <div id="gameContainer">
    <canvas id='gameCanvas' style="display:none"></canvas>
  </div>
</div>

<script type="module">

  let playerScore = 0;
let computerScore = 0;

    const instructionsStyle = `
  position: relative;
  margin: 64px auto 48px auto;
    background: linear-gradient(135deg, black, purple);
    color: white;
    padding: 30px;
    border-radius: 15px;
    z-index: 1000;
    max-width: 600px;
    width: 90%;
    max-height: 80vh;      /* added */
    overflow-y: auto;      /* added */
    font-family: 'Press Start 2P', cursive;
    border: 3px solid purple;
    box-shadow: 0 0 20px rgba(128, 0, 128, 0.5);
    text-align: center;
    `;

  const instructionsHTML = `
    <h2 style="color: purple; margin-bottom: 20px;">Rock Paper Scissors SHOOT!</h2>
    <div style="margin-bottom: 20px;">
      <p>Play the game from your browser console!</p>
      <p>Type <code>playRPS("rock")</code>, <code>playRPS("paper")</code>, or <code>playRPS("scissors")</code></p>
    </div>
    <div id="images" style="display:flex; justify-content:center; gap:20px; margin-bottom:14px;">
      <button id="rock-btn" style="background:none; border:none; padding:0; cursor:pointer;">
        <img id="rock-img" src="{{site.baseurl}}/images/rps/rock.jpg"
             style="width:100px; border:2px solid white; border-radius:10px;">
      </button>
      <button id="paper-btn" style="background:none; border:none; padding:0; cursor:pointer;">
        <img id="paper-img" src="{{site.baseurl}}/images/rps/paper.jpeg"
             style="width:100px; border:2px solid white; border-radius:10px;">
      </button>
      <button id="scissors-btn" style="background:none; border:none; padding:0; cursor:pointer;">
        <img id="scissors-img" src="{{site.baseurl}}/images/rps/scissors.jpeg"
             style="width:100px; border:2px solid white; border-radius:10px;">
      </button>
    </div>
    <div style="margin-bottom:18px; font-size:1.1em; color:#ffd700;">
      Click any icon to customize using the console!
    </div>
    <!-- mount battle canvas INSIDE the purple box so you can see it -->
    <div id="battleMount" style="display:block; margin:12px auto;"></div>

    <div id="resultBox" style="margin-top: 16px; font-size: 16px; color: yellow;"></div>
  `;
  const container = document.createElement("div");
  container.setAttribute("style", instructionsStyle);
  container.innerHTML = instructionsHTML;
  document.getElementById("mainGameBox").appendChild(container);

  // --- helper: highlight chosen image ---
  function highlightImage(id){
    ["rock-img","paper-img","scissors-img"].forEach(i=>{
      const el = document.getElementById(i);
      if(el) el.style.boxShadow = "";
    });
    const picked = document.getElementById(id);
    if(picked) picked.style.boxShadow = "0 0 30px 10px gold";
  }

  // --- OOP classes ---
  class BattleBackground {
    constructor(image, width, height, speedRatio=0.1){
      this.image = image;
      this.width = width;
      this.height = height;
      this.x = 0; this.y = 0;
      this.speed = 2 * speedRatio;
    }
    update(){ this.x = (this.x - this.speed) % this.width; }
    draw(ctx){
      if(!this.image.complete || this.image.naturalWidth===0) return;
      ctx.drawImage(this.image, this.x, this.y, this.width, this.height);
      ctx.drawImage(this.image, this.x + this.width, this.y, this.width, this.height);
    }
  }

  class BattleSprite {
    constructor(image, width, height, x, y){
      this.image = image;
      this.width = width; this.height = height;
      this.homeX = x; this.homeY = y;
      this.x = x; this.y = y;
      this.targetX = x; this.targetY = y;
      this.opacity = 1; this.scale = 1; this.rotation = 0;
      this.animating = false;
    }
    update(){
      if(this.animating){
        this.x += (this.targetX - this.x)*0.12;
        this.y += (this.targetY - this.y)*0.12;
      } else {
        // drift gently back to home
        this.x += (this.homeX - this.x)*0.08;
        this.y += (this.homeY - this.y)*0.08;
      }
    }
    draw(ctx){
      if(!this.image.complete || this.image.naturalWidth===0) return;
      ctx.save();
      ctx.globalAlpha = this.opacity;
      ctx.translate(this.x + this.width/2, this.y + this.height/2);
      ctx.rotate(this.rotation);
      ctx.scale(this.scale, this.scale);
      ctx.drawImage(this.image, -this.width/2, -this.height/2, this.width, this.height);
      ctx.restore();
    }
    resetVisuals(){
      this.opacity = 1; this.scale = 1; this.rotation = 0;
    }
    resetPosition(){
      this.x = this.homeX; this.y = this.homeY;
      this.targetX = this.homeX; this.targetY = this.homeY;
      this.animating = false;
    }
  }

  // --- Canvas mounted inside purple box ---
  const battleCanvas = document.createElement('canvas');
  battleCanvas.width = 360;
  battleCanvas.height = 180;
  battleCanvas.style.display = 'block';
  battleCanvas.style.margin = '0 auto';
  battleCanvas.style.background = '#111';
  battleCanvas.style.borderRadius = '12px';
  battleCanvas.style.boxShadow = '0 2px 12px rgba(0,0,0,0.18)';
  document.getElementById('battleMount').appendChild(battleCanvas);
  const ctx = battleCanvas.getContext('2d');

  // --- assets ---
  const bgImage = new Image();
  bgImage.src = '{{site.baseurl}}/images/platformer/backgrounds/alien_planet1.jpg';

  const rockImg = new Image();
  rockImg.src = '{{site.baseurl}}/images/rps/rock.jpg';
  const paperImg = new Image();
  paperImg.src = '{{site.baseurl}}/images/rps/paper.jpeg';
  const scissorsImg = new Image();
  scissorsImg.src = '{{site.baseurl}}/images/rps/scissors.jpeg';

  const bg = new BattleBackground(bgImage, battleCanvas.width, battleCanvas.height, 0.12);

  const sprites = {
  rock:     new BattleSprite(rockImg,     96, 96,  10, 42),
  paper:    new BattleSprite(paperImg,    96, 96, 132, 42),
  scissors: new BattleSprite(scissorsImg, 96, 96, 254, 42)
  };

  function resetAll(){
    Object.values(sprites).forEach(s=>{
      s.resetVisuals();
    });
    sprites.rock.x = 10; sprites.rock.y = 42; sprites.rock.targetX = 10; sprites.rock.targetY = 42; sprites.rock.homeX = 10; sprites.rock.homeY = 42;
    sprites.paper.x = 132; sprites.paper.y = 42; sprites.paper.targetX = 132; sprites.paper.targetY = 42; sprites.paper.homeX = 132; sprites.paper.homeY = 42;
    sprites.scissors.x = 254; sprites.scissors.y = 42; sprites.scissors.targetX = 254; sprites.scissors.targetY = 42; sprites.scissors.homeX = 254; sprites.scissors.homeY = 42;
  }

  // --- global battle state, rendered by a continuous loop ---
  const battle = {
    active: false,
    winner: null,
    loser: null,
    frames: 0,
    max: 120,
    tie: null
  };

  function startBattle(winner, loser){
    battle.active = true;
    battle.tie = null;
    battle.winner = winner;
    battle.loser = loser;
    battle.frames = 0;

    // set targets for "winner moves toward loser"
    sprites[winner].animating = true;
    sprites[winner].targetX = sprites[loser].homeX;
    sprites[winner].targetY = sprites[loser].homeY;

    // loser will fade/scale/rotate in the render loop
    sprites[loser].animating = false; // stays put, gets affected visually
  }

  function startTie(choice){
    battle.active = true;
    battle.tie = choice;
    battle.winner = null;
    battle.loser = null;
    battle.frames = 0;

    // small wiggle, no target move
    Object.values(sprites).forEach(s=>{ s.animating = false; });
  }

  // --- continuous render loop (always runs) ---
  function render(){
  ctx.clearRect(0,0,battleCanvas.width,battleCanvas.height);
  bg.update();  bg.draw(ctx);
  // Draw 'Animated Battle: OOP' text (smaller)
  ctx.save();
  ctx.font = "bold 14px 'Press Start 2P', cursive";
  ctx.fillStyle = "cyan";
  ctx.textAlign = "center";
  ctx.fillText("Animated Battle: OOP", battleCanvas.width/2, 24);
  ctx.restore();

    if(battle.active){
      const t = battle.frames / battle.max; // 0..1

      if(battle.tie){
        const wobble = Math.sin(battle.frames*0.3)*4;
        sprites[battle.tie].rotation = wobble * Math.PI/180;
      } else {
        // winner punch-in / pulse
        const w = sprites[battle.winner];
        const l = sprites[battle.loser];

        // winner pulse scale up then down
        const pulse = (battle.frames < battle.max/2)
          ? 1 + (battle.frames/(battle.max/2))*0.2
          : 1.2 - ((battle.frames - battle.max/2)/(battle.max/2))*0.2;
        w.scale = pulse;

        // loser fades & shrinks
        l.opacity = Math.max(0.15, 1 - t*0.85);
        l.scale   = Math.max(0.6, 1 - t*0.4);

        // matchup-specific flair
        if(battle.winner === "rock" && battle.loser === "scissors"){
          l.rotation = -t * (Math.PI/4);
        }
        if(battle.winner === "paper" && battle.loser === "rock"){
          // paper "covers" rock by moving slightly past center
          w.targetX = l.homeX - 6; w.targetY = l.homeY - 6;
        }
        if(battle.winner === "scissors" && battle.loser === "paper"){
          w.rotation =  t * (Math.PI/10);
          l.rotation = -t * (Math.PI/10);
        }
      }

      battle.frames++;
      if(battle.frames >= battle.max){
        battle.active = false;
        Object.values(sprites).forEach(s=>{ s.resetVisuals(); s.animating = false; });
      }
    }

    // update/draw sprites every frame
    Object.values(sprites).forEach(s=>{ s.update(); s.draw(ctx); });

    requestAnimationFrame(render);
  }
  render(); // kick off the engine once

  // --- game logic + console entry point ---
  window.playRPS = function(playerChoice){
    const choices = ["rock","paper","scissors"];
    if(!choices.includes(playerChoice)){
      console.log("Invalid choice. Use 'rock', 'paper', or 'scissors'.");
      return;
    }

    highlightImage(playerChoice+"-img");

    const computerChoice = choices[Math.floor(Math.random()*choices.length)];
    let resultText, winner=null, loser=null;

    if(playerChoice === computerChoice){
      resultText = "Tie!";
      startTie(playerChoice);
    } else if(
      (playerChoice==="rock" && computerChoice==="scissors") ||
      (playerChoice==="paper" && computerChoice==="rock") ||
      (playerChoice==="scissors" && computerChoice==="paper")
    ){
      resultText = "You Win!";
      winner = playerChoice; loser = computerChoice;
    } else {
      resultText = "You Lose!";
      winner = computerChoice; loser = playerChoice;
    }

    // --- Add scores here ---
    if(resultText === "You Win!") playerScore++;
    else if(resultText === "You Lose!") computerScore++;
    console.log(`Score → You: ${playerScore} | Computer: ${computerScore}`);

    document.getElementById("resultBox").innerHTML = `
      <p>You chose: <b>${playerChoice.toUpperCase()}</b></p>
      <p>Computer chose: <b>${computerChoice.toUpperCase()}</b></p>
      <h3 style="color: cyan;">${resultText}</h3>
    `;

    if(winner && loser) startBattle(winner, loser);

    console.log(`You chose: ${playerChoice.toUpperCase()}`);
    console.log(`Computer chose: ${computerChoice.toUpperCase()}`);
    console.log(`Result: ${resultText}`);
};


  class GameObject {
    constructor(id) {
      this.el = document.getElementById(id);
      if (!this.el) throw new Error(`Element #${id} not found`);
    }

    rotate(deg) {
      this.el.style.transform = `rotate(${deg}deg)`;
      return this;
    }

    setBorder(style) {
      this.el.style.border = style;
      return this;
    }

    setWidth(px) {
      this.el.style.width = `${px}px`;
      return this;
    }

    setColor(color) {
      this.el.style.backgroundColor = color;
      return this;
    }

    reset() {
      this.el.style.transform = "";
      this.el.style.border = "";
      this.el.style.width = "";
      this.el.style.backgroundColor = "";
      return this;
    }
  }

  // --- Specialized classes (extend GameObject) ---
  class Rock extends GameObject {
    constructor() { super("rock-img"); }
  }

  class Paper extends GameObject {
    constructor() { super("paper-img"); }
  }

  class Scissors extends GameObject {
    constructor() { super("scissors-img"); }
  }

  // --- Instances (global) ---
  const rock = new Rock();
  const paper = new Paper();
  const scissors = new Scissors();

  window.rock = rock;
  window.paper = paper;
  window.scissors = scissors;

  // --- inspect-learning alerts (unchanged) ---
  document.getElementById("rock-btn").addEventListener("click", () => {
    alert("🪨 Try in the console:\n\nrock.setBorder('4px solid lime');");
  });
  document.getElementById("paper-btn").addEventListener("click", () => {
    alert("📄 Try in the console:\n\npaper.rotate(15);");
  });
  document.getElementById("scissors-btn").addEventListener("click", () => {
    alert("✂️ Try in the console:\n\nscissors.setWidth(150);");
  });
  /* =======================
   🎛️ RPS Plugin Pack (Drop-in)
   Paste below your existing code in the same <script type="module">
   ======================= */

// --- tiny helpers ---
const $ = (sel) => document.querySelector(sel);
const resultBoxEl = document.getElementById("resultBox");

// --- 1) Floating HUD (score, reset, mute, auto) ---
const hud = document.createElement("div");
hud.style.cssText = `
  position: sticky; top: 12px; margin: 0 auto 16px; z-index: 1001;
  display:flex; gap:10px; align-items:center; justify-content:center;
`;
hud.innerHTML = `
  <div style="background:#191919;border:2px solid #7a00ff;border-radius:12px;
              padding:8px 12px;font:600 12px/1.2 system-ui, -apple-system, Segoe UI;
              color:#fff; box-shadow:0 6px 22px rgba(122,0,255,.25);">
    <span id="hudScore">Score → You: 0 | Computer: 0</span>
    <span style="margin-left:10px; opacity:.7;">(R/P/S keys work)</span>
    <div style="margin-top:6px; display:flex; gap:8px; justify-content:center;">
      <button id="btnReset" style="padding:6px 10px;border-radius:8px;border:1px solid #7a00ff;background:#2a2a2a;color:#fff;cursor:pointer;">Reset</button>
      <button id="btnMute"  style="padding:6px 10px;border-radius:8px;border:1px solid #7a00ff;background:#2a2a2a;color:#fff;cursor:pointer;">Mute: Off</button>
      <button id="btnAuto"  style="padding:6px 10px;border-radius:8px;border:1px solid #7a00ff;background:#2a2a2a;color:#fff;cursor:pointer;">Auto-Demo: Off</button>
    </div>
  </div>
`;
document.getElementById("mainGameBox").prepend(hud);

function updateHUD() {
  $("#hudScore").textContent = `Score → You: ${playerScore} | Computer: ${computerScore}`;
}

// --- 2) Persist scores (localStorage) ---
try {
  const saved = JSON.parse(localStorage.getItem("rpsScore") || "{}");
  if (Number.isFinite(saved.player) && Number.isFinite(saved.cpu)) {
    playerScore = saved.player;
    computerScore = saved.cpu;
    updateHUD();
  }
} catch (_) {}

function persistScores() {
  localStorage.setItem("rpsScore", JSON.stringify({ player: playerScore, cpu: computerScore }));
}

// --- 3) Add sounds (WebAudio) with Mute toggle ---
let audioCtx = null;
let muted = false;
function beep(freq = 440, dur = 0.12, type = "sine", gain = 0.06) {
  if (muted) return;
  if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  const o = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  o.type = type; o.frequency.value = freq;
  g.gain.value = gain;
  o.connect(g); g.connect(audioCtx.destination);
  o.start();
  setTimeout(() => { o.stop(); }, dur * 1000);
}
function playWinSfx(){ beep(880, .08, "triangle", .08); setTimeout(()=>beep(1175, .1, "triangle", .08), 100); }
function playLoseSfx(){ beep(330, .12, "sawtooth", .07); setTimeout(()=>beep(247, .14, "sawtooth", .07), 120); }
function playTieSfx(){ beep(523, .08, "square", .06); setTimeout(()=>beep(523, .08, "square", .06), 100); }

// --- 4) Confetti burst on wins (tiny canvas particles) ---
const confettiCanvas = document.createElement("canvas");
confettiCanvas.width = 360; confettiCanvas.height = 180;
confettiCanvas.style.cssText = "position:absolute; pointer-events:none; inset:0; margin:auto; border-radius:12px;";
document.getElementById("battleMount").style.position = "relative";
document.getElementById("battleMount").appendChild(confettiCanvas);
const cctx = confettiCanvas.getContext("2d");
let confetti = [];
function confettiBurst(n = 32) {
  confetti = Array.from({length:n}, () => ({
    x: Math.random()*confettiCanvas.width,
    y: confettiCanvas.height/2,
    vx: (Math.random()*2-1)*2.2,
    vy: -Math.random()*3-1.5,
    life: 60 + Math.random()*30,
    r: 2 + Math.random()*3
  }));
}
function confettiTick(){
  cctx.clearRect(0,0,confettiCanvas.width, confettiCanvas.height);
  confetti.forEach(p=>{
    p.x += p.vx; p.y += p.vy; p.vy += 0.06; p.life--;
    cctx.globalAlpha = Math.max(0, p.life/90);
    cctx.beginPath(); cctx.arc(p.x,p.y,p.r,0,Math.PI*2); cctx.fill();
  });
  confetti = confetti.filter(p=>p.life>0);
  requestAnimationFrame(confettiTick);
}
confettiTick();

// --- 5) Keyboard controls: R / P / S ---
document.addEventListener("keydown", (e)=>{
  const k = e.key.toLowerCase();
  if (k === "r") playRPS("rock");
  if (k === "p") playRPS("paper");
  if (k === "s") playRPS("scissors");
});

// --- 6) Auto-demo (plays itself) ---
let autoTimer = null;
function toggleAuto() {
  if (autoTimer) {
    clearInterval(autoTimer); autoTimer = null;
    $("#btnAuto").textContent = "Auto-Demo: Off";
  } else {
    const choices = ["rock","paper","scissors"];
    autoTimer = setInterval(()=> playRPS(choices[Math.floor(Math.random()*3)]), 1100);
    $("#btnAuto").textContent = "Auto-Demo: On";
  }
}

// --- 7) Buttons: reset / mute / auto ---
$("#btnReset").addEventListener("click", ()=>{
  playerScore = 0; computerScore = 0; updateHUD(); persistScores();
});
$("#btnMute").addEventListener("click", ()=>{
  muted = !muted; $("#btnMute").textContent = `Mute: ${muted ? "On" : "Off"}`;
});
$("#btnAuto").addEventListener("click", toggleAuto);

// --- 8) Wrap playRPS to add effects *after* the original runs (no logic change) ---
const _playRPS = window.playRPS;
window.playRPS = function(choice){
  const ret = _playRPS(choice);      // run original logic
  updateHUD();                       // refresh HUD
  persistScores();                   // save scores

  // inspect result text to decide SFX / confetti
  const text = (resultBoxEl?.textContent || "").toLowerCase();
  if (text.includes("you win")) { playWinSfx(); confettiBurst(36); }
  else if (text.includes("you lose")) { playLoseSfx(); }
  else if (text.includes("tie")) { playTieSfx(); }
  return ret;
};

// --- 9) Tiny easter egg: Konami code swaps the background image ---
const konami = ["arrowup","arrowup","arrowdown","arrowdown","arrowleft","arrowright","arrowleft","arrowright","b","a"];
let konamiIdx = 0;
document.addEventListener("keydown", (e)=>{
  konamiIdx = (e.key.toLowerCase() === konami[konamiIdx]) ? konamiIdx+1 : 0;
  if (konamiIdx === konami.length) {
    konamiIdx = 0;
    // swap to a second background (add your own image path if you want)
    try {
      bgImage.src = '{{site.baseurl}}/images/platformer/backgrounds/alien_planet2.jpg';
      alert("🕹️ Konami unlocked! New background loaded.");
    } catch(_) {}
  }
});

</script>

