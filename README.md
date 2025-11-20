# zhuuuuuuu2018-cmyk.github.io
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>职场粉碎机 - 最终版</title>
    <style>
        body {
            margin: 0;
            background-color: #111;
            color: #f0f0f0;
            font-family: 'Courier New', Courier, monospace;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
            user-select: none;
        }

        #game-container {
            position: relative;
            box-shadow: 0 0 50px rgba(0, 0, 0, 0.9);
            border: 4px solid #666;
            border-radius: 4px;
        }

        canvas {
            background-color: #1e1e1e;
            display: block;
            cursor: crosshair;
        }

        #ui-layer {
            position: absolute;
            top: 20px; left: 25px; right: 25px;
            display: flex;
            justify-content: space-between;
            pointer-events: none;
            font-size: 28px;
            font-weight: 900;
            text-shadow: 2px 2px 0 #000;
            z-index: 10;
        }

        /* 启动页与结算页样式 */
        #start-screen {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85); /* 加深背景，突出文字 */
            backdrop-filter: blur(4px);    /* 背景模糊效果 */
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 20;
            text-align: center;
        }

        .title {
            font-size: 48px;
            margin-bottom: 10px;
            text-transform: uppercase;
            letter-spacing: 4px;
            color: #fff;
            text-shadow: 0 0 10px rgba(255,255,255,0.5);
        }

        .tutorial-box {
            background: rgba(255, 255, 255, 0.1);
            border: 1px dashed #666;
            padding: 20px;
            border-radius: 8px;
            margin: 20px 0;
            line-height: 1.8;
            font-size: 16px;
            color: #ddd;
        }

        .key-highlight {
            background: #ddd;
            color: #000;
            padding: 2px 8px;
            border-radius: 4px;
            font-weight: bold;
            font-family: Arial, sans-serif;
        }

        .blink { animation: blinker 1s linear infinite; }
        @keyframes blinker { 50% { opacity: 0; } }
    </style>
