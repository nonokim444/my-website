<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>간단한 레이싱 게임</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      background: #333;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      font-family: sans-serif;
    }
    #gameArea {
      width: 300px;
      height: 500px;
      background: #111;
      position: relative;
      overflow: hidden;
      border: 5px solid white;
      border-radius: 10px;
    }
    #car {
      width: 40px;
      height: 80px;
      background: red;
      position: absolute;
      bottom: 10px;
      left: 130px;
      border-radius: 5px;
    }
    .obstacle {
      width: 40px;
      height: 80px;
      background: gray;
      position: absolute;
      top: -100px;
      left: 0;
      border-radius: 5px;
    }
    #scoreBoard {
      position: absolute;
      top: 10px;
      left: 10px;
      color: white;
      font-size: 18px;
      font-weight: bold;
      z-index: 10;
      user-select: none;
    }
  </style>
</head>
<body>

<div id="gameArea">
  <div id="scoreBoard">점수: 0</div>
  <div id="car"></div>
</div>

<script>
  const gameArea = document.getElementById('gameArea');
  const car = document.getElementById('car');
  const scoreBoard = document.getElementById('scoreBoard');
  let carX = 130;
  let score = 0;
  let gameOver = false;

  // 자동차 좌우 이동
  document.addEventListener('keydown', (e) => {
    if (gameOver) return;
    if (e.key === 'ArrowLeft' && carX > 0) {
      carX -= 20;
      car.style.left = carX + 'px';
    } else if (e.key === 'ArrowRight' && carX < 260) {
      carX += 20;
      car.style.left = carX + 'px';
    }
  });

  // 장애물 생성 및 이동 함수
  function createObstacle() {
    if (gameOver) return;

    const obs = document.createElement('div');
    obs.classList.add('obstacle');
    obs.style.left = Math.floor(Math.random() * 260) + 'px';
    gameArea.appendChild(obs);

    let obsY = -100;

    const move = setInterval(() => {
      if (gameOver) {
        clearInterval(move);
        return;
      }

      obsY += 5;
      obs.style.top = obsY + 'px';

      // 충돌 검사
      if (
        obsY + 80 >= 420 &&
        obs.offsetLeft < carX + 40 &&
        obs.offsetLeft + 40 > carX
      ) {
        gameOver = true;
        alert(`💥 충돌! 게임 오버! 점수: ${score}`);
        location.reload();
      }

      // 장애물이 아래로 지나가면 점수 증가 및 제거
      if (obsY > 500) {
        clearInterval(move);
        gameArea.removeChild(obs);
        if (!gameOver) {
          score++;
          scoreBoard.innerText = `점수: ${score}`;
        }
      }
    }, 20);
  }

  // 일정 간격으로 장애물 생성
  setInterval(createObstacle, 1200);
</script>

</body>
</html>
