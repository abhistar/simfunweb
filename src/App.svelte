<script lang="ts">
  import "./app.css";
  import svelteLogo from "./assets/svelte.svg";
  import viteLogo from "/vite.svg";
  import Simulator from "./lib/Simulator.svelte";
</script>

<main>
  <div class="h-full justify-center">
    <div class="bg-teal-600 h-64 w-full">
      <div class="absolute top-8 justify-self-anchor-center">
        <div class="text-white text-center text-6xl font-semibold italic">
          SimFun!
        </div>
        <div class="text-white text-center text-base">
          Simple Simluations For Fun
        </div>
      </div>
    </div>
    <div
      class="w-3/4 border-2 rounded-lg shadow-lg justify-center absolute top-32 justify-self-anchor-center opacity-100 bg-white"
    >
      <Simulator />
    </div>
  </div>
</main>

<style>
  .justify-self-anchor-center {
    justify-self: anchor-center;
  }
</style>
