<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>منصة الألعاب الكلاسيكية الواقعية</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            user-select: none;
        }
        body {
            background-color: #121216;
            color: #f3f4f6;
            font-family: 'Courier New', Courier, monospace; /* طابع كلاسيكي عريق */
            overflow: hidden;
            height: 100vh;
        }
        .screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            transition: opacity 0.3s ease;
        }
        .hidden {
            display: none !important;
        }

        /* القائمة الرئيسية */
        #menuScreen {
            background: linear-gradient(135deg, #1a1a24, #0d0d12);
        }
        h1 {
            font-size: 48px;
            color: #f59e0b;
            margin-bottom: 8px;
            text-shadow: 3px 3px 0px #b45309;
            letter-spacing: 2px;
        }
        .subtitle {
            font-size: 18px;
            color: #9ca3af;
            margin-bottom: 30px;
        }
        .games-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 20px;
            max-width: 950px;
            width: 90%;
        }
        .game-card {
            background: #1f1f2e;
            border: 3px solid #374151;
            border-radius: 12px;
            padding: 20px;
            text-align: center;
            cursor: pointer;
            transition: all 0.2s ease;
            box-shadow: 0 8px 20px rgba(0,0,0,0.5);
            position: relative;
        }
        .game-card:hover {
            transform: translateY(-5px);
            border-color: #f59e0b;
            background: #27273a;
            box-shadow: 0 0 20px rgba(245, 158, 11, 0.4);
        }
        .game-card .icon {
            font-size: 45px;
            margin-bottom: 8px;
            display: block;
        }
        .game-card h2 {
            font-size: 22px;
            margin-bottom: 6px;
            color: #fff;
        }
        .game-card p {
            font-size: 13px;
            color: #9ca3af;
        }

        /* زر الرجوع للقائمة */
        .back-btn {
            position: absolute;
            top: 20px;
            left: 20px;
            background: #1f1f2e;
            border: 2px solid #f59e0b;
            color: #f59e0b;
            padding: 8px 18px;
            border-radius: 8px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            z-index: 100;
            transition: 0.2s;
            font-family: 'Courier New', Courier, monospace;
        }
        .back-btn:hover {
            background: #f59e0b;
            color: #121216;
        }

        /* واجهة اللعب */
        canvas {
            display: block;
            cursor: crosshair;
        }
        #gameUi {
            position: absolute;
            top: 20px;
            right: 30px;
            background: rgba(31, 31, 46, 0.9);
            padding: 8px 20px;
            border-radius: 10px;
            border: 2px solid #374151;
            font-size: 24px;
            font-weight: bold;
            pointer-events: none;
            color: #f59e0b;
        }
    </style>
