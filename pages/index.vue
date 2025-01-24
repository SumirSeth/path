<template>
  <div>
    <div class="controls py-5 mb-2">
      <button @click="mode = 'start'">Set Start</button>
      <button @click="mode = 'end'">Set End</button>
      <button @click="mode = 'obstacle'">Set Obstacle</button>
      <button @click="resetGrid">Reset Grid</button>
    </div>
    <Grid ref="grid" :rows="15" :columns="15" :mode="mode" />
  </div>
</template>

<script lang="ts" setup>
import Grid from "../components/Grid.vue";
import { ref } from 'vue';

type GridInstance = InstanceType<typeof Grid>;

const mode = ref("obstacle");
const grid = ref<GridInstance | null>(null);

onMounted(() => {
  console.log("Grid is ready:", grid.value);
});


const resetGrid = () => {
  console.log(grid.value); // Should log the Grid instance
  if (grid.value) {
    grid.value.resetGrid();
  } else {
    console.error("Grid component not found");
  }
};


</script>

<style scoped>
button {
  @apply bg-blue-600 text-white rounded-md p-3 mx-2 hover:bg-blue-800
}
</style>
