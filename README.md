<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>GeoHeads: Instant Flip</title>
    <style>
        :root { --primary: #1b5e20; --secondary: #01579b; --correct: #2e7d32; --pass: #c62828; --light: #f1f8e9; }
        body { font-family: 'Arial Black', sans-serif; margin: 0; background: var(--light); text-align: center; height: 100vh; display: flex; flex-direction: column; overflow: hidden; }
        .screen { display: none; padding: 20px; flex-grow: 1; flex-direction: column; justify-content: center; }
        .active { display: flex; }
        #word-display { font-size: 5rem; font-weight: bold; text-transform: uppercase; line-height: 1.1; }
        #timer { font-size: 3rem; position: absolute; top: 10px; right: 20px; font-weight: bold; color: #ffeb3b; text-shadow: 2px 2px 4px rgba(0,0,0,0.5); }
        .btn { background: var(--primary); color: white; border: none; padding: 25px 50px; font-size: 2rem; border-radius: 15px; cursor: pointer; }
        #game-screen { background: var(--secondary); color: white; transition: background 0.2s; }
        .flash-correct { background-color: var(--correct) !important; }
        .flash-pass { background-color: var(--pass) !important; }
    </style>
</head>
<body>

    <div id="start-screen" class="screen active">
        <h1>🌍 COASTS 30s</h1>
        <p>Tilt DOWN for Correct<br>Tilt UP to Pass</p>
        <button class="btn" onclick="requestPermission()">START</button>
    </div>

    <div id="game-screen" class="screen">
        <div id="timer">30</div>
        <div id="word-display">READY?</div>
    </div>

    <div id="results-screen" class="screen">
        <h1>Done!</h1>
        <div id="final-score" style="font-size: 4rem; margin: 20px; color: var(--primary);"></div>
        <button class="btn" onclick="location.reload()">PLAY AGAIN</button>
    </div>

    <script>
        const coastalDeck = [
            "Abrasion", "Attrition", "Hydraulic Action", "Solution", "Longshore Drift", 
            "Swash", "Backwash", "Fetch", "Constructive Wave", "Destructive Wave",
            "Concordant Coast", "Discordant Coast", "Headland", "Bay", "Wave-cut Platform",
            "Cave", "Arch", "Stack", "Stump", "Spit", "Bar", "Tombolo", "Sand Dune",
            "Hard Engineering", "Soft Engineering", "Sea Wall", "Groyne", "Rock Armour",
            "Gabion", "Beach Nourishment", "Managed Retreat", "Cliff Regrading"
        ];

        let score = 0, timeLeft = 30, gameActive = false, canAction = true;
        let currentWordIndex = 0;

        function requestPermission() {
            if (typeof DeviceOrientationEvent.requestPermission === 'function') {
                DeviceOrientationEvent.requestPermission().then(res => { if(res === 'granted') startGame(); });
            } else { startGame(); }
        }

        function startGame() {
            document.getElementById('start-screen').classList.remove('active');
            document.getElementById('game-screen').classList.add('active');
            gameActive = true;
            coastalDeck.sort(() => Math.random() - 0.5);
            
            const gameClock = setInterval(() => {
                timeLeft--;
                document.getElementById('timer').innerText = timeLeft;
                if(timeLeft <= 0) { clearInterval(gameClock); endGame(); }
            }, 1000);

            window.addEventListener('deviceorientation', handleOrientation);
            showNextWord();
        }

        function showNextWord() {
            if (currentWordIndex >= coastalDeck.length) { endGame(); return; }
            document.getElementById('word-display').innerText = coastalDeck[currentWordIndex];
        }

        function handleOrientation(event) {
            if (!gameActive || !canAction) return;

            const tilt = event.beta; // 90 is vertical (on forehead)

            if (tilt < 45) { // Tilted Down
                recordResult(true);
            } else if (tilt > 135) { // Tilted Up
                recordResult(false);
            }
        }

        function recordResult(isCorrect) {
            canAction = false; // Lock sensor immediately
            const screen = document.getElementById('game-screen');
            
            if (isCorrect) {
                score++;
                screen.classList.add('flash-correct');
            } else {
                screen.classList.add('flash-pass');
            }

            // Wait for visual flash, then prep next word
            setTimeout(() => {
                screen.classList.remove('flash-correct', 'flash-pass');
                currentWordIndex++;
                showNextWord();
                
                // CRITICAL: Only unlock the sensor once the phone is vertical again (Neutral Zone)
                const checkNeutral = (e) => {
                    if (e.beta > 75 && e.beta < 105) {
                        canAction = true;
                        window.removeEventListener('deviceorientation', checkNeutral);
                    }
                };
                window.addEventListener('deviceorientation', checkNeutral);
            }, 600);
        }

        function endGame() {
            gameActive = false;
            document.getElementById('game-screen').classList.remove('active');
            document.getElementById('results-screen').classList.add('active');
            document.getElementById('final-score').innerText = `Words: ${score}`;
            window.removeEventListener('deviceorientation', handleOrientation);
        }
    </script>
</body>
</html>
