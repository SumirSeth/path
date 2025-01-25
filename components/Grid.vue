<template>
  <div ref="containerRef" class="w-full aspect-square max-w-2xl mx-auto">
    <canvas 
      ref="gridCanvas"
      class="w-full h-full"
    ></canvas>
  </div>
</template>

<script lang="ts" setup>
import { defineExpose } from 'vue'
import { ref, reactive, nextTick, onMounted, onUnmounted } from 'vue'

//props
const props = defineProps({
  rows: {
    type: Number,
    default: 20
  },
  columns: {
    type: Number,
    default: 20
  },
  mode: {
    type: String,
    default: 'obstacle'
  }
})

const getRandomPosition = () => ({
  row: Math.floor(Math.random() * props.rows),
  col: Math.floor(Math.random() * props.columns)
})
const initializeStartEnd = () => {
  // Get random start position
  const start = getRandomPosition()
  let end
  
  do {
    end = getRandomPosition()
  } while (end.row === start.row && end.col === start.col)
  
  gridStates[start.row][start.col].state = 'start'
  gridStates[end.row][end.col].state = 'end'
}

//grid logic
const containerRef = ref<HTMLDivElement>()
const gridCanvas = ref<HTMLCanvasElement>()
//each cell needs a state:
//empty, start, end, obstacle/obstacle
const gridStates = reactive(Array(props.rows).fill(null).map(() => 
  Array(props.columns).fill(null).map(() => ({
    state: 'empty',
    gCost: Infinity,
    hCost: Infinity,
    fCost: Infinity,
    parent: null as { row: number; col: number; } | null
  }))
))
  
onMounted(() => {
  const resizeObserver = new ResizeObserver(updateCanvasSize);
  if (containerRef.value) {
    resizeObserver.observe(containerRef.value);
  }
  updateCanvasSize();
  gridCanvas.value?.addEventListener("click", handleClick)
  gridCanvas.value?.addEventListener("mousemove", handleMouseMove)
  gridCanvas.value?.addEventListener("mouseleave", () => {
    hoveredCell.value = [-1, -1]
    drawGrid()
  })
  initializeStartEnd(); 
})

onUnmounted(() => {
  if (containerRef.value) {
    resizeObserver.disconnect();
  }
});

const updateCanvasSize = () => {
  const container = containerRef.value;
  const canvas = gridCanvas.value;
  if (!container || !canvas) return;

  const rect = container.getBoundingClientRect();
  const size = Math.min(rect.width, rect.height);
  
  canvas.width = size;
  canvas.height = size;
  
  drawGrid();
};

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

  if (props.mode === 'start') {
    for (let i = 0; i < props.rows; i++) {
      for(let j = 0; j < props.columns; j++) {
        if (gridStates[i][j].state === 'start') {
          gridStates[i][j].state = 'empty';
        }
      }
    }
    gridStates[row][col].state = 'start';
  } else if (props.mode === 'end') {
    for (let i = 0; i < props.rows; i++) {
      for(let j = 0; j < props.columns; j++) {
        if (gridStates[i][j].state === 'end') {
          gridStates[i][j].state = 'empty';
        }
      }
    }
    gridStates[row][col].state = 'end';
  }
   else if (props.mode === 'obstacle') {
      gridStates[row][col].state = gridStates[row][col].state === 'empty' ? 'obstacle' : 'empty'
  }



  drawGrid()

}

const drawCell = (ctx: CanvasRenderingContext2D, row: number, col: number) => {
  const cellWidth = ctx.canvas.width / props.columns;
  const cellHeight = ctx.canvas.height / props.rows;
  const x = col * cellWidth;
  const y = row * cellHeight;
  
  // Draw cell background based on state
  const state = gridStates[row][col].state;
  
  // Draw cell text
  if (state === 'start' || state === 'end') {
    ctx.fillStyle = '#ffffff';
    ctx.font = `${cellWidth * 0.55}px Sans-Serif`;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    const text = state === 'start' ? 'S' : 'E';
    ctx.fillText(text, x + cellWidth/2, y + cellHeight/2);
  }
}


