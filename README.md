<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>九宮格圖樣記憶挑戰</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background-color: #f0f2f5;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            background: white;
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            text-align: center;
            max-width: 400px;
            width: 100%;
        }

        h1 {
            font-size: 24px;
            color: #333;
            margin-bottom: 10px;
        }

        .rules {
            font-size: 14px;
            color: #666;
            margin-bottom: 20px;
            line-height: 1.5;
        }

        .status-bar {
            display: flex;
            justify-content: space-around;
            margin-bottom: 20px;
            font-weight: bold;
            font-size: 16px;
            color: #444;
        }

        .grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 12px;
            margin-bottom: 25px;
        }

        .card {
            aspect-ratio: 1;
            background-color: #2c3e50;
            border-radius: 12px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 36px;
            font-weight: bold;
            color: white;
            cursor: pointer;
            user-select: none;
            transition: transform 0.3s, background-color 0.3s;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        .card:hover:not(.flipped):not(.disabled) {
            transform: translateY(-3px);
            background-color: #34495e;
        }

        .card.flipped {
            background-color: #ffffff;
            border: 3px solid #3498db;
            color: #2c3e50;
            cursor: default;
        }

        .card.o-style { color: #e74c3c; }
        .card.x-style { color: #2980b9; }
        .card.t-style { color: #2ecc71; }

        .btn {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 12px 24px;
            font-size: 16px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
            transition: background 0.2s;
        }

        .btn:hover {
            background-color: #2980b9;
        }

        .message {
            margin-top: 15px;
            font-size: 18px;
            font-weight: bold;
            min-height: 27px;
        }

        .win { color: #2ecc71; }
        .lose { color: #e74c3c; }
    </style>
</head>
<body>

<div class="container">
    <h1>九宮格配對挑戰</h1>
    <p class="rules">翻開 3 張相同符號（Ｏ、Ｘ、△）即獲勝！<br>每次翻 3 張，共有 3 次機會。</p>
    
    <div class="status-bar">
        <div>剩餘機會：<span id="attempts">3</span></div>
        <div>本回合翻牌：<span id="picked-count">0</span>/3</div>
    </div>

    <div class="grid" id="grid"></div>

    <button class="btn" onclick="initGame()">重新開始</button>
    <div class="message" id="message"></div>
</div>

<script>
    const symbols = ['Ｏ', 'Ｏ', 'Ｏ', 'Ｘ', 'Ｘ', 'Ｘ', '△', '△', '△'];
    let cards = [];
    let selectedIndices = [];
    let remainingAttempts = 3;
    let isProcessing = false;

    function shuffle(array) {
        for (let i = array.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [array[i], array[j]] = [array[j], array[i]];
        }
        return array;
    }

    function initGame() {
        cards = shuffle([...symbols]);
        selectedIndices = [];
        remainingAttempts = 3;
        isProcessing = false;
        
        document.getElementById('attempts').textContent = remainingAttempts;
        document.getElementById('picked-count').textContent = '0';
        document.getElementById('message').textContent = '';
        document.getElementById('message').className = 'message';

        renderGrid();
    }

    function renderGrid() {
        const grid = document.getElementById('grid');
        grid.innerHTML = '';
        cards.forEach((_, index) => {
            const cardEl = document.createElement('div');
            cardEl.className = 'card';
            cardEl.dataset.index = index;
            cardEl.onclick = () => handleCardClick(index);
            grid.appendChild(cardEl);
        });
    }

    function handleCardClick(index) {
        if (isProcessing || remainingAttempts <= 0) return;
        if (selectedIndices.includes(index)) return;

        selectedIndices.push(index);
        revealCard(index);
        document.getElementById('picked-count').textContent = selectedIndices.length;

        if (selectedIndices.length === 3) {
            isProcessing = true;
            checkMatch();
        }
    }

    function revealCard(index) {
        const grid = document.getElementById('grid');
        const cardEl = grid.children[index];
        const symbol = cards[index];
        
        cardEl.textContent = symbol;
        cardEl.classList.add('flipped');

        if (symbol === 'Ｏ') cardEl.classList.add('o-style');
        if (symbol === 'Ｘ') cardEl.classList.add('x-style');
        if (symbol === '△') cardEl.classList.add('t-style');
    }

    function hideCard(index) {
        const grid = document.getElementById('grid');
        const cardEl = grid.children[index];
        cardEl.textContent = '';
        cardEl.className = 'card';
    }

    function checkMatch() {
        const firstSymbol = cards[selectedIndices[0]];
        const isWin = selectedIndices.every(idx => cards[idx] === firstSymbol);

        if (isWin) {
            document.getElementById('message').textContent = '🎉 恭喜通關！成功配對 3 張相同圖案！';
            document.getElementById('message').className = 'message win';
            disableAllCards();
        } else {
            remainingAttempts--;
            document.getElementById('attempts').textContent = remainingAttempts;

            setTimeout(() => {
                if (remainingAttempts > 0) {
                    selectedIndices.forEach(idx => hideCard(idx));
                    selectedIndices = [];
                    document.getElementById('picked-count').textContent = '0';
                    isProcessing = false;
                } else {
                    document.getElementById('message').textContent = '❌ 挑戰失敗，機會已用完！';
                    document.getElementById('message').className = 'message lose';
                    revealAllCards();
                }
            }, 1200);
        }
    }

    function disableAllCards() {
        const grid = document.getElementById('grid');
        Array.from(grid.children).forEach(card => card.classList.add('disabled'));
    }

    function revealAllCards() {
        cards.forEach((_, idx) => revealCard(idx));
    }

    // 初始化遊戲
    initGame();
</script>

</body>
</html>
