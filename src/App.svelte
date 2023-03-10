<!-- Script -->
<script lang="ts">
  // Module import
  import Game from "./components/Game.svelte";

  // Props
  export let boardSize = 15;
</script>

<!-- HTML -->
<main>
  <h1>Snake game</h1>
  <h2>Created with Svelte</h2>
  <Game boardSize={boardSize} --board-size={boardSize}/>
</main>

<!-- Style -->
<style>
  main {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;

    padding: 1em;
    max-width: 1200px;
    margin: 0 auto;
  }

  h1, h2 {
    margin-block: 0.25em;
  }

  h2 {
    font-weight: 500;
    margin-bottom: 1em
  }
</style>