const drawGrid = () => {
  const canvas = gridCanvas.value
  if (!canvas) return
  const ctx = canvas.getContext("2d")
  if (!ctx) return

  //clear canvas
  ctx.clearRect(0, 0, canvas.width, canvas.height)

  

  //dimensions
  const width = canvas.width
  const height = canvas.height
  const cellWidth = width / props.columns
  const cellHeight = height / props.rows

  
  //draw cells based on state
  for (let row = 0; row < props.rows; row++) {
    for (let col = 0; col < props.columns; col++) {
      const x = col * cellWidth
      const y = row * cellHeight

      if (gridStates[row][col].state === 'obstacle') {
        ctx.fillStyle = 'blue'
        ctx.fillRect(x, y, cellWidth, cellHeight)
      } else if (gridStates[row][col].state === 'start') {
        ctx.fillStyle = 'green'
        ctx.fillRect(x, y, cellWidth, cellHeight)
      } else if (gridStates[row][col].state === 'end') {
        ctx.fillStyle = 'red'
        ctx.fillRect(x, y, cellWidth, cellHeight)
      // } else if (gridStates[row][col].state === 'empty') {
      //   ctx.fillStyle = 'black'
        ctx.fillRect(x, y, cellWidth, cellHeight)
      } else if (gridStates[row][col].state === 'path') {
        ctx.fillStyle = 'yellow'
        ctx.fillRect(x, y, cellWidth, cellHeight)
      } else if (gridStates[row][col].state === 'explored') {
        ctx.fillStyle = 'brown'
        ctx.fillRect(x, y, cellWidth, cellHeight)
      } else if (gridStates[row][col].state === 'open') {
        ctx.fillStyle = 'orange'
        ctx.fillRect(x, y, cellWidth, cellHeight)
      } 
      if (hoveredCell.value[0] === row && hoveredCell.value[1] === col) {
        ctx.fillStyle = "rgba(100, 100, 100, 0.3)"
        ctx.fillRect(x, y, cellWidth, cellHeight)
      }
    }
  }

  //drawing the grid lines
  ctx.strokeStyle = "#fff"
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
  for (let row = 0; row < props.rows; row++) {
    for (let col = 0; col < props.columns; col++) {
      drawCell(ctx, row, col);
    }
  }
}


//visual feedback for the user

const hoveredCell = ref<[number, number]>([-1, -1])

const handleMouseMove = (e: MouseEvent) => {
  const canvas = gridCanvas.value
  if (!canvas) return
  const rect = canvas.getBoundingClientRect()

  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  
  const cellWidth = canvas.width / props.columns
  const cellHeight = canvas.height / props.rows

  const col = Math.floor(x / cellWidth)
  const row = Math.floor(y / cellHeight)

  hoveredCell.value = [row, col]

  drawGrid()
}

//reset the grid
const resetGrid = () => {
  for (let i = 0; i < props.rows; i++) {
    for(let j = 0; j < props.columns; j++) {
      gridStates[i][j] = {
        state: 'empty',
        gCost: Infinity,
        hCost: Infinity,
        fCost: Infinity,
        parent: null
      }
    }
  }
  drawGrid()
}
const resetNonStartEndGrid = () => {
  for (let i = 0; i < props.rows; i++) {
    for(let j = 0; j < props.columns; j++) {
      if (gridStates[i][j].state === 'path' || gridStates[i][j].state === 'explored') {
        gridStates[i][j] = {
          state: 'empty',
          gCost: Infinity,
          hCost: Infinity,
          fCost: Infinity,
          parent: null
        }
      }
    }
  }
  drawGrid()
}

//utils
const calculateHeuristic = (x1: number, y1: number, x2: number, y2: number) => {
  return Math.abs(x1 - x2) + Math.abs(y1 - y2)
}
const sleep = (ms: number) => {new Promise(resolve => setTimeout(resolve, ms))};

//A* algorithm
const openList = ref<{row: number; col: number}[]>([])
const closedList = reactive(new Set<string>())

const cellKey = (row: number, col: number) => `${row},${col}`

