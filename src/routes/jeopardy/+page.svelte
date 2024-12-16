<script lang="ts">
  import { writable } from 'svelte/store';

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

  const selectedQuestion = writable(defaultQuestion);
  const showModal = writable(false);

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

  const selectQuestion = (question: Question) => {
    selectedQuestion.set(question);
    question.visited = true;
    showModal.set(true);
  };

  const closeModal = () => {
    showModal.set(false);
    selectedQuestion.set(defaultQuestion);
  };
</script>

{#if !fileContent}
  <div class="container">
    <button class="back-button" on:click={() => window.history.back()}
      >{'< back'}</button
    >
    <h1>welcome to not-jeopardy</h1>
    <p>upload your custom game json</p>
    <input
      type="file"
      id="file-upload"
      accept=".json"
      on:change={handleFileUpload}
    />
  </div>
{/if}

{#if fileContent}
  <div class="jeopardy-container">
    <h1>{boardTitle}</h1>
    <div class="jeopardy-board">
      <div class="jeopardy-row">
        {#each categories as category}
          <div class="jeopardy-cell category">{category}</div>
        {/each}
      </div>
      <div class="">
        <div class="jeopardy-col">
          {#each questions as questionList}
            <div class="jeopardy-row">
              {#each questionList as question}
                <div
                  class="jeopardy-cell {question.visited ? 'visited' : ''}"
                  on:keydown={() => selectQuestion(question)}
                  on:click={() => selectQuestion(question)}
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
        <h2>{$selectedQuestion.question}</h2>
        <!-- <button
          class="answer-box"
          on:click={() => ($selectedQuestion.answerVisible = true)}
        >
          {$selectedQuestion.answerVisible
            ? $selectedQuestion.answer
            : 'Reveal Answer'}
        </button> -->
      {/if}
      <button class="close-button" on:click={closeModal}>Close</button>
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

  .jeopardy-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;
  }

  .jeopardy-board {
    display: grid;
    grid-template-rows: auto repeat(5, 1fr);
    grid-template-columns: repeat(5, 1fr);
    gap: 0.5rem;
    max-width: 1000px;
    width: 90%;
    display: flex;
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
    cursor: pointer;
    font-size: 1.5rem;
    font-weight: bold;
    width: 100%;
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
    color: #f9f9f9;
    max-width: 500px;
    width: 80%;
  }

  .answer-box {
    margin-top: 1rem;
    padding: 1rem;
    background: #333;
    color: #f9f9f9;
    border: 2px solid #54b3d6;
    border-radius: 8px;
    font-size: 1.2rem;
    cursor: pointer;
    transition: background 0.3s;
  }

  .answer-box:hover {
    background: #54b3d6;
    color: #000;
  }

  .close-button {
    margin-top: 1rem;
    padding: 0.5rem 1rem;
    background-color: #333;
    color: #f9f9f9;
    border: none;
    border-radius: 8px;
    font-size: 1rem;
    cursor: pointer;
  }

  .close-button:hover {
    background-color: #54b3d6;
    color: #1a1a1a;
  }
</style>
