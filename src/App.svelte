<script>
  import { setContext, onMount } from "svelte";
  import { getMotion, getData, setColors, getTopo, getBreaks, getColor } from "./utils.js";
  import { themes, colors } from "./config.js";
  import { ScatterChart } from "@onsvisual/svelte-charts";
  import Section from "./layout/Section.svelte";
  import Scroller from "./layout/Scroller.svelte";

  const intro = "an analysis of disparities in the";
  const title = "Deep South";
  const subtitle = "by Alex Jorns";

  let theme = "light";
  setContext("theme", theme);
  setColors(themes, theme);

  const threshold = 0.65;
  let animation = getMotion();
  let id = {};
  let idPrev = {};
  let hovered, selected;
  let geojson;

  let xKey = "education";
  let yKey = null;
  let zKey = null;
  let rKey = null;
  let explore = false;

  let data = { region: {} };
  let metadata = { region: {} };
  const datasets = ["region"];

  // Hardcoded data for testing (comment out when loading CSV)
  data.region.indicators = [
  { code: "01", name: "Alabama", education: 40, poverty: 15.6, pop: 27, region: "Deep South" },
  { code: "02", name: "Alaska", education: 28, poverty: 10.4, pop: 4, region: "West" },
  { code: "04", name: "Arizona", education: 31, poverty: 12.4, pop: 6, region: "West" },
  { code: "05", name: "Arkansas", education: 31, poverty: 15.7, pop: 16, region: "Deep South" },
  { code: "06", name: "California", education: 35, poverty: 12.0, pop: 6, region: "West" },
  { code: "08", name: "Colorado", education: 23, poverty: 9.3, pop: 5, region: "West" },
  { code: "09", name: "Connecticut", education: 18, poverty: 10.3, pop: 13, region: "Northeast" },
  { code: "10", name: "Delaware", education: 29, poverty: 10.5, pop: 24, region: "South" },
  { code: "11", name: "District of Columbia", education: 46, poverty: 14.0, pop: 44, region: "South" },
  { code: "12", name: "Florida", education: 30, poverty: 12.3, pop: 17, region: "South" },
  { code: "13", name: "Georgia", education: 32, poverty: 13.6, pop: 33, region: "Deep South" },
  { code: "15", name: "Hawaii", education: 28, poverty: 10.1, pop: 2, region: "West" },
  { code: "16", name: "Idaho", education: 22, poverty: 10.1, pop: 1, region: "West" },
  { code: "17", name: "Illinois", education: 26, poverty: 11.6, pop: 15, region: "Midwest" },
  { code: "18", name: "Indiana", education: 23, poverty: 12.3, pop: 10, region: "Midwest" },
  { code: "19", name: "Iowa", education: 17, poverty: 11.3, pop: 5, region: "Midwest" },
  { code: "20", name: "Kansas", education: 21, poverty: 11.2, pop: 6, region: "Midwest" },
  { code: "21", name: "Kentucky", education: 29, poverty: 16.4, pop: 9, region: "South" },
  { code: "22", name: "Louisiana", education: 36, poverty: 18.9, pop: 33, region: "Deep South" },
  { code: "23", name: "Maine", education: 22, poverty: 10.4, pop: 2, region: "Northeast" },
  { code: "24", name: "Maryland", education: 26, poverty: 9.5, pop: 32, region: "South" },
  { code: "25", name: "Massachusetts", education: 14, poverty: 10.4, pop: 10, region: "Northeast" },
  { code: "26", name: "Michigan", education: 30, poverty: 13.5, pop: 14, region: "Midwest" },
  { code: "27", name: "Minnesota", education: 17, poverty: 9.3, pop: 8, region: "Midwest" },
  { code: "28", name: "Mississippi", education: 39, poverty: 18.0, pop: 38, region: "Deep South" },
  { code: "29", name: "Missouri", education: 26, poverty: 12.0, pop: 12, region: "Midwest" },
  { code: "30", name: "Montana", education: 20, poverty: 11.7, pop: 1, region: "West" },
  { code: "31", name: "Nebraska", education: 24, poverty: 10.5, pop: 5, region: "Midwest" },
  { code: "32", name: "Nevada", education: 32, poverty: 12.0, pop: 11, region: "West" },
  { code: "33", name: "New Hampshire", education: 16, poverty: 7.2, pop: 2, region: "Northeast" },
  { code: "34", name: "New Jersey", education: 18, poverty: 9.7, pop: 16, region: "Northeast" },
  { code: "35", name: "New Mexico", education: 37, poverty: 17.8, pop: 3, region: "West" },
  { code: "36", name: "New York", education: 28, poverty: 14.2, pop: 18, region: "Northeast" },
  { code: "37", name: "North Carolina", education: 25, poverty: 12.8, pop: 22, region: "South" },
  { code: "38", name: "North Dakota", education: 18, poverty: 9.8, pop: 4, region: "Midwest" },
  { code: "39", name: "Ohio", education: 21, poverty: 13.3, pop: 13, region: "Midwest" },
  { code: "40", name: "Oklahoma", education: 31, poverty: 15.9, pop: 8, region: "South" },
  { code: "41", name: "Oregon", education: 27, poverty: 12.2, pop: 2, region: "West" },
  { code: "42", name: "Pennsylvania", education: 22, poverty: 12.0, pop: 12, region: "Northeast" },
  { code: "44", name: "Rhode Island", education: 26, poverty: 10.8, pop: 9, region: "Northeast" },
  { code: "45", name: "South Carolina", education: 31, poverty: 13.9, pop: 26, region: "Deep South" },
  { code: "46", name: "South Dakota", education: 21, poverty: 11.8, pop: 3, region: "Midwest" },
  { code: "47", name: "Tennessee", education: 31, poverty: 14.0, pop: 17, region: "Deep South" },
  { code: "48", name: "Texas", education: 20, poverty: 13.7, pop: 14, region: "Deep South" },
  { code: "49", name: "Utah", education: 25, poverty: 9.0, pop: 2, region: "West" },
  { code: "50", name: "Vermont", education: 16, poverty: 9.7, pop: 2, region: "Northeast" },
  { code: "51", name: "Virginia", education: 23, poverty: 10.2, pop: 20, region: "South" },
  { code: "53", name: "Washington", education: 21, poverty: 10.3, pop: 5, region: "West" },
  { code: "54", name: "West Virginia", education: 35, poverty: 16.7, pop: 4, region: "South" },
  { code: "55", name: "Wisconsin", education: 22, poverty: 10.7, pop: 7, region: "Midwest" },
  { code: "56", name: "Wyoming", education: 19, poverty: 11.3, pop: 1, region: "West" }
];



  // Reactive statement: get region based on selected code (fixed to avoid metadata lookup)
  $: region = selected
    ? data.region.indicators.find(d => d.code === selected)?.region ?? null
    : null;

  // Highlight all codes with matching region
  $: chartHighlighted = region
    ? data.region.indicators.filter(d => d.region === region).map(d => d.code)
    : [];

  const actions = {
    chart: {
      chart01: () => { xKey = "education"; yKey = null; zKey = null; rKey = null; explore = false; },
      chart02: () => { xKey = "education"; yKey = null; zKey = null; rKey = "pop"; explore = false; },
      chart03: () => { xKey = "education"; yKey = "poverty"; zKey = null; rKey = "pop"; explore = false; },
      chart04: () => { xKey = "education"; yKey = "poverty"; zKey = "region"; rKey = "pop"; explore = false; },
      chart05: () => { xKey = "education"; yKey = "poverty"; zKey = null; rKey = "pop"; explore = true; }
    }
  };

  function doSelect(e) {
    selected = e.detail.id;
  }

  function doHover(e) {
    hovered = e.detail.id;
  }

  function runActions(codes = []) {
    codes.forEach(code => {
      if (id[code] !== idPrev[code]) {
        actions[code]?.[id[code]]?.();
        idPrev[code] = id[code];
      }
    });
  }

  $: id && runActions(Object.keys(actions));

  /*
  // Uncomment this block to load data from CSV instead of using hardcoded data above
  datasets.forEach(geo => {
    getData(`./data/data_${geo}.csv`)
      .then(arr => {
        let meta = arr.map(d => ({
          code: d.code,
          name: d.name,
          parent: d.parent || null
        }));
        let lookup = Object.fromEntries(meta.map(d => [d.code, d]));
        metadata[geo].array = meta;
        metadata[geo].lookup = lookup;

        let indicators = arr.map((d, i) => ({
          ...meta[i],
          education: +d.education,
          poverty: +d.poverty,
          pop: +d.pop,
          region: d.region
        }));

        data[geo].indicators = indicators;
      });
  });
  */

  getTopo('./data/geo_lad2021.json', 'geog').then(geo => {
    geo.features.sort((a, b) => a.properties.AREANM.localeCompare(b.properties.AREANM));
    geojson = geo;
  });

  const articleSections = [
    "The <strong>Deep South</strong> and Southeast, which include many of the most historically marginalized states, are showing proficiency rates below 27%. That means fewer than 1 in 4 to 1 in 3 eighth graders in these regions can read at a level considered “proficient” by national standards.",
    "Compare that to the <strong>Northeast</strong>, where 1 in 3 or more students are meeting or exceeding proficiency. The gap is 7 percentage points between the Deep South and Northeast. That might not sound big, but in education data, that’s a huge performance divide affecting millions of kids.",
    "And the fact that national scores have <strong>stagnated—or in some cases declined</strong>—only sharpens the urgency. If you're building a project or story around this, this disparity is a powerful anchor point. It’s not just about test scores—it’s about which children are systematically being left behind and why.",
    "That means that only about <strong>1 in 3 eighth graders</strong> in the best-performing region are reading at a level the National Assessment of Educational Progress (NAEP) defines as 'proficient' — which itself is not an elite benchmark, but a solid, grade-level competency."
  ];

  onMount(() => {
    const observer = new IntersectionObserver(entries => {
      entries.forEach(entry => {
        const elem = entry.target;
        elem.style.backgroundColor = entry.intersectionRatio >= 0.9 ? "#FAD9F9" : "#DAA6D3";
      });
    }, { threshold: [0.85, 0.95] });

    document.querySelectorAll(".article-text").forEach(el => observer.observe(el));
  });
