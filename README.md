<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Click the Dot FAST!</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- Import Orbitron Font from Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700&display=swap" rel="stylesheet">
    <style>
        /* Reset and basic styling */
        html, body {
            height: 100%;
            margin: 0;
            padding: 0;
            overflow: hidden;
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            position: relative;
        }

        /* Game Title Styling */
        #gameTitle {
            position: absolute;
            top: 10px;
            left: 50%;
            transform: translateX(-50%);
            font-family: 'Orbitron', sans-serif;
            font-size: 2.5em;
            color: #ff6600;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
            text-align: center;
            z-index: 6;
        }

        /* Best Time Positioning */
        #bestTime {
            position: absolute;
            top: 80px; /* Adjusted to accommodate the game title */
            left: 20px;
            font-size: 1.2em;
            color: #ff6600;
            font-weight: bold;
            background-color: rgba(255, 255, 255, 0.8);
            padding: 5px 10px;
            border-radius: 5px;
        }

        /* Header styling */
        #header {
            position: absolute;
            top: 80px; /* Same as #bestTime top */
            right: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 10px;
            z-index: 5;
        }

        /* Toggle Button Styling */
        .toggle-button {
            background-color: #ff6600;
            color: #fff;
            border: none;
            padding: 10px 20px;
            font-size: 1em;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s;
        }

        .toggle-button:hover {
            background-color: #e55b00;
        }

        /* Content Container as Flexbox with Sidebar Leaderboard */
        #contentContainer {
            position: absolute;
            top: 120px; /* Adjusted to accommodate the best time and header */
            left: 50%;
            transform: translateX(-50%);
            width: 90%;
            max-width: 1200px;
            height: 60vh;
            background-color: #ffffff;
            border: 2px solid #000;
            border-radius: 10px;
            overflow: hidden;
            z-index: 3;
            display: flex;
        }

        /* Game Container */
        #gameContainer {
            flex: 3;
            position: relative;
        }

        /* Leaderboard Container as Sidebar */
        #leaderboardContainer {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
            background-color: #fafafa;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            border-left: 2px solid #000;
            border-radius: 0 10px 10px 0;
            display: none;
        }

        /* Show Leaderboard Sidebar */
        #leaderboardContainer.active {
            display: block;
        }

        /* Responsive adjustments */
        @media (max-width: 600px) {
            #contentContainer {
                height: 70vh; /* Taller on smaller screens */
                flex-direction: column;
            }

            #leaderboardContainer {
                width: 100%;
                border-left: none;
                border-top: 2px solid #000;
                border-radius: 10px 10px 0 0;
            }

            #leaderboardContainer.active {
                display: block;
            }

            /* Adjust game title for smaller screens */
            #gameTitle {
                font-size: 2em;
            }
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
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
        }
        #startButton:hover {
            background-color: #e55b00;
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
            width: 100%;
            height: 100%;
            z-index: 2;
            cursor: pointer;
        }

        /* Leaderboard Header */
        #leaderboardContainer h2 {
            text-align: center;
            color: #ff6600;
            margin-bottom: 20px;
            font-size: 2em;
            font-weight: bold;
        }

        /* Leaderboard List Items */
        #leaderboardList {
            list-style: none; /* Remove default numbering */
            padding-left: 0;
            margin: 0;
        }

        #leaderboardList li {
            background-color: #fff;
            margin-bottom: 15px;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
            display: flex;
            justify-content: space-between;
            align-items: center;
            transition: transform 0.2s, background-color 0.2s;
        }

        #leaderboardList li:hover {
            background-color: #f9f9f9;
            transform: translateY(-3px);
        }

        /* Leaderboard Rank Styling */
        #leaderboardList li .rank {
            font-size: 1.2em;
            font-weight: bold;
            color: #ff6600;
            width: 30px;
        }

        /* Leaderboard Name Styling */
        #leaderboardList li .name {
            font-size: 1em;
            color: #333;
            flex: 1;
            padding-left: 10px;
        }

        /* Leaderboard Time Styling */
        #leaderboardList li .time {
            font-size: 1em;
            color: #555;
            white-space: nowrap;
        }
    </style>
