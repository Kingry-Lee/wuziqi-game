<template>
  <div class="game-board" ref="boardContainer">
    <canvas ref="boardCanvas" @click="handleClick" @mousemove="handleMouseMove" @mouseleave="handleMouseLeave"></canvas>
    <!-- 棋子层 -->
    <div class="pieces-layer">
      <div 
        v-for="(pos, index) in piecePositions" 
        :key="'piece'+index"
        class="piece"
        :class="[pos.player === 1 ? 'black' : 'white', { 'last-piece': isLastMove(pos) }]"
        :style="getPieceStyle(pos)"
      >
        <div v-if="isLastMove(pos)" class="last-move-indicator"></div>
      </div>
      <!-- 悬停提示 -->
      <div 
        v-if="hoverPos && !boardData.gameOver && !boardData.isMyTurn"
        class="hover-piece"
        :class="boardData.currentPlayer === 1 ? 'black' : 'white'"
        :style="getPieceStyle(hoverPos)"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'
import { useGame } from '../composables/useGame'

const props = defineProps({
  mode: {
    type: String,
    default: 'ai'
  },
  onlineGame: {
    type: Object,
    default: null
  }
})

const { cells, boardSize, currentPlayer, gameOver, lastMove, isAIThinking, placePiece } = useGame()

const boardContainer = ref(null)
const boardCanvas = ref(null)
const hoverPos = ref(null)

let ctx = null

// 获取所有棋子位置（使用 boardData）
const piecePositions = computed(() => {
  const positions = []
  const cellsData = boardData.value.cells
  const size = boardData.value.boardSize
  
  if (!cellsData || !Array.isArray(cellsData)) return positions
  
  for (let row = 0; row < size; row++) {
    for (let col = 0; col < size; col++) {
      if (cellsData[row] && cellsData[row][col] !== 0) {
        positions.push({ row, col, player: cellsData[row][col] })
      }
    }
  }
  return positions
})

// 根据模式获取棋盘数据
const boardData = computed(() => {
  // 在线模式
  if (props.mode === 'online' && props.onlineGame) {
    return {
      cells: props.onlineGame.cells || [],
      boardSize: props.onlineGame.boardSize || 15,
      currentPlayer: props.onlineGame.currentPlayer || 1,
      gameOver: props.onlineGame.gameOver || false,
      lastMove: props.onlineGame.lastMove || null,
      isMyTurn: props.onlineGame.isMyTurn || false
    }
  }
  // 本地/AI 模式
  return {
    cells: cells.value,
    boardSize: boardSize.value,
    currentPlayer: currentPlayer.value,
    gameOver: gameOver.value,
    lastMove: lastMove.value,
    isMyTurn: true
  }
})

function isLastMove(pos) {
  return boardData.value.lastMove && boardData.value.lastMove.row === pos.row && boardData.value.lastMove.col === pos.col
}

function getPieceStyle(pos) {
  const cellSize = 100 / boardData.value.boardSize
  const offset = cellSize / 2
  return {
    left: `${pos.col * cellSize + offset}%`,
    top: `${pos.row * cellSize + offset}%`,
  }
}

