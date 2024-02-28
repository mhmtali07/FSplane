<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gamecik</title>
<style>
  body, html {
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: skyblue;
  }
  canvas {
    image-rendering: pixelated;
  }
  .menu {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    text-align: center;
  }
  #topPlayers {
    position: absolute;
    top: 10px;
    right: 10px;
    text-align: right;
    font-size: 14px;
    color: black;
  }
</style>
</head>
<body>
<canvas id="gameCanvas"></canvas>
<div class="menu">
  <h1 id="gameOverMessage">Oyun Bitti!</h1>
  <h2 id="scoreDisplay"></h2>
  <input type="text" id="nicknameInput" placeholder="Nickname">
  <button id="startButton">Oyuna Başla</button>
  <button id="restartButton" style="display: none;">Tekrar Oyna</button>
  <div>
    <label for="difficultyRange">Zorluk Seviyesi: </label>
    <input type="range" id="difficultyRange" min="1" max="5" value="3">
    <span id="difficultyValue">3</span>
  </div>
  <div>
    <label for="cloudDensityRange">Bulut Miktari: </label>
    <input type="range" id="cloudDensityRange" min="1" max="5" value="3">
    <span id="cloudDensityValue">3</span>
  </div>
</div>
<div id="score" style="position: absolute; top: 10px; left: 10px; font-size: 24px; color: black;">Skor: 0</div>
<div id="topPlayers">
  <h3>En iyi 5 Oyuncu</h3>
  <ol id="topPlayersList">
    <!-- Buraya JavaScript ile en iyi 5 oyuncu ekleyeceğiz -->
  </ol>
