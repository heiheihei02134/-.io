<!DOCTYPE html>
<html>
<head>
    <title>Charlotte的游戏</title>
    <style>
        body {
            margin: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            background-color: pink;
            position: relative;
            overflow: hidden;
        }

        /* 添加飘浮的心形 */
        .floating-heart {
            position: absolute;
            animation: float 10s infinite linear;
            opacity: 0.6;
            z-index: 0;
        }

        @keyframes float {
            0% {
                transform: translateY(100vh) scale(1);
            }
            100% {
                transform: translateY(-100px) scale(0.5);
            }
        }

        .heart {
            position: fixed;
            top: 30%;
            left: 50%;
            transform: translate(-50%, -50%);
            animation: heartbeat 1s infinite;
            z-index: 1;
            width: 150px;
            height: 150px;
            background-color: #ff4d4d;
            transform-origin: center;
            clip-path: path('M12,21.35L10.55,20.03C5.4,15.36 2,12.27 2,8.5C2,5.41 4.42,3 7.5,3C9.24,3 10.91,3.81 12,5.08C13.09,3.81 14.76,3 16.5,3C19.58,3 22,5.41 22,8.5C22,12.27 18.6,15.36 13.45,20.03L12,21.35Z');
        }

        .heart-text {
            position: fixed;
            top: 30%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            font-size: 28px;
            font-weight: bold;
            z-index: 999;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
            animation: glow 2s ease-in-out infinite alternate;
            white-space: nowrap;
        }

        @keyframes glow {
            from {
                text-shadow: 0 0 5px #fff,
                           0 0 10px #fff,
                           0 0 15px #e60073,
                           0 0 20px #e60073,
                           0 0 25px #e60073,
                           0 0 30px #e60073,
                           0 0 35px #e60073;
            }
            to {
                text-shadow: 0 0 10px #fff,
                           0 0 20px #ff4da6,
                           0 0 30px #ff4da6,
                           0 0 40px #ff4da6,
                           0 0 50px #ff4da6,
                           0 0 60px #ff4da6,
                           0 0 70px #ff4da6;
            }
        }

        @keyframes heartbeat {
            0% { transform: translate(-50%, -50%) scale(1); }
            50% { transform: translate(-50%, -50%) scale(1.2); }
            100% { transform: translate(-50%, -50%) scale(1); }
        }

        .game-container {
            margin-top: 250px;
            z-index: 3;
            position: relative;
        }

        canvas {
            border: 3px solid #ff4d4d;
            background-color: rgba(255, 255, 255, 0.9);
            border-radius: 15px;
            box-shadow: 0 0 20px rgba(255, 77, 77, 0.3);
        }

        #score {
            font-size: 28px;
            margin: 10px;
            color: #ff4d4d;
            text-shadow: 2px 2px 4px rgba(255, 255, 255, 0.5);
        }

        #startButton {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 15px 30px;
            font-size: 24px;
            background-color: #ff4d4d;
            color: white;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 0 15px rgba(255, 77, 77, 0.5);
            z-index: 10;
        }

        #startButton:hover {
            background-color: #ff3366;
            transform: translate(-50%, -50%) scale(1.1);
        }
    </style>
