<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Click the Dot FAST!</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
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

        /* Header styling */
        #header {
            position: absolute;
            top: 100px;
            left: 50%;
            transform: translateX(-50%);
            width: 90%;
            max-width: 500px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            z-index: 5;
        }

        /* Best time display styling */
        #bestTime {
            font-size: 1em;
            color: #333;
        }

        /* Toggle switch styling */
        .switch {
            position: relative;
            display: inline-block;
            width: 60px;
            height: 34px;
            margin-right: 10px;
        }

        .switch input { 
            opacity: 0;
            width: 0;
            height: 0;
        }

        .slider {
            position: absolute;
            cursor: pointer;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: #ccc;
            transition: .4s;
            border-radius: 34px;
        }

        .slider:before {
            position: absolute;
            content: "";
            height: 26px;
            width: 26px;
            left: 4px;
            bottom: 4px;
            background-color: white;
            transition: .4s;
            border-radius: 50%;
        }

        input:checked + .slider {
            background-color: #ff6600;
        }

        input:checked + .slider:before {
            transform: translateX(26px);
        }

        /* Toggle label styling */
        .toggle-label {
            font-size: 1em;
            color: #ff6600;
            cursor: pointer;
            user-select: none;
        }

        /* Content Container styling */
        #contentContainer {
            position: absolute;
            top: 120px; /* Adjusted to accommodate the header */
            left: 50%;
            transform: translateX(-50%);
            width: 90%;
            max-width: 500px;
            height: 60vh; /* Adjust as needed */
            background-color: #ffffff;
            border: 2px solid #000;
            border-radius: 10px;
            overflow: hidden;
            z-index: 3;
        }

        /* Responsive adjustments */
        @media (max-width: 600px) {
            #contentContainer {
                height: 70vh; /* Taller on smaller screens */
            }
        }

        @media (min-width: 601px) and (max-width: 1200px) {
            #contentContainer {
                height: 60vh; /* Default for medium screens */
            }
        }

        @media (min-width: 1201px) {
            #contentContainer {
                height: 50vh; /* Shorter for larger screens */
            }
        }

        /* Game and Leaderboard container styling */
        #gameContainer, #leaderboardContainer {
            width: 100%;
            height: 100%;
            display: none; /* Hidden by default */
            position: relative;
        }

        /* Initially show the game container */
        #gameContainer.active, #leaderboardContainer.active {
            display: block;
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

        /* Enhanced Leaderboard container styling */
        #leaderboardContainer {
            padding: 20px;
            overflow-y: auto; /* Add scroll if content overflows */
            background-color: #fafafa;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            border-radius: 10px;
        }
        #leaderboardContainer h2 {
            text-align: center;
            color: #ff6600;
            margin-bottom: 20px;
            font-size: 1.8em;
            font-weight: bold;
        }
        #leaderboardList {
            list-style: none; /* Remove default numbering */
            padding-left: 0;
            margin: 0;
        }
        #leaderboardList li {
            background-color: #fff;
            margin-bottom: 10px;
            padding: 15px 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            display: flex;
            justify-content: space-between;
            align-items: center;
            transition: background-color 0.3s, transform 0.3s;
        }
        #leaderboardList li:hover {
            background-color: #f0f0f0;
            transform: translateY(-2px);
        }
        #leaderboardList li .rank {
            font-size: 1.2em;
            font-weight: bold;
            color: #ff6600;
        }
        #leaderboardList li .name {
            font-size: 1em;
            color: #333;
        }
        #leaderboardList li .time {
            font-size: 1em;
            color: #555;
        }
    </style>
</head>
<body>
    <div id="header">
        <div id="bestTime">BEST TIME: N/A</div>
        <div>
            <label class="switch">
                <input type="checkbox" id="toggleLeaderboard">
                <span class="slider round"></span>
            </label>
            <label for="toggleLeaderboard" class="toggle-label">Leaderboard</label>
        </div>
    </div>
    <div id="contentContainer">
        <div id="gameContainer" class="active">
            <button id="startButton">Start</button>
            <div id="message"></div>
            <canvas id="gameCanvas"></canvas>
        </div>
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

        // Get references to HTML elements
        const gameContainer = document.getElementById('gameContainer');
        const leaderboardContainer = document.getElementById('leaderboardContainer');
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startButton = document.getElementById('startButton');
        const messageDiv = document.getElementById('message');
        const bestTimeDiv = document.getElementById('bestTime');
        const leaderboardList = document.getElementById('leaderboardList');
        const toggleLeaderboard = document.getElementById('toggleLeaderboard');

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
                if (distance <= dotRadius) {
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

        // Leaderboard Toggle event listener
        toggleLeaderboard.addEventListener('change', () => {
            if (toggleLeaderboard.checked) {
                // Show leaderboard and hide game
                gameContainer.classList.remove('active');
                leaderboardContainer.classList.add('active');
            } else {
                // Show game and hide leaderboard
                leaderboardContainer.classList.remove('active');
                gameContainer.classList.add('active');
            }
        });

        // Fetch and display the leaderboard on page load
        window.onload = function() {
            getTopScores();
        };
    </script>
</body>
</html>
