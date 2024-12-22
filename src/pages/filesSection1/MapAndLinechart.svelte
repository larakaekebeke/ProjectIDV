<script>
  import { data_adm0 } from './data_adm0.js';
  import { EU_countries } from './EU_countries.js';
  import {Graphic, PolygonLayer, fitScales } from '@snlab/florence';
  import DataContainer from '@snlab/florence-datacontainer';
  import { scaleSequential, scaleDiverging, scaleLinear, scaleTime } from 'd3-scale'; // dingen toegevoegd (Louise)
  import { interpolateBlues, interpolateRdBu, interpolateReds } from 'd3-scale-chromatic';
  import { onMount } from 'svelte';
  import LineChartComponent from './Linechart.svelte'; // import the linechart (Louise)


  // Initialize DataContainers
  let polygons_EU = new DataContainer(EU_countries);
  const data_EU = new DataContainer(data_adm0);

  polygons_EU.setKey('polygonId')

  // Current selected year and variable 
  let selectedYear = 2000;
  let selectedVariable = 'birthRate';
  let variables = ['birthRate', 'deathRate', 'netMgr', 'popChange'];

  // Data per year 
  let yearData = data_EU.filter(row => row.year === selectedYear);
  yearData.setKey('polygonId')
  yearData.addColumn('geometry', yearData.map('polygonId', polygonId => polygons_EU.row({key: polygonId})['$geometry']))

  function updateYear(year) {
    selectedYear = year;

    yearData = data_EU.filter(row => row.year === selectedYear);
    yearData.setKey('polygonId');
    yearData.addColumn('geometry', yearData.map('polygonId', polygonId => polygons_EU.row({ key: polygonId })['$geometry']));
    // Update color scale
    myColorScale = getColorScale(selectedVariable);
  }

  // console.log("yearData", yearData.data())
  // console.log("polygons_EU", polygons_EU.data())


  // Scales
  const myGeoScale = fitScales(polygons_EU.domain('$geometry'));

  function getColorScale(variable) {
    if (variable === 'netMgr' || variable === 'popChange') {
      // divergent scale for net migration
      return scaleDiverging(interpolateRdBu)  
      .domain(yearData.domain(variable));
    } else {
      // sequential scale for birth and death rate 
      return scaleSequential()
      .domain(yearData.domain(variable)) 
      .interpolator(variable === 'birthRate' ? interpolateBlues : interpolateReds); }
  }
  $: myColorScale = getColorScale(selectedVariable);
 
  
  // Function to handle click event
  // let selectedCountry = null;
  // function handleClick(event) {
  //     // Set selectedCountry to the clicked country
  //     selectedCountry = event.key || null;

  //     if (selectedCountry) {
  //     // Get country data
  //     const countryData = yearData.row({ key: selectedCountry });
  //     if (countryData) {
  //       const value = countryData[selectedVariable];
  //       const valueText = value !== 0 ? value.toFixed(2) : 'NA';
  //     }
  //   } else {
  //   }
  // }

  // hovering over countries 
  let activeCountry = null;
  let tooltipElement;
  let isHovering = false;
  let isClicking = false;
  const fadedColor = 'rgb(200, 200, 200, 0.5)';

  function handleMouseover(event) {
    isHovering = true;
    activeCountry = event.key || null; 
    if (activeCountry) {
      const countryData = yearData.row({ key: activeCountry });
      const countryName = countryData ? countryData['adminName'] : 'Unknown';
      const infoText = `<div><strong>Country:</strong> ${countryName}</div>`;

      if (tooltipElement) {
          tooltipElement.innerHTML = infoText;
          const offset = 10;
          tooltipElement.style.left = event.clientX + offset + 'px'; 
          tooltipElement.style.top = event.clientY - tooltipElement.offsetHeight - offset + 'px'; 
          tooltipElement.style.display = 'block';
      }
    }
  }

  function handleMouseout() {
    isHovering = false;
    activeCountry = null; // Reset activeCountry on mouse out
    if (tooltipElement) {
      tooltipElement.style.display = 'none';
      tooltipElement.innerHTML = '';
    }
  }

  onMount(() => {
  tooltipElement = document.createElement('div');
  tooltipElement.id = 'countryTooltip';
  tooltipElement.style.position = 'absolute';
  tooltipElement.style.display = 'none'; 
  document.body.appendChild(tooltipElement);
  });

// Linechart (Louise)

  const country_data = new DataContainer(data_adm0);

  let selectedCountry = ''; // default: no country selected yet
  let countryData = '';

  $: console.log(selectedCountry)
  // Extract unique country names from the data (line 125 nog nodig?)
  // const countries = [...new Set(data_adm0.map(row => row.adminName))];

  // Reactive block to filter and process data based on selectedCountry
  $: if (selectedCountry) {
    countryData = country_data
      .filter(row => row.adminName === selectedCountry)
      .arrange({ year: "ascending" })
      .mutate({ date_as_date: row => new Date(row.year, 0, 1) });
  }

  // Scales
  // default scales when no country has been selected
  let scaleX = '';
  let scaleY = '';
  // let scaleX = scaleTime().domain([new Date(2000, 0, 1), new Date(2019, 0, 1)]);
  // let scaleY = scaleLinear().domain([0, 12]).nice();

  $: if (selectedCountry) {
      // Update scaleX and scaleY dynamically based on the selected country's data
      scaleX = scaleTime().domain(countryData.domain('date_as_date'));

      const values = [
          ...countryData.column('birthRate'),
          ...countryData.column('deathRate'),
          ...countryData.column('popChange'),
          ...countryData.column('netMgr')
      ];

      const minY = Math.min(...values);
      const maxY = Math.max(...values);

      scaleY = scaleLinear().domain([minY, maxY]).nice();
  }
  
  // Constants
  let x = ''
  $: if (selectedCountry) {
    x = countryData.column('date_as_date')
  }
  const strokeWidth = 2;

  // Items for linechart
  const legendItems = [
    { label: "Birth Rate", color: "blue", dashArray: null, dataKey: "birthRate" },
    { label: "Death Rate", color: "red", dashArray: null, dataKey: "deathRate" },
    { label: "Population Change", color: "purple", dashArray: "4", dataKey: "popChange" },
    { label: "Net Migration", color: "black", dashArray: null, dataKey: "netMgr" },
  ];

