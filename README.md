<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ShellGus Soport Game</title>
    <style>
        @keyframes neon {
            0%, 100% {
                text-shadow: 0 0 5px #ffffff, 0 0 10px #ff00ff, 0 0 20px #ff00ff, 0 0 30px #ff00ff, 0 0 40px #ff00ff, 0 0 50px #ff00ff, 0 0 60px #ff00ff;
                color: #ff00ff; /* Cambio de color a dorado brillante */
            }
            50% {
                text-shadow: 0 0 2px #ffffff, 0 0 5px #ff00ff, 0 0 10px #ff00ff, 0 0 15px #ff00ff, 0 0 20px #ff00ff, 0 0 25px #ff00ff, 0 0 30px #ff00ff;
                color: #ff00ff; /* Cambio de color a dorado brillante */
            }
        }

        body {
            margin: 0;
            padding: 0;
            font-family: 'Courier New', Courier, monospace;
            background: linear-gradient(45deg, #000000, #ff00ff);
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            color: #fff;
            text-align: center;
        }

        #welcome {
            font-size: 48px;
            animation: neon 1.5s infinite alternate, focus 3s ease-in-out;
        }

        @keyframes focus {
            0% { opacity: 0.2; }
            50% { opacity: 1; }
            100% { opacity: 0.2; }
        }

        #game-container, #result, #error {
            display: none;
        }

        #phone {
            background: #333;
            border-radius: 20px;
            width: 300px;
            height: 600px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            box-shadow: 0 0 10px #ff00ff;
        }

        .container {
            width: 90%;
            margin: 20px auto;
        }

        .game {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .button {
            font-size: 24px;
            padding: 10px 20px;
            background-color: #ff00ff;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }

        .button:hover {
            background-color: #ffffff;
            color: #ff00ff;
        }

        #hangmanCanvas {
            margin: 20px 0;
            border: 1px solid #fff;
        }

        #wordDisplay, #wrongLettersDisplay {
            font-size: 24px;
            margin: 10px 0;
        }

        footer {
            position: absolute;
            bottom: 10px;
            width: 100%;
            text-align: center;
        }
    </style>
</head>
<body>
    <div id="welcome">
        <p>Welcome to ShellGus</p>
    </div>

    <div id="game-container">
        <div id="phone">
            <div class="container">
                <h1 style="color: #ff00ff;">Shellgus Hangman Game</h1> <!-- Cambio de color a dorado brillante -->
                <div class="game" id="hangman-game">
                    <canvas id="hangmanCanvas" width="300" height="400"></canvas>
                    <div class="word" id="wordDisplay"></div>
                    <div class="wrong-letters" id="wrongLettersDisplay"></div>
                    <button class="button" onclick="startHangman()">Start Game</button>
                </div>
            </div>
        </div>
    </div>

    <div id="result">
        <p>Congratulations! You won a 5% discount!</p>
        <img id="qr-code" src="https://api.qrserver.com/v1/create-qr-code/?size=150x150&data=DISCOUNT5" alt="QR Code">
    </div>

    <div id="error">
        <p>Sorry, you lost! Better luck next time!</p>
    </div>

    <footer>
        Elaborado por ShellGus
    </footer>

    <script>
        const hangmanWords = ['APPLE', 'BANANA', 'CHERRY', 'ORANGE', 'RED', 'BLUE', 'GREEN'];
        let hangmanWord = '';
        let hangmanGuesses = [];
        let hangmanWrongGuesses = [];
        let hangmanCanvas, hangmanContext;

        function startHangman() {
            hangmanCanvas = document.getElementById('hangmanCanvas');
            hangmanContext = hangmanCanvas.getContext('2d');
            hangmanWord = hangmanWords[Math.floor(Math.random() * hangmanWords.length)];
            hangmanGuesses = [];
            hangmanWrongGuesses = [];
            document.getElementById('wordDisplay').textContent = '_ '.repeat(hangmanWord.length);
            document.getElementById('wrongLettersDisplay').textContent = '';
            hangmanContext.clearRect(0, 0, hangmanCanvas.width, hangmanCanvas.height);
            drawHangman();
            window.addEventListener('keydown', handleKeydown);
        }

        function handleKeydown(event) {
            const letter = event.key.toUpperCase();
            if (!letter.match(/[A-Z]/) || hangmanGuesses.includes(letter) || hangmanWrongGuesses.includes(letter)) {
                return;
            }
            if (hangmanWord.includes(letter)) {
                hangmanGuesses.push(letter);
            } else {
                hangmanWrongGuesses.push(letter);
                drawHangman();
            }
            updateDisplay();
            checkGameStatus();
        }

        function updateDisplay() {
            const wordDisplay = document.getElementById('wordDisplay');
            wordDisplay.textContent = hangmanWord.split('').map(letter => (hangmanGuesses.includes(letter) ? letter : '_')).join(' ');
            document.getElementById('wrongLettersDisplay').textContent = hangmanWrongGuesses.join(' ');
        }

        function checkGameStatus() {
            const wordDisplay = document.getElementById('wordDisplay').textContent.replace(/\s/g, '');
            if (wordDisplay === hangmanWord) {
                window.removeEventListener('keydown', handleKeydown);
                document.getElementById('hangman-game').style.display = 'none';
                document.getElementById('result').style.display = 'block';
            } else if (hangmanWrongGuesses.length >= 6) {
                window.removeEventListener('keydown', handleKeydown);
                document.getElementById('hangman-game').style.display = 'none';
                document.getElementById('error').style.display = 'block';
            }
        }

        function drawHangman() {
            const stages = [
                () => drawLine(60, 140, 140, 140),
                () => drawLine(100, 140, 100, 60),
                () => drawLine(100, 60, 140, 60),
                () => drawLine(140, 60, 140, 80),
                () => drawCircle(140, 90, 10),
                () => drawLine(140, 100, 140, 120),
                () => drawLine(140, 105, 130, 115),
                () => drawLine(140, 105, 150, 115),
                () => drawLine(140, 120, 130, 130),
                () => drawLine(140, 120, 150, 130)
            ];
            hangmanContext.clearRect(0, 0, hangmanCanvas.width, hangmanCanvas.height);
            stages.slice(0, hangmanWrongGuesses.length).forEach(stage => stage());
        }

        function drawLine(x1, y1, x2, y2) {
            hangmanContext.beginPath();
            hangmanContext.moveTo(x1, y1);
            hangmanContext.lineTo(x2, y2);
            hangmanContext.strokeStyle = '#FF00FF';
            hangmanContext.lineWidth = 2;
            hangmanContext.stroke();
        }

        function drawCircle(x, y, radius) {
            hangmanContext.beginPath();
            hangmanContext.arc(x, y, radius, 0, 2 * Math.PI);
            hangmanContext.strokeStyle = '#FF00FF';
            hangmanContext.lineWidth = 2;
            hangmanContext.stroke();
        }

        setTimeout(() => {
            document.getElementById('welcome').style.display = 'none';
            document.getElementById('game-container').style.display = 'flex';
        }, 3000);
    </script>
</body>
</html>
