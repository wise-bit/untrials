<script lang="ts">
  let selectedIndex = 0;

  const games = [
    {
      title: 'not jeopardy',
      description: 'a game of not jeopardy',
      link: '/not-jeopardy',
    },
    {
      title: 'non-trivial pursuit',
      description: 'a game of non trivial pursuit',
      link: '/non-trivial-pursuit',
    },
    {
      title: 'coming soon',
      description: 'even crazier trials coming soon in 20XX',
    },
  ];
</script>

<div class="container">
  <div class="title">
    <span>[untitled]</span>
    trials
  </div>
  <div class="games">
    {#each games as game, i}
      {#if game.link}
        <button
          on:click={() => (location.href = game.link)}
          class="game {selectedIndex === i ? 'active' : ''}"
        >
          <h2>{game.title}</h2>
          <p>{game.description}</p>
        </button>
      {:else}
        <button
          class="game {selectedIndex === i ? 'active' : ''}"
          disabled
          style="pointer-events: none; scale: 0.9; opacity: 0.5;"
        >
          <h2>{game.title}</h2>
          <p>{game.description}</p>
        </button>
      {/if}
    {/each}
  </div>
</div>

<style>
  :global(body) {
    margin: 0;
    font-family: 'Comic Sans MS', 'Arial', sans-serif;
    background-color: #101114;
    color: #f9f9f9;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100vh;
    overflow: hidden;
  }

  button {
    font-family: 'Comic Sans MS', 'Arial', sans-serif;
  }

  .container {
    text-align: center;
  }

  .title {
    font-size: 3rem;
    font-weight: bold;
    margin-bottom: 2rem;
  }

  .title span {
    display: block;
    font-size: 4rem;
    color: #52caf6;
  }

  .games {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;
    max-height: 300px;
    overflow-y: auto;
    padding: 1rem;
    scrollbar-width: thin;
    scrollbar-color: #54b3d6 transparent;
    transform-box: content-box;
    overflow: visible;
  }

  .games::-webkit-scrollbar {
    width: 8px;
  }

  .games::-webkit-scrollbar-thumb {
    background: #54b3d6;
    border-radius: 4px;
  }

  .game {
    font-size: 1rem;
    padding: 1rem;
    border-radius: 8px;
    width: 400px;
    text-align: center;
    background: #6f7a90;
    color: #f9f9f9;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    transition:
      transform 0.3s,
      background 0.3s;
    cursor: pointer;
    border: none;
    outline: none;
  }

  .game:hover,
  .game:focus {
    transform: scale(1.1);
    background: #acbee4;
    color: #1a1a1a;
  }

  .game h2 {
    margin: 0;
  }

  .game p {
    margin: 0.5rem 0 0;
  }
</style>