</head>
<body>

    <div id="game-container">
        <canvas id="gameCanvas" width="800" height="500"></canvas>
        
        <div id="ui-layer">
            <span id="scoreDisplay">SCORE: 0</span>
            <span id="hpDisplay" style="color:#ff3333">HP: ❤️❤️❤️</span>
        </div>

        <div id="start-screen">
            <div class="title">职场粉碎机</div>
            
            <div class="tutorial-box">
                <p>🔴 出现 <b>红色 BUG</b> ➜ 按 <span class="key-highlight">空格键</span> 粉碎！</p>
                <p>🟢 出现 <b>绿色 咖啡</b> ➜ <b>不要按键</b> (只有空挥声)</p>
                <p>⚠️ 漏掉BUG 或 打翻咖啡 都会扣血</p>
            </div>

            <p class="blink" style="font-size:22px; color:#44ff44; margin-top:10px;">[ 按空格键 开始工作 ]</p>
            <p style="font-size:12px; color:#666; margin-top:30px;">💡 提示：请开启声音体验最佳打击感</p>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreEl = document.getElementById('scoreDisplay');
        const hpEl = document.getElementById('hpDisplay');
        const startScreen = document.getElementById('start-screen');

        // Web Audio API
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        
        function playSound(type) {
            if (audioCtx.state === 'suspended') audioCtx.resume();
            const osc = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            osc.connect(gainNode);
            gainNode.connect(audioCtx.destination);
            const now = audioCtx.currentTime;
            
            if (type === 'hit') {
                osc.type = 'square';
                osc.frequency.setValueAtTime(800, now);
                osc.frequency.exponentialRampToValueAtTime(100, now + 0.15);
                gainNode.gain.setValueAtTime(0.3, now);
                gainNode.gain.exponentialRampToValueAtTime(0.01, now + 0.15);
                osc.start(now); osc.stop(now + 0.15);
            } else if (type === 'damage') {
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(150, now);
                osc.frequency.linearRampToValueAtTime(50, now + 0.3);
                gainNode.gain.setValueAtTime(0.3, now);
                gainNode.gain.linearRampToValueAtTime(0.01, now + 0.3);
                osc.start(now); osc.stop(now + 0.3);
            } else if (type === 'spawn') {
                osc.type = 'sine';
                osc.frequency.setValueAtTime(600, now);
                gainNode.gain.setValueAtTime(0.05, now);
                gainNode.gain.exponentialRampToValueAtTime(0.001, now + 0.1);
                osc.start(now); osc.stop(now + 0.1);
            }
        }

        // 游戏变量
        let gameState = "IDLE"; 
        let score = 0;
        let hp = 3;
        let lastTime = 0;
        let spawnTimer = 0;
        let currentSpawnInterval = 1200;
        let currentTarget = null;
        let particles = [];
        let shakeIntensity = 0;
        let screenFlash = 0;

        const TARGET_TYPES = { BUG: 1, COFFEE: 2 };

        function createRandomPolygon(x, y, size, color) {
            const sides = Math.floor(Math.random() * 4) + 4; 
            const points = [];
            for (let i = 0; i < sides; i++) {
                const angle = (i / sides) * Math.PI * 2;
                const r = size * (0.5 + Math.random() * 0.8); 
                points.push({ x: x + Math.cos(angle) * r, y: y + Math.sin(angle) * r });
            }
            return { x, y, points, color, type: 'polygon', rotation: 0, rotationSpeed: (Math.random()-0.5)*0.2 };
        }

        function gameLoop(timestamp) {
            if (gameState === "IDLE") return;
            let dt = timestamp - lastTime;
            lastTime = timestamp;
            update(dt);
            draw();
            requestAnimationFrame(gameLoop);
        }

        function update(dt) {
            if (gameState === "GAMEOVER") return;

            currentSpawnInterval = Math.max(400, 1200 - (score * 0.8));

            if (!currentTarget) {
                spawnTimer -= dt;
                if (spawnTimer <= 0) {
                    spawnTarget();
                    spawnTimer = currentSpawnInterval;
                }
            } else {
                currentTarget.displayTimer -= dt;
                if (currentTarget.shape) currentTarget.shape.rotation += currentTarget.shape.rotationSpeed;
                if (currentTarget.scale < 1) currentTarget.scale += 0.1;

                if (currentTarget.displayTimer <= 0) {
                    if (currentTarget.type === TARGET_TYPES.BUG) {
                        playSound('damage');
                        takeDamage(1);
                        shakeIntensity = 10;
                    }
                    currentTarget = null;
                }
            }

            for (let i = particles.length - 1; i >= 0; i--) {
                particles[i].x += particles[i].vx;
                particles[i].y += particles[i].vy;
                particles[i].life -= 0.03;
                if (particles[i].life <= 0) particles.splice(i, 1);
            }

            if (shakeIntensity > 0) shakeIntensity *= 0.9;
            if (shakeIntensity < 0.5) shakeIntensity = 0;
            if (screenFlash > 0) screenFlash -= 0.1;
        }

        function spawnTarget() {
            playSound('spawn');
            const isBug = Math.random() > 0.3; 
            const padding = 80;
            const x = Math.random() * (canvas.width - padding * 2) + padding;
            const y = Math.random() * (canvas.height - padding * 2) + padding;
            
            let shape;
            if (isBug) {
                shape = createRandomPolygon(x, y, 50, '#ff2222');
            } else {
                shape = { x, y, color: '#44ff44', type: 'circle', radius: 40 };
            }

            currentTarget = {
                type: isBug ? TARGET_TYPES.BUG : TARGET_TYPES.COFFEE,
                scale: 0,
                displayTimer: Math.max(600, 1000 - score/2),
                shape: shape
            };
        }

        function draw() {
            ctx.save();
            let dx = (Math.random() - 0.5) * shakeIntensity;
            let dy = (Math.random() - 0.5) * shakeIntensity;
            ctx.translate(dx, dy);

            ctx.fillStyle = '#1e1e1e';
            ctx.fillRect(-dx, -dy, canvas.width, canvas.height);

            if (currentTarget) {
                const s = currentTarget.shape;
                const scale = currentTarget.scale;
                ctx.shadowBlur = 20;
                ctx.shadowColor = s.color;
                ctx.fillStyle = s.color;

                if (s.type === 'polygon') {
                    ctx.save();
                    ctx.translate(s.x, s.y);
                    ctx.scale(scale, scale);
                    ctx.rotate(s.rotation);
                    ctx.beginPath();
                    s.points.forEach((p, i) => {
                        const rx = p.x - s.x;
                        const ry = p.y - s.y;
                        if (i===0) ctx.moveTo(rx, ry); else ctx.lineTo(rx, ry);
                    });
                    ctx.closePath();
                    ctx.fill();
                    ctx.shadowBlur = 0;
                    ctx.fillStyle = '#fff';
                    ctx.font = 'bold 16px Arial';
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'middle';
                    ctx.rotate(-s.rotation);
                    ctx.fillText("BUG", 0, 0);
                    ctx.restore();
                } else if (s.type === 'circle') {
                    ctx.beginPath();
                    ctx.arc(s.x, s.y, s.radius * scale, 0, Math.PI*2);
                    ctx.fill();
                    ctx.shadowBlur = 0;
                    ctx.fillStyle = '#000';
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'middle';
                    ctx.fillText("Coffee", s.x, s.y);
                }
            }

            particles.forEach(p => {
                ctx.fillStyle = p.color;
                ctx.globalAlpha = p.life;
                ctx.fillRect(p.x, p.y, p.size, p.size);
            });

            if (screenFlash > 0) {
                ctx.fillStyle = `rgba(255, 255, 255, ${screenFlash * 0.3})`;
                ctx.fillRect(0, 0, canvas.width, canvas.height);
            }
            ctx.restore();
        }

        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space') {
                e.preventDefault();
                if (gameState === "IDLE" || gameState === "GAMEOVER") {
                    startGame();
                } else {
                    handleInput();
                }
            }
        });

        function handleInput() {
            if (!currentTarget) {
                score = Math.max(0, score - 20);
                updateUI();
                return;
            }

            if (currentTarget.type === TARGET_TYPES.BUG) {
                score += 100;
                shakeIntensity = 30;
                screenFlash = 1.0;
                playSound('hit');
                createExplosion(currentTarget.shape.x, currentTarget.shape.y, '#ff2222');
                currentTarget = null;
                spawnTimer = 150;
            } else {
                takeDamage(1);
                playSound('damage');
                shakeIntensity = 20;
                createExplosion(currentTarget.shape.x, currentTarget.shape.y, '#44ff44');
                currentTarget = null;
            }
            updateUI();
        }

        function createExplosion(x, y, color) {
            for(let i=0; i<30; i++) {
                particles.push({
                    x: x, y: y,
                    vx: (Math.random() - 0.5) * 15,
                    vy: (Math.random() - 0.5) * 15,
                    life: 1.0,
                    size: Math.random() * 8 + 2,
                    color: color
                });
            }
        }

        function takeDamage(amount) {
            hp -= amount;
            updateUI();
            if (hp <= 0) gameOver();
        }

        function startGame() {
            if (audioCtx.state === 'suspended') audioCtx.resume();
            gameState = "PLAYING";
            score = 0;
            hp = 3;
            particles = [];
            currentTarget = null;
            lastTime = performance.now();
            startScreen.style.display = 'none';
            updateUI();
            requestAnimationFrame(gameLoop);
        }

        // 结束界面：复用 HTML 结构，但动态插入分数
        function gameOver() {
            gameState = "GAMEOVER";
            startScreen.style.display = 'flex';
            startScreen.innerHTML = `
                <div class="title" style="color:#ff4444;">GAME OVER</div>
                <p style="font-size:30px; font-weight:bold; margin:10px 0;">最终得分: ${score}</p>
                
                <div class="tutorial-box">
                    <p>🔴 出现 <b>红色 BUG</b> ➜ 按 <span class="key-highlight">空格键</span> 粉碎！</p>
                    <p>🟢 出现 <b>绿色 咖啡</b> ➜ <b>不要按键</b> (只有空挥声)</p>
                    <p>⚠️ 漏掉BUG 或 打翻咖啡 都会扣血</p>
                </div>

                <p class="blink" style="font-size:22px; color:#fff; margin-top:10px;">[ 按空格键 重新挑战 ]</p>
            `;
        }

        function updateUI() {
            scoreEl.innerText = `SCORE: ${score}`;
            hpEl.innerText = `HP: ${"❤️".repeat(hp)}`;
        }

    </script>
</body>
</html>
