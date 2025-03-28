<script lang="ts">
  import { Button } from "$lib/components/ui/button";
  import { Input } from "$lib/components/ui/input";
  import type { BarChartData } from "chartist";
  import BarChart from "./BarChart.svelte";

  let initialPace: number = $state(1.84);
  let paceDelta: number = $state(0.01);
  let deltaProbability: number = $state(0.33);
  let targetTimeMinutes: number = $state(30);
  let modelRangeInPercent: number = $state(10);
  let numberOfSimulations: number = $state(10);
  let bucketSizeInSecond: number = $state(30);

  let stepTimeCounterData: BarChartData | null = $state(null);

  const stepsTimeCount = () => {
    let targetTimeSeconds = targetTimeMinutes * 60;
    let numberOfSteps = initialPace * targetTimeSeconds;

    let minTargetTime = (targetTimeSeconds * (100 - modelRangeInPercent)) / 100;
    let maxTargetTime = (targetTimeSeconds * (100 + modelRangeInPercent)) / 100;

    let stepTimeCounts = new Map<number, number>();

    for (let t = minTargetTime; t <= maxTargetTime; t += bucketSizeInSecond) {
      stepTimeCounts.set(t / 60, 0);
    }

    for (let i = 0; i < numberOfSimulations; i++) {
      let timeTaken = getTimeTakenForSteps(numberOfSteps);

      let timeBucket: number | null = null;
      for (let t = minTargetTime; t <= maxTargetTime; t += bucketSizeInSecond) {
        if (timeTaken >= t && timeTaken < t + bucketSizeInSecond) {
          timeBucket = t / 60;
          break;
        }
      }
      if (timeBucket != null) {
        let timeCount = stepTimeCounts.get(timeBucket) ?? 0;
        stepTimeCounts.set(timeBucket, timeCount + 1);
      }
    }
    let labels = [];
    let series = [];
    for (const entry of stepTimeCounts.entries()) {
      const [key, value] = entry;
      labels.push(key.toFixed(2));
      series.push(value);
    }
    stepTimeCounterData = {
      labels: labels,
      series: [series],
    };
  };

  const getTimeTakenForSteps = (stepsLimit: number): number => {
    let pace = initialPace;
    let steps = 0;
    let totalTime = 0;
    while (steps + pace * 60 <= stepsLimit) {
      steps += pace * 60;
      totalTime += 60;
      let randomNumber = Math.random();
      if (randomNumber <= deltaProbability) {
        pace -= paceDelta;
      } else if (randomNumber > 1 - deltaProbability) {
        pace += paceDelta;
      }
    }

    if (steps < stepsLimit) {
      totalTime += (stepsLimit - steps) / pace;
    }
    return totalTime;
  };
</script>

<div class="flex flex-col w-full">
  <div class="p-8 w-full grid grid-cols-2 gap-4">
    <div class="min-w-fit">
      <div>Initial Pace per second</div>
      <Input type="number" bind:value={initialPace} min="1" max="3" />
    </div>
    <div class="min-w-fit">
      <div>Change delta for pace per second</div>
      <Input type="number" bind:value={paceDelta} min="0" max="0.03" />
    </div>
    <div class="min-w-fit">
      <div>Probability of change</div>
      <Input type="number" bind:value={deltaProbability} min="0" max="1" />
    </div>
    <div class="min-w-fit">
      <div>Target time in minutes</div>
      <Input type="number" bind:value={targetTimeMinutes} />
    </div>
    <div class="min-w-fit">
      <div>Challenge range in percent</div>
      <Input type="number" bind:value={modelRangeInPercent} min="0" max="100" />
    </div>
    <div class="min-w-fit">
      <div>Bucket size of bars in seconds</div>
      <Input type="number" bind:value={bucketSizeInSecond} min="0" />
    </div>
    <div class="min-w-fit">
      <div>Number of simulations to run</div>
      <Input
        type="number"
        bind:value={numberOfSimulations}
        min="1"
        max="100000"
      />
    </div>
    <div class="min-w-fit self-end">
      <Button variant="outline" onclick={stepsTimeCount}
        >Click to get simfun</Button
      >
    </div>
  </div>
  {#if stepTimeCounterData != null}
    <div class="w-full min-h-full">
      <BarChart data={stepTimeCounterData} height={numberOfSimulations} />
    </div>
  {/if}
</div>