function drawBoard() {
  if (!boardCanvas.value || !ctx) {
    // 尝试重新获取 context
    if (boardCanvas.value) {
      ctx = boardCanvas.value.getContext('2d')
    }
    if (!ctx) return
  }
  
  const canvas = boardCanvas.value
  const container = boardContainer.value
  if (!container) return
  
  // 确保 canvas 尺寸与容器一致
  const rect = container.getBoundingClientRect()
  if (rect.width === 0 || rect.height === 0) return
  
  // 确保 boardSize 有效
  const size = boardData.value.boardSize
  if (!size || size < 2) return
  
  canvas.width = rect.width
  canvas.height = rect.height
  
  const padding = 20
  const gridSize = (Math.min(canvas.width, canvas.height) - padding * 2) / (size - 1)
  
  // 清空画布
  ctx.fillStyle = '#DEB887'
  ctx.fillRect(0, 0, canvas.width, canvas.height)
  
  // 画网格线
  ctx.strokeStyle = '#000'
  ctx.lineWidth = 1
  
  for (let i = 0; i < size; i++) {
    const pos = padding + i * gridSize
    
    // 横线
    ctx.beginPath()
    ctx.moveTo(padding, pos)
    ctx.lineTo(padding + (size - 1) * gridSize, pos)
    ctx.stroke()
    
    // 竖线
    ctx.beginPath()
    ctx.moveTo(pos, padding)
    ctx.lineTo(pos, padding + (size - 1) * gridSize)
    ctx.stroke()
  }
  
  // 画星位
  const starPoints = getStarPoints(size)
  ctx.fillStyle = '#000'
  for (const star of starPoints) {
    const x = padding + star.col * gridSize
    const y = padding + star.row * gridSize
    ctx.beginPath()
    ctx.arc(x, y, 5, 0, Math.PI * 2)
    ctx.fill()
  }
}

function getStarPoints(size) {
  if (size < 13) return []
  
  const stars = []
  const offsets = [
    { r: 3, c: 3 },
    { r: 3, c: size - 4 },
    { r: size - 4, c: 3 },
    { r: size - 4, c: size - 4 },
  ]
  
  if (size === 15 || size === 19) {
    const center = Math.floor(size / 2)
    stars.push({ row: center, col: center })
  }
  
  offsets.forEach(o => {
    if (o.r >= 0 && o.r < size && o.c >= 0 && o.c < size) {
      stars.push({ row: o.r, col: o.c })
    }
  })
  
  return stars
}

function handleClick(e) {
  // 在线模式检查
  if (props.mode === 'online' && props.onlineGame) {
    // 在线对战：检查是否轮到自己
    if (!props.onlineGame.gameStarted || props.onlineGame.gameOver || !props.onlineGame.isMyTurn) return
    
    const rect = boardCanvas.value.getBoundingClientRect()
    const x = e.clientX - rect.left
    const y = e.clientY - rect.top
    
    const size = props.onlineGame.boardSize || 15
    const padding = 20
    const gridSize = (Math.min(rect.width, rect.height) - padding * 2) / (size - 1)
    
    const col = Math.round((x - padding) / gridSize)
    const row = Math.round((y - padding) / gridSize)
    
    if (row >= 0 && row < size && col >= 0 && col < size) {
      if (props.onlineGame.cells[row][col] === 0) {
        props.onlineGame.placePiece(row, col)
      }
    }
    return
  }
  
  // 本地/AI 模式
  if (gameOver.value || isAIThinking.value) return
  
  const rect = boardCanvas.value.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  
  const size = boardSize.value
  const padding = 20
  const gridSize = (Math.min(rect.width, rect.height) - padding * 2) / (size - 1)
  
  const col = Math.round((x - padding) / gridSize)
  const row = Math.round((y - padding) / gridSize)
  
  if (row >= 0 && row < size && col >= 0 && col < size) {
    if (cells.value[row][col] === 0) {
      placePiece(row, col)
    }
  }
}

