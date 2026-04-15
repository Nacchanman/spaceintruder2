<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover,user-scalable=no" />
<title>SPACE INTRUDER</title>
<style>
  :root{
    --bg1:#060a16;
    --bg2:#101b3a;
    --panel:rgba(9,14,31,.94);
    --line:rgba(255,255,255,.12);
    --text:#eef4ff;
    --accent:#71d7ff;
    --good:#61ffa8;
    --warn:#ffd166;
    --danger:#ff708f;
  }
  *{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
  html,body{
    margin:0;padding:0;width:100%;height:100%;overflow:hidden;
    background:
      radial-gradient(circle at 50% 18%, rgba(92,130,255,.22), transparent 34%),
      linear-gradient(180deg,var(--bg2),var(--bg1));
    color:var(--text);
    font-family:system-ui,-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif;
    touch-action:none;
  }
  #app{
    position:fixed; inset:0; margin:0 auto;
    width:min(100vw, calc((100vh - 4px) * 0.62), 430px);
    min-width:320px;
    display:flex; flex-direction:column;
  }
  #hud{
    display:grid;
    grid-template-columns:repeat(6,1fr);
    gap:6px;
    padding:calc(env(safe-area-inset-top,0px) + 8px) 8px 8px;
    z-index:4;
    pointer-events:none;
    font-size:10px;
    font-weight:800;
    letter-spacing:.03em;
  }
  .hud-box{
    background:rgba(7,12,28,.7);
    border:1px solid var(--line);
    border-radius:10px;
    padding:6px 4px;
    text-align:center;
    box-shadow:0 8px 20px rgba(0,0,0,.16);
  }
  #gameWrap{
    position:relative;
    flex:1;
    min-height:0;
    padding:0 6px;
  }
  canvas{
    width:100%;
    height:100%;
    display:block;
    border-radius:18px;
    border:1px solid rgba(255,255,255,.08);
    background:
      radial-gradient(circle at 50% 6%, rgba(255,255,255,.06), transparent 20%),
      linear-gradient(180deg, rgba(255,255,255,.03), rgba(255,255,255,0)),
      #030712;
    box-shadow:0 16px 40px rgba(0,0,0,.28);
  }
  #overlay{
    position:absolute;
    inset:0;
    display:flex;
    align-items:center;
    justify-content:center;
    padding:14px;
    z-index:5;
    pointer-events:none;
  }
  #panel{
    width:min(100%, 440px);
    max-height:92%;
    overflow:auto;
    background:var(--panel);
    border:1px solid var(--line);
    border-radius:20px;
    padding:18px 16px;
    box-shadow:0 18px 44px rgba(0,0,0,.34);
    text-align:center;
    pointer-events:auto;
    backdrop-filter:blur(8px);
  }
  #panel h1,#panel h2,#panel h3{margin:0 0 10px;line-height:1.2}
  #panel p{margin:8px 0;line-height:1.5;font-size:14px;color:#dbe7ff}
  #panel .small{font-size:12px;opacity:.92}
  .menu-grid,.shop-grid{display:grid;gap:10px;text-align:left;margin-top:10px}
  .menu-item,.shop-item{
    border:1px solid rgba(255,255,255,.12);
    border-radius:14px;
    padding:12px;
    background:rgba(255,255,255,.04);
  }
  .menu-row,.shop-row{display:flex;justify-content:space-between;align-items:center;gap:10px}
  button,.menu-btn,.shop-btn,.ctrl{
    appearance:none;
    border:1px solid rgba(255,255,255,.12);
    border-bottom-width:3px;
    border-radius:16px;
    font-weight:900;
    letter-spacing:.03em;
    user-select:none;
    touch-action:manipulation;
    box-shadow:0 10px 18px rgba(0,0,0,.22);
  }
  .menu-btn,.shop-btn,#panel button{
    min-height:44px;
    padding:0 12px;
    color:#091120;
    background:linear-gradient(180deg,#91f2ff,#53c7ff);
  }
  .menu-btn.alt{background:linear-gradient(180deg,#e8d8ff,#9b7bff)}
  .shop-btn{background:linear-gradient(180deg,#ffe08a,#ffc43c)}
  #controls{
    display:grid;
    grid-template-columns:1fr 1fr;
    grid-template-areas:
      "left right"
      "pause fire"
      "pause fire";
    gap:6px;
    padding:8px 8px calc(env(safe-area-inset-bottom,0px) + 10px);
    z-index:4;
  }
  .ctrl{
    min-height:54px;
    font-size:16px;
    color:var(--text);
    background:linear-gradient(180deg, rgba(255,255,255,.12), rgba(255,255,255,.04));
  }
  .ctrl:active,.ctrl.active{
    transform:translateY(1px) scale(.985);
    background:linear-gradient(180deg, rgba(83,199,255,.22), rgba(83,199,255,.08));
  }
  #leftBtn{grid-area:left}
  #rightBtn{grid-area:right}
  #pauseBtn{grid-area:pause;font-size:13px}
  #fireBtn{
    grid-area:fire;
    min-height:114px;
    font-size:20px;
    color:#091120;
    background:linear-gradient(180deg,#ffd166,#ffb703);
    border-color:rgba(255,220,120,.45);
  }
  #fireBtn:active,#fireBtn.active{
    background:linear-gradient(180deg,#ffe08a,#ffc43c);
  }
  .hidden{display:none!important}
  .accent{color:var(--accent)}
  .good{color:var(--good)}
  .warn{color:var(--warn)}
  .danger{color:var(--danger)}
</style>
</head>
<body>
<div id="app">
  <div id="hud">
    <div class="hud-box">STAGE<br><span id="stageText">1</span></div>
    <div class="hud-box">LIFE<br><span id="lifeText">3</span></div>
    <div class="hud-box">SCORE<br><span id="scoreText">0</span></div>
    <div class="hud-box">COIN<br><span id="coinText">0</span></div>
    <div class="hud-box">POWER<br><span id="powerText">1</span></div>
    <div class="hud-box">MODE<br><span id="modeText">NORMAL</span></div>
  </div>

  <div id="gameWrap">
    <canvas id="game"></canvas>
    <div id="overlay"><div id="panel"></div></div>
  </div>

  <div id="controls">
    <button id="leftBtn" class="ctrl" aria-label="左へ">◀</button>
    <button id="rightBtn" class="ctrl" aria-label="右へ">▶</button>
    <button id="pauseBtn" class="ctrl" aria-label="ポーズ">Pause</button>
    <button id="fireBtn" class="ctrl" aria-label="発射">Fire</button>
  </div>
</div>

<script>
(() => {
  const canvas = document.getElementById("game");
  const ctx = canvas.getContext("2d");
  const DPR = Math.max(1, Math.min(window.devicePixelRatio || 1, 2));

  const ui = {
    stageText: document.getElementById("stageText"),
    lifeText: document.getElementById("lifeText"),
    scoreText: document.getElementById("scoreText"),
    coinText: document.getElementById("coinText"),
    powerText: document.getElementById("powerText"),
    modeText: document.getElementById("modeText"),
    overlay: document.getElementById("overlay"),
    panel: document.getElementById("panel"),
    leftBtn: document.getElementById("leftBtn"),
    rightBtn: document.getElementById("rightBtn"),
    pauseBtn: document.getElementById("pauseBtn"),
    fireBtn: document.getElementById("fireBtn"),
    gameWrap: document.getElementById("gameWrap"),
  };

  const STORAGE_KEY = "space_intruder_stable_v1";

  const difficultyDefs = {
    easy:   { name:"EASY",   enemyFire:0.75, enemySpeed:0.88, enemyHP:0.9, reward:0.9, playerHP:1 },
    normal: { name:"NORMAL", enemyFire:1.00, enemySpeed:1.00, enemyHP:1.0, reward:1.0, playerHP:0 },
    hard:   { name:"HARD",   enemyFire:1.18, enemySpeed:1.10, enemyHP:1.15, reward:1.2, playerHP:-1 }
  };

  const stageDefs = [
    { name:"STAGE 1", subtitle:"前哨宙域", rows:3, cols:5, spacingX:58, spacingY:56, enemyHP:[1],       fireChance:0.0042, enemySpeed:16, moveDrop:16, moveInterval:1.08, types:["scout"], boss:null,       bgHue:210 },
    { name:"STAGE 2", subtitle:"防衛ライン", rows:4, cols:6, spacingX:54, spacingY:54, enemyHP:[1,2],   fireChance:0.0054, enemySpeed:20, moveDrop:18, moveInterval:0.96, types:["scout","guard"], boss:null, bgHue:225 },
    { name:"STAGE 3", subtitle:"旗艦迎撃", rows:4, cols:6, spacingX:52, spacingY:52, enemyHP:[1,2,2],   fireChance:0.0062, enemySpeed:24, moveDrop:18, moveInterval:0.88, types:["scout","guard","elite"], boss:"dreadnought", bgHue:245 },
    { name:"STAGE 4", subtitle:"火星軌道", rows:4, cols:6, spacingX:50, spacingY:50, enemyHP:[2,2,3],   fireChance:0.0068, enemySpeed:26, moveDrop:18, moveInterval:0.82, types:["guard","elite"], boss:null, bgHue:15 },
    { name:"STAGE 5", subtitle:"機械要塞", rows:5, cols:6, spacingX:48, spacingY:48, enemyHP:[2,3,3],   fireChance:0.0074, enemySpeed:28, moveDrop:18, moveInterval:0.76, types:["guard","elite","sniper"], boss:null, bgHue:150 },
    { name:"STAGE 6", subtitle:"コアネスト", rows:5, cols:6, spacingX:48, spacingY:46, enemyHP:[2,3,4], fireChance:0.0080, enemySpeed:30, moveDrop:18, moveInterval:0.70, types:["guard","elite","sniper"], boss:"overmind", bgHue:290 },
  ];

  const state = {
    running:false,
    paused:false,
    gameOver:false,
    win:false,
    stageIndex:0,
    phase:"menu",
    difficulty:"normal",
    score:0,
    coins:0,
    combo:1,
    comboTimer:0,
    stars:[],
    nebula:[],
    particles:[],
    shockwaves:[],
    pickups:[],
    enemies:[],
    playerBullets:[],
    enemyBullets:[],
    boss:null,
    moveDir:1,
    enemyMoveTimer:0,
    lastTs:0,
    messageTimer:0,
    screenFlash:0,
    shake:0,
    input:{ left:false, right:false, fire:false, pointerActive:false },
    player:null,
    runStats:{ kills:0, pickups:0, bosses:0, startAt:0 },
    meta:{
      bestScore:0,
      totalCoins:0,
      soundOn:true,
      permHP:0,
      permPower:0,
    }
  };

  // ---------- audio: safe and simple ----------
  let audioCtx = null;
  function ensureAudio(){
    if (!state.meta.soundOn) return;
    if (!audioCtx){
      const Ctx = window.AudioContext || window.webkitAudioContext;
      if (Ctx) audioCtx = new Ctx();
    }
    if (audioCtx && audioCtx.state === "suspended"){
      audioCtx.resume().catch(() => {});
    }
  }
  function sfx(freq=440, dur=0.05, type="square", vol=0.02, slide=1){
    if (!state.meta.soundOn) return;
    ensureAudio();
    if (!audioCtx) return;
    const t = audioCtx.currentTime;
    const o = audioCtx.createOscillator();
    const g = audioCtx.createGain();
    o.type = type;
    o.frequency.setValueAtTime(freq, t);
    o.frequency.exponentialRampToValueAtTime(Math.max(60, freq * slide), t + dur);
    g.gain.setValueAtTime(0.0001, t);
    g.gain.exponentialRampToValueAtTime(vol, t + 0.01);
    g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
    o.connect(g).connect(audioCtx.destination);
    o.start(t);
    o.stop(t + dur + 0.02);
  }
  function playSfx(name){
    if (name === "shot") sfx(760,0.05,"square",0.012,0.87);
    if (name === "enemy") sfx(220,0.07,"sawtooth",0.018,0.95);
    if (name === "hit") sfx(160,0.09,"square",0.03,0.65);
    if (name === "pickup") sfx(620,0.09,"triangle",0.03,1.18);
    if (name === "boss") { sfx(110,0.14,"sawtooth",0.03,0.82); setTimeout(()=>sfx(90,0.16,"square",0.02,0.75), 70); }
    if (name === "clear") { sfx(392,0.08,"triangle",0.025,1.25); setTimeout(()=>sfx(622,0.10,"triangle",0.025,1.25),80); }
  }

  // ---------- persistence ----------
  function loadMeta(){
    try{
      const raw = localStorage.getItem(STORAGE_KEY);
      if (!raw) return;
      const data = JSON.parse(raw);
      if (data && typeof data === "object") Object.assign(state.meta, data);
    }catch(_e){}
  }
  function saveMeta(){
    try{ localStorage.setItem(STORAGE_KEY, JSON.stringify(state.meta)); }catch(_e){}
  }

  // ---------- geometry ----------
  function viewW(){ return canvas.width / DPR; }
  function viewH(){ return canvas.height / DPR; }
  function clamp(v, min, max){ return Math.max(min, Math.min(max, v)); }
  function rand(min, max){ return Math.random() * (max - min) + min; }

  function resize(){
    const rect = ui.gameWrap.getBoundingClientRect();
    canvas.width = Math.floor(rect.width * DPR);
    canvas.height = Math.floor(rect.height * DPR);
    canvas.style.width = rect.width + "px";
    canvas.style.height = rect.height + "px";
    ctx.setTransform(DPR,0,0,DPR,0,0);
    initBackground();
    if (!state.player){
      createPlayer();
    } else {
      state.player.x = clamp(state.player.x, state.player.w/2 + 8, viewW() - state.player.w/2 - 8);
      state.player.y = Math.max(190, viewH() - 86);
    }
  }

  function initBackground(){
    state.stars = Array.from({length:84}, () => ({
      x:Math.random()*viewW(),
      y:Math.random()*viewH(),
      r:Math.random()*1.8 + 0.4,
      s:Math.random()*18 + 8,
      a:Math.random()*0.55 + 0.18
    }));
    state.nebula = Array.from({length:4}, () => ({
      x:rand(40, Math.max(60, viewW()-40)),
      y:rand(40, Math.max(80, viewH()*0.7)),
      r:rand(46, 96),
      a:rand(0.04, 0.08)
    }));
  }

  function getStageLayout(stage){
    const topMargin = 48;
    const bottomReserve = 240;
    const available = Math.max(120, viewH() - topMargin - bottomReserve);
    const desiredHeight = Math.max(0, (stage.rows - 1) * stage.spacingY);
    const maxHeight = Math.max(90, available * 0.62);
    const scaleY = desiredHeight > 0 ? Math.min(1, maxHeight / desiredHeight) : 1;
    const spacingY = Math.max(36, stage.spacingY * scaleY);

    const desiredWidth = Math.max(0, (stage.cols - 1) * stage.spacingX);
    const maxWidth = Math.max(160, viewW() - 70);
    const scaleX = desiredWidth > 0 ? Math.min(1, maxWidth / desiredWidth) : 1;
    const spacingX = Math.max(34, stage.spacingX * scaleX);

    const formationHeight = (stage.rows - 1) * spacingY;
    const startY = Math.max(34, Math.min(62, topMargin + Math.max(0, (maxHeight - formationHeight) * 0.12)));
    return { spacingX, spacingY, startY };
  }

  // ---------- entities ----------
  function createPlayer(){
    const diff = difficultyDefs[state.difficulty];
    const hp = Math.max(1, 3 + state.meta.permHP + diff.playerHP);
    state.player = {
      x:viewW()/2,
      y:Math.max(190, viewH() - 86),
      w:34,
      h:38,
      speed:245,
      cooldown:0,
      hp,
      maxHp:hp,
      invincible:0,
      flash:0,
      power:Math.min(4, 1 + state.meta.permPower),
      shieldHits:0
    };
  }

  function effectiveHP(base){
    return Math.max(1, Math.round(base * difficultyDefs[state.difficulty].enemyHP));
  }

  function resetRun(){
    state.running = true;
    state.paused = false;
    state.gameOver = false;
    state.win = false;
    state.stageIndex = 0;
    state.phase = "wave";
    state.score = 0;
    state.coins = 0;
    state.combo = 1;
    state.comboTimer = 0;
    state.playerBullets = [];
    state.enemyBullets = [];
    state.pickups = [];
    state.particles = [];
    state.shockwaves = [];
    state.enemies = [];
    state.boss = null;
    state.moveDir = 1;
    state.enemyMoveTimer = 0;
    state.messageTimer = 0;
    state.screenFlash = 0;
    state.shake = 0;
    state.runStats = { kills:0, pickups:0, bosses:0, startAt:performance.now() };
    createPlayer();
    loadStage(0);
    ui.overlay.classList.add("hidden");
    ui.pauseBtn.textContent = "Pause";
    updateHUD();
  }

  function loadStage(index){
    const stage = stageDefs[index];
    const layout = getStageLayout(stage);
    state.stageIndex = index;
    state.phase = "wave";
    state.enemies = [];
    state.playerBullets = [];
    state.enemyBullets = [];
    state.pickups = [];
    state.moveDir = 1;
    state.enemyMoveTimer = 0;
    state.messageTimer = 2.1;
    state.boss = null;

    const totalWidth = (stage.cols - 1) * layout.spacingX;
    const startX = viewW()/2 - totalWidth/2;
    const startY = layout.startY;

    for (let r=0; r<stage.rows; r++){
      for (let c=0; c<stage.cols; c++){
        const type = stage.types[(r + c + index) % stage.types.length];
        let hp = type === "sniper" ? 2 : type === "elite" ? 3 : type === "guard" ? 2 : 1;
        hp = Math.max(hp, stage.enemyHP[Math.min(stage.enemyHP.length - 1, (r + c) % stage.enemyHP.length)]);
        hp = effectiveHP(hp);
        state.enemies.push({
          x:startX + c * layout.spacingX,
          y:startY + r * layout.spacingY,
          w:type === "elite" ? 38 : type === "guard" ? 34 : 30,
          h:type === "elite" ? 28 : type === "sniper" ? 26 : 24,
          type,
          hp,
          maxHp:hp,
          alive:true
        });
      }
    }
    updateHUD();
  }

  function spawnBoss(kind){
    const bossY = Math.max(66, Math.min(96, viewH() * 0.16));
    state.playerBullets = [];
    state.enemyBullets = [];
    state.pickups = [];
    playSfx("boss");

    if (kind === "dreadnought"){
      state.phase = "boss1";
      state.boss = {
        kind,
        form:1,
        x:viewW()/2,
        y:bossY,
        w:120,
        h:56,
        hp:effectiveHP(34),
        maxHp:effectiveHP(34),
        dir:1,
        speed:78,
        fireTimer:0,
        patternTimer:0,
        shield:0,
      };
    } else {
      state.phase = "boss1";
      state.boss = {
        kind,
        form:1,
        x:viewW()/2,
        y:bossY,
        w:86,
        h:86,
        hp:effectiveHP(44),
        maxHp:effectiveHP(44),
        dir:1,
        speed:88,
        fireTimer:0,
        patternTimer:0,
        shield:0,
      };
    }
    state.messageTimer = 2.5;
  }

  function bossNextForm(){
    if (!state.boss) return;
    const kind = state.boss.kind;
    state.playerBullets = [];
    state.enemyBullets = [];
    state.screenFlash = 0.38;
    state.shake = 0.26;
    addShockwave(viewW()/2, state.boss.y, 24, "#ff7aa6");
    addParticles(viewW()/2, state.boss.y, "#ff7aa6", 42, 140);
    playSfx("boss");

    if (kind === "dreadnought"){
      if (state.phase === "boss1"){
        state.phase = "boss2";
        state.boss = {
          kind,
          form:2,
          x:viewW()/2,
          y:Math.max(66, Math.min(96, viewH() * 0.16)),
          w:92,
          h:64,
          hp:effectiveHP(42),
          maxHp:effectiveHP(42),
          dir:1,
          speed:98,
          fireTimer:0,
          patternTimer:0,
          shield:0,
        };
      } else {
        killBoss();
      }
    } else {
      if (state.phase === "boss1"){
        state.phase = "boss2";
        state.boss = {
          kind,
          form:2,
          x:viewW()/2,
          y:Math.max(64, Math.min(94, viewH() * 0.16)),
          w:106,
          h:106,
          hp:effectiveHP(48),
          maxHp:effectiveHP(48),
          dir:1,
          speed:100,
          fireTimer:0,
          patternTimer:0,
          shield:0,
        };
      } else if (state.phase === "boss2"){
        state.phase = "boss3";
        state.boss = {
          kind,
          form:3,
          x:viewW()/2,
          y:Math.max(62, Math.min(92, viewH() * 0.15)),
          w:118,
          h:118,
          hp:effectiveHP(56),
          maxHp:effectiveHP(56),
          dir:1,
          speed:110,
          fireTimer:0,
          patternTimer:0,
          shield:0,
        };
      } else {
        killBoss();
      }
    }
    state.messageTimer = 2.4;
  }

  // ---------- effects ----------
  function addParticles(x,y,color,count=10,power=70){
    const limit = 220;
    const room = Math.max(0, limit - state.particles.length);
    const actual = Math.min(count, room);
    for (let i=0; i<actual; i++){
      const a = Math.random()*Math.PI*2;
      const s = Math.random()*power;
      const p = {
        x,y,
        vx:Math.cos(a)*s,
        vy:Math.sin(a)*s,
        life:Math.random()*0.35+0.25,
        maxLife:0,
        color,
        size:Math.random()*3+1.2
      };
      p.maxLife = p.life;
      state.particles.push(p);
    }
  }
  function addShockwave(x,y,r=10,color="#fff"){
    state.shockwaves.push({ x,y,r,life:0.42,maxLife:0.42,color });
  }

  // ---------- game rules ----------
  function updateHUD(){
    ui.stageText.textContent = String(state.stageIndex + 1);
    ui.lifeText.textContent = String(state.player ? state.player.hp : 0);
    ui.scoreText.textContent = String(state.score);
    ui.coinText.textContent = String(state.coins);
    ui.powerText.textContent = String(state.player ? state.player.power : 1);
    ui.modeText.textContent = difficultyDefs[state.difficulty].name;
  }

  function addScore(base){
    state.score += Math.round(base * difficultyDefs[state.difficulty].reward);
    updateHUD();
  }

  function gainCoins(base){
    const n = Math.max(1, Math.round(base * difficultyDefs[state.difficulty].reward));
    state.coins += n;
    state.meta.totalCoins += n;
    saveMeta();
    updateHUD();
  }

  function spawnPickup(x,y){
    const roll = Math.random();
    let kind = null;
    if (roll < 0.13) kind = "power";
    else if (roll < 0.23) kind = "heal";
    else if (roll < 0.32) kind = "shield";
    if (!kind) return;
    state.pickups.push({ x,y,w:18,h:18,vy:54,kind,spin:0 });
  }

  function applyPickup(kind){
    const p = state.player;
    if (kind === "power") p.power = Math.min(4, p.power + 1);
    if (kind === "heal") p.hp = Math.min(p.maxHp, p.hp + 1);
    if (kind === "shield") p.shieldHits = Math.min(3, p.shieldHits + 1);
    state.runStats.pickups++;
    addParticles(p.x,p.y-8, kind==="heal" ? "#61ffa8" : kind==="shield" ? "#b396ff" : "#ffd166", 12, 90);
    playSfx("pickup");
    updateHUD();
  }

  function spawnPlayerBullet(){
    const p = state.player;
    if (!state.running || state.paused || p.cooldown > 0) return;

    const bullets = [];
    const speed = -410;
    bullets.push({ x:p.x, y:p.y-p.h*0.52, vy:speed, vx:0, r:3.4, dmg:1, hp:1 });
    if (p.power >= 2){
      bullets.push({ x:p.x-12, y:p.y-p.h*0.2, vy:speed, vx:0, r:3.0, dmg:1, hp:1 });
      bullets.push({ x:p.x+12, y:p.y-p.h*0.2, vy:speed, vx:0, r:3.0, dmg:1, hp:1 });
    }
    if (p.power >= 3){
      bullets.push({ x:p.x-20, y:p.y-2, vy:speed+12, vx:-24, r:2.8, dmg:1, hp:1 });
      bullets.push({ x:p.x+20, y:p.y-2, vy:speed+12, vx:24, r:2.8, dmg:1, hp:1 });
    }
    if (p.power >= 4){
      bullets.push({ x:p.x, y:p.y-p.h*0.8, vy:speed-24, vx:0, r:4.0, dmg:2, hp:1 });
    }

    state.playerBullets.push(...bullets);
    if (state.playerBullets.length > 70){
      state.playerBullets.splice(0, state.playerBullets.length - 70);
    }
    p.cooldown = 0.22;
    addParticles(p.x,p.y-14,"#78e8ff",4,24);
    playSfx("shot");
  }

  function spawnEnemyBullet(enemy){
    const diff = difficultyDefs[state.difficulty];
    const base = (150 + state.stageIndex * 12 + (enemy.type==="elite" ? 34 : enemy.type==="guard" ? 16 : enemy.type==="sniper" ? 24 : 0)) * diff.enemySpeed;

    if (enemy.type === "elite" && Math.random() < 0.34){
      state.enemyBullets.push(
        { x:enemy.x-8, y:enemy.y+enemy.h/2, vx:-18, vy:base, r:4.5, type:"orb" },
        { x:enemy.x+8, y:enemy.y+enemy.h/2, vx:18, vy:base, r:4.5, type:"orb" }
      );
    } else if (enemy.type === "guard" && Math.random() < 0.28){
      for (let i=-1;i<=1;i++){
        state.enemyBullets.push({ x:enemy.x, y:enemy.y+enemy.h/2, vx:i*24, vy:base+14, r:4.0, type:"shot" });
      }
    } else if (enemy.type === "sniper"){
      const dx = state.player.x - enemy.x;
      const sign = Math.sign(dx) || 1;
      state.enemyBullets.push({ x:enemy.x, y:enemy.y+enemy.h/2, vx:sign*34, vy:base+26, r:4.4, type:"core" });
    } else {
      state.enemyBullets.push({ x:enemy.x, y:enemy.y+enemy.h/2, vx:0, vy:base, r:enemy.type==="scout" ? 3.8 : 4.6, type:"shot" });
    }

    if (state.enemyBullets.length > 180){
      state.enemyBullets.splice(0, state.enemyBullets.length - 180);
    }
    if (Math.random() < 0.28) playSfx("enemy");
  }

  function bossFirePattern(dt){
    const b = state.boss;
    if (!b) return;
    const diff = difficultyDefs[state.difficulty];
    b.fireTimer += dt;
    b.patternTimer += dt;

    if (b.kind === "dreadnought"){
      if (state.phase === "boss1"){
        if (b.fireTimer > 0.82 / diff.enemyFire){
          b.fireTimer = 0;
          const mode = Math.floor((b.patternTimer % 6) / 2);
          if (mode === 0){
            for (let i=-2;i<=2;i++){
              state.enemyBullets.push({ x:b.x+i*18, y:b.y+20, vx:i*10, vy:178*diff.enemySpeed, r:4.8, type:"orb" });
            }
          } else if (mode === 1){
            for (let i=-3;i<=3;i++){
              state.enemyBullets.push({ x:b.x, y:b.y+18, vx:i*22, vy:(164 + Math.abs(i)*8)*diff.enemySpeed, r:4.2, type:"shot" });
            }
          } else {
            state.enemyBullets.push(
              { x:b.x-30, y:b.y+16, vx:-10, vy:202*diff.enemySpeed, r:5.3, type:"orb" },
              { x:b.x+30, y:b.y+16, vx:10, vy:202*diff.enemySpeed, r:5.3, type:"orb" },
              { x:b.x, y:b.y+20, vx:0, vy:224*diff.enemySpeed, r:6.2, type:"core" }
            );
          }
          playSfx("enemy");
        }
      } else {
        if (b.fireTimer > 0.56 / diff.enemyFire){
          b.fireTimer = 0;
          const mode = Math.floor((b.patternTimer % 9) / 3);
          if (mode === 0){
            for (let i=0;i<10;i++){
              const a = (Math.PI*2/10)*i;
              state.enemyBullets.push({ x:b.x, y:b.y+4, vx:Math.cos(a)*84, vy:Math.sin(a)*84+82, r:4.2, type:"shot" });
            }
          } else if (mode === 1){
            for (let i=-3;i<=3;i++){
              state.enemyBullets.push({ x:b.x+i*14, y:b.y+18, vx:i*10, vy:220*diff.enemySpeed, r:4.8, type:"core" });
            }
          } else {
            for (let i=-2;i<=2;i++){
              state.enemyBullets.push({ x:b.x+i*16, y:b.y+14, vx:i*6, vy:148*diff.enemySpeed, r:5.4, type:"orb", acc:55 });
            }
          }
          playSfx("enemy");
        }
      }
    } else {
      if (state.phase === "boss1"){
        if (b.fireTimer > 0.66 / diff.enemyFire){
          b.fireTimer = 0;
          for (let ring=0; ring<2; ring++){
            for (let i=0;i<8;i++){
              const a = (Math.PI*2/8)*i + ring*0.22 + b.patternTimer*0.4;
              state.enemyBullets.push({ x:b.x, y:b.y, vx:Math.cos(a)*(60 + ring*18), vy:Math.sin(a)*(60 + ring*18)+74, r:4.2, type:"orb" });
            }
          }
          playSfx("enemy");
        }
      } else if (state.phase === "boss2"){
        if (b.fireTimer > 0.50 / diff.enemyFire){
          b.fireTimer = 0;
          for (let i=-3;i<=3;i++){
            state.enemyBullets.push({ x:b.x, y:b.y, vx:i*16, vy:(156 + Math.abs(i)*16)*diff.enemySpeed, r:4.6, type:"core" });
          }
          playSfx("enemy");
        }
      } else {
        if (b.fireTimer > 0.42 / diff.enemyFire){
          b.fireTimer = 0;
          for (let i=0;i<14;i++){
            const a = (Math.PI*2/14)*i + b.patternTimer*0.6;
            state.enemyBullets.push({ x:b.x, y:b.y, vx:Math.cos(a)*92, vy:Math.sin(a)*92+84, r:4.8, type:i%4===0 ? "core" : "orb" });
          }
          playSfx("enemy");
        }
      }
    }
    if (state.enemyBullets.length > 180){
      state.enemyBullets.splice(0, state.enemyBullets.length - 180);
    }
  }

  function hitPlayer(){
    const p = state.player;
    if (!state.running || p.invincible > 0) return;

    if (p.shieldHits > 0){
      p.shieldHits--;
      p.invincible = 0.45;
      addParticles(p.x,p.y,"#b396ff",16,100);
      addShockwave(p.x,p.y,12,"#b396ff");
      updateHUD();
      return;
    }

    p.hp--;
    p.invincible = 1.1;
    p.flash = 0.18;
    state.shake = Math.max(state.shake, 0.22);
    addParticles(p.x,p.y,"#ff7b95",18,110);
    addShockwave(p.x,p.y,12,"#ff7b95");
    playSfx("hit");
    updateHUD();

    if (p.hp <= 0){
      state.running = false;
      state.gameOver = true;
      finishRun(false);
      showGameOverPanel();
    }
  }

  function killEnemy(enemy){
    enemy.alive = false;
    state.runStats.kills++;
    state.shake = Math.max(state.shake, enemy.type==="elite" ? 0.12 : 0.07);
    const base = enemy.type==="elite" ? 260 : enemy.type==="guard" ? 160 : enemy.type==="sniper" ? 200 : 100;
    addScore(base);
    gainCoins(enemy.type==="elite" ? 8 : enemy.type==="guard" ? 5 : enemy.type==="sniper" ? 6 : 3);
    addParticles(enemy.x,enemy.y, enemy.type==="elite" ? "#ffd166" : "#9be7ff", 18, 100);
    addShockwave(enemy.x,enemy.y,10);
    spawnPickup(enemy.x,enemy.y);

    if (state.phase === "wave" && state.enemies.every(e => !e.alive)){
      const stage = stageDefs[state.stageIndex];
      if (stage.boss){
        state.running = false;
        showPanel("WARNING",
          `<p><span class="warn">ボス接近。</span></p><p>${stage.name} のボス戦が始まります。</p>`,
          "ボス戦へ",
          () => { state.running = true; spawnBoss(stage.boss); ui.overlay.classList.add("hidden"); }
        );
      } else if (state.stageIndex < stageDefs.length - 1){
        state.running = false;
        showShopPanel();
      } else {
        state.running = false;
        state.win = true;
        finishRun(true);
        showClearPanel();
      }
    }
    updateHUD();
  }

  function killBoss(){
    state.runStats.bosses++;
    state.shake = Math.max(state.shake, 0.42);
    addScore(4500);
    gainCoins(110);
    addParticles(state.boss.x,state.boss.y,"#ffd166",50,190);
    addShockwave(state.boss.x,state.boss.y,28,"#ffd166");
    playSfx("clear");

    state.boss = null;
    if (state.stageIndex < stageDefs.length - 1){
      state.running = false;
      showShopPanel(true);
    } else {
      state.running = false;
      state.win = true;
      finishRun(true);
      showClearPanel();
    }
  }

  function finishRun(_cleared){
    state.meta.bestScore = Math.max(state.meta.bestScore, state.score);
    saveMeta();
  }

  // ---------- UI ----------
  function resultSummaryHTML(){
    const secs = Math.max(0, Math.floor((performance.now() - (state.runStats.startAt || performance.now())) / 1000));
    const mins = Math.floor(secs / 60);
    const rem = String(secs % 60).padStart(2, "0");
    return `
      <div class="shop-grid">
        <div class="shop-item">
          <strong>RESULT</strong><br>
          <span class="small">
            撃破数: ${state.runStats.kills}<br>
            取得アイテム: ${state.runStats.pickups}<br>
            撃破ボス数: ${state.runStats.bosses}<br>
            プレイ時間: ${mins}:${rem}<br>
            難易度: ${difficultyDefs[state.difficulty].name}
          </span>
        </div>
      </div>
    `;
  }

  function showPanel(title, html, buttonText, onClick){
    ui.overlay.classList.remove("hidden");
    ui.panel.innerHTML = `<h2>${title}</h2>${html}<button id="panelBtn">${buttonText}</button>`;
    document.getElementById("panelBtn").onclick = () => {
      ensureAudio();
      if (onClick) onClick();
      else showTitleScreen();
    };
  }

  function showTitleScreen(){
    state.phase = "menu";
    state.running = false;
    state.paused = false;
    state.gameOver = false;
    state.win = false;
    createPlayer();
    updateHUD();
    ui.pauseBtn.textContent = "Pause";
    ui.overlay.classList.remove("hidden");
    ui.panel.innerHTML = `
      <h1 style="font-size:34px;letter-spacing:.08em;">SPACE INTRUDER</h1>
      <p><span class="accent">スマホ縦向き</span>で遊ぶ安定版です。</p>
      <p>6ステージ、ショップ、ボス戦つき。<br>Fire は右下配置です。</p>
      <div class="menu-grid">
        <div class="menu-item">
          <div class="menu-row">
            <div><strong>難易度</strong><br><span class="small">現在: ${difficultyDefs[state.difficulty].name}</span></div>
            <div>
              <button class="menu-btn alt" data-diff="easy">EASY</button>
              <button class="menu-btn alt" data-diff="normal">NORMAL</button>
              <button class="menu-btn alt" data-diff="hard">HARD</button>
            </div>
          </div>
        </div>
        <div class="menu-item">
          <div class="menu-row">
            <div><strong>サウンド</strong><br><span class="small">${state.meta.soundOn ? "ON" : "OFF"}</span></div>
            <button class="menu-btn alt" id="soundToggle">${state.meta.soundOn ? "OFFにする" : "ONにする"}</button>
          </div>
        </div>
        <div class="menu-item">
          <strong>記録</strong><br>
          <span class="small">
            BEST SCORE: ${state.meta.bestScore}<br>
            累計コイン: ${state.meta.totalCoins}<br>
            恒久強化: HP +${state.meta.permHP} / Power +${state.meta.permPower}
          </span>
        </div>
      </div>
      <button id="startGameBtn">ゲーム開始</button>
      <p class="small">操作: 左右ボタン or 画面ドラッグ / 右下 Fire 長押し / Pause</p>
    `;
    ui.panel.querySelectorAll("[data-diff]").forEach(btn => {
      btn.onclick = () => {
        state.difficulty = btn.dataset.diff;
        showTitleScreen();
      };
    });
    document.getElementById("soundToggle").onclick = () => {
      state.meta.soundOn = !state.meta.soundOn;
      saveMeta();
      showTitleScreen();
    };
    document.getElementById("startGameBtn").onclick = () => {
      ensureAudio();
      resetRun();
    };
  }

  function showShopPanel(fromBoss=false){
    ui.overlay.classList.remove("hidden");
    const nextStage = state.stageIndex + 2;
    ui.panel.innerHTML = `
      <h2>${fromBoss ? "BOSS CLEAR" : `STAGE ${state.stageIndex + 1} CLEAR`}</h2>
      <p><span class="good">中継ショップ</span>で強化できます。</p>
      <p>所持コイン: <strong>${state.coins}</strong></p>
      <div class="shop-grid">
        <div class="shop-item"><div class="shop-row"><div><strong>回復</strong><br><span class="small">HPを1回復</span></div><button class="shop-btn" data-buy="heal">10C</button></div></div>
        <div class="shop-item"><div class="shop-row"><div><strong>火力強化</strong><br><span class="small">Power +1（最大4）</span></div><button class="shop-btn" data-buy="power">18C</button></div></div>
        <div class="shop-item"><div class="shop-row"><div><strong>シールド</strong><br><span class="small">被弾を1回無効</span></div><button class="shop-btn" data-buy="shield">14C</button></div></div>
      </div>
      <button id="nextStageBtn">${nextStage <= stageDefs.length ? `STAGE ${nextStage} へ` : "エンディングへ"}</button>
    `;
    ui.panel.querySelectorAll("[data-buy]").forEach(btn => {
      btn.onclick = () => buyShopItem(btn.dataset.buy, fromBoss);
    });
    document.getElementById("nextStageBtn").onclick = () => {
      state.running = true;
      loadStage(state.stageIndex + 1);
      ui.overlay.classList.add("hidden");
    };
  }

  function buyShopItem(kind, fromBoss){
    const p = state.player;
    const costs = { heal:10, power:18, shield:14 };
    const cost = costs[kind];
    if (state.coins < cost) return;
    state.coins -= cost;
    if (kind === "heal") p.hp = Math.min(p.maxHp, p.hp + 1);
    if (kind === "power") p.power = Math.min(4, p.power + 1);
    if (kind === "shield") p.shieldHits = Math.min(3, p.shieldHits + 1);
    updateHUD();
    playSfx("pickup");
    showShopPanel(fromBoss);
  }

  function showMetaUpgradeBlock(){
    return `
      <div class="shop-grid">
        <div class="shop-item"><div class="shop-row"><div><strong>最大HP強化</strong><br><span class="small">現在 +${state.meta.permHP}</span></div><button class="shop-btn" data-meta="hp">${40 + state.meta.permHP*20}C</button></div></div>
        <div class="shop-item"><div class="shop-row"><div><strong>初期Power強化</strong><br><span class="small">現在 +${state.meta.permPower}</span></div><button class="shop-btn" data-meta="power">${60 + state.meta.permPower*30}C</button></div></div>
      </div>
    `;
  }

  function bindMetaButtons(){
    ui.panel.querySelectorAll("[data-meta]").forEach(btn => {
      btn.onclick = () => buyMeta(btn.dataset.meta);
    });
  }

  function buyMeta(kind){
    const costs = {
      hp: 40 + state.meta.permHP*20,
      power: 60 + state.meta.permPower*30,
    };
    const cost = costs[kind];
    if (state.meta.totalCoins < cost) return;
    state.meta.totalCoins -= cost;
    if (kind === "hp") state.meta.permHP++;
    if (kind === "power") state.meta.permPower = Math.min(2, state.meta.permPower + 1);
    saveMeta();
    if (state.gameOver) showGameOverPanel();
    else if (state.win) showClearPanel();
    else showTitleScreen();
  }

  function showGameOverPanel(){
    ui.overlay.classList.remove("hidden");
    ui.panel.innerHTML = `
      <h2>GAME OVER</h2>
      <p><span class="danger">宇宙船が撃墜されました。</span></p>
      <p>スコア: <strong>${state.score}</strong> / ベスト: <strong>${state.meta.bestScore}</strong></p>
      <p>累計コイン: <strong>${state.meta.totalCoins}</strong></p>
      ${resultSummaryHTML()}
      <h3>恒久アップグレード</h3>
      ${showMetaUpgradeBlock()}
      <button id="backTitleBtn">タイトルへ戻る</button>
    `;
    bindMetaButtons();
    document.getElementById("backTitleBtn").onclick = () => showTitleScreen();
  }

  function showClearPanel(){
    ui.overlay.classList.remove("hidden");
    ui.panel.innerHTML = `
      <h2>ALL CLEAR</h2>
      <p><span class="good">SPACE INTRUDER を制覇しました。</span></p>
      <p>最終スコア: <strong>${state.score}</strong></p>
      <p>ベストスコア: <strong>${state.meta.bestScore}</strong></p>
      <p>累計コイン: <strong>${state.meta.totalCoins}</strong></p>
      ${resultSummaryHTML()}
      <h3>恒久アップグレード</h3>
      ${showMetaUpgradeBlock()}
      <button id="backTitleBtn">タイトルへ戻る</button>
    `;
    bindMetaButtons();
    document.getElementById("backTitleBtn").onclick = () => showTitleScreen();
  }

  // ---------- collision ----------
  function rectsOverlap(a,b){
    return (
      a.x-a.w/2 < b.x+b.w/2 &&
      a.x+a.w/2 > b.x-b.w/2 &&
      a.y-a.h/2 < b.y+b.h/2 &&
      a.y+a.h/2 > b.y-b.h/2
    );
  }

  function bulletHitRect(bullet, target){
    const bw = bullet.r * 2.2;
    const bh = bullet.r * 5;
    return (
      bullet.x > target.x - target.w/2 - bw/2 &&
      bullet.x < target.x + target.w/2 + bw/2 &&
      bullet.y > target.y - target.h/2 - bh/2 &&
      bullet.y < target.y + target.h/2 + bh/2
    );
  }

  // ---------- update ----------
  function update(dt){
    for (const s of state.stars){
      s.y += s.s * dt;
      if (s.y > viewH()){
        s.y = -3;
        s.x = Math.random() * viewW();
      }
    }

    for (const sw of state.shockwaves){
      sw.r += 140 * dt;
      sw.life -= dt;
    }
    state.shockwaves = state.shockwaves.filter(sw => sw.life > 0);

    for (const item of state.pickups){
      item.y += item.vy * dt;
      item.spin += dt * 6;
    }
    state.pickups = state.pickups.filter(i => i.y < viewH() + 24);

    for (const part of state.particles){
      part.x += part.vx * dt;
      part.y += part.vy * dt;
      part.vx *= 0.985;
      part.vy *= 0.985;
      part.life -= dt;
    }
    state.particles = state.particles.filter(p => p.life > 0);

    if (state.screenFlash > 0) state.screenFlash -= dt;
    if (state.shake > 0) state.shake = Math.max(0, state.shake - dt * 1.6);
    if (state.messageTimer > 0) state.messageTimer -= dt;

    if (state.player){
      if (state.player.cooldown > 0) state.player.cooldown -= dt;
      if (state.player.invincible > 0) state.player.invincible -= dt;
      if (state.player.flash > 0) state.player.flash -= dt;
    }

    if (!state.running || state.paused) return;

    const p = state.player;

    const move = (state.input.left ? -1 : 0) + (state.input.right ? 1 : 0);
    p.x += move * p.speed * dt;
    p.x = clamp(p.x, p.w/2 + 8, viewW() - p.w/2 - 8);

    if (state.input.fire) spawnPlayerBullet();

    for (const b of state.playerBullets){
      b.x += (b.vx || 0) * dt;
      b.y += b.vy * dt;
    }
    state.playerBullets = state.playerBullets.filter(b => b.y > -30 && b.x > -50 && b.x < viewW()+50);

    for (const b of state.enemyBullets){
      if (b.acc) b.vy += b.acc * dt;
      b.x += (b.vx || 0) * dt;
      b.y += b.vy * dt;
    }
    state.enemyBullets = state.enemyBullets.filter(b => b.y < viewH() + 40 && b.x > -70 && b.x < viewW()+70);

    const playerBox = { x:p.x, y:p.y, w:p.w, h:p.h };

    if (state.phase === "wave"){
      const stage = stageDefs[state.stageIndex];
      const diff = difficultyDefs[state.difficulty];
      state.enemyMoveTimer += dt;

      if (state.enemyMoveTimer >= stage.moveInterval / diff.enemySpeed){
        state.enemyMoveTimer = 0;

        let minX = Infinity, maxX = -Infinity;
        for (const e of state.enemies){
          if (!e.alive) continue;
          minX = Math.min(minX, e.x - e.w/2);
          maxX = Math.max(maxX, e.x + e.w/2);
        }

        const edgePadding = 14;
        let hitEdge = false;
        if (maxX + stage.enemySpeed * diff.enemySpeed * state.moveDir > viewW() - edgePadding) hitEdge = true;
        if (minX + stage.enemySpeed * diff.enemySpeed * state.moveDir < edgePadding) hitEdge = true;

        if (hitEdge){
          state.moveDir *= -1;
          for (const e of state.enemies) if (e.alive) e.y += stage.moveDrop;
        } else {
          for (const e of state.enemies) if (e.alive) e.x += stage.enemySpeed * diff.enemySpeed * state.moveDir;
        }
      }

      const living = state.enemies.filter(e => e.alive);

      for (const e of living){
        if (Math.random() < stage.fireChance * diff.enemyFire){
          const sameColumnFront = living.some(other => other !== e && Math.abs(other.x - e.x) < 10 && other.y > e.y);
          if (!sameColumnFront || Math.random() < 0.24) spawnEnemyBullet(e);
        }
      }

      for (const bullet of state.playerBullets){
        for (const enemy of living){
          if (!enemy.alive) continue;
          if (bulletHitRect(bullet, enemy)){
            enemy.hp -= bullet.dmg;
            bullet.hp -= 1;
            addParticles(bullet.x, bullet.y, "#93ebff", 4, 30);
            if (enemy.hp <= 0) killEnemy(enemy);
            if (bullet.hp <= 0){
              bullet.y = -999;
              break;
            }
          }
        }
      }
      state.playerBullets = state.playerBullets.filter(b => b.y > -100);

      for (const e of living){
        if (e.y + e.h/2 > p.y - 8){
          state.running = false;
          state.gameOver = true;
          finishRun(false);
          showGameOverPanel();
          break;
        }
      }
    } else if (state.phase.startsWith("boss") && state.boss){
      const b = state.boss;
      b.x += b.dir * b.speed * dt;
      if (b.x > viewW() - b.w/2 - 12){ b.x = viewW() - b.w/2 - 12; b.dir = -1; }
      if (b.x < b.w/2 + 12){ b.x = b.w/2 + 12; b.dir = 1; }
      bossFirePattern(dt);

      for (const bullet of state.playerBullets){
        if (bulletHitRect(bullet, b)){
          b.hp -= bullet.dmg;
          bullet.hp -= 1;
          b.shield = 0.08;
          addParticles(bullet.x, bullet.y, "#b2f4ff", 4, 26);
          if (b.hp <= 0){
            bossNextForm();
            break;
          }
          if (bullet.hp <= 0) bullet.y = -999;
        }
      }
      state.playerBullets = state.playerBullets.filter(b => b.y > -100);
    }

    for (const b of state.enemyBullets){
      const box = { x:b.x, y:b.y, w:b.r*2.3, h:b.r*2.3 };
      if (rectsOverlap(playerBox, box)){
        b.y = viewH() + 100;
        hitPlayer();
      }
    }

    for (const item of state.pickups){
      const box = { x:item.x, y:item.y, w:item.w, h:item.h };
      if (rectsOverlap(playerBox, box)){
        item.y = viewH() + 100;
        applyPickup(item.kind);
      }
    }

    updateHUD();
  }

  // ---------- draw ----------
  function roundRect(x,y,w,h,r,fill=false){
    ctx.beginPath();
    ctx.moveTo(x+r,y);
    ctx.arcTo(x+w,y,x+w,y+h,r);
    ctx.arcTo(x+w,y+h,x,y+h,r);
    ctx.arcTo(x,y+h,x,y,r);
    ctx.arcTo(x,y,x+w,y,r);
    if (fill) ctx.fill();
    else ctx.stroke();
  }

  function drawBackground(){
    const hue = stageDefs[Math.min(state.stageIndex, stageDefs.length - 1)].bgHue;
    const grad = ctx.createLinearGradient(0,0,0,viewH());
    grad.addColorStop(0, `hsla(${hue}, 65%, 20%, 0.22)`);
    grad.addColorStop(0.55, `hsla(${(hue+20)%360}, 70%, 12%, 0.12)`);
    grad.addColorStop(1, "rgba(0,0,0,0)");
    ctx.fillStyle = grad;
    ctx.fillRect(0,0,viewW(),viewH());

    for (const n of state.nebula){
      ctx.globalAlpha = n.a;
      ctx.fillStyle = `hsl(${hue}, 80%, 60%)`;
      ctx.beginPath();
      ctx.arc(n.x, n.y, n.r, 0, Math.PI*2);
      ctx.fill();
    }

    ctx.globalAlpha = 0.08;
    ctx.strokeStyle = `hsla(${(hue+40)%360},90%,70%,0.3)`;
    ctx.lineWidth = 1;
    const t = performance.now() * 0.02;
    for (let i=0;i<7;i++){
      const y = (i*76 + t) % (viewH()+80) - 40;
      ctx.beginPath();
      ctx.moveTo(-20, y);
      ctx.lineTo(viewW()+20, y - 40);
      ctx.stroke();
    }
    ctx.globalAlpha = 1;
  }

  function drawStars(){
    for (const s of state.stars){
      ctx.globalAlpha = s.a;
      ctx.fillStyle = "#fff";
      ctx.beginPath();
      ctx.arc(s.x,s.y,s.r,0,Math.PI*2);
      ctx.fill();
    }
    ctx.globalAlpha = 1;
  }

  function drawPlayer(){
    const p = state.player;
    if (!p) return;
    ctx.save();
    ctx.translate(p.x,p.y);

    if (p.invincible > 0 && Math.floor(p.invincible * 12) % 2 === 0){
      ctx.globalAlpha = 0.45;
    }

    if (p.shieldHits > 0){
      ctx.globalAlpha = 0.42 + Math.sin(performance.now()/120) * 0.12;
      ctx.strokeStyle = "#b396ff";
      ctx.lineWidth = 3;
      ctx.beginPath();
      ctx.arc(0,0,24,0,Math.PI*2);
      ctx.stroke();
      ctx.globalAlpha = 1;
    }

    ctx.fillStyle = "#8ce8ff";
    ctx.beginPath();
    ctx.moveTo(0,-18);
    ctx.lineTo(13,12);
    ctx.lineTo(5,8);
    ctx.lineTo(0,17);
    ctx.lineTo(-5,8);
    ctx.lineTo(-13,12);
    ctx.closePath();
    ctx.fill();

    ctx.fillStyle = "#dff8ff";
    ctx.beginPath();
    ctx.moveTo(0,-12);
    ctx.lineTo(5,4);
    ctx.lineTo(-5,4);
    ctx.closePath();
    ctx.fill();

    ctx.fillStyle = "#53c7ff";
    ctx.fillRect(-15,8,30,5);

    ctx.fillStyle = "#ffd166";
    ctx.beginPath();
    ctx.moveTo(-5,17);
    ctx.lineTo(0,28 + Math.random()*3);
    ctx.lineTo(5,17);
    ctx.closePath();
    ctx.fill();

    if (p.flash > 0){
      ctx.globalAlpha = p.flash * 4;
      ctx.strokeStyle = "#ff7b95";
      ctx.lineWidth = 3;
      ctx.beginPath();
      ctx.arc(0,0,26,0,Math.PI*2);
      ctx.stroke();
    }

    ctx.restore();
    ctx.globalAlpha = 1;
  }

  function drawEnemy(e){
    ctx.save();
    ctx.translate(e.x,e.y);
    let body = "#b4befe", wing = "#7aa2ff";
    if (e.type === "guard"){ body = "#ffc857"; wing = "#ff9f1c"; }
    if (e.type === "elite"){ body = "#ff8fa3"; wing = "#ff5d8f"; }
    if (e.type === "sniper"){ body = "#9effd2"; wing = "#2fe0a0"; }

    ctx.fillStyle = wing;
    ctx.beginPath();
    ctx.moveTo(-e.w/2,0);
    ctx.lineTo(-8,-e.h/2);
    ctx.lineTo(-4,0);
    ctx.lineTo(-8,e.h/2-2);
    ctx.closePath();
    ctx.fill();

    ctx.beginPath();
    ctx.moveTo(e.w/2,0);
    ctx.lineTo(8,-e.h/2);
    ctx.lineTo(4,0);
    ctx.lineTo(8,e.h/2-2);
    ctx.closePath();
    ctx.fill();

    ctx.fillStyle = body;
    roundRect(-e.w*0.28,-e.h/2,e.w*0.56,e.h,8,true);

    ctx.fillStyle = "#fff2";
    ctx.fillRect(-e.w*0.16,-e.h*0.18,e.w*0.32,4);

    if (e.type === "sniper"){
      ctx.fillStyle = "#08111e";
      ctx.fillRect(-2,-10,4,20);
    }

    if (e.hp > 1){
      ctx.fillStyle = "rgba(255,255,255,.18)";
      ctx.fillRect(-e.w/2,e.h/2+6,e.w,4);
      ctx.fillStyle = body;
      ctx.fillRect(-e.w/2,e.h/2+6,e.w*(e.hp/e.maxHp),4);
    }

    ctx.restore();
  }

  function drawBoss(){
    const b = state.boss;
    if (!b) return;

    ctx.save();
    ctx.translate(b.x,b.y);

    if (b.shield > 0){
      ctx.globalAlpha = 0.65;
      ctx.strokeStyle = b.kind === "overmind" ? "#ff6bff" : "#91f2ff";
      ctx.lineWidth = 4;
      ctx.beginPath();
      ctx.ellipse(0,0,b.w*0.62,b.h*0.74,0,0,Math.PI*2);
      ctx.stroke();
      ctx.globalAlpha = 1;
    }

    if (b.kind === "dreadnought"){
      if (state.phase === "boss1"){
        ctx.fillStyle = "#6f5bff";
        roundRect(-b.w/2,-b.h/2,b.w,b.h,18,true);
        ctx.fillStyle = "#ff6b8a";
        roundRect(-28,-18,56,36,12,true);
        ctx.fillStyle = "#dff8ff";
        ctx.fillRect(-16,-6,32,6);
      } else {
        ctx.fillStyle = "#3a244f";
        roundRect(-b.w/2,-b.h/2,b.w,b.h,22,true);
        ctx.fillStyle = "#ff4365";
        roundRect(-24,-22,48,44,14,true);
        ctx.fillStyle = "#ffe08a";
        ctx.beginPath();
        ctx.arc(0,0,8,0,Math.PI*2);
        ctx.fill();
      }
    } else {
      if (state.phase === "boss1"){
        ctx.fillStyle = "#7b2cff";
        ctx.beginPath(); ctx.arc(0,0,38,0,Math.PI*2); ctx.fill();
        ctx.fillStyle = "#ff6bff";
        for (let i=0;i<6;i++){
          const a = i*Math.PI/3;
          roundRect(Math.cos(a)*44-7, Math.sin(a)*44-7, 14, 14, 4, true);
        }
      } else if (state.phase === "boss2"){
        ctx.fillStyle = "#4d1f63";
        ctx.beginPath(); ctx.arc(0,0,46,0,Math.PI*2); ctx.fill();
        ctx.fillStyle = "#e74cff";
        for (let i=0;i<8;i++){
          const a = i*Math.PI/4 + performance.now()/1000*0.35;
          roundRect(Math.cos(a)*56-8, Math.sin(a)*56-8, 16, 16, 4, true);
        }
      } else {
        ctx.fillStyle = "#2c0f38";
        ctx.beginPath(); ctx.arc(0,0,54,0,Math.PI*2); ctx.fill();
        ctx.strokeStyle = "#ff6bff";
        ctx.lineWidth = 4;
        for (let i=0;i<3;i++){
          ctx.beginPath();
          ctx.arc(0,0,62+i*10,0,Math.PI*2);
          ctx.stroke();
        }
      }
      ctx.fillStyle = "#fff";
      ctx.beginPath();
      ctx.arc(0,0,10,0,Math.PI*2);
      ctx.fill();
    }
    ctx.restore();

    const barW = Math.min(viewW()-40, 290);
    const x = viewW()/2 - barW/2;
    const y = 74;
    ctx.fillStyle = "rgba(255,255,255,.18)";
    roundRect(x,y,barW,10,6,true);
    ctx.fillStyle = b.kind === "overmind" ? "#ff6bff" : "#ff6b8a";
    roundRect(x,y,barW*(b.hp/b.maxHp),10,6,true);
    ctx.fillStyle = "#fff";
    ctx.font = "700 12px system-ui, sans-serif";
    ctx.textAlign = "center";
    ctx.fillText(b.kind === "overmind" ? `OVERMIND PHASE ${b.form}` : `DREADNOUGHT PHASE ${b.form}`, viewW()/2, 68);
  }

  function drawBullets(){
    for (const b of state.playerBullets){
      ctx.fillStyle = "#78e8ff";
      ctx.shadowColor = "#78e8ff";
      ctx.shadowBlur = 12;
      ctx.fillRect(b.x-2.5,b.y-10,5,18);
    }
    ctx.shadowBlur = 0;

    for (const b of state.enemyBullets){
      ctx.fillStyle = b.type === "core" ? "#ff4365" : b.type === "orb" ? "#ff8fa3" : "#ffd166";
      ctx.shadowColor = ctx.fillStyle;
      ctx.shadowBlur = 10;
      ctx.beginPath();
      ctx.arc(b.x,b.y,b.r,0,Math.PI*2);
      ctx.fill();
    }
    ctx.shadowBlur = 0;
  }

  function drawPickups(){
    const map = { heal:"H", power:"P", shield:"S" };
    for (const item of state.pickups){
      ctx.save();
      ctx.translate(item.x,item.y);
      ctx.rotate(item.spin);
      ctx.fillStyle = item.kind === "heal" ? "#61ffa8" : item.kind === "shield" ? "#b396ff" : "#53c7ff";
      roundRect(-9,-9,18,18,5,true);
      ctx.fillStyle = "#07111f";
      ctx.font = "900 10px system-ui, sans-serif";
      ctx.textAlign = "center";
      ctx.textBaseline = "middle";
      ctx.fillText(map[item.kind],0,1);
      ctx.restore();
    }
  }

  function drawParticles(){
    for (const p of state.particles){
      ctx.globalAlpha = Math.max(0, p.life/p.maxLife);
      ctx.fillStyle = p.color;
      ctx.beginPath();
      ctx.arc(p.x,p.y,p.size,0,Math.PI*2);
      ctx.fill();
    }
    ctx.globalAlpha = 1;
  }

  function drawShockwaves(){
    for (const sw of state.shockwaves){
      ctx.globalAlpha = sw.life/sw.maxLife;
      ctx.strokeStyle = sw.color;
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.arc(sw.x,sw.y,sw.r,0,Math.PI*2);
      ctx.stroke();
    }
    ctx.globalAlpha = 1;
  }

  function drawBanner(){
    if (state.messageTimer <= 0) return;
    ctx.save();
    ctx.globalAlpha = Math.min(1, state.messageTimer);
    ctx.fillStyle = "rgba(6,10,24,.64)";
    roundRect(viewW()/2 - 132, 24, 264, 44, 12, true);
    ctx.fillStyle = "#dff8ff";
    ctx.font = "700 20px system-ui, sans-serif";
    ctx.textAlign = "center";
    let text = stageDefs[Math.min(state.stageIndex, stageDefs.length-1)].name;
    if (state.phase === "boss1") text = "BOSS BATTLE";
    if (state.phase === "boss2") text = "SECOND FORM";
    if (state.phase === "boss3") text = "FINAL FORM";
    ctx.fillText(text, viewW()/2, 52);
    ctx.restore();
  }

  function drawBottomInfo(){
    ctx.save();
    ctx.fillStyle = "rgba(0,0,0,.25)";
    roundRect(12, viewH()-42, 190, 26, 10, true);
    ctx.fillStyle = "#fff";
    ctx.font = "700 12px system-ui, sans-serif";
    ctx.textAlign = "left";
    ctx.fillText(`Best ${state.meta.bestScore}  Coins ${state.meta.totalCoins}`, 22, viewH()-24);
    ctx.restore();
  }

  function drawPause(){
    if (!state.paused) return;
    ctx.save();
    ctx.fillStyle = "rgba(0,0,0,.38)";
    ctx.fillRect(0,0,viewW(),viewH());
    ctx.fillStyle = "#fff";
    ctx.font = "900 28px system-ui, sans-serif";
    ctx.textAlign = "center";
    ctx.fillText("PAUSED", viewW()/2, viewH()/2);
    ctx.restore();
  }

  function drawFlash(){
    if (state.screenFlash <= 0) return;
    ctx.save();
    ctx.globalAlpha = state.screenFlash;
    ctx.fillStyle = "#bfefff";
    ctx.fillRect(0,0,viewW(),viewH());
    ctx.restore();
  }

  function render(){
    ctx.clearRect(0,0,viewW(),viewH());
    ctx.save();
    if (state.shake > 0){
      ctx.translate((Math.random()-0.5)*16*state.shake, (Math.random()-0.5)*16*state.shake);
    }
    drawBackground();
    drawStars();
    for (const e of state.enemies) if (e.alive) drawEnemy(e);
    if (state.boss) drawBoss();
    drawPlayer();
    drawBullets();
    drawPickups();
    drawParticles();
    drawShockwaves();
    drawBanner();
    drawBottomInfo();
    ctx.restore();
    drawPause();
    drawFlash();
  }

  function loop(ts){
    if (!state.lastTs) state.lastTs = ts;
    let dt = (ts - state.lastTs) / 1000;
    state.lastTs = ts;
    dt = Math.min(dt, 0.033);
    update(dt);
    render();
    requestAnimationFrame(loop);
  }

  // ---------- controls ----------
  function bindHoldButton(el, key){
    const on = (ev) => {
      ev.preventDefault();
      ensureAudio();
      state.input[key] = true;
      el.classList.add("active");
    };
    const off = (ev) => {
      ev.preventDefault();
      state.input[key] = false;
      el.classList.remove("active");
    };
    el.addEventListener("pointerdown", on);
    el.addEventListener("pointerup", off);
    el.addEventListener("pointercancel", off);
    el.addEventListener("pointerleave", (ev) => {
      if (ev.pressure === 0) off(ev);
    });
  }

  bindHoldButton(ui.leftBtn, "left");
  bindHoldButton(ui.rightBtn, "right");
  bindHoldButton(ui.fireBtn, "fire");

  ui.pauseBtn.addEventListener("click", () => {
    if (!state.player) return;
    if (!ui.overlay.classList.contains("hidden") && !state.running) return;
    if (state.gameOver || state.win) return;
    state.paused = !state.paused;
    ui.pauseBtn.textContent = state.paused ? "Resume" : "Pause";
  });

  canvas.addEventListener("pointerdown", (ev) => {
    if (!state.running || state.paused) return;
    state.input.pointerActive = true;
    movePlayerToPointer(ev);
  });
  canvas.addEventListener("pointermove", (ev) => {
    if (!state.input.pointerActive || !state.running || state.paused) return;
    movePlayerToPointer(ev);
  });
  const endPointer = () => state.input.pointerActive = false;
  canvas.addEventListener("pointerup", endPointer);
  canvas.addEventListener("pointercancel", endPointer);

  function movePlayerToPointer(ev){
    const rect = canvas.getBoundingClientRect();
    const x = ev.clientX - rect.left;
    state.player.x = clamp(x, state.player.w/2 + 8, viewW() - state.player.w/2 - 8);
  }

  window.addEventListener("keydown", (e) => {
    ensureAudio();
    if (e.key === "ArrowLeft") state.input.left = true;
    if (e.key === "ArrowRight") state.input.right = true;
    if (e.key === " " || e.key === "Enter") state.input.fire = true;
    if (e.key.toLowerCase() === "p"){
      state.paused = !state.paused;
      ui.pauseBtn.textContent = state.paused ? "Resume" : "Pause";
    }
  });
  window.addEventListener("keyup", (e) => {
    if (e.key === "ArrowLeft") state.input.left = false;
    if (e.key === "ArrowRight") state.input.right = false;
    if (e.key === " " || e.key === "Enter") state.input.fire = false;
  });

  // ---------- boot ----------
  window.addEventListener("resize", resize);
  document.addEventListener("visibilitychange", () => {
    if (document.hidden) state.paused = true;
  });

  loadMeta();
  resize();
  showTitleScreen();
  requestAnimationFrame(loop);
})();
</script>
</body>
</html>
