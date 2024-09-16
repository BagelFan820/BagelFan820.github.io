<!DOCTYPE html>
<html>
<head>
    <title>Click the Dot FAST</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            margin: 0;
            overflow: hidden;
            cursor: none; /* Hide the default cursor */
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
        }
        #gameContainer {
            position: relative;
            width: 100%;
            max-width: 500px;
            height: 80vh;
            margin: 0 auto;
            background-color: #ffffff;
            overflow: hidden;
            border: 2px solid #000;
        }
        #title {
            text-align: center;
            font-size: 2em;
            margin: 20px 0;
            color: #ff6600;
        }
        #startButton {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 2;
            padding: 15px 30px;
            font-size: 1.5em;
        }
        #bestTime {
            position: absolute;
            bottom: 10px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 2;
            font-size: 1.2em;
            color: #333;
        }
        #message {
            position: absolute;
            top: 40%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 2;
            font-size: 2em;
            color: #ff6600;
            text-align: center;
        }
        #gameCanvas {
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }
    </style>
</head>
<body>
    <div id="title">Click the Dot FAST</div>
    <div id="gameContainer">
        <button id="startButton">Start</button>
        <div id="message"></div>
        <div id="bestTime"></div>
        <canvas id="gameCanvas"></canvas>
    </div>

    <script>
        // Get references to HTML elements
        var gameContainer = document.getElementById('gameContainer');
        var canvas = document.getElementById('gameCanvas');
        var ctx = canvas.getContext('2d');
        var startButton = document.getElementById('startButton');
        var messageDiv = document.getElementById('message');
        var bestTimeDiv = document.getElementById('bestTime');

        // Set canvas size to match gameContainer
        canvas.width = gameContainer.clientWidth;
        canvas.height = gameContainer.clientHeight;

        // Game variables
        var dotRadius = 20; // Size of the dot
        var dotX, dotY;
        var dotVisible = false;
        var dotAppearanceTime;
        var gameStarted = false;
        var crosshairSize = 20;
        var bestTime = null;
        var showCrosshair = false;

        // Function to draw the crosshair and dot
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

            // Draw the crosshair if it's enabled
            if (showCrosshair) {
                ctx.strokeStyle = 'red';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.moveTo(crosshairX - crosshairSize, crosshairY);
                ctx.lineTo(crosshairX + crosshairSize, crosshairY);
                ctx.moveTo(crosshairX, crosshairY - crosshairSize);
                ctx.lineTo(crosshairX, crosshairY + crosshairSize);
                ctx.stroke();
            }

            requestAnimationFrame(draw);
        }

        var crosshairX = canvas.width / 2;
        var crosshairY = canvas.height / 2;

        // Update the crosshair position
        function updateCrosshairPosition(e) {
            var rect = canvas.getBoundingClientRect();
            if (e.touches && e.touches.length > 0) {
                crosshairX = e.touches[0].clientX - rect.left;
                crosshairY = e.touches[0].clientY - rect.top;
            } else {
                crosshairX = e.clientX - rect.left;
                crosshairY = e.clientY - rect.top;
            }
        }

        // Handle mouse and touch movements
        document.addEventListener('mousemove', function(e) {
            updateCrosshairPosition(e);
        });

        document.addEventListener('touchmove', function(e) {
            updateCrosshairPosition(e);
            e.preventDefault();
        }, { passive: false });

        // Handle click and touch events
        function handleInteraction(e) {
            if (!gameStarted) return; // Ignore interactions before the game starts

            if (dotVisible) {
                // Calculate distance between interaction and dot center
                var rect = canvas.getBoundingClientRect();
                var clickX, clickY;
                if (e.touches && e.touches.length > 0) {
                    clickX = e.touches[0].clientX - rect.left;
                    clickY = e.touches[0].clientY - rect.top;
                } else {
                    clickX = e.clientX - rect.left;
                    clickY = e.clientY - rect.top;
                }

                var distance = Math.hypot(clickX - dotX, clickY - dotY);
                if (distance <= dotRadius) {
                    // User clicked on the dot
                    var reactionTimeMs = performance.now() - dotAppearanceTime;
                    var reactionTimeSec = reactionTimeMs / 1000;
                    messageDiv.textContent = 'Reaction Time:\n' + reactionTimeSec.toFixed(3) + ' seconds';

                    // Update best time
                    if (bestTime === null || reactionTimeSec < bestTime) {
                        bestTime = reactionTimeSec;
                    }

                    dotVisible = false;
                    gameStarted = false;
                    showCrosshair = false;
                    startButton.style.display = 'block';

                    // Display best time
                    bestTimeDiv.textContent = 'BEST TIME: ' + bestTime.toFixed(3) + ' seconds';
                }
            }
        }

        document.addEventListener('click', handleInteraction);
        document.addEventListener('touchstart', handleInteraction);

        // Start button event listener
        startButton.addEventListener('click', function() {
            messageDiv.textContent = '';
            startButton.style.display = 'none';
            bestTimeDiv.textContent = ''; // Hide best time during the attempt
            gameStarted = true;
            showCrosshair = false; // Hide crosshair until the dot appears

            // Random delay between 2 and 10 seconds (2000 to 10000 milliseconds)
            var delay = Math.random() * 8000 + 2000;

            setTimeout(function() {
                // Show the dot at a random position within the canvas
                dotX = Math.random() * (canvas.width - 2 * dotRadius) + dotRadius;
                dotY = Math.random() * (canvas.height - 2 * dotRadius) + dotRadius;

                dotVisible = true;
                dotAppearanceTime = performance.now();
                showCrosshair = true; // Show crosshair when the dot appears
            }, delay);
        });

        // Adjust canvas size when the window is resized
        window.addEventListener('resize', function() {
            canvas.width = gameContainer.clientWidth;
            canvas.height = gameContainer.clientHeight;
            crosshairX = canvas.width / 2;
            crosshairY = canvas.height / 2;
        });

        // Start the drawing loop
        draw();
    </script>
</body>
</html>
