<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Flappy Bird Clone</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    body {
      margin: 0;
      overflow: hidden;
      font-family: 'Arial', sans-serif;
      background: #70c5ce;
    }
    canvas {
      display: block;
      margin: auto;
      background: #70c5ce;
    }
    #ui {
      position: absolute;
      top: 20px;
      left: 0;
      width: 100%;
      text-align: center;
      z-index: 2;
    }
    .button {
      padding: 12px 28px;
      background: #ffcc00;
      color: #000;
      font-size: 20px;
      font-weight: bold;
      border: none;
      border-radius: 12px;
      cursor: pointer;
      margin: 10px;
      transition: transform 0.2s;
    }
    .button:hover {
      transform: scale(1.05);
    }
    #title {
      font-size: 48px;
      color: white;
      text-shadow: 2px 2px #444;
    }
    #highscore {
      font-size: 22px;
      color: white;
    }
    @media (max-width: 500px) {
      canvas {
        width: 100vw;
        height: 75vh;
      }
      .button {
        font-size: 18px;
        padding: 10px 20px;
      }
    }
  </style>
</head>
<body>
  <div id="ui">
    <div id="menu">
      <div id="title">Flappy Bird Clone</div>
      <div id="highscore">High Score: <span id="best">0</span></div>
      <button class="button" onclick="startGame()">Start</button>
    </div>
    <div id="restart" style="display:none;">
      <button class="button" onclick="resetGame()">Restart</button>
    </div>
  </div>
  <canvas id="gameCanvas" width="400" height="600"></canvas>

  <audio id="flapSound" src="https://actions.google.com/sounds/v1/cartoon/cartoon_boing.ogg"></audio>
  <audio id="hitSound" src="https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg"></audio>
  <audio id="scoreSound" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>
  <audio id="winSound" src="https://cdn.pixabay.com/audio/2022/10/22/audio_e0c70dfd0e.mp3"></audio>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    const flapSound = document.getElementById('flapSound');
    const hitSound = document.getElementById('hitSound');
    const scoreSound = document.getElementById('scoreSound');
    const winSound = document.getElementById('winSound');

    const gravity = 0.5;
    const bird = { x: 80, y: 200, w: 34, h: 24, velocity: 0 };
    let pipes = [];
    let frame = 0;
    let score = 0;
    let best = 0;
    let gameState = 'menu';

    const bg = "#70c5ce";
    const birdSprite = new Image();
    birdSprite.src = "https://opengameart.org/sites/default/files/bird_yellow_0.png";
    let birdImgLoaded = false;
    birdSprite.onload = () => birdImgLoaded = true;

    function drawBird() {
      if (birdImgLoaded) {
        ctx.drawImage(birdSprite, bird.x, bird.y, bird.w, bird.h);
      } else {
        ctx.fillStyle = "yellow";
        ctx.fillRect(bird.x, bird.y, bird.w, bird.h);
      }
    }

    function drawPipes() {
      pipes.forEach(pipe => {
        ctx.fillStyle = "#228B22";
        ctx.fillRect(pipe.x, 0, pipe.width, pipe.top);
        ctx.fillRect(pipe.x, pipe.bottom, pipe.width, canvas.height - pipe.bottom);
      });
    }

    function drawScore() {
      ctx.fillStyle = "white";
      ctx.font = "32px Arial";
      ctx.fillText(score, canvas.width / 2 - 10, 50);
    }

    function update() {
      if (gameState === 'play') {
        bird.velocity += gravity;
        bird.y += bird.velocity;

        if (frame % 90 === 0) {
          let top = Math.random() * 200 + 50;
          let gap = 130;
          pipes.push({
            x: canvas.width,
            width: 50,
            top: top,
            bottom: top + gap,
            scored: false
          });
        }

        pipes.forEach(pipe => {
          pipe.x -= 2;

          if (!pipe.scored && pipe.x + pipe.width < bird.x) {
            score++;
            pipe.scored = true;
            scoreSound.play();
            if (score % 5 === 0) winSound.play();
          }

          if (
            bird.x < pipe.x + pipe.width &&
            bird.x + bird.w > pipe.x &&
            (bird.y < pipe.top || bird.y + bird.h > pipe.bottom)
          ) {
            hitSound.play();
            endGame();
          }
        });

        pipes = pipes.filter(pipe => pipe.x + pipe.width > 0);

        if (bird.y + bird.h >= canvas.height) {
          hitSound.play();
          endGame();
        }
      }
    }

    function draw() {
      ctx.fillStyle = bg;
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      if (gameState === 'menu') {
        drawBird();
      } else if (gameState === 'play') {
        drawPipes();
        drawBird();
        drawScore();
      } else if (gameState === 'gameover') {
        drawPipes();
        drawBird();
        drawScore();
      }
    }

    function loop() {
      frame++;
      update();
      draw();
      requestAnimationFrame(loop);
    }

    function startGame() {
      document.getElementById("menu").style.display = "none";
      document.getElementById("restart").style.display = "none";
      gameState = "play";
      score = 0;
      bird.y = 200;
      bird.velocity = 0;
      pipes = [];
    }

    function endGame() {
      gameState = "gameover";
      if (score > best) {
        best = score;
        document.getElementById("best").textContent = best;
      }
      document.getElementById("restart").style.display = "block";
    }

    function resetGame() {
      document.getElementById("restart").style.display = "none";
      document.getElementById("menu").style.display = "block";
      gameState = "menu";
    }

    function handleFlap() {
      if (gameState === 'play') {
        bird.velocity = -8;
        flapSound.play();
      }
    }

    document.addEventListener('keydown', function (e) {
      if (e.code === "Space" || e.key === "ArrowUp") handleFlap();
    });

    document.addEventListener('mousedown', handleFlap);
    document.addEventListener('touchstart', handleFlap);

    loop();
  </script>
</body>
</html>
