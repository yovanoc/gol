<script lang="ts">
  import { onMount } from "svelte";

  let canvas: HTMLCanvasElement;

  let BOARD: boolean[][];
  let isGamePaused = false;
  let gameSpeed = 20;

  onMount(() => {
    const ctx = canvas.getContext("2d");

    canvas.addEventListener("click", (e) => {
      const x = Math.floor((e.clientX - canvas.offsetLeft) / TILE_SIZE);
      const y = Math.floor((e.clientY - canvas.offsetTop) / TILE_SIZE);
      BOARD[x][y] = !BOARD[x][y];
      drawAll();
    });

    const generateRandom = () => {
      const board = prepareBoard();
      for (let i = 0; i < TILES_X; i++) {
        for (let j = 0; j < TILES_Y; j++) {
          board[i][j] = Math.random() < 0.2;
        }
      }
      return board;
    };

    document.addEventListener("keydown", (e) => {
      if (e.key === "p") {
        isGamePaused = !isGamePaused;
      } else if (e.key === "+") {
        gameSpeed = Math.max(10, gameSpeed - 50);
      } else if (e.key === "-") {
        gameSpeed = Math.min(5000, gameSpeed + 50);
      } else if (e.key === "r") {
        BOARD = generateRandom();
        drawAll();
      }
    });

    let frame: number;

    const width = canvas.width;
    const height = canvas.height;
    const TILE_SIZE = 5;
    const TILES_X = width / TILE_SIZE;
    const TILES_Y = height / TILE_SIZE;
    ctx.fillStyle = "rgb(100, 240, 150)";
    ctx.strokeStyle = "rgb(90, 90, 90)";
    ctx.lineWidth = 1;

    const prepareBoard = (): boolean[][] => {
      const b = [];
      for (let i = 0; i < TILES_X; i++) {
        const row = [];
        for (let j = 0; j < TILES_Y; j++) {
          row.push(false);
        }
        b.push(row);
      }
      return b;
    };

    BOARD = prepareBoard();

    BOARD[1][0] = true;
    BOARD[2][1] = true;
    BOARD[0][2] = true;
    BOARD[1][2] = true;
    BOARD[2][2] = true;

    const drawBorders = () => {
      for (let i = 0; i < TILES_X; i++) {
        ctx.beginPath();
        ctx.moveTo(i * TILE_SIZE - 0.5, 0);
        ctx.lineTo(i * TILE_SIZE - 0.5, height);
        ctx.stroke();
      }
      for (let j = 0; j < TILES_Y; j++) {
        ctx.beginPath();
        ctx.moveTo(0, j * TILE_SIZE - 0.5);
        ctx.lineTo(width, j * TILE_SIZE - 0.5);
        ctx.stroke();
      }
    };

    const isAlive = (x: number, y: number): number => {
      if (x < 0 || x >= TILES_X || y < 0 || y >= TILES_Y) {
        return 0;
      }
      return BOARD[x][y] ? 1 : 0;
    };
    const neighboursCount = (x: number, y: number): number => {
      let count = 0;
      for (let i of [-1, 0, 1]) {
        for (let j of [-1, 0, 1]) {
          if (!(i === 0 && j === 0)) {
            count += isAlive(x + i, y + j);
          }
        }
      }
      return count;
    };

    const drawBoard = () => {
      for (let i = 0; i < TILES_X; i++) {
        for (let j = 0; j < TILES_Y; j++) {
          if (!isAlive(i, j)) {
            continue;
          }
          ctx.fillRect(i * TILE_SIZE, j * TILE_SIZE, TILE_SIZE, TILE_SIZE);
        }
      }
    };

    const computeNextGeneration = () => {
      const board = prepareBoard();
      for (let i = 0; i < TILES_X; i++) {
        for (let j = 0; j < TILES_Y; j++) {
          if (!isAlive(i, j)) {
            if (neighboursCount(i, j) === 3) {
              board[i][j] = true;
            }
          } else {
            const count = neighboursCount(i, j);
            if (count == 2 || count == 3) {
              board[i][j] = true;
            }
          }
        }
      }
      return board;
    };

    const clear = () => {
      ctx.clearRect(0, 0, width, height);
    };
    const drawAll = () => {
      clear();
      drawBoard();
      drawBorders();
    };
    const nextGen = () => {
      if (isGamePaused) {
        return;
      }
      BOARD = computeNextGeneration();
      drawAll();
    };

    const nextGenLoop = () => {
      nextGen();
      setTimeout(nextGenLoop, gameSpeed);
    };

    nextGenLoop();

    // (function loop() {
    //   frame = requestAnimationFrame(loop);

    //   nextGen();
    // })();

    return () => {
      cancelAnimationFrame(frame);
    };
  });
</script>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
  }
  canvas {
    width: 100%;
    height: 100%;
    background-color: #333;
  }
</style>

<canvas bind:this={canvas} width={2000} height={1500} />
