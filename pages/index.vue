<template>
  <div>
    <div class="controls">
      <button @click="mode = 'start'">Set Start</button>
      <button @click="mode = 'end'">Set End</button>
      <button @click="mode = 'obstacle'">Set Obstacle</button>
    </div>
    <Grid :rows="15" :columns="15" :mode="mode" />
  </div>
</template>

<script setup>
import { ref } from "vue";
import Grid from "~/components/Grid.vue";

const mode = ref("obstacle");
</script>

<style scoped>
.controls {
  margin-bottom: 16px;
}
button {
  margin-right: 8px;
  padding: 8px 16px;
  border: 1px solid #ccc;
  background-color: #f0f0f0;
  cursor: pointer;
}
button:hover {
  background-color: #e0e0e0;
}
</style>
