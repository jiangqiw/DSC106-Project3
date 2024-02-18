<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let data = [];
  let dataChina = [];
  let selectedYear = 2002;
  let filteredData = [];
  let filteredDataChina = []
  let maxYValue = 0;
  let maxYValueChina = 0;
  var minValue = 1; // Minimum value to ensure hit area has height
  var hitAreaHeight = 20; // Minimum hit area height


  onMount(async () => {
    fetch('/renewable_energy_data_US_prop.json')
      .then(response => {
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        return response.json();
      })
      .then(fetchedData => {
        data = fetchedData;
        updateData(selectedYear);
        maxYValue = d3.max(data, d => 
          d3.max(Object.entries(d), ([key, value]) => 
            key.includes('_per_capita') ? value : 0
          )
        );

      })
      .catch(error => {
        console.error('There has been a problem with your fetch operation:', error);
      });
    fetch('/renewable_energy_data_China_prop.json')
      .then(response => {
        if (!response.ok) {
          throw new Error('Network response was not ok for China dataset');
        }
        return response.json();
      })
      .then(fetchedDataChina => {
        dataChina = fetchedDataChina;
        updateDataChina(selectedYear);
        maxYValueChina = d3.max(dataChina, d => 
          d3.max(Object.entries(d), ([key, value]) => 
            key.includes('_per_capita') ? value : 0
          )
        );
      })
      .catch(error => {
        console.error('Problem with fetching China data:', error);
      });
  });

  $: if (data.length > 0 && dataChina.length > 0) {
    updateData(selectedYear);
    updateDataChina(selectedYear);
  }

  function drawBarChart(filteredData) {
    d3.select("#bar-chart").selectAll("*").remove();

    const margin = { top: 25, right: 20, bottom: 85, left: 40 },
          width = 650 - margin.left - margin.right,
          height = 550 - margin.top - margin.bottom;

    const svg = d3.select("#bar-chart")
                  .append("svg")
                  .attr("width", width + margin.left + margin.right)
                  .attr("height", height + margin.top + margin.bottom)
                  .append("g")
                  .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

    const xScale = d3.scaleBand()
                     .range([0, width])
                     .padding(0.1)
                     .domain(filteredData.map(d => d.category));

    const yScale = d3.scaleLinear()
                     .domain([0, maxYValue])
                     .range([height, 0]);

    const tooltip = d3.select('body').selectAll('.tooltip').data([null]); // Ensures only one tooltip exists
    tooltip.enter()
      .append('div')
      .attr('class', 'tooltip')
      .style('opacity', 0) // Initial opacity set to 0
      .merge(tooltip) // Merge enter and update selections
      .style('position', 'absolute')
      .style('text-align', 'center')
      .style('width', 'auto')
      .style('padding', '40px 40px')
      .style('font', '14px sans-serif')
      .style('color', '#ffffff')
      .style('background', '#4a5568')
      .style('border', '1px solid #cbd5e0')
      .style('border-radius', '8px')
      .style('pointer-events', 'none')
      .style('box-shadow', '0px 0px 10px rgba(0, 0, 0, 0.5)')
      .style('z-index', '10000');


    svg.selectAll(".bar")
       .data(filteredData)
       .enter().append("rect")
       .attr("class", "bar")
       .attr("x", d => xScale(d.category))
       .attr("width", xScale.bandwidth())
       .attr("y", d => yScale(d.value))
       .attr("height", d => height - yScale(d.value))
       .attr("fill", "#007bff")
       .on("mouseover", (event, d) => {
         tooltip.transition()
                .duration(200)
                .style("opacity", .9);
         tooltip.html(`Country: USA<br>Year: ${selectedYear}<br>Type: ${d.category}<br>Amount: ${d.value + "%"}`)
                .style("left", (event.pageX) + "px")
                .style("top", (event.pageY - 28) + "px");
       })
       .on("mouseout", () => {
         tooltip.transition()
                .duration(500)
                .style("opacity", 0);
       });
    var minValue = 1; // Minimum value to ensure hit area has height
var hitAreaHeight = 20; // Minimum hit area height

// Add hit areas for each bar
  svg.selectAll(".bar-hit-area")
      .data(filteredData)
      .enter().append("rect")
      .attr("class", "bar-hit-area")
      .attr("x", d => xScale(d.category))
      .attr("y", d => yScale(d.value) - 50) // Extend the hit area 50px above the bar
      .attr("width", xScale.bandwidth())
      .attr("height", d => (height - yScale(d.value)) + 100) // Increase the height to include the extended area
      .style("fill", "none") // Keep the hit area invisible
      .style("pointer-events", "all") // Ensure it can catch mouse events
      .on("mouseover", (event, d) => {
          tooltip.transition()
               .duration(200)
               .style("opacity", .9);
          tooltip.html(`Country: China<br>Year: ${selectedYear}<br>Type: ${d.category}<br>Amount: ${d.value + "%"}`)
               .style("left", (event.pageX) + "px")
               .style("top", (event.pageY - 28) + "px");
      })
      .on("mouseout", () => {
          tooltip.transition()
               .duration(500)
               .style("opacity", 0);
      });
    
    // X Axis
    svg.append("g")
       .attr("transform", `translate(0, ${height})`)
       .call(d3.axisBottom(xScale))
       .selectAll("text")
       .style("text-anchor", "end")
       .attr("dx", "-.8em")
       .attr("dy", ".15em")
       .attr("transform", "rotate(-15)") // Rotating column names by 45 degrees
       .style("font-size", "12px")
       .style("font-weight", "bold");

    // Adding X Axis Name
    svg.append("text")
       .attr("transform", `translate(${width / 2}, ${height + margin.top + 50})`)
       .style("text-anchor", "middle")
       .text("Energy Consumption Type(per capita)")
       .style("font-weight", "bold");

    // Y Axis
    svg.append("g")
       .call(d3.axisLeft(yScale));

    // Adding Y Axis Name
    svg.append("text")
    .attr("transform", "rotate(-90)")
    .attr("y", 0 - margin.left)
    .attr("x", 0 - (height / 2))
    .attr("dy", "1em")
    .style("text-anchor", "middle")
    .style("font-size", "15px") // Adjust font size here
    .text("Consumption Percentage")
    .style("font-weight", "bold");


    // Adding a Title
    svg.append("text")
       .attr("x", (width / 2))
       .attr("y", 5 - (margin.top / 2))
       .attr("text-anchor", "middle")
       .style("font-size", "20px")
       .text(`Renewable Energy Consumption by Type (USA) in ${selectedYear}`)
       .style("font-weight", "bold");
  }

  function drawBarChartChina(filteredData) {
    d3.select("#bar-chart-china").selectAll("*").remove();

    const margin = { top: 25, right: 20, bottom: 85, left: 40 },
          width = 650 - margin.left - margin.right,
          height = 550 - margin.top - margin.bottom;

    const svg = d3.select("#bar-chart-china")
                  .append("svg")
                  .attr("width", width + margin.left + margin.right)
                  .attr("height", height + margin.top + margin.bottom)
                  .append("g")
                  .attr("transform", `translate(${margin.left},${margin.top})`);

    const xScale = d3.scaleBand()
                     .range([0, width])
                     .padding(0.1)
                     .domain(filteredData.map(d => d.category));

    const yScale = d3.scaleLinear()
                     .domain([0, maxYValueChina])
                     .range([height, 0]);
    
    const tooltip = d3.select('body').selectAll('.tooltip').data([null]);

    tooltip.enter()
      .append('div')
      .attr('class', 'tooltip')
      .style('opacity', 0) // Initial opacity set to 0
      .merge(tooltip) // Merge enter and update selections
      .style('position', 'absolute')
      .style('text-align', 'center')
      .style('width', 'auto')
      .style('padding', '40px 40px')
      .style('font', '14px sans-serif')
      .style('color', '#ffffff')
      .style('background', '#4a5568')
      .style('border', '1px solid #cbd5e0')
      .style('border-radius', '8px')
      .style('pointer-events', 'none')
      .style('box-shadow', '0px 0px 10px rgba(0, 0, 0, 0.5)')
      .style('z-index', '1000');

    svg.selectAll(".bar")
       .data(filteredData)
       .enter().append("rect")
       .attr("class", "bar")
       .attr("x", d => xScale(d.category))
       .attr("width", xScale.bandwidth())
       .attr("y", d => yScale(d.value))
       .attr("height", d => height - yScale(d.value))
       .attr("fill", "#FF0000")
       .on("mouseover", (event, d) => {
         tooltip.transition()
                .duration(200)
                .style("opacity", .9);
         tooltip.html(`Country: China<br>Year: ${selectedYear}<br>Type: ${d.category}<br>Amount: ${d.value + "%"}`)
                .style("left", (event.pageX) + "px")
                .style("top", (event.pageY - 28) + "px");
       })
       .on("mouseout", () => {
         tooltip.transition()
                .duration(500)
                .style("opacity", 0);
       });

    var minValue = 0.1; // Minimum value to ensure hit area has height
    var hitAreaHeight = 0.1; // Minimum hit area height

// Add hit areas for each bar
  svg.selectAll(".bar-hit-area")
      .data(filteredData)
      .enter().append("rect")
      .attr("class", "bar-hit-area")
      .attr("x", d => xScale(d.category))
      .attr("y", d => yScale(d.value) - 50) // Extend the hit area 50px above the bar
      .attr("width", xScale.bandwidth())
      .attr("height", d => (height - yScale(d.value)) + 50) // Increase the height to include the extended area
      .style("fill", "none") // Keep the hit area invisible
      .style("pointer-events", "all") // Ensure it can catch mouse events
      .on("mouseover", (event, d) => {
          tooltip.transition()
               .duration(200)
               .style("opacity", .9);
          tooltip.html(`Country: China<br>Year: ${selectedYear}<br>Type: ${d.category}<br>Amount: ${d.value + "%"}`)
               .style("left", (event.pageX) + "px")
               .style("top", (event.pageY - 28) + "px");
      })
      .on("mouseout", () => {
          tooltip.transition()
               .duration(500)
               .style("opacity", 0);
      });

    svg.append("g")
       .attr("transform", `translate(0,${height})`)
       .call(d3.axisBottom(xScale))
       .selectAll("text")
       .style("text-anchor", "end")
       .attr("dx", "-.8em")
       .attr("dy", ".15em")
       .attr("transform", "rotate(-15)")
       .style("font-size", "12px")
       .style("font-weight", "bold");

    svg.append("g")
       .call(d3.axisLeft(yScale));

    svg.append("text")
       .attr("transform", `translate(${width/2}, ${height + margin.top + 50})`)
       .style("text-anchor", "middle")
       .text("Energy Consumption Type(per capita)")
       .style("font-weight", "bold");

    svg.append("text")
       .attr("transform", "rotate(-90)")
       .attr("y", 0 - margin.left)
       .attr("x", 0 - (height / 2))
       .attr("dy", "1em")
       .style("text-anchor", "middle")
       .text("Consumption Percentage")
       .style("font-weight", "bold");
    
    // Adding a Title
    svg.append("text")
       .attr("x", (width / 2))
       .attr("y", 5 - (margin.top / 2))
       .attr("text-anchor", "middle")
       .style("font-size", "20px")
       .text(`Renewable Energy Consumption by Type (China) in ${selectedYear}`)
       .style("font-weight", "bold");
  }

  function updateData(year) {
    const yearData = data.find(d => d.year === year);
    const selectedCat = [ 'country', 'year','iso_code', 'biofuel cons',
    'hydro energy',
    'other renewables energy',
    'solar energy',
    'wind energy'
];

    filteredData = Object.entries(yearData)
        .filter(([key, value]) => key.includes('_per_capita') && value > 0)
        .map(([key, value]) => ({
            category: key.split('_').join(' ').replace('per capita', '').trim(),
            value
        }));
    filteredData = filteredData.filter(d => selectedCat.includes(d.category));
    drawBarChart(filteredData);
  }

  function updateDataChina(year) {
    const yearDataChina = dataChina.find(d => d.year === year);
    const selectedCat = [ 'country', 'year','iso_code', 'biofuel cons',
    'hydro energy',
    'other renewables energy',
    'solar energy',
    'wind energy'
];

    filteredDataChina = Object.entries(yearDataChina)
        .filter(([key, value]) => key.includes('_per_capita') && value > 0)
        .map(([key, value]) => ({
            category: key.split('_').join(' ').replace('per capita', '').trim(),
            value
        }));
    filteredDataChina = filteredDataChina.filter(d => selectedCat.includes(d.category));

    drawBarChartChina(filteredDataChina, 'bar-chart-china');
}

