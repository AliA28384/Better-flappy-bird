<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flappy Bird Clone</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            /* More interesting sky background */
            background: linear-gradient(to bottom, #87CEEB 0%, #64B5F6 50%, #42A5F5 100%);
            font-family: 'Inter', sans-serif;
            overflow: hidden; /* Prevent scrollbars */
            position: relative; /* For ground positioning */
        }

        #game-container {
            position: relative;
            background-color: #70C5CE; /* Lighter blue for game area, will be covered by canvas and ground */
            /* Full screen coverage */
            width: 100vw; /* Occupy full viewport width */
            height: 100vh; /* Occupy full viewport height */
            border-radius: 0; /* Remove border-radius for full screen */
            border: none; /* Remove border for full screen */
            overflow: hidden; /* Hide pipes outside canvas */
        }

        /* Specific styling for the main game canvas */
        #mainGameCanvas {
            display: block;
            background-color: transparent; /* Canvas background will be transparent to show body's background */
            width: 100%; /* Make canvas fill its container */
            height: 100%; /* Make canvas fill its container */
            z-index: 2; /* Ensure canvas is above ground */
            position: absolute; /* Position canvas over ground */
            top: 0;
            left: 0;
        }

        #game-over-screen, #start-screen, #loading-screen, #pause-screen, #shop-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            text-align: center;
            font-size: 2em;
            font-weight: bold;
            border-radius: 0; /* Full screen overlay */
            z-index: 10;
        }

        #pause-screen {
            font-size: 4em;
            color: #FFD700;
            text-shadow: 2px 2px 8px rgba(0, 0, 0, 0.8);
            display: flex; /* Ensure flex for boost options */
            flex-direction: column;
            gap: 20px; /* Space between PAUSED text and boost options */
        }

        #boost-options {
            display: flex;
            flex-direction: column;
            gap: 10px;
            margin-top: 20px;
            width: 80%;
            max-width: 300px;
        }

        #boost-options button {
            background: linear-gradient(145deg, #FFD700, #DAA520); /* Gold gradient for boost buttons */
            color: #333;
            font-size: 1.1em;
            padding: 10px 20px;
            border-radius: 30px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.3);
        }
        #boost-options button:hover {
            background: linear-gradient(145deg, #DAA520, #FFD700);
            transform: translateY(-2px);
            box-shadow: 0 6px 12px rgba(0,0,0,0.4);
        }


        #start-screen h1 {
            font-size: 3em;
            color: #FFD700; /* Gold */
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
            margin-bottom: 20px;
        }

        #start-screen p {
            margin-bottom: 30px;
        }

        #level-selection-container {
            max-height: 60vh; /* Max height for scrollable area */
            overflow-y: auto; /* Enable vertical scrolling */
            padding-right: 10px; /* Space for scrollbar */
            width: fit-content; /* Adjust width to content */
            margin: 0 auto; /* Center the container */
        }

        #level-selection {
            display: flex;
            flex-direction: column; /* Stack buttons vertically */
            gap: 15px; /* Space between level buttons */
            padding: 10px; /* Padding inside the scrollable area */
        }

        /* Webkit browsers (Chrome, Safari) scrollbar styling */
        #level-selection-container::-webkit-scrollbar {
            width: 8px;
        }

        #level-selection-container::-webkit-scrollbar-track {
            background: #388E3C;
            border-radius: 10px;
        }

        #level-selection-container::-webkit-scrollbar-thumb {
            background-color: #4CAF50;
            border-radius: 10px;
            border: 2px solid #388E3C;
        }


        button {
            padding: 15px 30px;
            font-size: 1.2em;
            font-weight: bold;
            background: linear-gradient(145deg, #4CAF50, #388E3C); /* Green gradient */
            color: white;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
            transition: all 0.3s ease;
            outline: none;
            min-width: 200px; /* Ensure buttons have a consistent width */
        }

        button:hover {
            background: linear-gradient(145deg, #388E3C, #4CAF50);
            transform: translateY(-3px);
            box-shadow: 0 8px 20px rgba(0, 0, 0, 0.4);
        }

        button:active {
            transform: translateY(0);
            box-shadow: 0 3px 10px rgba(0, 0, 0, 0.2);
        }

        button.locked {
            background: linear-gradient(145deg, #757575, #616161); /* Greyed out for locked levels */
            cursor: not-allowed;
            box-shadow: 0 3px 10px rgba(0, 0, 0, 0.2);
        }

        button.locked:hover {
            transform: none;
            box-shadow: 0 3px 10px rgba(0, 0, 0, 0.2);
        }

        button.unpurchasable-locked {
            background: #424242; /* Even darker grey for unpurchasable locked levels */
            color: #BDBDBD; /* Lighter text color */
            cursor: default; /* No pointer cursor */
            box-shadow: none;
            text-shadow: none;
        }
        button.unpurchasable-locked:hover {
            transform: none;
            box-shadow: none;
        }

        #score-display {
            position: absolute;
            top: 60px; /* Moved down to avoid collision */
            width: 100%;
            text-align: center;
            font-size: 3em;
            font-weight: bold;
            color: white;
            text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.7);
            z-index: 5;
            pointer-events: none; /* Allow clicks to pass through */
        }

        #top-left-hud { /* Container for currency displays */
            position: absolute;
            top: 20px;
            left: 20px;
            display: flex;
            gap: 20px; /* Space between currency displays */
            z-index: 6;
            align-items: center;
        }

        #coin-count-display, #gem-count-display, #revive-indicator {
            font-size: 1.8em;
            font-weight: bold;
            text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.7);
            display: flex;
            align-items: center;
            position: static; /* Ensure they behave within the flex container */
        }

        #coin-count-display {
            color: #FFD700; /* Gold for coins */
        }

        #gem-count-display {
            color: #00FFFF; /* Cyan for gems */
        }

        #revive-indicator {
            color: #FF69B4; /* Hot pink for heart */
        }

        #coin-icon, #gem-icon {
            margin-right: 8px;
            font-size: 1.5em; /* Adjust icon size */
        }

        #revive-indicator span {
            margin-left: 5px; /* Space between heart and count */
        }


        #pause-button {
            position: absolute;
            top: 20px;
            right: 20px;
            padding: 5px 8px; /* Smaller padding */
            font-size: 1.8em; /* Larger font size for emoji */
            background: rgba(0, 0, 0, 0.5);
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            z-index: 11; /* Above other UI elements */
            transition: background-color 0.2s;
            line-height: 1; /* Adjust line height for emoji */
        }

        #pause-button:hover {
            background: rgba(0, 0, 0, 0.7);
        }

        #ground {
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 100px; /* Height of the ground */
            background: linear-gradient(to top, #6D4C41 0%, #8D6E63 50%, #A1887F 100%); /* Brown gradient for ground */
            z-index: 1; /* Below pipes and bird, above game-container background */
        }

        #message-box { /* For insufficient gems message */
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(255, 0, 0, 0.8);
            color: white;
            padding: 20px 30px;
            border-radius: 10px;
            font-size: 1.5em;
            font-weight: bold;
            text-align: center;
            z-index: 100; /* Ensure it's on top of everything */
            opacity: 0;
            transition: opacity 0.5s ease-in-out;
            pointer-events: none; /* Allow clicks to pass through when hidden */
        }

        #message-box.show {
            opacity: 1;
        }

        /* Shop Specific Styles */
        #shop-screen {
            display: none; /* Hidden by default */
            flex-direction: column;
            padding: 20px;
            box-sizing: border-box;
        }

        #shop-screen h2 {
            font-size: 2.5em;
            color: #FFD700;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
            margin-bottom: 20px;
        }

        #shop-nav {
            display: flex;
            justify-content: center;
            gap: 10px; /* Reduced gap for smaller text */
            margin-bottom: 30px;
            flex-wrap: wrap; /* Allow tabs to wrap on smaller screens */
        }

        .shop-tab-button {
            padding: 10px 20px; /* Reduced padding */
            font-size: 1em; /* Smaller font size */
            font-weight: bold;
            background: linear-gradient(145deg, #007BFF, #0056b3); /* Blue gradient */
            color: white;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
            transition: all 0.2s ease;
            outline: none;
        }

        .shop-tab-button:hover {
            background: linear-gradient(145deg, #0056b3, #007BFF);
            transform: translateY(-2px);
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
        }

        .shop-tab-button.active {
            background: linear-gradient(145deg, #00C9FF, #007BFF); /* Lighter blue when active */
            transform: translateY(-1px);
            box-shadow: 0 3px 8px rgba(0, 0, 0, 0.3);
            border: 2px solid #00FFFF; /* Highlight active tab */
        }

        #shop-content {
            flex-grow: 1; /* Take remaining space */
            width: 90%;
            max-width: 800px;
            background-color: rgba(0, 0, 0, 0.5);
            border-radius: 15px;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            overflow-y: auto; /* Enable scrolling for content if needed */
            box-sizing: border-box;
        }

        .shop-menu {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            justify-content: center;
            width: 100%;
            padding: 10px;
        }

        .shop-item {
            background-color: rgba(255, 255, 255, 0.1);
            border: 2px solid rgba(255, 255, 255, 0.2);
            border-radius: 10px;
            padding: 15px;
            width: 150px;
            text-align: center;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
            transition: transform 0.2s ease;
        }

        .shop-item:hover {
            transform: translateY(-5px);
        }

        .shop-item h3 {
            margin-top: 0;
            font-size: 1.1em;
            color: #FFD700;
        }

        .shop-item p {
            font-size: 0.9em;
            margin: 5px 0 10px 0;
            color: #E0E0E0;
        }

        .shop-item .price {
            font-size: 1.1em;
            font-weight: bold;
            color: #FFD700; /* Gold for coin prices */
            margin-bottom: 10px;
        }

        .shop-item .price.gem-price {
            color: #00FFFF; /* Cyan for gem prices */
        }

        .shop-item button {
            padding: 8px 15px;
            font-size: 0.9em;
            min-width: unset; /* Override general button min-width */
            border-radius: 20px;
            background: linear-gradient(145deg, #FF6F00, #E65100); /* Orange for buy */
            box-shadow: 0 3px 6px rgba(0, 0, 0, 0.2);
        }

        .shop-item button:hover {
            background: linear-gradient(145deg, #E65100, #FF6F00);
            transform: translateY(-2px);
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
        }

        .shop-item button.equipped {
            background: linear-gradient(145deg, #4CAF50, #388E3C); /* Green for equipped */
            cursor: default;
            transform: none;
            box-shadow: none;
        }

        .shop-item button.owned {
            background: linear-gradient(145deg, #2196F3, #1976D2); /* Blue for owned (but not equipped) */
        }

        #close-shop-button {
            margin-top: 30px;
            padding: 15px 40px;
            font-size: 1.3em;
            background: linear-gradient(145deg, #D32F2F, #B71C1C); /* Red gradient */
        }

        /* Shop button styling when part of the flow, not absolutely positioned */
        #start-screen #shop-button,
        #game-over-screen #shop-button-gameover {
            position: static; /* Remove absolute positioning */
            margin-top: 20px; /* Add some space above/below */
            margin-bottom: 15px; /* Add some space above/below */
            padding: 10px 20px; /* Adjust padding for better fit */
            font-size: 1.2em; /* Adjust font size */
            min-width: 200px; /* Match other buttons */
        }


        /* Responsive adjustments */
        @media (max-width: 600px) {
            #score-display {
                font-size: 2em;
                top: 40px; /* Adjusted for smaller screens */
            }
            #top-left-hud {
                top: 10px;
                left: 10px;
                gap: 10px; /* Smaller gap on small screens */
            }
            #coin-count-display, #revive-indicator, #gem-count-display {
                font-size: 1.4em;
            }
            #pause-button {
                top: 10px;
                right: 10px;
                padding: 4px 6px; /* Even smaller padding */
                font-size: 1.5em; /* Smaller emoji on small screens */
            }
            /* Remove absolute positioning for shop button on small screens too */
            #start-screen #shop-button,
            #game-over-screen #shop-button-gameover {
                position: static;
                margin-top: 10px;
                margin-bottom: 10px;
                padding: 8px 15px;
                font-size: 1em;
                min-width: 150px;
            }
            #start-screen h1 {
                font-size: 2em;
            }
            #game-over-screen {
                font-size: 1.2em;
            }
            button {
                padding: 10px 20px;
                font-size: 1em;
                min-width: 150px;
            }
            #level-selection {
                gap: 10px;
            }
            #ground {
                height: 80px;
            }
            #level-selection-container {
                max-height: 50vh; /* Adjust for smaller screens */
            }
            #pause-screen {
                font-size: 3em;
            }
            #message-box {
                font-size: 1.2em;
                padding: 15px 20px;
            }
            #shop-screen h2 {
                font-size: 1.8em;
            }
            .shop-tab-button {
                padding: 8px 15px;
                font-size: 0.9em;
            }
            .shop-item {
                width: 120px;
                padding: 10px;
            }
            .shop-item h3 {
                font-size: 1em;
            }
            .shop-item p {
                font-size: 0.8em;
            }
            .shop-item button {
                font-size: 0.8em;
                padding: 6px 10px;
            }
            #close-shop-button {
                padding: 10px 20px;
                font-size: 1em;
            }
        }
    </style>
    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

        // Expose Firebase modules to the global scope for use in the main script
        window.firebase = {
            initializeApp,
            getAuth,
            signInAnonymously,
            signInWithCustomToken,
            onAuthStateChanged,
            getFirestore,
            doc,
            getDoc,
            setDoc
        };
    </script>