function handleMouseMove(e) {
  // 在线模式检查
  if (props.mode === 'online' && props.onlineGame) {
    if (!props.onlineGame.gameStarted || props.onlineGame.gameOver || !props.onlineGame.isMyTurn) {
      hoverPos.value = null
      return
    }
    
    const rect = boardCanvas.value.getBoundingClientRect()
    const x = e.clientX - rect.left
    const y = e.clientY - rect.top
    
    const size = props.onlineGame.boardSize || 15
    const padding = 20
    const gridSize = (Math.min(rect.width, rect.height) - padding * 2) / (size - 1)
    
    const col = Math.round((x - padding) / gridSize)
    const row = Math.round((y - padding) / gridSize)
    
    if (row >= 0 && row < size && col >= 0 && col < size) {
      if (props.onlineGame.cells[row][col] === 0) {
        hoverPos.value = { row, col }
      } else {
        hoverPos.value = null
      }
    } else {
      hoverPos.value = null
    }
    return
  }
  
  // 本地/AI 模式
  if (boardData.value.gameOver || isAIThinking.value) {
    hoverPos.value = null
    return
  }
  
  const rect = boardCanvas.value.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  
  const size = boardData.value.boardSize
  const padding = 20
  const gridSize = (Math.min(rect.width, rect.height) - padding * 2) / (size - 1)
  
  const col = Math.round((x - padding) / gridSize)
  const row = Math.round((y - padding) / gridSize)
  
  if (row >= 0 && row < size && col >= 0 && col < size) {
    if (boardData.value.cells[row][col] === 0) {
      hoverPos.value = { row, col }
    } else {
      hoverPos.value = null
    }
  } else {
    hoverPos.value = null
  }
}

function handleMouseLeave() {
  hoverPos.value = null
}

function handleResize() {
  drawBoard()
}

// 使用 ResizeObserver 监听容器大小变化
let resizeObserver = null

// 监听棋盘数据变化
watch(() => boardData.value.boardSize, () => {
  setTimeout(drawBoard, 100)
})

watch(() => boardData.value.cells, () => {
  // 棋子位置通过Vue渲染，不需要重画
}, { deep: true })

// 监听模式变化，确保重绘
watch(() => props.mode, () => {
  setTimeout(drawBoard, 100)
})

watch(() => props.onlineGame?.gameStarted, () => {
  setTimeout(drawBoard, 100)
})

onMounted(() => {
  // 等待 DOM 渲染完成后初始化 canvas
  setTimeout(() => {
    if (boardCanvas.value) {
      ctx = boardCanvas.value.getContext('2d')
      drawBoard()
      
      // 使用 ResizeObserver 监听容器大小变化
      if (boardContainer.value) {
        resizeObserver = new ResizeObserver(() => {
          drawBoard()
        })
        resizeObserver.observe(boardContainer.value)
      }
    }
  }, 50)
})

onUnmounted(() => {
  if (resizeObserver) {
    resizeObserver.disconnect()
  }
  window.removeEventListener('resize', handleResize)
})
</script>

<style scoped>
.game-board {
  position: relative;
  width: 100%;
  aspect-ratio: 1;
  border-radius: 8px;
  box-shadow: 
    0 4px 20px rgba(0, 0, 0, 0.3),
    inset 0 0 0 3px #8B4513;
  overflow: hidden;
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  cursor: pointer;
}

.pieces-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.piece {
  position: absolute;
  width: 6%;
  height: 6%;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  box-shadow: 
    2px 2px 4px rgba(0, 0, 0, 0.4),
    -1px -1px 2px rgba(255, 255, 255, 0.2);
  pointer-events: none;
  transition: transform 0.1s ease;
}

.piece.black {
  background: radial-gradient(circle at 30% 30%, #444, #000);
}

.piece.white {
  background: radial-gradient(circle at 30% 30%, #fff, #ddd);
}

.piece.last-piece {
  transform: translate(-50%, -50%) scale(1.15);
}

.last-move-indicator {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 30%;
  height: 30%;
  border-radius: 50%;
  transform: translate(-50%, -50%);
}

.piece.black .last-move-indicator {
  background: #fff;
}

.piece.white .last-move-indicator {
  background: #000;
}

.hover-piece {
  position: absolute;
  width: 6%;
  height: 6%;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  opacity: 0.5;
  pointer-events: none;
}

.hover-piece.black {
  background: radial-gradient(circle at 30% 30%, #444, #000);
}

.hover-piece.white {
  background: radial-gradient(circle at 30% 30%, #fff, #ddd);
}
</style>
