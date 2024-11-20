<script>
  export let data = {}; // Data prop for each project
  export let hLevel = 2; // Heading level prop with default value of 2
</script>

<article>
  <svelte:element this={'h' + hLevel}>{data.title}</svelte:element>
  <img src={data.image} alt={data.title} />
  <p>{data.description}</p>
  <span class="year">c. {data.year}</span> <!-- Adds "c." before the year -->
</article>

<style>
  article {
      display: flex;
      flex-direction: column;
      gap: 1rem; /* Adds spacing between elements */
      margin: 0;
  }

  h1, h2 {
      line-height: 1.1;
      text-wrap: balance;
  }

  img {
      max-width: 100%;
      height: auto;
  }

  .year {
      font-family: Baskerville, serif;
      font-variant-numeric: oldstyle-nums;
  }
</style>





