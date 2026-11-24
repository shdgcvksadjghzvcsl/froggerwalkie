<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🐱 emoji cat crosser · slow & simple</title>
    <style>
        * {
            box-sizing: border-box;
            user-select: none;
        }
        body {
            margin: 0;
            min-height: 100vh;
            background: #1f3b1f;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Segoe UI', Roboto, system-ui, sans-serif;
            padding: 12px;
        }
        /* ultra-flat container – no nested borders, minimal chrome */
        .game-area {
            background: #1f3b1f;
            padding: 10px;
            border-radius: 20px;
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.5);
            max-width: 680px;
            width: 100%;
        }
        canvas {
            display: block;
            margin: 0 auto;
            border-radius: 16px;
            background: #2a6b2a;
            width: 100%;
            height: auto;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
            cursor: crosshair;
            touch-action: none;
        }
        /* single flat row for score + restart */
        .top-bar {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
            padding: 0 4px;
            color: #e9f5d0;
            font-weight: bold;
            font-size: 1.3rem;
            text-shadow: 2px 2px 0 #0a1f0a;
        }
        .score {
            background: #0d2b0d;
            padding: 4px 18px;
            border-radius: 40px;
            letter-spacing: 1px;
            box-shadow: inset 0 -2px 0 #4a7a4a;
        }
        .restart {
            background: #e7b42e;
            border: none;
            font-size: 1.3rem;
            padding: 4px 18px 6px;
            border-radius: 40px;
            font-weight: 800;
            color: #1e2a0e;
            box-shadow: 0 4px 0 #8f6f1a;
            cursor: pointer;
            font-family: inherit;
            display: flex;
            align-items: center;
            gap: 6px;
            transition: 0.05s linear;
            transform: translateY(-1px);
        }
        .restart:active {
            box-shadow: 0 1px 0 #8f6f1a;
            transform: translateY(3px);
        }
        .hint {
            text-align: center;
            margin-top: 8px;
            color: #b8d9a4;
            font-size: 0.95rem;
            font-weight: 500;
            letter-spacing: 0.5px;
        }
    </style>
</head>
<body>
<div class="game-area">
    <!-- single row – no nested divs beyond this -->
    <div class="top-bar">
        <span class="score">🏆 <span id="scoreDisplay">0</span></span>
        <button class="restart" id="restartButton">↻ RESTART</button>
    </div>
    <canvas id="gameCanvas" width="640" height="480"></canvas>
    <div class="hint">🐱 arrow keys / swipe · slow cars · reach 🏁</div>
</div>
<script>
    (function() {
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreSpan = document.getElementById('scoreDisplay');

        // ---------- grid ----------
        const COLS = 8;
        const ROWS = 9;
        const CELL_W = canvas.width / COLS;   // 80
        const CELL_H = canvas.height / ROWS;  // 53.333

        // ---------- state ----------
        let score = 0;
        let cat = { col: 3, row: 0 };  // bottom row 0

        // cars: smooth fractional column, slower speeds
        let cars = [];

        const carEmojis = ['🚗', '🚙', '🚕', '🚌', '🚚
