<script lang="ts">
  import { onMount } from "svelte";
  import { BarChart, type BarChartData, type BarChartOptions } from "chartist";
  import "chartist/dist/index.css"; // Import Chartist styles

  let type: string = "Line"; // Accepts "Line", "Bar", "Pie"

  let { data, height } = $props();

  let options: BarChartOptions = {
    high: height,
    low: 0,
  };

  let chartContainer: Element;

  $effect(() => {
    if (chartContainer) {
      options.high = height;
      new BarChart(chartContainer, data, options);
    }
  });
</script>

<div bind:this={chartContainer} class="ct-chart" style="height: 40vh;"></div>
