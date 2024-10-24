<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Online Casino Game</title>
    <style>
        body {
            font-family: 'Montserrat', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background: linear-gradient(135deg, #1e5799, #2989d8, #207cca, #7db9e8);
            background-size: 400% 400%;
            animation: gradientBG 15s ease infinite;
            overflow: hidden;
        }
        @keyframes gradientBG {
            0% {background-position: 0% 50%;}
            50% {background-position: 100% 50%;}
            100% {background-position: 0% 50%;}
        }
        .game-container {
            text-align: center;
            background-color: rgba(255, 255, 255, 0.9);
            padding: 40px;
            border-radius: 30px;
            box-shadow: 0 0 30px rgba(0,0,0,0.3);
            transition: all 0.5s ease;
            position: relative;
            overflow: hidden;
        }
        .game-container::before {
            content: '';
            position: absolute;
            top: -50%;
            left: -50%;
            width: 200%;
            height: 200%;
            background: linear-gradient(
                to bottom right,
                rgba(255, 255, 255, 0.3) 0%,
                rgba(255, 255, 255, 0.1) 50%,
                transparent 100%
            );
            transform: rotate(45deg);
            z-index: -1;
        }
        .game-container:hover {
            transform: translateY(-10px) scale(1.02);
            box-shadow: 0 15px 35px rgba(0,0,0,0.4);
        }
        button {
            font-size: 20px;
            font-weight: bold;
            padding: 15px 30px;
            margin: 20px;
            cursor: pointer;
            background: linear-gradient(45deg, #4CAF50, #45a049);
            color: white;
            border: none;
            border-radius: 50px;
            transition: all 0.3s ease;
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
        }
        button:hover {
            background: linear-gradient(45deg, #45a049, #4CAF50);
            transform: translateY(-3px) scale(1.05);
            box-shadow: 0 8px 20px rgba(0,0,0,0.3);
        }
        button:active {
            transform: translateY(1px);
        }
        #result {
            font-size: 32px;
            margin-top: 30px;
            font-weight: bold;
            color: #2c3e50;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
            transition: all 0.3s ease;
        }
        #slot-machine {
            display: flex;
            justify-content: center;
            margin: 30px 0;
            perspective: 1000px;
        }
        .reel {
            font-size: 60px;
            margin: 0 15px;
            padding: 20px;
            background: linear-gradient(135deg, #f1c40f, #f39c12);
            border-radius: 15px;
            box-shadow: inset 0 0 20px rgba(0,0,0,0.3), 0 10px 20px rgba(0,0,0,0.2);
            transition: all 0.5s ease;
            transform-style: preserve-3d;
        }
        .reel:hover {
            transform: rotateY(10deg);
        }
        @keyframes spin {
            0% { transform: rotateX(0deg); }
            100% { transform: rotateX(360deg); }
        }
        .spinning {
            animation: spin 0.5s linear infinite;
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1>Lucky Slots Extravaganza</h1>
        <p>Your balance: $<span id="balance">10000</span></p>
        <div id="slot-machine">
            <div class="reel" id="reel1"></div>
            <div class="reel" id="reel2"></div>
            <div class="reel" id="reel3"></div>
        </div>
        <button id="spinButton">Spin ($10)</button>
        <div id="result"></div>
        <div id="jackpot">Mega Jackpot: $<span id="jackpotAmount">10000</span></div>
        <button id="cashOutButton">Cash Out</button>
        <div id="bonusWheel"></div>
        <button id="bonusButton" style="display: none;">Spin Bonus Wheel</button>
        
        <!-- Color Game Addition -->
        <div id="colorGame" style="margin-top: 30px; display: none;">
            <h2>Color Bonus Game</h2>
            <p>Guess the correct color to win extra credits!</p>
            <div id="colorOptions" style="display: flex; justify-content: space-around; margin-top: 20px;">
                <button class="colorButton" data-color="red" style="background-color: red; width: 50px; height: 50px;"></button>
                <button class="colorButton" data-color="blue" style="background-color: blue; width: 50px; height: 50px;"></button>
                <button class="colorButton" data-color="green" style="background-color: green; width: 50px; height: 50px;"></button>
                <button class="colorButton" data-color="yellow" style="background-color: yellow; width: 50px; height: 50px;"></button>
            </div>
            <p id="colorGameResult" style="margin-top: 20px;"></p>
        </div>
    </div>

    <script>
        let balance = 10000;
        let jackpot = 10000;
        const spinButton = document.getElementById('spinButton');
        const cashOutButton = document.getElementById('cashOutButton');
        const balanceDisplay = document.getElementById('balance');
        const jackpotDisplay = document.getElementById('jackpotAmount');
        const resultDisplay = document.getElementById('result');
        const bonusButton = document.getElementById('bonusButton');
        const bonusWheel = document.getElementById('bonusWheel');
        const colorGame = document.getElementById('colorGame');
        const reels = [
            document.getElementById('reel1'),
            document.getElementById('reel2'),
            document.getElementById('reel3')
        ];

        const symbols = ['🍒', '🍊', '🍋', '🍇', '🔔', '💎', '🎰', '🌟'];
        const symbolProbabilities = [0.2, 0.2, 0.15, 0.15, 0.1, 0.05, 0.05, 0.1]; // Probabilities for each symbol

        function updateDisplay() {
            balanceDisplay.textContent = balance;
            jackpotDisplay.textContent = jackpot;
            spinButton.disabled = balance < 10;
            cashOutButton.disabled = balance <= 0;
        }

        function getRandomSymbol() {
            const randomValue = Math.random();
            let cumulativeProbability = 0;
            for (let i = 0; i < symbols.length; i++) {
                cumulativeProbability += symbolProbabilities[i];
                if (randomValue < cumulativeProbability) {
                    return symbols[i];
                }
            }
            return symbols[symbols.length - 1];
        }

        function spin() {
            if (balance >= 10) {
                balance -= 10;
                jackpot += 5; // Only half of the bet goes to the jackpot
                updateDisplay();

                let spinning = true;
                let spins = 0;
                const maxSpins = 30;
                const spinDuration = 1500; // Total spin duration in milliseconds
                const spinInterval = spinDuration / maxSpins;

                reels.forEach(reel => reel.classList.add('spinning'));

                const spinAnimation = setInterval(() => {
                    if (spins >= maxSpins) {
                        clearInterval(spinAnimation);
                        spinning = false;
                        reels.forEach(reel => reel.classList.remove('spinning'));
                        checkWin();
                    } else {
                        reels.forEach(reel => {
                            reel.textContent = getRandomSymbol();
                        });
                        spins++;
                    }
                }, spinInterval);
            } else {
                resultDisplay.innerHTML = '<span style="color: #e74c3c;">Insufficient funds. Please add more credits.</span>';
            }
        }

        function checkWin() {
            const result = reels.map(reel => reel.textContent);

            if (result[0] === result[1] && result[1] === result[2]) {
                let winAmount;
                if (result[0] === '💎') {
                    winAmount = jackpot;
                    jackpot = 10000;
                    resultDisplay.innerHTML = `<span style="color: gold; font-size: 36px;">MEGA JACKPOT! You won $${winAmount}! 🎉🎊</span>`;
                    playWinSound('jackpot');
                } else if (result[0] === '🎰') {
                    winAmount = 500;
                    resultDisplay.innerHTML = `<span style="color: #4CAF50; font-size: 24px;">Big Win! You won $${winAmount}! 🎉</span>`;
                    playWinSound('big');
                } else {
                    winAmount = 100;
                    resultDisplay.innerHTML = `<span style="color: #3498db;">You won $${winAmount}!</span>`;
                    playWinSound('small');
                }
                balance += winAmount;
                activateBonusWheel();
                activateColorGame();
            } else if (result.includes('🌟')) {
                resultDisplay.innerHTML = `<span style="color: #f39c12;">Free Spin! 🌟</span>`;
                playWinSound('free');
                setTimeout(spin, 1000);
            } else {
                resultDisplay.textContent = 'Try again!';
                playLoseSound();
            }

            updateDisplay();
        }

        function activateBonusWheel() {
            bonusButton.style.display = 'inline-block';
            bonusButton.addEventListener('click', spinBonusWheel, { once: true });
        }

        function spinBonusWheel() {
            const bonusMultipliers = [1.5, 2, 2.5, 3];
            const randomMultiplier = bonusMultipliers[Math.floor(Math.random() * bonusMultipliers.length)];
            
            bonusWheel.innerHTML = `<span style="color: #e74c3c; font-size: 24px;">Bonus Multiplier: ${randomMultiplier}x</span>`;
            balance = Math.floor(balance * randomMultiplier);
            updateDisplay();
            playWinSound('bonus');
            
            setTimeout(() => {
                bonusWheel.innerHTML = '';
                bonusButton.style.display = 'none';
            }, 3000);
        }

        function playWinSound(type) {
            // Implement sound effects for different win types
            console.log(`Playing ${type} win sound`);
        }

        function playLoseSound() {
            // Implement sound effect for losing
            console.log('Playing lose sound');
        }

        spinButton.addEventListener('click', spin);

        cashOutButton.addEventListener('click', () => {
            alert(`Congratulations! You cashed out $${balance}. Thanks for playing!`);
            balance = 100;
            jackpot = 10000;
            updateDisplay();
            resultDisplay.textContent = '';
            bonusWheel.innerHTML = '';
            bonusButton.style.display = 'none';
            colorGame.style.display = 'none';
        });

        // Add keyboard controls
        document.addEventListener('keydown', (event) => {
            if (event.code === 'Space') {
                event.preventDefault();
                spin();
            } else if (event.code === 'KeyC') {
                cashOutButton.click();
            }
        });

        // Add responsive design
        function adjustFontSize() {
            const width = window.innerWidth;
            if (width < 600) {
                document.body.style.fontSize = '14px';
            } else if (width < 1024) {
                document.body.style.fontSize = '16px';
            } else {
                document.body.style.fontSize = '18px';
            }
        }

        window.addEventListener('resize', adjustFontSize);
        adjustFontSize();

        // Color Game Addition
        function activateColorGame() {
            colorGame.style.display = 'block';
            const colorButtons = document.querySelectorAll('.colorButton');
            const colorGameResult = document.getElementById('colorGameResult');
            const colors = ['red', 'blue', 'green', 'yellow'];
            const correctColor = colors[Math.floor(Math.random() * colors.length)];

            colorButtons.forEach(button => {
                button.disabled = false;
                button.onclick = function() {
                    const selectedColor = this.getAttribute('data-color');
                    if (selectedColor === correctColor) {
                        const bonusAmount = 50;
                        balance += bonusAmount;
                        colorGameResult.innerHTML = `<span style="color: green;">Correct! You won ${bonusAmount} credits!</span>`;
                        updateDisplay();
                    } else {
                        colorGameResult.innerHTML = `<span style="color: red;">Wrong! The correct color was ${correctColor}.</span>`;
                    }
                    colorButtons.forEach(btn => btn.disabled = true);
                    setTimeout(() => {
                        colorGameResult.innerHTML = '';
                        colorGame.style.display = 'none';
                        colorButtons.forEach(btn => btn.disabled = false);
                    }, 3000);
                };
            });
        }

        updateDisplay();
    </script>
</body>
</html>