// Interaction (Louise)
  function handleClick(event) {
    const countryKey = event.key || null; // Get the clicked country key
    if (countryKey) {
      const countryData = yearData.row({ key: countryKey });
      const countryName = countryData ? countryData['adminName'] : null;

      if (countryName) {
        selectedCountry = countryName; // Update the selected country for the linechart
      }
    }
  }

</script>

<!-- nieuwe div gemaakt ("chart"), voor de layout, die dan opgesplitst wordt in de divs "map" en "linechart" (Louise) -->
<div class="chart">
  <div class="linechart">
    {#if selectedCountry}
    <!-- Render the linechart when a country is selected (Louise) -->
      <LineChartComponent 
      {selectedCountry} {scaleX} {scaleY} {countryData} {legendItems} {strokeWidth} {x}
      />
    {:else}
      <div class="caption-placeholder">
        <p>Click on a country to see it's linechart!</p>
      </div>
    {/if}
  </div>
  <div class="map">
      <div class="title">
        <h1>MIGRATION STATISTICS</h1>
      </div>

      <div class="drop-down-menu">
      <label for="variable-select">Select a variable: </label>
      <select id="variable-select" bind:value={selectedVariable}>
        {#each variables as variable}
          <option value={variable}>{variable}</option>
        {/each}
      </select>
      </div>

      <Graphic width={700} height={700} {...myGeoScale} flipY >
          <PolygonLayer 
            geometry={polygons_EU.column('$geometry')}
            stroke={yearData.map('polygonId', polygonId => 
              { return polygonId === activeCountry ? 'red' : 'black'})}
            strokeWidth={yearData.map('polygonId', polygonId => 
              { return polygonId === activeCountry ? '3' : '0.5'})}
            fill={yearData.map(selectedVariable, myColorScale)}
            keys={yearData.column('polygonId')}
            onMouseover={handleMouseover}
            onMouseout={handleMouseout}
            onClick={handleClick}
            >
          </PolygonLayer>
      </Graphic>


      <!-- timeline -->
      <div class="timeline">
        <label for="year-slider">Year:</label>
        <input 
          type="range"
          autoPlay="true"
          id="year-slider" 
          min="2000" 
          max="2019" 
          bind:value={selectedYear} 
          on:input={(e) => updateYear(+e.target.value)}>
        <span>{selectedYear}</span>
      </div>
  </div>
</div>


<style>

  h1,
	h2 {
		font-family: 'Spectral', serif;
		font-weight: 200;
		text-align: center;
		color: rgb(60, 60, 60);
	}

  h1 {
		font-size: 30px;
	}

	h2 {
		font-size: 20px;
	}

  h3 {
		font-size: 16px;
	}

  /* chart layout (Louise) */
  .chart {
    display: flex; /* Use flexbox for horizontal layout (chatgpt) */
    justify-content: space-between; /* Add space between the map and linechart (chatgpt) */
    align-items: center; /* Align at the bottom */
    width: 100%; /* Full-width container (chatgpt) */
    max-width: 1200px; /* Limit the maximum width (chatgpt) */
    margin: 0 auto; /* Center the layout horizontally (chatgpt) */
  }

  /* linechart layout (Louise) */
  .linechart {
    flex: 1; /* make linechart half as big as the map */
    max-width: 350px; /* Ensure it is exactly half the width of the map (chatgpt) */
    margin-right: 20px; /* Add spacing between map and linechart */
    padding: 10px; /* Optional: Add some padding (chatgpt) */
  }

  /* caption-placeholder layout (Louise) */
  .caption-placeholder {
   display: flex;
   justify-content: center;
   width: 100%; /* Same width as the linechart */
   height: 100%; /* Same height as the linechart */
   background-color: rgb(250, 239, 233); /* Match the background of the linechart-container */
   font-size: 16px; /* Font size for readability */
   font-weight: bold; /* Make the text stand out */
   border: 1px solid #ccc; /* Optional: Add a light border */
   border-radius: 4px; /* Optional: Round the corners slightly */ 
  }

	.map {
		max-width: 700px; /* limit max size of the map (Louise) */
		background-color: rgb(255, 250, 240);
		padding: 10px;
    margin: 0 auto;
    margin-bottom: 20px; /* snap niet echt wrm (Louise) */
    flex: 2; /* make map twice as big as linechart (Louise) */
	}  

	.title {
		width: 700px;
    padding: 30px;
	}

   :global(#countryTooltip) {
  position: absolute;
  font-family: "Spectral", serif;
  font-size: 12px;
  fill: rgb(0, 0, 0);
  background-color: rgba(250, 250, 250, 0.95);
  padding: 5px; 
  border-color: black;
  border-width: 1px;
  border-style: solid;
  border-radius: 5px; 
  display: none; 
  z-index: 1000;
  }

</style>
    