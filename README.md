<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Uçak Oyunu</title>
<style>
  body {
    margin: 0;
    overflow: hidden;
  }
  canvas {
    display: block;
    background-color: skyblue;
  }
  .menu {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    text-align: center;
    display: none;
  }
</style>
</head>
<body>
<canvas id="gameCanvas" width="800" height="600"></canvas>
<div class="menu">
  <h1>Oyun Bitti!</h1>
  <button id="restartButton">Tekrar Oyna</button>
  <div>
    <label for="difficultyRange">Zorluk Seviyesi: </label>
    <input type="range" id="difficultyRange" min="1" max="5" value="3">
    <span id="difficultyValue">3</span>
  </div>
  <div>
    <label for="cloudDensityRange">Bulut Miktarı: </label>
    <input type="range" id="cloudDensityRange" min="1" max="5" value="3">
    <span id="cloudDensityValue">3</span>
  </div>
</div>
<div id="score" style="position: absolute; top: 10px; left: 10px; font-size: 24px; color: white;">Skor: 0</div>
<script>
  const canvas = document.getElementById("gameCanvas");
  const ctx = canvas.getContext("2d");
  const planeWidths = [60, 80, 100, 120]; // Uçağın farklı genişlikleri
  const planeHeight = 20;
  let planeX = canvas.width / 2 - planeWidths[0] / 2;
  let planeY = canvas.height - planeHeight - 10;
  const planeSpeed = 5;
  const cloudRadius = 20;
  let clouds = [];
  let gameRunning = true;
  let score = 0;
  let difficultyLevel = 3;
  let cloudDensity = 3;
  let planeIndex = 0; // Uçağın genişliklerinin dizideki indeksi

  function drawPlane() {
    ctx.fillStyle = "black";
    ctx.beginPath();
    ctx.moveTo(planeX, planeY);
    ctx.lineTo(planeX + planeWidths[planeIndex] / 2, planeY - planeHeight);
    ctx.lineTo(planeX + planeWidths[planeIndex], planeY);
    ctx.closePath();
    ctx.fill();
  }

  function movePlane() {
    if (isPlaneMovingLeft && planeX > 0) {
      planeX -= planeSpeed;
    }
    if (isPlaneMovingRight && planeX < canvas.width - planeWidths[planeIndex]) {
      planeX += planeSpeed;
    }
    if (isPlaneMovingUp && planeY > 0) {
      planeY -= planeSpeed;
    }
    if (isPlaneMovingDown && planeY < canvas.height - planeHeight) {
      planeY += planeSpeed;
    }
  }

  function clearCanvas() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
  }

  function drawCloud(x, y) {
    ctx.fillStyle = "white";
    ctx.beginPath();
    ctx.arc(x, y, cloudRadius, 0, Math.PI * 2);
    ctx.fill();
  }

  function moveClouds() {
    for (let i = 0; i < clouds.length; i++) {
      clouds[i].y += difficultyLevel / 2;
      if (clouds[i].y > canvas.height) {
        clouds[i].y = 0;
        clouds[i].x = Math.random() * canvas.width;
        score++;
        document.getElementById("score").innerText = "Skor: " + score;
        checkPlaneUpgrade();
      }
      if (clouds[i].x > planeX && clouds[i].x < planeX + planeWidths[planeIndex] &&
          clouds[i].y > planeY && clouds[i].y < planeY + planeHeight) {
        endGame();
        return;
      }
    }
  }

  function drawClouds() {
    for (let i = 0; i < clouds.length; i++) {
      drawCloud(clouds[i].x, clouds[i].y);
    }
  }

  function restartGame() {
    gameRunning = true;
    score = 0;
    planeIndex = 0;
    planeX = canvas.width / 2 - planeWidths[planeIndex] / 2;
    planeY = canvas.height - planeHeight - 10;
    document.querySelector(".menu").style.display = "none";
    updateGame();
  }

  function endGame() {
    gameRunning = false;
    document.querySelector(".menu").style.display = "block";
  }

  function updateGame() {
    clearCanvas();
    drawClouds();
    drawPlane();
    movePlane();
    moveClouds();
    if (gameRunning) {
      requestAnimationFrame(updateGame);
    }
  }

  // Bulut oluşturma
  function createClouds() {
    clouds = [];
    const cloudCount = 10 * cloudDensity;
    for (let i = 0; i < cloudCount; i++) {
      clouds.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height
      });
    }
  }

  function updateCloudDensity() {
    cloudDensity = parseInt(document.getElementById("cloudDensityRange").value);
    document.getElementById("cloudDensityValue").textContent = cloudDensity;
    createClouds();
  }

  function checkPlaneUpgrade() {
    // Score sırayla 20, 50, 100 ve 200 olunca ve her 100 artışta uçağın şekli değişir
    if (score === 20 || score === 50 || score === 100 || score === 200 || (score > 200 && (score - 200) % 100 === 0)) {
      planeIndex++;
      if (planeIndex >= planeWidths.length) {
        planeIndex = planeWidths.length - 1;
      }
    }
  }

  createClouds();

  let isPlaneMovingLeft = false;
  let isPlaneMovingRight = false;
  let isPlaneMovingUp = false;
  let isPlaneMovingDown = false;

  document.addEventListener("keydown", function(event) {
    if (event.key === "ArrowLeft") {
      isPlaneMovingLeft = true;
    } else if (event.key === "ArrowRight") {
      isPlaneMovingRight = true;
    } else if (event.key === "ArrowUp") {
      isPlaneMovingUp = true;
    } else if (event.key === "ArrowDown") {
      isPlaneMovingDown = true;
    }
  });

  document.addEventListener("keyup", function(event) {
    if (event.key === "ArrowLeft") {
      isPlaneMovingLeft = false;
    } else if (event.key === "ArrowRight") {
      isPlaneMovingRight = false;
    } else if (event.key === "ArrowUp") {
      isPlaneMovingUp = false;
    } else if (event.key === "ArrowDown") {
      isPlaneMovingDown = false;
    }
  });

  document.getElementById("restartButton").addEventListener("click", restartGame);

  const difficultyRange = document.getElementById("difficultyRange");
  const difficultyValue = document.getElementById("difficultyValue");
  
  difficultyRange.addEventListener("input", function() {
    difficultyValue.textContent = this.value;
    difficultyLevel = parseInt(this.value);
  });

  const cloudDensityRange = document.getElementById("cloudDensityRange");
  cloudDensityRange.addEventListener("input", updateCloudDensity);

  updateGame();
</script>
</body>
</html>
