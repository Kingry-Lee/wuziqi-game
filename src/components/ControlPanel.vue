<template>
  <div class="control-panel">
    <!-- 游戏状态显示 -->
    <div class="game-status">
      <div v-if="gameOver" class="status-gameover">
        <span class="winner-text">
          {{ winner === 1 ? '黑棋' : '白棋' }} 获胜！
        </span>
      </div>
      <div v-else class="status-playing">
        <span :class="['current-player', currentPlayer === 1 ? 'black' : 'white']">
          {{ currentPlayer === 1 ? '黑棋' : '白棋' }} {{ isAIThinking ? '思考中...' : '落子' }}
        </span>
      </div>
    </div>

    <!-- 计时显示 -->
    <div class="timer-display">
      <div class="timer-item" :class="{ active: currentPlayer === 1 }">
        <span class="timer-label">黑棋</span>
        <span class="timer-value">{{ formatTime(gameStats.blackTime) }}</span>
      </div>
      <div class="timer-divider">:</div>
      <div class="timer-item" :class="{ active: currentPlayer === 2 }">
        <span class="timer-label">白棋</span>
        <span class="timer-value">{{ formatTime(gameStats.whiteTime) }}</span>
      </div>
    </div>

    <!-- 操作按钮 -->
    <div class="action-buttons">
      <button class="btn btn-primary" @click="restart">
        重新开始
      </button>
      <button class="btn btn-secondary" @click="undo" :disabled="moveHistory.length === 0 || gameOver">
        悔棋
      </button>
      <button class="btn btn-danger" @click="resign" :disabled="gameOver">
        认输
      </button>
    </div>

    <!-- 菜单位置 -->
    <div class="menu-buttons">
      <button class="btn btn-menu" @click="$emit('open-settings')">
        ⚙️ 设置
      </button>
      <button class="btn btn-menu" @click="$emit('open-history')">
        📋 历史
      </button>
    </div>
  </div>
</template>

<script setup>
import { useGame } from '../composables/useGame'

const { 
  currentPlayer, 
  gameOver, 
  winner, 
  gameStats, 
  moveHistory,
  isAIThinking,
  restart, 
  undo, 
  resign 
} = useGame()

defineEmits(['open-settings', 'open-history'])

function formatTime(seconds) {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
}
</script>

<style scoped>
.control-panel {
  display: flex;
  flex-direction: column;
  gap: 16px;
  padding: 16px;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
}

.game-status {
  text-align: center;
  padding: 12px;
  background: #f5f5f5;
  border-radius: 8px;
}

.status-gameover {
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

.winner-text {
  font-size: 20px;
  font-weight: bold;
  color: #e74c3c;
}

.current-player {
  font-size: 18px;
  font-weight: 500;
}

.current-player.black {
  color: #333;
}

.current-player.white {
  color: #666;
}

.timer-display {
  display: flex;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 8px;
  padding: 12px;
  background: #f5f5f5;
  border-radius: 8px;
}

.timer-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 8px 16px;
  border-radius: 6px;
  transition: all 0.3s;
}

.timer-item.active {
  background: #e8f5e9;
  box-shadow: 0 0 0 2px #4caf50;
}

.timer-label {
  font-size: 12px;
  color: #666;
}

.timer-value {
  font-size: 24px;
  font-weight: bold;
  font-family: monospace;
  color: #333;
}

.timer-divider {
  font-size: 24px;
  font-weight: bold;
}

.action-buttons {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.btn {
  flex: 1;
  min-width: 80px;
  padding: 10px 16px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.btn:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-primary {
  background: #4caf50;
  color: white;
}

.btn-secondary {
  background: #2196f3;
  color: white;
}

.btn-danger {
  background: #f44336;
  color: white;
}

.btn-menu {
  background: #9e9e9e;
  color: white;
}

.menu-buttons {
  display: flex;
  gap: 8px;
}

.menu-buttons .btn {
  flex: 1;
}
</style>
