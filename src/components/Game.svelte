<!-- Script -->
<script lang="ts">
  // Loadash
  import _ from "lodash";

  // Props
  export let boardSize: number;

  // Snake
  let snake = [[4, 4]];
  $: snakeLen = snake.length;
  let food = randomFood();
  let foodEaten = false;

  // Game stuff
  let direction = [0, 1];
  let gameover = false;
  let start = true;
  let lastRenderTime: number;
  $: snakeSpeed = 2 * (1 + snakeLen / 5);
  let snakeBite = false;

  // Generate a random integer
  function getRandomInt(max: number) {
    return Math.floor(Math.random() * Math.floor(max));
  }

  // Update food position randomly
  function randomFood() {
    let pos = [getRandomInt(boardSize - 1), getRandomInt(boardSize - 1)];
    while (snake.some((el) => _.isEqual(el, pos))) {
      pos = [getRandomInt(boardSize - 1), getRandomInt(boardSize - 1)];
    }
    return pos;
  }

  // Handle the keypress events
  function handleKeydown(e: KeyboardEvent) {
    // We start or restart the game
    if (gameover || start) {
      startGame();
      requestAnimationFrame(loop);
      return;
    }
    // Move the snake
    switch (e.key) {
      case "ArrowLeft":
        direction = [0, -1];
        break;
      case "ArrowRight":
        direction = [0, 1];
        break;
      case "ArrowUp":
        direction = [-1, 0];
        break;
      case "ArrowDown":
        direction = [1, 0];
        break;
      default:
        return;
    }
  }

  // Start the game
  function startGame() {
    gameover = false;
    start = false;
    snake = [[4, 4]];
    food = randomFood();
    snakeBite = false;
  }

  function playGame() {
    let snakeTmp = [...snake];
    // Move the snake body
    for (let i = snake.length - 1; i >= 1; i--) {
      snakeTmp[i] = [snakeTmp[i - 1][0], snakeTmp[i - 1][1]];
    }
    // Move the snake head
    snakeTmp[0] = [
      snakeTmp[0][0] + direction[0],
      snakeTmp[0][1] + direction[1],
    ];

    // Check if snake eats the food
    if (_.isEqual(snakeTmp[0], food)) {
      snakeTmp.push(food);
      foodEaten = true;
      food = randomFood();
    } else {
      foodEaten = false;
    }

    // Check out of bounds
    if (
      isOutOfBounds(snakeTmp[0][0]) ||
      isOutOfBounds(snakeTmp[0][1]) ||
      snakeCollision(snakeTmp)
    ) {
      gameover = true;
    }

    // Update the snake
    snake = [...snakeTmp];
  }

  // Check if the snake is out of bounds
  function isOutOfBounds(n: number) {
    return n < 0 || n > boardSize - 1;
  }

  // Check if the snake collides with itself
  function snakeCollision(sn: number[][]) {
    const stringifiedSnake = sn.map((s) => JSON.stringify(s));
    console.log(new Set(stringifiedSnake), stringifiedSnake);
    if (
      snake.length > 1 &&
      new Set(stringifiedSnake).size != stringifiedSnake.length &&
      !foodEaten
    ) {
      snakeBite = true;
      return true;
    }
  }

  // Loop
  function loop(currentTime) {
    if (gameover || start) return;
    // New animation frame
    requestAnimationFrame(loop);
    // Time elapsed
    let delta = (currentTime - lastRenderTime) / 1000;
    // Should the snake be moved?
    if (delta < 1 / snakeSpeed) return;
    // Play the game
    playGame();
    // Update rendering times
    lastRenderTime = currentTime;
  }
</script>

<!-- HTML -->
<svelte:window on:keydown={(e) => handleKeydown(e)} />
<div class="gameinfo">
  <span>Snake length</span><span>{snakeLen}</span>
</div>
<div class="board">
  {#each new Array(boardSize) as row, i}
    <div class="row">
      {#each new Array(boardSize) as cell, k}
        <div
          class="square"
          class:snake={snake.some((el) => _.isEqual(el, [i, k]))}
          class:food={_.isEqual(food, [i, k])}
        />
      {/each}
    </div>
  {/each}
  {#if gameover || start}
    <div class="game-overlay">
      {#if start}
        <h2>Welcome to the snake game!</h2>
        <p>Press any key to start.</p>
      {:else}
        <h2>The game has ended!</h2>
        {#if snakeBite}
          <p class="msg">Your snake had a length of {snakeLen} but it bit itself.</p>
        {:else}
          <p class="msg">Your snake had a length of {snakeLen} but went out of bounds.</p>
        {/if}
        <p>Press any key to play again.</p>
      {/if}
    </div>
  {/if}
</div>

<!-- Style -->
<style>
  .gameinfo {
    display: flex;
    justify-content: flex-start;
    gap: 1em;
    padding: 0.5em;
    margin-bottom: 1em;
    padding: 0.25em 0.5em;
    background: hsl(210, 100%, 76%);
  }

  .board {
    position: relative;
    width: 60vmin;
    aspect-ratio: 1;
    display: grid;
    grid-template-rows: repeat(var(--board-size), 1fr);
    background-color: hsl(0, 0%, 92%);
    border: 0.5vmin solid hsl(0, 0%, 85%);
  }

  .game-overlay {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    position: absolute;
    inset: -0.5vmin;
    color: rgba(255, 255, 255, 0.9);
    background-color: rgba(0, 0, 0, 0.5);
    z-index: 99;
  }

  .row {
    display: grid;
    grid-template-columns: repeat(var(--board-size), 1fr);
  }

  .square {
    aspect-ratio: 1;
  }

  .snake {
    background-color: hsl(210, 90%, 62%);
    border: 0.25vmin solid rgba(0, 0, 0, 0.25);
  }

  .food {
    background-color: hsl(33, 80%, 60%);
    border: 0.25vmin solid rgba(0, 0, 0, 0.25);
  }

  .msg {
    padding: 0.25em 0.5em;
    max-width: 80%;
    margin: 0 auto;
    background: linear-gradient(90deg, firebrick, dodgerblue);
    color: inherit;
    border-radius: 4px;
  }
</style>
