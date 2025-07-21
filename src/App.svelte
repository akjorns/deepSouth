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

  let xKey = "area";
  let yKey = null;
  let zKey = null;
  let rKey = null;
  let explore = false;

  let data = { district: {}, region: {} };
  let metadata = { district: {}, region: {} };
  const datasets = ["region", "district"];

  $: region = selected && metadata.district.lookup ? metadata.district.lookup[selected].parent : null;
  $: chartHighlighted = metadata.district.array && region
    ? metadata.district.array.filter(d => d.parent == region).map(d => d.code)
    : [];

  const actions = {
    chart: {
      chart01: () => { xKey = "area"; yKey = null; zKey = null; rKey = null; explore = false; },
      chart02: () => { xKey = "area"; yKey = null; zKey = null; rKey = "pop"; explore = false; },
      chart03: () => { xKey = "area"; yKey = "density"; zKey = null; rKey = "pop"; explore = false; },
      chart04: () => { xKey = "area"; yKey = "density"; zKey = "parent_name"; rKey = "pop"; explore = false; },
      chart05: () => { xKey = "area"; yKey = "density"; zKey = null; rKey = "pop"; explore = true; }
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
          area: d.area,
          pop: d['2020'],
          density: d.density,
          age_med: d.age_med
        }));

        if (geo === "district") {
          ['density', 'age_med'].forEach(key => {
            const values = indicators.map(d => d[key]).sort((a, b) => a - b);
            const breaks = getBreaks(values);
            indicators.forEach(d => d[`${key}_color`] = getColor(d[key], breaks, colors.seq));
          });
        }

        data[geo].indicators = indicators;

        const years = Array.from({ length: 20 }, (_, i) => 2001 + i);
        data[geo].timeseries = arr.flatMap(d =>
          years.map(year => ({
            code: d.code,
            name: d.name,
            value: d[year],
            year
          }))
        );
      });
  });

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

<!-- 🌟 TITLE CARD -->
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

<!-- 📊 VISUALIZATION SECTION -->
<Section>
  <h2>This is a dynamic chart section</h2>
  <p>The chart below will respond to the captions as you scroll down.</p>
</Section>

<Scroller {threshold} bind:id={id['chart']} splitscreen={true}>
  <div slot="background">
    <figure>
      <div class="col-wide height-full">
        {#if data.district.indicators && metadata.region.lookup}
          <div class="chart">
            <ScatterChart
              height="calc(100vh - 150px)"
              data={data.district.indicators.map(d => ({
                ...d,
                parent_name: metadata.region.lookup[d.parent]?.name
              }))}
              colors={explore ? ['lightgrey'] : colors.cat}
              {xKey} {yKey} {zKey} {rKey}
              idKey="code" labelKey="name"
              r={[3, 10]}
              xScale="log"
              xTicks={[10, 100, 1000, 10000]}
              xFormatTick={d => d.toLocaleString()}
              xSuffix=" sq.km"
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
    <section data-id="chart01"><div class="col-medium"><p>This chart shows the <strong>area</strong> of each district.</p></div></section>
    <section data-id="chart02"><div class="col-medium"><p>Radius shows the <strong>population</strong>.</p></div></section>
    <section data-id="chart03"><div class="col-medium"><p>Vertical axis = <strong>density</strong>.</p></div></section>
    <section data-id="chart04"><div class="col-medium"><p>Color shows <strong>region</strong>.</p></div></section>
    <section data-id="chart05">
      <div class="col-medium">
        <h3>Select a district</h3>
        <p>You can use the dropdown to highlight all districts in the same region.</p>
        {#if geojson}
          <select bind:value={selected}>
            <option value={null}>Select one</option>
            {#each geojson.features as place}
              <option value={place.properties.AREACD}>{place.properties.AREANM}</option>
            {/each}
          </select>
        {/if}
      </div>
    </section>
  </div>
</Scroller>

<!-- 📘 ENDING NARRATIVE SECTIONS -->
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
    background-color: pink;
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
</style>