</head>
<body>
    <div class="heart"></div>
    <div class="heart-text">Charlotte美好的一天</div>
    
    <div class="game-container">
        <div id="score">得分: 0</div>
        <canvas id="gameCanvas" width="800" height="600"></canvas>
        <button id="startButton">开始游戏</button>
    </div>

    <script>
        // 添加飘浮的心形
        function createFloatingHearts() {
            const colors = ['#4CAF50', '#2196F3']; // 绿色和蓝色
            for (let i = 0; i < 20; i++) {
                const heart = document.createElement('div');
                heart.className = 'floating-heart';
                heart.style.left = Math.random() * 100 + 'vw';
                heart.style.animationDelay = Math.random() * 5 + 's';
                heart.style.fontSize = (Math.random() * 20 + 10) + 'px';
                heart.innerHTML = '❤';
                heart.style.color = colors[Math.floor(Math.random() * colors.length)];
                document.body.appendChild(heart);
            }
        }
        createFloatingHearts();

        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const startButton = document.getElementById('startButton');
        
        const gridSize = 20;
        let snake = [
            {x: 20, y: 15},
            {x: 19, y: 15},
            {x: 18, y: 15}
        ];
        let food = {x: 25, y: 15};
        let dx = 1;
        let dy = 0;
        let score = 0;
        let gameSpeed = 100;
        let gameLoop;
        let gameStarted = false;

        function drawSnakePart(x, y, isHead) {
            ctx.fillStyle = isHead ? '#ff3366' : '#ff4d4d';
            ctx.strokeStyle = '#ffffff';
            ctx.lineWidth = 2;
            
            if (isHead) {
                ctx.beginPath();
                ctx.arc(x * gridSize + gridSize/2, y * gridSize + gridSize/2, 
                       gridSize/2, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();
                
                ctx.fillStyle = 'white';
                ctx.beginPath();
                ctx.arc(x * gridSize + gridSize*0.7, y * gridSize + gridSize*0.3, 
                       3, 0, Math.PI * 2);
                ctx.fill();
                ctx.beginPath();
                ctx.arc(x * gridSize + gridSize*0.7, y * gridSize + gridSize*0.7, 
                       3, 0, Math.PI * 2);
                ctx.fill();
            } else {
                ctx.beginPath();
                ctx.roundRect(x * gridSize, y * gridSize, gridSize-1, gridSize-1, 5);
                ctx.fill();
                ctx.stroke();
            }
        }

        function drawFood() {
            ctx.fillStyle = '#ff1a1a';
            ctx.strokeStyle = '#ffffff';
            ctx.lineWidth = 2;
            
            const x = food.x * gridSize;
            const y = food.y * gridSize;
            const size = gridSize;
            
            ctx.beginPath();
            ctx.moveTo(x + size/2, y + size/4);
            ctx.bezierCurveTo(x + size/2, y, x, y, x, y + size/4);
            ctx.bezierCurveTo(x, y + size/2, x + size/2, y + size*3/4, x + size/2, y + size*3/4);
            ctx.bezierCurveTo(x + size/2, y + size*3/4, x + size, y + size/2, x + size, y + size/4);
            ctx.bezierCurveTo(x + size, y, x + size/2, y, x + size/2, y + size/4);
            ctx.fill();
            ctx.stroke();
        }

        function drawGame() {
            ctx.fillStyle = 'white';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            snake.forEach((part, index) => {
                drawSnakePart(part.x, part.y, index === 0);
            });

            drawFood();
        }

        function moveSnake() {
            const head = {x: snake[0].x + dx, y: snake[0].y + dy};
            snake.unshift(head);

            if (head.x === food.x && head.y === food.y) {
                score += 10;
                scoreElement.innerText = `得分: ${score}`;
                generateFood();
            } else {
                snake.pop();
            }
        }

        function generateFood() {
            food = {
                x: Math.floor(Math.random() * (canvas.width / gridSize)),
                y: Math.floor(Math.random() * (canvas.height / gridSize))
            };
        }

        function checkGameOver() {
            if (snake[0].x < 0 || snake[0].x >= canvas.width / gridSize ||
                snake[0].y < 0 || snake[0].y >= canvas.height / gridSize) {
                return true;
            }

            for (let i = 1; i < snake.length; i++) {
                if (snake[i].x === snake[0].x && snake[i].y === snake[0].y) {
                    return true;
                }
            }
            return false;
        }

        function changeDirection(event) {
            const LEFT_KEY = 37;
            const RIGHT_KEY = 39;
            const UP_KEY = 38;
            const DOWN_KEY = 40;

            const keyPressed = event.keyCode;
            const goingUp = dy === -1;
            const goingDown = dy === 1;
            const goingRight = dx === 1;
            const goingLeft = dx === -1;

            if (keyPressed === LEFT_KEY && !goingRight) {
                dx = -1;
                dy = 0;
            }
            if (keyPressed === UP_KEY && !goingDown) {
                dx = 0;
                dy = -1;
            }
            if (keyPressed === RIGHT_KEY && !goingLeft) {
                dx = 1;
                dy = 0;
            }
            if (keyPressed === DOWN_KEY && !goingUp) {
                dx = 0;
                dy = 1;
            }
        }

        function gameStart() {
            if (checkGameOver()) {
                alert(`游戏结束！得分：${score}`);
                resetGame();
                return;
            }
            moveSnake();
            drawGame();
        }

        function resetGame() {
            snake = [
                {x: 20, y: 15},
                {x: 19, y: 15},
                {x: 18, y: 15}
            ];
            dx = 1;
            dy = 0;
            score = 0;
            scoreElement.innerText = "得分: 0";
            generateFood();
            clearInterval(gameLoop);
            gameStarted = false;
            startButton.style.display = 'block';
            drawGame();
        }

        function startGame() {
            if (!gameStarted) {
                gameStarted = true;
                startButton.style.display = 'none';
                document.addEventListener('keydown', changeDirection);
                gameLoop = setInterval(() => {
                    if (gameStarted) {
                        gameStart();
                    }
                }, gameSpeed);
            }
        }

        startButton.addEventListener('click', startGame);
        drawGame(); // 初始绘制游戏界面
    </script>
</body>
</html>
