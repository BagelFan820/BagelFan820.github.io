<!DOCTYPE html>
<html>
<head>
    <title>Reaction Time Game</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            cursor: none; /* Hide the default cursor */
        }
        #gameCanvas {
            background-color: #f0f0f0;
        }
        #startButton {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 1;
            padding: 15px 30px;
            font-size: 24px;
        }
        #message {
            position: absolute;
            top: 30%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 1;
            font-size: 48px;
            font-family: 'Comic Sans MS', cursive, sans-serif;
            color: #ff6600;
            text-shadow: 2px 2px #000000;
            text-align: center;
        }
        #topTimes {
            position: absolute;
            top: 10px;
            left: 10px;
            z-index: 1;
            font-size: 20px;
            font-family: Arial, sans-serif;
            background-color: rgba(255, 255, 255, 0.8);
            padding: 10px;
            border-radius: 8px;
        }
    </style>
</head>
<body>
    <button id="startButton">Start</button>
    <div id="message"></div>
    <div id="topTimes"></div>
    <canvas id="gameCanvas"></canvas>

    <script>
        // Get references to HTML elements
        var canvas = document.getElementById('gameCanvas');
        var ctx = canvas.getContext('2d');
        var startButton = document.getElementById('startButton');
        var messageDiv = document.getElementById('message');
        var topTimesDiv = document.getElementById('topTimes');

        // Set canvas size to full window
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        // Game variables
        var dotRadius = 20; // Size of the dot
        var dotX, dotY;
        var dotVisible = false;
        var dotAppearanceTime;
        var gameStarted = false;
        var crosshairSize = 20;
        var reactionTimes = [];
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

                    // Add the reaction time to the array
                    reactionTimes.push(reactionTimeSec);

                    // Sort the array and keep only top 5 times
                    reactionTimes.sort(function(a, b) { return a - b; });
                    if (reactionTimes.length > 5) {
                        reactionTimes = reactionTimes.slice(0, 5);
                    }

                    // Update the topTimes div
                    var topTimesHTML = '<strong>Top 5 Times:</strong><br>';
                    for (var i = 0; i < reactionTimes.length; i++) {
                        topTimesHTML += (i + 1) + '. ' + reactionTimes[i].toFixed(3) + ' seconds<br>';
                    }
                    topTimesDiv.innerHTML = topTimesHTML;

                    dotVisible = false;
                    gameStarted = false;
                    showCrosshair = false;
                    startButton.style.display = 'block';
                }
            }
        }

        document.addEventListener('click', handleInteraction);
        document.addEventListener('touchstart', handleInteraction);

        // Start button event listener
        startButton.addEventListener('click', function() {
            messageDiv.textContent = '';
            startButton.style.display = 'none';
            gameStarted = true;
            showCrosshair = false; // Hide crosshair until the dot appears

            // Random delay between 2 and 10 seconds (2000 to 10000 milliseconds)
            var delay = Math.random() * 8000 + 2000;

            setTimeout(function() {
                // Define the area to avoid (topTimesDiv area)
                var avoidArea = {
                    x: 0,
                    y: 0,
                    width: topTimesDiv.offsetWidth + 20, // Add some padding
                    height: topTimesDiv.offsetHeight + 20
                };

                // Show the dot at a random position outside the avoidArea
                do {
                    dotX = Math.random() * (canvas.width - 2 * dotRadius) + dotRadius;
                    dotY = Math.random() * (canvas.height - 2 * dotRadius) + dotRadius;
                } while (
                    dotX - dotRadius < avoidArea.x + avoidArea.width &&
                    dotY - dotRadius < avoidArea.y + avoidArea.height
                );

                dotVisible = true;
                dotAppearanceTime = performance.now();
                showCrosshair = true; // Show crosshair when the dot appears
            }, delay);
        });

        // Adjust canvas size when the window is resized
        window.addEventListener('resize', function() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        });

        // Start the drawing loop
        draw();
    </script>
</body>
</html>
