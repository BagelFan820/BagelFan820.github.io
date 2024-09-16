<html>
<head>
    <title>Click the Dot FAST!</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        /* Reset and basic styling */
        body {
            margin: 0;
            overflow: hidden;
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            position: relative;
            height: 100vh;
        }

        /* Game container styling */
        #gameContainer {
            position: relative;
            width: 100%;
            max-width: 500px;
            height: 60vh; /* 60% of the viewport height */
            margin: 0 auto;
            background-color: #ffffff;
            overflow: hidden;
            border: 2px solid #000;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        /* Title styling */
        #title {
            text-align: center;
            font-size: 2em;
            margin: 20px 0;
            color: #ff6600;
        }

        /* Start button styling */
        #startButton {
            position: absolute;
            z-index: 4; /* Higher z-index to be above the message */
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

        /* Best time display styling */
        #bestTime {
            position: absolute;
            top: 116px; /* Between the title and game container */
            left: 15px; /* Align to the left */
            z-index: 3;
            font-size: 1em;
            color: #333;
        }

        /* Message display styling */
        #message {
            position: absolute;
            top: 30%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 3;
            font-size: 1.5em;
            color: #ff6600;
            text-align: center;
            padding: 10px 20px;
            word-wrap: break-word;
            background-color: rgba(255, 255, 255, 0.9);
            border-radius: 10px;
            display: none; /* Hidden by default */
            pointer-events: none; /* Allow clicks to pass through */
        }

        /* Canvas styling */
        #gameCanvas {
            position: absolute;
            top: 0;
            left: 0;
            z-index: 2;
            cursor: pointer;
        }

        /* Leaderboard container styling */
        #leaderboardContainer {
            position: absolute;
            bottom: 20px; /* Position at the bottom of the page */
            width: 100%;
            max-width: 500px;
            margin: 0 auto;
            padding: 10px;
            background-color: #ffffff;
            border: 2px solid #000;
            border-radius: 10px;
            z-index: 3;
        }
        #leaderboardContainer h2 {
            text-align: center;
            color: #ff6600;
            margin-bottom: 10px;
        }
        #leaderboardList {
            list-style: decimal inside;
            padding-left: 0;
            margin: 0;
        }
        #leaderboardList li {
            margin-bottom: 5px;
            font-size: 1em;
            color: #333;
        }
    </style>
    <!-- Firebase SDKs -->
    <script type="module">
      // Import the functions you need from the SDKs you need
      import { initializeApp } from "https://www.gstatic.com/firebasejs/10.13.1/firebase-app.js";
      import { getAnalytics } from "https://www.gstatic.com/firebasejs/10.13.1/firebase-analytics.js";
      import { getFirestore, collection, addDoc, query, orderBy, limit, getDocs } from "https://www.gstatic.com/firebasejs/10.13.1/firebase-firestore.js";

      // Your web app's Firebase configuration
      const firebaseConfig = {
        apiKey: "AIzaSyBbqBN4156KJNwTiEzPQqDfBFmdtkfL1Z8",
        authDomain: "clickthedotfast-84528.firebaseapp.com",
        projectId: "clickthedotfast-84528",
        storageBucket: "clickthedotfast-84528.appspot.com",
        messagingSenderId: "603486229797",
        appId: "1:603486229797:web:c6e3092250cc9df0f3b6cc",
        measurementId: "G-JVPWNZ4LB4"
      };

      // Initialize Firebase
      const app = initializeApp(firebaseConfig);
      const analytics = getAnalytics(app);

      // Initialize Firestore
      const db = getFirestore(app);
    </script>
