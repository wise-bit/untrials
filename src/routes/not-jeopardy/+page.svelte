<script lang="ts">
  import { writable } from 'svelte/store';

  interface Question {
    points: string;
    question: string;
    answer: string;
    visited: boolean;
    showAnswer: boolean;
  }

  interface FinalQuestion extends Question {
    showQuestion: boolean;
  }

  interface Category {
    [categoryName: string]: Question[];
  }

  interface JeopardyData {
    [boardTitle: string]: Category[];
  }

  let fileContent: string | null = null;
  let boardTitle = '';
  let categories: string[] = [];
  let questions: Question[][] = [];

  const defaultQuestion: Question = {
    points: '',
    question: '',
    answer: '',
    visited: false,
    showAnswer: false,
  };

  const finalJeopardyQuestion: FinalQuestion = {
    points: '',
    question: '',
    answer: '',
    visited: false,
    showAnswer: false,
    showQuestion: false,
  };

  const selectedQuestion = writable(defaultQuestion);
  const finalQuestion = writable(finalJeopardyQuestion);
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

        const finalJeopardyLoad: FinalQuestion = {
          ...data['final-jeopardy-title-dont-change'][0][
            'default-category-dont-change'
          ][0],
          visited: false,
          showAnswer: false,
          showQuestion: false,
        };
        finalQuestion.set(finalJeopardyLoad);
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
    if ($showFinalJeopardy) {
      showFinalJeopardy.set(false);
      return;
    }
    location.href = '/';
  };

  function downloadSample() {
    const a = document.createElement('a');
    a.href = 'samples/trials-not-jeopardy-sample.json';
    a.download = 'sample.json';

    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
  }
</script>

<button class="back-button" on:click={() => goBack()}>{'< back'}</button>