const findPath = async () => {
  resetNonStartEndGrid();
  openList.value = [];
  closedList.clear();
  
  let start = null, end = null;
  
  // Reset and find start/end while preserving obstacles
  for (let row = 0; row < props.rows; row++) {
    for (let col = 0; col < props.columns; col++) {
      const currentState = gridStates[row][col].state;
      
      if (currentState === 'obstacle') {
        continue; // Skip obstacles completely
      }
      
      // Reset only non-obstacle cells
      gridStates[row][col].gCost = Infinity;
      gridStates[row][col].hCost = Infinity;
      gridStates[row][col].fCost = Infinity;
      gridStates[row][col].parent = null;
      
      if (currentState === 'start') start = {row, col};
      if (currentState === 'end') end = {row, col};
    }
  }

  if (!start || !end) {
    alert("Start or End point is missing!");
    return;
  };

  // Initialize start
  gridStates[start.row][start.col] = {
    ...gridStates[start.row][start.col],
    gCost: 0,
    hCost: calculateHeuristic(start.row, start.col, end.row, end.col),
    fCost: calculateHeuristic(start.row, start.col, end.row, end.col),
  };
  
  openList.value.push(start);

  while (openList.value.length > 0) {
    // Get current node with lowest fCost
    const currentIndex = openList.value.reduce((lowest, current, index) => 
      gridStates[openList.value[lowest].row][openList.value[lowest].col].fCost > 
      gridStates[current.row][current.col].fCost ? index : lowest, 0);
    
    const current = openList.value[currentIndex];
    
    // Check if reached end
    if (current.row === end.row && current.col === end.col) {
      await reconstructPath(current);
      return;
    }

    // Move to closed list
    openList.value.splice(currentIndex, 1);
    closedList.add(cellKey(current.row, current.col));

    // Mark as visited if not special cell
    const currentState = gridStates[current.row][current.col].state;
    if (!['start', 'end', 'obstacle'].includes(currentState)) {
      gridStates[current.row][current.col].state = 'visited';
      await nextTick();
      await sleep(10);
    }

    // Get valid neighbors
    const neighbors = [
      {row: current.row - 1, col: current.col},
      {row: current.row + 1, col: current.col},
      {row: current.row, col: current.col - 1},
      {row: current.row, col: current.col + 1}
    ].filter(n => {
      // Validate boundaries
      if (n.row < 0 || n.row >= props.rows || n.col < 0 || n.col >= props.columns) {
        return false;
      }
      
      // Check obstacle and visited states
      const neighborState = gridStates[n.row][n.col].state;
      if (neighborState === 'obstacle' || closedList.has(cellKey(n.row, n.col))) {
        return false;
      }
      
      return true;
    });

    // Process valid neighbors
    for (const neighbor of neighbors) {
      const tentativeGCost = gridStates[current.row][current.col].gCost + 1;
      const neighborState = gridStates[neighbor.row][neighbor.col].state;
      const inOpenList = openList.value.some(n => n.row === neighbor.row && n.col === neighbor.col);

      if (!inOpenList || tentativeGCost < gridStates[neighbor.row][neighbor.col].gCost) {
        // Update costs
        gridStates[neighbor.row][neighbor.col] = {
          ...gridStates[neighbor.row][neighbor.col],
          parent: current,
          gCost: tentativeGCost,
          hCost: calculateHeuristic(neighbor.row, neighbor.col, end.row, end.col),
        };
        gridStates[neighbor.row][neighbor.col].fCost = 
          gridStates[neighbor.row][neighbor.col].gCost + 
          gridStates[neighbor.row][neighbor.col].hCost;

        if (!inOpenList) {
          openList.value.push(neighbor);
          // Only update state if not a special cell
          if (!['start', 'end', 'obstacle'].includes(neighborState)) {
            await updateCellState(neighbor.row, neighbor.col, 'exploring');
            await sleep(10);
          }
        }
      }
    }
  }
  alert("No path found!");
};

const updateCellState = async (row: number, col: number, newState: string) => {
  gridStates[row][col].state = newState;
  await nextTick();
}

const reconstructPath = async (endNode: {row: number; col: number}) => {
  let current: {row: number; col: number} | null = endNode;
  const pathNodes: {row: number; col: number}[] = [];
  
  while (current) {
    pathNodes.unshift(current);
    current = gridStates[current.row][current.col].parent;
  }
  
  for (const node of pathNodes) {
    if (gridStates[node.row][node.col].state !== 'start' && 
        gridStates[node.row][node.col].state !== 'end') {
      await updateCellState(node.row, node.col, 'path');
      await sleep(100);
      drawGrid();
    }
  }
};

//funcs
defineExpose({
  resetGrid,
  findPath
})

</script>

<style>

</style>