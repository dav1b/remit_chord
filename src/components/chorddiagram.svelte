<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
  
    let container; // Reference for the SVG container
    let matrix = [];
    let countries = [];
  
    

    onMount(async () => {
        const data = await d3.csv('./data/trade_data2.csv');
        
        // Log the data to see what we're getting
        console.log('CSV Data:', data);
        
        // Extract countries from the first column
        countries = data.map(row => Object.values(row)[0]);
        
        // Convert the rest of the data into the matrix format
        matrix = data.map(row => {
            const values = Object.values(row);
            // Remove the country name and convert remaining values to numbers
            return values.slice(1).map(Number);
        });

        console.log('Countries:', countries);
        console.log('Matrix:', matrix);


      drawChordDiagram();
    });
  
    function drawChordDiagram() {
      const width = 1000, height = 1000, margin = 120, outerRadius = Math.min(width, height) * 0.5 - margin;
  
      const chord = d3.chord().padAngle(0.05).sortSubgroups(d3.descending);
      const arc = d3.arc().innerRadius(outerRadius).outerRadius(outerRadius + 20);
      const ribbon = d3.ribbon().radius(outerRadius);
  
      const color = d3.scaleOrdinal(d3.schemeCategory10).domain(countries);
  
      const chords = chord(matrix);
  
      const svg = d3.select(container)
        .append("svg")
        .attr("width", width)
        .attr("height", height)
        .append("g")
        .attr("transform", `translate(${width / 2},${height / 2})`);
  
      // Draw arcs
      const group = svg.append("g")
        .selectAll("g")
        .data(chords.groups)
        .enter().append("g"); //change
  
      group.append("path")
        .attr("d", arc)
        .style("fill", d => color(d.index))
        .style("stroke", "#000");
  
      // Add text labels
      group.append("text")
        .attr("dy", ".35em")
        .attr("transform", d => {
            const angle = (d.startAngle + d.endAngle) / 2 * 180 / Math.PI - 90;
            const rotation = angle > 90 ? angle + 180 : angle;
            return `rotate(${angle}) translate(${outerRadius + 40}) ${rotation > 90 ? "rotate(180)" : ""}`;
        })
        .style("text-anchor", d => {
            const angle = (d.startAngle + d.endAngle) / 2 * 180 / Math.PI;
            return angle > 90 && angle < 270 ? "end" : "start";
        })
        .selectAll("tspan")
        .data(d => countries[d.index].split(" "))
        .join("tspan")
        .attr("x", 0)
        .attr("dy", (d, i) => i === 0 ? 0 : "1.2em")
        .text(d => d);
  
      // Draw ribbons
      svg.append("g")
        .selectAll("path")
        .data(chords)
        .enter().append("path")
        .attr("d", ribbon)
        .style("fill", d => color(d.source.index))
        .style("opacity", 0.7);
    }
  </script>
  
  <div bind:this={container}></div>