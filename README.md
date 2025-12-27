# angel[Uploading index (2).html…]()
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=10">
    <title>Angel Game</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@470/css/font-awesome.min.css" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#4682B4',
                        secondary: '#32CD32',
                        danger: '#DC143C',
                        warning: '#FFD700',
                        info: '#1E90FF',
                        light: '#F8F9FA',
                        dark: '#343A40',
                        'bg-color': '#F0F0F0',
                        'panel-color': '#FAFAFA',
                        'button-blue': '#4682E6',
                        'button-gray': '#B4B4B4',
                        'button-red': '#C83C3C',
                        'button-disabled': '#969696',
                    },
                    fontFamily: {
                        sans: ['Arial', 'sans-serif'],
                    }
                }
            }
        }
    </script>
    <style type="text/tailwindcss">
        @layer utilities {
            .grid-cell {
                width: 15px;
                height: 15px;
                border: 1px solid #DCDCDC;
                background-color: white;
            }
            .grid-cell-eliminated {
                background-color: #C8C8C8 !important;
                border: 1px solid #A0A0A0 !important;
            }
            .grid-cell-path {
                border-radius: 50%;
                background-color: orange !important;
            }
            .angel {
                width: 12px;
                height: 12px;
                border-radius: 50%;
                background-color: #FF0000;
                display: flex;
                align-items: center;
                justify-content: center;
                color: white;
                font-size: 10px;
                font-weight: bold;
            }
            .btn {
                @apply px-4 py-2 rounded-lg font-medium transition-all duration-200 flex items-center justify-center;
            }
            .btn-primary {
                @apply bg-button-blue text-white hover:bg-blue-700;
            }
            .btn-secondary {
                @apply bg-button-gray text-white hover:bg-gray-600;
            }
            .btn-danger {
                @apply bg-button-red text-white hover:bg-red-700;
            }
            .btn-success {
                @apply bg-green-500 text-white hover:bg-green-600;
            }
            .btn-disabled {
                @apply bg-button-disabled text-white cursor-not-allowed;
            }
            .panel {
                @apply bg-panel-color p-4 rounded-lg shadow-md;
            }
            .message {
                @apply bg-red-50 border border-red-200 text-red-600 p-3 rounded-lg;
            }
            .status-badge {
                @apply px-2 py-1 rounded text-xs font-medium;
            }
            .status-active {
                @apply bg-green-100 text-green-800;
            }
            .status-over {
                @apply bg-red-100 text-red-800;
            }
        }
    </style>
