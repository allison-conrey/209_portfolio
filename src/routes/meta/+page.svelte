<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import { computePosition, autoPlacement, offset } from '@floating-ui/dom';
    import Pie from '$lib/Pie.svelte';
    import { format } from 'd3-format';

    let width = 1000, height = 600;
    let margin = { top: 10, right: 10, bottom: 30, left: 20 };
    let usableArea = {
        top: margin.top,
        right: width - margin.right,
        bottom: height - margin.bottom,
        left: margin.left,
        width: width - margin.left - margin.right,
        height: height - margin.top - margin.bottom
    };
    
    let data = [];
    let commits = [];
    let xScale, yScale, rScale;
    let xAxis, yAxis, yAxisGridlines;
    let svg;

    // Tooltip variables
    let hoveredIndex = -1;
    let tooltipPosition = { x: 0, y: 0 };
    let commitTooltip;
    $: hoveredCommit = commits[hoveredIndex] ?? {};

    // Brush variables
    let brush = d3.brush()
        .extent([[usableArea.left, usableArea.top], [usableArea.right, usableArea.bottom]])
        .on('start brush end', brushed);
    let brushSelection = null;

    // Selected commits and lines
    $: selectedCommits = brushSelection
        ? commits.filter(isCommitSelected)
        : [];
    $: hasSelection = brushSelection && selectedCommits.length > 0;

    $: selectedLines = hasSelection
        ? selectedCommits.flatMap(d => d.lines)
        : commits.flatMap(d => d.lines);

    $: typeBreakdown = d3.rollups(
        selectedLines,
        v => v.length,
        d => d.type
    );

    $: pieData = Array.from(typeBreakdown, ([type, count]) => ({
        label: type,
        value: count
    }));

    // Define colors for each type
    const typeColors = {
        JavaScript: '#f39c12',
        Python: '#3498db',
        HTML: '#e74c3c',
        CSS: '#9b59b6',
        Other: '#95a5a6'
    };

    const percentFormat = format('.1%');

    onMount(async () => {
        data = await d3.csv('/loc.csv', (row) => ({
            ...row,
            line: +row.line,
            depth: +row.depth,
            length: +row.length,
            type: row.type, // Updated to use 'type' instead of 'language'
            date: new Date(row.date + 'T00:00' + row.timezone),
            datetime: new Date(row.datetime),
            hourFrac: new Date(row.datetime).getHours() + new Date(row.datetime).getMinutes() / 60,
        }));

        // Define scales for X, Y, and Radius
        xScale = d3.scaleTime().domain(d3.extent(data, d => d.datetime)).range([usableArea.left, usableArea.right]).nice();
        yScale = d3.scaleLinear().domain([0, 24]).range([usableArea.bottom, usableArea.top]);

        const [minLines, maxLines] = d3.extent(data, d => d.line);
        rScale = d3.scaleSqrt().domain([minLines, maxLines]).range([2, 30]);

        // Group data by commit and sort in descending order of totalLines
        commits = d3.groups(data, (d) => d.commit).map(([commit, lines]) => {
            let first = lines[0];
            return {
                id: commit,
                url: 'https://github.com/vis-society/lab-7/commit/' + commit,
                author: first.author,
                datetime: first.datetime,
                hourFrac: first.datetime.getHours() + first.datetime.getMinutes() / 60,
                totalLines: lines.length,
                lines
            };
        });
        
        // Sort commits in descending order so smaller dots are painted last
        commits = d3.sort(commits, d => -d.totalLines);
    });

    // Function to handle dot interactions for tooltip
    async function dotInteraction(index, evt) {
        let hoveredDot = evt.target;
        if (evt.type === 'mouseenter' || evt.type === 'focus') {
            hoveredIndex = index;

            // Compute tooltip position
            tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
                strategy: 'fixed',
                middleware: [
                    offset(5),
                    autoPlacement(),
                ],
            });
        } else if (evt.type === 'mouseleave' || evt.type === 'blur') {
            hoveredIndex = -1;
        }
    }

    // Brush event handler
    function brushed(event) {
        brushSelection = event.selection;
    }

    // Function to check if a commit is within the brush selection
    function isCommitSelected(commit) {
        if (!brushSelection) {
            return false;
        }
        const [[x0, y0], [x1, y1]] = brushSelection;
        const x = xScale(commit.datetime);
        const y = yScale(commit.hourFrac);
        return x0 <= x && x <= x1 && y0 <= y && y <= y1;
    }

    // Define and apply axes and gridlines
    $: {
        if (xAxis) d3.select(xAxis).call(d3.axisBottom(xScale));
        if (yAxis) d3.select(yAxis).call(
            d3.axisLeft(yScale).tickFormat((d) => String(d % 24).padStart(2, '0') + ':00')
        );

        if (yAxisGridlines) {
            d3.select(yAxisGridlines).call(
                d3.axisLeft(yScale).tickSize(-usableArea.width).tickFormat('')
            );
        }

        // Apply the brush to the SVG and raise dots
        if (svg) {
            d3.select(svg).call(brush);
            d3.select(svg).selectAll('.dots').raise();
        }
    }
