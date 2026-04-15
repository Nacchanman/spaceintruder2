<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover, user-scalable=no" />
<title>SPACE INTRUDER</title>
<meta name="theme-color" content="#0b1020" />
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
<link rel="manifest" href="space_intruder_manifest.webmanifest" />
<link rel="apple-touch-icon" href="space_intruder_icon.svg" />
<style>
  :root{
    --bg1:#040713; --bg2:#0e1a3a; --panel:rgba(9,14,31,.93); --line:rgba(255,255,255,.12);
    --text:#eef4ff; --accent:#53c7ff; --danger:#ff6b8a; --good:#61ffa8; --warn:#ffd166; --purple:#9b7bff;
  }
  *{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
  html,body{
    margin:0;padding:0;width:100%;height:100%;overflow:hidden;
    background:
      radial-gradient(circle at 50% 20%, rgba(84,132,255,.16), transparent 35%),
      linear-gradient(180deg,var(--bg2),var(--bg1));
    color:var(--text);font-family:system-ui,-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif;
    touch-action:none
  }
  #app{
    position:fixed;inset:0;display:flex;flex-direction:column;
    width:min(100vw, calc((100vh - 8px) * 0.62), 430px);
    min-width:320px;
    margin:0 auto
  }
  #hud{
    display:grid;grid-template-columns:repeat(7,1fr);gap:8px;
    padding:calc(env(safe-area-inset-top,0px) + 10px) 12px 10px;z-index:3;pointer-events:none;
    font-size:11px;font-weight:800;letter-spacing:.02em
  }
  .hud-box{
    background:rgba(7,12,28,.66);border:1px solid var(--line);border-radius:12px;padding:8px 6px;min-width:0;
    text-align:center;backdrop-filter:blur(6px);box-shadow:0 8px 24px rgba(0,0,0,.18)
  }
  #gameWrap{position:relative;flex:1;min-height:0;padding:0 10px}
  canvas{
    width:100%;height:100%;display:block;border-radius:18px;border:1px solid rgba(255,255,255,.08);
    background:
      radial-gradient(circle at 50% 5%, rgba(255,255,255,.08), transparent 20%),
      linear-gradient(180deg, rgba(255,255,255,.03), rgba(255,255,255,0)),
      #030712;
    box-shadow:0 16px 40px rgba(0,0,0,.28)
  }
  #controls{
    display:grid;
    grid-template-columns:1fr 1fr;
    grid-template-areas:
      "left right"
      "pause fire"
      "pause fire";
    gap:8px;
    padding:10px 10px calc(env(safe-area-inset-bottom,0px) + 12px);
    z-index:3
  }
  button.ctrl, #panel button, .shop-btn, .menu-btn{
    appearance:none;border:1px solid rgba(255,255,255,.12);border-bottom-width:3px;color:var(--text);
    border-radius:18px;min-height:60px;font-size:18px;font-weight:900;letter-spacing:.03em;
    touch-action:manipulation;user-select:none;box-shadow:0 10px 18px rgba(0,0,0,.22)
  }
  button.ctrl{background:linear-gradient(180deg, rgba(255,255,255,.12), rgba(255,255,255,.04))}
  button.ctrl:active,button.ctrl.active{
    transform:translateY(1px) scale(.985);
    background:linear-gradient(180deg, rgba(83,199,255,.22), rgba(83,199,255,.08))
  }
  #fireBtn{color:#07111f;background:linear-gradient(180deg,#ffd166,#ffb703);border-color:rgba(255,220,120,.45)}
  #fireBtn:active,#fireBtn.active{background:linear-gradient(180deg,#ffe08a,#ffc43c)}
  #skillBtn{font-size:14px;background:linear-gradient(180deg,#b396ff,#7f6aff)}
  #pauseBtn{font-size:15px}
  #overlay{
    position:absolute;inset:0;display:flex;align-items:center;justify-content:center;padding:16px;z-index:4;pointer-events:none
  }
  #panel{
    width:min(100%,460px);max-height:92%;overflow:auto;background:var(--panel);border:1px solid var(--line);
    border-radius:20px;padding:20px 18px;box-shadow:0 18px 44px rgba(0,0,0,.32);text-align:center;pointer-events:auto;
    backdrop-filter:blur(8px)
  }
  #panel h1,#panel h2,#panel h3{margin:0 0 10px;line-height:1.2}
  #panel p{margin:8px 0;line-height:1.55;font-size:14px;color:#dbe7ff}
  #panel .small{font-size:13px;opacity:.92}
  #panel button,.menu-btn{margin-top:12px;min-width:180px;min-height:50px;color:#08111e;background:linear-gradient(180deg,#91f2ff,#53c7ff)}
  .hidden{display:none!important}
  .accent{color:var(--accent)} .danger{color:var(--danger)} .good{color:var(--good)} .warn{color:var(--warn)} .purple{color:var(--purple)}
  .shop-grid,.menu-grid{display:grid;grid-template-columns:1fr;gap:10px;margin-top:10px;text-align:left}
  .shop-item,.menu-item{border:1px solid rgba(255,255,255,.12);border-radius:14px;padding:12px;background:rgba(255,255,255,.04)}
  .shop-row,.menu-row{display:flex;justify-content:space-between;align-items:center;gap:10px}
  .shop-btn,.menu-btn{min-height:42px;border-radius:12px;font-size:14px;padding:0 12px}
  .shop-btn{background:linear-gradient(180deg,#ffe08a,#ffc43c);color:#08111e}
  .menu-btn.alt{background:linear-gradient(180deg,#d8d8ff,#9b7bff);color:#08111e}

@media (min-aspect-ratio: 1/1){
  #app{
    width:min(420px, calc(100vh * 0.60), 100vw);
  }
}
@media (max-width: 420px){
  #hud{gap:6px;font-size:10px;padding:calc(env(safe-area-inset-top,0px) + 8px) 8px 8px}
  .hud-box{padding:7px 4px;border-radius:10px}
  #gameWrap{padding:0 6px}
  #controls{gap:6px;padding:8px 8px calc(env(safe-area-inset-bottom,0px) + 10px)}
  button.ctrl{min-height:54px;font-size:16px}
  #fireBtn{min-height:114px;font-size:20px}
  #pauseBtn{font-size:13px}
}

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
    <div id="overlay">
      <div id="panel"></div>
    </div>
  </div>

  <div id="controls">
    <button id="leftBtn" class="ctrl" aria-label="左へ">◀</button>
    <button id="rightBtn" class="ctrl" aria-label="右へ">▶</button>
    <button id="fireBtn" class="ctrl" aria-label="発射">Fire</button>
    
    <button id="pauseBtn" class="ctrl" aria-label="ポーズ">Pause</button>
  </div>
</div>

<script>
(() => {
  const STORAGE_KEY = "space_intruder_infinity_save_v1";
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  const ui = {
    stageText: document.getElementById('stageText'),
    lifeText: document.getElementById('lifeText'),
    scoreText: document.getElementById('scoreText'),
    coinText: document.getElementById('coinText'),
    powerText: document.getElementById('powerText'),
    skillText: document.getElementById('skillText'),
    modeText: document.getElementById('modeText'),
    overlay: document.getElementById('overlay'),
    panel: document.getElementById('panel'),
    leftBtn: document.getElementById('leftBtn'),
    rightBtn: document.getElementById('rightBtn'),
    fireBtn: document.getElementById('fireBtn'),
    skillBtn: document.getElementById('skillBtn'),
    pauseBtn: document.getElementById('pauseBtn'),
    gameWrap: document.getElementById('gameWrap')
  };

  const DPR = Math.max(1, Math.min(window.devicePixelRatio || 1, 2));
  const difficultyDefs = {
    easy:   { name:"EASY",   enemyFire:0.82, enemySpeed:0.92, enemyHP:0.92, reward:0.9, skillGain:1.18, playerHP:1 },
    normal: { name:"NORMAL", enemyFire:1.00, enemySpeed:1.00, enemyHP:1.00, reward:1.0, skillGain:1.00, playerHP:0 },
    hard:   { name:"HARD",   enemyFire:1.22, enemySpeed:1.12, enemyHP:1.16, reward:1.2, skillGain:0.9,  playerHP:-1 }
  };

  const stageDefs = [
    {name:"STAGE 1", subtitle:"前哨宙域", rows:3, cols:5, spacingX:58, spacingY:52, enemyHP:[1], fireChance:0.006, enemySpeed:18, moveDrop:18, moveInterval:1.10, types:["scout"], boss:null, bgHue:210},
    {name:"STAGE 2", subtitle:"防衛ライン", rows:4, cols:6, spacingX:54, spacingY:48, enemyHP:[1,2], fireChance:0.009, enemySpeed:24, moveDrop:20, moveInterval:0.86, types:["scout","guard"], boss:null, bgHue:225},
    {name:"STAGE 3", subtitle:"旗艦迎撃", rows:4, cols:6, spacingX:52, spacingY:45, enemyHP:[1,2,3], fireChance:0.013, enemySpeed:30, moveDrop:22, moveInterval:0.68, types:["scout","guard","elite"], boss:"dreadnought", bgHue:245},
    {name:"STAGE 4", subtitle:"火星軌道", rows:5, cols:7, spacingX:46, spacingY:42, enemyHP:[2,2,3], fireChance:0.015, enemySpeed:34, moveDrop:22, moveInterval:0.62, types:["guard","elite"], boss:null, bgHue:12},
    {name:"STAGE 5", subtitle:"機械要塞", rows:5, cols:7, spacingX:46, spacingY:40, enemyHP:[2,3,3], fireChance:0.018, enemySpeed:36, moveDrop:24, moveInterval:0.57, types:["scout","guard","elite","sniper"], boss:null, bgHue:160},
    {name:"STAGE 6", subtitle:"コアネスト", rows:5, cols:7, spacingX:46, spacingY:40, enemyHP:[2,3,4], fireChance:0.020, enemySpeed:38, moveDrop:24, moveInterval:0.52, types:["guard","elite","sniper"], boss:"overmind", bgHue:300}
  ];

  const state = {
    running:false, paused:false, gameOver:false, win:false,
    stageIndex:0, phase:"menu", difficulty:"normal",
    score:0, coins:0, combo:1, comboTimer:0,
    stars:[], nebula:[], particles:[], shockwaves:[], pickups:[],
    enemies:[], playerBullets:[], enemyBullets:[], boss:null,
    moveDir:1, enemyMoveTimer:0, lastTs:0, messageTimer:0, screenFlash:0,
    shake:0, cutinTimer:0, cutinText:"", deferredPrompt:null, canInstall:false,
    input:{left:false,right:false,fire:false,pointerActive:false},
    player:null,
    runStats:{kills:0, pickups:0, skills:0, bosses:0, startAt:0},
    meta:{
      bestScore:0, totalCoins:0, soundOn:true,
      permHP:0, permRapid:0, permSkillRate:0, permPower:0
    }
  };

  // -------- audio ----------
  let audioCtx = null;
  let activeMusicOsc = null;
  let activeMusicGain = null;

  function stopMusicVoice(){
    try { if (activeMusicOsc) activeMusicOsc.stop(); } catch (e) {}
    try { if (activeMusicOsc) activeMusicOsc.disconnect(); } catch (e) {}
    try { if (activeMusicGain) activeMusicGain.disconnect(); } catch (e) {}
    activeMusicOsc = null;
    activeMusicGain = null;
  }

  function ensureAudio(){
    if (!state.meta.soundOn) return;
    if (!audioCtx){
      const Ctx = window.AudioContext || window.webkitAudioContext;
      if (Ctx) audioCtx = new Ctx();
    }
    if (audioCtx && audioCtx.state === "suspended") audioCtx.resume();
  }

  function beep(freq=440, dur=0.05, type="square", vol=0.03, slide=1, channel="sfx"){
    if (!state.meta.soundOn) return;
    ensureAudio();
    if (!audioCtx) return;
    const t = audioCtx.currentTime;

    if (channel === "music"){
      stopMusicVoice();
    }

    const o = audioCtx.createOscillator();
    const g = audioCtx.createGain();
    o.type = type;
    o.frequency.setValueAtTime(freq, t);
    o.frequency.exponentialRampToValueAtTime(Math.max(60, freq*slide), t + dur);
    g.gain.setValueAtTime(0.0001, t);
    g.gain.exponentialRampToValueAtTime(vol, t + 0.01);
    g.gain.exponentialRampToValueAtTime(0.0001, t + dur);
    o.connect(g).connect(audioCtx.destination);
    o.start(t);
    o.stop(t + dur + 0.02);

    if (channel === "music"){
      activeMusicOsc = o;
      activeMusicGain = g;
      o.onended = () => {
        if (activeMusicOsc === o) {
          activeMusicOsc = null;
          activeMusicGain = null;
        }
      };
    }
  }
  function playSfx(name){
    if (name === "shot") beep(830, 0.05, "square", 0.015, 0.84);
    if (name === "enemyShot") beep(220, 0.08, "sawtooth", 0.02, 0.95);
    if (name === "hit") beep(180, 0.07, "square", 0.03, 0.65);
    if (name === "pickup") beep(660, 0.10, "triangle", 0.04, 1.2);
    if (name === "skill") { beep(300, 0.12, "sawtooth", 0.05, 1.4); setTimeout(() => beep(620,0.15,"triangle",0.04,1.2), 50); }
    if (name === "boss") { beep(110,0.18,"sawtooth",0.05,0.8); setTimeout(() => beep(92,0.22,"square",0.04,0.7), 90); }
    if (name === "clear") { beep(440,0.08,"triangle",0.04,1.3); setTimeout(() => beep(660,0.12,"triangle",0.04,1.3),80); setTimeout(() => beep(990,0.14,"triangle",0.04,1.2),170); }
  }

  let musicTimer = 0;
  const musicPatterns = {
    menu:  [392, 523, 659, 523, 392, 523, 784, 659],
    stage: [262, 330, 392, 330, 294, 349, 440, 349],
    boss:  [196, 196, 294, 196, 175, 175, 262, 175],
    clear: [392, 523, 659, 784, 659, 523, 988]
  };
  function tickMusic(dt){
    if (!state.meta.soundOn) return;
    musicTimer -= dt;
    if (musicTimer > 0) return;

    let pattern = musicPatterns.menu;
    let stepTime = 0.24;

    if (state.running && (state.phase === "wave" || state.phase === "boss1" || state.phase === "boss2" || state.phase === "boss3")){
      pattern = state.phase.startsWith("boss") ? musicPatterns.boss : musicPatterns.stage;
      stepTime = state.phase.startsWith("boss") ? 0.18 : 0.22;
    } else if (state.win){
      pattern = musicPatterns.clear;
      stepTime = 0.2;
    } else if (state.gameOver){
      stepTime = 0.32;
    }

    state._musicStep = (state._musicStep || 0) + 1;
    const note = pattern[state._musicStep % pattern.length];
    beep(note, 0.09, "triangle", 0.012, 1.02, "music");
    musicTimer = stepTime;
  }

  // -------- helpers ----------
  function loadMeta(){
    try{
      const raw = localStorage.getItem(STORAGE_KEY);
      if (!raw) return;
      const data = JSON.parse(raw);
      if (data && typeof data === "object"){
        Object.assign(state.meta, data);
      }
    }catch(e){}
  }
  function saveMeta(){
    try{ localStorage.setItem(STORAGE_KEY, JSON.stringify(state.meta)); }catch(e){}
  }
  function viewW(){ return canvas.width / DPR; }
  function viewH(){ return canvas.height / DPR; }
  function clamp(v,min,max){ return Math.max(min, Math.min(max, v)); }
  function rand(min,max){ return Math.random() * (max-min) + min; }

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
      state.player.x = clamp(state.player.x, state.player.w/2+8, viewW()-state.player.w/2-8);
      state.player.y = Math.max(160, viewH()-74);
    }
  }

  function initBackground(){
    state.stars = Array.from({length:95}, () => ({
      x:Math.random()*viewW(), y:Math.random()*viewH(), r:Math.random()*2+.4, s:Math.random()*24+8, a:Math.random()*.55+.2
    }));
    state.nebula = Array.from({length:4}, (_,i)=>({
      x:rand(40, viewW()-40), y:rand(40, viewH()*0.7), r:rand(50,100), a:0.06 + i*0.01
    }));
  }

  function createPlayer(){
    const diff = difficultyDefs[state.difficulty];
    state.player = {
      x:viewW()/2, y:Math.max(160, viewH()-74), w:34, h:38, speed:250,
      cooldown:0, hp:Math.max(1, 3 + state.meta.permHP + diff.playerHP), maxHp:Math.max(1, 5 + state.meta.permHP + diff.playerHP),
      invincible:0, flash:0, power:1 + state.meta.permPower, rapid:0, /* disabled */ skill:0, skillCost:100,
      pierce:0, /* disabled */ sideDrone:false, shieldHits:0
    };
    state.player.power = Math.min(5, state.player.power);
  }

  function resetRun(){
    state.running = true; state.paused = false; state.gameOver = false; state.win = false;
    state.stageIndex = 0; state.phase = "wave"; state.score = 0; state.coins = 0; state.combo = 1; state.comboTimer = 0;
    state.particles = []; state.playerBullets = []; state.enemyBullets = []; state.pickups = []; state.shockwaves = [];
    state.boss = null; state.screenFlash = 0; state._musicStep = 0;
    state.shake = 0; state.cutinTimer = 0; state.cutinText = "";
    state.runStats = {kills:0, pickups:0, skills:0, bosses:0, startAt:performance.now()};
    createPlayer(); loadStage(0);
    ui.overlay.classList.add("hidden");
    ui.pauseBtn.textContent = "Pause";
    updateHUD();
  }

  function updateHUD(){
    ui.stageText.textContent = String(state.stageIndex + 1);
    ui.lifeText.textContent = String(state.player ? state.player.hp : 0);
    ui.scoreText.textContent = String(state.score);
    ui.coinText.textContent = String(state.coins);
    ui.powerText.textContent = String(state.player ? state.player.power : 1);
    
    ui.modeText.textContent = difficultyDefs[state.difficulty].name;
  }

  function effectiveHP(base){
    return Math.max(1, Math.round(base * difficultyDefs[state.difficulty].enemyHP));
  }


  function getStageLayout(stage){
    const topMargin = 60; // 上に寄せる
    const bottomReserve = 200; // 下の安全距離を広げる
    const available = Math.max(120, viewH() - topMargin - bottomReserve);
    const desiredHeight = Math.max(0, (stage.rows - 1) * stage.spacingY);
    const maxHeight = Math.max(80, available * 0.55); // 縦間隔を広げる
    const spacingScale = desiredHeight > 0 ? Math.min(1, maxHeight / desiredHeight) : 1;
    const spacingY = Math.max(34, stage.spacingY * spacingScale);

    const desiredWidth = Math.max(0, (stage.cols - 1) * stage.spacingX);
    const maxWidth = Math.max(140, viewW() - 80);
    const spacingScaleX = desiredWidth > 0 ? Math.min(1, maxWidth / desiredWidth) : 1;
    const spacingX = Math.max(30, stage.spacingX * spacingScaleX);

    const formationHeight = (stage.rows - 1) * spacingY;
    const startY = Math.max(40, Math.min(70, topMargin)); // 常に上寄りに配置

    return { spacingX, spacingY, startY };
  }

  function loadStage(index){
    const stage = stageDefs[index];
    state.stageIndex = index;
    state.phase = "wave";
    state.enemies = []; state.enemyBullets = []; state.playerBullets = []; state.pickups = [];
    state.moveDir = 1; state.enemyMoveTimer = 0; state.messageTimer = 2.2; state.boss = null;

    const layout = getStageLayout(stage);
    const totalWidth = (stage.cols - 1) * layout.spacingX;
    const startX = viewW()/2 - totalWidth/2;
    const startY = layout.startY;

    for (let r=0;r<stage.rows;r++){
      for (let c=0;c<stage.cols;c++){
        const type = stage.types[(r+c+index)%stage.types.length];
        let hp = type==="sniper" ? 2 : type==="elite" ? 3 : type==="guard" ? 2 : 1;
        hp = Math.max(hp, stage.enemyHP[Math.min(stage.enemyHP.length-1, (r+c)%stage.enemyHP.length)]);
        hp = effectiveHP(hp);
        state.enemies.push({
          x:startX + c*layout.spacingX, y:startY + r*layout.spacingY,
          w:type==="elite"?38:type==="guard"?34:type==="sniper"?30:30,
          h:type==="elite"?28:type==="sniper"?26:24,
          type, hp, maxHp:hp, alive:true
        });
      }
    }
    updateHUD();
  }

  function spawnBoss(kind){
    playSfx("boss");
    state.enemyBullets = []; state.playerBullets = []; state.pickups = [];
    const bossY = Math.max(72, Math.min(110, viewH() * 0.18));
    if (kind === "dreadnought"){
      state.phase = "boss1";
      state.boss = {kind, form:1, x:viewW()/2, y:bossY, w:126, h:58, hp:effectiveHP(42), maxHp:effectiveHP(42), dir:1, speed:88, fireTimer:0, patternTimer:0, shield:0, dashTimer:0};
    } else {
      state.phase = "boss1";
      state.boss = {kind, form:1, x:viewW()/2, y:bossY, w:88, h:88, hp:effectiveHP(58), maxHp:effectiveHP(58), dir:1, speed:102, fireTimer:0, patternTimer:0, shield:0, dashTimer:0};
    }
    state.messageTimer = 2.8;
    triggerCutin(kind === "overmind" ? "WARNING - OVERMIND" : "WARNING - DREADNOUGHT");
  }

  function bossNextForm(){
    if (!state.boss) return;
    const kind = state.boss.kind;
    if (kind === "dreadnought"){
      if (state.phase === "boss1"){
        state.phase = "boss2";
        state.boss = {kind, form:2, x:viewW()/2, y:Math.max(72, Math.min(104, viewH() * 0.18)), w:98, h:68, hp:effectiveHP(54), maxHp:effectiveHP(54), dir:1, speed:118, fireTimer:0, patternTimer:0, shield:0, dashTimer:0};
      } else {
        killBoss();
        return;
      }
    } else if (kind === "overmind"){
      if (state.phase === "boss1"){
        state.phase = "boss2";
        state.boss = {kind, form:2, x:viewW()/2, y:Math.max(72, Math.min(100, viewH() * 0.18)), w:110, h:110, hp:effectiveHP(62), maxHp:effectiveHP(62), dir:1, speed:112, fireTimer:0, patternTimer:0, shield:0, dashTimer:0};
      } else if (state.phase === "boss2"){
        state.phase = "boss3";
        state.boss = {kind, form:3, x:viewW()/2, y:Math.max(70, Math.min(96, viewH() * 0.17)), w:124, h:124, hp:effectiveHP(72), maxHp:effectiveHP(72), dir:1, speed:130, fireTimer:0, patternTimer:0, shield:0, dashTimer:0};
      } else {
        killBoss();
        return;
      }
    }
    state.enemyBullets = [];
    state.playerBullets = [];
    state.screenFlash = 0.45;
    state.messageTimer = 2.6;
    triggerCutin(state.boss.kind === "overmind" ? `OVERMIND PHASE ${state.boss.form}` : `DREADNOUGHT PHASE ${state.boss.form}`);
    addParticles(viewW()/2, 108, "#ff6b8a", 100, 190);
    addShockwave(viewW()/2, 108, 24, "#ff6b8a");
    playSfx("boss");
  }

  function addParticles(x,y,color,count=10,power=70){
    const limit = 260;
    const room = Math.max(0, limit - state.particles.length);
    const actual = Math.min(count, room);
    for(let i=0;i<actual;i++){
      const a = Math.random()*Math.PI*2;
      const s = Math.random()*power;
      const p = {x,y,vx:Math.cos(a)*s,vy:Math.sin(a)*s,life:Math.random()*0.35+0.25,maxLife:0,color,size:Math.random()*3+1.4};
      p.maxLife = p.life;
      state.particles.push(p);
    }
  }
  function addShockwave(x,y,r=10,color="#ffffff"){
    state.shockwaves.push({x,y,r,life:0.45,maxLife:0.45,color});
  }

  function addScore(base){
    const gain = Math.round(base * state.combo * difficultyDefs[state.difficulty].reward);
    state.score += gain;
    state.combo = Math.min(7, +(state.combo + 0.2).toFixed(1));
    state.comboTimer = 4;
    
    updateHUD();
  }
  function gainCoins(n){
    n = Math.max(1, Math.round(n * difficultyDefs[state.difficulty].reward));
    state.coins += n;
    state.meta.totalCoins += n;
    saveMeta();
    updateHUD();
  }
  function decayCombo(dt){
    if (state.comboTimer > 0) state.comboTimer -= dt;
    else if (state.combo > 1) state.combo = Math.max(1, +(state.combo - dt*0.25).toFixed(1));
  }

  function spawnPickup(x,y){
    const roll = Math.random();
    let kind = null;
    if (roll < 0.12) kind = "power";
    else if (roll < 0.20) kind = "heal";
    else if (roll < 0.28) kind = "rapid";
    else if (roll < 0.35) kind = "shield";
    else if (roll < 0.42) kind = "pierce";
    
    if (!kind) return;
    state.pickups.push({x,y,w:18,h:18,vy:58,kind,spin:0});
  }

  function applyPickup(kind){
    const p = state.player;
    if (kind === "power") p.power = Math.min(5, p.power + 1);
    if (kind === "heal") p.hp = Math.min(p.maxHp, p.hp + 1);
    /* rapid removed */
    if (kind === "shield") p.shieldHits = Math.min(3, p.shieldHits + 1);
    /* pierce removed */
    /* skill removed */
    state.runStats.pickups++;
    addParticles(p.x, p.y-10, kind==="shield" ? "#b396ff" : kind==="heal" ? "#61ffa8" : "#ffd166", 12, 90);
    playSfx("pickup");
    updateHUD();
  }

  function spawnPlayerBullet(){
    const p = state.player;
    if (p.cooldown > 0 || !state.running || state.paused) return;
    const bullets = [];
    const speed = -440;
    const pierce = false;
    bullets.push({x:p.x,y:p.y-p.h*0.55,vy:speed,r:3.6,dmg:1,hp:pierce?3:1,beam:pierce});
    if (p.power >= 2){
      bullets.push({x:p.x-12,y:p.y-p.h*0.2,vy:speed,r:3.2,dmg:1,hp:pierce?2:1,beam:pierce});
      bullets.push({x:p.x+12,y:p.y-p.h*0.2,vy:speed,r:3.2,dmg:1,hp:pierce?2:1,beam:pierce});
    }
    if (p.power >= 3){
      bullets.push({x:p.x-22,y:p.y,vx:-40,vy:speed+15,r:3,dmg:1,hp:1,beam:false});
      bullets.push({x:p.x+22,y:p.y,vx:40,vy:speed+15,r:3,dmg:1,hp:1,beam:false});
    }
    if (p.power >= 4){
      bullets.push({x:p.x,y:p.y-p.h*0.85,vy:speed-30,r:4.2,dmg:2,hp:pierce?3:1,beam:true});
    }
    if (p.power >= 5){
      bullets.push({x:p.x-28,y:p.y-6,vy:speed,r:3.0,dmg:1,hp:1,beam:false});
      bullets.push({x:p.x+28,y:p.y-6,vy:speed,r:3.0,dmg:1,hp:1,beam:false});
    }
    if (p.sideDrone){
      bullets.push({x:p.x-36,y:p.y-6,vy:speed,r:2.8,dmg:1,hp:1,beam:false});
      bullets.push({x:p.x+36,y:p.y-6,vy:speed,r:2.8,dmg:1,hp:1,beam:false});
    }
    state.playerBullets.push(...bullets);
    if (state.playerBullets.length > 80) state.playerBullets.splice(0, state.playerBullets.length - 80);
    p.cooldown = 0.22;
    addParticles(p.x, p.y-16, "#78e8ff", 5, 28);
    playSfx("shot");
  }

  function spawnEnemyBullet(enemy){
    const diff = difficultyDefs[state.difficulty];
    const speedBase = (160 + state.stageIndex*16 + (enemy.type==="elite" ? 40 : enemy.type==="guard" ? 18 : enemy.type==="sniper" ? 28 : 0)) * diff.enemySpeed;
    if (enemy.type==="elite" && Math.random() < 0.35){
      state.enemyBullets.push(
        {x:enemy.x-8,y:enemy.y+enemy.h/2,vx:-18,vy:speedBase,r:4.5,type:"orb"},
        {x:enemy.x+8,y:enemy.y+enemy.h/2,vx:18,vy:speedBase,r:4.5,type:"orb"}
      );
    } else if (enemy.type==="guard" && Math.random() < 0.35){
      for(let i=-1;i<=1;i++) state.enemyBullets.push({x:enemy.x,y:enemy.y+enemy.h/2,vx:i*28,vy:speedBase+18,r:4.2,type:"shot"});
    } else if (enemy.type==="sniper"){
      const dx = state.player.x - enemy.x;
      const mag = Math.max(1, Math.abs(dx));
      state.enemyBullets.push({x:enemy.x,y:enemy.y+enemy.h/2,vx:(dx/mag)*42,vy:speedBase+28,r:4.6,type:"core"});
    } else {
      state.enemyBullets.push({x:enemy.x,y:enemy.y+enemy.h/2,vx:0,vy:speedBase,r:enemy.type==="scout"?4:5,type:enemy.type==="scout"?"shot":"orb"});
    }
    if (state.enemyBullets.length > 220) {
      state.enemyBullets.splice(0, state.enemyBullets.length - 220);
    }
    if (Math.random() < 0.35) playSfx("enemyShot");
  }

  function bossFirePattern(dt){
    const b = state.boss;
    if (!b) return;
    b.fireTimer += dt;
    b.patternTimer += dt;
    const diff = difficultyDefs[state.difficulty];

    if (b.kind === "dreadnought"){
      if (state.phase === "boss1"){
        if (b.fireTimer > 0.78 / diff.enemyFire){
          b.fireTimer = 0;
          const mode = Math.floor((b.patternTimer % 6)/2);
          if (mode === 0){
            for(let i=-2;i<=2;i++) state.enemyBullets.push({x:b.x+i*18,y:b.y+22,vx:i*14,vy:185*diff.enemySpeed,r:5,type:"orb"});
          } else if (mode === 1){
            for(let i=-3;i<=3;i++) state.enemyBullets.push({x:b.x,y:b.y+18,vx:i*30,vy:(170+Math.abs(i)*8)*diff.enemySpeed,r:4.5,type:"shot"});
          } else {
            state.enemyBullets.push(
              {x:b.x-34,y:b.y+10,vx:-12,vy:215*diff.enemySpeed,r:6,type:"orb"},
              {x:b.x+34,y:b.y+10,vx:12,vy:215*diff.enemySpeed,r:6,type:"orb"},
              {x:b.x,y:b.y+22,vx:0,vy:235*diff.enemySpeed,r:7,type:"core"}
            );
          }
        }
      } else {
        b.dashTimer += dt;
        if (b.dashTimer > 1.8){ b.dashTimer = 0; b.dir = state.player.x > b.x ? 1 : -1; b.speed = 180; }
        else b.speed += (118 - b.speed) * 0.04;
        if (b.fireTimer > 0.48 / diff.enemyFire){
          b.fireTimer = 0;
          const mode = Math.floor((b.patternTimer % 9)/3);
          if (mode === 0){
            for(let i=0;i<12;i++){
              const a = (Math.PI*2/12)*i;
              state.enemyBullets.push({x:b.x,y:b.y+4,vx:Math.cos(a)*92,vy:Math.sin(a)*92 + 85,r:4.2,type:"shot"});
            }
          } else if (mode === 1){
            for(let i=-4;i<=4;i++) state.enemyBullets.push({x:b.x+i*10,y:b.y+18,vx:i*16,vy:245*diff.enemySpeed,r:4.8,type:"core"});
          } else {
            for(let i=-2;i<=2;i++) state.enemyBullets.push({x:b.x+i*16,y:b.y+14,vx:i*8,vy:150*diff.enemySpeed,r:5.5,type:"orb",acc:70});
          }
          playSfx("enemyShot");
        }
      }
    } else {
      if (state.phase === "boss1"){
        b.x += Math.sin(b.patternTimer*1.3) * 0.9;
        if (b.fireTimer > 0.58 / diff.enemyFire){
          b.fireTimer = 0;
          for(let ring=0; ring<2; ring++){
            for(let i=0;i<10;i++){
              const a = (Math.PI*2/10) * i + ring*0.2 + b.patternTimer*0.4;
              state.enemyBullets.push({x:b.x,y:b.y+6,vx:Math.cos(a)*(68 + ring*20),vy:Math.sin(a)*(68 + ring*20)+70,r:4.2,type:"orb"});
            }
          }
          playSfx("enemyShot");
        }
      } else if (state.phase === "boss2"){
        if (b.fireTimer > 0.45 / diff.enemyFire){
          b.fireTimer = 0;
          const dx = state.player.x - b.x;
          const sign = dx >= 0 ? 1 : -1;
          for(let i=-3;i<=3;i++){
            state.enemyBullets.push({x:b.x,y:b.y,vx:sign*60 + i*10,vy:(150+Math.abs(i)*18)*diff.enemySpeed,r:4.6,type:"core"});
          }
          for(let i=0;i<8;i++){
            const a = i*(Math.PI*2/8);
            state.enemyBullets.push({x:b.x,y:b.y,vx:Math.cos(a)*50,vy:Math.sin(a)*50+110,r:4.1,type:"shot"});
          }
          playSfx("enemyShot");
        }
      } else {
        b.dashTimer += dt;
        if (b.dashTimer > 1.5){
          b.dashTimer = 0;
          b.dir = state.player.x > b.x ? 1 : -1;
          b.speed = 220;
        } else {
          b.speed += (130 - b.speed) * 0.05;
        }
        if (b.fireTimer > 0.34 / diff.enemyFire){
          b.fireTimer = 0;
          for(let i=0;i<16;i++){
            const a = (Math.PI*2/16) * i + b.patternTimer;
            state.enemyBullets.push({x:b.x,y:b.y,vx:Math.cos(a)*96,vy:Math.sin(a)*96+88,r:4.6,type:i%4===0?"core":"orb"});
          }
          for(let i=-2;i<=2;i++){
            state.enemyBullets.push({x:b.x+i*18,y:b.y+28,vx:i*14,vy:230*diff.enemySpeed,r:6.0,type:"core"});
          }
          playSfx("enemyShot");
        }
      }
    }
  }

  function useSkill(){
    return;
  }

  function hitPlayer(){
    const p = state.player;
    if (p.invincible > 0 || !state.running) return;

    if (p.shieldHits > 0){
      p.shieldHits--;
      p.invincible = 0.5;
      addParticles(p.x,p.y,"#b396ff",18,110);
      addShockwave(p.x,p.y,12,"#b396ff");
      updateHUD();
      return;
    }

    p.hp--;
    p.invincible = 1.25;
    p.flash = 0.18;
    state.combo = 1; state.comboTimer = 0;
    state.shake = Math.max(state.shake, 0.28);
    addParticles(p.x,p.y,"#ff7b95",18,110);
    addShockwave(p.x,p.y,12,"#ff7b95");
    playSfx("hit");
    updateHUD();

    if (p.hp <= 0){
      state.running = false; state.gameOver = true;
      finishRun(false);
      showGameOverPanel();
    }
  }

  function finishRun(cleared){
    state.meta.bestScore = Math.max(state.meta.bestScore, state.score);
    saveMeta();
  }

  function killEnemy(enemy){
    enemy.alive = false;
    state.runStats.kills++;
    state.shake = Math.max(state.shake, enemy.type==="elite" ? 0.14 : 0.08);
    const base = enemy.type==="elite" ? 300 : enemy.type==="guard" ? 180 : enemy.type==="sniper" ? 230 : 100;
    addScore(base);
    gainCoins(enemy.type==="elite" ? 9 : enemy.type==="guard" ? 6 : enemy.type==="sniper" ? 7 : 4);
    addParticles(enemy.x, enemy.y, enemy.type==="elite" ? "#ffd166" : "#9be7ff", 22, 120);
    addShockwave(enemy.x, enemy.y, 10);
    spawnPickup(enemy.x, enemy.y);

    if (state.phase === "wave" && state.enemies.every(e => !e.alive)){
      const stage = stageDefs[state.stageIndex];
      if (stage.boss){
        state.running = false;
        showPanel(
          "WARNING",
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
    state.shake = Math.max(state.shake, 0.45);
    addScore(5000);
    gainCoins(120);
    addParticles(state.boss.x, state.boss.y, "#ffd166", 120, 220);
    addShockwave(state.boss.x, state.boss.y, 30, "#ffd166");
    playSfx("clear");

    if (state.stageIndex < stageDefs.length - 1){
      state.boss = null;
      state.running = false;
      showShopPanel(true);
    } else {
      state.boss = null;
      state.running = false;
      state.win = true;
      finishRun(true);
      showClearPanel();
    }
  }

  function showPanel(title, html, buttonText, onClick){
    ui.overlay.classList.remove("hidden");
    ui.panel.innerHTML = `<h2>${title}</h2>${html}<button id="panelBtn">${buttonText}</button>`;
    document.getElementById("panelBtn").onclick = () => { ensureAudio(); if (onClick) onClick(); else showTitleScreen(); };
  }


  function triggerCutin(textValue){
    state.cutinText = textValue;
    state.cutinTimer = 1.5;
    state.shake = Math.max(state.shake, 0.15);
  }

  function resultSummaryHTML(){
    const secs = Math.max(0, Math.floor(((performance.now() - (state.runStats.startAt || performance.now())) / 1000)));
    const mins = Math.floor(secs / 60);
    const rem = String(secs % 60).padStart(2, "0");
    return `
      <div class="shop-grid">
        <div class="shop-item"><strong>RESULT</strong><br>
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

  function showTitleScreen(){
    state.phase = "menu";
    state.running = false;
    state.paused = false;
    state.gameOver = false;
    state.win = false;
    musicTimer = 0;
    stopMusicVoice();
    createPlayer();
    updateHUD();
    ui.pauseBtn.textContent = "Pause";
    ui.overlay.classList.remove("hidden");
    ui.panel.innerHTML = `
      <h1 style="font-size:34px; letter-spacing:.08em;">SPACE INTRUDER</h1>
      <p><span class="accent">スマホ縦向き対応</span>のアーケードSTG。</p>
      <p>6ステージ、ショップ、恒久強化、必殺技、複数ボス形態、簡易BGM/SE入り。</p>
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
          <div><strong>記録</strong><br><span class="small">BEST SCORE: ${state.meta.bestScore}<br>累計コイン: ${state.meta.totalCoins}<br>恒久強化: HP +${state.meta.permHP} / Rapid +${state.meta.permRapid} / Skill +${state.meta.permSkillRate} / Power +${state.meta.permPower}</span></div>
        </div>
      </div>
      <button id="startGameBtn">ゲーム開始</button>
      <button id="installBtn" class="menu-btn alt" style="display:${'${state.canInstall ? "inline-flex" : "none"}'};margin-left:8px;">ホームに追加</button>
      <p class="small">操作: 左右ボタン or 画面ドラッグ / 右下Fire長押し連射 / Pause</p>
    `;
    ui.panel.querySelectorAll("[data-diff]").forEach(btn => {
      btn.onclick = () => {
        state.difficulty = btn.dataset.diff;
        showTitleScreen();
      };
    });
    document.getElementById("soundToggle").onclick = () => {
      state.meta.soundOn = !state.meta.soundOn;
      if (!state.meta.soundOn) stopMusicVoice();
      saveMeta();
      showTitleScreen();
    };
    document.getElementById("startGameBtn").onclick = () => {
      ensureAudio();
      resetRun();
    };
    const installBtn = document.getElementById("installBtn");
    if (installBtn){
      installBtn.onclick = async () => {
        if (!state.deferredPrompt) return;
        state.deferredPrompt.prompt();
        try { await state.deferredPrompt.userChoice; } catch (e) {}
      };
    }
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
        <div class="shop-item"><div class="shop-row"><div><strong>火力強化</strong><br><span class="small">Power +1（最大5）</span></div><button class="shop-btn" data-buy="power">18C</button></div></div>
        <div class="shop-item"><div class="shop-row"><div><strong>サイドドローン</strong><br><span class="small">補助弾を追加</span></div><button class="shop-btn" data-buy="drone">22C</button></div></div>
        <div class="shop-item"><div class="shop-row"><div><strong>シールド</strong><br><span class="small">被弾を1回無効</span></div><button class="shop-btn" data-buy="shield">14C</button></div></div>
        <div class="shop-item"><div class="shop-row"><div><strong>Skillチャージ</strong><br><span class="small">Skill +35%</span></div><button class="shop-btn" data-buy="skill">16C</button></div></div>
      </div>
      <button id="nextStageBtn">${nextStage <= stageDefs.length ? `STAGE ${nextStage} へ` : "エンディングへ"}</button>
    `;
    ui.panel.querySelectorAll("[data-buy]").forEach(btn => btn.onclick = () => buyShopItem(btn.dataset.buy, fromBoss));
    document.getElementById("nextStageBtn").onclick = () => {
      state.running = true;
      loadStage(state.stageIndex + 1);
      ui.overlay.classList.add("hidden");
    };
  }

  function buyShopItem(kind, fromBoss){
    const p = state.player;
    const costs = {heal:10, power:18, drone:22, shield:14, skill:16};
    const cost = costs[kind];
    if (state.coins < cost) return;
    state.coins -= cost;
    if (kind === "heal") p.hp = Math.min(p.maxHp, p.hp + 1);
    if (kind === "power") p.power = Math.min(5, p.power + 1);
    if (kind === "drone") p.sideDrone = true;
    if (kind === "shield") p.shieldHits = Math.min(3, p.shieldHits + 1);
    if (kind === "skill") p.skill = Math.min(100, p.skill + 35);
    playSfx("pickup");
    updateHUD();
    showShopPanel(fromBoss);
  }

  function showMetaUpgradeBlock(){
    return `
      <div class="shop-grid">
        <div class="shop-item"><div class="shop-row"><div><strong>最大HP強化</strong><br><span class="small">現在 +${state.meta.permHP}</span></div><button class="shop-btn" data-meta="hp">${40 + state.meta.permHP*20}C</button></div></div>
        <div class="shop-item"><div class="shop-row"><div><strong>連射強化</strong><br><span class="small">現在 +${state.meta.permRapid}</span></div><button class="shop-btn" data-meta="rapid">${40 + state.meta.permRapid*20}C</button></div></div>
        <div class="shop-item"><div class="shop-row"><div><strong>Skill蓄積強化</strong><br><span class="small">現在 +${state.meta.permSkillRate}</span></div><button class="shop-btn" data-meta="skill">${40 + state.meta.permSkillRate*20}C</button></div></div>
        <div class="shop-item"><div class="shop-row"><div><strong>初期Power強化</strong><br><span class="small">現在 +${state.meta.permPower}</span></div><button class="shop-btn" data-meta="power">${60 + state.meta.permPower*30}C</button></div></div>
      </div>
    `;
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
      <button id="retryBtn">タイトルへ戻る</button>
    `;
    bindMetaButtons();
    document.getElementById("retryBtn").onclick = () => showTitleScreen();
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
      <button id="restartBtn">タイトルへ戻る</button>
    `;
    bindMetaButtons();
    document.getElementById("restartBtn").onclick = () => showTitleScreen();
  }

  function bindMetaButtons(){
    ui.panel.querySelectorAll("[data-meta]").forEach(btn => btn.onclick = () => buyMeta(btn.dataset.meta));
  }

  function buyMeta(kind){
    const costs = {
      hp: 40 + state.meta.permHP*20,
      rapid: 40 + state.meta.permRapid*20,
      skill: 40 + state.meta.permSkillRate*20,
      power: 60 + state.meta.permPower*30
    };
    const cost = costs[kind];
    if (state.meta.totalCoins < cost) return;
    state.meta.totalCoins -= cost;
    if (kind === "hp") state.meta.permHP++;
    if (kind === "rapid") state.meta.permRapid++;
    if (kind === "skill") state.meta.permSkillRate++;
    if (kind === "power") state.meta.permPower = Math.min(2, state.meta.permPower + 1);
    saveMeta();
    if (state.gameOver) showGameOverPanel();
    else if (state.win) showClearPanel();
    else showTitleScreen();
  }

  function rectsOverlap(a,b){
    return a.x-a.w/2 < b.x+b.w/2 && a.x+a.w/2 > b.x-b.w/2 && a.y-a.h/2 < b.y+b.h/2 && a.y+a.h/2 > b.y-b.h/2;
  }

  function bulletHitRect(bullet, target){
    const bw = bullet.r*2.2, bh = bullet.r*5;
    return bullet.x > target.x-target.w/2-bw/2 && bullet.x < target.x+target.w/2+bw/2 &&
           bullet.y > target.y-target.h/2-bh/2 && bullet.y < target.y+target.h/2+bh/2;
  }

  function update(dt){
    tickMusic(dt);

    for (const s of state.stars){
      s.y += s.s * dt;
      if (s.y > viewH()){ s.y = -3; s.x = Math.random()*viewW(); }
    }
    for (const sw of state.shockwaves){ sw.r += 150*dt; sw.life -= dt; }
    state.shockwaves = state.shockwaves.filter(sw => sw.life > 0);
    if (state.screenFlash > 0) state.screenFlash -= dt;
    if (state.shake > 0) state.shake = Math.max(0, state.shake - dt * 1.6);
    if (state.cutinTimer > 0) state.cutinTimer -= dt;

    const p = state.player;
    if (p){
      if (p.cooldown > 0) p.cooldown -= dt;
      if (p.invincible > 0) p.invincible -= dt;
      if (p.flash > 0) p.flash -= dt;
      if (p.rapid > 0) p.rapid -= dt;
      if (p.pierce > 0) p.pierce -= dt;
    }

    decayCombo(dt);

    if (p){
      const move = (state.input.left ? -1 : 0) + (state.input.right ? 1 : 0);
      p.x += move * p.speed * dt;
      p.x = clamp(p.x, p.w/2+8, viewW()-p.w/2-8);
      if (state.input.fire) spawnPlayerBullet();
    }

    for (const b of state.playerBullets){
      b.x += (b.vx || 0) * dt;
      b.y += b.vy * dt;
    }
    state.playerBullets = state.playerBullets.filter(b => b.y > -30 && b.x > -50 && b.x < viewW()+50);

    for (const b of state.enemyBullets){
      if (b.acc) b.vy += b.acc*dt;
      b.x += (b.vx || 0) * dt;
      b.y += b.vy * dt;
    }
    state.enemyBullets = state.enemyBullets.filter(b => b.y < viewH()+40 && b.x > -70 && b.x < viewW()+70);

      const enemyRemoved = new Set();
      const keptPlayerBullets = [];

      for (let i=0; i<state.playerBullets.length; i++){
        const pb = state.playerBullets[i];
        const cx = Math.floor(pb.x / cellSize);
        const cy = Math.floor(pb.y / cellSize);
        let canceled = false;

        for (let gx = cx - 1; gx <= cx + 1 && !canceled; gx++){
          for (let gy = cy - 1; gy <= cy + 1 && !canceled; gy++){
            const key = gx + "," + gy;
            const indices = grid.get(key);
            if (!indices) continue;

            for (let k = 0; k < indices.length; k++){
              const idx = indices[k];
              if (enemyRemoved.has(idx)) continue;
              const e = state.enemyBullets[idx];
              const dx = pb.x - e.x;
              const dy = pb.y - e.y;
              const r = (pb.r || 3) + (e.r || 4) + 2;
              if (dx*dx + dy*dy <= r*r){
                enemyRemoved.add(idx);
                canceled = true;
                addParticles((pb.x + e.x) * 0.5, (pb.y + e.y) * 0.5, "#d8f6ff", 4, 26);
                break;
              }
            }
          }
        }

        if (!canceled) keptPlayerBullets.push(pb);
      }

      state.playerBullets = keptPlayerBullets;
      state.enemyBullets = state.enemyBullets.filter((_, idx) => !enemyRemoved.has(idx));
    }

    for (const item of state.pickups){
      item.y += item.vy * dt;
      item.spin += dt*6;
    }
    state.pickups = state.pickups.filter(i => i.y < viewH()+24);

    for (const part of state.particles){
      part.x += part.vx*dt; part.y += part.vy*dt; part.vx *= 0.985; part.vy *= 0.985; part.life -= dt;
    }
    state.particles = state.particles.filter(p => p.life > 0);

    if (!state.running || state.paused){
      if (state.messageTimer > 0) state.messageTimer -= dt;
      return;
    }

    const playerBox = {x:p.x, y:p.y, w:p.w, h:p.h};

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
        if (maxX + stage.enemySpeed*diff.enemySpeed*state.moveDir > viewW()-edgePadding) hitEdge = true;
        if (minX + stage.enemySpeed*diff.enemySpeed*state.moveDir < edgePadding) hitEdge = true;
        if (hitEdge){
          state.moveDir *= -1;
          for (const e of state.enemies) if (e.alive) e.y += stage.moveDrop;
        } else {
          for (const e of state.enemies) if (e.alive) e.x += stage.enemySpeed*diff.enemySpeed*state.moveDir;
        }
      }

      const living = state.enemies.filter(e => e.alive);
      for (const e of living){
        if (Math.random() < stage.fireChance * diff.enemyFire){
          const sameColumnVisible = living.some(other => other !== e && Math.abs(other.x-e.x) < 10 && other.y > e.y);
          if (!sameColumnVisible || Math.random() < 0.25) spawnEnemyBullet(e);
        }
      }

      for (const bullet of state.playerBullets){
        for (const enemy of living){
          if (!enemy.alive) continue;
          if (bulletHitRect(bullet, enemy)){
            enemy.hp -= bullet.dmg;
            bullet.hp -= 1;
            addParticles(bullet.x, bullet.y, "#93ebff", 4, 32);
            if (enemy.hp <= 0) killEnemy(enemy);
            if (bullet.hp <= 0){ bullet.y = -999; break; }
          }
        }
      }
      state.playerBullets = state.playerBullets.filter(b => b.y > -100);

      for (const e of living){
        if (e.y + e.h/2 > p.y - 10){
          state.running = false; state.gameOver = true; finishRun(false); showGameOverPanel(); break;
        }
      }
    } else if (state.phase.startsWith("boss") && state.boss){
      const b = state.boss;
      b.x += b.dir * b.speed * dt;
      if (b.x > viewW()-b.w/2-12){ b.x = viewW()-b.w/2-12; b.dir = -1; }
      if (b.x < b.w/2+12){ b.x = b.w/2+12; b.dir = 1; }
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
      state.playerBullets = state.playerBullets.filter(bb => bb.y > -100);
    }

    for (const b of state.enemyBullets){
      const box = {x:b.x, y:b.y, w:b.r*2.3, h:b.r*2.3};
      if (rectsOverlap(playerBox, box)){ b.y = viewH()+100; hitPlayer(); }
    }

    for (const item of state.pickups){
      const box = {x:item.x,y:item.y,w:item.w,h:item.h};
      if (rectsOverlap(playerBox, box)){ item.y = viewH()+100; applyPickup(item.kind); }
    }

    updateHUD();
  }

  // -------- draw ----------
  function roundRect(x,y,w,h,r,fill=false){
    ctx.beginPath();
    ctx.moveTo(x+r,y);
    ctx.arcTo(x+w,y,x+w,y+h,r);
    ctx.arcTo(x+w,y+h,x,y+h,r);
    ctx.arcTo(x,y+h,x,y,r);
    ctx.arcTo(x,y,x+w,y,r);
    if (fill) ctx.fill(); else ctx.stroke();
  }

  function drawBackground(){
    const hue = stageDefs[Math.min(state.stageIndex, stageDefs.length-1)].bgHue;
    const grad = ctx.createLinearGradient(0,0,0,viewH());
    grad.addColorStop(0, `hsla(${hue}, 65%, 20%, 0.22)`);
    grad.addColorStop(0.55, `hsla(${(hue+25)%360}, 70%, 12%, 0.12)`);
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

    ctx.globalAlpha = 0.10;
    ctx.strokeStyle = `hsla(${(hue+40)%360}, 90%, 70%, 0.28)`;
    ctx.lineWidth = 1;
    const t = performance.now() * 0.02;
    for (let i=0; i<8; i++){
      const y = (i * 70 + t) % (viewH() + 80) - 40;
      ctx.beginPath();
      ctx.moveTo(-20, y);
      ctx.lineTo(viewW()+20, y - 40);
      ctx.stroke();
    }

    ctx.globalAlpha = 0.07;
    for (let i=0; i<3; i++){
      const x = (Math.sin(performance.now()/1400 + i) * 0.5 + 0.5) * viewW();
      ctx.fillStyle = `hsla(${(hue+i*35)%360}, 90%, 65%, 0.35)`;
      ctx.fillRect(x-8, 0, 16, viewH());
    }

    ctx.globalAlpha = 1;
  }

  function drawStars(){
    for (const s of state.stars){
      ctx.globalAlpha = s.a;
      ctx.fillStyle = "#fff";
      ctx.beginPath();
      ctx.arc(s.x, s.y, s.r, 0, Math.PI*2);
      ctx.fill();
    }
    ctx.globalAlpha = 1;
  }

  function drawPlayer(){
    const p = state.player;
    if (!p) return;
    ctx.save();
    ctx.translate(p.x,p.y);
    if (p.invincible > 0 && Math.floor(p.invincible*12)%2===0) ctx.globalAlpha = 0.45;

    if (p.shieldHits > 0){
      ctx.globalAlpha = 0.45 + Math.sin(performance.now()/120)*0.12;
      ctx.strokeStyle = "#b396ff";
      ctx.lineWidth = 3;
      ctx.beginPath(); ctx.arc(0,0,24,0,Math.PI*2); ctx.stroke();
      ctx.globalAlpha = 1;
    }

    ctx.fillStyle = "#8ce8ff";
    ctx.beginPath();
    ctx.moveTo(0,-18); ctx.lineTo(13,12); ctx.lineTo(5,8); ctx.lineTo(0,17); ctx.lineTo(-5,8); ctx.lineTo(-13,12); ctx.closePath();
    ctx.fill();
    ctx.fillStyle = "#dff8ff";
    ctx.beginPath(); ctx.moveTo(0,-12); ctx.lineTo(5,4); ctx.lineTo(-5,4); ctx.closePath(); ctx.fill();
    ctx.fillStyle = "#53c7ff"; ctx.fillRect(-15,8,30,5);

    if (p.sideDrone){
      ctx.fillStyle = "#91f2ff";
      roundRect(-31,-2,7,11,4,true); roundRect(24,-2,7,11,4,true);
    }
    ctx.fillStyle = p.rapid > 0 ? "#ffef96" : "#ffd166";
    ctx.beginPath(); ctx.moveTo(-5,17); ctx.lineTo(0,28+Math.random()*3); ctx.lineTo(5,17); ctx.closePath(); ctx.fill();
    if (p.flash > 0){
      ctx.globalAlpha = p.flash*4; ctx.strokeStyle = "#ff7b95"; ctx.lineWidth = 3; ctx.beginPath(); ctx.arc(0,0,26,0,Math.PI*2); ctx.stroke();
    }
    ctx.restore();
    ctx.globalAlpha = 1;
  }

  function drawEnemy(e){
    ctx.save();
    ctx.translate(e.x,e.y);
    let body = "#b4befe", wing = "#7aa2ff";
    if (e.type==="guard"){ body = "#ffc857"; wing = "#ff9f1c"; }
    if (e.type==="elite"){ body = "#ff8fa3"; wing = "#ff5d8f"; }
    if (e.type==="sniper"){ body = "#9effd2"; wing = "#2fe0a0"; }
    ctx.fillStyle = wing;
    ctx.beginPath(); ctx.moveTo(-e.w/2,0); ctx.lineTo(-8,-e.h/2); ctx.lineTo(-4,0); ctx.lineTo(-8,e.h/2-2); ctx.closePath(); ctx.fill();
    ctx.beginPath(); ctx.moveTo(e.w/2,0); ctx.lineTo(8,-e.h/2); ctx.lineTo(4,0); ctx.lineTo(8,e.h/2-2); ctx.closePath(); ctx.fill();
    ctx.fillStyle = body; roundRect(-e.w*0.28,-e.h/2,e.w*0.56,e.h,8,true);
    ctx.fillStyle = "#fff2"; ctx.fillRect(-e.w*0.16,-e.h*0.18,e.w*0.32,4);
    if (e.type==="sniper"){ ctx.fillStyle = "#07111f"; ctx.fillRect(-2,-10,4,20); }
    if (e.hp > 1){
      ctx.fillStyle = "rgba(255,255,255,.18)"; ctx.fillRect(-e.w/2,e.h/2+6,e.w,4);
      ctx.fillStyle = body; ctx.fillRect(-e.w/2,e.h/2+6,e.w*(e.hp/e.maxHp),4);
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
      ctx.strokeStyle = b.kind==="dreadnought" ? "#91f2ff" : "#ff6bff";
      ctx.lineWidth = 4;
      ctx.beginPath();
      ctx.ellipse(0,0,b.w*0.62,b.h*0.74,0,0,Math.PI*2);
      ctx.stroke();
      ctx.globalAlpha = 1;
    }

    if (b.kind === "dreadnought"){
      if (state.phase === "boss1"){
        ctx.fillStyle = "#6f5bff"; roundRect(-b.w/2,-b.h/2,b.w,b.h,18,true);
        ctx.fillStyle = "#ff6b8a"; roundRect(-30,-20,60,40,12,true);
        ctx.fillStyle = "#dff8ff"; ctx.fillRect(-18,-6,36,6);
        ctx.fillStyle = "#7aa2ff"; roundRect(-b.w/2-12,-10,20,20,8,true); roundRect(b.w/2-8,-10,20,20,8,true);
      } else {
        ctx.fillStyle = "#3a244f"; roundRect(-b.w/2,-b.h/2,b.w,b.h,22,true);
        ctx.fillStyle = "#ff4365"; roundRect(-26,-24,52,48,16,true);
        ctx.fillStyle = "#ffe08a"; ctx.beginPath(); ctx.arc(0,0,8,0,Math.PI*2); ctx.fill();
        ctx.fillStyle = "#8c52ff";
        roundRect(-b.w/2-10,-16,18,32,8,true); roundRect(b.w/2-8,-16,18,32,8,true);
      }
    } else {
      if (state.phase === "boss1"){
        ctx.fillStyle = "#7b2cff";
        ctx.beginPath(); ctx.arc(0,0,38,0,Math.PI*2); ctx.fill();
        ctx.fillStyle = "#ff6bff";
        for(let i=0;i<6;i++){ const a = i*Math.PI/3; roundRect(Math.cos(a)*44-7, Math.sin(a)*44-7, 14, 14, 4, true); }
        ctx.fillStyle = "#fff"; ctx.beginPath(); ctx.arc(0,0,9,0,Math.PI*2); ctx.fill();
      } else if (state.phase === "boss2"){
        ctx.fillStyle = "#4d1f63";
        ctx.beginPath(); ctx.arc(0,0,46,0,Math.PI*2); ctx.fill();
        ctx.fillStyle = "#e74cff";
        for(let i=0;i<8;i++){ const a = i*Math.PI/4 + performance.now()/1000*0.4; roundRect(Math.cos(a)*56-8, Math.sin(a)*56-8, 16, 16, 4, true); }
        ctx.fillStyle = "#ffd166"; ctx.beginPath(); ctx.arc(0,0,11,0,Math.PI*2); ctx.fill();
      } else {
        ctx.fillStyle = "#2c0f38";
        ctx.beginPath(); ctx.arc(0,0,54,0,Math.PI*2); ctx.fill();
        ctx.strokeStyle = "#ff6bff"; ctx.lineWidth = 4;
        for(let i=0;i<3;i++){ ctx.beginPath(); ctx.arc(0,0,62+i*10,0,Math.PI*2); ctx.stroke(); }
        ctx.fillStyle = "#ffffff"; ctx.beginPath(); ctx.arc(0,0,14,0,Math.PI*2); ctx.fill();
      }
    }
    ctx.restore();

    const barW = Math.min(viewW()-40, 300), x = viewW()/2 - barW/2, y = 74;
    ctx.fillStyle = "rgba(255,255,255,.18)"; roundRect(x,y,barW,10,6,true);
    ctx.fillStyle = b.kind==="overmind" ? "#ff6bff" : "#ff6b8a"; roundRect(x,y,barW*(b.hp/b.maxHp),10,6,true);
    ctx.fillStyle = "#fff"; ctx.font = "700 12px system-ui, sans-serif"; ctx.textAlign = "center";
    const label = b.kind==="overmind" ? `OVERMIND PHASE ${b.form}` : `DREADNOUGHT PHASE ${b.form}`;
    ctx.fillText(label, viewW()/2, 68);
  }

  function drawBullets(){
    for (const b of state.playerBullets){
      ctx.fillStyle = b.beam ? "#b4f7ff" : "#78e8ff";
      ctx.shadowColor = ctx.fillStyle; ctx.shadowBlur = 12;
      ctx.fillRect(b.x-2.5,b.y-10,5,18);
    }
    ctx.shadowBlur = 0;

    for (const b of state.enemyBullets){
      ctx.fillStyle = b.type==="core" ? "#ff4365" : b.type==="orb" ? "#ff8fa3" : "#ffd166";
      ctx.shadowColor = ctx.fillStyle; ctx.shadowBlur = 10;
      ctx.beginPath(); ctx.arc(b.x,b.y,b.r,0,Math.PI*2); ctx.fill();
    }
    ctx.shadowBlur = 0;
  }

  function drawPickups(){
    const map = {heal:"H", rapid:"R", power:"P", shield:"S", pierce:"X", skill:"K"};
    for (const item of state.pickups){
      ctx.save();
      ctx.translate(item.x,item.y);
      ctx.rotate(item.spin);
      ctx.fillStyle =
        item.kind==="heal" ? "#61ffa8" :
        item.kind==="rapid" ? "#ffd166" :
        item.kind==="shield" ? "#b396ff" :
        item.kind==="pierce" ? "#ff8fa3" :
        item.kind==="skill" ? "#91f2ff" : "#53c7ff";
      roundRect(-9,-9,18,18,5,true);
      ctx.fillStyle = "#07111f";
      ctx.font = "900 10px system-ui, sans-serif";
      ctx.textAlign = "center"; ctx.textBaseline = "middle";
      ctx.fillText(map[item.kind],0,1);
      ctx.restore();
    }
  }

  function drawParticles(){
    for (const p of state.particles){
      ctx.globalAlpha = Math.max(0, p.life/p.maxLife);
      ctx.fillStyle = p.color;
      ctx.beginPath(); ctx.arc(p.x,p.y,p.size,0,Math.PI*2); ctx.fill();
    }
    ctx.globalAlpha = 1;
  }

  function drawShockwaves(){
    for (const sw of state.shockwaves){
      ctx.globalAlpha = sw.life/sw.maxLife;
      ctx.strokeStyle = sw.color; ctx.lineWidth = 2;
      ctx.beginPath(); ctx.arc(sw.x,sw.y,sw.r,0,Math.PI*2); ctx.stroke();
    }
    ctx.globalAlpha = 1;
  }

  function drawBanner(){
    if (state.messageTimer <= 0) return;
    ctx.save();
    ctx.globalAlpha = Math.min(1, state.messageTimer);
    ctx.fillStyle = "rgba(6,10,24,.64)";
    roundRect(viewW()/2-132, 24, 264, 44, 12, true);
    ctx.fillStyle = "#dff8ff"; ctx.font = "700 20px system-ui, sans-serif"; ctx.textAlign = "center";
    let text = stageDefs[Math.min(state.stageIndex, stageDefs.length-1)].name;
    if (state.phase === "boss1") text = "BOSS BATTLE";
    if (state.phase === "boss2") text = "SECOND FORM";
    if (state.phase === "boss3") text = "FINAL FORM";
    ctx.fillText(text, viewW()/2, 52);
    ctx.restore();
  }


  function drawCutin(){
    if (state.cutinTimer <= 0) return;
    const t = state.cutinTimer;
    const alpha = Math.min(1, t * 1.8);
    ctx.save();
    ctx.globalAlpha = alpha;
    const slide = (1 - Math.min(1, (1.5 - t) / 0.35));
    const y = 110 + slide * 40;
    ctx.fillStyle = "rgba(255,255,255,0.08)";
    ctx.fillRect(0, y - 26, viewW(), 56);
    ctx.fillStyle = "rgba(255,80,120,0.82)";
    ctx.fillRect(0, y - 18, viewW(), 8);
    ctx.fillStyle = "rgba(145,242,255,0.85)";
    ctx.fillRect(0, y + 8, viewW(), 6);
    ctx.fillStyle = "#ffffff";
    ctx.textAlign = "center";
    ctx.font = "900 28px system-ui, sans-serif";
    ctx.fillText(state.cutinText, viewW()/2, y + 6);
    ctx.restore();
  }

  function drawBottomInfo(){
    ctx.save();
    ctx.fillStyle = "rgba(0,0,0,.25)";
    roundRect(12, viewH()-44, 220, 28, 10, true);
    ctx.fillStyle = "#fff";
    ctx.font = "700 12px system-ui, sans-serif";
    ctx.textAlign = "left";
    ctx.fillText(`Combo x${state.combo.toFixed(1).replace(".0","")}  Best ${state.meta.bestScore}`, 22, viewH()-25);
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
      const sx = (Math.random() - 0.5) * 18 * state.shake;
      const sy = (Math.random() - 0.5) * 18 * state.shake;
      ctx.translate(sx, sy);
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
    drawCutin();
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

  // -------- controls ----------
  function bindHoldButton(el,key){
    const on = ev => { ev.preventDefault(); ensureAudio(); state.input[key] = true; el.classList.add("active"); };
    const off = ev => { ev.preventDefault(); state.input[key] = false; el.classList.remove("active"); };
    el.addEventListener("pointerdown", on);
    el.addEventListener("pointerup", off);
    el.addEventListener("pointercancel", off);
    el.addEventListener("pointerleave", ev => { if (ev.pressure === 0) off(ev); });
  }
  bindHoldButton(ui.leftBtn, "left");
  bindHoldButton(ui.rightBtn, "right");
  bindHoldButton(ui.fireBtn, "fire");

  ui.pauseBtn.addEventListener("click", () => {
    if (state.gameOver || state.win || (!ui.overlay.classList.contains("hidden") && !state.running)) return;
    state.paused = !state.paused;
    ui.pauseBtn.textContent = state.paused ? "Resume" : "Pause";
  });

  

  canvas.addEventListener("pointerdown", ev => {
    if (!state.running || state.paused) return;
    state.input.pointerActive = true;
    movePlayerToPointer(ev);
  });
  canvas.addEventListener("pointermove", ev => {
    if (!state.input.pointerActive || !state.running || state.paused) return;
    movePlayerToPointer(ev);
  });
  const endPointer = () => state.input.pointerActive = false;
  canvas.addEventListener("pointerup", endPointer);
  canvas.addEventListener("pointercancel", endPointer);

  function movePlayerToPointer(ev){
    const rect = canvas.getBoundingClientRect();
    const x = ev.clientX - rect.left;
    state.player.x = clamp(x, state.player.w/2+8, viewW()-state.player.w/2-8);
  }

  window.addEventListener("keydown", e => {
    ensureAudio();
    if (e.key === "ArrowLeft") state.input.left = true;
    if (e.key === "ArrowRight") state.input.right = true;
    if (e.key === " " || e.key === "Enter") state.input.fire = true;
    if (e.key.toLowerCase() === "p"){
      state.paused = !state.paused;
      ui.pauseBtn.textContent = state.paused ? "Resume" : "Pause";
    }
    if (e.key.toLowerCase() === "x") useSkill();
  });
  window.addEventListener("keyup", e => {
    if (e.key === "ArrowLeft") state.input.left = false;
    if (e.key === "ArrowRight") state.input.right = false;
    if (e.key === " " || e.key === "Enter") state.input.fire = false;
  });


  window.addEventListener("beforeinstallprompt", (e) => {
    e.preventDefault();
    state.deferredPrompt = e;
    state.canInstall = true;
    if (state.phase === "menu") showTitleScreen();
  });

  document.addEventListener("visibilitychange", () => {
    if (document.hidden) {
      stopMusicVoice();
    } else {
      musicTimer = 0;
      if (audioCtx && audioCtx.state === "suspended" && state.meta.soundOn) {
        audioCtx.resume().catch(() => {});
      }
    }
  });

  window.addEventListener("appinstalled", () => {
    state.canInstall = false;
    state.deferredPrompt = null;
  });

  if ("serviceWorker" in navigator) {
    window.addEventListener("load", () => {
      navigator.serviceWorker.register("./space_intruder_sw.js").catch(() => {});
    });
  }

  // start
  loadMeta();
  resize();
  showTitleScreen();
  requestAnimationFrame(loop);
})();
</script>
</body>
</html>