</head>
<body>

    <!-- القائمة الرئيسية -->
    <div id="menuScreen" class="screen">
        <h1>🕹️ أركيد الكلاسيكيات الواقعية</h1>
        <p class="subtitle">اختر لعبتك المفضلة وتحكم بالماوس أو الريموت على الشاشة</p>
        <div class="games-grid">
            <div class="game-card" onclick="startTowerGame()">
                <span class="icon">🏢</span>
                <h2>بناء ناطحة السحاب</h2>
                <p>اضغط بالوقت المناسب لتركيب الطوابق فوق بعضها والوصول للسماء.</p>
            </div>
            <div class="game-card" onclick="startMinerGame()">
                <span class="icon">🪙</span>
                <h2>صياد الذهب والكنوز</h2>
                <p>انقر لإسقاط خطاف الصنارة وصيد الذهب والألماس الثمين.</p>
            </div>
            <div class="game-card" onclick="startWesternGame()">
                <span class="icon">🤠</span>
                <h2>قناص الغرب الأمريكي</h2>
                <p>صوّب واقضِ على اللصوص فور ظهورهم خلف النوافذ والأبواب.</p>
            </div>
            <div class="game-card" onclick="startBubbleGame()">
                <span class="icon">🎈</span>
                <h2>تفجير الفقاعات الملونة</h2>
                <p>لعبة كلاسيكية هادئة وممتعة لتفجير الفقاعات المرتفعة.</p>
            </div>
        </div>
    </div>

    <!-- شاشة اللعب العامة -->
    <div id="gameScreen" class="screen hidden">
        <button class="back-btn" onclick="returnToMenu()">⬅ رجوع للقائمة</button>
        <div id="gameUi">النقاط: <span id="score">0</span></div>
        <canvas id="gameCanvas"></canvas>
    </div>

    <script>
        const menuScreen = document.getElementById('menuScreen');
        const gameScreen = document.getElementById('gameScreen');
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreEl = document.getElementById('score');

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        let currentGame = null;
        let score = 0;
        let animationFrameId;
        let mouse = { x: canvas.width / 2, y: canvas.height / 2 };
        let particles = [];

        window.addEventListener('resize', () => {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        });

        window.addEventListener('mousemove', (e) => {
            mouse.x = e.clientX;
            mouse.y = e.clientY;
        });

        function createExplosion(x, y, colors, count = 15) {
            for (let i = 0; i < count; i++) {
                particles.push({
                    x: x, y: y,
                    vx: (Math.random() - 0.5) * 12,
                    vy: (Math.random() - 0.5) * 12,
                    radius: Math.random() * 5 + 2,
                    color: colors[Math.floor(Math.random() * colors.length)],
                    alpha: 1
                });
            }
        }

        function updateParticles() {
            particles.forEach((p, i) => {
                p.x += p.vx; p.y += p.vy; p.vy += 0.2; p.alpha -= 0.04;
                ctx.save();
                ctx.globalAlpha = Math.max(0, p.alpha);
                ctx.fillStyle = p.color;
                ctx.beginPath(); ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2); ctx.fill();
                ctx.restore();
                if (p.alpha <= 0) particles.splice(i, 1);
            });
        }

        function returnToMenu() {
            cancelAnimationFrame(animationFrameId);
            currentGame = null;
            window.onmousedown = null;
            gameScreen.classList.add('hidden');
            menuScreen.classList.remove('hidden');
        }

        function initGame() {
            menuScreen.classList.add('hidden');
            gameScreen.classList.remove('hidden');
            score = 0;
            scoreEl.innerText = score;
            particles = [];
        }

        // ================= 1. بناء ناطحة السحاب (Tower Builder) =================
        function startTowerGame() {
            initGame();
            currentGame = 'tower';
            let towers = [{ x: canvas.width / 2 - 100, y: canvas.height - 120, width: 200, height: 40, color: '#f59e0b' }];
            let movingBlock = { x: 100, y: canvas.height - 180, width: 200, height: 40, dx: 4, color: '#3b82f6' };

            window.onmousedown = () => {
                if (currentGame !== 'tower') return;
                let last = towers[towers.length - 1];
                
                // حساب التداخل
                let overlapLeft = Math.max(movingBlock.x, last.x);
                let overlapRight = Math.min(movingBlock.x + movingBlock.width, last.x + last.width);
                let newWidth = overlapRight - overlapLeft;

                if (newWidth > 0) {
                    score += 15;
                    scoreEl.innerText = score;
                    createExplosion(movingBlock.x + newWidth/2, movingBlock.y, ['#f59e0b', '#3b82f6'], 15);

                    // انزل كل الأبراج للأسفل
                    towers.forEach(t => t.y += 45);

                    towers.push({
                        x: overlapLeft,
                        y: canvas.height - 120,
                        width: newWidth,
                        height: 40,
                        color: ['#f59e0b', '#3b82f6', '#10b981', '#ef4444', '#8b5cf6'][towers.length % 5]
                    });

                    movingBlock.width = newWidth;
                    movingBlock.y = canvas.height - 120 - (towers.length * 45);
                    movingBlock.dx += (movingBlock.dx > 0 ? 0.5 : -0.5);
                } else {
                    // خسارة
                    score = 0;
                    scoreEl.innerText = "خسارة! اضغط للبدء مجدداً";
                    towers = [{ x: canvas.width / 2 - 100, y: canvas.height - 120, width: 200, height: 40, color: '#f59e0b' }];
                    movingBlock.y = canvas.height - 180;
                    movingBlock.width = 200;
                }
            };

            function loop() {
                if (currentGame !== 'tower') return;
                ctx.fillStyle = '#0f172a';
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                // حركة الكتلة العلوية
                movingBlock.x += movingBlock.dx;
                if (movingBlock.x < 50 || movingBlock.x + movingBlock.width > canvas.width - 50) {
                    movingBlock.dx *= -1;
                }

                // رسم الأبراج الثابتة
                towers.forEach(t => {
                    ctx.fillStyle = t.color;
                    ctx.fillRect(t.x, t.y, t.width, t.height);
                    ctx.strokeStyle = '#000';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(t.x, t.y, t.width, t.height);
                });

                // رسم الكتلة المتحركة
                ctx.fillStyle = movingBlock.color;
                ctx.fillRect(movingBlock.x, movingBlock.y, movingBlock.width, movingBlock.height);
                ctx.strokeRect(movingBlock.x, movingBlock.y, movingBlock.width, movingBlock.height);

                updateParticles();
                animationFrameId = requestAnimationFrame(loop);
            }
            loop();
        }

        // ================= 2. صياد الذهب والكنوز (Gold Miner) =================
        function startMinerGame() {
            initGame();
            currentGame = 'miner';
            let hookAngle = 0;
            let hookState = 'swing'; // swing, drop
            let hookLength = 60;
            let items = [
                { x: canvas.width / 2 - 200, y: canvas.height - 150, type: 'gold', val: 50, radius: 25 },
                { x: canvas.width / 2 + 250, y: canvas.height - 180, type: 'diamond', val: 100, radius: 18 },
                { x: canvas.width / 2 - 80, y: canvas.height - 120, type: 'rock', val: 10, radius: 35 }
            ];

            window.onmousedown = () => {
                if (currentGame === 'miner' && hookState === 'swing') {
                    hookState = 'drop';
                }
            };

            function loop() {
                if (currentGame !== 'miner') return;
                ctx.fillStyle = '#1e1b4b';
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                let originX = canvas.width / 2;
                let originY = 80;

                // حركة الخطاف
                if (hookState === 'swing') {
                    hookAngle += 0.03;
                    if (hookAngle > 1.2 || hookAngle < -1.2) hookAngle *= -1;
                } else if (hookState === 'drop') {
                    hookLength += 8;
                    if (hookLength > canvas.height || hookLength < 60) hookState = 'return';
                } else if (hookState === 'return') {
                    hookLength -= 8;
                    if (hookLength <= 60) { hookLength = 60; hookState = 'swing'; }
                }

                let hx = originX + Math.sin(hookAngle) * hookLength;
                let hy = originY + Math.cos(hookAngle) * hookLength;

                // رسم الحبل والخطاف
                ctx.strokeStyle = '#f59e0b';
                ctx.lineWidth = 4;
                ctx.beginPath(); ctx.moveTo(originX, originY); ctx.lineTo(hx, hy); ctx.stroke();

                ctx.fillStyle = '#ef4444';
                ctx.beginPath(); ctx.arc(hx, hy, 12, 0, Math.PI * 2); ctx.fill();

                // فحص التصادم مع الكنوز
                items.forEach((item, i) => {
                    ctx.fillStyle = item.type === 'diamond' ? '#38bdf8' : (item.type === 'gold' ? '#fbbf24' : '#64748b');
                    ctx.beginPath(); ctx.arc(item.x, item.y, item.radius, 0, Math.PI * 2); ctx.fill();

                    if (Math.hypot(hx - item.x, hy - item.y) < item.radius + 12 && hookState === 'drop') {
                        score += item.val;
                        scoreEl.innerText = score;
                        createExplosion(item.x, item.y, ['#fbbf24', '#ffffff'], 15);
                        items.splice(i, 1);
                        hookState = 'return';
                    }
                });

                updateParticles();
                animationFrameId = requestAnimationFrame(loop);
            }
            loop();
        }

        // ================= 3. قناص الغرب الأمريكي (Wild West) =================
        function startWesternGame() {
            initGame();
            currentGame = 'western';
            let targets = [];
            let spawnTimer = 0;

            window.onmousedown = () => {
                if (currentGame !== 'western') return;
                createExplosion(mouse.x, mouse.y, ['#f59e0b', '#ffffff'], 10);
                targets.forEach((t, i) => {
                    if (Math.hypot(mouse.x - t.x, mouse.y - t.y) < 40) {
                        score += 25;
                        scoreEl.innerText = score;
                        createExplosion(t.x, t.y, ['#ef4444', '#fbbf24'], 25);
                        targets.splice(i, 1);
                    }
                });
            };

            function loop() {
                if (currentGame !== 'western') return;
                ctx.fillStyle = '#78350f'; // خلفية صخرية وغربية
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                spawnTimer++;
                if (spawnTimer > 45 && targets.length < 4) {
                    targets.push({
                        x: Math.random() * (canvas.width - 200) + 100,
                        y: 150 + Math.random() * (canvas.height - 300),
                        timer: 80
                    });
                    spawnTimer = 0;
                }

                targets.forEach((t, i) => {
                    t.timer--;
                    ctx.fillStyle = '#451a03';
                    ctx.fillRect(t.x - 35, t.y - 45, 70, 90); // نافذة خشبية

                    ctx.font = '35px sans-serif';
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'middle';
                    ctx.fillText('🤠', t.x, t.y);

                    if (t.timer <= 0) targets.splice(i, 1);
                });

                updateParticles();

                // مؤشر التصويب الكلاسيكي
                ctx.strokeStyle = '#ef4444'; ctx.lineWidth = 3;
                ctx.beginPath(); ctx.arc(mouse.x, mouse.y, 25, 0, Math.PI * 2); ctx.stroke();
                ctx.beginPath(); ctx.moveTo(mouse.x - 35, mouse.y); ctx.lineTo(mouse.x + 35, mouse.y); ctx.stroke();
                ctx.beginPath(); ctx.moveTo(mouse.x, mouse.y - 35); ctx.lineTo(mouse.x, mouse.y + 35); ctx.stroke();

                animationFrameId = requestAnimationFrame(loop);
            }
            loop();
        }

        // ================= 4. تفجير الفقاعات الكلاسيكية (Bubble Pop) =================
        function startBubbleGame() {
            initGame();
            currentGame = 'bubble';
            let bubbles = [];
            let spawnTimer = 0;

            window.onmousedown = () => {
                if (currentGame !== 'bubble') return;
                bubbles.forEach((b, i) => {
                    if (Math.hypot(mouse.x - b.x, mouse.y - b.y) < b.radius) {
                        score += 10;
                        scoreEl.innerText = score;
                        createExplosion(b.x, b.y, [b.color, '#ffffff'], 15);
                        bubbles.splice(i, 1);
                    }
                });
            };

            function loop() {
                if (currentGame !== 'bubble') return;
                ctx.fillStyle = '#0f172a';
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                spawnTimer++;
                if (spawnTimer > 30) {
                    bubbles.push({
                        x: Math.random() * (canvas.width - 100) + 50,
                        y: canvas.height + 40,
                        radius: Math.random() * 25 + 20,
                        speed: Math.random() * 2 + 1.5,
                        color: ['#38bdf8', '#f472b6', '#34d399', '#fbbF24', '#a78bfa'][Math.floor(Math.random() * 5)]
                    });
                    spawnTimer = 0;
                }

                bubbles.forEach((b, i) => {
                    b.y -= b.speed;
                    ctx.strokeStyle = b.color;
                    ctx.lineWidth = 3;
                    ctx.fillStyle = 'rgba(255,255,255,0.1)';
                    ctx.beginPath(); ctx.arc(b.x, b.y, b.radius, 0, Math.PI * 2); ctx.fill(); ctx.stroke();

                    if (b.y < -50) bubbles.splice(i, 1);
                });

                updateParticles();
                animationFrameId = requestAnimationFrame(loop);
            }
            loop();
        }
    </script>
</body>
</html>
<link rel="icon" href="https://images.unsplash.com/photo-1550745165-9bc0b252726f?w=200&auto=format&fit=crop&q=80">