</head>
<body>
    <!-- Game Title -->
    <div id="gameTitle">Click the Dot FAST!</div>

    <!-- Best Time Display -->
    <div id="bestTime">BEST TIME: N/A</div>

    <!-- Header with Leaderboard Toggle -->
    <div id="header">
        <button id="toggleLeaderboardButton" class="toggle-button">Leaderboard</button>
    </div>

    <!-- Content Container -->
    <div id="contentContainer">
        <!-- Game Container -->
        <div id="gameContainer" class="active">
            <button id="startButton">Start</button>
            <div id="message"></div>
            <canvas id="gameCanvas"></canvas>
        </div>
        <!-- Leaderboard Container -->
        <div id="leaderboardContainer">
            <h2>Leaderboard</h2>
            <ol id="leaderboardList">
                <!-- Scores will be dynamically inserted here -->
            </ol>
        </div>
    </div>

    <!-- Firebase and Game Script -->
    <script type="module">
        // Import Firebase SDK functions
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.13.1/firebase-app.js";
        import { getAnalytics } from "https://www.gstatic.com/firebasejs/10.13.1/firebase-analytics.js";
        import { getFirestore, collection, addDoc, query, orderBy, limit, getDocs } from "https://www.gstatic.com/firebasejs/10.13.1/firebase-firestore.js";

        // Your web app's Firebase configuration
        const firebaseConfig = {
            apiKey: "YOUR_API_KEY",
            authDomain: "YOUR_AUTH_DOMAIN",
            projectId: "YOUR_PROJECT_ID",
            storageBucket: "YOUR_STORAGE_BUCKET",
            messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
            appId: "YOUR_APP_ID",
            measurementId: "YOUR_MEASUREMENT_ID"
        };

        // Initialize Firebase
        const app = initializeApp(firebaseConfig);
        const analytics = getAnalytics(app);

        // Initialize Firestore
        const db = getFirestore(app);

        // Get references to HTML elements
        const gameContainer = document.getElementById('gameContainer');
        const leaderboardContainer = document.getElementById('leaderboardContainer');
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startButton = document.getElementById('startButton');
        const messageDiv = document.getElementById('message');
        const bestTimeDiv = document.getElementById('bestTime');
        const leaderboardList = document.getElementById('leaderboardList');
        const toggleLeaderboardButton = document.getElementById('toggleLeaderboardButton');
        const gameTitle = document.getElementById('gameTitle');

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
        let isPromptActive = false; // Prevent multiple prompts

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

                // Create rank element
                const rank = document.createElement('span');
                rank.classList.add('rank');
                rank.textContent = `#${index + 1}`;

                // Create name element
                const name = document.createElement('span');
                name.classList.add('name');
                name.textContent = score.name;

                // Create time element
                const time = document.createElement('span');
                time.classList.add('time');
                time.textContent = `${score.time.toFixed(3)} sec`;

                // Append elements to listItem
                listItem.appendChild(rank);
                listItem.appendChild(name);
                listItem.appendChild(time);

                // Append listItem to leaderboardList
                leaderboardList.appendChild(listItem);
            });
        }

        // Handle click and touch events using pointerdown
        async function handleInteraction(e) {
            // Prevent default behavior for touch events to avoid unwanted side effects
            if (e.pointerType === 'touch') {
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

                if (e.pointerType === 'touch') {
                    clickX = e.clientX - rect.left;
                    clickY = e.clientY - rect.top;
                } else {
                    clickX = e.clientX - rect.left;
                    clickY = e.clientY - rect.top;
                }

                const distance = Math.hypot(clickX - dotX, clickY - dotY);
                if (distance <= dotRadius && !isPromptActive) {
                    isPromptActive = true; // Set the flag

                    // User clicked on the dot
                    const reactionTimeMs = performance.now() - dotAppearanceTime;
                    const reactionTimeSec = reactionTimeMs / 1000;

                    // Create explosion animation
                    createExplosion(dotX, dotY);

                    // Fetch current top 10 scores to determine if this score qualifies
                    try {
                        const q = query(collection(db, "leaderboard"), orderBy("time", "asc"), limit(10));
                        const querySnapshot = await getDocs(q);
                        let qualifies = false;

                        if (querySnapshot.empty) {
                            // No scores yet, so it qualifies
                            qualifies = true;
                        } else if (querySnapshot.size < 10) {
                            // Less than 10 scores, it qualifies
                            qualifies = true;
                        } else {
                            // Get the worst (10th) score
                            let worstScore = 0;
                            querySnapshot.forEach((doc) => {
                                worstScore = doc.data().time;
                            });
                            if (reactionTimeSec < worstScore) {
                                qualifies = true;
                            }
                        }

                        if (qualifies) {
                            // Prompt for player's name
                            let playerName = prompt("Congratulations! Enter your name for the leaderboard:", "Player");
                            if (playerName === null || playerName.trim() === "") {
                                playerName = "Anonymous";
                            } else {
                                playerName = sanitizeInput(playerName.trim());
                            }

                            // Save the score to Firestore
                            await saveScore(playerName, reactionTimeSec);
                        }
                    } catch (error) {
                        console.error("Error during qualification check or saving score:", error);
                    }

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

                    // Reset the flag after a short delay to allow new interactions
                    setTimeout(() => {
                        isPromptActive = false;
                    }, 1000); // 1-second cooldown
                }
            }
        }

        // Function to sanitize player input
        function sanitizeInput(input) {
            const div = document.createElement('div');
            div.textContent = input;
            return div.innerHTML;
        }

        // Event listeners for pointerdown
        document.addEventListener('pointerdown', handleInteraction);

        // Start button event listener
        startButton.addEventListener('click', () => {
            // Reset messages and hide start button
            messageDiv.textContent = '';
            messageDiv.style.display = 'none';
            startButton.style.display = 'none';
            gameState = 'countdown';

            startCountdown(); // Begin countdown before game starts
        });

        // Leaderboard Toggle Button event listener
        toggleLeaderboardButton.addEventListener('click', () => {
            if (leaderboardContainer.classList.contains('active')) {
                // Hide leaderboard
                leaderboardContainer.classList.remove('active');
            } else {
                // Show leaderboard
                leaderboardContainer.classList.add('active');
            }
        });

        // Fetch and display the leaderboard on page load
        window.onload = function() {
            getTopScores();
        };
    </script>
</body>
</html>
