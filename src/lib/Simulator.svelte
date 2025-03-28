<script lang="ts">
  import { Button } from "$lib/components/ui/button";
  import { Input } from "$lib/components/ui/input";
  import type { BarChartData } from "chartist";
  import BarChart from "./BarChart.svelte";

  let initialStepRate: number = $state(1.84);
  let paceDelta: number = $state(0.01);
  let deltaProbability: number = $state(0.33);
  let targetTimeMinutes: number = $state(30);
  let modelRangeInPercent: number = $state(10);
  let numberOfSimulations: number = $state(10);
  let bucketSizeInSecond: number = $state(30);
  let averageTimeInSecondForPaceChange: number = $state(10);

  let stepTimeCounterData: BarChartData | null = $state(null);

  const stepsTimeCount = () => {
    let targetTimeSeconds = targetTimeMinutes * 60;
    let numberOfSteps = initialStepRate * targetTimeSeconds;

    let minTargetTime =
      (targetTimeSeconds * (100 - 2 * modelRangeInPercent)) / 100;
    let maxTargetTime =
      (targetTimeSeconds * (100 + 2 * modelRangeInPercent)) / 100;

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
    let pace = initialStepRate;
    let steps = 0;
    let totalTime = 0;
    while (steps + pace * averageTimeInSecondForPaceChange <= stepsLimit) {
      steps += pace * averageTimeInSecondForPaceChange;
      totalTime += averageTimeInSecondForPaceChange;
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

<div class="flex flex-col w-full gap-4 p-8">
  <div class="w-full p-6">
    <div class="text-lg font-semibold">Adam walk simulator</div>
    <div class="text-sm text-justify">
      In season 10 of <a
        href="https://youtu.be/v4KHDMvFQgg?t=1841"
        class="text-blue-800 underline"
        target="_blank">Jet Lag: The Game</a
      >
      where two team play a game to control Australia's 8 regions, Adam Chase, member
      of team red takes up on the challenge of sensing the passage of 30 mins with
      the use of a makeshift clock. In order to achieve the given time, Adam measures
      his walking pace and begins to count for exactly 3312 steps which would amount
      to 30 minutes per his walking pace of 1.84 steps/second. Inspired by the game
      show and by another video on Markov chains by
      <a
        href="https://www.youtube.com/@cadaeicstudios"
        class="text-blue-800 underline"
        target="_blank">Cadaeic Studios</a
      >
      on the same challenge, here's the simulation which plots the number of times
      the walk could have ended at a given time, given the target time Adam wanted
      to measure by his makeshift 11-o-clock.<br /> <br />Input parameters are
      as follows:
      <ul class="list-disc">
        <li>
          <b>Initial Steps per second rate:</b> Defines the number of steps per second
          as measured before starting the challenge.
        </li>
        <li>
          <b>Change delta for steps per second:</b> The change in step rate per second
          after every couple of seconds. This is the amount by which the step rate
          can increase or decrease every couple of steps.
        </li>
        <li>
          <b>Probability of change:</b>The probability that the step rate will
          increase or decrease. For probability p the chance of increase is p,
          chance of decrease is p and chance of same step rate is 1-2*p.
        </li>
        <li>
          <b>Target time in minutes:</b> Time which is supposed to be passed and
          measured by our makeshift clock.
        </li>
        <li>
          <b>Challenge range in percent:</b> If the makeshift clock time is called
          within this range of target time then the challenge is successfully completed.
        </li>
        <li>
          <b>Bucket size of bars in seconds:</b> Determines the time range in seconds
          within which all times measured will be counted into.
        </li>
        <li>
          <b>Average time in seconds for change in pace:</b> The average time in
          which the step rate can change.
        </li>
        <li><b>Number of simulations to run:</b> Self explanatory.</li>
      </ul>
    </div>
  </div>
  <div class="w-full grid grid-cols-2 gap-4">
    <div class="min-w-fit">
      <div>Initial steps per second rate</div>
      <Input type="number" bind:value={initialStepRate} min="1" max="3" />
    </div>
    <div class="min-w-fit">
      <div>Change delta for steps per second</div>
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
      <div>Average time in seconds for change in pace</div>
      <Input
        type="number"
        bind:value={averageTimeInSecondForPaceChange}
        min="1"
        max="60"
      />
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
  </div>
  <div class="self-center">
    <Button variant="outline" onclick={stepsTimeCount}
      >Click to get simfun</Button
    >
  </div>
  {#if stepTimeCounterData != null}
    <div class="w-full min-h-full">
      <BarChart data={stepTimeCounterData} height={numberOfSimulations} />
    </div>
  {/if}
</div>
