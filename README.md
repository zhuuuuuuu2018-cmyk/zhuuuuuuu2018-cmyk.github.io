<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>职场粉碎机</title>
    <style>
        /* === 1. 全局重置与霸屏设置 === */
        :root { --btn-color-top: #00c6ff; --btn-color-bot: #0072ff; --btn-shadow: #004090; }
        
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }

        body {
            margin: 0; background-color: #000; overflow: hidden; 
            font-family: 'Courier New', Courier, monospace;
        }

        /* 核心修复：创建一个覆盖全屏的容器，盖住 GitHub 的默认标题 */
        #app-root {
            position: fixed;
            top: 0; left: 0; width: 100vw; height: 100vh; height: 100dvh;
            background-color: #0a0a0a;
            z-index: 99999; /* 层级极高，覆盖一切干扰 */
            display: flex; flex-direction: column; align-items: center;
            padding-bottom: env(safe-area-inset-bottom); /* 适配 iPhone 底部 */
        }

        /* === 2. 游戏容器优化 === */
        #game-container {
            position: relative;
            width: 94vw; max-width: 800px;
            /* 调整高度比例，给底部按钮留出更多空间 */
            height: 55vh; max-height: 500px;
            margin-top: 10vh; /* 顶部留白，视觉平衡 */
            box-shadow: 0 0 40px rgba(0, 198, 255, 0.1);
            border: 2px solid #333; border-radius: 12px;
            background: #000;
        }

        canvas { width: 100%; height: 100%; display: block; border-radius: 10px; }

        /* === 3. 按钮区域优化 === */
        #controls-area {
            /* 使用 flex 填充剩余空间，确保按钮在下方居中 */
            flex: 1;
            width: 90vw; max-width: 600px;
            display: flex; justify-content: center; align-items: center;
            /* 增加底部边距，防止被浏览器工具栏遮挡 */
            padding-bottom: 40px; 
        }

        #smash-btn {
            background: linear-gradient(180deg, var(--btn-color-top), var(--btn-color-bot)); color: white; border: none;
            border-radius: 50px; width: 100%; height: 70px; font-size: 24px; font-weight: 900;
            letter-spacing: 4px; text-shadow: 0 2px 0 rgba(0,0,0,0.2);
            box-shadow: 0 6px 0 var(--btn-shadow), 0 15px 20px rgba(0, 114, 255, 0.3), inset 0 1px 0 rgba(255,255,255,0.4);
            transition: all 0.05s; cursor: pointer; display: flex; align-items: center; justify-content: center; gap: 10px;
        }
        #smash-btn:active, .btn-active {
            transform: translateY(6px); box-shadow: 0 0 0 var(--btn-shadow), 0 0 20px rgba(0, 198, 255, 0.6), inset 0 2px 5px rgba(0,0,0,0.2);
            background: linear-gradient(180deg, #0099cc, #0055cc);
        }

        /* === 4. UI 细节 === */
        #ui-layer {
            position: absolute; top: 0; left: 0; right: 0; padding: 12px 18px; display: flex; justify-content: space-between;
            pointer-events: none; z-index: 10; background: linear-gradient(to bottom, rgba(0,0,0,0.9), transparent); border-radius: 10px 10px 0 0;
        }
        .stat-box { text-align: center; text-shadow: 1px 1px 0 #000; }
        .stat-label { font-size: 10px; color: #888; display: block; margin-bottom: 2px; font-weight: bold;}
        .stat-value { font-size: 18px; font-weight: 900; }
        #scoreDisplay { color: #fff; font-size: 26px; }
        #hpDisplay { margin-top: 2px; font-size: 14px; letter-spacing: 2px;}
        
        #energy-bar-container {
            position: absolute; bottom: 0; left: 0; right: 0; height: 8px; background: #222; z-index: 15; border-top: 1px solid #555; border-radius: 0 0 10px 10px;
        }
        #energy-bar {
            width: 0%; height: 100%; background: linear-gradient(90deg, #00ffff, #0088ff);
            box-shadow: 0 0 10px #00ffff; transition: width 0.1s linear; border-radius: 0 0 0 10px;
        }
        .energy-full { background: linear-gradient(90deg, #ff00ff, #ff0088) !important; box-shadow: 0 0 20px #ff00ff !important; }
        
        #combo-display {
            position: absolute; top: 40%; left: 50%; transform: translateX(-50%); font-size: 28px; color: #ffcc00;
            opacity: 0; font-weight: 900; font-style: italic; pointer-events: none; text-shadow: 0 0 10px orange; transition: transform 0.1s; z-index: 5;
        }

        /* 结算页 */
        #start-screen {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(10,10,10,0.96);
            backdrop-filter: blur(8px); display: flex; flex-direction: column; justify-content: center; align-items: center;
            z-index: 20; text-align: center; border-radius: 10px;
        }
        .title {
            font-size: 32px; margin-bottom: 5px; text-transform: uppercase; letter-spacing: 4px; font-weight: 900;
            color: #fff; background: linear-gradient(45deg, #fff, #888, #fff); -webkit-background-clip: text; -webkit-text-fill-color: transparent;
        }
        .rank-title {
            font-size: 22px; margin: 10px 0; padding: 6px 20px;
            border: 1px solid rgba(255,215,0,0.3); border-radius: 8px;
            color: #ffd700; background: rgba(255,215,0,0.1); text-shadow: 0 0 10px rgba(255,215,0,0.5);
        }
        .hr-comment { font-size: 13px; color: #aaa; font-style: italic; margin-bottom: 15px; width: 80%; }
        .tutorial-box {
            background: rgba(255, 255, 255, 0.05); border: 1px solid #333; padding: 15px; border-radius: 8px;
            margin: 5px 0; line-height: 1.6; font-size: 13px; color: #aaa; text-align: left; width: 90%;
        }
        .key-highlight { background: #333; color: #fff; padding: 1px 6px; border-radius: 3px; font-weight: bold; font-size: 12px; border: 1px solid #555;}
        .blink { animation: blinker 1s step-end infinite; }
        @keyframes blinker { 50% { opacity: 0; } }
        .fever-border { border-color: #ff00ff !important; box-shadow: 0 0 60px rgba(255, 0, 255, 0.5) !important; }
        
        /* AI演示标记 */
        #demo-tag {
            position: absolute; bottom: 10px; right: 10px; color: #00ffff; font-weight: bold; 
            font-size: 12px; animation: pulse 2s infinite; text-shadow: 0 0 5px #00ffff; z-index: 50;
            background: rgba(0,0,0,0.6); padding: 4px 8px; border-radius: 4px; border: 1px solid rgba(0,255,255,0.3);
        }
        @keyframes pulse { 0%{opacity:0.6;} 50%{opacity:1;} 100%{opacity:0.6;} }
    </style>
</head>
<body>
    <div id="app-root">
        <div id="game-container">
            <canvas id="gameCanvas" width="800" height="500"></canvas>
            <div id="ui-layer">
                <div style="display:flex; gap:15px;">
                    <div class="stat-box"><span class="stat-label">🏆 BEST</span><span id="globalBest" class="stat-value">0</span></div>
                </div>
                <div class="stat-box"><span id="scoreDisplay">0</span><div id="hpDisplay">❤️❤️❤️</div></div>
                <div class="stat-box"><span class="stat-label">📊 RANK</span><span id="currentRank" class="stat-value">-</span></div>
            </div>
            <div id="energy-bar-container"><div id="energy-bar"></div></div>
            <div id="combo-display">COMBO x0</div>
            <div id="start-screen"></div>
            </div>

        <div id="controls-area">
            <button id="smash-btn" onmousedown="btnPress(event)" onmouseup="btnRelease(event)" ontouchstart="btnPress(event)" ontouchend="btnRelease(event)">
                <span style="font-size: 28px;">⚡</span> 粉碎
            </button>
        </div>
    </div>
    
    <script>
        // 错误屏蔽
        window.onerror = function(){return true;};

        const CONFIG = {
            spawnBaseInterval: 1350, spawnMinInterval: 400, feverSpawnInterval: 180, 
            energyGain: 15, energyGainElite: 40, minDrainRate: 0.01, maxDrainRate: 0.05, energyDrainAccel: 0.0015
        };

        const StorageSys = {
            getItem(key) { try { return localStorage.getItem(key); } catch(e) { return null; } },
            setItem(key, val) { try { localStorage.setItem(key, val); } catch(e) {} }
        };

        const AudioSys = {
            ctx: null, enabled: true, bpm: 100, nextNoteTime: 0, timerID: null,
            init() {
                if (this.ctx) return;
                try { const AC = window.AudioContext || window.webkitAudioContext; if (AC) this.ctx = new AC(); else this.enabled = false; } catch(e) { this.enabled = false; }
            },
            startBGM() { if (!this.enabled || !this.ctx) return; if (this.ctx.state === 'suspended') this.ctx.resume(); this.nextNoteTime = this.ctx.currentTime; this.scheduler(); },
            stopBGM() { clearTimeout(this.timerID); },
            setBPM(score, isFever) { let target = isFever ? 160 : 100 + Math.min(40, score / 100); this.bpm += (target - this.bpm) * 0.1; },
            scheduler() { while (this.nextNoteTime < this.ctx.currentTime + 0.1) { this.playBeat(this.nextNoteTime); this.nextNoteTime += 60.0 / this.bpm; } this.timerID = setTimeout(() => this.scheduler(), 25); },
            playBeat(time) {
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator(); const gain = this.ctx.createGain(); osc.connect(gain); gain.connect(this.ctx.destination);
                osc.frequency.setValueAtTime(150, time); osc.frequency.exponentialRampToValueAtTime(0.01, time + 0.5);
                gain.gain.setValueAtTime(0.25, time); gain.gain.exponentialRampToValueAtTime(0.01, time + 0.5); osc.start(time); osc.stop(time + 0.5);
            },
            playSFX(type, mod = 0) {
                if (!this.enabled || !this.ctx) return; const t = this.ctx.currentTime; const osc = this.ctx.createOscillator(); const gain = this.ctx.createGain(); osc.connect(gain); gain.connect(this.ctx.destination);
                if (type === 'hit') { osc.type = 'square'; osc.frequency.setValueAtTime(400 + mod*20, t); osc.frequency.exponentialRampToValueAtTime(100, t+0.1); gain.gain.setValueAtTime(0.15, t); } 
                else if (type === 'elite') { osc.type = 'sawtooth'; osc.frequency.setValueAtTime(200, t); osc.frequency.linearRampToValueAtTime(800, t+0.2); gain.gain.setValueAtTime(0.2, t); } 
                else if (type === 'fever_start') { osc.type = 'sine'; osc.frequency.setValueAtTime(200, t); osc.frequency.linearRampToValueAtTime(1000, t+1.0); gain.gain.setValueAtTime(0.5, t); }
                gain.gain.exponentialRampToValueAtTime(0.01, t+0.3); osc.start(t); osc.stop(t+0.3);
            }
        };

        const RankSystem = {
            globalScores: [], personalBest: 0,
            init() {
                const savedPB = StorageSys.getItem('office_smasher_pb_v3'); this.personalBest = savedPB ? parseInt(savedPB) : 0;
                const savedGlobal = StorageSys.getItem('office_smasher_global_v3');
                if (savedGlobal) this.globalScores = JSON.parse(savedGlobal);
                else { this.globalScores = Array.from({length: 50}, () => Math.floor(Math.random() * 14500) + 500); this.globalScores.push(30000); this.globalScores.sort((a, b) => b - a); }
                this.updateUI();
            },
            saveScore(score) {
                if (score > this.personalBest) { this.personalBest = score; StorageSys.setItem('office_smasher_pb_v3', this.personalBest); }
                this.globalScores.push(score); this.globalScores.sort((a, b) => b - a);
                if(this.globalScores.length > 100) this.globalScores.length = 100; StorageSys.setItem('office_smasher_global_v3', JSON.stringify(this.globalScores)); this.updateUI();
            },
            updateUI() { document.getElementById('globalBest').innerText = Math.max(this.globalScores[0]||0, this.personalBest); },
            getRankData(score) {
                if (score < 800) return { title: "🐟 摸鱼艺术家", comment: "HR: 键盘上的最大都比你动得快。" };
                if (score < 2500) return { title: "☕ 饮水机守护神", comment: "HR: 只要我不努力，老板就过不上好日子。" };
                if (score < 5500) return { title: "🔨 合格工具人", comment: "HR: 你的努力，老板看在眼里（大概）。" };
                if (score < 10000) return { title: "💇‍♂️ 秃头预备役", comment: "HR: 变强了，也变秃了。值得吗？" };
                if (score < 20000) return { title: "🔥 这里的卷王", comment: "HR: 这种手速，单身多少年了？" };
                if (score < 35000) return { title: "🤖 莫得感情", comment: "HR: 建议去把服务器修一下，你比脚本还快。" };
                return { title: "👑 公司买下来", comment: "HR: 别打了，老板把位置让给你坐。" };
            }
        };

        const canvas = document.getElementById('gameCanvas'); const ctx = canvas.getContext('2d');
        let gameState = "IDLE"; let score = 0, hp = 3, combo = 0; let lastTime = 0, spawnTimer = 0, currentTarget = null;
        let energy = 0, feverMode = false, feverDrainRate = 0; let particles = [], floatingTexts = [], matrixColumns = [];
        let shakeIntensity = 0, screenFlash = 0; const TARGET_TYPES = { BUG: 1, COFFEE: 2, ELITE: 3 };

        function initMatrix() { matrixColumns = []; const cols = Math.floor(canvas.width / 20); for(let i=0; i<cols; i++) matrixColumns.push({x: i*20, y: Math.random()*canvas.height, s: Math.random()*2+1}); }
        function createPolygon(x, y, size, color, sides) {
            if(!sides) sides = Math.floor(Math.random()*3)+4; const pts = [];
            for(let i=0; i<sides; i++) { const a = (i/sides)*Math.PI*2; const r = size * (0.6 + Math.random()*0.4); pts.push({x: x+Math.cos(a)*r, y: y+Math.sin(a)*r}); }
            return {x, y, pts, color, r:0, rs: (Math.random()-0.5)*0.1};
        }

        function gameLoop(timestamp) {
            if (gameState === "IDLE") return; let dt = timestamp - lastTime; if (dt > 100) dt = 16; lastTime = timestamp; update(dt); draw(); requestAnimationFrame(gameLoop);
        }
        function update(dt) {
            if (gameState === "GAMEOVER") return;
            if (feverMode) { feverDrainRate += CONFIG.energyDrainAccel; energy -= feverDrainRate; if (energy <= 0) endFever(); } 
            else { let scoreFactor = Math.min(1, score / 10000); let currentDrain = CONFIG.minDrainRate + scoreFactor * (CONFIG.maxDrainRate - CONFIG.minDrainRate); energy = Math.max(0, energy - currentDrain); }
            updateEnergyBar(); AudioSys.setBPM(score, feverMode);
            let targetInterval = Math.max(CONFIG.spawnMinInterval, CONFIG.spawnBaseInterval - score * 0.5); if (feverMode) targetInterval = CONFIG.feverSpawnInterval;
            if (!currentTarget) { spawnTimer -= dt; if (spawnTimer <= 0) { spawnTarget(); spawnTimer = targetInterval; } } 
            else { currentTarget.displayTimer -= dt; if(currentTarget.shape) currentTarget.shape.r += currentTarget.shape.rs; if (currentTarget.scale < 1) currentTarget.scale += 0.1; if (currentTarget.displayTimer <= 0) { if (currentTarget.type !== TARGET_TYPES.COFFEE) { takeDamage(1); resetComboAndEnergy(); createFloatingText("MISS", currentTarget.shape.x, currentTarget.shape.y, "#888"); } currentTarget = null; } }
            particles.forEach((p,i) => { p.x+=p.vx; p.y+=p.vy; p.life-=0.03; if(p.life<=0) particles.splice(i,1); });
            floatingTexts.forEach((t,i) => { t.y+=t.vy; t.life-=0.02; if(t.life<=0) floatingTexts.splice(i,1); });
            matrixColumns.forEach(c => { c.y += c.s * (feverMode ? 8 : 2); if(c.y > canvas.height) c.y = -20; });
            if(shakeIntensity>0) shakeIntensity *= 0.9; if(screenFlash>0) screenFlash -= 0.1;
        }
        function spawnTarget() {
            let type = TARGET_TYPES.BUG; const rand = Math.random();
            if (feverMode) { let eliteChance = score > 5000 ? 0.2 : 0.1; type = rand < eliteChance ? TARGET_TYPES.ELITE : TARGET_TYPES.BUG; } 
            else {
                if (score < 2000) { if (rand < 0.05) type = TARGET_TYPES.COFFEE; else type = TARGET_TYPES.BUG; } 
                else if (score < 5000) { if (rand < 0.05) type = TARGET_TYPES.ELITE; else if (rand < 0.25) type = TARGET_TYPES.COFFEE; else type = TARGET_TYPES.BUG; } 
                else { if (rand < 0.15) type = TARGET_TYPES.ELITE; else if (rand < 0.45) type = TARGET_TYPES.COFFEE; else type = TARGET_TYPES.BUG; }
            }
            const pad = 100; const x = Math.random()*(canvas.width-pad*2)+pad; const y = Math.random()*(canvas.height-pad*2)+pad; let shape, hp=1, duration=1000;
            if (type === TARGET_TYPES.ELITE) { shape = createPolygon(x, y, 70, '#bf00ff', 8); hp = 3; duration = 2000; } 
            else if (type === TARGET_TYPES.COFFEE) { shape = createPolygon(x, y, 45, '#44ff44', 5); duration = 1300; } 
            else { shape = createPolygon(x, y, 50, feverMode?'#00ffff':'#ff4444'); duration = Math.max(600, 1100 - score/5); }
            currentTarget = { type, hp, maxHp:hp, scale:0, displayTimer:duration, shape };
        }
        function handleHit(points, isElite) {
            combo++; score += points + combo * (isElite?20:5);
            if (!feverMode) { energy += isElite ? CONFIG.energyGainElite : CONFIG.energyGain; if (energy >= 100) startFever(); } else { energy = Math.min(100, energy + 4); }
            shakeIntensity = isElite ? 40 : 20; screenFlash = isElite ? 0.5 : 0.2; AudioSys.playSFX('hit', combo/50);
            const color = isElite ? '#bf00ff' : (feverMode?'#00ffff':'#ff4444');
            createExplosion(currentTarget.shape.x, currentTarget.shape.y, color, isElite?40:20);
            createFloatingText(feverMode?"SMASH!":"+"+points, currentTarget.shape.x, currentTarget.shape.y, "#fff");
            const comboEl = document.getElementById('combo-display'); comboEl.innerText = `COMBO x${combo}`; comboEl.style.opacity = 1; comboEl.style.transform = "translateX(-50%) scale(1.3)"; setTimeout(()=>comboEl.style.transform = "translateX(-50%) scale(1)", 100);
            currentTarget = null; spawnTimer = feverMode ? 80 : 150; 
        }
        function handleInput() {
            if (!currentTarget) { score = Math.max(0, score - 20); resetComboAndEnergy(); createFloatingText("-20", canvas.width/2, canvas.height/2, "#888"); return; }
            if (currentTarget.type === TARGET_TYPES.COFFEE) { takeDamage(1); resetComboAndEnergy(); createFloatingText("NO!", currentTarget.shape.x, currentTarget.shape.y, "#44ff44"); currentTarget = null; } 
            else {
                currentTarget.hp--;
                if (currentTarget.hp <= 0) handleHit(currentTarget.type===TARGET_TYPES.ELITE?500:100, currentTarget.type===TARGET_TYPES.ELITE);
                else { shakeIntensity = 10; AudioSys.playSFX('elite'); currentTarget.shape.color = '#fff'; setTimeout(()=>currentTarget?currentTarget.shape.color='#bf00ff':null, 50); createFloatingText("HIT", currentTarget.shape.x, currentTarget.shape.y, "#fff", 16); }
            }
        }
        function startFever() { feverMode = true; energy = 100; feverDrainRate = CONFIG.minDrainRate; AudioSys.playSFX('fever_start'); document.getElementById('game-container').classList.add('fever-border'); document.getElementById('energy-bar').classList.add('energy-full'); createFloatingText("⚡ OVERLOAD ⚡", canvas.width/2, canvas.height/2, "#ff00ff", 40); }
        function endFever() { feverMode = false; energy = 0; document.getElementById('game-container').classList.remove('fever-border'); document.getElementById('energy-bar').classList.remove('energy-full'); createFloatingText("COOLDOWN", canvas.width/2, canvas.height/2, "#888", 30); }
        function resetComboAndEnergy() { combo = 0; document.getElementById('combo-display').style.opacity = 0; if (feverMode) energy -= 30; else energy = Math.max(0, energy - 30); }
        function updateEnergyBar() { document.getElementById('energy-bar').style.width = `${Math.max(0, Math.min(100, energy))}%`; }
        function createExplosion(x, y, color, n) { for(let i=0; i<n; i++) particles.push({x, y, vx:(Math.random()-0.5)*15, vy:(Math.random()-0.5)*15, life:1, color, size:Math.random()*5+2}); }
        function createFloatingText(txt, x, y, c, s=24) { floatingTexts.push({text:txt, x, y, color:c, size:s, life:1, vy:-3}); }
        function takeDamage(amt) { hp -= amt; shakeIntensity = 30; AudioSys.playSFX('hit', -5); document.getElementById('hpDisplay').innerText = "❤️".repeat(Math.max(0, hp)); if(hp<=0) gameOver(); }
        function draw() {
            ctx.save(); let dx = (Math.random()-0.5)*shakeIntensity, dy = (Math.random()-0.5)*shakeIntensity; ctx.translate(dx, dy);
            ctx.fillStyle = '#050505'; ctx.fillRect(-dx, -dy, canvas.width, canvas.height);
            ctx.fillStyle = feverMode?'rgba(255,0,255,0.1)':'rgba(0,255,0,0.05)'; ctx.font = '14px monospace';
            matrixColumns.forEach(c=>ctx.fillText(String.fromCharCode(0x30A0+Math.random()*96), c.x, c.y));
            if(currentTarget) {
                const s = currentTarget.shape; ctx.translate(s.x, s.y); ctx.scale(currentTarget.scale, currentTarget.scale);
                ctx.fillStyle = s.color; ctx.shadowBlur = feverMode?40:20; ctx.shadowColor = s.color;
                ctx.beginPath(); s.pts.forEach((p,i)=>{ const rx = p.x-s.x, ry = p.y-s.y; if(i===0)ctx.moveTo(rx,ry); else ctx.lineTo(rx,ry); }); ctx.fill();
                ctx.shadowBlur = 0; ctx.fillStyle = '#fff'; ctx.textAlign='center'; ctx.textBaseline='middle';
                ctx.font = currentTarget.type===TARGET_TYPES.ELITE?'bold 16px Arial':'14px Arial';
                ctx.fillText(currentTarget.type===TARGET_TYPES.ELITE?`ELITE [${currentTarget.hp}]`:currentTarget.type===TARGET_TYPES.COFFEE?'COFFEE':'BUG', 0, 0);
                ctx.translate(-s.x, -s.y);
            }
            particles.forEach(p=>{ ctx.fillStyle=p.color; ctx.globalAlpha=p.life; ctx.fillRect(p.x, p.y, p.size, p.size); });
            floatingTexts.forEach(t=>{ ctx.globalAlpha=Math.max(0,t.life); ctx.fillStyle=t.color; ctx.font=`bold ${t.size}px Arial`; ctx.fillText(t.text, t.x, t.y); });
            if(screenFlash>0) { ctx.fillStyle=`rgba(255,255,255,${screenFlash})`; ctx.fillRect(0,0,canvas.width,canvas.height); }
            ctx.restore();
        }

        function showStartScreen() {
            const screen = document.getElementById('start-screen'); screen.style.display = 'flex';
            screen.innerHTML = `
                <div class="title">职场粉碎机</div>
                <div style="color:#00ffff; font-size:14px; letter-spacing:2px; margin-bottom:10px;">v1.9 霸屏修复版</div>
                <div class="tutorial-box">
                    <p>🟣 精英怪: 坚硬, 连按 <span class="key-highlight">3次</span></p>
                    <p>⚡ 能量条: 满条进入 <span style="color:#ff00ff"><b>暴走</b></span></p>
                    <p>🔴 红色BUG: 点击 <span class="key-highlight">蓝色按钮</span></p>
                </div>
                <p class="blink" style="font-size:20px; color:#44ff44; margin-top:15px; cursor:pointer;" onclick="triggerStart()">>> TAP TO START <<</p>
            `;
        }
        function triggerStart() { if(gameState!=="PLAYING") { AudioSys.init(); AudioSys.startBGM(); startGame(); } }
        function triggerAction() { if(gameState==="PLAYING") handleInput(); else triggerStart(); }
        function btnPress(e) { if(e) e.preventDefault(); const btn = document.getElementById('smash-btn'); btn.classList.add('btn-active'); triggerAction(); }
        function btnRelease(e) { if(e) e.preventDefault(); document.getElementById('smash-btn').classList.remove('btn-active'); }
        document.addEventListener('keydown', e=>{ if(e.code==='Space') {e.preventDefault(); btnPress(); setTimeout(btnRelease, 100);} });
        document.addEventListener('mousedown', e=>{ if(e.target.tagName==='CANVAS') {e.preventDefault(); triggerAction();} });
        document.addEventListener('touchstart', e=>{ if(e.target.tagName==='CANVAS') {e.preventDefault(); triggerAction();} }, {passive:false});

        function startGame() {
            initMatrix(); RankSystem.init(); gameState = "PLAYING"; score=0; hp=3; combo=0; energy=0; feverMode=false;
            particles=[]; floatingTexts=[]; currentTarget=null; lastTime=performance.now();
            document.getElementById('start-screen').style.display='none'; document.getElementById('game-container').classList.remove('fever-border');
            document.getElementById('scoreDisplay').innerText='0'; document.getElementById('hpDisplay').innerText='❤️❤️❤️';
            requestAnimationFrame(gameLoop);
        }

        function gameOver() {
            gameState = "GAMEOVER"; AudioSys.stopBGM(); RankSystem.saveScore(score);
            const rankData = RankSystem.getRankData(score);
            const screen = document.getElementById('start-screen'); screen.style.display = 'flex';
            screen.innerHTML = `
                <div class="title" style="color:#ff4444; font-size:32px;">GAME OVER</div>
                <div style="font-size:24px; margin-bottom:5px;">SCORE: <span style="color:#fff; font-weight:bold;">${score}</span></div>
                <div class="rank-title">${rankData.title}</div>
                <div class="hr-comment">${rankData.comment}</div>
                <div class="tutorial-box" style="width:80%; font-size:12px; margin-top:0;">
                    <p>1. 🟣 精英怪收益高，需连按 3 次。</p>
                    <p>2. 🔥 暴走模式手速要快。</p>
                    <p>3. ❌ 漏怪/误触扣能量。</p>
                </div>
                <p class="blink" style="font-size:20px; color:#fff; cursor:pointer; margin-top:10px;" onclick="triggerStart()">[ 重新入职 ]</p>
            `;
        }

        showStartScreen(); RankSystem.init();
    </script>
</body>
</html>
