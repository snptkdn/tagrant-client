<script>
  import { page } from '$app/stores';
  import { onMount } from 'svelte';

  $: id = $page.url.searchParams.get('id');
  $: name = $page.url.searchParams.get('name');

  let playerX = 50;
  let playerY = 50;
  let enemyX = 20;
  let enemyY = 20;
  let score = 0;
  const speed = 0.5;
  const collisionDistance = 1; // プレイヤーと敵の衝突判定距離（%単位）

  let keys = {
    w: false,
    s: false,
    a: false,
    d: false
  };

  let isGameOver = false;
  let gameLoopId;

  function handleKeydown(event) {
    if (event.key.toLowerCase() in keys) {
      keys[event.key.toLowerCase()] = true;
    }
  }

  function handleKeyup(event) {
    if (event.key.toLowerCase() in keys) {
      keys[event.key.toLowerCase()] = false;
    }
  }

  function movePlayer() {
    if (keys.w) playerY = Math.max(0, playerY - speed);
    if (keys.s) playerY = Math.min(100, playerY + speed);
    if (keys.a) playerX = Math.max(0, playerX - speed);
    if (keys.d) playerX = Math.min(100, playerX + speed);
  }

  function moveEnemy() {
    const dx = playerX - enemyX;
    const dy = playerY - enemyY;
    const distance = Math.sqrt(dx * dx + dy * dy);

    if (distance > 0) {
      enemyX += (dx / distance) * speed;
      enemyY += (dy / distance) * speed;
    }

    // 衝突判定
    if (distance < collisionDistance) {
      gameOver();
    }
  }

  function gameLoop() {
    if (!isGameOver) {
      movePlayer();
      moveEnemy();
      score += 1;
      gameLoopId = requestAnimationFrame(gameLoop);
    }
  }

  function gameOver() {
    isGameOver = true;
    cancelAnimationFrame(gameLoopId);
  }

  function restartGame() {
    playerX = 50;
    playerY = 50;
    enemyX = 20;
    enemyY = 20;
    score = 0;
    isGameOver = false;
    gameLoop();
  }

  onMount(() => {
    window.addEventListener('keydown', handleKeydown);
    window.addEventListener('keyup', handleKeyup);
    gameLoop();
    return () => {
      window.removeEventListener('keydown', handleKeydown);
      window.removeEventListener('keyup', handleKeyup);
      cancelAnimationFrame(gameLoopId);
    };
  });
</script>

<div class="game-container">
  <div class="game-area">
    <div class="player" style="left: {playerX}%; top: {playerY}%;"></div>
    <div class="enemy" style="left: {enemyX}%; top: {enemyY}%;"></div>
  </div>

  {#if isGameOver}
    <div class="game-over">
      <h2>{name} is Dead!</h2>
      <h2>Your Score: {score}</h2>
      <button on:click={restartGame}>Restart</button>
    </div>
  {/if}
</div>

<style>
  .game-container {
    position: relative;
  }

  .game-area {
    width: 100%;
    height: 400px;
    background-color: #f0f0f0;
    position: relative;
    border: 1px solid #ccc;
  }

  .player, .enemy {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    position: absolute;
    transform: translate(-50%, -50%);
  }

  .player {
    background-color: blue;
  }

  .enemy {
    background-color: red;
  }

  .game-over {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.7);
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    color: white;
  }

  button {
    margin-top: 20px;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
  }
</style>
