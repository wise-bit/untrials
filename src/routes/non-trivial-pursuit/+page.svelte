<script lang="ts">
  import { writable } from 'svelte/store';
  import { onMount } from 'svelte';

  interface Question {
    points: number;
    question: string;
    options: string[];
    answerKey: number;
    visited: boolean;
  }

  interface Category {
    [categoryName: string]: Question[];
  }

  interface Player {
    name: string;
    score: number;
  }

  interface NonTrivialPursuitData {
    categories: Category[];
  }

  interface InputData {
    title: string;
    subtitle: string;
    categories: Category[];
  }

  let fileContent: string | null = null;
  let boardTitle: string = 'upload game file !';
  let boardSubtitle: string = "i'm a subtitle !";
  let triviaData: NonTrivialPursuitData = { categories: [] };
  let players: Player[] = [];

  let currentFace = 1;
  let rolling = false;
  let rollInterval: string | number | NodeJS.Timeout | undefined;
  let currentBoardPos = -1;
  let rolledOnce = false;

  let unansweredMatrix: number[][] = [];
  let correctAnswerIndex = -1;
  let userAnswerIndex = -1;

  const defaultQuestion: Question = {
    points: 0,
    question: '',
    options: ['', '', ''],
    answerKey: 0,
    visited: false,
  };

  const selectedQuestion = writable(defaultQuestion);
  const showQuestionModal = writable(false);
  const showEditorModal = writable(false);

  const handleFileUpload = async (event: Event) => {
    const input = event.target as HTMLInputElement;
    if (input.files && input.files[0]) {
      const file = input.files[0];
      const reader = new FileReader();

      reader.onload = (e) => {
        fileContent = e.target?.result as string;
        const data: InputData = JSON.parse(fileContent);

        boardTitle = data.title;
        boardSubtitle = data.subtitle;

        triviaData = {
          categories: data.categories.map((categoryObject) =>
            Object.fromEntries(
              Object.entries(categoryObject).map(
                ([categoryName, questions], _categoryIndex) => [
                  categoryName,
                  questions.map((question, _questionIndex) => ({
                    // id: (categoryIndex + 1) * 100 + questionIndex + 1,
                    points: question.points,
                    question: question.question,
                    options: question.options,
                    answerKey: question.answerKey,
                    visited: false,
                  })),
                ],
              ),
            ),
          ),
        };

        unansweredMatrix = triviaData.categories.map((category) =>
          Array.from(
            { length: category[Object.keys(category)[0]].length },
            (_, i) => i,
          ),
        );

        currentBoardPos = 0;
      };

      reader.readAsText(file);
    }
  };

  $: {
    if (unansweredMatrix) {
      console.log('unansweredMatrix changed, reloading component');
    }
  }

  // ----------------------------------
  // TODO: remove after
  // import jsonFile from './trials-non-trivial-pursuit-sample.json';
  // onMount(() => {
  //   const files = [
  //     new File([JSON.stringify(jsonFile)], 'dummy.json', {
  //       type: 'application/json',
  //       lastModified: 0,
  //     }),
  //   ];
  //   const mockEvent = new Event('change', {
  //     bubbles: true,
  //     cancelable: true,
  //   });
  //   Object.defineProperty(mockEvent, 'target', {
  //     value: {
  //       files,
  //     },
  //   });
  //   handleFileUpload(mockEvent);
  // });
  // ----------------------------------

  const selectQuestion = (categoryIndex: number) => {
    if (categoryIndex == -1) {
      selectedQuestion.set(defaultQuestion);
      return;
    }

    const randomIndex = Math.floor(
      Math.random() * unansweredMatrix[categoryIndex].length,
    );
    const questionIndex = unansweredMatrix[categoryIndex].splice(
      randomIndex,
      1,
    )[0];

    const question =
      triviaData.categories[categoryIndex][
        Object.keys(triviaData.categories[categoryIndex])[0]
      ][questionIndex];
    selectedQuestion.set(question);
  };

  function checkAnswer(selectedIndex: number) {
    userAnswerIndex = selectedIndex;
    correctAnswerIndex = $selectedQuestion.answerKey - 1;
  }

  function getClassForOption(index: number) {
    if (userAnswerIndex === -1) return '';
    if (index === userAnswerIndex) {
      if (userAnswerIndex === correctAnswerIndex) {
        return 'correct';
      } else {
        return 'incorrect';
      }
    } else if (index === correctAnswerIndex) {
      return 'correct';
    }
    return '';
  }

  const openQuestionModal = () => {
    selectQuestion(currentBoardPos % unansweredMatrix.length);
    showQuestionModal.set(true);
    rolledOnce = false;
  };

  const closeQuestionModal = () => {
    showQuestionModal.set(false);
    selectedQuestion.set(defaultQuestion);

    correctAnswerIndex = -1;
    userAnswerIndex = -1;

    unansweredMatrix = [...unansweredMatrix];
  };

  const openEditorModal = () => {
    showEditorModal.set(true);
  };

  const closeEditorModal = () => {
    showEditorModal.set(false);
    unansweredMatrix = [...unansweredMatrix];
  };

  const goBack = () => {
    const confirmBack = confirm('are you sure you want to go back to home?');
    if (confirmBack) {
      location.href = '/';
    }
  };

  function downloadSample() {
    const a = document.createElement('a');
    a.href = 'samples/trials-non-trivial-pursuit-sample.json';
    a.download = 'sample.json';

    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
  }

  function updateScore(playerIndex: number, change: number) {
    players[playerIndex].score += change;
  }

  function addNewPlayer() {
    const playerNameInput = document.querySelector(
      '.player-name-input',
    ) as HTMLInputElement;
    const newPlayerName = playerNameInput.value;
    if (newPlayerName) {
      players = [...players, { name: newPlayerName, score: 0 }];
      playerNameInput.value = '';
    }
  }

  // dice stuff

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
    currentBoardPos = nextAvailablePosition(currentBoardPos + currentFace);

    rolledOnce = true;
  };

  function categoryHasQuestionsLeft(categoryIndex: number) {
    if (categoryIndex == -1) return false;
    return (
      unansweredMatrix.length > categoryIndex &&
      unansweredMatrix[categoryIndex].length > 0
    );
  }

  function nextAvailablePosition(categoryIndex: number) {
    for (let i = 0; i < unansweredMatrix.length; i++) {
      const next_index = (categoryIndex + i) % 16;
      if (categoryHasQuestionsLeft(next_index % unansweredMatrix.length)) {
        return next_index;
      }
    }
    console.log('no available categories');
    return -1;
  }

  function questionAvailable(categoryIndex: number, questionIndex: number) {
    return unansweredMatrix[categoryIndex].indexOf(questionIndex) !== -1;
  }

  function toggleQuestionStatus(categoryIndex: number, questionIndex: number) {
    const matrixIndex = unansweredMatrix[categoryIndex].indexOf(questionIndex);
    if (matrixIndex === -1) {
      unansweredMatrix[categoryIndex].push(questionIndex);
    } else {
      unansweredMatrix[categoryIndex].splice(matrixIndex, 1);
    }
    unansweredMatrix = [...unansweredMatrix];
  }

  onMount(() => {
    return () => clearInterval(rollInterval);
  });

  // board stuff
  // TODO: cleanup someday

  // Generate a 5x5 grid where only the outer layer is active
  const gridSize = 5;
  const board = Array(gridSize)
    .fill(null)
    .map((_, row) =>
      Array(gridSize)
        .fill(null)
        .map((_, col) => {
          const active =
            row === 0 ||
            col === 0 ||
            row === gridSize - 1 ||
            col === gridSize - 1;
          return {
            row,
            col,
            active,
            categoryIndex: -1,
            index: -1,
          };
        }),
    );

  let index = 0;
  let directions = [
    [0, 1],
    [1, 0],
    [0, -1],
    [-1, 0],
  ];

  let row = 0;
  let col = 0;
  let dir = 0;

  while (board[row][col].active) {
    board[row][col].index = index;
    board[row][col].categoryIndex = index % 4;
    index++;

    let nextRow = row + directions[dir][0];
    let nextCol = col + directions[dir][1];

    if (
      nextRow < 0 ||
      nextRow >= gridSize ||
      nextCol < 0 ||
      nextCol >= gridSize ||
      !board[nextRow][nextCol].active ||
      board[nextRow][nextCol].index !== -1
    ) {
      dir = (dir + 1) % 4;
      nextRow = row + directions[dir][0];
      nextCol = col + directions[dir][1];
    }

    if (board[nextRow][nextCol].index !== -1) break;
    row = nextRow;
    col = nextCol;
  }
