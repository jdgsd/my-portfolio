<script>
    import * as d3 from "d3";

    import { onMount } from "svelte";
    let data = [];
    let commits = [];
    let longestLine = null;
    let files = [];

    import {
	computePosition,
	autoPlacement,
	offset,
    } from '@floating-ui/dom';

    /*Commits Chart*/
    let width = 1000, height = 600;
    
    $: minDate = d3.min(commits.map(d => d.date));
    $: maxDate =  d3.max(commits.map(d => d.date));
    $: maxDatePlusOne = new Date(maxDate);
    $: maxDatePlusOne.setDate(maxDatePlusOne.getDate() +1);

    $: xScale = d3.scaleTime()
        .domain([minDate, maxDatePlusOne])
        .range([usableArea.left, usableArea.right])
        .nice();
    
    $: yScale = d3.scaleLinear()
        .domain([24,0])
        .range([usableArea.bottom, usableArea.top]);
    
    let margin = {top: 10, right: 10, bottom: 30, left: 20};
    let usableArea = {
        top: margin.top,
        right: width - margin.right,
        bottom: height - margin.bottom,
        left: margin.left
    };
    usableArea.width = usableArea.right - usableArea.left;
    usableArea.height = usableArea.bottom - usableArea.top;

    let xAxis, yAxis;
    let yAxisGridlines;

    $: {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
        d3.select(yAxisGridlines).call(
            d3.axisLeft(yScale)
                .tickFormat("")
                .tickSize(-usableArea.width)
        );
    }

    let hoveredIndex = -1;
    $: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};

    let cursor = {x: 0, y: 0};

    let commitTooltip;
    let tooltipPosition = {x: 0, y: 0};

    async function dotInteraction (index, evt) {
        let hoveredDot = evt.target;
        if (evt.type === "mouseenter") {
            hoveredIndex = index;
            cursor = {x: evt.x, y: evt.y};
            tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
                strategy: "fixed", // because we use position: fixed
                middleware: [
                    offset(5), // spacing from tooltip to dot
                    autoPlacement() // see https://floating-ui.com/docs/autoplacement
                ],
            });        }
        else if (evt.type === "mouseleave") {
            hoveredIndex = -1
        }
    }

    
    onMount(async () => {
        data = await d3.csv("/loc.csv", row => ({
            ...row,
            line: Number(row.line), // or just +row.line
            depth: Number(row.depth),
            length: Number(row.length),
            date: new Date(row.date + "T00:00" + row.timezone),
            datetime: new Date(row.datetime)
        }));

        longestLine = d3.max(data, d => d.length);

        files = d3.group(data, d => d.file).size;

        commits = d3.groups(data, d => d.commit).map(([commit, lines]) => {
            let first = lines[0];
            let {author, date, time, timezone, datetime} = first;
            let ret = {
                id: commit,
                url: "https://github.com/vis-society/lab-7/commit/" + commit,
                author, date, time, timezone, datetime,
                hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
                totalLines: lines.length
            };

            // Like ret.lines = lines
            // but non-enumerable so it doesn’t show up in JSON.stringify
            Object.defineProperty(ret, "lines", {
                value: lines,
                configurable: true,
                writable: true,
                enumerable: false,
            });

            return ret;
        });
        console.log(commits)
    });
</script>

<body>
    <h1> Meta</h1>
    <p>
        This page includes stats about the code of this website.
    </p>

    <dl class="stats">
        <div class="stat">
            <dt>Commits</dt>
            <dd>{commits.length}</dd>
        </div>
        <!-- <div class="stat">
            <dt>Files</dt>
            <dd>{files.length}</dd>
        </div> -->
        <div class="stat">
            <dt>Total <abbr title="Lines of code">LOC</abbr></dt>
            <dd>{data.length}</dd>
        </div>
        <div class="stat">
            <dt>Longest Line</dt>
        <dd>{longestLine || 0} characters</dd>
        </div>
    </dl>

    <div class= chartdisplay>
        <div class= chart>
            <h3>Commits by Time of Day</h3>
            <svg viewBox="0 0 {width} {height}">
                <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
                <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
                <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
                <g class="dots">
                    {#each commits as commit, index }
                        <circle
                            on:mouseenter={evt => dotInteraction(index, evt)}
                            on:mouseleave={evt => dotInteraction(index, evt)}
                            cx={ xScale(commit.datetime) }
                            cy={ yScale(commit.hourFrac) }
                            r="5"
                            fill="steelblue"
                        />
                    {/each}
                </g>   
            </svg>
        </div>

        <div class= detail>
            <dl class="info tooltip" hidden={hoveredIndex === -1} style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px" 
                bind:this={commitTooltip}>
                <dt>Commit</dt>
                <dd><a href="{ hoveredCommit.url }" target="_blank">{ hoveredCommit.id }</a></dd>
            
                <dt>Date</dt>
                <dd>{ hoveredCommit.datetime?.toLocaleString("en", {dateStyle: "full"}) }</dd>
            
                <!-- Add: Time, author, lines edited -->
            </dl>
        </div>
    </div>
</body>

<style>
    /* .chartdisplay {
        display: grid;
        grid-template-columns: 3fr 6fr;
    }

    .chart {
        grid-column: 2;
    }

    .detail {
        grid-column: 1;
    } */
    
    svg {
        overflow: visible;
    }

    .gridlines {
        stroke-opacity: .2;
    }

    dl.info {
        display: grid;
        grid-template-columns: 1fr 2fr;
        transition-duration: 500ms;
        transition-property: opacity, visibility;

        &[hidden]:not(:hover, :focus-within) {
            opacity: 0;
            visibility: hidden;
        }
    }

    dl.info dt {
        grid-column: 1;
        color: #9A9A9A;
    }

    dl.info dd {
        grid-column: 2;
    }

    .tooltip {
        position: fixed;
        top: 1em;
        left: 1em;
        background-color: #F6F6F6;
        box-shadow: 0 2px 10px rgba(0,0,0,0.2);
        padding: 15px;
        border: 5px;
        margin: 10px;
        max-width: 200px;
    }

    circle {
        transition: 200ms;
        &:hover {
            transform: scale(1.5);
        }
        transform-origin: center;
        transform-box: fill-box;
    }

</style>