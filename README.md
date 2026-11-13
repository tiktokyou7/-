<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ألعاب التلفاز | TV Games</title>
    <link rel="icon" href="https://images.unsplash.com/photo-1550745165-9bc0b252726f?w=200&auto=format&fit=crop&q=80">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; user-select: none; }
        body { background-color: #121216; color: #f3f4f6; font-family: 'Courier New', Courier, monospace; overflow: hidden; height: 100vh; }
        .screen { position: absolute; top: 0; left: 0; width: 100vw; height: 100vh; display: flex; flex-direction: column; justify-content: center; align-items: center; transition: opacity 0.3s ease; }
        .hidden { display: none !important; }
        
        /* القائمة الرئيسية */
        #menuScreen { background: linear-gradient(135deg, #1a1a24 0%, #0d0d12 100%); }
        h1 { font-size: 48px; color: #f59e0b; text-shadow: 0 0 20px rgba(245, 158, 11, 0.4); margin-bottom: 10px; letter-spacing: 2px; }
        .subtitle { font-size: 18px; color: #9ca3af; margin-bottom: 40px; }
        .grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 20px; max-width: 900px; width: 90%; }
        .card { background: #1f2937; border: 2px solid #374151; border-radius: 12px; padding: 25px 15px; text-align: center; cursor: pointer; transition: all 0.2s ease; }
        .card:hover, .card.selected { border-color: #f59e0b; background: #374151; transform: translateY(-5px); box-shadow: 0 10px 25px rgba(245, 158, 11, 0.2); }
        .card h3 { font-size: 20px; color: #fff; margin-bottom: 8px; }
        .card p { font-size: 14px; color: #9ca3af; }

        /* منطقة الألعاب */
        #gameScreen { background: #000; }
        canvas { background: #111; border: 4px solid #374151; box-shadow: 0 0 30px rgba(0,0,0,0.8); }
        .back-btn { position: absolute; top: 20px; left: 20px; background: #dc2626; color: #fff; border: none; padding: 10px 20px; font-size: 16px; border-radius: 6px; cursor: pointer; font-family: inherit; font-weight: bold; }
        .back-btn:hover { background: #b91c1c; }
    </style>
</head>
<body>

    <!-- القائمة الرئيسية -->
    <div id="menuScreen" class="screen">
        <h1>ألعاب التلفاز | TV Games</h1>
        <p class="subtitle">اختر لعبتك المفضلة واستمتع بالتجربة الكلاسيكية</p>
        <div class="grid" id="gameGrid">
            <div class="card" onclick="startGame('snake')">
                <h3>لعبة الثعبان</h3>
                <p>Snake Battle</p>
            </div>
            <div class="card" onclick="startGame('fruit')">
                <h3>تقطيع الفواكه</h3>
                <p>Fruit Ninja</p>
            </div>
            <div class="card" onclick="startGame('miner')">
                <h3>منقب الذهب</h3>
                <p>Gold Miner</p>
            </div>
            <div class="card" onclick="startGame('bubbles')">
                <h3>فرقعة البالونات</h3>
                <p>Bubble Pop</p>
            </div>
        </div>
    </div>

    <!-- شاشة اللعب -->
    <div id="gameScreen" class="screen hidden">
        <button class="back-btn" onclick="returnToMenu()">الرجوع للقائمة</button>
        <canvas id="gameCanvas" width="800" height="500"></canvas>
    </div>

    <script>
        const menuScreen = document.getElementById('menuScreen');
        const gameScreen = document.getElementById('gameScreen');
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        let currentGame = null;
        let gameInterval = null;

        function startGame(type) {
            currentGame = type;
            menuScreen.classList.add('hidden');
            gameScreen.classList.remove('hidden');
            if(type === 'snake') initSnake();
            else if(type === 'bubbles') initBubbles();
            else {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#fff';
                ctx.font = '28px Courier New';
                ctx.textAlign = 'center';
                ctx.fillText('اللعبة قيد التحديث، جاري التشغيل...', canvas.width/2, canvas.height/2);
            }
        }

        function returnToMenu() {
            clearInterval(gameInterval);
            gameScreen.classList.add('hidden');
            menuScreen.classList.remove('hidden');
            currentGame = null;
        }

        // لعبة الثعبان البسيطة السريعة
        function initSnake() {
            let snake = [{x: 10, y: 10}];
            let food = {x: 15, y: 15};
            let dx = 1, dy = 0;
            let score = 0;

            window.onkeydown = (e) => {
                if(e.key === 'ArrowUp' && dy === 0) { dx = 0; dy = -1; }
                if(e.key === 'ArrowDown' && dy === 0) { dx = 0; dy = 1; }
                if(e.key === 'ArrowLeft' && dx === 0) { dx = -1; dy = 0; }
                if(e.key === 'ArrowRight' && dx === 0) { dx = 1; dy = 0; }
            };

            gameInterval = setInterval(() => {
                let head = {x: snake[0].x + dx, y: snake[0].y + dy};
                if(head.x < 0 || head.x >= 40 || head.y < 0 || head.y >= 25) {
                    returnToMenu();
                    alert('انتهت اللعبة! النتيجة: ' + score);
                    return;
                }
                snake.unshift(head);
                if(head.x === food.x && head.y === food.y) {
                    score += 10;
                    food = {x: Math.floor(Math.random()*38), y: Math.floor(Math.random()*23)};
                } else {
                    snake.pop();
                }

                ctx.fillStyle = '#111';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                // رسم الطعام
                ctx.fillStyle = '#ef4444';
                ctx.fillRect(food.x * 20, food.y * 20, 18, 18);

                // رسم الثعبان
                ctx.fillStyle = '#10b981';
                snake.forEach(part => ctx.fillRect(part.x * 20, part.y * 20, 18, 18));
            }, 100);
        }

        // لعبة فرقعة البالونات
        function initBubbles() {
            let bubbles = [];
            for(let i=0; i<10; i++) {
                bubbles.push({x: Math.random()*canvas.width, y: Math.random()*canvas.height, r: 20, speed: 2});
            }
            gameInterval = setInterval(() => {
                ctx.fillStyle = '#111';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                bubbles.forEach(b => {
                    b.y -= b.speed;
                    if(b.y < -20) b.y = canvas.height + 20;
                    ctx.beginPath();
                    ctx.arc(b.x, b.y, b.r, 0, Math.PI*2);
                    ctx.fillStyle = '#3b82f6';
                    ctx.fill();
                });
            }, 30);
        }
    </script>
</body>
</html>
