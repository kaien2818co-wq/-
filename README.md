<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ブロックブラスト 8×8 完全版</title>
    <style>
        body {
            font-family: 'Arial Rounded MT Bold', 'Helvetica Neue', Arial, sans-serif;
            background-color: #0b0c10;
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
            max-width: 420px;
            width: 100%;
            padding: 15px;
            box-sizing: border-box;
        }

        header {
            display: flex;
            justify-content: space-between;
            width: 100%;
            margin-bottom: 20px;
        }

        .score-box {
            background: linear-gradient(145deg, #1f2330, #161922);
            padding: 10px 20px;
            border-radius: 12px;
            text-align: center;
            min-width: 100px;
            border: 1px solid #2c3245;
        }

        .score-label {
            font-size: 11px;
            color: #8f9cae;
            letter-spacing: 1px;
            margin-bottom: 2px;
        }

        .score-value {
            font-size: 24px;
            font-weight: bold;
            color: #00ffcc;
            text-shadow: 0 0 8px rgba(0,255,204,0.3);
        }

        #grid {
            display: grid;
            grid-template-columns: repeat(8, 1fr);
            grid-template-rows: repeat(8, 1fr);
            gap: 4px;
            background-color: #151821;
            padding: 8px;
            border-radius: 16px;
            width: 92vw;
            height: 92vw;
            max-width: 380px;
            max-height: 380px;
            box-shadow: 0 12px 32px rgba(0,0,0,0.6);
            border: 2px solid #222735;
        }

        .cell {
            background-color: #202433;
            border-radius: 6px;
            transition: background-color 0.15s ease, transform 0.1s;
        }

        .cell.filled {
            box-shadow: inset 0 0 10px rgba(0,0,0,0.5), 0 2px 4px rgba(0,0,0,0.3);
        }

        #pieces-container {
            display: flex;
            justify-content: space-around;
            align-items: center;
            width: 100%;
            height: 140px;
            margin-top: 25px;
            background-color: #151821;
            border-radius: 16px;
            padding: 10px;
            box-sizing: border-box;
            border: 1px solid #222735;
        }

        .piece-slot {
            width: 110px;
            height: 110px;
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
        }

        .piece.dragging {
            cursor: grabbing;
            transform: scale(1.1);
            opacity: 0.95;
            z-index: 100;
            position: fixed;
            pointer-events: none;
        }

        .block {
            border-radius: 5px;
            box-shadow: inset 0 3px 6px rgba(255,255,255,0.3), inset 0 -3px 6px rgba(0,0,0,0.3);
        }

        #game-over {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(7, 8, 12, 0.9);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 200;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s ease;
            backdrop-filter: blur(5px);
        }

        #game-over.show {
            opacity: 1;
            pointer-events: auto;
        }

        #game-over h1 {
            color: #ff3366;
            font-size: 46px;
            margin-bottom: 5px;
            text-shadow: 0 0 15px rgba(255,51,102,0.4);
        }

        #game-over p {
            font-size: 22px;
            margin-bottom: 30px;
            color: #cfd8dc;
        }

        #restart-btn {
            background: linear-gradient(135deg, #00ffcc, #00b3ff);
            color: #0b0c10;
            border: none;
            padding: 15px 40px;
            font-size: 20px;
            font-weight: bold;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 6px 20px rgba(0,255,204,0.4);
            transition: transform 0.1s, box-shadow 0.1s;
        }

        #restart-btn:active {
            transform: scale(0.95);
            box-shadow: 0 2px 10px rgba(0,255,204,0.2);
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
        <p>SCORE: <span id="final-score">0</span></p>
        <button id="restart-btn">もう一度プレイ</button>
    </div>

    <script>
        const GRID_SIZE = 8; // しっかり8×8マス
        const gridElement = document.getElementById('grid');
        const scoreElement = document.getElementById('score');
        const bestScoreElement = document.getElementById('best-score');
        const finalScoreElement = document.getElementById('final-score');
        const gameOverScreen = document.getElementById('game-over');
        const restartBtn = document.getElementById('restart-btn');
        const slots = [document.getElementById('slot-0'), document.getElementById('slot-1'), document.getElementById('slot-2')];

        let grid = Array(GRID_SIZE).fill().map(() => Array(GRID_SIZE).fill(0));
        let score = 0;
        let bestScore = localStorage.getItem('block_blast_8x8_best') || 0;
        bestScoreElement.textContent = bestScore
