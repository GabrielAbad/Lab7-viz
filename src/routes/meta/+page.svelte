<script>
import * as d3 from "d3";
import { onMount } from "svelte";
import {
    computePosition,
    autoPlacement,
    offset,
} from '@floating-ui/dom';
import Bar from '$lib/Bar.svelte';


let data = [];
let width = 1000, height = 600;
let commits = [];
let xScale, yScale;
let margin = {top: 10, right: 10, bottom: 30, left: 20};
let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left
};
usableArea.width = usableArea.right - usableArea.left;
usableArea.height = usableArea.bottom - usableArea.top;
let xAxis, yAxis, yAxisGridlines;
let hoveredIndex = -1;
let tooltipPosition = {x: 0, y: 0};
let commitTooltip;
$: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};
let clickedCommits = [];

async function dotInteraction(index, evt) {
    let hoveredDot = evt.target;
    if (evt.type === "mouseenter") {
        hoveredIndex = index;
        tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
            strategy: "fixed",
            middleware: [
                offset(5),
                autoPlacement()
            ],
        });
    }
    else if (evt.type === "mouseleave") {
        hoveredIndex = -1;
    }
	else if (evt.type === "click") {
		let commit = commits[index]
		if (!clickedCommits.includes(commit)) {
			// Add the commit to the clickedCommits array
			clickedCommits = [...clickedCommits, commit];
		}
		else {
				// Remove the commit from the array
				clickedCommits = clickedCommits.filter(c => c !== commit);
		}
	}

}

onMount(async () => {
    data = await d3.csv("./loc.csv", row => ({
        ...row,
        line: Number(row.line),
        depth: Number(row.depth),
        length: Number(row.length),
        date: new Date(row.date + "T00:00" + row.timezone),
        datetime: new Date(row.datetime)
    }));
    
    commits = d3.groups(data, d => d.commit).map(([commit, lines]) => {
        let first = lines[0];
        let {author, date, time, timezone, datetime} = first;
        let ret = {
            id: commit,
            url: "https://github.com/USERNAME/REPO/commit/" + commit,
            author, date, time, timezone, datetime,
            hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
            totalLines: lines.length
        };

        Object.defineProperty(ret, "lines", {
            value: lines,
            configurable: true,
            writable: true,
            enumerable: false,
        });

        return ret;
    });

	commits = d3.sort(commits, d => -d.totalLines);

});

$: minDate = d3.min(commits.map(d => d.date));
$: maxDate = d3.max(commits.map(d => d.date));
$: maxDatePlusOne = new Date(maxDate);
$: maxDatePlusOne.setDate(maxDatePlusOne.getDate() + 1);

$: xScale = d3.scaleTime()
              .domain([minDate, maxDatePlusOne])
              .range([0, width])
              .nice();

$: yScale = d3.scaleLinear()
              .domain([24, 0])
              .range([height, 0]);

$: {
    if (xAxis && yAxis) {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
    }
}

$: {
    if (yAxisGridlines) {
        d3.select(yAxisGridlines).call(
            d3.axisLeft(yScale)
              .tickFormat("")
              .tickSize(-usableArea.width)
        );
    }
}

$: rScale = d3.scaleSqrt()
                .domain(d3.extent(commits.map(d=>d.totalLines)))
                .range([2, 30]);


$: allTypes = Array.from(new Set(data.map(d => d.type)));
$: selectedLines = (clickedCommits.length > 0 ? clickedCommits : commits).flatMap(d => d.lines);
$: selectedCounts = d3.rollup(
    selectedLines,
    v => v.length,
    d => d.type
);
$: languageBreakdown = allTypes.map(type => [type, selectedCounts.get(type) || 0]);



</script>

<svelte:head>
<title>Meta</title>
</svelte:head>
<h1>Meta</h1>

<h3>Heading</h3>
<svg viewBox="0 0 {width} {height}">
	<Bar data={languageBreakdown} width={width} />
    <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />

    {#if commits.length > 0}
        <g class="dots">
            {#each commits as commit, index}
                <circle
					on:click={ evt => dotInteraction(index, evt) }
					class:selected={ clickedCommits.includes(commit) }
					on:mouseenter={evt => dotInteraction(index, evt)}
					on:mouseleave={evt => dotInteraction(index, evt)}
					cx={ xScale(commit.datetime) }
					cy={ yScale(commit.hourFrac) }
					r={ rScale(commit.totalLines) }
					fill="steelblue"
					fill-opacity="0.5"
				/>
            {/each}
        </g>
    {/if}
</svg>

<dl class="info tooltip" 
    bind:this={commitTooltip}
    hidden={hoveredIndex === -1} 
    style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px">
    <dt>Commit</dt>
    <dd><a href={hoveredCommit.url} target="_blank">{hoveredCommit.id}</a></dd>

    <dt>Date</dt>
    <dd>{hoveredCommit.datetime?.toLocaleString("en", {dateStyle: "full"})}</dd>

    <dt>Author</dt>
    <dd>{hoveredCommit.author}</dd>

    <dt>Time</dt>
    <dd>{hoveredCommit.time}</dd>
</dl>

<section>
    <h2>Summary</h2>
    <dl class="stats">
    <dt>Total <abbr title="Lines of code">LOC</abbr></dt>
    <dd>{data.length}</dd>
    <dt>Files</dt>
    <dd>{d3.groups(data, d => d.file).length}</dd>
    <dt>Commits</dt>
    <dd>{d3.groups(data, d => d.commit).length}</dd>
    </dl>
</section>

<style>
dl{
    display: grid;
    grid-template-columns: auto;
}
dt{
    grid-row: 1;
    font-family: inherit;
    font-weight: bold;
    color: var(--border-gray);
    text-transform: uppercase;
}
dd{
    font-family: inherit;
    font-weight: bold;
}
section{
    border-width:0.15em;
    border-style:solid;
    border-color:var(--border-gray);
    padding-left: 1em;
    padding-right: 1em;
}
svg {
    overflow: visible;
}
.gridlines {
    stroke-opacity: .2;
}

/* Tooltip styles */
.info {
    display: grid;
    margin: 0;
    grid-template-columns: 2;
    background-color: oklch(100% 0% 0 / 80%);
    box-shadow: 1px 1px 3px 3px gray;
    border-radius: 5px;
    backdrop-filter: blur(10px);
    padding: 10px;
    transition-duration: 500ms;
    transition-property: opacity, visibility;
}

.info dt {
    grid-column: 1;
    grid-row: auto;
}

.info dd {
    grid-column: 2;
    grid-row: auto;
    font-weight: 400;
}

.tooltip {
    position: fixed;
    z-index: 100;
}

.info[hidden]:not(:hover, :focus-within) {
    opacity: 0;
    visibility: hidden;
}

/* Circle hover effects */
circle {
    transition: 200ms;
    transform-origin: center;
    transform-box: fill-box;
}

circle:hover {
    transform: scale(1.5);
}

.selected {
    fill: var(--color-accent);
}

</style>