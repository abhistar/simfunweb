<script lang="ts">
  import "./app.css";
  import Simulator from "./lib/Simulator.svelte";
</script>

<main>
  <div class="flex flex-col justify-between">
    <div>
      <div class="bg-emerald-600 h-64 w-full">
        <div class="relative top-8 justify-self-anchor-center">
          <div class="text-white text-center text-6xl font-semibold italic">
            SimFun!
          </div>
          <div class="text-white text-center text-base">
            Simple Simluations For Fun
          </div>
        </div>
      </div>
      <div
        class="md:w-3/4 w-full border-2 rounded-lg shadow-lg relative -top-24 justify-center justify-self-anchor-center opacity-100 bg-white z-10"
      >
        <Simulator />
      </div>
    </div>
    <div
      class="h-32 bg-emerald-600 flex items-center justify-center text-white"
    >
      <div>Made with love: abhistar</div>
    </div>
  </div>
</main>

<style>
  .justify-self-anchor-center {
    justify-self: anchor-center;
  }
</style>
