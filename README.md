<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <title>테트리스</title>
  <style>
    body {
      background: #000;
      color: white;
      font-family: Arial, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
    }
    #scoreDisplay {
      font-size: 24px;
      margin-bottom: 10px;
    }
    canvas {
      border: 2px solid #fff;
      background: #111;
    }
  </style>
</head>
<body>
  <div id="scoreDisplay">점수: 0</div>
  <canvas id="tetris" width="240" height="400"></canvas>

  <script>
    const canvas = document.getElementById('tetris');
    const context = canvas.getContext('2d');
    context.scale(20, 20); // 1칸 = 20px

    let score = 0;
    let dropInterval = 1000;
    let dropCounter = 0;
    let lastTime = 0;

    const scoreDisplay = document.getElementById('scoreDisplay');

    const colors = [
      null,
      '#FF0D72', // T
      '#0DC2FF', // O
      '#0DFF72', // L
      '#F538FF', // J
      '#FF8E0D', // I
      '#FFE138', // S
      '#3877FF', // Z
    ];

    const arena = createMatrix(12, 20);
    const player = {
      pos: { x: 0, y: 0 },
      matrix: null,
    };

    function updateScore(points = 0) {
      score += points;
      scoreDisplay.textContent = '점수: ' + score;

      // 점수에 따라 속도 증가
      if (score >= 1500) {
        dropInterval = 400;
      } else if (score >= 1000) {
        dropInterval = 600;
      } else if (score >= 500) {
        dropInterval = 800;
      } else {
        dropInterval = 1000;
      }
    }

    function createMatrix(w, h) {
      const matrix = [];
      while (h--) {
        matrix.push(new Array(w).fill(0));
      }
      return matrix;
    }

    function createPiece(type) {
      if (type === 'T') {
        return [
          [0, 0, 0],
          [1, 1, 1],
          [0, 1, 0],
        ];
      } else if (type === 'O') {
        return [
          [2, 2],
          [2, 2],
        ];
      } else if (type === 'L') {
        return [
          [0, 3, 0],
          [0, 3, 0],
          [0, 3, 3],
        ];
      } else if (type === 'J') {
        return [
          [0, 4, 0],
          [0, 4, 0],
          [4, 4, 0],
        ];
      } else if (type === 'I') {
        return [
          [0, 5, 0, 0],
          [0, 5, 0, 0],
          [0, 5, 0, 0],
          [0, 5, 0, 0],
        ];
      } else if (type === 'S') {
        return [
          [0, 6, 6],
          [6, 6, 0],
          [0, 0, 0],
        ];
      } else if (type === 'Z') {
        return [
          [7, 7, 0],
          [0, 7, 7],
          [0, 0, 0],
        ];
      }
    }

    function drawMatrix(matrix, offset) {
      matrix.forEach((row, y) => {
        row.forEach((value, x) => {
          if (value !== 0) {
            context.fillStyle = colors[value];
            context.fillRect(x + offset.x, y + offset.y, 1, 1);
            context.strokeStyle = '#000';
            context.lineWidth = 0.05;
            context.strokeRect(x + offset.x, y + offset.y, 1, 1);
          }
        });
      });
    }

    function drawGrid() {
      context.strokeStyle = '#333'; // 그리드 선 색
      context.lineWidth = 0.05;

      for (let x = 0; x < arena[0].length; x++) {
        context.beginPath();
        context.moveTo(x, 0);
        context.lineTo(x, arena.length);
        context.stroke();
      }

      for (let y = 0; y < arena.length; y++) {
        context.beginPath();
        context.moveTo(0, y);
        context.lineTo(arena[0].length, y);
        context.stroke();
      }
    }

    function draw() {
      context.fillStyle = '#000';
      context.fillRect(0, 0, canvas.width, canvas.height);

      drawGrid(); // 배경 그리드 먼저 그리기
      drawMatrix(arena, { x: 0, y: 0 });
      drawMatrix(player.matrix, player.pos);
    }

    function merge(arena, player) {
      player.matrix.forEach((row, y) => {
        row.forEach((value, x) => {
          if (value !== 0) {
            arena[y + player.pos.y][x + player.pos.x] = value;
          }
        });
      });
    }

    function collide(arena, player) {
      const [m, o] = [player.matrix, player.pos];
      for (let y = 0; y < m.length; ++y) {
        for (let x = 0; x < m[y].length; ++x) {
          if (m[y][x] !== 0 &&
              (arena[y + o.y] && arena[y + o.y][x + o.x]) !== 0) {
            return true;
          }
        }
      }
      return false;
    }

    function arenaSweep() {
      let rowsCleared = 0;
      outer: for (let y = arena.length - 1; y >= 0; --y) {
        for (let x = 0; x < arena[y].length; ++x) {
          if (arena[y][x] === 0) continue outer;
        }
        arena.splice(y, 1);
        arena.unshift(new Array(arena[0].length).fill(0));
        ++rowsCleared;
      }
      if (rowsCleared > 0) {
        updateScore(rowsCleared * 100);
      }
    }

    function playerDrop() {
      player.pos.y++;
      if (collide(arena, player)) {
        player.pos.y--;
        merge(arena, player);
        playerReset();
        arenaSweep();
      }
      dropCounter = 0;
    }

    function playerMove(dir) {
      player.pos.x += dir;
      if (collide(arena, player)) {
        player.pos.x -= dir;
      }
    }

    function playerReset() {
      const pieces = 'TJLOSZI';
      player.matrix = createPiece(pieces[Math.floor(Math.random() * pieces.length)]);
      player.pos.y = 0;
      player.pos.x = (arena[0].length / 2 | 0) - (player.matrix[0].length / 2 | 0);

      if (collide(arena, player)) {
        arena.forEach(row => row.fill(0));
        alert('게임 오버! 최종 점수: ' + score);
        score = 0;
        updateScore(0);
      }
    }

    function playerRotate(dir) {
      const pos = player.pos.x;
      let offset = 1;
      rotate(player.matrix, dir);
      while (collide(arena, player)) {
        player.pos.x += offset;
        offset = -(offset + (offset > 0 ? 1 : -1));
        if (offset > player.matrix[0].length) {
          rotate(player.matrix, -dir);
          player.pos.x = pos;
          return;
        }
      }
    }

    function rotate(matrix, dir) {
      for (let y = 0; y < matrix.length; ++y) {
        for (let x = 0; x < y; ++x) {
          [matrix[x][y], matrix[y][x]] = [matrix[y][x], matrix[x][y]];
        }
      }
      if (dir > 0) matrix.forEach(row => row.reverse());
      else matrix.reverse();
    }

    function update(time = 0) {
      const deltaTime = time - lastTime;
      lastTime = time;
      dropCounter += deltaTime;

      if (dropCounter > dropInterval) {
        playerDrop();
      }

      draw();
      requestAnimationFrame(update);
    }

    document.addEventListener('keydown', event => {
      if (event.key === 'ArrowLeft') playerMove(-1);
      else if (event.key === 'ArrowRight') playerMove(1);
      else if (event.key === 'ArrowDown') playerDrop();
      else if (event.key === 'q') playerRotate(-1);
      else if (event.key === 'ArrowUp') playerRotate(1);
    });

    playerReset();
    update();
  </script>
</body>
</html>