function calculateSum(dataArray) {
    return dataArray.reduce((acc, curr) => {
      const year = curr.year;
      const sum = acc[year] || 0;
      acc[year] = sum + curr.biofuel_cons + curr.hydro_energy + curr.other_renewables_energy + curr.solar_energy + curr.wind_energy;
      return acc;
    }, {});
  }

  // Function to prepare data for the line chart
  function prepareLineChartData(usData, chinaData) {
    const usSum = calculateSum(usData);
    const chinaSum = calculateSum(chinaData);

    // Transform sums into arrays of objects
    const usArray = Object.keys(usSum).map(year => ({
      year: +year,
      renewable_energy_sum: usSum[year]
    }));

    const chinaArray = Object.keys(chinaSum).map(year => ({
      year: +year,
      renewable_energy_sum: chinaSum[year]
    }));

    return { usArray, chinaArray };
  }

  // Updated drawLineChart function to use the aggregated data
  function drawLineChart(usData, chinaData) {
    const { usArray, chinaArray } = prepareLineChartData(usData, chinaData);

    const margin = { top: 20, right: 30, bottom: 30, left: 60 },
          width = 960 - margin.left - margin.right,
          height = 500 - margin.top - margin.bottom;

    const svg = d3.select("#line-chart").append("svg")
                  .attr("width", width + margin.left + margin.right)
                  .attr("height", height + margin.top + margin.bottom)
                  .append("g")
                  .attr("transform", `translate(${margin.left},${margin.top})`);

    // Define scales and axes
    const x = d3.scaleLinear()
                .domain(d3.extent(usArray.concat(chinaArray), d => d.year))
                .range([0, width]);

    const y = d3.scaleLinear()
                .domain([0, d3.max([...usArray, ...chinaArray], d => d.renewable_energy_sum)])
                .range([height, 0]);

    svg.append("g")
       .attr("transform", `translate(0, ${height})`)
       .call(d3.axisBottom(x).tickFormat(d3.format("d")));

    svg.append("g")
       .call(d3.axisLeft(y));

    // Add US line
    svg.append("path")
       .datum(usArray)
       .attr("fill", "none")
       .attr("stroke", "blue")
       .attr("stroke-width", 1.5)
       .attr("d", d3.line()
         .x(d => x(d.year))
         .y(d => y(d.renewable_energy_sum))
       );

    // Add China line
    svg.append("path")
       .datum(chinaArray)
       .attr("fill", "none")
       .attr("stroke", "red")
       .attr("stroke-width", 1.5)
       .attr("d", d3.line()
         .x(d => x(d.year))
         .y(d => y(d.renewable_energy_sum))
       );

    // Add current year line
    svg.append("line")
       .attr("x1", x(selectedYear))
       .attr("x2", x(selectedYear))
       .attr("y1", 0)
       .attr("y2", height)
       .attr("stroke", "red")
       .attr("stroke-dasharray", "4");
  }
