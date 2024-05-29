<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import { simulateMultipleGames } from '../lib/montyHallSimulation';

    let nGames = 1000;
    let resultsWithSwitching = [];
    let resultsWithoutSwitching = [];

    const runSimulation = () => {
        resultsWithSwitching = simulateMultipleGames(nGames, true);
        resultsWithoutSwitching = simulateMultipleGames(nGames, false);
        drawChart();
    };

    const drawChart = () => {
        const games = d3.range(1, nGames + 1);
        const cumulativeSwitching = d3.cumsum(resultsWithSwitching);
        const cumulativeNotSwitching = d3.cumsum(resultsWithoutSwitching);

        const svg = d3.select("#chart").attr("width", 800).attr("height", 400); // Adjust size
        svg.selectAll("*").remove();

        const margin = { top: 20, right: 30, bottom: 50, left: 60 }; // Adjusted margins
        const width = +svg.attr("width") - margin.left - margin.right;
        const height = +svg.attr("height") - margin.top - margin.bottom;

        const x = d3.scaleLinear().domain([0, nGames]).range([0, width]);
        const y = d3.scaleLinear().domain([0, nGames]).range([height, 0]);

        const g = svg.append("g").attr("transform", `translate(${margin.left},${margin.top})`);

        g.append("g")
            .attr("transform", `translate(0,${height})`)
            .call(d3.axisBottom(x).ticks(10).tickSize(-height));

        g.append("g")
            .call(d3.axisLeft(y).ticks(10).tickSize(-width));

        const lineSwitching = g.append("path")
            .datum(games)
            .attr("fill", "none")
            .attr("stroke", "blue")
            .attr("stroke-width", 1.5)
            .attr("d", d3.line().x(d => x(d)).y(d => y(cumulativeSwitching[d - 1])));

        const lineNotSwitching = g.append("path")
            .datum(games)
            .attr("fill", "none")
            .attr("stroke", "red")
            .attr("stroke-width", 1.5)
            .attr("d", d3.line().x(d => x(d)).y(d => y(cumulativeNotSwitching[d - 1])));

        // Adding a legend
        const legend = svg.append("g")
            .attr("transform", `translate(${width - 80}, ${20})`);

        legend.append("rect")
            .attr("width", 100)
            .attr("height", 50)
            .attr("fill", "white")
            .attr("stroke", "black");

        legend.append("line")
            .attr("x1", 10)
            .attr("y1", 15)
            .attr("x2", 30)
            .attr("y2", 15)
            .attr("stroke", "blue")
            .attr("stroke-width", 2);

        legend.append("text")
            .attr("x", 35)
            .attr("y", 15)
            .attr("dy", ".35em")
            .style("text-anchor", "start")
            .text("Switching");

        legend.append("line")
            .attr("x1", 10)
            .attr("y1", 35)
            .attr("x2", 30)
            .attr("y2", 35)
            .attr("stroke", "red")
            .attr("stroke-width", 2);

        legend.append("text")
            .attr("x", 35)
            .attr("y", 35)
            .attr("dy", ".35em")
            .style("text-anchor", "start")
            .text("Not Switching");

        // Adding interactivity - highlighting and tooltip
        const focus = g.append("g").style("display", "none");

        focus.append("circle").attr("r", 4.5);

        const tooltip = svg.append("text")
            .attr("class", "tooltip")
            .attr("x", 0)
            .attr("y", 0)
            .style("opacity", 0)
            .style("font-size", "12px")
            .style("fill", "black")
            .attr("text-anchor", "middle")
            .attr("alignment-baseline", "middle");

        // Interaction event handlers
        svg.on("mouseover", function() { focus.style("display", null); tooltip.style("opacity", 1); })
            .on("mouseout", function() { focus.style("display", "none"); tooltip.style("opacity", 0); })
            .on("mousemove", mousemove);

        function mousemove() {
            const mouseX = d3.event.pageX - svg.node().getBoundingClientRect().left - margin.left;
            const mouseY = d3.event.pageY - svg.node().getBoundingClientRect().top - margin.top;
            const xValue = x.invert(mouseX);
            const yValue = y.invert(mouseY);
            focus.attr("transform", `translate(${x(mouseX)},${y(cumulativeSwitching[Math.round(xValue) - 1])})`);
            tooltip.text(`x: ${xValue.toFixed(2)}, y: ${yValue.toFixed(2)}`)
                .attr("x", mouseX)
                .attr("y", mouseY - 10);
        }

        // Adding hover effect to the lines
        const hoverEffect = (line) => {
            line.on("mouseover", function() {
                d3.select(this).attr("stroke-width", 3); // Increase stroke width on hover
            }).on("mouseout", function() {
                d3.select(this).attr("stroke-width", 1.5); // Reset stroke width on mouseout
            });
        };

        hoverEffect(lineSwitching);
        hoverEffect(lineNotSwitching);
    };

    onMount(runSimulation);
</script>

<div class="section">
    <h2>Interactive Histogram</h2>
    <p>Number of games: <input type="number" bind:value={nGames} min="100" max="10000" step="100" /></p>
    <button on:click={runSimulation}>Run Simulation</button>
    <svg id="chart"></svg>
    <p class="description">
        This chart shows the cumulative wins over multiple Monty Hall game simulations. 
        The blue line represents the cumulative wins when the contestant always switches their choice, 
        while the red line represents the cumulative wins when the contestant never switches. 
        As the number of games increases, you can observe that switching doors leads to more wins on average.
    </p>
</div>

<style>
  #chart {
    display: block;
    margin: 0 auto;
  }
  
  .tooltip {
    visibility: hidden;
  }

  .section {
    height: 100vh; /* Full height of the viewport */
    overflow-y: auto; /* Enable scrolling within each section */
    display: flex; /* Use flexbox to center content vertically */
    flex-direction: column; /* Stack content vertically */
    justify-content: center; /* Center content horizontally */
    align-items: center; /* Center content vertically */
    text-align: center; /* Center text horizontally */
    padding: 20px; /* Add padding for better readability */
    background: linear-gradient(to bottom, #ff7e5f, #feb47b); /* Gradient background */
    color: white; /* Text color for better readability against gradient */
  }

  .section h2 {
    font-size: 2.5rem; /* Larger font size for headings */
  }

  .section p {
    font-size: 1.5rem; /* Larger font size for paragraphs */
  }

  .description {
    margin-top: 20px;
    font-size: 1.2rem;
    max-width: 800px;
  }
</style>