{#if !fileContent}
  <div class="container">
    <h1>welcome to not-jeopardy</h1>
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

{#if fileContent && !$showFinalJeopardy}
  <div class="jeopardy-container">
    <h1>{boardTitle}</h1>
    <div class="jeopardy-board">
      <div class="jeopardy-col">
        {#each categories as category}
          <div class="jeopardy-cell category">{category}</div>
        {/each}
      </div>
      <div class="">
        <div class="jeopardy-col">
          {#each questions as questionList, i}
            <div class="jeopardy-row">
              {#each questionList as question, j}
                <div
                  class="jeopardy-cell jeopardy-question {question.visited
                    ? 'visited'
                    : ''}"
                  on:keydown={() => selectQuestion(question, i, j)}
                  on:click={() => selectQuestion(question, i, j)}
                  aria-label="Question for {question.points} points"
                  role="button"
                  tabindex="0"
                >
                  {question.points}
                </div>
              {/each}
            </div>
          {/each}
        </div>
      </div>
    </div>
    <div>
      <button
        class="final-jeopardy-button"
        on:click={() => showFinalJeopardy.set(true)}>final jeopardy</button
      >
    </div>
  </div>
{/if}

{#if $showModal}
  <div
    class="modal-overlay"
    role="button"
    tabindex="0"
    on:click={closeModal}
    on:keydown={closeModal}
  >
    <div
      class="modal"
      role="button"
      tabindex="0"
      on:click|stopPropagation
      on:keydown|stopPropagation
    >
      {#if $selectedQuestion}
        <h4>[{$selectedQuestion.points} points]</h4>
        <br />
        <h2>{$selectedQuestion.question}</h2>
        {#if $selectedQuestion.showAnswer}
          <p>{$selectedQuestion.answer}</p>
        {/if}
        <div class="final-jeopardy-buttons">
          <button
            class="reveal-button"
            on:click={() =>
              ($selectedQuestion.showAnswer = !$selectedQuestion.showAnswer)}
          >
            reveal answer
          </button>
          <button class="close-button" on:click={closeModal}>close</button>
        </div>
      {/if}
    </div>
  </div>
{/if}

{#if $showFinalJeopardy}
  <div class="final-jeopardy jeopardy-container">
    <h1>final jeopardy</h1>
    {#if $finalQuestion.showQuestion}
      <h2>{$finalQuestion.question}</h2>
    {/if}
    {#if $finalQuestion.showAnswer}
      <div>{$finalQuestion.answer}</div>
    {/if}
    <div class="final-jeopardy-buttons">
      <button
        class="question-reveal-button"
        on:click={() =>
          ($finalQuestion.showQuestion = !$finalQuestion.showQuestion)}
      >
        reveal question
      </button>
      <button
        class="reveal-button"
        on:click={() =>
          ($finalQuestion.showAnswer = !$finalQuestion.showAnswer)}
      >
        reveal answer
      </button>
    </div>
  </div>
{/if}

<style>
  :global(body) {
    margin: 0;
    font-family: 'Comic Sans MS', 'Arial', sans-serif;
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
    padding: 1rem;
  }

  .upload-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin: 1rem;
    padding: 1rem;
    text-align: center;
    border: 2px dashed #f8f8f8;
    border-radius: 1rem;
    margin-top: 3rem;
  }

  .download-sample-button {
    padding: 0.5rem 1rem;
    background-color: #333;
    color: #f9f9f9;
    border: none;
    border-radius: 1rem;
    font-size: 1rem;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .back-button {
    position: absolute;
    top: 1rem;
    left: 1rem;
    padding: 0.5rem 1rem;
    background-color: #333;
    color: #f9f9f9;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .back-button:hover {
    background-color: #54b3d6;
    color: #1a1a1a;
  }

  button {
    font-family: 'Comic Sans MS', 'Arial', sans-serif;
  }

  button:hover {
    background-color: #1e1e1e;
    color: #d1d1d1;
  }

  .jeopardy-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;
  }

  .final-jeopardy {
    max-width: 700px;
    width: 90%;
    background: black;
    border-radius: 2rem;
    padding: 3rem;
  }

  .jeopardy-board {
    display: grid;
    grid-template-rows: auto repeat(5, 1fr);
    grid-template-columns: repeat(5, 1fr);
    gap: 0.5rem;
    max-width: 1000px;
    width: 90%;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  .jeopardy-row {
    display: flex;
    flex-direction: row;
    justify-content: center;
    flex-wrap: wrap;
    margin-bottom: 10px;
  }

  .jeopardy-col {
    display: flex;
    flex-direction: row;
    align-items: center;
    margin-right: 10px;
  }

  .jeopardy-cell {
    background: #333;
    color: #fff;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 1rem;
    margin: 0.2rem;
    border: 1px solid #bbccd2;
    border-radius: 10px;
    text-align: center;
    font-size: 1.5rem;
    font-weight: bold;
    width: 100%;
  }

  .jeopardy-question {
    cursor: pointer;
  }

  .visited {
    background: #0e0e0e;
    color: #484848;
  }

  .jeopardy-cell:hover {
    background: #54b3d6;
    color: #000;
  }

  .category {
    background: #54b3d6;
    color: #000;
    font-weight: bold;
  }

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
    padding: 2rem;
    border-radius: 10px;
    text-align: center;
    align-items: center;
    color: #f9f9f9;
    max-width: 500px;
    width: 80%;
  }

  .final-jeopardy-buttons {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    gap: 1rem;
  }

  .reveal-button {
    margin-top: 1rem;
    padding: 0.5rem 1rem;
    background-color: #2b8e40;
    color: #fff;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    cursor: pointer;
    font-weight: 500;
  }

  .close-button {
    margin-top: 1rem;
    padding: 0.5rem 1rem;
    background-color: #ab4444;
    color: #fff;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    cursor: pointer;
    font-weight: 500;
  }

  .final-jeopardy-button {
    margin-top: 1rem;
    padding: 0.5rem 1rem;
    background-color: #335f8d;
    color: #fff;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    cursor: pointer;
    font-weight: 500;
  }

  .question-reveal-button {
    margin-top: 1rem;
    padding: 0.5rem 1rem;
    background-color: #335f8d;
    color: #fff;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    cursor: pointer;
    font-weight: 500;
  }

  .close-button:hover {
    background-color: #54b3d6;
    color: #1a1a1a;
  }
</style>
