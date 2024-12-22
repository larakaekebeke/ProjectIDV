<script>
  export let navigate;
</script>

<div class="container">
  <h1>Migration Patterns in Europe</h1>
  <div class="layout">
    <nav>
      <button on:click={() => navigate('Introduction')}>Introduction</button>
      <button on:click={() => navigate('Section1')}>Section 1</button>
      <button on:click={() => navigate('Section2')}>Section 2</button>
    </nav>
    <div class="content">
      <p>Welcome to the Introduction page! Here is some content about migration patterns in Europe.</p>
    </div>
  </div>
</div>

<style>
  .container {
    text-align: center;
    font-family: sans-serif;
  }

  .layout {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    align-items: flex-start;
  }

  nav {
    display: flex;
    flex-direction: column;
    margin-right: 20px; /* Space between the buttons and the content */
  }

  nav button {
    margin: 5px 0;
    padding: 10px;
    width: 150px;
    text-align: left;
    border: 1px solid #ccc;
    border-radius: 5px;
    background-color: #f5f5dc;
    cursor: pointer;
    font-size: 16px;
  }

  nav button:hover {
    background-color: #eaeaea;
  }

  .content {
    text-align: left; /* Align content to the left of the page */
  }
</style>
