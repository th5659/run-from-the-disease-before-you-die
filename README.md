# run-from-the-disease-before-you-die
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Run from the Plague!</title>
  <style>
    html, body {
      margin: 0; padding: 0; overflow: hidden;
      background: #f0e6d2;
      font-family: 'Georgia', serif;
    }
    canvas {
      display: block;
      margin: 0 auto;
      background: linear-gradient(to top, #c2bb9b, #f0e6d2);
      touch-action: none;
    }
    #score, #health {
      position: fixed;
      top: 10px;
      left: 50%;
      transform: translateX(-50%);
      font-size: 20px;
      font-weight: bold;
      text-shadow: 1px 1px #fff;
      color: #333;
    }
    #health {
      top: 40px;
    }
  </style>
</head>
<body>

<div id="score">Score: 0</div>
<div id="health">Health: 100</div>
<canvas id="gameCanvas" width="400" height="600"></canvas>

<script>
(() => {
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');

  const lanesX = [80, 160, 240];
  const groundY = 500;

  let playerLane = 1;
  let playerY = groundY;
  let isJumping = false;
  let jumpVelocity = 0;
  const gravity = 0.6;

  let health = 100;
  let score = 0;
  let frameCount = 0;
  let gameSpeed = 5;

  const medicines = [];
  const plagueFogs = [];

  function drawPlayer(x, y) {
    ctx.fillStyle = '#d9c490'; // skin
    ctx.beginPath();
    ctx.arc(x, y - 40, 12, 0, Math.PI * 2);
    ctx.fill();
    ctx.fillStyle = '#4a3e31'; // toga
    ctx.fillRect(x - 10, y - 30, 20, 40);
    ctx.strokeStyle = '#00aa00'; // laurel
    ctx.beginPath();
    ctx.arc(x, y - 40, 14, 0, Math.PI * 2);
    ctx.stroke();
  }

  function drawMedicine(x, y) {
    ctx.fillStyle = '#5bc0de';
    ctx.fillRect(x - 6, y - 20, 12, 20);
    ctx.fillStyle = '#fff';
    ctx.fillRect(x - 3, y - 12, 6, 8);
  }

  function drawPlague(x, y) {
    const gradient = ctx.createRadialGradient(x, y, 10, x, y, 40);
    gradient.addColorStop(0, 'rgba(139,0,0,0.8)');
    gradient.addColorStop(1, 'rgba(139,0,0,0)');
    ctx.fillStyle = gradient;
    ctx.beginPath();
    ctx.arc(x, y, 40, 0, Math.PI * 2);
    ctx.fill();

    ctx.fillStyle = 'yellow';
    ctx.beginPath();
    ctx.arc(x - 10, y - 10, 5, 0, Math.PI * 2);
    ctx.arc(x + 10, y - 10, 5, 0, Math.PI * 2);
    ctx.fill();
  }

  function resetGame() {
    playerLane = 1;
    playerY = groundY;
    isJumping = false;
    jumpVelocity = 0;
    health = 100;
    score = 0;
    frameCount = 0;
    gameSpeed = 5;
    medicines.length = 0;
    plagueFogs.length = 0;
    document.getElementById('score').textContent = 'Score: 0';
    document.getElementById('health').textContent = 'Health: 100';
  }

  function moveLeft() {
    if (playerLane > 0) playerLane--;
  }

  function moveRight() {
    if (playerLane < lanesX.length - 1) playerLane++;
  }

  function jump() {
    if (!isJumping) {
      isJumping = true;
      jumpVelocity = -12;
    }
  }

  function gameOver() {
    alert('Game Over! Your score: ' + score);
    resetGame();
  }

  function update() {
    frameCount++;
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (isJumping) {
      playerY += jumpVelocity;
      jumpVelocity += gravity;
      if (playerY > groundY) {
        playerY = groundY;
        isJumping = false;
      }
    }

    // Reduce health over time
    if (frameCount % 60 === 0) {
      health -= 1;
      document.getElementById('health').textContent = 'Health: ' + health;
      if (health <= 0) {
        gameOver();
        return;
      }
    }

    if (frameCount % 80 === 0) {
      medicines.push({ lane: Math.floor(Math.random() * 3), y: -30 });
    }

    if (frameCount % 150 === 0) {
      plagueFogs.push({ lane: Math.floor(Math.random() * 3), y: -50, speed: gameSpeed });
    }

    medicines.forEach((med, i) => {
      med.y += gameSpeed;
      drawMedicine(lanesX[med.lane], med.y);

      if (
        med.lane === playerLane &&
        med.y > playerY - 40 &&
        med.y < playerY + 10
      ) {
        medicines.splice(i, 1);
        score++;
        health = Math.min(health + 10, 100);
        gameSpeed += 0.1;
        document.getElementById('score').textContent = 'Score: ' + score;
        document.getElementById('health').textContent = 'Health: ' + health;
      }
    });

    plagueFogs.forEach((fog, i) => {
      fog.y += fog.speed;
      drawPlague(lanesX[fog.lane], fog.y);

      if (
        fog.lane === playerLane &&
        fog.y > playerY - 50 &&
        fog.y < playerY + 10 &&
        !isJumping
      ) {
        health -= 20;
        document.getElementById('health').textContent = 'Health: ' + health;
        plagueFogs.splice(i, 1);
        if (health <= 0) {
          gameOver();
          return;
        }
      }

      if (fog.y > canvas.height + 50) {
        plagueFogs.splice(i, 1);
      }
    });

    drawPlayer(lanesX[playerLane], playerY);
    requestAnimationFrame(update);
  }

  // Controls: keyboard + swipe
  window.addEventListener('keydown', (e) => {
    if (e.key === 'ArrowLeft') moveLeft();
    if (e.key === 'ArrowRight') moveRight();
    if (e.key === 'ArrowUp') jump();
  });

  let touchStartX = null;
  let touchStartY = null;

  canvas.addEventListener('touchstart', e => {
    touchStartX = e.touches[0].clientX;
    touchStartY = e.touches[0].clientY;
  });

  canvas.addEventListener('touchend', e => {
    if (touchStartX === null || touchStartY === null) return;
    const dx = e.changedTouches[0].clientX - touchStartX;
    const dy = e.changedTouches[0].clientY - touchStartY;

    if (Math.abs(dx) > Math.abs(dy)) {
      if (dx > 30) moveRight();
      else if (dx < -30) moveLeft();
    } else {
      if (dy < -30) jump();
    }

    touchStartX = null;
    touchStartY = null;
  });

  resetGame();
  update();
})();
</script>
</body>
</html>