</div>
<script>
  const canvas = document.getElementById("gameCanvas");
  const ctx = canvas.getContext("2d");
  const initialCanvasWidth = 800;
  const initialCanvasHeight = 600;
  const planeWidth = 40; // Uçağın genişliği
  const planeHeight = 25; // Uçağın yüksekliği
  let planeX = canvas.width / 2 - planeWidth / 2;
  let planeY = canvas.height - planeHeight - 60;
  const planeSpeed = 2;
  const cloudRadius = 20; // Bulutların boyutu
  const redCloudRadius = 10; // Kırmızı bulutun boyutu
  const redCloudProbability = 25000; // Her 6000 bulutta bir kırmızı bulut oluşturulacak
  const magnetForce = 1; // Mıknatıs etkisi hızı
  let clouds = [];
  let gameRunning = false;
  let score = 0;
  let difficultyLevel = 3;
  let cloudDensity = 3;
  let isMagnetActive = false; // Mıknatıs etkisinin aktif olup olmadığını belirtir
  let nickname = "";
  let topPlayers = JSON.parse(localStorage.getItem("topPlayers")) || [];

  function drawPlane() {
    ctx.fillStyle = "black";
    ctx.beginPath();
    ctx.moveTo(planeX, planeY);
    ctx.lineTo(planeX + planeWidth / 2, planeY - planeHeight);
    ctx.lineTo(planeX + planeWidth, planeY);
    ctx.closePath();
    ctx.fill();
  }

  function movePlane() {
    if (isPlaneMovingLeft && planeX > 0) {
      planeX -= planeSpeed + planeSpeed * 0.2; // Uçağın hareket hassasiyeti 2 katına çıkarıldı
    }
    if (isPlaneMovingRight && planeX < canvas.width - planeWidth) {
      planeX += planeSpeed + planeSpeed * 0.2; // Uçağın hareket hassasiyeti 2 katına çıkarıldı
    }
    if (isPlaneMovingUp && planeY > 0) {
      planeY -= planeSpeed + planeSpeed * 0.2; // Uçağın hareket hassasiyeti 2 katına çıkarıldı
    }
    if (isPlaneMovingDown && planeY < canvas.height - planeHeight) {
      planeY += planeSpeed + planeSpeed * 0.2; // Uçağın hareket hassasiyeti 2 katına çıkarıldı
    }
  }

  function clearCanvas() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
  }

  function drawCloud(x, y, isRed) {
    ctx.fillStyle = isRed ? "red" : "white";
    ctx.beginPath();
    ctx.arc(x, y, isRed ? redCloudRadius : cloudRadius, 0, Math.PI * 2);
    ctx.fill();
  }

  function moveClouds() {
    for (let i = 0; i < clouds.length; i++) {
      let forceX = 0;
      let forceY = 0;
      if (isMagnetActive) {
        const deltaX = clouds[i].x - planeX;
        const deltaY = clouds[i].y - planeY;
        const distance = Math.sqrt(deltaX * deltaX + deltaY * deltaY);
        forceX = (magnetForce / distance) * deltaX;
        forceY = (magnetForce / distance) * deltaY;
      }
      clouds[i].x += forceX;
      clouds[i].y += forceY + difficultyLevel / 2;
      if (clouds[i].y > canvas.height) {
        clouds[i].y = 0;
        clouds[i].x = Math.random() * canvas.width;
        if (score % redCloudProbability === 0) {
          // Her 6000 bulutta bir kırmızı bulut oluştur
          createRedCloud();
        }
        score++;
        document.getElementById("score").innerText = "Skor: " + score;
      }
      if (clouds[i].x > planeX && clouds[i].x < planeX + planeWidth &&
          clouds[i].y > planeY && clouds[i].y < planeY + planeHeight) {
        if (clouds[i].isRed) {
          // Eğer kırmızı buluta dokunulduysa
          isMagnetActive = true; // Mıknatıs etkisini aktifleştir
          setTimeout(() => {
            isMagnetActive = false; // 3 saniye sonra mıknatıs pasif hale gelir
          }, 3000);
          clouds.splice(i, 1); // Dokunulan kırmızı bulutu yok et
        } else {
          endGame();
          return;
        }
      }
    }
  }

  function drawClouds() {
    for (let i = 0; i < clouds.length; i++) {
      drawCloud(clouds[i].x, clouds[i].y, clouds[i].isRed);
    }
  }

  function createClouds() {
    clouds = [];
    const cloudCount = 20 * cloudDensity; // Bulut sayısı 2 katına çıkarıldı
    for (let i = 0; i < cloudCount; i++) {
      clouds.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        isRed: false
      });
    }
  }

  function createRedCloud() {
    clouds.push({
      x: Math.random() * canvas.width,
      y: 0,
      isRed: true
    });
  }

  function updateCloudDensity() {
    cloudDensity = parseInt(document.getElementById("cloudDensityRange").value);
    document.getElementById("cloudDensityValue").textContent = cloudDensity;
    createClouds();
  }

  canvas.width = initialCanvasWidth;
  canvas.height = initialCanvasHeight;

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

  document.getElementById("startButton").addEventListener("click", startGame);

  function startGame() {
    gameRunning = true;
    score = 0;
    planeX = canvas.width / 2 - planeWidth / 2; // Uçağı başlangıç konumuna döndür
    planeY = canvas.height - planeHeight - 60; // Uçağı başlangıç konumuna döndür
    nickname = document.getElementById("nicknameInput").value || "Player"; // default nickname is "Player" if not provided
    document.querySelector(".menu").style.display = "none";
    updateGame();
  }

  const difficultyRange = document.getElementById("difficultyRange");
  const difficultyValue = document.getElementById("difficultyValue");
  
  difficultyRange.addEventListener("input", function() {
    difficultyValue.textContent = this.value;
    difficultyLevel = parseInt(this.value);
  });

  const cloudDensityRange = document.getElementById("cloudDensityRange");
  cloudDensityRange.addEventListener("input", updateCloudDensity);

  function endGame() {
    gameRunning = false;
    document.querySelector(".menu").style.display = "block";
    document.getElementById("scoreDisplay").innerText = "Skorunuz: " + score;

    // Skoru en iyi 5 oyuncu listesine ekleyelim
    topPlayers.push({ name: nickname, score: score });
    topPlayers.sort((a, b) => b.score - a.score); // Skora göre sırala
    topPlayers = topPlayers.slice(0, 5); // En iyi 5 oyuncuyu seç
    localStorage.setItem("topPlayers", JSON.stringify(topPlayers));

    // En iyi 5 oyuncu listesini güncelle
    updateTopPlayersList();
  }

  function updateTopPlayersList() {
    const topPlayersList = document.getElementById("topPlayersList");
    topPlayersList.innerHTML = "";
    topPlayers.forEach((player, index) => {
      const listItem = document.createElement("li");
      listItem.textContent = `${player.name}: ${player.score}`;
      topPlayersList.appendChild(listItem);
    });
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

  setInterval(() => {
    isMagnetActive = false; // 3 saniye sonra mıknatıs pasif hale gelir
  }, 3000);

  updateGame();
</script>
</body>
</html>
