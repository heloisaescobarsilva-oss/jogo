<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Blocky Blast</title>
  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #111;
      color: white;
      text-align: center;
    }
    #loginScreen, #gameScreen {
      width: 100vw;
      height: 100vh;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
    }
    input {
      padding: 12px;
      margin: 10px;
      width: 280px;
      font-size: 18px;
      border-radius: 8px;
      border: none;
    }
    button {
      padding: 12px 30px;
      font-size: 18px;
      margin: 10px;
      border: none;
      border-radius: 8px;
      background: #ff3366;
      color: white;
      cursor: pointer;
    }
    button:hover {
      background: #ff5588;
    }
    canvas {
      border: 3px solid #fff;
      border-radius: 10px;
      box-shadow: 0 0 20px rgba(255, 255, 255, 0.3);
    }
    #score {
      font-size: 24px;
      margin-bottom: 10px;
    }
  </style>
</head>
<body>

  <!-- Tela de Login -->
  <div id="loginScreen">
    <h1>🔥 Blocky Blast</h1>
    <h2>Faça login para jogar</h2>
    <input type="text" id="username" placeholder="Nome de usuário" autocomplete="off"><br>
    <input type="password" id="password" placeholder="Senha (qualquer uma)"><br>
    <button onclick="login()">Entrar no Jogo</button>
    <p style="margin-top: 20px; font-size: 14px;">Dica: A senha pode ser qualquer coisa</p>
  </div>

  <!-- Tela do Jogo -->
  <div id="gameScreen" style="display: none;">
    <div id="score">Pontos: <span id="points">0</span> | Vidas: <span id="lives">3</span></div>
    <canvas id="gameCanvas" width="480" height="600"></canvas>
    <button onclick="logout()" style="margin-top: 15px; background: #666;">Sair</button>
  </div>

  <script>
    // ==================== LOGIN ====================
    function login() {
      const username = document.getElementById("username").value.trim();
      
      if (username === "") {
        alert("Por favor, digite um nome de usuário!");
        return;
      }

      // Salva o jogador
      localStorage.setItem("blockyPlayer", username);

      // Esconde login e mostra jogo
      document.getElementById("loginScreen").style.display = "none";
      document.getElementById("gameScreen").style.display = "flex";

      initGame();
    }

    function logout() {
      if (confirm("Deseja realmente sair?")) {
        document.getElementById("gameScreen").style.display = "none";
        document.getElementById("loginScreen").style.display = "flex";
        document.getElementById("username").value = "";
      }
    }

    // ==================== JOGO ====================
    let canvas, ctx, playerName;
    let paddleX, ballX, ballY, ballDX, ballDY;
    let ballRadius = 8;
    let paddleHeight = 12;
    let paddleWidth = 90;
    let score = 0;
    let lives = 3;

    let bricks = [];
    const brickRowCount = 6;
    const brickColumnCount = 8;
    const brickWidth = 50;
    const brickHeight = 20;
    const brickPadding = 8;
    const brickOffsetTop = 60;
    const brickOffsetLeft = 35;

    function initGame() {
      canvas = document.getElementById("gameCanvas");
      ctx = canvas.getContext("2d");

      playerName = localStorage.getItem("blockyPlayer") || "Jogador";

      // Posições iniciais
      paddleX = (canvas.width - paddleWidth) / 2;
      ballX = canvas.width / 2;
      ballY = canvas.height - 80;
      ballDX = 4;
      ballDY = -4;

      score = 0;
      lives = 3;
      updateScore();

      // Criar blocos
      createBricks();

      // Controles do mouse e teclado
      document.addEventListener("mousemove", mouseMoveHandler);
      document.addEventListener("keydown", keyDownHandler);
      document.addEventListener("keyup", keyUpHandler);

      requestAnimationFrame(gameLoop);
    }

    function createBricks() {
      bricks = [];
      for (let r = 0; r < brickRowCount; r++) {
        bricks[r] = [];
        for (let c = 0; c < brickColumnCount; c++) {
          bricks[r][c] = { x: 0, y: 0, status: 1 };
        }
      }
    }

    let rightPressed = false;
    let leftPressed = false;

    function keyDownHandler(e) {
      if (e.key === "Right" || e.key === "ArrowRight") rightPressed = true;
      if (e.key === "Left" || e.key === "ArrowLeft") leftPressed = true;
    }

    function keyUpHandler(e) {
      if (e.key === "Right" || e.key === "ArrowRight") rightPressed = false;
      if (e.key === "Left" || e.key === "ArrowLeft") leftPressed = false;
    }

    function mouseMoveHandler(e) {
      const relativeX = e.clientX - canvas.offsetLeft;
      if (relativeX > paddleWidth/2 && relativeX < canvas.width - paddleWidth/2) {
        paddleX = relativeX - paddleWidth/2;
      }
    }

    function updateScore() {
      document.getElementById("points").textContent = score;
      document.getElementById("lives").textContent = lives;
    }

    function drawPaddle() {
      ctx.fillStyle = "#00ffcc";
      ctx.fillRect(paddleX, canvas.height - paddleHeight - 10, paddleWidth, paddleHeight);
    }

    function drawBall() {
      ctx.beginPath();
      ctx.arc(ballX, ballY, ballRadius, 0, Math.PI * 2);
      ctx.fillStyle = "#ffff00";
      ctx.fill();
      ctx.closePath();
    }

    function drawBricks() {
      for (let r = 0; r < brickRowCount; r++) {
        for (let c = 0; c < brickColumnCount; c++) {
          if (bricks[r][c].status === 1) {
            const brickX = c * (brickWidth + brickPadding) + brickOffsetLeft;
            const brickY = r * (brickHeight + brickPadding) + brickOffsetTop;
            
            bricks[r][c].x = brickX;
            bricks[r][c].y = brickY;

            ctx.fillStyle = `hsl(${r * 40}, 100%, 60%)`;
            ctx.fillRect(brickX, brickY, brickWidth, brickHeight);
          }
        }
      }
    }

    function collisionDetection() {
      for (let r = 0; r < brickRowCount; r++) {
        for (let c = 0; c < brickColumnCount; c++) {
          const b = bricks[r][c];
          if (b.status === 1) {
            if (
              ballX + ballRadius > b.x &&
              ballX - ballRadius < b.x + brickWidth &&
              ballY + ballRadius > b.y &&
              ballY - ballRadius < b.y + brickHeight
            ) {
              ballDY = -ballDY;
              b.status = 0;
              score += 10;
              updateScore();

              // Verifica se ganhou
              if (score >= brickRowCount * brickColumnCount * 10) {
                alert("🎉 Parabéns " + playerName + "! Você venceu!");
                resetGame();
              }
            }
          }
        }
      }
    }

    function gameLoop() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      drawBricks();
      drawPaddle();
      drawBall();

      collisionDetection();

      // Movimento da bola
      ballX += ballDX;
      ballY += ballDY;

      // Colisão com paredes
      if (ballX + ballRadius > canvas.width || ballX - ballRadius < 0) {
        ballDX = -ballDX;
      }
      if (ballY - ballRadius < 0) {
        ballDY = -ballDY;
      }

      // Colisão com paddle
      if (
        ballY + ballRadius > canvas.height - paddleHeight - 10 &&
        ballX > paddleX &&
        ballX < paddleX + paddleWidth
      ) {
        ballDY = -ballDY * 1.02; // aumenta um pouco a velocidade
      }

      // Caiu para baixo (perde vida)
      if (ballY - ballRadius > canvas.height) {
        lives--;
        updateScore();
        if (lives <= 0) {
          alert("Game Over! Pontuação final: " + score);
          resetGame();
          return;
        } else {
          // Reset bola
          ballX = canvas.width / 2;
          ballY = canvas.height - 80;
          ballDX = 4;
          ballDY = -4;
        }
      }

      // Movimento do paddle com setas
      if (rightPressed && paddleX < canvas.width - paddleWidth) {
        paddleX += 8;
      }
      if (leftPressed && paddleX > 0) {
        paddleX -= 8;
      }

      requestAnimationFrame(gameLoop);
    }

    function resetGame() {
      document.getElementById("gameScreen").style.display = "none";
      document.getElementById("loginScreen").style.display = "flex";
    }

    // Iniciar com a tela de login
    console.log("%cBlocky Blast carregado! Faça login para jogar.", "color: #ff3366; font-size: 16px");
  </script>
</body>
</html>#