</head>
<body>
    <div id="game-container">
        <canvas id="mainGameCanvas"></canvas> <!-- Changed ID to mainGameCanvas -->
        <div id="score-display">0</div>
        
        <div id="top-left-hud"> <!-- New container for currency displays -->
            <div id="coin-count-display">
                <span id="coin-icon">💰</span> <span id="coins-collected">0</span>
            </div>
            <div id="revive-indicator" style="display: none;">
                ❤️ <span id="revive-count">0</span>
            </div>
            <div id="gem-count-display" style="display: none;">
                <span id="gem-icon">💎</span> <span id="gems-collected">0</span>
            </div>
        </div>

        <div id="ground"></div> <!-- Ground element -->

        <button id="pause-button" style="display: none;">⏸️</button> <!-- Changed to emoji -->
        <!-- The shop button is now part of the screen flows, not absolutely positioned -->

        <div id="loading-screen">
            <p>Loading game...</p>
        </div>

        <div id="start-screen" style="display: none;">
            <h1>Flappy Bird</h1>
            <p>Choose your difficulty:</p>
            <button id="shop-button">🛒 Shop</button> <!-- Shop button moved here -->
            <div id="level-selection-container">
                <div id="level-selection">
                    <!-- Level buttons will be dynamically inserted here by JavaScript -->
                </div>
            </div>
        </div>

        <div id="pause-screen" style="display: none;">
            PAUSED
            <div id="boost-options">
                <!-- Boost buttons will be dynamically inserted here -->
            </div>
        </div>

        <div id="game-over-screen" style="display: none;">
            <p>Game Over!</p>
            <p>Score: <span id="final-score">0</span></p>
            <p>Best Score for Level <span id="best-score-level"></span>: <span id="best-score-display">0</span></p>
            <button id="reviveButton" style="display: none; margin-bottom: 15px;">Revive!</button>
            <button id="restartButton">Play Again</button>
            <button id="selectLevelButton" style="margin-top: 15px;">Choose Level</button>
            <button id="shop-button-gameover">🛒 Shop</button> <!-- Shop button for game over screen -->
        </div>

        <!-- Shop Screen -->
        <div id="shop-screen" style="display: none;">
            <h2>The Shop</h2>
            <div id="shop-nav">
                <button class="shop-tab-button active" data-tab="skins">Skins</button>
                <button class="shop-tab-button" data-tab="boosts">Boosts</button>
                <button class="shop-tab-button" data-tab="exchanges">Exchanges</button>
            </div>
            <div id="shop-content">
                <!-- Shop menu content will be dynamically loaded here -->
            </div>
            <button id="close-shop-button">Close Shop</button>
        </div>

        <div id="message-box" style="display: none;"></div> <!-- New message box -->
    </div>

    <script type="module">
        // Import Firebase modules from the global scope
        const { initializeApp, getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged, getFirestore, doc, getDoc, setDoc } = window.firebase;

        // Get the canvas element and its 2D rendering context
        const mainGameCanvas = document.getElementById('mainGameCanvas'); // Changed variable name
        const ctx = mainGameCanvas.getContext('2d');

        // Get UI elements
        const scoreDisplay = document.getElementById('score-display');
        const coinCountDisplay = document.getElementById('coins-collected');
        const reviveIndicator = document.getElementById('revive-indicator');
        const reviveCountDisplay = document.getElementById('revive-count'); // New element for revive count
        const gemCountDisplayElement = document.getElementById('gems-collected'); // New element for gem count
        const gemIndicator = document.getElementById('gem-count-display'); // New element for gem indicator
        const loadingScreen = document.getElementById('loading-screen');
        const startScreen = document.getElementById('start-screen');
        const levelSelectionContainer = document.getElementById('level-selection-container');
        const levelSelectionDiv = document.getElementById('level-selection');
        const gameOverScreen = document.getElementById('game-over-screen');
        const finalScoreDisplay = document.getElementById('final-score');
        const bestScoreLevelDisplay = document.getElementById('best-score-level');
        const bestScoreDisplay = document.getElementById('best-score-display');
        const restartButton = document.getElementById('restartButton');
        const selectLevelButton = document.getElementById('selectLevelButton');
        const reviveButton = document.getElementById('reviveButton'); // New revive button
        const groundElement = document.getElementById('ground');
        const pauseButton = document.getElementById('pause-button');
        const shopButton = document.getElementById('shop-button'); // Shop button on start screen
        const shopButtonGameOver = document.getElementById('shop-button-gameover'); // Shop button on game over screen
        const pauseScreen = document.getElementById('pause-screen');
        const boostOptionsDiv = document.getElementById('boost-options'); // New boost options div
        const shopScreen = document.getElementById('shop-screen'); // New shop screen element
        const shopNav = document.getElementById('shop-nav'); // Shop navigation
        const shopContent = document.getElementById('shop-content'); // Shop content area
        const closeShopButton = document.getElementById('close-shop-button'); // Close shop button
        const messageBox = document.getElementById('message-box'); // New message box element

        // Firebase variables
        let app;
        let db;
        let auth;
        let userId = null;
        let isAuthReady = false;

        // Game variables
        let gameRunning = false;
        let isPaused = false; // New pause state
        let score = 0;
        let coinsCollected = 0; // Persistent coin count
        let reviveCount = 0; // Persistent revive count
        let gemsCollected = 0; // New persistent gem count
        let animationFrameId = null; // To store the requestAnimationFrame ID
        let currentPipeGap = 150; // Default pipe gap, will be set by level
        let selectedLevel = 1; // Store the currently selected level
        let bestScores = {}; // Object to store best scores for all levels
        let unlockedLevels = [1]; // Initially, only level 1 is unlocked

        // Shop related state
        let equippedSkin = 'default'; // Default bird color
        let ownedSkins = ['default']; // Initially own only the default skin
        let purchasedBoosts = []; // Stores boosts purchased but not yet activated

        // To keep track of which screen was open before shop
        let previousScreen = null;

        // Bird properties
        const bird = {
            x: 50,
            y: 0, // Initial y will be set to mainGameCanvas center in resetGame
            radius: 15,
            velocity: 0,
            gravity: 0.15, // Adjusted gravity to make it fall much slower
            lift: -4, // Adjusted lift to make it jump less high (from -6 to -4)
            color: '#FFD700' // Gold color for the bird (default)
        };

        // Pipe properties
        const pipes = [];
        const pipeWidth = 50;
        const pipeSpeed = 3;
        let pipeSpawnInterval = 1500; // Milliseconds between pipe spawns
        let lastPipeSpawnTime = 0;

        // Collectible properties (coins and hearts)
        const collectibles = []; // Array to hold all active collectibles
        const coinRadius = 10;
        const heartSize = 20; // Size for drawing the heart
        const goldCoinSpawnChance = 0.28; // 28% chance for a gold coin
        const redCoinSpawnChance = 0.09; // 9% chance for a red coin
        const reviveSpawnChance = 0.05; // 5% chance for a heart
        const gemSpawnChance = 0.04; // 4% chance for a gem

        // Level configurations: pipeGap decreases for higher levels
        const NUM_LEVELS = 25;
        const BASE_PIPE_GAP = 250; // Base gap for Level 1 (further apart)
        const MIN_PIPE_GAP = 80; // Minimum gap for hardest level (adjusted for new base - Level 25)
        const PIPE_GAP_DECREMENT_PER_LEVEL = (BASE_PIPE_GAP - MIN_PIPE_GAP) / (NUM_LEVELS - 1);

        const LEVEL_CONFIGS = {};
        for (let i = 1; i <= NUM_LEVELS; i++) {
            const gap = BASE_PIPE_GAP - (i - 1) * PIPE_GAP_DECREMENT_PER_LEVEL;
            LEVEL_CONFIGS[i] = {
                pipeGap: Math.max(MIN_PIPE_GAP, gap), // Ensure gap doesn't go below min
                name: `Level ${i}`
            };
        }

        // Building properties for background art
        const buildings = [];
        const buildingColors = ['#4A4A4A', '#5A5A5A', '#6A6A6A', '#7A7A7A']; // Shades of grey
        const buildingSpeed = pipeSpeed * 0.2; // Slower than pipes for parallax effect

        // Shop Data
        const SKINS_DATA = {
            'default': { name: 'Default (Gold)', cost: 0, currency: 'coins', color: '#FFD700' },
            'blue': { name: 'Blue Bird', cost: 100, currency: 'coins', color: '#00BFFF' },
            'green': { name: 'Green Bird', cost: 100, currency: 'coins', color: '#32CD32' },
            'yellow': { name: 'Yellow Bird', cost: 150, currency: 'coins', color: '#FFFF00' },
            'orange': { name: 'Orange Bird', cost: 150, currency: 'coins', color: '#FFA500' },
            'red': { name: 'Red Bird', cost: 150, currency: 'coins', color: '#FF0000' },
            'pink': { name: 'Pink Bird', cost: 200, currency: 'coins', color: '#FF69B4' },
            'white': { name: 'White Bird', cost: 200, currency: 'coins', color: '#FFFFFF' },
            'purple': { name: 'Purple Bird', cost: 200, currency: 'coins', color: '#800080' },
            // New skins
            'newcastle': { name: 'Newcastle', cost: 300, currency: 'coins', type: 'striped' }, // Custom type for drawing
            'liverpool': { name: 'Liverpool', cost: 300, currency: 'coins', type: 'layered_circle', outerColor: 'white', innerColor: 'red', text: 'L' },
            'man_city': { name: 'Man City', cost: 300, currency: 'coins', type: 'layered_circle', outerColor: '#ADD8E6', innerColor: '#87CEEB', text: 'M' }
        };

        const BOOSTS_DATA = {
            'score5': { name: '+5 Score', cost: 10, currency: 'coins', value: 5 },
            'score50': { name: '+50 Score', cost: 75, currency: 'coins', value: 50 },
            'score100': { name: '+100 Score', cost: 175, currency: 'coins', value: 100 }
        };

        // Function to initialize bird position based on current mainGameCanvas height
        function initializeBirdPosition() {
            bird.y = mainGameCanvas.height / 2;
        }

        // --- Drawing Functions ---

        // Draw the bird
        function drawBird() {
            // Standard bird properties
            const birdX = bird.x;
            const birdY = bird.y;
            const birdRadius = bird.radius;

            const currentSkin = SKINS_DATA[equippedSkin];

            // Save the current canvas state before applying transformations/clips
            ctx.save();
            ctx.beginPath();
            ctx.arc(birdX, birdY, birdRadius, 0, Math.PI * 2);
            ctx.clip(); // Clip future drawings to this circle

            if (currentSkin.type === 'striped') {
                // Newcastle: White background with central black vertical stripe
                ctx.fillStyle = 'white';
                ctx.fillRect(birdX - birdRadius, birdY - birdRadius, birdRadius * 2, birdRadius * 2); // Fill the entire circle area with white

                // Draw central black stripe
                const stripeWidth = birdRadius / 3; // Adjust stripe width as needed
                ctx.fillStyle = 'black';
                ctx.fillRect(birdX - stripeWidth / 2, birdY - birdRadius, stripeWidth, birdRadius * 2);

            } else if (currentSkin.type === 'layered_circle') {
                const outerRadius = birdRadius;
                const innerRadius = birdRadius * 0.6; // Inner circle is smaller to show more border

                // Outer circle
                ctx.beginPath();
                ctx.arc(birdX, birdY, outerRadius, 0, Math.PI * 2);
                ctx.fillStyle = currentSkin.outerColor;
                ctx.fill();
                ctx.closePath();

                // Inner circle
                ctx.beginPath();
                ctx.arc(birdX, birdY, innerRadius, 0, Math.PI * 2);
                ctx.fillStyle = currentSkin.innerColor;
                ctx.fill();
                ctx.closePath();

                // Add text to the inner circle
                if (currentSkin.text) {
                    ctx.fillStyle = 'white'; // Text color for contrast
                    ctx.font = `bold ${innerRadius * 1.2}px Arial`; // Adjust font size based on inner circle
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'middle';
                    ctx.fillText(currentSkin.text, birdX, birdY);
                }
            } else {
                // Default single color circle
                ctx.beginPath();
                ctx.arc(birdX, birdY, birdRadius, 0, Math.PI * 2);
                ctx.fillStyle = currentSkin.color;
                ctx.fill();
                ctx.closePath();
            }

            // Restore the canvas state, removing the clip
            ctx.restore();

            // Always draw the black outline on top (outside the clip, so it's a full circle)
            ctx.beginPath();
            ctx.arc(birdX, birdY, birdRadius, 0, Math.PI * 2);
            ctx.strokeStyle = 'black';
            ctx.lineWidth = 2;
            ctx.stroke();
            ctx.closePath();

            // Always draw the eye on top
            ctx.beginPath();
            ctx.arc(birdX + birdRadius / 2.5, birdY - birdRadius / 3, birdRadius / 4, 0, Math.PI * 2);
            ctx.fillStyle = 'black';
            ctx.fill();
            ctx.closePath();
        }

        // Draw a single pipe segment
        function drawPipe(x, y, height) {
            ctx.fillStyle = '#4CAF50'; // Green color for pipes
            ctx.fillRect(x, y, pipeWidth, height);
            ctx.strokeStyle = '#388E3C'; // Darker green border
            ctx.lineWidth = 3;
            ctx.strokeRect(x, y, pipeWidth, height);

            // Add a darker cap to the pipe for better visual
            const capHeight = 15;
            ctx.fillStyle = '#388E3C';
            if (y === 0) { // Top pipe
                ctx.fillRect(x - 5, height - capHeight, pipeWidth + 10, capHeight);
            } else { // Bottom pipe
                ctx.fillRect(x - 5, y, pipeWidth + 10, capHeight);
            }
        }

        // Create a single building object
        function createBuilding(xOffset) {
            const groundHeight = groundElement.offsetHeight;
            const minBuildingHeight = 50;
            // Max height to ensure buildings don't go too high and leave space for bird/pipes
            const maxBuildingHeight = mainGameCanvas.height - groundHeight - 150;
            const buildingHeight = Math.random() * (maxBuildingHeight - minBuildingHeight) + minBuildingHeight;
            const minBuildingWidth = 40;
            const maxBuildingWidth = 100;
            const buildingWidth = Math.random() * (maxBuildingWidth - minBuildingWidth) + minBuildingWidth;
            const color = buildingColors[Math.floor(Math.random() * buildingColors.length)];

            buildings.push({
                x: xOffset,
                y: mainGameCanvas.height - groundHeight - buildingHeight,
                width: buildingWidth,
                height: buildingHeight,
                color: color
            });
        }

        // Initialize a set of buildings across the screen
        function initializeBuildings() {
            buildings.length = 0; // Clear existing buildings
            let currentX = 0;
            while (currentX < mainGameCanvas.width + 200) { // Add some off-screen buildings initially
                createBuilding(currentX);
                currentX += buildings[buildings.length - 1].width + Math.random() * 20; // Add some random gap between buildings
            }
        }

        // Draw all buildings (without windows)
        function drawBuildings() {
            for (const b of buildings) {
                ctx.fillStyle = b.color;
                ctx.fillRect(b.x, b.y, b.width, b.height);
            }
        }

        // Update building positions and manage spawning/removal
        function updateBuildings() {
            for (let i = buildings.length - 1; i >= 0; i--) {
                buildings[i].x -= buildingSpeed;
                // Remove off-screen buildings
                if (buildings[i].x + buildings[i].width < 0) {
                    buildings.splice(i, 1);
                }
            }

            // Add new buildings if the last one is visible enough (or if array is empty)
            if (buildings.length === 0 || buildings[buildings.length - 1].x < mainGameCanvas.width - 100) {
                createBuilding(mainGameCanvas.width + Math.random() * 50); // Spawn new building slightly off-screen to the right
            }
        }

        // Draw a gold coin
        function drawGoldCoin(x, y) {
            ctx.beginPath();
            ctx.arc(x, y, coinRadius, 0, Math.PI * 2);
            ctx.fillStyle = '#FFD700'; // Gold color
            ctx.fill();
            ctx.strokeStyle = '#B8860B';
            ctx.lineWidth = 2;
            ctx.stroke();
            ctx.closePath();
            ctx.fillStyle = '#B8860B';
            ctx.font = 'bold 12px Arial';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText('$', x, y);
        }

        // Draw a red coin
        function drawRedCoin(x, y) {
            ctx.beginPath();
            ctx.arc(x, y, coinRadius, 0, Math.PI * 2);
            ctx.fillStyle = '#FF0000'; // Red color
            ctx.fill();
            ctx.strokeStyle = '#8B0000'; // Darker red border
            ctx.lineWidth = 2;
            ctx.stroke();
            ctx.closePath();
            ctx.fillStyle = '#FFFFFF'; // White for the 'S'
            ctx.font = 'bold 12px Arial';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText('S', x, y); // Use 'S' for red coin
        }

        // Draw a heart
        function drawHeart(x, y) {
            ctx.fillStyle = '#FF69B4'; // Hot pink color
            ctx.beginPath();
            // Left arc
            ctx.arc(x - heartSize / 4, y - heartSize / 4, heartSize / 2, Math.PI, 0);
            // Right arc
            ctx.arc(x + heartSize / 4, y - heartSize / 4, heartSize / 2, Math.PI, 0);
            // Bottom point
            ctx.lineTo(x, y + heartSize / 2);
            ctx.closePath();
            ctx.fill();
        }

        // Draw a gem (using a simple diamond shape for now)
        function drawGem(x, y) {
            ctx.fillStyle = '#00FFFF'; // Cyan color for gem
            ctx.beginPath();
            ctx.moveTo(x, y - coinRadius); // Top point
            ctx.lineTo(x + coinRadius, y); // Right point
            ctx.lineTo(x, y + coinRadius); // Bottom point
            ctx.lineTo(x - coinRadius, y); // Left point
            ctx.closePath();
            ctx.fill();
            ctx.strokeStyle = '#00AAAA'; // Darker cyan border
            ctx.lineWidth = 2;
            ctx.stroke();
        }

        // Draw all collectibles
        function drawCollectibles() {
            for (const c of collectibles) {
                if (c.type === 'goldCoin') {
                    drawGoldCoin(c.x, c.y);
                } else if (c.type === 'redCoin') {
                    drawRedCoin(c.x, c.y);
                } else if (c.type === 'heart') {
                    drawHeart(c.x, c.y);
                } else if (c.type === 'gem') { // Draw gem
                    drawGem(c.x, c.y);
                }
            }
        }

        // Update collectible positions and remove off-screen
        function updateCollectibles() {
            for (let i = collectibles.length - 1; i >= 0; i--) {
                const c = collectibles[i];
                c.x -= pipeSpeed; // Move with pipes

                // Remove if off-screen
                if (c.x + c.radius < 0) {
                    collectibles.splice(i, 1);
                }
            }
        }

        // --- Game Logic Functions ---

        // Update bird's position and velocity
        function updateBird() {
            bird.velocity += bird.gravity; // Apply gravity
            bird.y += bird.velocity; // Update position

            // Prevent bird from going above the top of the mainGameCanvas
            if (bird.y - bird.radius < 0) {
                bird.y = bird.radius;
                bird.velocity = 0;
            }

            // Game Over if bird hits the ground (adjusted for ground height)
            const groundHeight = groundElement.offsetHeight;
            if (bird.y + bird.radius > mainGameCanvas.height - groundHeight) {
                bird.y = mainGameCanvas.height - groundHeight - bird.radius;
                gameOver();
            }
        }

        // Generate new pipes and collectibles
        function spawnPipes() {
            const currentTime = Date.now();
            if (currentTime - lastPipeSpawnTime > pipeSpawnInterval) {
                const groundHeight = groundElement.offsetHeight;
                // Random height for the top pipe, considering the ground
                const minHeight = 50;
                const maxHeight = mainGameCanvas.height - currentPipeGap - minHeight - groundHeight;
                const topPipeHeight = Math.floor(Math.random() * (maxHeight - minHeight + 1)) + minHeight;

                // Create top pipe - this will be the scoring pipe
                pipes.push({
                    x: mainGameCanvas.width,
                    y: 0,
                    height: topPipeHeight,
                    passed: false,
                    isScoringPipe: true // Only the top pipe of a pair increments the score
                });

                // Create bottom pipe
                pipes.push({
                    x: mainGameCanvas.width,
                    y: topPipeHeight + currentPipeGap,
                    height: mainGameCanvas.height - (topPipeHeight + currentPipeGap) - groundHeight,
                    passed: false,
                    isScoringPipe: false // This pipe does not increment the score
                });

                // Calculate center of the pipe gap for collectibles
                const collectibleY = topPipeHeight + currentPipeGap / 2;
                const collectibleX = mainGameCanvas.width + pipeWidth / 2; // Roughly center of the pipe

                // Determine which collectible to spawn based on rarity (rarest first)
                const randomChance = Math.random();
                if (randomChance < reviveSpawnChance) {
                    collectibles.push({ type: 'heart', x: collectibleX, y: collectibleY, radius: heartSize / 2 });
                } else if (randomChance < reviveSpawnChance + redCoinSpawnChance) {
                    collectibles.push({ type: 'redCoin', x: collectibleX, y: collectibleY, radius: coinRadius });
                } else if (randomChance < reviveSpawnChance + redCoinSpawnChance + gemSpawnChance) { // New gem spawn chance
                    collectibles.push({ type: 'gem', x: collectibleX, y: collectibleY, radius: coinRadius }); // Using coinRadius for gem size
                } else if (randomChance < reviveSpawnChance + redCoinSpawnChance + gemSpawnChance + goldCoinSpawnChance) {
                    collectibles.push({ type: 'goldCoin', x: collectibleX, y: collectibleY, radius: coinRadius });
                }

                lastPipeSpawnTime = currentTime;
            }
        }

        // Update pipe positions and remove off-screen pipes
        function updatePipes() {
            for (let i = pipes.length - 1; i >= 0; i--) {
                const p = pipes[i];
                p.x -= pipeSpeed; // Move pipe to the left

                // Check for scoring, only if it's a scoring pipe and hasn't been passed yet
                if (!p.passed && p.isScoringPipe && p.x + pipeWidth < bird.x - bird.radius) {
                    score++;
                    scoreDisplay.textContent = score;
                    p.passed = true; // Mark as passed
                }

                // Remove pipe if it's off-screen
                if (p.x + pipeWidth < 0) {
                    pipes.splice(i, 1);
                }
            }
        }

        // Check for collisions between bird and pipes
        function checkCollision() {
            for (let i = 0; i < pipes.length; i++) {
                const p = pipes[i];

                // Check if bird's x-range overlaps with pipe's x-range
                if (bird.x + bird.radius > p.x && bird.x - bird.radius < p.x + pipeWidth) {
                    // Check if bird's y-range overlaps with pipe's y-range
                    if (bird.y + bird.radius > p.y && bird.y - bird.radius < p.y + p.height) {
                        gameOver(); // Collision detected
                        return; // Exit function early
                    }
                }
            }
        }

        // Check for collisions with collectibles
        function checkCollectibleCollision() {
            for (let i = collectibles.length - 1; i >= 0; i--) {
                const c = collectibles[i];
                // Simple circular collision detection
                const dx = bird.x - c.x;
                const dy = bird.y - c.y;
                const distance = Math.sqrt(dx * dx + dy * dy);

                if (distance < bird.radius + c.radius) {
                    // Collision detected
                    if (c.type === 'goldCoin') {
                        coinsCollected += 1;
                        updateCoinDisplay(); // Update display
                    } else if (c.type === 'redCoin') {
                        coinsCollected += 10; // Red coin worth 10 normal coins
                        updateCoinDisplay(); // Update display
                    } else if (c.type === 'heart') {
                        reviveCount++;
                        updateReviveDisplay();
                    } else if (c.type === 'gem') { // Collect gem
                        gemsCollected++;
                        updateGemDisplay();
                    }
                    collectibles.splice(i, 1); // Remove collected item
                    saveUserData(); // Save currency changes immediately
                }
            }
        }

        // --- Firebase Functions ---

        async function loadUserData() {
            if (!db || !userId) {
                console.log("Firestore or userId not ready for loading user data.");
                return;
            }
            try {
                const docRef = doc(db, `artifacts/${__app_id}/users/${userId}/flappyBirdData`, 'userDataDocument');
                const docSnap = await getDoc(docRef);

                if (docSnap.exists()) {
                    const data = docSnap.data();
                    bestScores = data.bestScores || {};
                    // Ensure unlockedLevels is an array and level 1 is always included
                    unlockedLevels = Array.isArray(data.unlockedLevels) ? data.unlockedLevels : [1];
                    if (!unlockedLevels.includes(1)) {
                        unlockedLevels.push(1);
                        unlockedLevels.sort((a, b) => a - b);
                    }
                    coinsCollected = data.coinsCollected || 0;
                    reviveCount = data.reviveCount || 0;
                    gemsCollected = data.gemsCollected || 0;
                    equippedSkin = data.equippedSkin || 'default';
                    ownedSkins = Array.isArray(data.ownedSkins) ? data.ownedSkins : ['default'];
                    purchasedBoosts = Array.isArray(data.purchasedBoosts) ? data.purchasedBoosts : [];


                    console.log("Loaded user data:", data);
                } else {
                    console.log("No user data document found, starting fresh.");
                    bestScores = {};
                    unlockedLevels = [1]; // Ensure level 1 is always unlocked for new users
                    coinsCollected = 0;
                    reviveCount = 0;
                    gemsCollected = 0;
                    equippedSkin = 'default';
                    ownedSkins = ['default'];
                    purchasedBoosts = [];
                    // Save initial data if document doesn't exist
                    await saveUserData();
                }
            } catch (e) {
                console.error("Error loading user data: ", e);
            }
        }

        async function saveUserData() {
            if (!db || !userId) {
                console.log("Firestore or userId not ready for saving user data.");
                return;
            }
            try {
                const docRef = doc(db, `artifacts/${__app_id}/users/${userId}/flappyBirdData`, 'userDataDocument');
                await setDoc(docRef, {
                    bestScores: bestScores,
                    unlockedLevels: unlockedLevels,
                    coinsCollected: coinsCollected,
                    reviveCount: reviveCount,
                    gemsCollected: gemsCollected,
                    equippedSkin: equippedSkin,
                    ownedSkins: ownedSkins,
                    purchasedBoosts: purchasedBoosts
                }, { merge: true });
                console.log("User data saved.");
            } catch (e) {
                console.error("Error saving user data: ", e);
            }
        }

        // --- Game State Management ---

        // Function to get the cost of unlocking a level
        function getLevelUnlockCost(level) {
            if (level === 1) return 0; // Level 1 is free
            return level + 1; // Level 2 costs 3, Level 3 costs 4, etc.
        }

        // Function to show a temporary message
        let messageBoxTimeout;
        function showTemporaryMessage(message) {
            messageBox.textContent = message;
            messageBox.style.display = 'block';
            messageBox.classList.add('show');

            clearTimeout(messageBoxTimeout);
            messageBoxTimeout = setTimeout(() => {
                messageBox.classList.remove('show');
                // Give time for fade out before hiding
                setTimeout(() => {
                    messageBox.style.display = 'none';
                }, 500);
            }, 2000); // Message visible for 2 seconds
        }

        // Game Over function
        function gameOver() {
            gameRunning = false;
            if (animationFrameId) {
                cancelAnimationFrame(animationFrameId); // Stop the animation loop
            }

            // Update best score if current score is higher
            const currentLevelBestScore = bestScores[`level_${selectedLevel}`] || 0;
            if (score > currentLevelBestScore) {
                bestScores[`level_${selectedLevel}`] = score;
            }
            saveUserData(); // Save all user data including best scores and current currency

            finalScoreDisplay.textContent = score;
            bestScoreLevelDisplay.textContent = selectedLevel;
            bestScoreDisplay.textContent = bestScores[`level_${selectedLevel}`] || 0; // Display the updated best score

            // Show/hide revive button based on reviveCount
            if (reviveCount > 0) {
                reviveButton.style.display = 'block';
            } else {
                reviveButton.style.display = 'none';
            }

            gameOverScreen.style.display = 'flex';
            pauseButton.style.display = 'none'; // Hide pause button on game over
            shopButton.style.display = 'none'; // Hide start screen shop button
            shopButtonGameOver.style.display = 'block'; // Show game over shop button
        }

        // Resets bird position and velocity without resetting other game elements
        function resetBirdPositionAndPhysics() {
            initializeBirdPosition();
            bird.velocity = 0;
        }

        // Reset game state for a new game session (resets score, pipes, collectibles, but keeps coins/revives/gems)
        function resetGameSession() {
            resetBirdPositionAndPhysics();
            pipes.length = 0; // Clear all pipes
            collectibles.length = 0; // Clear all collectibles
            score = 0;
            scoreDisplay.textContent = score;
            // coinsCollected, reviveCount, and gemsCollected are NOT reset here
            lastPipeSpawnTime = 0; // Reset pipe spawn timer
            gameOverScreen.style.display = 'none';
            pauseScreen.style.display = 'none'; // Ensure pause screen is hidden
            isPaused = false; // Ensure game is not paused
            pauseButton.style.display = 'none'; // Hide pause button until game starts
            shopButton.style.display = 'none'; // Hide shop button during game
            shopButtonGameOver.style.display = 'none'; // Hide game over shop button
            reviveButton.style.display = 'none'; // Hide revive button
            boostOptionsDiv.innerHTML = ''; // Clear boost options on game reset
        }

        // Start the game with a chosen level
        function startGame(level) {
            console.log(`Starting game at Level ${level} with pipe gap: ${LEVEL_CONFIGS[level].pipeGap}`);
            selectedLevel = level; // Store the selected level
            currentPipeGap = LEVEL_CONFIGS[level].pipeGap; // Set the pipe gap for the chosen level

            // Reset only game-specific elements for a new game session
            resetGameSession();

            if (!gameRunning) { // Prevent multiple game loops if button is clicked rapidly
                gameRunning = true;
                startScreen.style.display = 'none';
                gameOverScreen.style.display = 'none'; // Ensure game over screen is hidden
                shopScreen.style.display = 'none'; // Ensure shop screen is hidden
                pauseButton.style.display = 'block'; // Show pause button
                // Immediately allow the first pipe to spawn
                lastPipeSpawnTime = Date.now() - pipeSpawnInterval;
                initializeBuildings(); // Initialize buildings for the new game
                gameLoop(); // Start the game loop
            }
        }

        // Activate a purchased boost
        function activateBoost(boostToActivate) {
            score += boostToActivate.value;
            scoreDisplay.textContent = score;
            showTemporaryMessage(`${boostToActivate.name} activated!`);

            // Remove the activated boost from the purchasedBoosts array
            const index = purchasedBoosts.findIndex(b => b.type === boostToActivate.type && b.name === boostToActivate.name && b.value === boostToActivate.value);
            if (index > -1) {
                purchasedBoosts.splice(index, 1);
            }
            saveUserData(); // Save updated purchased boosts
            renderBoostOptions(); // Re-render boost options on pause screen
        }

        // Toggle pause state
        function togglePause() {
            if (!gameRunning) return; // Cannot pause if game is not running

            isPaused = !isPaused;
            if (isPaused) {
                cancelAnimationFrame(animationFrameId); // Stop animation
                pauseScreen.style.display = 'flex'; // Show pause screen
                pauseButton.textContent = '▶️'; // Change to play emoji
                renderBoostOptions(); // Render available boosts when pausing
            } else {
                pauseScreen.style.display = 'none'; // Hide pause screen
                lastPipeSpawnTime = Date.now(); // Reset pipe spawn timer on resume
                gameLoop(); // Resume animation
                pauseButton.textContent = '⏸️'; // Change back to pause emoji
                boostOptionsDiv.innerHTML = ''; // Clear boost options when unpausing
            }
        }

        // Render boost options on the pause screen
        function renderBoostOptions() {
            boostOptionsDiv.innerHTML = ''; // Clear existing buttons
            if (purchasedBoosts.length > 0) {
                const boostHeader = document.createElement('h3');
                boostHeader.textContent = 'Available Boosts:';
                boostHeader.style.color = '#FFF';
                boostHeader.style.fontSize = '0.8em';
                boostOptionsDiv.appendChild(boostHeader);

                purchasedBoosts.forEach((boost, index) => {
                    const boostButton = document.createElement('button');
                    boostButton.textContent = `Activate ${boost.name}`;
                    boostButton.addEventListener('click', () => {
                        activateBoost(boost);
                    });
                    boostOptionsDiv.appendChild(boostButton);
                });
            } else {
                const noBoostsMessage = document.createElement('p');
                noBoostsMessage.textContent = 'No boosts available.';
                noBoostsMessage.style.color = '#AAA';
                noBoostsMessage.style.fontSize = '0.6em';
                boostOptionsDiv.appendChild(noBoostsMessage);
            }
        }

        // Update coin display
        function updateCoinDisplay() {
            coinCountDisplay.textContent = coinsCollected;
        }

        // Update revive display
        function updateReviveDisplay() {
            reviveCountDisplay.textContent = reviveCount;
            if (reviveCount > 0) {
                reviveIndicator.style.display = 'flex'; // Show if count > 0
            } else {
                reviveIndicator.style.display = 'none'; // Hide if count is 0
            }
        }

        // Update gem display
        function updateGemDisplay() {
            gemCountDisplayElement.textContent = gemsCollected;
            if (gemsCollected > 0) {
                gemIndicator.style.display = 'flex'; // Show if count > 0
            } else {
                gemIndicator.style.display = 'none'; // Hide if count is 0
            }
        }

        // --- Shop Functions ---
        function openShop() {
            // Determine which screen was active before opening shop
            if (startScreen.style.display === 'flex') {
                previousScreen = startScreen;
            } else if (gameOverScreen.style.display === 'flex') {
                previousScreen = gameOverScreen;
            } else {
                previousScreen = null; // Should not happen during game, but fallback
            }

            // Hide all other screens
            startScreen.style.display = 'none';
            gameOverScreen.style.display = 'none';
            pauseScreen.style.display = 'none';
            pauseButton.style.display = 'none'; // Hide game buttons
            shopButton.style.display = 'none'; // Hide start screen shop button
            shopButtonGameOver.style.display = 'none'; // Hide game over shop button

            mainGameCanvas.style.display = 'none'; // Explicitly hide the main game canvas

            shopScreen.style.display = 'flex';
            showShopMenu('skins'); // Default to skins tab
            updateCurrencyDisplaysInShop(); // Ensure currency is updated
        }

        function closeShop() {
            shopScreen.style.display = 'none';
            mainGameCanvas.style.display = 'block'; // Show the main game canvas again

            if (previousScreen) {
                previousScreen.style.display = 'flex'; // Show previous screen
                // Re-show shop button on start/game over screens
                if (previousScreen === startScreen) {
                    shopButton.style.display = 'block';
                } else if (previousScreen === gameOverScreen) {
                    shopButtonGameOver.style.display = 'block';
                }
            } else {
                // Fallback if previousScreen wasn't set (e.g., direct shop open)
                startScreen.style.display = 'flex';
                shopButton.style.display = 'block';
            }
        }

        function updateCurrencyDisplaysInShop() {
            updateCoinDisplay();
            updateReviveDisplay();
            updateGemDisplay();
        }

        function showShopMenu(menuType) {
            // Deactivate all tabs
            document.querySelectorAll('.shop-tab-button').forEach(button => {
                button.classList.remove('active');
            });
            // Activate the clicked tab
            document.querySelector(`.shop-tab-button[data-tab="${menuType}"]`).classList.add('active');

            // Clear current shop content
            shopContent.innerHTML = '';

            switch (menuType) {
                case 'skins':
                    renderSkinsMenu();
                    break;
                case 'boosts':
                    renderBoostsMenu();
                    break;
                case 'exchanges':
                    renderExchangesMenu();
                    break;
            }
        }

        function renderSkinsMenu() {
            const menuDiv = document.createElement('div');
            menuDiv.classList.add('shop-menu');

            for (const skinKey in SKINS_DATA) {
                const skin = SKINS_DATA[skinKey];
                const itemDiv = document.createElement('div');
                itemDiv.classList.add('shop-item');

                // Visual representation of the skin (a colored circle or custom drawing)
                const colorCircle = document.createElement('canvas'); // Use canvas for more complex previews
                colorCircle.width = 40;
                colorCircle.height = 40;
                const pCtx = colorCircle.getContext('2d');
                const previewRadius = 15; // Smaller radius for preview
                const previewX = colorCircle.width / 2;
                const previewY = colorCircle.height / 2;

                // Save the current canvas state before applying transformations/clips for preview
                pCtx.save();
                pCtx.beginPath();
                pCtx.arc(previewX, previewY, previewRadius, 0, Math.PI * 2);
                pCtx.clip(); // Clip future drawings to this circle

                if (skin.type === 'striped') {
                    // Newcastle: White background with central black vertical stripe
                    pCtx.fillStyle = 'white';
                    pCtx.fillRect(previewX - previewRadius, previewY - previewRadius, previewRadius * 2, previewRadius * 2); // Fill the entire circle area with white

                    // Draw central black stripe
                    const stripeWidth = previewRadius / 3;
                    pCtx.fillStyle = 'black';
                    pCtx.fillRect(previewX - stripeWidth / 2, previewY - previewRadius, stripeWidth, previewRadius * 2);

                } else if (skin.type === 'layered_circle') {
                    const outerRadius = previewRadius;
                    const innerRadius = previewRadius * 0.6; // Inner circle is smaller to show more border

                    // Outer circle
                    pCtx.beginPath();
                    pCtx.arc(previewX, previewY, outerRadius, 0, Math.PI * 2);
                    pCtx.fillStyle = skin.outerColor;
                    pCtx.fill();
                    pCtx.closePath();

                    // Inner circle
                    pCtx.beginPath();
                    pCtx.arc(previewX, previewY, innerRadius, 0, Math.PI * 2);
                    pCtx.fillStyle = skin.innerColor;
                    pCtx.fill();
                    pCtx.closePath();

                    // Add text to the inner circle
                    if (skin.text) {
                        pCtx.fillStyle = 'white'; // Text color for contrast
                        pCtx.font = `bold ${innerRadius * 1.2}px Arial`; // Adjust font size based on inner circle
                        pCtx.textAlign = 'center';
                        pCtx.textBaseline = 'middle';
                        pCtx.fillText(skin.text, previewX, previewY);
                    }
                } else {
                    // Default single color circle
                    pCtx.beginPath();
                    pCtx.arc(previewX, previewY, previewRadius, 0, Math.PI * 2);
                    pCtx.fillStyle = skin.color;
                    pCtx.fill();
                    pCtx.closePath();
                }

                // Restore the canvas state, removing the clip
                pCtx.restore();

                // Always draw the black outline on the preview (outside the clip, so it's a full circle)
                pCtx.beginPath();
                pCtx.arc(previewX, previewY, previewRadius, 0, Math.PI * 2);
                pCtx.strokeStyle = 'black';
                pCtx.lineWidth = 1; // Thinner for preview
                pCtx.stroke();
                pCtx.closePath();

                colorCircle.style.marginBottom = '10px';
                itemDiv.appendChild(colorCircle);

                const nameH3 = document.createElement('h3');
                nameH3.textContent = skin.name;
                itemDiv.appendChild(nameH3);

                const button = document.createElement('button');
                const isOwned = ownedSkins.includes(skinKey);
                const isEquipped = equippedSkin === skinKey;

                if (isEquipped) {
                    button.textContent = 'Equipped';
                    button.classList.add('equipped');
                    button.disabled = true;
                } else if (isOwned) {
                    button.textContent = 'Equip';
                    button.classList.add('owned');
                    button.addEventListener('click', async () => {
                        equippedSkin = skinKey;
                        await saveUserData();
                        showShopMenu('skins'); // Re-render to update equipped state
                    });
                } else {
                    const priceSpan = document.createElement('span');
                    priceSpan.classList.add('price');
                    priceSpan.textContent = `${skin.cost} $`;
                    itemDiv.appendChild(priceSpan);

                    button.textContent = 'Buy';
                    button.addEventListener('click', async () => {
                        if (coinsCollected >= skin.cost) {
                            coinsCollected -= skin.cost;
                            ownedSkins.push(skinKey);
                            await saveUserData();
                            updateCoinDisplay();
                            showTemporaryMessage(`Purchased ${skin.name}!`);
                            showShopMenu('skins'); // Re-render to update owned state
                        } else {
                            showTemporaryMessage("Insufficient Coins!");
                        }
                    });
                }
                itemDiv.appendChild(button);
                menuDiv.appendChild(itemDiv);
            }
            shopContent.appendChild(menuDiv);
        }

        function renderBoostsMenu() {
            const menuDiv = document.createElement('div');
            menuDiv.classList.add('shop-menu');

            for (const boostKey in BOOSTS_DATA) {
                const boost = BOOSTS_DATA[boostKey];
                const itemDiv = document.createElement('div');
                itemDiv.classList.add('shop-item');

                const nameH3 = document.createElement('h3');
                nameH3.textContent = boost.name;
                itemDiv.appendChild(nameH3);

                const priceSpan = document.createElement('span');
                priceSpan.classList.add('price');
                priceSpan.textContent = `${boost.cost} $`;
                itemDiv.appendChild(priceSpan);

                const button = document.createElement('button');
                button.textContent = 'Buy';
                button.addEventListener('click', async () => {
                    if (coinsCollected >= boost.cost) {
                        coinsCollected -= boost.cost;
                        purchasedBoosts.push({ type: boostKey, name: boost.name, value: boost.value }); // Store boost for later activation
                        await saveUserData(); // Save updated coins and purchased boosts
                        updateCoinDisplay();
                        showTemporaryMessage(`Purchased ${boost.name}! Check Pause Menu to activate.`);
                    } else {
                        showTemporaryMessage("Insufficient Coins!");
                    }
                });
                itemDiv.appendChild(button);
                menuDiv.appendChild(itemDiv);
            }
            shopContent.appendChild(menuDiv);
        }

        function renderExchangesMenu() {
            const menuDiv = document.createElement('div');
            menuDiv.classList.add('shop-menu');

            // 1 Gem for 75$ (Updated price)
            let itemDiv1 = document.createElement('div');
            itemDiv1.classList.add('shop-item');
            itemDiv1.innerHTML = `<h3>1 💎 for 75 $</h3><p>Exchange your gems for coins.</p><button>Exchange</button>`;
            itemDiv1.querySelector('button').addEventListener('click', async () => {
                if (gemsCollected >= 1) {
                    gemsCollected -= 1;
                    coinsCollected += 75; // Updated value
                    await saveUserData();
                    updateGemDisplay();
                    updateCoinDisplay();
                    showTemporaryMessage("Exchanged 1 💎 for 75 $!");
                } else {
                    showTemporaryMessage("Insufficient Gems!");
                }
            });
            menuDiv.appendChild(itemDiv1);

            // 1 Revive for 50$ (Updated price)
            let itemDiv2 = document.createElement('div');
            itemDiv2.classList.add('shop-item');
            itemDiv2.innerHTML = `<h3>1 ❤️ for 50 $</h3><p>Exchange your revives for coins.</p><button>Exchange</button>`;
            itemDiv2.querySelector('button').addEventListener('click', async () => {
                if (reviveCount >= 1) {
                    reviveCount -= 1;
                    coinsCollected += 50; // Updated value
                    await saveUserData();
                    updateReviveDisplay();
                    updateCoinDisplay();
                    showTemporaryMessage("Exchanged 1 ❤️ for 50 $!");
                } else {
                    showTemporaryMessage("Insufficient Revives!");
                }
            });
            menuDiv.appendChild(itemDiv2);

            // 100$ for 1 Gem (remains same)
            let itemDiv3 = document.createElement('div');
            itemDiv3.classList.add('shop-item');
            itemDiv3.innerHTML = `<h3>100 $ for 1 💎</h3><p>Exchange your coins for gems.</p><button>Exchange</button>`;
            itemDiv3.querySelector('button').addEventListener('click', async () => {
                if (coinsCollected >= 100) {
                    coinsCollected -= 100;
                    gemsCollected += 1;
                    await saveUserData();
                    updateCoinDisplay();
                    updateGemDisplay();
                    showTemporaryMessage("Exchanged 100 $ for 1 💎!");
                } else {
                    showTemporaryMessage("Insufficient Coins!");
                }
            });
            menuDiv.appendChild(itemDiv3);

            // 75$ for 1 Revive (remains same)
            let itemDiv4 = document.createElement('div');
            itemDiv4.classList.add('shop-item');
            itemDiv4.innerHTML = `<h3>75 $ for 1 ❤️</h3><p>Exchange your coins for revives.</p><button>Exchange</button>`;
            itemDiv4.querySelector('button').addEventListener('click', async () => {
                if (coinsCollected >= 75) {
                    coinsCollected -= 75;
                    reviveCount += 1;
                    await saveUserData();
                    updateCoinDisplay();
                    updateReviveDisplay();
                    showTemporaryMessage("Exchanged 75 $ for 1 ❤️!");
                } else {
                    showTemporaryMessage("Insufficient Coins!");
                }
            });
            menuDiv.appendChild(itemDiv4);

            shopContent.appendChild(menuDiv);
        }


        // --- Event Listeners ---

        // Make bird flap on click/tap
        mainGameCanvas.addEventListener('click', () => {
            if (gameRunning && !isPaused) {
                bird.velocity = bird.lift;
            } else if (isPaused) { // If paused, clicking mainGameCanvas unpauses
                togglePause();
            }
        });

        pauseButton.addEventListener('click', togglePause);
        shopButton.addEventListener('click', openShop);
        shopButtonGameOver.addEventListener('click', openShop); // Add listener for game over shop button
        closeShopButton.addEventListener('click', closeShop);

        // Add event listeners for shop tabs
        shopNav.querySelectorAll('.shop-tab-button').forEach(button => {
            button.addEventListener('click', () => {
                showShopMenu(button.dataset.tab);
            });
        });


        // Generate level buttons dynamically
        function createLevelButtons() {
            levelSelectionDiv.innerHTML = ''; // Clear existing buttons
            const maxUnlockedLevel = Math.max(...unlockedLevels);
            const nextPurchasableLevel = maxUnlockedLevel + 1;

            for (let i = 1; i <= NUM_LEVELS; i++) {
                const button = document.createElement('button');
                button.classList.add('levelButton');
                button.dataset.level = i;

                const isUnlocked = unlockedLevels.includes(i);
                const cost = getLevelUnlockCost(i);

                if (isUnlocked) {
                    button.textContent = LEVEL_CONFIGS[i].name;
                    button.addEventListener('click', () => {
                        startGame(i);
                    });
                } else if (i === nextPurchasableLevel && i <= NUM_LEVELS) { // This is the next level to unlock and within bounds
                    button.textContent = `${LEVEL_CONFIGS[i].name} (Buy: ${cost} 💎)`;
                    button.classList.add('locked'); // Still "locked" visually, but purchasable
                    button.addEventListener('click', async () => {
                        console.log(`Attempting to buy Level ${i}. Current gems: ${gemsCollected}, Cost: ${cost}`);
                        if (gemsCollected >= cost) {
                            gemsCollected -= cost;
                            unlockedLevels.push(i);
                            unlockedLevels.sort((a, b) => a - b); // Keep sorted
                            updateGemDisplay();
                            await saveUserData(); // Save updated gems and unlocked levels
                            showTemporaryMessage(`Level ${i} Unlocked!`);
                            console.log(`Level ${i} unlocked. Gems remaining: ${gemsCollected}`);
                            createLevelButtons(); // Re-render buttons to show unlocked state
                            // Optionally, start the game immediately after unlocking
                            // startGame(i);
                        } else {
                            showTemporaryMessage("Insufficient Gems!");
                            console.log(`Insufficient gems to buy Level ${i}. Needed: ${cost}, Have: ${gemsCollected}`);
                        }
                    });
                } else { // All other levels are unpurchasable and locked
                    button.textContent = `${LEVEL_CONFIGS[i].name} 🔒`; // Add a lock emoji
                    button.classList.add('locked', 'unpurchasable-locked');
                    // Add a click listener that just shows "Locked" message
                    button.addEventListener('click', () => {
                        showTemporaryMessage("This level is locked. Unlock previous levels first!");
                    });
                }
                levelSelectionDiv.appendChild(button);
            }
        }

        restartButton.addEventListener('click', () => {
            // Restart with the last chosen level, keeping coins/revives/gems
            resetGameSession(); // Reset core game state
            startGame(selectedLevel); // Use the stored selected level
        });

        selectLevelButton.addEventListener('click', () => {
            // When choosing a new level, currencies are NOT reset.
            updateReviveDisplay(); // Ensure display is updated with current persistent value
            updateCoinDisplay(); // Ensure display is updated with current persistent value
            updateGemDisplay(); // Ensure gem display is updated
            resetGameSession(); // Reset core game state (score, pipes, collectibles)
            startScreen.style.display = 'flex'; // Show start screen to choose level
            createLevelButtons(); // Re-render buttons in case a level was unlocked during gameplay
            shopButton.style.display = 'block'; // Show shop button on start screen
            shopButtonGameOver.style.display = 'none'; // Hide game over shop button
        });

        reviveButton.addEventListener('click', () => {
            if (reviveCount > 0) {
                reviveCount--; // Consume one revive
                updateReviveDisplay(); // Update display
                saveUserData(); // Save updated revive count
                gameOverScreen.style.display = 'none'; // Hide game over screen
                resetBirdPositionAndPhysics(); // Reset bird position
                gameRunning = true; // Set game back to running
                lastPipeSpawnTime = Date.now(); // Reset pipe spawn timer on revive
                pauseButton.style.display = 'block'; // Show pause button again
                shopButton.style.display = 'none'; // Hide start screen shop button
                shopButtonGameOver.style.display = 'none'; // Hide game over shop button
                gameLoop(); // Resume game loop
                console.log("Revived! Remaining revives:", reviveCount);
            } else {
                // This case should ideally not happen if button is hidden when count is 0
                console.warn("Attempted to revive with no revives left.");
            }
        });


        // --- Main Game Loop ---

        function gameLoop() {
            if (!gameRunning || isPaused) {
                return; // Stop the loop if game is not running or paused
            }

            // Clear the mainGameCanvas
            ctx.clearRect(0, 0, mainGameCanvas.width, mainGameCanvas.height);

            // Update and Draw Buildings (background)
            updateBuildings();
            drawBuildings();

            // Update game elements
            updateBird();
            spawnPipes();
            updatePipes();
            updateCollectibles(); // Update collectible positions
            checkCollision();
            checkCollectibleCollision(); // Check for collectible collisions

            // Draw game elements (pipes, collectibles, and bird are drawn on top of buildings)
            for (const p of pipes) {
                drawPipe(p.x, p.y, p.height);
            }
            drawCollectibles(); // Draw collectibles
            drawBird();

            // Request next animation frame
            animationFrameId = requestAnimationFrame(gameLoop);
        }

        // Initial setup on window load
        window.onload = async function () {
            console.log("Window loaded. Showing loading screen.");
            loadingScreen.style.display = 'flex'; // Show loading screen

            // Set mainGameCanvas dimensions based on game-container
            const container = document.getElementById('game-container');
            mainGameCanvas.width = container.offsetWidth;
            mainGameCanvas.height = container.offsetHeight;

            initializeBirdPosition(); // Set bird's initial Y based on mainGameCanvas height

            let firebaseInitialized = false;
            try {
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
                const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');

                app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                // Authenticate user
                if (typeof __initial_auth_token !== 'undefined') {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }

                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        console.log("User authenticated:", userId);
                        await loadUserData(); // Load all user data after authentication
                        isAuthReady = true;
                        // Update displays AFTER data is loaded
                        updateReviveDisplay();
                        updateCoinDisplay();
                        updateGemDisplay();
                        createLevelButtons(); // Create level buttons after loading unlocked levels
                    } else {
                        console.log("No user signed in.");
                        // If no user, still create buttons, but they won't save progress
                        createLevelButtons();
                    }
                    // Ensure screens are managed regardless of auth state
                    if (!firebaseInitialized) { // Only manage screens once after initial auth check
                        loadingScreen.style.display = 'none'; // Hide loading screen
                        startScreen.style.display = 'flex'; // Show start screen
                        shopButton.style.display = 'block'; // Show shop button on start screen
                        firebaseInitialized = true;
                    }
                });
            } catch (e) {
                console.error("Failed to initialize Firebase or authenticate:", e);
                // Ensure screens are managed even on Firebase error
                if (!firebaseInitialized) {
                    loadingScreen.style.display = 'none'; // Hide loading screen
                    startScreen.style.display = 'flex'; // Show start screen
                    shopButton.style.display = 'block'; // Show shop button on start screen
                    firebaseInitialized = true;
                }
            }
        };

        // Re-initialize bird position and mainGameCanvas size on window resize
        window.addEventListener('resize', () => {
            const container = document.getElementById('game-container');
            mainGameCanvas.width = container.offsetWidth;
            mainGameCanvas.height = container.offsetHeight;
            initializeBirdPosition();
            if (gameRunning) {
                // Re-initialize buildings to fit new mainGameCanvas size without gaps
                initializeBuildings();
            }
        });

    </script>
</body>
</html>
