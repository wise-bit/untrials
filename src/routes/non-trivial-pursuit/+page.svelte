<script lang="ts">
  import { writable } from 'svelte/store';
  import { onMount } from 'svelte';

  interface Question {
    points: string;
    question: string;
    answer: string;
    visited: boolean;
    showAnswer: boolean;
  }

  interface Category {
    [categoryName: string]: Question[];
  }

  interface JeopardyData {
    [boardTitle: string]: Category[];
  }

  let fileContent: string | null = null;
  let boardTitle = 'dummy-board';
  let categories: string[] = [];
  let questions: Question[][] = [];

  const defaultQuestion: Question = {
    points: '',
    question: '',
    answer: '',
    visited: false,
    showAnswer: false,
  };

  const selectedQuestion = writable(defaultQuestion);
  const showModal = writable(false);
  const showFinalJeopardy = writable(false);

  const handleFileUpload = async (event: Event) => {
    const input = event.target as HTMLInputElement;
    if (input.files && input.files[0]) {
      const file = input.files[0];
      const reader = new FileReader();

      reader.onload = (e) => {
        fileContent = e.target?.result as string;
        const data: JeopardyData = JSON.parse(fileContent);

        boardTitle = Object.keys(data)[0];

        categories = data[boardTitle].map(
          (categoryObject) => Object.keys(categoryObject)[0],
        );

        questions = data[boardTitle].map((categoryObject) => {
          const categoryName = Object.keys(categoryObject)[0];
          return categoryObject[categoryName].map((question) => ({
            points: question.points,
            question: question.question,
            answer: question.answer,
            visited: false,
            showAnswer: false,
          }));
        });
      };

      reader.readAsText(file);
    }
  };

  const selectQuestion = (question: Question, i: number, j: number) => {
    selectedQuestion.set(question);
    questions[i][j].visited = true;
    showModal.set(true);
  };

  const closeModal = () => {
    showModal.set(false);
    selectedQuestion.set(defaultQuestion);
  };

  const goBack = () => {
    if (false) {
      return;
    }
    location.href = '/';
  };

  function downloadSample() {
    const a = document.createElement('a');
    a.href = 'samples/trials-jeopardy-sample.json';
    a.download = 'sample.json';

    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
  }

  // dice stuff

  let currentFace = 1;
  let rolling = false;
  let rollInterval: string | number | NodeJS.Timeout | undefined;

  const rollDice = () => {
    if (rolling) return;

    rolling = true;

    rollInterval = setInterval(() => {
      currentFace = Math.floor(Math.random() * 6) + 1;
    }, 100);
  };

  const switchRoll = () => {
    if (!rolling) {
      rollDice();
      return;
    }

    clearInterval(rollInterval);
    rolling = false;

    currentFace = Math.floor(Math.random() * 6) + 1;
    console.log(`Rolled: ${currentFace}`);
  };

  onMount(() => {
    return () => clearInterval(rollInterval);
  });

  // board stuff

  // Generate a 5x5 grid where only the outer layer is active
  const gridSize = 5;
  const board = Array(gridSize)
    .fill(null)
    .map((_, row) =>
      Array(gridSize)
        .fill(null)
        .map((_, col) => ({
          row,
          col,
          active:
            row === 0 ||
            col === 0 ||
            row === gridSize - 1 ||
            col === gridSize - 1,
        })),
    );
</script>

<button class="back-button" on:click={() => goBack()}>{'< back'}</button>

{#if false}
  <div class="container">
    <h1>welcome to non-trivial-pursuit</h1>
    <p>upload your custom game json</p>
    <div class="upload-container">
      <input
        type="file"
        id="file-upload"
        accept=".json"
        on:change={handleFileUpload}
      />
    </div>
    <div>
      <button class="download-sample-button" on:click={() => downloadSample()}
        >download sample</button
      >
    </div>
  </div>
{/if}

{#if true}
  <div class="container">
    <div class="board-container">
      <!-- hovering dice -->
      <div class="dice-container">
        <h1>{boardTitle}</h1>
        <div
          class="dice"
          aria-label="Roll the dice"
          role="button"
          tabindex="0"
          on:keydown={() => {}}
          on:click={switchRoll}
        >
          {currentFace}
        </div>
        <div class="dice-result">
          {#if rolling}
            rolling...
          {:else}
            Rolled a {currentFace} !
          {/if}
        </div>
      </div>

      <!-- main board -->
      <div class="board-container">
        <div class="board">
          {#each board as row}
            {#each row as cell}
              <div class="cell {cell.active ? 'active' : ''}">
                {cell.active ? `${cell.row},${cell.col}` : ''}
              </div>
            {/each}
          {/each}
        </div>
      </div>
    </div>

    <div class="score-container">asd</div>
  </div>
{/if}

<style>
  @import url('https://fonts.googleapis.com/css2?family=Fira+Mono:wght@400;500;700&family=Pixelify+Sans:wght@400..700&display=swap');

  :global(body) {
    margin: 0;
    font-family: 'Pixelify Sans', 'Comic Sans MS', 'Arial', sans-serif;
    background-color: #1a1a1a;
    color: #f9f9f9;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    height: 100vh;
  }

  .container {
    text-align: center;
    display: flex;
    flex-direction: row;
    height: 100vh;
    width: 100vw;
  }

  .back-button {
    font-family: 'Pixelify Sans', 'Comic Sans MS', 'Arial', sans-serif;
    position: absolute;
    top: 1rem;
    left: 1rem;
    padding: 0.5rem 1rem;
    background-color: #333;
    color: #f9f9f9;
    border: none;
    border-radius: 8px;
    font-size: 1.4rem;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .back-button:hover {
    background-color: #54b3d6;
    color: #1a1a1a;
  }

  .board-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    flex-basis: 70%;
    width: 70%;
    height: 100%;
    background-color: #000;
  }

  .dice-container {
    position: absolute;
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-bottom: 2rem;
  }

  .dice {
    width: 100px;
    height: 100px;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 2rem;
    font-size: 4rem;
    border: 4px solid rgb(183, 170, 170);
    border-radius: 4px;
    user-select: none;
    cursor: pointer;
    background-color: #783333;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
    transition: transform 0.2s;
  }

  .dice:hover {
    transform: scale(1.15);
  }

  .dice:active {
    transform: scale(0.95);
  }

  .dice-result {
    margin-top: 1rem;
    font-size: 2rem;
  }

  .score-container {
    flex: 1;
    flex-basis: 30%;
    width: 30%;
    background-color: #242e2b;
    border-left: 10px solid #101212;
    padding: 2rem;
  }

  /* board */

  .board {
    display: grid;
    grid-template-columns: repeat(5, 10rem);
    grid-template-rows: repeat(5, 10rem);
    gap: 2px;
    margin: 20px auto;
  }

  .cell {
    width: 10rem;
    height: 10rem;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #000000;
    border: 2px solid #000000;
    border-radius: 8px;
    font-size: 2rem;
  }

  .cell.active {
    background-color: #a0a0a0;
    color: rgb(0, 0, 0);
    font-weight: bold;
  }
</style>