</head>
<body>
    <div id="title">Click the Dot FAST!</div>
    <div id="bestTime">BEST TIME: N/A</div>
    <div id="gameContainer">
        <button id="startButton">Start</button>
        <div id="message"></div>
        <canvas id="gameCanvas"></canvas>
    </div>

    <!-- Leaderboard Section -->
    <div id="leaderboardContainer">
        <h2>Leaderboard</h2>
        <ol id="leaderboardList">
            <!-- Scores will be dynamically inserted here -->
        </ol>
    </div>

    <script>
        // Get references to HTML elements
        const gameContainer = document.getElementById('gameContainer');
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startButton = document.getElementById('startButton');
        const messageDiv = document.getElementById('message');
        const bestTimeDiv = document.getElementById('bestTime');
        const leaderboardList = document.getElementById('leaderboardList');

        // Set canvas size to match gameContainer
        function setCanvasSize() {
            canvas.width = gameContainer.clientWidth;
            canvas.height = gameContainer.clientHeight;
        }
        setCanvasSize();

        // Handle window resizing
        window.addEventListener('resize', setCanvasSize);

        // Game variables
        const dotRadius = 40; // Size of the dot
        let dotX, dotY;
        let dotVisible = false;
        let dotAppearanceTime = 0;
        let gameState = 'idle'; // 'idle', 'countdown', 'waiting', 'dotVisible', 'exploding'
        let bestTime = null;
        let dotTimeout; // Timeout for dot appearance
        let particles = []; // Array to hold explosion particles

        // Initialize bestTime from localStorage if available
        if (localStorage.getItem('bestTime')) {
            bestTime = parseFloat(localStorage.getItem('bestTime'));
            bestTimeDiv.textContent = 'BEST TIME: ' + bestTime.toFixed(3) + ' seconds';
        }

        // Function to draw the dot and explosion
        function draw() {
            // Clear the canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw the dot if visible
            if (dotVisible) {
                ctx.fillStyle = 'black';
                ctx.beginPath();
                ctx.arc(dotX, dotY, dotRadius, 0, Math.PI * 2);
                ctx.fill();
            }

            // Draw explosion particles if any
            if (particles.length > 0) {
                particles.forEach((particle, index) => {
                    ctx.fillStyle = `rgba(${particle.color.r}, ${particle.color.g}, ${particle.color.b}, ${particle.alpha})`;
                    ctx.beginPath();
                    ctx.arc(particle.x, particle.y, particle.size, 0, Math.PI * 2);
                    ctx.fill();

                    // Update particle position
                    particle.x += particle.vx;
                    particle.y += particle.vy;

                    // Fade out the particle
                    particle.alpha -= particle.fade;
                    particle.size *= 0.96; // Reduce size over time

                    // Remove particle if alpha or size is too low
                    if (particle.alpha <= 0 || particle.size <= 0.5) {
                        particles.splice(index, 1);
                    }
                });
            }

            requestAnimationFrame(draw);
        }

        // Start the drawing loop
        draw();

        // Function to handle countdown
        function startCountdown() {
            const countdownNumbers = ['3', '2', '1'];
            let currentCount = 0;

            gameState = 'countdown';
            messageDiv.style.display = 'block';
            messageDiv.style.backgroundColor = 'rgba(255, 255, 255, 0.9)';
            messageDiv.style.color = '#ff6600';
            messageDiv.textContent = countdownNumbers[currentCount];
            currentCount++;

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

            setTimeout(showCountdown, 500); // Start countdown after first display
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
                messageDiv.style.backgroundColor = 'rgba(255, 255, 255, 0.9)';
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

        // Function to create explosion particles
        function createExplosion(x, y) {
            const particleCount = 30; // Number of particles
            for (let i = 0; i < particleCount; i++) {
                const angle = Math.random() * Math.PI * 2;
                const speed = Math.random() * 4 + 2;
                const vx = Math.cos(angle) * speed;
                const vy = Math.sin(angle) * speed;
                const size = Math.random() * 3 + 2;
                const color = getRandomColor();
                const alpha = 1;
                const fade = 0.02;

                particles.push({
                    x: x,
                    y: y,
                    vx: vx,
                    vy: vy,
                    size: size,
                    color: color,
                    alpha: alpha,
                    fade: fade
                });
            }
        }

        // Function to generate a random RGB color
        function getRandomColor() {
            const r = Math.floor(Math.random() * 256);
            const g = Math.floor(Math.random() * 256);
            const b = Math.floor(Math.random() * 256);
            return { r, g, b };
        }

        // Function to save a score to Firestore
        async function saveScore(playerName, reactionTime) {
            try {
                await addDoc(collection(db, "leaderboard"), {
                    name: playerName,
                    time: reactionTime,
                    timestamp: new Date()
                });
                console.log("Score saved successfully!");
            } catch (e) {
                console.error("Error saving score: ", e);
            }
        }

        // Function to retrieve and display top scores
        async function getTopScores(limitNumber = 10) {
            try {
                const q = query(collection(db, "leaderboard"), orderBy("time", "asc"), limit(limitNumber));
                const querySnapshot = await getDocs(q);
                const scores = [];
                querySnapshot.forEach((doc) => {
                    scores.push(doc.data());
                });
                displayLeaderboard(scores);
            } catch (e) {
                console.error("Error fetching scores: ", e);
            }
        }

        // Function to display the leaderboard
        function displayLeaderboard(scores) {
            leaderboardList.innerHTML = ""; // Clear existing scores

            scores.forEach((score, index) => {
                const listItem = document.createElement('li');
                listItem.textContent = `${score.name}: ${score.time.toFixed(3)} seconds`;
                leaderboardList.appendChild(listItem);
            });
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

                    // Create explosion animation
                    createExplosion(dotX, dotY);

                    // Prompt for player's name
                    let playerName = prompt("Congratulations! Enter your name for the leaderboard:", "Player");
                    if (playerName === null || playerName.trim() === "") {
                        playerName = "Anonymous";
                    } else {
                        playerName = sanitizeInput(playerName.trim());
                    }

                    // Save the score to Firestore
                    saveScore(playerName, reactionTimeSec);

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

                    // Refresh the leaderboard
                    getTopScores();
                }
            }
        }

        // Function to sanitize player input
        function sanitizeInput(input) {
            const div = document.createElement('div');
            div.textContent = input;
            return div.innerHTML;
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

        // Fetch and display the leaderboard on page load
        window.onload = function() {
            getTopScores();
        };
    </script>
</body>
</html>
