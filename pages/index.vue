<template>
  <div class="min-h-screen bg-gradient-to-br from-gray-950 to-gray-800 p-8 font-[Outfit]">
    <div class="max-w-4xl mx-auto">
      <!-- Header -->
      <header class="text-center mb-12">
        <h1 class="text-4xl font-bold text-white mb-4">Pathfinding Visualizer</h1>
        <p class="text-gray-400">Explore A* algorithmic pathfinding in real-time</p>
      </header>

      <!-- Mode Selection -->
      <div class="flex justify-center gap-3 mb-6">
        <button
          @click="mode = 'start'"
          :class="[
            'px-4 py-2 rounded-lg font-medium transform transition-all duration-200',
            mode === 'start' 
              ? 'bg-blue-500 text-white'
              : 'bg-gray-700 text-gray-300 hover:bg-gray-600'
          ]"
        >
          Set Start
        </button>
        <button
          @click="mode = 'end'"
          :class="[
            'px-4 py-2 rounded-lg font-medium transform transition-all duration-200',
            mode === 'end'
              ? 'bg-blue-500 text-white'
              : 'bg-gray-700 text-gray-300 hover:bg-gray-600'
          ]"
        >
          Set End
        </button>
        <button
          @click="mode = 'obstacle'"
          :class="[
            'px-4 py-2 rounded-lg font-medium transform transition-all duration-200',
            mode === 'obstacle'
              ? 'bg-blue-500 text-white'
              : 'bg-gray-700 text-gray-300 hover:bg-gray-600'
          ]"
        >
          Draw Walls
        </button>
      </div>

      <!-- Grid Container -->
      <div class="bg-gray-800 rounded-xl p-6 shadow-2xl">
        <Grid ref="grid" :rows="15" :columns="15" :mode="mode" />
        
        <!-- Controls -->
        <div class="flex justify-center gap-4 mt-8">
          <button
            @click="resetGrid"
            class="px-6 py-3 bg-red-500 text-white rounded-lg font-medium
                   transform transition-all duration-200 hover:scale-105
                   hover:bg-red-600 focus:ring-2 focus:ring-red-400
                   active:scale-95"
          >
            Reset Grid
          </button>
          
          <button
            @click="findPath"
            class="px-6 py-3 bg-emerald-500 text-white rounded-lg font-medium
                   transform transition-all duration-200 hover:scale-105
                   hover:bg-emerald-600 focus:ring-2 focus:ring-emerald-400
                   active:scale-95"
          >
            Find Path
          </button>
        </div>
      </div>

      <div class="mt-8 text-gray-400 text-center text-sm">
        <p>Click Draw Walls to create walls • Click Set Start or Set End to set them<br>Click Reset Grid to reset the grid after each use</p>
        <p>Made with ❤️ by <a class="underline italic" href="https://github.com/sumirseth" target="_blank">Sumir</a></p>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import Grid from "../components/Grid.vue";
import { ref } from 'vue';

type GridInstance = InstanceType<typeof Grid>;
type Mode = 'start' | 'end' | 'obstacle';

const mode = ref<Mode>('obstacle');
const grid = ref<GridInstance | null>(null);

onMounted(() => {
  console.log("Grid is ready:", grid.value);
});

const resetGrid = () => {
  if (grid.value) {
    grid.value.resetGrid();
  } else {
    console.error("Grid component not found");
  }
};

const findPath = () => {
  if (grid.value) {
    grid.value.findPath();
  } else {
    console.error("Grid component not found");
  }
};
</script>

