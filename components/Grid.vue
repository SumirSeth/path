<template>
  <div>
    <canvas ref="gridCanvas" class="grid-canvas border border-spacing-1 border-solid border-slate-600 block m-auto">
    </canvas>
  </div>
</template>

<script lang="ts" setup>

//props
const props = defineProps({
  rows: {
    type: Number,
    default: 20
  },
  columns: {
    type: Number,
    default: 20
  }
})

//grid logic
const gridCanvas = ref<HTMLCanvasElement>()
//each cell needs a state:
//empty, start, end, obstacle/wall
const gridStates = reactive(
  Array.from({
    length: props.rows
  },
  () => Array(props.columns).fill('empty')
  )
)
  
onMounted(() => {
  const canvas = gridCanvas.value
  if (!canvas) return
  const ctx = canvas.getContext("2d")


  //dimensions
  const width = 1000;
  const height = 1000;

  canvas.width = width;
  canvas.height = height;

  //calculate grid properties

  const cellWidth = width / props.columns;
  const cellHeight = height / props.rows;

  //drawing the grid
  if (!ctx) return
  ctx.strokeStyle = "#000";
  

  for (let i = 0; i < props.rows; i++) {
    const x = i * cellWidth;
    ctx.beginPath();
    ctx.moveTo(x, 0);
    ctx.lineTo(x, height);
    ctx.stroke();
  }

  for (let i = 0; i < props.columns; i++) {
    const y = i * cellHeight;
    ctx.beginPath();
    ctx.moveTo(0, y);
    ctx.lineTo(width, y);
    ctx.stroke();
  }

  //listen to mouse events
  gridCanvas.value?.addEventListener("click", handleClick)
})

const handleClick = (e: MouseEvent) => {
  const canvas = gridCanvas.value
  if (!canvas) return
  const rect = canvas.getBoundingClientRect()

  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  
  const cellWidth = canvas.width / props.columns
  const cellHeight = canvas.height / props.rows

  const col = Math.floor(x / cellWidth)
  const row = Math.floor(y / cellHeight)

  gridStates[row][col] = gridStates[row][col] === 'empty' ? 'obstacle' : 'empty'

  drawGrid()

}


function drawGrid() {
  const canvas = gridCanvas.value
  if (!canvas) return
  const ctx = canvas.getContext("2d")
  if (!ctx) return

  // Clear canvas
  ctx.clearRect(0, 0, canvas.width, canvas.height)

  // Dimensions
  const width = canvas.width
  const height = canvas.height
  const cellWidth = width / props.columns
  const cellHeight = height / props.rows

  // Draw cells based on state
  for (let row = 0; row < props.rows; row++) {
    for (let col = 0; col < props.columns; col++) {
      const x = col * cellWidth
      const y = row * cellHeight

      if (gridStates[row][col] === 'obstacle') {
        ctx.fillStyle = '#333'
        ctx.fillRect(x, y, cellWidth, cellHeight)
      }
    }
  }

  // Draw grid lines
  ctx.strokeStyle = "#000"
  for (let i = 0; i <= props.columns; i++) {
    const x = i * cellWidth
    ctx.beginPath()
    ctx.moveTo(x, 0)
    ctx.lineTo(x, height)
    ctx.stroke()
  }

  for (let i = 0; i <= props.rows; i++) {
    const y = i * cellHeight
    ctx.beginPath()
    ctx.moveTo(0, y)
    ctx.lineTo(width, y)
    ctx.stroke()
  }
}
</script>

<style>

</style>