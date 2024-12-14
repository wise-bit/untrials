<script lang="ts">
  let fileContent: string | null = null;

  const handleFileUpload = async (event: Event) => {
    const input = event.target as HTMLInputElement;
    if (input.files && input.files[0]) {
      const file = input.files[0];
      const reader = new FileReader();

      reader.onload = (e) => {
        fileContent = e.target?.result as string;
        console.log(JSON.parse(fileContent));
      };

      reader.readAsText(file);
    }
  };

  const goBack = () => {
    window.history.back();
  };
</script>

<div class="container">
  <button class="back-button" on:click={goBack}>{'< back'}</button>
  <h1>welcome to not-jeopardy</h1>
  <p>upload your custom game json</p>
  <input
    type="file"
    id="file-upload"
    accept=".json"
    on:change={handleFileUpload}
  />
</div>

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

  button, input {
    font-family: 'Comic Sans MS', 'Arial', sans-serif;
  }

  .container {
    text-align: center;
    padding: 1rem;
  }

  h1 {
    font-size: 2.5rem;
    color: #54b3d6;
    margin-bottom: 1rem;
  }

  p {
    font-size: 1.2rem;
    margin-bottom: 1.5rem;
    color: #d1d1d1;
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

  #file-upload {
    padding: 0.5rem;
    border-radius: 8px;
    border: 2px solid #54b3d6;
    background-color: #333;
    color: #f9f9f9;
    cursor: pointer;
    transition: all 0.3s;
    font-size: 1rem;
  }

  #file-upload:hover {
    background-color: #54b3d6;
    color: #1a1a1a;
    border-color: #1a1a1a;
  }
</style>
