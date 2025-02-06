<!DOCTYPE html>
<html>
<head>
    <title>Tetris Game</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f0f0f0;
            font-family: Arial, sans-serif;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
            touch-action: manipulation;
            overscroll-behavior: none;
        }

        .container {
            text-align: center;
            max-width: 100%;
            padding: 0 10px;
        }

        h1 {
            color: #333;
            margin-bottom: 20px;
            font-size: clamp(20px, 4vw, 32px);
        }
        
        #game-board {
            border: 2px solid #333;
            background-color: #111;
            cursor: pointer;
            box-shadow: 0 0 20px rgba(0,0,0,0.2);
            max-width: 100%;
            height: auto;
            touch-action: none;
        }

        .game-stats {
            margin: 15px 0;
            font-size: clamp(16px, 3.5vw, 22px);
            color: #333;
            font-weight: bold;
        }

        .controls {
            margin: 15px 0;
            padding: 12px;
            background: #fff;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }

        .controls p {
            margin: 8px 0;
            color: #666;
            font-size: clamp(12px, 2.5vw, 14px);
        }

        .pause-overlay {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: clamp(20px, 5vw, 28px);
            color: white;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
            display: none;
        }

        .mobile-controls {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 8px;
            width: min(280px, 80vw);
            margin: 15px auto;
        }

        #achievement-image {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            max-width: min(200px, 50vw);
            display: none;
            z-index: 1000;
        }

        .control-btn {
            padding: 15px 10px;
            background: #333;
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 18px;
            touch-action: manipulation;
            user-select: none;
            -webkit-tap-highlight-color: transparent;
        }

        .control-btn:active {
            background: #555;
        }

        @media (min-width: 769px) {
            .mobile-controls {
                display: none;
            }
        }

        @media (max-width: 768px) {
            body {
                padding: 10px;
            }

            .controls p:not(:first-child) {
                display: none;
            }

            .container {
                padding: 0 5px;
            }
        }

        footer {
            margin-top: 15px;
            font-size: 14px;
            color: #666;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>TETRIS BUATAN ORANG SIGMA</h1>
        <div class="game-stats">
            Score: <span id="score">0</span><br>
            Best Score: <span id="best-score">0</span>
        </div>
        <canvas id="game-board" width="300" height="600"></canvas>
        <div class="pause-overlay" id="pause-text">PAUSED</div>
        <img id="achievement-image" src="WhatsApp Image 2025-02-06 at 07.27.35_3bc29a35.jpg" alt="New High Score!">
        <div class="controls">
            <p>Controls:</p>
            <p>← → : Move</p>
            <p>↑ : Rotate</p>
            <p>↓ : Soft Drop</p>
            <p>Space : Hard Drop</p>
            <p>P : Pause/Resume</p>
            <p>Mouse/Touch : Drag pieces</p>
        </div>
        <div class="mobile-controls">
            <button class="control-btn" id="left-btn">←</button>
            <button class="control-btn" id="rotate-btn">↑</button>
            <button class="control-btn" id="right-btn">→</button>
            <button class="control-btn" id="down-btn">↓</button>
            <button class="control-btn" id="drop-btn">⬇️</button>
            <button class="control-btn" id="pause-btn">⏸️</button>
        </div>
        <footer>
            <p>Created by daffa alzeinna | develover baik hati</p>
        </footer>
    </div>

    <script>
        const canvas = document.getElementById('game-board');
        const ctx = canvas.getContext('2d');
        const BLOCK_SIZE = 30;
        const BOARD_WIDTH = 10;
        const BOARD_HEIGHT = 20;
        let isPaused = false;
        let score = 0;
        let bestScore = localStorage.getItem('tetrisBestScore') || 0;
        let currentPiece = null;
        let dropCounter = 0;
        let dropInterval = 1000;
        let lastTime = 0;
        let isDragging = false;
        let dragStartX = 0;
        let dragStartY = 0;
        let originalPiecePos = {x: 0, y: 0};
        let touchStartX = null;
        let touchStartY = null;

        document.getElementById('best-score').textContent = bestScore;

        // Prevent scrolling when touching the canvas
        canvas.addEventListener('touchstart', function(e) {
            e.preventDefault();
        }, { passive: false });

        canvas.addEventListener('touchmove', function(e) {
            e.preventDefault();
        }, { passive: false });

        // Adjust canvas size based on screen size
        function resizeCanvas() {
            const container = document.querySelector('.container');
            const maxWidth = Math.min(300, container.offsetWidth - 20);
            const height = (maxWidth / 300) * 600;
            
            canvas.style.width = maxWidth + 'px';
            canvas.style.height = height + 'px';
        }

        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        const PIECES = [
            [[1,1,1,1]], // I
            [[1,1],[1,1]], // O
            [[0,1,0],[1,1,1]], // T
            [[1,0],[1,0],[1,1]], // L
            [[0,1],[0,1],[1,1]], // J
            [[1,1,0],[0,1,1]], // S
            [[0,1,1],[1,1,0]] // Z
        ];

        const COLORS = [
            '#00f0f0', '#f0f000', '#a000f0',
            '#f0a000', '#0000f0', '#00f000', '#f00000'
        ];

        let board = Array(BOARD_HEIGHT).fill().map(() => Array(BOARD_WIDTH).fill(0));

        function createPiece() {
            const piece = PIECES[Math.floor(Math.random() * PIECES.length)];
            return {
                pos: {x: Math.floor(BOARD_WIDTH/2) - Math.floor(piece[0].length/2), y: 0},
                matrix: piece,
                color: COLORS[Math.floor(Math.random() * COLORS.length)]
            };
        }

        function showAchievementImage() {
            const img = document.getElementById('achievement-image');
            img.style.display = 'block';
            setTimeout(() => {
                img.style.display = 'none';
            }, 3000);
        }

        function drawBlock(x, y, color) {
            ctx.fillStyle = color;
            ctx.fillRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE - 1, BLOCK_SIZE - 1);
        }

        function drawBoard() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Draw board
            for (let y = 0; y < BOARD_HEIGHT; y++) {
                for (let x = 0; x < BOARD_WIDTH; x++) {
                    if (board[y][x]) {
                        drawBlock(x, y, board[y][x]);
                    }
                }
            }

            // Draw current piece
            if (currentPiece) {
                currentPiece.matrix.forEach((row, y) => {
                    row.forEach((value, x) => {
                        if (value) {
                            drawBlock(x + currentPiece.pos.x, y + currentPiece.pos.y, currentPiece.color);
                        }
                    });
                });
            }

            if (isPaused) {
                ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                document.getElementById('pause-text').style.display = 'block';
            } else {
                document.getElementById('pause-text').style.display = 'none';
            }
        }

        function collide() {
            const [m, o] = [currentPiece.matrix, currentPiece.pos];
            for (let y = 0; y < m.length; y++) {
                for (let x = 0; x < m[y].length; x++) {
                    if (m[y][x] !== 0 &&
                        (board[y + o.y] &&
                        board[y + o.y][x + o.x]) !== 0) {
                        return true;
                    }
                }
            }
            return false;
        }

        function merge() {
            currentPiece.matrix.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value) {
                        board[y + currentPiece.pos.y][x + currentPiece.pos.x] = currentPiece.color;
                    }
                });
            });
        }

        function rotate() {
            const matrix = currentPiece.matrix;
            const N = matrix.length;
            const rotated = Array(N).fill().map(() => Array(N).fill(0));
            
            for (let y = 0; y < N; y++) {
                for (let x = 0; x < N; x++) {
                    rotated[x][N - 1 - y] = matrix[y][x];
                }
            }
            
            const pos = currentPiece.pos;
            currentPiece.matrix = rotated;
            
            if (collide()) {
                currentPiece.matrix = matrix;
            }
        }

        function clearLines() {
            let linesCleared = 0;
            
            outer: for (let y = board.length - 1; y >= 0; y--) {
                for (let x = 0; x < board[y].length; x++) {
                    if (board[y][x] === 0) {
                        continue outer;
                    }
                }
                
                const row = board.splice(y, 1)[0].fill(0);
                board.unshift(row);
                linesCleared++;
                y++;
            }

            if (linesCleared > 0) {
                score += [0, 100, 300, 500, 800][linesCleared];
                document.getElementById('score').textContent = score;
                
                if (score > bestScore) {
                    bestScore = score;
                    localStorage.setItem('tetrisBestScore', bestScore);
                    document.getElementById('best-score').textContent = bestScore;
                    showAchievementImage();
                }
            }
        }

        function drop() {
            currentPiece.pos.y++;
            if (collide()) {
                currentPiece.pos.y--;
                merge();
                clearLines();
                currentPiece = createPiece();
                if (collide()) {
                    // Game Over
                    board.forEach(row => row.fill(0));
                    score = 0;
                    document.getElementById('score').textContent = score;
                }
            }
            dropCounter = 0;
        }

        function move(dir) {
            currentPiece.pos.x += dir;
            if (collide()) {
                currentPiece.pos.x -= dir;
            }
        }

        function hardDrop() {
            while (!collide()) {
                currentPiece.pos.y++;
            }
            currentPiece.pos.y--;
            drop();
        }

        function update(time = 0) {
            if (!isPaused && !isDragging) {
                const deltaTime = time - lastTime;
                lastTime = time;
                
                dropCounter += deltaTime;
                if (dropCounter > dropInterval) {
                    drop();
                }
            }
            
            drawBoard();
            requestAnimationFrame(update);
        }

        function togglePause() {
            isPaused = !isPaused;
            if (!isPaused) {
                lastTime = performance.now();
            }
        }

        // Mouse/Touch controls
        function getMousePos(e) {
            const rect = canvas.getBoundingClientRect();
            const scaleX = canvas.width / rect.width;
            const scaleY = canvas.height / rect.height;
            
            if (e.touches) {
                return {
                    x: Math.floor((e.touches[0].clientX - rect.left) * scaleX / BLOCK_SIZE),
                    y: Math.floor((e.touches[0].clientY - rect.top) * scaleY / BLOCK_SIZE)
                };
            }
            
            return {
                x: Math.floor((e.clientX - rect.left) * scaleX / BLOCK_SIZE),
                y: Math.floor((e.clientY - rect.top) * scaleY / BLOCK_SIZE)
            };
        }

        canvas.addEventListener('mousedown', startDrag);
        canvas.addEventListener('touchstart', startDrag);
        canvas.addEventListener('mousemove', drag);
        canvas.addEventListener('touchmove', drag);
        canvas.addEventListener('mouseup', endDrag);
        canvas.addEventListener('touchend', endDrag);

        function startDrag(e) {
            if (isPaused) return;
            if (e.type.startsWith('touch')) {
                e.preventDefault();
            }
            
            const pos = getMousePos(e);
            dragStartX = pos.x;
            dragStartY = pos.y;
            originalPiecePos = {x: currentPiece.pos.x, y: currentPiece.pos.y};
            isDragging = true;
        }

        function drag(e) {
            if (!isDragging || isPaused) return;
            if (e.type.startsWith('touch')) {
                e.preventDefault();
            }
            
            const pos = getMousePos(e);
            const deltaX = pos.x - dragStartX;
            const deltaY = pos.y - dragStartY;
            
            currentPiece.pos.x = originalPiecePos.x + deltaX;
            currentPiece.pos.y = originalPiecePos.y + deltaY;
            
            if (collide()) {
                currentPiece.pos.x = originalPiecePos.x;
                currentPiece.pos.y = originalPiecePos.y;
            }
            
            // Keep piece within bounds
            currentPiece.pos.x = Math.max(0, Math.min(BOARD_WIDTH - currentPiece.matrix[0].length, currentPiece.pos.x));
            currentPiece.pos.y = Math.max(0, Math.min(BOARD_HEIGHT - currentPiece.matrix.length, currentPiece.pos.y));
        }

        function endDrag() {
            isDragging = false;
        }

        document.addEventListener('keydown', (e) => {
            if (!isPaused) {
                switch(e.key) {
                    case 'ArrowLeft':
                        move(-1);
                        break;
                    case 'ArrowRight':
                        move(1);
                        break;
                    case 'ArrowDown':
                        drop();
                        break;
                    case 'ArrowUp':
                        rotate();
                        break;
                    case ' ':
                        hardDrop();
                        break;
                }
            }
            if (e.key === 'p' || e.key === 'P') {
                togglePause();
            }
        });

        // Mobile controls with touch feedback
        const addMobileButton = (id, action) => {
            const btn = document.getElementById(id);
            let pressTimer;
            
            btn.addEventListener('touchstart', (e) => {
                e.preventDefault();
                if (!isPaused) {
                    action();
                    btn.style.background = '#555';
                    if (id === 'down-btn') {
                        pressTimer = setInterval(action, 100);
                    }
                }
            });

            btn.addEventListener('touchend', () => {
                btn.style.background = '#333';
                if (pressTimer) {
                    clearInterval(pressTimer);
                }
            });
        };

        addMobileButton('left-btn', () => move(-1));
        addMobileButton('right-btn', () => move(1));
        addMobileButton('down-btn', () => drop());
        addMobileButton('rotate-btn', () => rotate());
        addMobileButton('drop-btn', () => hardDrop());
        
        document.getElementById('pause-btn').addEventListener('touchstart', (e) => {
            e.preventDefault();
            togglePause();
        });

        // Prevent default touch behaviors
        document.addEventListener('touchmove', (e) => {
            if (e.target.tagName !== 'CANVAS') {
                e.preventDefault();
            }
        }, { passive: false });

        currentPiece = createPiece();
        update();
    </script>
</body>
</html>