</head>
<body class="bg-bg-color min-h-screen font-sans">
    <div class="container mx-auto px-4 py-8">
        <div class="flex flex-col lg:flex-row gap-6">
            <!-- Game Area -->
            <div class="lg:w-2/3">
                <div class="bg-white rounded-lg shadow-lg p-4">
                    <div class="flex justify-between items-center mb-4">
                        <h1 class="text-2xl font-bold text-gray-800">Angel Game</h1>
                        <div class="flex gap-2">
                            <button id="resetBtn" class="btn btn-secondary text-sm">
                                <i class="fa fa-refresh mr-1"></i> 重置 (Ctrl+N)
                            </button>
                        </div>
                    </div>
                    <div class="relative overflow-auto" style="max-width: 615px; max-height: 615px; background-color: #f8f9fa; border: 1px solid #dee2e6; padding: 10px;">
                        <div id="gameGrid" style="display: grid; grid-template-columns: repeat(41, 15px); gap: 0; border: 1px solid #D1D5DB; background-color: white;"></div>
                    </div>
                    <div id="gameMessage" class="mt-4 hidden message"></div>
                </div>
            </div>

            <!-- Info Panel -->
            <div class="lg:w-1/3">
                <div class="panel h-full">
                    <h2 class="text-xl font-bold mb-4 text-gray-800 border-b pb-2">Game Info</h2>
                    
                    <div class="space-y-4">
                        <div class="grid grid-cols-2 gap-2">
                            <div class="bg-gray-50 p-3 rounded">
                                <div class="text-sm text-gray-600">Turn</div>
                                <div id="turnCount" class="text-lg font-semibold">0</div>
                            </div>
                            <div class="bg-gray-50 p-3 rounded">
                                <div class="text-sm text-gray-600">Position</div>
                                <div id="angelPos" class="text-lg font-semibold">(0, 0)</div>
                            </div>
                            <div class="bg-gray-50 p-3 rounded">
                                <div class="text-sm text-gray-600">Eliminated</div>
                                <div id="eliminatedCount" class="text-lg font-semibold">0</div>
                            </div>
                            <div class="bg-gray-50 p-3 rounded">
                                <div class="text-sm text-gray-600">Status</div>
                                <div id="gameStatus" class="status-badge status-active">进行中</div>
                            </div>
                        </div>

                        <div class="space-y-3 mt-6">
                            <button id="undoBtn" class="btn btn-primary w-full">
                                <i class="fa fa-undo mr-1"></i> 撤销 (Ctrl+R)
                            </button>
                            <button id="togglePathBtn" class="btn btn-secondary w-full">
                                <i class="fa fa-eye mr-1"></i> 路径显示 (Ctrl+P)
                            </button>
                        </div>

                        <div class="text-xs text-gray-500 mt-4">
                            Hint: Click on grid cells to eliminate them and trap the Angel!
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        class Game {
            constructor() {
                this.GRID_SIZE = 41;
                this.OFFSET = Math.floor(this.GRID_SIZE / 2);
                thisangelPos = [0, 0];
                this.eliminated = Array(this.GRID_SIZE).fill().map(() => Array(this.GRID_SIZE).fill(false));
                thisturn = 0;
                this.gameOver = false;
                this.message = "";
                this.showPath = false;
                this.history = [];
                this.currentPath = [];
                this.messageType = "";
                
                this.initializeGrid();
                this.bindEvents();
                this.updateDisplay();
            }
            
            reset() {
                thisangelPos = [0, 0];
                this.eliminated = Array(this.GRID_SIZE).fill().map(() => Array(this.GRID_SIZE).fill(false));
                thisturn = 0;
                this.gameOver = false;
                this.message = "";
                this.showPath = false;
                this.history = [];
                this.currentPath = [];
                this.messageType = "";
                
                this.updateGrid();
                this.updateDisplay();
                this.hideMessage();
            }
            
            saveState() {
                const state = {
                    angelPos: [...thisangelPos],
                    eliminated: this.eliminated.map(row => [...row]),
                    turn: thisturn,
                    gameOver: this.gameOver,
                    message: this.message,
                    messageType: this.messageType
                };
                this.history.push(state);
                if (this.history.length > 20) {
                    this.history.shift();
                }
            }
            
            undo() {
                if (this.history.length === 0) {
                    this.showMessage("No history", "game_over");
                    return false;
                }
                
                const lastState = this.history.pop();
                thisangelPos = lastStateangelPos;
                this.eliminated = lastState.eliminated;
                thisturn = lastStateturn;
                this.gameOver = lastState.gameOver;
                this.message = "Undone";
                this.messageType = "game_over";
                
                this.updateGrid();
                this.updateDisplay();
                this.showMessage(this.message, this.messageType);
                return true;
            }
            
            userToIndex(x, y) {
                return [x + this.OFFSET, y + this.OFFSET];
            }
            
            indexToUser(i, j) {
                return [i - this.OFFSET, j - this.OFFSET];
            }
            
            clearMessage() {
                if (["invalid_angel", "already_eliminated", "angel_moving"].includes(this.messageType)) {
                    this.message = "";
                    this.messageType = "";
                    this.hideMessage();
                }
            }
            
            eliminate(x, y) {
                this.saveState();
                const [ix, iy] = this.userToIndex(x, y);
                
                if (ix >= 0 && ix < this.GRID_SIZE && iy >= 0 && iy < this.GRID_SIZE) {
                    if (!this.eliminated[ix][iy] && JSON.stringify([x, y]) !== JSON.stringify(thisangelPos)) {
                        this.eliminated[ix][iy] = true;
                        this.clearMessage();
                        this.updateGrid();
                        this.updateDisplay();
                        
                        setTimeout(() => {
                            this.moveAngel();
                        }, 300);
                        
                        return true;
                    } else {
                        if (JSON.stringify([x, y]) === JSON.stringify(thisangelPos)) {
                            this.showMessage("Cannot eliminate Angel's position", "invalid_angel");
                        } else {
                            this.showMessage("Cell already eliminated", "already_eliminated");
                        }
                    }
                }
                return false;
            }
            
            isEnclosed() {
                const visited = Array(this.GRID_SIZE).fill().map(() => Array(this.GRID_SIZE).fill(false));
                const queue = [];
                const [ix, iy] = this.userToIndex(thisangelPos[0], thisangelPos[1]);
                queue.push([ix, iy]);
                visited[ix][iy] = true;
                
                while (queue.length > 0) {
                    const [x, y] = queue.shift();
                    
                    if (x === 0 || x === this.GRID_SIZE - 1 || y === 0 || y === this.GRID_SIZE - 1) {
                        return false;
                    }
                    
                    for (let dx = -1; dx <= 1; dx++) {
                        for (let dy = -1; dy <= 1; dy++) {
                            if (dx === 0 && dy === 0) continue;
                            
                            const nx = x + dx;
                            const ny = y + dy;
                            
                            if (nx >= 0 && nx < this.GRID_SIZE && ny >= 0 && ny < this.GRID_SIZE) {
                                if (!this.eliminated[nx][ny] && !visited[nx][ny]) {
                                    visited[nx][ny] = true;
                                    queue.push([nx, ny]);
                                }
                            }
                        }
                    }
                }
                
                return true;
            }
            
            findShortestPath() {
                const [startIx, startIy] = this.userToIndex(thisangelPos[0], thisangelPos[1]);
                const visited = Array(this.GRID_SIZE).fill().map(() => Array(this.GRID_SIZE).fill(false));
                const parent = Array(this.GRID_SIZE).fill().map(() => Array(this.GRID_SIZE).fill(null));
                const queue = [];
                queue.push([startIx, startIy]);
                visited[startIx][startIy] = true;
                
                let target = null;
                
                while (queue.length > 0) {
                    const [x, y] = queue.shift();
                    const [userX, userY] = this.indexToUser(x, y);
                    
                    if (Math.abs(userX) === this.OFFSET || Math.abs(userY) === this.OFFSET) {
                        target = [x, y];
                        break;
                    }
                    
                    const directions = [];
                    for (let dx = -1; dx <= 1; dx++) {
                        for (let dy = -1; dy <= 1; dy++) {
                            if (dx !== 0 || dy !== 0) {
                                directions.push([dx, dy]);
                            }
                        }
                    }
                    
                    for (const [dx, dy] of directions) {
                        const nx = x + dx;
                        const ny = y + dy;
                        const [nxUser, nyUser] = this.indexToUser(nx, ny);
                        
                        if (Math.abs(nxUser) > this.OFFSET || Math.abs(nyUser) > this.OFFSET) {
                            continue;
                        }
                        
                        if (nx >= 0 && nx < this.GRID_SIZE && ny >= 0 && ny < this.GRID_SIZE) {
                            if (!this.eliminated[nx][ny] && !visited[nx][ny]) {
                                visited[nx][ny] = true;
                                parent[nx][ny] = [x, y];
                                queue.push([nx, ny]);
                            }
                        }
                    }
                }
                
                if (target === null) {
                    return [];
                }
                
                const path = [];
                let current = target;
                while (JSON.stringify(current) !== JSON.stringify([startIx, startIy])) {
                    path.push(current);
                    current = parent[current[0]][current[1]];
                }
                path.reverse();
                
                const userPath = path.map(([x, y]) => this.indexToUser(x, y));
                return userPath;
            }
            
            findBestMove() {
                const path = this.findShortestPath();
                return path.length > 0 ? path[0] : null;
            }
            
            moveAngel() {
                if (this.gameOver) return false;
                
                this.clearMessage();
                
                const bestMove = this.findBestMove();
                if (bestMove === null) {
                    this.gameOver = true;
                    this.showMessage("Angel trapped! Player wins!", "game_over");
                    this.updateDisplay();
                    return false;
                }
                
                thisangelPos = [...bestMove];
                thisturn++;
                this.currentPath = this.findShortestPath();
                
                if (Math.abs(thisangelPos[0]) === this.OFFSET || Math.abs(thisangelPos[1]) === this.OFFSET) {
                    this.gameOver = true;
                    this.showMessage("Angel reached the boundary!", "game_over");
                    this.updateDisplay();
                    return false;
                }
                
                if (this.isEnclosed()) {
                    this.gameOver = true;
                    this.showMessage("Angel surrounded! Player wins!", "game_over");
                    this.updateDisplay();
                    return false;
                }
                
                this.updateGrid();
                this.updateDisplay();
                return true;
            }
            
            initializeGrid() {
                const grid = document.getElementById('gameGrid');
                grid.innerHTML = '';
                
                for (let j = 0; j < this.GRID_SIZE; j++) {
                    for (let i = 0; i < this.GRID_SIZE; i++) {
                        const cell = document.createElement('div');
                        cell.className = 'grid-cell bg-white';
                        cell.dataset.x = i - this.OFFSET;
                        cell.dataset.y = j - this.OFFSET;
                        
                        cell.addEventListener('click', () => {
                            if (!this.gameOver) {
                                const x = parseInt(cell.dataset.x);
                                const y = parseInt(cell.dataset.y);
                                this.eliminate(x, y);
                            }
                        });
                        
                        grid.appendChild(cell);
                    }
                }
                
                this.updateGrid();
            }
            
            updateGrid() {
                const cells = document.querySelectorAll('#gameGrid .grid-cell');
                
                cells.forEach(cell => {
                    const x = parseInt(cell.dataset.x);
                    const y = parseInt(cell.dataset.y);
                    const [ix, iy] = this.userToIndex(x, y);
                    
                    cell.innerHTML = '';
                    cell.classList.remove('grid-cell-eliminated', 'grid-cell-path');
                    
                    if (this.eliminated[ix][iy]) {
                        cell.classList.add('grid-cell-eliminated');
                    }
                    
                    if (x === thisangelPos[0] && y === thisangelPos[1]) {
                        const angel = document.createElement('div');
                        angel.className = 'angel';
                        angel.textContent = 'A';
                        cell.appendChild(angel);
                    }
                });
                
                if (this.showPath && !this.gameOver) {
                    this.currentPath = this.findShortestPath();
                    
                    if (this.currentPath.length > 0) {
                        this.currentPath.forEach(([x, y], index) => {
                            const cell = document.querySelector(`[data-x="${x}"][data-y="${y}"]`);
                            if (cell && !this.eliminated[this.userToIndex(x, y)[0]][this.userToIndex(x, y)[1]]) {
                                cell.classList.add('grid-cell-path');
                                const ratio = index / Math.max(this.currentPath.length, 1);
                                const r = Math.floor(255 * (1 - ratio));
                                const g = Math.floor(255 * ratio);
                                cell.style.backgroundColor = `rgb(${r}, ${g}, 0) !important`;
                            }
                        });
                    }
                }
            }
            
            updateDisplay() {
                document.getElementById('turnCount').textContent = thisturn;
                document.getElementById('angelPos').textContent = `(${thisangelPos[0]}, ${thisangelPos[1]})`;
                
                const eliminatedCount = this.eliminated.flat().filter(Boolean).length;
                document.getElementById('eliminatedCount').textContent = eliminatedCount;
                
                const statusElement = document.getElementById('gameStatus');
                if (this.gameOver) {
                    statusElement.textContent = '已结束';
                    statusElement.className = 'status-badge status-over';
                } else {
                    statusElement.textContent = '进行中';
                    statusElement.className = 'status-badge status-active';
                }
                
                const undoBtn = document.getElementById('undoBtn');
                if (this.history.length > 0) {
                    undoBtn.classList.remove('btn-disabled');
                    undoBtn.classList.add('btn-primary');
                    undoBtn.disabled = false;
                } else {
                    undoBtn.classList.remove('btn-primary');
                    undoBtn.classList.add('btn-disabled');
                    undoBtn.disabled = true;
                }
            }
            
            showMessage(text, type) {
                this.message = text;
                this.messageType = type;
                
                const messageElement = document.getElementById('gameMessage');
                messageElement.textContent = text;
                messageElement.classList.remove('hidden');
                
                if (["invalid_angel", "already_eliminated", "angel_moving"].includes(type)) {
                    setTimeout(() => {
                        this.hideMessage();
                    }, 3000);
                }
            }
            
            hideMessage() {
                const messageElement = document.getElementById('gameMessage');
                messageElement.classList.add('hidden');
            }
            
            bindEvents() {
                document.getElementById('resetBtn').addEventListener('click', () => this.reset());
                document.getElementById('undoBtn').addEventListener('click', () => thisundo());
                document.getElementById('togglePathBtn').addEventListener('click', () => {
                    this.showPath = !this.showPath;
                    this.updateGrid();
                });

                document.addEventListener('keydown', (e) => {
                    if (ectrlKey) {
                        switch (e.key.toLowerCase()) {
                            case 'r':
                                e.preventDefault();
                                thisundo();
                                break;
                            case 'p':
                                e.preventDefault();
                                this.showPath = !this.showPath;
                                this.updateGrid();
                                break;
                            case 'n':
                                e.preventDefault();
                                this.reset();
                                break;
                        }
                    } else {
                        switch (e.key.toLowerCase()) {
                            case 'r':
                                thisundo();
                                break;
                            case 'p':
                                this.showPath = !this.showPath;
                                this.updateGrid();
                                break;
                            case 'n':
                                this.reset();
                                break;
                        }
                    }
                });
            }
        }
        
        document.addEventListener('DOMContentLoaded', () => {
            const game = new Game();
        });
    </script>
</body>
</html>