</script>

<!-- TITLE CARD -->
<div class="title-card">
  <div class="content">
    {#if intro}
      <p class="intro-sub">{intro}</p>
    {/if}
    <h1>{title}</h1>
    {#if subtitle}
      <p class="intro-sub">{subtitle}</p>
    {/if}
  </div>
</div>

<!-- VISUALIZATION SECTION -->
<Section>
  <h2>The correlation between poverty and education</h2>
  <p>by regions in the United States</p>
</Section>

<Scroller {threshold} bind:id={id['chart']} splitscreen={true}>
  <div slot="background">
    <figure>
      <div class="col-wide height-full">
        <!-- metadata lookup commented since not used -->
		{#if data.region.indicators}

          <div class="chart">
            <ScatterChart
              height="calc(100vh - 150px)"
              data={data.region.indicators.map(d => ({
                ...d,
                // parent_name: metadata.region.lookup[d.region]?.name // commented because metadata not used now
              }))}
              colors={explore ? ['lightgrey'] : colors.cat}
              {xKey} {yKey} {zKey} {rKey}
              idKey="code" labelKey="name"
              r={[3, 10]}
              xScale="linear"
				xTicks={[0, 20, 40, 60, 80, 100]}
				xFormatTick={d => `${d}%`}
              yFormatTick={d => d.toLocaleString()}
              legend={zKey != null}
              labels
              select={explore}
              selected={explore ? selected : null}
              on:select={doSelect}
              hover
              {hovered}
              on:hover={doHover}
              highlighted={explore ? chartHighlighted : []}
              colorSelect="#206095"
              colorHighlight="#999"
              overlayFill
              {animation}
            />
          </div>
        {/if}
      </div>
    </figure>
  </div>

  <div slot="foreground">
    <section data-id="chart01"><div class="col-medium"><p>This chart shows the <strong>education</strong> of each state.</p></div></section>
    <section data-id="chart02"><div class="col-medium"><p>Radius shows the <strong>population</strong>.</p></div></section>
    <section data-id="chart03"><div class="col-medium"><p>Vertical axis = <strong>poverty</strong>.</p></div></section>
    <section data-id="chart04"><div class="col-medium"><p>Color shows <strong>region</strong>.</p></div></section>
    <section data-id="chart05">
      <div class="col-medium">
        <h3>Select a district</h3>
        <p>You can use the dropdown to highlight all districts in the same region.</p>
        {#if data.region.indicators}
          <select bind:value={selected}>
            <option value={null}>Select one</option>
            {#each data.region.indicators as d}
              <option value={d.code}>{d.name}</option>
            {/each}
          </select>
        {/if}
      </div>
    </section>
  </div>
</Scroller>

<!-- ENDING NARRATIVE SECTIONS -->
{#each articleSections as text (text)}
  <div class="article-text">
    <p>{@html text}</p>
  </div>
{/each}

<style>
  @import url('https://fonts.googleapis.com/css2?family=Abril+Fatface&family=Nunito:wght@400;600&display=swap');

  /* Title Card */
  .title-card {
    background-color: #034c36;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
    padding: 2rem;
    box-sizing: border-box;
    font-family: "Nunito", sans-serif;
  }

  .content {
    max-width: 1000px;
    background-color: #DAA6D3;
    padding: 2rem 25rem;
    border: 3px solid white;
    border-radius: 2rem;
    box-shadow: 16px 16px #FAD9F9;
  }

  .intro-sub {
    color: #FAD9F9;
    font-size: 1.4rem;
    margin-bottom: 1rem;
    font-weight: 600;
    letter-spacing: 0.05em;
  }

  h1 {
    font-family: "Abril Fatface", serif;
    font-size: 4rem;
    color: white;
    margin-bottom: 1rem;
    text-shadow: 2px 2px 6px rgba(0, 0, 0, 0.5);
  }

  h2 {
    font-family: "Abril Fatface", serif;
    font-size: 2rem;
    color: white;
    margin-bottom: 1rem;
    text-shadow: 2px 2px 6px rgba(0, 0, 0, 0.5);
  }

  .chart {
    margin-top: 45px;
    width: calc(100% - 5px);
  }

  select {
    max-width: 350px;
  }

  .article-text {
    margin: 50vh auto;
    width: 60%;
    background-color: #ff99fc;
    color: #007052;
    border: 3px solid #034c36;
    border-radius: 20px;
    padding: 2rem;
    box-shadow: 16px 16px white;
    font-size: 1.4rem;
    line-height: 1.6;
    text-align: center;
    font-family: "Nunito", sans-serif;
    transition: background-color 0.5s ease;
  }

  strong {
    font-weight: bold;
  }

  :global(body) {
    background-color: #034c36;
    transition: background-color 1s ease;
  }

  @media (max-width: 900px) {
    .content {
      max-width: 90vw;
      padding: 2rem;
    }
    h1 {
      font-size: 3rem;
    }
    .intro-sub {
      font-size: 1.2rem;
    }
  }

  @media (max-width: 600px) {
    h1 {
      font-size: 2.4rem;
    }
    .intro-sub {
      font-size: 1rem;
    }
  }

  section[data-id^="chart"] {
    background-color: #034c36;
    color: white; 
  }
</style>
