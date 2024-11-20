<script>
    import * as d3 from 'd3';
  
    export let data = [];
    export let selectedIndex = -1;
  
    let colors = d3.scaleOrdinal(d3.schemeTableau10);
    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);
    let sliceGenerator = d3.pie().value((d) => d.value);
  
    $: arcData = sliceGenerator(data);
    $: arcs = arcData.map(d => arcGenerator(d));
  
    function toggleWedge(index, event) {
      if (!event.key || event.key === 'Enter') {
        selectedIndex = selectedIndex === index ? -1 : index;
      }
    }
  </script>
  
  <div class="container">
    <!-- Pie Chart SVG -->
    <svg viewBox="-50 -50 100 100" width="200" height="200">
      {#each arcs as arc, index}
        <path
          d="{arc}"
          fill="{colors(index)}"
          class:selected={selectedIndex === index}
          tabindex="0"
          role="button"
          aria-label="Select year {data[index].label}"
          on:click={e => toggleWedge(index, e)}
          on:keyup={e => toggleWedge(index, e)}
        />
      {/each}
    </svg>
  
    <!-- Legend -->
    <ul class="legend">
        {#each data as d, index}
          <li>
            <button
              style="--color: {colors(index)}"
              class:selected={selectedIndex === index}
              on:click={() => selectedIndex = index}
              on:keydown={(e) => e.key === 'Enter' && (selectedIndex = index)}
              aria-label="Select year {d.label}"
            >
              <span class="swatch"></span>
              {d.label} <em>({d.value})</em>
            </button>
          </li>
        {/each}
      </ul>      
  </div>
  
  <style>
    .container {
      display: flex;
      align-items: center;
      gap: 1em;
    }
  
    svg {
      max-width: 10em; /* Control the size of the pie chart */
      margin-block: 2em;
      overflow: visible;
    }
  
    path {
      transition: 200ms;
      outline: none;
      cursor: pointer;
    }
  
    /* Reduced scale for a more subtle effect */
    .selected {
      transform: scale(1.05) translateY(-4px);
    }
  
    /* Legend styling remains intact */
    .legend {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
      gap: 1em;
      padding: 1em;
      border: 1px solid #ddd;
      border-radius: 0.5em;
      width: 100%;
    }
  
    .legend li {
      display: flex;
      align-items: center;
      gap: 0.5em;
      min-width: 100px;
    }
  
    .swatch {
      display: inline-block;
      width: 1em;
      height: 1em;
      background-color: var(--color);
      border-radius: 50%;
    }
  </style>
  
    