</script>
<style>

  .graph-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 120%; /* Adjust based on your layout needs */
  }
  .charts-container {
    display: flex;
    justify-content: space-around;
    align-items: center;
  }
  .slider-container {
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    padding: 10px;
    background: #f8f9fa; /* Example background color */
  }

  .main-title {
    text-align: center;
    margin-top: 20px; /* Adjust as needed */
    font-size: 40px; /* Adjust as needed */
    font-weight: bold;
    margin-bottom: 30px;
    margin-top: 5px;
  }

  #bar-chart {
    margin: auto; /* This helps in centering the SVG if the container flex doesn't apply as expected */
  }
  h1 {
    text-align: center; /* Centers the title */
  }
</style>

<div class="content-container">
  <header class="main-title">
    From Hydroenergy Focus to Energy Diversity in USA and China
  </header>

  <div class="charts-container">
    <div id="bar-chart"></div>
    <div id="bar-chart-china"></div>
  </div>
</div>



<div class="slider-container">
  <label for="yearSlider">Year:</label>
  <input type="range" id="yearSlider" min="2002" max="2022" bind:value={selectedYear} />
  <div style="display: flex; justify-content: space-between;">
    <span>{selectedYear}</span>
  </div>
</div>

<div class="charts-container">
    <div id="bar-chart"></div>
    <div id="bar-chart-china"></div>
</div>

<div class="line-chart-container" style="text-align: center;">
  <div id="line-chart"></div>
</div>