</script>

<div class="aux-btns">
  <button class="back-button hov-btn" on:click={() => goBack()}>
    {'< back'}
  </button>

  <div class="file-buttons-container">
    <div class="upload-container">
      <input
        type="file"
        id="file-upload"
        accept=".json"
        on:change={handleFileUpload}
      />
    </div>
    <div
      class="download-sample-button hov-btn"
      aria-label="download sample"
      role="button"
      tabindex="0"
      on:click={downloadSample}
      on:keydown={downloadSample}
    >
      {'download template'}
    </div>
  </div>
</div>

<button class="questions-editor hov-btn" on:click={openEditorModal}>
  {'questions editor'}
</button>

{#if true}
  <div class="container">
    <div class="board-container">
      <!-- hovering dice -->
      <div class="dice-container">
        <div>
          <h1>{boardTitle}</h1>
          <h2>{boardSubtitle}</h2>
        </div>
        <div
          class="dice hov-btn"
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

        <div>
          <button
            class="show-question-btn hov-btn"
            on:click={openQuestionModal}
            disabled={!fileContent || !rolledOnce}
          >
            show question!
          </button>
        </div>
      </div>

      <!-- main board -->
      <div class="board-container">
        <div class="board">
          {#each board as row}
            {#each row as cell}
              <div
                class={`cell
                  ${cell.active ? 'active' : ''}
                  cat-${cell.categoryIndex}
                  ${!categoryHasQuestionsLeft(cell.categoryIndex) ? 'inactive-cell' : ''}
                `}
              >
                {#if cell.active}
                  <div
                    class={`cell-number
                      ${currentBoardPos === cell.index ? 'current-cell' : ''} 
                    `}
                  >
                    {cell.index + 1}
                  </div>
                {/if}
              </div>
            {/each}
          {/each}
        </div>
      </div>
    </div>

    <div class="score-container">
      <div class="category-guide">
        {#if !fileContent}
          <div style="font-size: 20px">upload game file to view categories</div>
        {/if}
        {#each triviaData.categories as category, categoryIndex}
          {#each Object.keys(category) as categoryName}
            <div class="category catname-{categoryIndex}">
              {categoryIndex + 1}. {categoryName}
            </div>
          {/each}
        {/each}
      </div>
      <div class="player-score new-player-box">
        <input
          class="player-name-input"
          type="text"
          placeholder="player name"
        />
        <div>
          <button class="add-player-btn hov-btn" on:click={addNewPlayer}
            >add!</button
          >
        </div>
      </div>

      {#if players.length === 0}
        <div class="no-players-message">no current players...</div>
      {/if}
      {#each players as player, i}
        <div class="player-score">
          <div>{player.name}</div>
          <div class="score-holder">
            <div>{player.score}</div>
            <div>
              <button
                class="score-update-btn"
                on:click={() => updateScore(i, -50)}>-</button
              >
              <button
                class="score-update-btn"
                on:click={() => updateScore(i, +50)}>+</button
              >
            </div>
          </div>
        </div>
      {/each}
    </div>
  </div>
{/if}

{#if $showQuestionModal}
  <div
    class="modal-overlay"
    role="button"
    tabindex="0"
    on:click={closeQuestionModal}
    on:keydown={() => {}}
  >
    <div
      class="modal"
      role="button"
      tabindex="0"
      on:click|stopPropagation
      on:keydown|stopPropagation
    >
      {#if $selectedQuestion}
        <div>[ {$selectedQuestion.points} points ]</div>
        <h4>{$selectedQuestion.question}</h4>

        <div>
          {#key userAnswerIndex}
            {#each $selectedQuestion.options as option, index}
              <div
                aria-label="select answer"
                role="button"
                tabindex="0"
                class="option opt-{getClassForOption(index)}"
                on:click={() => checkAnswer(index)}
                on:keydown={() => {}}
              >
                {option}
              </div>
            {/each}
          {/key}
        </div>

        <div class="button-set">
          <button class="close-button hov-btn" on:click={closeQuestionModal}
            >close</button
          >
        </div>
      {/if}
    </div>
  </div>
{/if}

{#if $showEditorModal}
  <div
    class="modal-overlay"
    role="button"
    tabindex="0"
    on:click={closeQuestionModal}
    on:keydown={() => {}}
  >
    <div
      class="modal"
      role="button"
      tabindex="0"
      on:click|stopPropagation
      on:keydown|stopPropagation
    >
      <h4>edit questions</h4>

      {#key unansweredMatrix}
        {#each triviaData.categories as category, categoryIndex}
          <div>{Object.keys(category)[0]}</div>
          <div class="editable-question">
            {#each Object.values(category)[0] as question, questionIndex}
              <div
                class="editable-cell {questionAvailable(
                  categoryIndex,
                  questionIndex,
                )
                  ? ''
                  : 'not-available-cell'}"
                role="button"
                tabindex="0"
                on:click={() =>
                  toggleQuestionStatus(categoryIndex, questionIndex)}
                on:keydown={() => {}}
              >
                {question.question}
              </div>
            {/each}
          </div>
          <br />
        {/each}
      {/key}

      <div class="button-set">
        <button class="close-button hov-btn" on:click={closeEditorModal}
          >close</button
        >
      </div>
    </div>
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

  .aux-btns {
    position: absolute;
    top: 32px;
    left: 32px;
    display: flex;
    flex-direction: row;
    gap: 28px;
  }

  .back-button {
    font-family: 'Pixelify Sans', 'Comic Sans MS', 'Arial', sans-serif;
    padding: 8px 16px;
    background-color: #333;
    color: #f9f9f9;
    border: none;
    border-radius: 8px;
    font-size: 22px;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .back-button:hover {
    background-color: #54b3d6;
    color: #1a1a1a;
  }

  .file-buttons-container {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    background-color: #3e5e40;
    padding-inline: 8px 0px;
    border-radius: 8px;
  }

  .upload-container > input {
    font-family: 'Pixelify Sans', 'Comic Sans MS', 'Arial', sans-serif;
  }

  input::file-selector-button {
    font-family: 'Pixelify Sans', 'Comic Sans MS', 'Arial', sans-serif;
    background-color: #2b3a2c;
    color: #f9f9f9;
    border: none;
    border-radius: 8px;
    font-size: 22px;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .download-sample-button {
    font-size: 20px;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    padding-left: 32px;
    padding-right: 16px;
    border-left: 4px solid #eee;
    height: 100%;
    cursor: pointer;
    background-color: #555;
    border-end-end-radius: 8px;
    border-start-end-radius: 8px;
  }

  .questions-editor {
    position: absolute;
    bottom: 32px;
    left: 32px;
    display: flex;
    flex-direction: row;
    gap: 28px;
    font-family: 'Pixelify Sans', 'Comic Sans MS', 'Arial', sans-serif;
    padding: 8px 16px;
    background-color: #3b4154;
    color: #f9f9f9;
    border: none;
    border-radius: 8px;
    font-size: 22px;
    cursor: pointer;
    transition: background-color 0.3s;
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
    border-right: 16px solid #0a0a0a;
  }
  .board-container > * {
    border-right: none;
  }

  .dice-container {
    position: absolute;
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-bottom: 32px;
  }

  .dice {
    width: 120px;
    height: 120px;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 32px;
    font-size: 84px;
    border: 4px solid rgb(183, 170, 170);
    border-radius: 4px;
    user-select: none;
    cursor: pointer;
    background-color: #783333;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
  }

  .hov-btn {
    transition: transform 0.2s;
  }

  .hov-btn:hover {
    transform: scale(1.15);
  }

  .hov-btn:active {
    transform: scale(0.95);
  }

  .dice-result {
    margin-top: 16px;
    font-size: 32px;
  }

  /* board */

  .board {
    display: grid;
    grid-template-columns: repeat(5, 160px);
    grid-template-rows: repeat(5, 160px);
    gap: 16px;
    margin: 20px auto;
  }

  .cell {
    width: 160px;
    height: 160px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #000000;
    border: 2px solid #000000;
    border-radius: 16px;
    font-size: 48px;
  }

  .cell.active {
    background-color: #a0a0a0;
    color: rgb(0, 0, 0);
    font-weight: bold;
  }

  .cell.active.inactive-cell {
    background-color: #222 !important;
  }

  .cell.active.cat-0 {
    background-color: #ff8888;
  }

  .cell.active.cat-1 {
    background-color: #88aeff;
  }

  .cell.active.cat-2 {
    background-color: #ff88e1;
  }

  .cell.active.cat-3 {
    background-color: #ffd988;
  }

  .category-guide {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    background: #000;
    padding: 16px;
    margin-bottom: 16px;
    border-radius: 8px;
    color: #fff;
    font-size: 24px;
  }

  .catname-0 {
    color: #ff8888;
  }

  .catname-1 {
    color: #88aeff;
  }

  .catname-2 {
    color: #ff88e1;
  }

  .catname-3 {
    color: #ffd988;
  }

  .show-question-btn {
    font-family: 'Pixelify Sans', 'Comic Sans MS', 'Arial', sans-serif;
    margin-top: 32px;
    padding: 8px 16px;
    background-color: #64355b;
    color: #fff;
    border: 4px solid #391f34;
    border-radius: 4px;
    font-size: 28px;
    cursor: pointer;
    font-weight: 500;
    transition: transform 0.2s;
  }

  .show-question-btn:disabled {
    background-color: #333;
    border-color: #111;
  }

  .current-cell {
    border: 8px solid #000;
    border-radius: 4px;
    background-color: #fff;
    width: 100px;
    height: 100px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  /* scores */

  .score-container {
    flex: 1;
    flex-direction: row;
    flex-basis: 30%;
    width: 30%;
    font-size: 24px;
    background-color: #1e1a43;
    border-left: 16px solid #161426;
    padding: 32px;
  }

  .player-score {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 16px;
    font-size: 36px;
    padding-inline: 32px;
    padding-block: 36px;
    line-height: 18px;
    border-radius: 8px;
    background-color: #0b0920;
  }

  .score-update-btn {
    font-family: 'Pixelify Sans', 'Comic Sans MS', 'Arial', sans-serif;
    border: 0px solid #0b0920;
    width: 32px;
    align-items: center;
    border-radius: 4px;
    font-size: 28px;
    cursor: pointer;
    font-weight: 500;
    transition: transform 0.2s;
  }

  .score-holder {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 24px;
  }

  .add-player-btn {
    font-family: 'Pixelify Sans', 'Comic Sans MS', 'Arial', sans-serif;
    border: 0px solid #0b0920;
    padding-inline: 16px;
    padding-block: 8px;
    align-items: center;
    border-radius: 4px;
    font-size: 28px;
    cursor: pointer;
    font-weight: 500;
    transition: transform 0.2s;
  }

  .player-name-input {
    font-family: 'Pixelify Sans', 'Comic Sans MS', 'Arial', sans-serif;
    outline: none;
    background-color: #0b0920;
    color: #f9f9f9;
    border: none;
    border-bottom: 4px solid #fff;
    padding-block: 4px;
    font-size: 32px;
    width: 300px;
    transition: background-color 0.3s;
  }

  .new-player-box {
    margin-bottom: 64px;
  }

  /* modal */

  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.8);
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .modal {
    background: #1a1a1a;
    padding: 32px;
    border: 4px solid white;
    border-radius: 4px;
    text-align: center;
    align-items: center;
    color: #f9f9f9;
    max-width: 500px;
    width: 80%;
    font-size: 32px;
  }

  .button-set {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    gap: 32px;
  }

  .close-button {
    font-family: 'Pixelify Sans', 'Comic Sans MS', 'Arial', sans-serif;
    margin-top: 36px;
    padding: 8px 16px;
    background-color: #845454;
    color: #fff;
    border: 4px solid #463131;
    border-radius: 4px;
    font-size: 24px;
    cursor: pointer;
    font-weight: 500;
  }

  .option {
    font-family: 'Pixelify Sans', 'Comic Sans MS', 'Arial', sans-serif;
    background: #eee;
    color: #000;
    margin-top: 16px;
    padding: 8px 16px;
    border: 4px solid #000;
    border-radius: 8px;
    font-size: 24px;
    cursor: pointer;
  }

  .opt-correct {
    background-color: #5ed791;
  }

  .opt-incorrect {
    background-color: #db6464;
  }

  .opt-correct-answer {
    background-color: #db6464;
  }

  .editable-question {
    font-size: 20px;
    margin-top: 30px;
  }

  .editable-cell {
    text-align: start;
    background-color: #000;
    cursor: pointer;
    margin-bottom: 10px;
    padding: 10px;
    border-radius: 4px;
  }

  .not-available-cell {
    color: #444 !important;
  }

  .modal {
    max-height: 90vh;
    overflow: scroll;
  }
</style>
