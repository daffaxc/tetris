<!DOCTYPE html>
<html>
<head>
    <title>Tetris Game</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <h1>TETRIS</h1>
        <div class="game-stats">
            Score: <span id="score">0</span><br>
            Best Score: <span id="best-score">0</span>
        </div>
        <canvas id="game-board" width="240" height="480"></canvas>
        <div class="pause-overlay" id="pause-text">PAUSED</div>
        <img id="achievement-image" src="achievement.png" alt="New High Score!">
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
            <p>created by daffa alzeinna</data>
            </p>
        </footer>
    </div>

    <script src="tetris.js"></script>
</body>
</html>
