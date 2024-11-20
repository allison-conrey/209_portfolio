<script>
    import projects from '$lib/projects.json';
    import Project from '$lib/Project.svelte';
    import Pie from '$lib/Pie.svelte';
    import * as d3 from 'd3';
  
    let query = '';
    let selectedYearIndex = -1;
    let selectedYear;
  
    // Reactive variable for selected year
    $: selectedYear = selectedYearIndex > -1 ? pieData[selectedYearIndex].label : null;
  
    let pieData;
    let filteredProjects;
    let filteredByYear;
  
    $: filteredProjects = projects.filter((project) => {
      let values = Object.values(project).join('\n').toLowerCase();
      return values.includes(query.toLowerCase());
    });
  
    $: {
      let rolledData = d3.rollups(
        filteredProjects,
        (v) => v.length,
        (d) => d.year
      );
      pieData = rolledData.map(([year, count]) => ({
        value: count,
        label: year
      }));
    }
  
    $: filteredByYear = filteredProjects.filter((project) => {
      return selectedYear ? project.year === selectedYear : true;
    });
  </script>
  
  <h1>{filteredByYear.length} Projects</h1>
  
  <div class="chart-container">
    <input
      type="search"
      bind:value="{query}"
      aria-label="Search projects"
      placeholder="🔍 Search projects…"
    />
    
    <Pie data="{pieData}" bind:selectedIndex="{selectedYearIndex}" />
  </div>
  
  <div class="projects">
    {#each filteredByYear as project}
      <Project data={project} />
    {/each}
  </div>
  
  <style>
    .chart-container {
      display: flex;
      gap: 1em;
      max-width: 500px;
      margin: 0 auto;
      flex-direction: column;
    }
  </style>  