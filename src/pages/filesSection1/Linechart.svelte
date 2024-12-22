<script>
  import { Graphic, Line, LineLayer, XAxis, YAxis, LabelLayer } from '@snlab/florence';
  export let scaleX;
  export let scaleY;
  export let selectedCountry;
  export let legendItems;
  export let x;
  export let strokeWidth;
  export let countryData;
</script>


<div class="graph">
  <!-- Title -->
  <div class="title">
    <h4>Demography of {selectedCountry} (2000-2019)</h4>
  </div>

  <!-- Main Chart -->
  <div class="main-chart">
    <Graphic {scaleX} {scaleY} width={350} height={350} padding={40} flipY>
      {#each legendItems as item, index}
        <Line
          key={index}
          {x}
          {strokeWidth}
          y={countryData.column(item.dataKey)}
          stroke={item.color}
          dashArray={item.dashArray || undefined}
        />
      {/each}
      <XAxis title="Year" />
      <YAxis title="Per 1000 inhabitants" />
    </Graphic>
  </div>

  <!-- Legend section below the chart -->
  <div class="title legend">
   <h5>Legend</h5>
  </div>
  <div class="content legend">
   <Graphic width={300} height={100} scaleX={[0, 10]} scaleY={[0, 10]} padding={5}>
    <LineLayer 
    x={[[0, 1], [0, 1], [0, 1], [0, 1]]} 
    y={[[1, 1], [2.5, 2.5], [4, 4], [5.5, 5.5]]}
    stroke={['blue', 'red', 'black', 'purple']}
    dashArray={[null, null, null, "4"]}
    />
    <LabelLayer 
    x={[2, 2, 2, 2]} 
    y={[1, 2.5, 4, 5.5]} 
    text={['birth rate', 'death rate', 'net migration rate', 'population change rate']}
    fontSize=12
    anchorPoint=l
    />
   </Graphic>
  </div>
</div>

<!-- Short description of the project -->
<!-- <div class="description">
 <p>
 This project is about the quantification of human migration patterns in 2000-2019 for Europe. The line chart above displays the annual birth rate, death rate, net migration rate and population change rate for the European country that was selected by the user. The y-axis represents the variables in number per 1000 inhabitants, while the x-axis represents the time in years. The goal of the visualization is to show how the demography for different countries in Europe, particularly focusing on migration patterns, has changed over time.   
 </p>
</div> -->

<style>
  .graph {
    width: 350px;
    height: 445px;
    background-color: rgb(255, 250, 240);
    padding: 7px;
    border: 1px solid black;  /* Add a border to visually confirm the area */ 
  }

  .title h4, .title.legend h5 {
    margin: 0; /* remove default margins */
  }
</style>
