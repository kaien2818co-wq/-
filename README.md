<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ブロックブラスト風パズル</title>
    <style>
        body {
            font-family: 'Helvetica Neue', Arial, sans-serif;
            background-color: #121214;
            color: #fff;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            overflow: hidden;
            user-select: none;
            touch-action: none;
        }

        #game-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            max-width: 400px;
            width: 100%;
            padding: 10px;
        }

        header {
            display: flex;
            justify-content: space-between;
            width: 100%;
            margin-bottom: 15px;
        }

        .score-box {
            background-color: #1e1e22;
            padding: 8px 16px;
            border-radius: 8px;
            text-align: center;
            min-width: 80px;
        }

        .score-label {
            font-size: 12px;
            color: #aaa;
            text-transform: uppercase;
        }

        .score-value {
            font-size: 20px;
            font-weight: bold;
            color: #ffcc00;
        }

        #grid {
            display: grid;
            grid-template-columns: repeat(8, 1fr);
            grid-template-rows: repeat(8, 1fr);
            gap: 4px;
            background-color: #1e1e22;
            padding: 8px;
            border-radius: 12px;
            width: 90vw;
            height: 90vw;
            max-width: 360px;
            max-height: 360px;
            box-shadow: 0 8px 24px rgba(0,0,0,0.5);
        }

        .cell {
            background-color: #2a2a30;
            border-radius: 4px;
            transition: background-color 0.2s;
        }

        .cell.filled {
            box-shadow: inset 0 0 8px rgba(0,0,0,0.3);
        }

        #pieces-container {
            display: flex;
            justify-content: space-around;
            align-items: center;
            width: 100%;
            height: 120px;
            margin-top: 25px;
            background-color: #1e1e22;
            border-radius: 12px;
            padding: 10px 0;
        }

        .piece-slot {
            width: 100px;
            height: 100px;
            display: flex;
            align-items: center;
            justify-content: center;
            position: relative;
        }

        .piece {
            display: grid;
            gap: 2px;
            position: absolute;
            cursor: grab;
            transition: transform 0.1s;
        }

        .piece.dragging {
            cursor: grabbing;
            transform: scale(1.1);
            opacity: 0.9;
            z-index: 100;
            position: fixed; /* ドラッグ中は画面絶対配置に切り替え */
            pointer-events: none; /* 下の要素を検知できるようにする */
        }

        .block {
            border-radius: 4px;
            box-shadow: inset 0 0 4px rgba(255,255,255,0.4);
        }

        #game-over {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0,0,0,0.85);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 200;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s ease;
        }

        #game-over.show {
            opacity: 1;
            pointer-events: auto;
        }

        #game-over h1 {
            color: #ff4444;
            font-size: 40px;
            margin-bottom: 10px;
        }

        #game-over p {
            font-size: 20px;
            margin-bottom: 20px;
        }

        #restart-btn {
            background-color: #ffcc00;
            color: #121214;
            border: none;
            padding: 12px 30px;
            font-size: 18px;
            font-weight: bold;
            border-radius: 25px;
            cursor: pointer;
            box-shadow: 0 4px 12px rgba(255,204,0,0.4);
            transition: transform 0.1s;
        }

        #restart-btn:active {
            transform: scale(0.95);
        }
    </style>
</head>
<body>

    <div id="game-container">
        <header>
            <div class="score-box">
                <div class="score-label">SCORE</div>
                <div id="score" class="score-value">0</div>
            </div>
            <div class="score-box">
                <div class="score-label">BEST</div>
                <div id="best-score" class="score-value">0</div>
            </div>
        </header>

        <div id="grid"></div>

        <div id="pieces-container">
            <div class="piece-slot" id="slot-0"></div>
            <div class="piece-slot" id="slot-1"></div>
            <div class="piece-slot" id="slot-2"></div>
        </div>
    </div>

    <div id="game-over">
        <h1>GAME OVER</h1>
        <p>最終スコア: <span id="final-score">0</span></p>
        <button id="restart-btn">もう一度プレイ</button>
    </div>

    <script>
        const GRID_SIZE = 8;
        const gridElement = document.getElementById('grid');
        const scoreElement = document.getElementById('score');
        const bestScoreElement = document.getElementById('best-score');
        const finalScoreElement = document.getElementById('final-score');
        const gameOverScreen = document.getElementById('game-over');
        const restartBtn = document.getElementById('restart-btn');
        const slots = [document.getElementById('slot-0'), document.getElementById('slot-1'), document.getElementById('slot-2')];

        let grid = Array(GRID_SIZE).fill().map(() => Array(GRID_SIZE).fill(0));
        let score = 0;
        let bestScore = localStorage.getItem('block_blast_best') || 0;
        bestScoreElement.textContent = bestScore;

        // ブロックの定義（形状、色、行数、列数）
        const COLORS = ['#ff3366', '#33ccff', '#33ff66', '#ffcc00', '#9933ff', '#ff9933', '#00eeee'];
        const SHAPES = [
            { shape: [[1]], color: 0 }, // 1x1
            { shape: [[1, 1]], color: 1 }, // 1x2 横
            { shape: [[1], [1]], color: 1 }, // 2x1 縦
            { shape: [[1, 1, 1]], color: 2 }, // 1x3 横
            { shape: [[1], [1], [1]], color: 2 }, // 3x1 縦
            { shape: [[1, 1], [1, 1]], color: 3 }, // 2x2 正方形
            { shape: [[1, 1, 1], [0, 1, 0]], color: 4 }, // T型
            { shape: [[1, 0], [1, 0], [1, 1]], color: 5 }, // L型
            { shape: [[1, 1], [0, 1], [0, 1]], color: 6 }  // 逆L型
        ];

        let activePieces = [null, null, null];
        let draggingElement = null;
        let draggingIndex = null;
        let dragOffset = { x: 0, y: 0 };

        // グリッドの初期化
        function initGrid() {
            gridElement.innerHTML = '';
            for (let r = 0; r < GRID_SIZE; r++) {
