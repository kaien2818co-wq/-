<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Block Blast Minimal</title>
    <style>
        :root {
            --bg-color: #0f1115;
            --grid-bg: #1a1d24;
            --cell-empty: #282c37;
            --text-color: #ffffff;
        }

        body {
            font-family: 'Helvetica Neue', Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            padding: 10px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            user-select: none;
            -webkit-user-select: none;
        }

        .game-wrapper {
            width: 100%;
            max-width: 400px;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        /* スコア表示 */
        .score-container {
            display: flex;
            justify-content: space-between;
            width: 100%;
        }

        .score-box {
            background: #1f232e;
            padding: 10px 20px;
            border-radius: 10px;
            text-align: center;
            flex: 1;
            margin: 0 5px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
        }

        .score-label {
            font-size: 11px;
            color: #8e94a6;
            letter-spacing: 1px;
            margin-bottom: 2px;
        }

        .score-val {
            font-size: 24px;
            font-weight: bold;
            color: #ffeb3b;
        }

        /* 10x10のメイン盤面 */
        #grid {
            display: grid;
            grid-template-columns: repeat(10, 1fr);
            grid-template-rows: repeat(10, 1fr);
            gap: 4px;
            background-color: var(--grid-bg);
            padding: 8px;
            border-radius: 12px;
            width: 100%;
            aspect-ratio: 1 / 1;
            box-sizing: border-box;
            box-shadow: 0 10px 25px rgba(0,0,0,0.5);
        }

        .cell {
            background-color: var(--cell-empty);
            border-radius: 4px;
            transition: background-color 0.15s ease, transform 0.1s;
            cursor: pointer;
        }

        /* 下部のブロック置き場 */
        #dock {
            display: flex;
            justify-content: space-around;
            align-items: center;
            width: 100%;
            height: 110px;
            background: #15181f;
            border-radius: 12px;
            padding: 10px;
            box-sizing: border-box;
            box-shadow: inset 0 2px 8px rgba(0,0,0,0.5);
        }

        .dock-slot {
            width: 30%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 8px;
            border: 2px dashed transparent;
            box-sizing: border-box;
            cursor: pointer;
            transition: all 0.2s;
        }

        .dock-slot.selected {
            border-color: #ff3366;
            background: rgba(255, 51, 102, 0.1);
            transform: scale(1.05);
        }

        .dock-slot.disabled {
            opacity: 0.2;
            pointer-events: none;
        }

        .piece-grid {
            display: grid;
            gap: 2px;
        }

        /* ブロックの色パターン */
        .c-1 { background: #ff3366; box-shadow: inset -2px -2px 5px rgba(0,0,0,0.4), inset 2px 2px 5px rgba(255,255,255,0.4); }
        .c-2 { background: #33ccff; box-shadow: inset -2px -2px 5px rgba(0,0,0,0.4), inset 2px 2px 5px rgba(255,255,255,0.4); }
        .c-3 { background: #33ff66; box-shadow: inset -2px -2px 5px rgba(0,0,0,0.4), inset 2px 2px 5px rgba(255,255,255,0.4); }
        .c-4 { background: #ffcc00; box-shadow: inset -2px -2px 5px rgba(0,0,0,0.4), inset 2px 2px 5px rgba(255,255,255,0.4); }
        .c-5 { background: #ae33ff; box-shadow: inset -2px -2px 5px rgba(0,0,0,0.4), inset 2px 2px 5px rgba(255,255,255,0.4); }
        .c-6 { background: #ff6633; box-shadow: inset -2px -2px 5px rgba(0,0,0,0.4), inset 2px 2px 5px rgba(255,255,255,0.4); }

        /* ゲームオーバー画面 */
        #overlay {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85);
            justify-content: center;
            align-items: center;
            z-index: 10;
        }

        .modal {
            background: #1f232e;
            padding: 30px;
            border-radius: 16px;
            text-align: center;
            box-shadow: 0 10px 30px rgba(0,0,0,0.6);
            max-width: 300px;
            width: 80%;
        }

        button {
            background: #ff3366;
            color: white;
            border: none;
            padding: 12px 30px;
            font-size: 16px;
            font-weight: bold;
            border-radius: 25px;
            cursor: pointer;
            margin-top: 15px;
            box-shadow: 0 4px 10px rgba(255,51,102,0.4);
            transition: 0.2s;
        }

        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 15px rgba(255,51,102,0.6);
        }
    </style>
</head>
<body>

<div class="game-wrapper">
    <div class="score-container">
        <div class="score-box">
            <div class="score-label">SCORE</div>
            <div id="score" class="score-val">0</div>
        </div>
        <div class="score-box">
            <div class="score-label">BEST</div>
            <div id="best" class="score-val">0</div>
        </div>
    </div>

    <div id="grid"></div>

    <div id="dock">
        <div class="dock-slot" id="slot-0"></div>
        <div class="dock-slot" id="slot-1"></div>
        <div class="dock-slot" id="slot-2"></div>
    </div>
</div>

<div id="overlay">
    <div class="modal">
        <h2 style="color: #ff3366; margin-top:0;">GAME OVER</h2>
        <p>スコア: <span id="final-score" style="font-size: 24px; font-weight:bold; color:#ffeb3b;">0</span></p>
        <button onclick="resetGame()">もう一度挑戦</button>
    </div>
</div>

<script>
    const BOARD_SIZE = 10;
    let board = Array(BOARD_SIZE).fill().map(() => Array(BOARD_SIZE).fill(0));
    let score = 0;
    let bestScore = localStorage.getItem('bb_best') || 0;
    let dockPieces = [null, null, null];
    let selectedIdx = null;

    // ブロックの形状データ (1がブロックの存在するマス)
    const SHAPES = [
        { matrix: [[1]], color: 1 },
        { matrix: [[1, 1]], color: 2 },
        { matrix: [[1], [1]], color: 2 },
        { matrix: [[1, 1, 1]], color: 3 },
        { matrix: [[1], [1], [1]], color: 3 },
        { matrix: [[1, 1], [1, 1]], color: 4 },
        { matrix: [[1, 1, 1], [0, 1, 0]], color: 5 },
        { matrix: [[0, 1], [0, 1], [1, 1]], color: 6 },
        { matrix: [[1, 1, 1], [1, 0, 0]], color: 6 }
    ];

    const gridEl = document.getElementById('grid');
    document.getElementById('best').innerText = bestScore;

    // 盤面マスの生成
    function createBoard() {
        gridEl.innerHTML = '';
        for (let r = 0; r < BOARD_SIZE; r++) {
            for (let c = 0; c < BOARD_SIZE; c++) {
                const cell = document.createElement('div');
                cell.classList.add('cell');
                cell.addEventListener('click', (e) => {
                    e.stopPropagation();
                    handleBoardClick(r, c);
                });
                gridEl.appendChild(cell);
            }
        }
    }

    // 盤面の描画更新
    function updateBoardView() {
        const cells = gridEl.children;
        for (let r = 0; r < BOARD_SIZE; r++) {
            for (let c = 0; c < BOARD_SIZE; c++) {
                const idx = r * BOARD_SIZE + c;
                cells[idx].className = 'cell'; 
                if (board[r][c] > 0) {
                    cells[idx].classList.add(`c-${board[r][c]}`);
                }
            }
        }
    }

    // 下部スロットに新しい3つのブロックを補充
    function refreshDock() {
        for (let i = 0; i < 3; i++) {
            const rand = Math.floor(Math.random() * SHAPES.length);
            dockPieces[i] = JSON.parse(JSON.stringify(SHAPES[rand]));
            renderDockPiece(i);
        }
    }

    // 各スロットにミニブロックを描画
    function renderDockPiece(idx) {
        const slot = document.getElementById(`slot-${idx}`);
        slot.innerHTML = '';
        slot.className = 'dock-slot';
        
        const piece = dockPieces[idx];
        if (!piece) {
            slot.classList.add('disabled');
            return;
        }

        const pGrid = document.createElement('div');
        pGrid.classList.add('piece-grid');
        pGrid.style.gridTemplateColumns = `repeat(${piece.matrix[0].length}, 16px)`;

        for (let r = 0; r < piece.matrix.length; r++) {
            for (let c = 0; c < piece.matrix[0].length; c++) {
                const block = document.createElement('div');
                block.style.width = '16px';
                block.style.height = '16px';
                if (piece.matrix[r][c] === 1) {
                    block.classList.add(`c-${piece.color}`);
                    block.style.border = '1px solid rgba(0,0,0,0.1)';
                    block.style.borderRadius = '2px';
                }
                pGrid.appendChild(block);
            }
        }
        slot.appendChild(pGrid);
        slot.onclick = (e) => {
            e.stopPropagation();
            selectPiece(idx);
        };
    }

    // ブロックの選択処理
    function selectPiece(idx) {
        if (!dockPieces[idx]) return;
        
        if (selectedIdx === idx) {
            document.getElementById(`slot-${idx}`).classList.remove('selected');
            selectedIdx = null;
            return;
        }

        for(let i=0; i<3; i++) document.getElementById(`slot-${i}`).classList.remove('selected');
        
        selectedIdx = idx;
        document.getElementById(`slot-${idx}`).classList.add('selected');
    }

    // 盤面をクリックした時にブロックを配置
    function handleBoardClick(row, col) {
        if (selectedIdx === null) return;
        const piece = dockPieces[selectedIdx];
        if (!piece) return;

        if (canPlace(board, piece.matrix, row, col)) {
            // 配置処理
            let placedCount = 0;
            for (let r = 0; r < piece.matrix.length; r++) {
                for (let c = 0; c < piece.matrix[0].length; c++) {
                    if (piece.matrix[r][c] === 1) {
                        board[row + r][col + c] = piece.color;
                        placedCount++;
                    }
                }
            }
            score += placedCount;
            
            // スロットのクリア
            dockPieces[selectedIdx] = null;
            renderDockPiece(selectedIdx);
            selectedIdx = null;

            // ライン消去確認と描画更新
            checkLines();
            updateBoardView();

            // 3つのスロットがすべて空なら再補充
            if (dockPieces.every(p => p === null)) {
                refreshDock();
            }

            // ゲームオーバー判定
            if (isGameOver()) {
                document.getElementById('final-score').innerText = score;
                document.getElementById('overlay').style.display = 'flex';
            }
        }
    }

    // 指定位置にブロックが置けるかチェック
    function canPlace(targetBoard, matrix, sRow, sCol) {
        for (let r = 0; r < matrix.length; r++) {
            for (let c = 0; c < matrix[0].length; c++) {
                if (matrix[r][c] === 1) {
                    const tRow = sRow + r;
                    const tCol = sCol + c;
                    if (tRow >= BOARD_SIZE || tCol >= BOARD_SIZE || targetBoard[tRow][tCol] > 0) {
                        return false;
                    }
                }
            }
        }
        return true;
    }

    // 揃った列（縦・横）を消去する
    function checkLines() {
        let rowsToClear = [];
        let colsToClear = [];

        for (let r = 0; r < BOARD_SIZE; r++) {
            if (board[r].every(v => v > 0)) rowsToClear.push(r);
        }

        for (let c = 0; c < BOARD_SIZE; c++) {
            let filled = true;
            for (let r = 0; r < BOARD_SIZE; r++) {
                if (board[r][c] === 0) { filled = false; break; }
            }
            if (filled) colsToClear.push(c);
        }

        if (rowsToClear.length > 0 || colsToClear.length > 0) {
            const clearedLines = rowsToClear.length + colsToClear.length;
            score += clearedLines * 100; // コンボボーナス
            
            rowsToClear.forEach(r => board[r] = Array(BOARD_SIZE).fill(0));
            colsToClear.forEach(c => {
                for (let r = 0; r < BOARD_SIZE; r++) board[r][c] = 0;
            });
        }
        
        document.getElementById('score').innerText = score;
        if (score > bestScore) {
            bestScore = score;
            document.getElementById('best').innerText = bestScore;
            localStorage.setItem('bb_best', bestScore);
        }
    }

    // 残りのブロックがどこにも置けないか判定
    function isGameOver() {
        for (let i = 0; i < 3; i++) {
            const piece = dockPieces[i];
            if (!piece) continue;

            for (let r = 0; r < BOARD_SIZE; r++) {
                for (let c = 0; c < BOARD_SIZE; c++) {
                    if (canPlace(board, piece.matrix, r, c)) return false;
                }
            }
        }
        return true;
    }

    // ゲームリセット
    function resetGame() {
        board = Array(BOARD_SIZE).fill().map(() => Array(BOARD_SIZE).fill(0));
        score = 0;
        selectedIdx = null;
        document.getElementById('score').innerText = 0;
        document.getElementById('overlay').style.display = 'none';
        createBoard();
        updateBoardView();
        refreshDock();
    }

    // 選択解除用
    document.body.onclick = () => {
        if (selectedIdx !== null) {
            document.getElementById(`slot-${selectedIdx}`).classList.remove('selected');
            selectedIdx = null;
        }
    };

    // 初期起動
    createBoard();
    updateBoardView();
    refreshDock();
</script>

</body>
</html>
