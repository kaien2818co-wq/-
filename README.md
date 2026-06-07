<!

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
