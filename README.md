<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>GeoHeads: IGCSE Edition</title>
    <style>
        :root {
            --primary: #2e7d32; /* Forest Green */
            --secondary: #0288d1; /* Ocean Blue */
            --correct: #4caf50;
            --pass: #f44336;
            --dark: #333;
            --light: #f9f9f9;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            background: var(--light);
            color: var(--dark);
            text-align: center;
            overflow: hidden; /* Prevent scrolling during game */
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
        }

        /* Screens */
        .screen { display: none; padding: 20px; flex-grow: 1; flex-direction: column; justify-content: center; }
        .active { display: flex; }

        h1 { color: var(--primary); font-size: 2.5rem; margin-bottom: 0.5rem; }
        p { font-size: 1.2rem; color: #666; }

        .btn {
            background: var(--primary);
            color: white;
            border: none;
            padding: 15px 30px;
            font-size: 1.2rem;
            border-radius: 50px;
            margin: 10px;
            cursor: pointer;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        /* Game Board */
        #game-screen { background: var(--secondary); color: white; transition: background 0.2s; }
        #word-display { font-size: 5rem; font-weight: bold; text-transform: uppercase; word-wrap: break-word; }
        #timer { font-size: 2rem; position: absolute; top: 20px; right: 20px; opacity: 0.7; }
        
        /* Feedback Colors */
        .flash-correct { background-color: var(--correct) !important; }
        .flash-pass { background-color: var(--pass) !important; }

        /* Summary List */
        .summary-list { text-align: left; max-height: 300px; overflow-y: auto; margin: 20px auto; width: 80%; }
        .score { font-size: 3rem; color: var(--primary); font-weight: bold; }
    </style>
</head>
<body>

    <div id="start-screen" class="screen active">
        <h1>🌍 GeoHeads</h1>
        <p>IGCSE Edexcel Geography Revision</p>
        <button class="btn" onclick="requestPermission()">Start Revision Session</button>
        <p style="font-size: 0.8rem;">Note: Requires Motion/Orientation access</p>
    </div>

    <div id="countdown-screen" class="screen">
        <h1 id="count-text" style="font-size: 8rem;">3</h1>
        <p>Place phone on your forehead!</p>
    </div>

    <div id="game-screen" class="screen">
        <div id="timer">60</div>
        <div id="word-display">READY?</div>
    </div>

    <div id="results-screen" class="screen">
        <h1>Time's Up!</h1>
        <div class="score" id="final-score">0</div>
        <div class="summary-list" id="summary"></div>
        <button class="btn" onclick="location.reload()">Play Again</button>
    </div>

    <script>
        const decks = {
            coasts: ["Longshore Drift", "Hydraulic Action", "Spit", "Groyne", "Stack", "Abrasion", "Managed Retreat", "Swash", "Backwash", "Atoll"],
            hazards: ["Pyroclastic Flow", "Epicentre", "Lahar", "Saffir-Simpson", "Convection", "Subduction", "Eye Wall", "Mercalli Scale"]
        };

        let currentDeck = [...decks.coasts, ...decks.hazards]; // Combined for now
        let currentWordIndex = 0;
        let score = 0;
        let timeLeft = 60;
        let gameActive = false;
        let canTilt = true;
        let results = [];

        // 1. Request Permission (iOS Requirement)
        function requestPermission() {
            if (typeof DeviceOrientationEvent.requestPermission === 'function') {
                DeviceOrientationEvent.requestPermission()
                    .then(response => {
                        if (response == 'granted') startCountdown();
                    })
                    .catch(console.error);
            } else {
                startCountdown(); // Android/Desktop
            }
        }

        // 2. Countdown Logic
        function startCountdown() {
            showScreen('countdown-screen');
            let count = 3;
            const interval = setInterval(() => {
                count--;
                document.getElementById('count-text').innerText = count;
                if (count === 0) {
                    clearInterval(interval);
                    startGame();
                }
            }, 1000);
        }

        // 3. Game Loop
        function startGame() {
            showScreen('game-screen');
            gameActive = true;
            shuffle(currentDeck);
            nextWord();
            
            const timerInterval = setInterval(() => {
                timeLeft--;
                document.getElementById('timer').innerText = timeLeft;
                if (timeLeft <= 0) {
                    clearInterval(timerInterval);
                    endGame();
                }
            }, 1000);

            window.addEventListener('deviceorientation', handleOrientation);
        }

        function nextWord() {
            if (currentWordIndex < currentDeck.length) {
                document.getElementById('word-display').innerText = currentDeck[currentWordIndex];
            } else {
                endGame();
            }
        }

        // 4. Tilt Detection
        function handleOrientation(event) {
            if (!gameActive || !canTilt) return;

            const tilt = event.beta; // Tilt front-to-back in degrees

            // Tilt Down (Correct) - Phone face down
            if (tilt < 45) { 
                recordResult(true);
            } 
            // Tilt Up (Pass) - Phone face up
            else if (tilt > 135) {
                recordResult(false);
            }
        }

        function recordResult(isCorrect) {
            canTilt = false; // Debounce
            const screen = document.getElementById('game-screen');
            
            if (isCorrect) {
                score++;
                screen.classList.add('flash-correct');
                results.push({word: currentDeck[currentWordIndex], status: '✅'});
            } else {
                screen.classList.add('flash-pass');
                results.push({word: currentDeck[currentWordIndex], status: '❌'});
            }

            setTimeout(() => {
                screen.classList.remove('flash-correct', 'flash-pass');
                currentWordIndex++;
                nextWord();
                canTilt = true;
            }, 800);
        }

        function endGame() {
            gameActive = false;
            showScreen('results-screen');
            document.getElementById('final-score').innerText = `Score: ${score}`;
            const summaryHTML = results.map(r => `<div>${r.status} ${r.word}</div>`).join('');
            document.getElementById('summary').innerHTML = summaryHTML;
        }

        // Helpers
        function showScreen(id) {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            document.getElementById(id).classList.add('active');
        }

        function shuffle(array) {
            array.sort(() => Math.random() - 0.5);
        }
    </script>
</body>
</html>
