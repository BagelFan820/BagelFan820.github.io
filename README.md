<html>
<head>
    <title>Click the Dot FAST</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            position: relative;
            height: 100vh;
        }
        #gameContainer {
            position: relative;
            width: 100%;
            max-width: 500px;
            height: 60vh; /* Adjusted height to 60% of the viewport */
            margin: 0 auto;
            background-color: #ffffff;
            overflow: hidden;
            border: 2px solid #000;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        #title {
            text-align: center;
            font-size: 2em;
            margin: 20px 0;
            color: #ff6600;
        }
        #startButton {
            position: absolute;
            z-index: 2;
            padding: 15px 30px;
            font-size: 1.5em;
            cursor: pointer;
            background-color: #ff6600;
            border: none;
            border-radius: 5px;
            color: #fff;
            transition: background-color 0.3s;
        }
        #startButton:hover {
            background-color: #e55b00;
        }
        #bestTime {
            position: absolute;
            top: 116px; /* Position it between the title and game container */
            left: 15px; /* Align to the left */
            z-index: 3;
            font-size: 1em;
            color: #333;
        }
        #message {
            position: absolute;
            top: 40%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 3;
            font-size: 1.5em;
            color: #ff6600;
            text-align: center;
            padding: 0 20px;
            word-wrap: break-word;
            background-color: rgba(255, 255, 255, 0.8);
            border-radius: 10px;
            display: none; /* Hidden by default */
        }
        #gameCanvas {
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div id="title">Click the Dot FAST!</div>
    <div id="bestTime">BEST TIME: N/A</div>
    <div id="gameContainer">
        <button id="startButton">Start</button>
        <div id="message"></div>
        <canvas id="gameCanvas"></canvas>
    </div>

    <script>
        // Get references to HTML elements
        const gameContainer = document.getElementById('gameContainer');
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startButton = document.getElementById('startButton');
        const messageDiv = document.getElementById('message');
        const bestTimeDiv = document.getElementById('bestTime');

        // Set canvas size to match gameContainer
        function setCanvasSize() {
            canvas.width = gameContainer.clientWidth;
            canvas.height = gameContainer.clientHeight;
        }
        setCanvasSize();

        // Game variables
        const dotRadius = 40; // Increased size of the dot
        let dotX, dotY;
        let dotVisible = false;
        let dotAppearanceTime = 0;
        let gameState = 'idle'; // Possible states: 'idle', 'countdown', 'waiting', 'dotVisible'
        let bestTime = null;
        let dotTimeout; // Timeout for dot appearance

        // Initialize bestTime from localStorage if available
        if (localStorage.getItem('bestTime')) {
            bestTime = parseFloat(localStorage.getItem('bestTime'));
            bestTimeDiv.textContent = 'BEST TIME: ' + bestTime.toFixed(3) + ' seconds';
        }

        // Function to draw the dot
        function draw() {
            // Clear the canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // If the dot is visible, draw it
            if (dotVisible) {
                ctx.fillStyle = 'black';
                ctx.beginPath();
                ctx.arc(dotX, dotY, dotRadius, 0, Math.PI * 2);
                ctx.fill();
            }

            requestAnimationFrame(draw);
        }

        // Function to handle countdown
        function startCountdown() {
            const countdownNumbers = ['3', '2', '1'];
            let currentCount = 0;

            gameState = 'countdown';
            messageDiv.style.display = 'block';
            messageDiv.style.backgroundColor = 'rgba(255, 255, 255, 0.8)';
            messageDiv.style.color = '#ff6600';

            // Function to display each countdown number
            function showCountdown() {
                if (currentCount < countdownNumbers.length) {
                    messageDiv.textContent = countdownNumbers[currentCount];
                    currentCount++;
                    setTimeout(showCountdown, 500); // Show each number for 0.5 seconds
                } else {
                    messageDiv.textContent = ''; // Clear message after countdown
                    messageDiv.style.display = 'none';
                    startGameDelay(); // Start the delay for dot appearance
                }
            }

            showCountdown(); // Start the countdown
        }

        // Function to handle delay for dot appearance
        function startGameDelay() {
            gameState = 'waiting'; // Set to 'waiting' when waiting for dot appearance

            // Random delay between 2 and 8 seconds (2000 to 8000 milliseconds)
            const delay = Math.random() * 6000 + 2000;

            dotTimeout = setTimeout(() => {
                // Show the dot at a random position within the canvas
                dotX = Math.random() * (canvas.width - 2 * dotRadius) + dotRadius;
                dotY = Math.random() * (canvas.height - 2 * dotRadius) + dotRadius;

                dotVisible = true;
                gameState = 'dotVisible'; // Set state to 'dotVisible' after dot appears
                dotAppearanceTime = performance.now();
            }, delay);
        }

        // Function to reset the game to idle state
        function resetGame(message = '') {
            gameState = 'idle';
            dotVisible = false;
            messageDiv.textContent = message;

            if (message) {
                messageDiv.style.display = 'block';
                messageDiv.style.backgroundColor = 'rgba(255, 255, 255, 0.8)';
                messageDiv.style.color = '#ff6600';
            } else {
                messageDiv.style.display = 'none';
            }

            startButton.style.display = 'block';
            if (dotTimeout) {
                clearTimeout(dotTimeout); // Clear the timeout to stop the dot from appearing
                dotTimeout = null;
            }
        }

        // Handle click and touch events
        function handleInteraction(e) {
            // Prevent default behavior for touch events to avoid unwanted side effects
            if (e.type === 'touchstart') {
                e.preventDefault();
            }

            if (gameState === 'idle') return; // Ignore interactions before the game starts

            if (gameState === 'countdown') {
                // Ignore interactions during countdown
                return;
            }

            if (gameState === 'waiting') {
                // If user taps before dot appears, end the game
                resetGame('Patience is Required');
                return;
            }

            if (gameState === 'dotVisible') {
                // Calculate distance between interaction and dot center
                const rect = canvas.getBoundingClientRect();
                let clickX, clickY;

                if (e.touches && e.touches.length > 0) {
                    clickX = e.touches[0].clientX - rect.left;
                    clickY = e.touches[0].clientY - rect.top;
                } else {
                    clickX = e.clientX - rect.left;
                    clickY = e.clientY - rect.top;
                }

                const distance = Math.hypot(clickX - dotX, clickY - dotY);
                if (distance <= dotRadius) {
                    // User clicked on the dot
                    const reactionTimeMs = performance.now() - dotAppearanceTime;
                    const reactionTimeSec = reactionTimeMs / 1000;

                    // Show the reaction time message
                    resetGame('Reaction Time: ' + reactionTimeSec.toFixed(3) + ' seconds');

                    // Update best time if necessary
                    if (bestTime === null || reactionTimeSec < bestTime) {
                        bestTime = reactionTimeSec;
                        bestTimeDiv.textContent = 'BEST TIME: ' + bestTime.toFixed(3) + ' seconds';
                        bestTimeDiv.style.display = 'block'; // Ensure it is visible

                        // Save best time to localStorage
                        localStorage.setItem('bestTime', bestTime.toFixed(3));
                    }
                }
            }
        }

        // Event listeners for clicks and touches
        document.addEventListener('click', handleInteraction);
        document.addEventListener('touchstart', handleInteraction);

        // Start button event listener
        startButton.addEventListener('click', () => {
            // Reset messages and hide start button
            messageDiv.textContent = '';
            messageDiv.style.display = 'none';
            startButton.style.display = 'none';
            gameState = 'countdown';

            startCountdown(); // Begin countdown before game starts
        });

        // Adjust canvas size when the window is resized
        window.addEventListener('resize', setCanvasSize);

        // Start the drawing loop
        draw();
    </script>
</body>
</html>
