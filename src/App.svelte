<script>
  import Introduction from './pages/introduction.svelte';
  import Section1 from './pages/section1.svelte';
  import Section2 from './pages/section2.svelte';

  let currentPage = 'Introduction'; // Default to the Introduction page

  function navigate(page) {
    currentPage = page;
  }
</script>

<div>
  {#if currentPage === 'Introduction'}
    <Introduction {navigate} />
  {:else if currentPage === 'Section1'}
    <Section1 {navigate} />
  {:else if currentPage === 'Section2'}
    <Section2 {navigate} />
  {/if}
</div>

<style>
  /*.container {
    font-family: sans-serif;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 0px 20px;
  }

  .container > * {
    width: 100%;
    max-width: 700px;
  }

  navigation bar style
  nav {
   text-align: center;
   word-spacing: 30px;
   padding: 10px;
   background-color: #f5f5dc;
  }

  navigation bar button style
  nav a {
   color: #000000;
   text-decoration: none;
  }
  */
</style>