</script>

<h2>Commits by Time of Day</h2>
<svg viewBox="0 0 {width} {height}" bind:this={svg}>
    <!-- Grid lines -->
    <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this="{yAxisGridlines}"></g>

    <!-- Axis groups -->
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis}></g>
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis}></g>

    <!-- Dots for the scatter plot with variable radius -->
    <g class="dots">
        {#each commits as commit, index}
        <g
            tabindex="0"
            role="button"
            aria-describedby="commit-tooltip"
            aria-label={`Commit by ${commit.author} on ${commit.datetime.toLocaleString("en", {dateStyle: "full"})}`}
            on:mouseenter={evt => dotInteraction(index, evt)}
            on:mouseleave={evt => dotInteraction(index, evt)}
            on:focus={evt => dotInteraction(index, evt)}
            on:blur={evt => dotInteraction(index, evt)}
        >
            <circle
                cx="{xScale(commit.datetime)}"
                cy="{yScale(commit.hourFrac)}"
                r="{rScale(commit.totalLines)}"
                fill="steelblue"
                fill-opacity="{hoveredIndex === index ? 1 : 0.7}"
                class:selected={isCommitSelected(commit)}
            />
        </g>
        {/each}
    </g>
</svg>

<!-- Tooltip element -->
<dl class="info tooltip" bind:this={commitTooltip} hidden={hoveredIndex === -1} style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px">
    <dt>Commit</dt>
    <dd><a href="{hoveredCommit.url}" target="_blank">{hoveredCommit.id}</a></dd>

    <dt>Date</dt>
    <dd>{hoveredCommit.datetime?.toLocaleString("en", {dateStyle: "full"})}</dd>

    <dt>Author</dt>
    <dd>{hoveredCommit.author}</dd>

    <dt>Lines Edited</dt>
    <dd>{hoveredCommit.totalLines}</dd>
</dl>

<!-- Display selected commits count -->
<p>{hasSelection ? `${selectedCommits.length} commits selected` : 'No commits selected'}</p>

<!-- Type Breakdown Legend -->
<div class="legend">
    <h3>Type Breakdown</h3>
    <ul>
        {#each typeBreakdown as [type, count]}
        <li>
            <span
                class="legend-color"
                style="background-color: {typeColors[type] || typeColors.Other}"
            ></span>
            {type}: {count} lines ({percentFormat(count / selectedLines.length)})
        </li>
        {/each}
    </ul>
</div>

<!-- Pie chart of type breakdown -->
<Pie data={pieData} />

<style>
    svg {
        overflow: visible;
    }

    .gridlines {
        stroke-opacity: 0.2;
    }

    dl.info {
        display: grid;
        grid-template-columns: auto auto;
        gap: 0.5em;
        margin: 0;
        padding: 0.5em;
        background-color: white;
        border: 1px solid #ddd;
        border-radius: 4px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        transition-duration: 500ms;
        transition-property: opacity, visibility;
    }

    .tooltip {
        position: fixed;
        background-color: white;
        border-radius: 8px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        padding: 1em;
    }

    dl.info[hidden] {
        opacity: 0;
        visibility: hidden;
    }

    circle {
        transition: transform 200ms, fill-opacity 200ms;
        transform-origin: center;
        transform-box: fill-box;
    }

    circle:hover {
        transform: scale(1.5);
    }

    /* Style for selected circles */
    .selected {
        fill: orange;
        stroke: black;
        stroke-width: 1px;
    }

    /* Styles for the legend */
    .legend {
        margin-top: 1em;
        font-size: 0.9em;
    }

    .legend ul {
        list-style: none;
        padding: 0;
        display: flex;
        flex-wrap: wrap;
    }

    .legend li {
        margin-right: 1em;
        display: flex;
        align-items: center;
    }

    .legend-color {
        width: 12px;
        height: 12px;
        margin-right: 0.5em;
        border-radius: 50%;
    }

    @keyframes marching-ants {
        to {
            stroke-dashoffset: -8;
        }
    }

    svg :global(.selection) {
        fill-opacity: 0.1;
        stroke: black;
        stroke-opacity: 0.7;
        stroke-dasharray: 5 3;
        animation: marching-ants 2s linear infinite;
    }
</style>
