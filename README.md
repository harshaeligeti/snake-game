<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Snake Game</title>
  <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
</head>
<body>
  <div id="score">Score: 0</div>
  <canvas id="gameCanvas" width="400" height="400"></canvas>
  <div id="gameOver">Game Over</div>
  <button id="restartButton" onclick="startGame()">Restart</button>

  <style>
    body {
      background-color: #000;
      margin: 0;
      overflow: hidden;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      font-family: 'Press Start 2P', cursive;
    }

    #score {
      font-size: 24px;
      color: #fff;
      margin-bottom: 10px;
    }

    #gameCanvas {
      background-color: #000;
      border: 20px solid #6df8ff; /* Snake green color */
      image-rendering: pixelated;
    }

    #gameOver {
      display: none;
      font-size: 24px;
      color: #ff0000; /* Red color */
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      animation: blink 1s ease-in-out infinite;
      text-align: center;
    }

    #restartButton {
      font-size: 16px;
      padding: 10px;
      margin-top: 20px;
      background-color: #3498db; /* Blue color */
      color: #fff;
      border: 2px solid #fff; /* White border */
      cursor: pointer;
      outline: none;
      display: none;
    }

    @keyframes blink {
      0%, 100% { opacity: 0; }
      50% { opacity: 1; }
    }
  </style>

  <script>
    const GAME_SPEED = 100;
    const CANVAS_BORDER_COLOUR = '#2ecc71'; /* Snake green color */
    const CANVAS_BACKGROUND_COLOUR = "white";
    const SNAKE_COLOUR = 'lightgreen';
    const SNAKE_BORDER_COLOUR = 'darkgreen';
    const FOOD_COLOUR = 'red';
    const FOOD_BORDER_COLOUR = 'darkred';
    let gameOverElement = document.getElementById('gameOver');
    let restartButton = document.getElementById('restartButton');
    let gameOver = false;

    let snake = [
      { x: 150, y: 150 },
      { x: 140, y: 150 },
      { x: 130, y: 150 },
      { x: 120, y: 150 },
      { x: 110, y: 150 }
    ]

    let score = 0;
    let changingDirection = false;
    let dx = 10;
    let dy = 0;
    let foodX;
    let foodY;
    let foodEaten = false;

    const gameCanvas = document.getElementById("gameCanvas");
    const ctx = gameCanvas.getContext("2d");

    main();
    createFood();
    document.addEventListener("keydown", changeDirection);

    function main() {
      if (didGameEnd() || gameOver) return;

      setTimeout(function onTick() {
        changingDirection = false;
        clearCanvas();
        drawFood();
        advanceSnake();
        drawSnake();
        main();
      }, GAME_SPEED);
    }

    function clearCanvas() {
      ctx.fillStyle = CANVAS_BACKGROUND_COLOUR;
      ctx.strokestyle = CANVAS_BORDER_COLOUR;
      ctx.fillRect(0, 0, gameCanvas.width, gameCanvas.height);
      ctx.strokeRect(0, 0, gameCanvas.width, gameCanvas.height);
    }

    function drawFood() {
      if (!foodEaten) {
        ctx.fillStyle = FOOD_COLOUR;
        ctx.strokestyle = FOOD_BORDER_COLOUR;
        ctx.fillRect(foodX, foodY, 10, 10);
        ctx.strokeRect(foodX, foodY, 10, 10);
      }
    }

    function advanceSnake() {
      const head = { x: snake[0].x + dx, y: snake[0].y + dy };
      snake.unshift(head);

      const didEatFood = snake[0].x === foodX && snake[0].y === foodY;
      if (didEatFood) {
        score += 10;
        createFood();
        foodEaten = false;
      } else {
        snake.pop();
      }

      document.getElementById('score').innerHTML = 'Score: ' + score;
    }

    function didGameEnd() {
      for (let i = 4; i < snake.length; i++) {
        if (snake[i].x === snake[0].x && snake[i].y === snake[0].y) {
          gameOverElement.style.display = 'block';
          restartButton.style.display = 'block';
          return true;
        }
      }

      const hitLeftWall = snake[0].x < 0;
      const hitRightWall = snake[0].x > gameCanvas.width - 10;
      const hitTopWall = snake[0].y < 0;
      const hitBottomWall = snake[0].y > gameCanvas.height - 10;

      if (hitLeftWall || hitRightWall || hitTopWall || hitBottomWall) {
        gameOverElement.style.display = 'block';
        restartButton.style.display = 'block';
        gameOver = true;
        return true;
      }

      return false;
    }

    function randomTen(min, max) {
      return Math.round((Math.random() * (max - min) + min) / 10) * 10;
    }

    function createFood() {
      foodX = randomTen(0, gameCanvas.width - 10);
      foodY = randomTen(0, gameCanvas.height - 10);

      snake.forEach(function isFoodOnSnake(part) {
        const foodIsoNsnake = part.x == foodX && part.y == foodY;
        if (foodIsoNsnake) createFood();
      });
    }

    function drawSnake() {
      snake.forEach(drawSnakePart);
    }

    function drawSnakePart(snakePart) {
      ctx.fillStyle = SNAKE_COLOUR;
      ctx.strokestyle = SNAKE_BORDER_COLOUR;
      ctx.fillRect(snakePart.x, snakePart.y, 10, 10);
      ctx.strokeRect(snakePart.x, snakePart.y, 10, 10);
    }

    function changeDirection(event) {
      const LEFT_KEY = 37;
      const RIGHT_KEY = 39;
      const UP_KEY = 38;
      const DOWN_KEY = 40;

      if (changingDirection) return;
      changingDirection = true;
      
      const keyPressed = event.keyCode;

      const goingUp = dy === -10;
      const goingDown = dy === 10;
      const goingRight = dx === 10;
      const goingLeft = dx === -10;

      if (keyPressed === LEFT_KEY && !goingRight) {
        dx = -10;
        dy = 0;
      }
      
      if (keyPressed === UP_KEY && !goingDown) {
        dx = 0;
        dy = -10;
      }
      
      if (keyPressed === RIGHT_KEY && !goingLeft) {
        dx = 10;
        dy = 0;
      }
      
      if (keyPressed === DOWN_KEY && !goingUp) {
        dx = 0;
        dy = 10;
      }
    }

    function startGame() {
      gameOverElement.style.display = 'none';
      restartButton.style.display = 'none';

      score = 0;
      changingDirection = false;
      dx = 10;
      dy = 0;
      foodEaten = false;
      gameOver = false;

      snake = [
        { x: 150, y: 150 },
        { x: 140, y: 150 },
        { x: 130, y: 150 },
        { x: 120, y: 150 },
        { x: 110, y: 150 }
      ];

      gameCanvas.style.display = 'block';
      drawSnake();
      drawFood();
      main();
    }
  </script>
</body>
</html>
