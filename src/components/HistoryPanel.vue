<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="history-panel">
      <div class="panel-header">
        <h2>对局历史</h2>
        <button class="close-btn" @click="$emit('close')">×</button>
      </div>
      
      <div class="panel-content">
        <div v-if="history.length === 0" class="empty-state">
          <p>暂无对局记录</p>
        </div>
        
        <div v-else class="history-list">
          <div v-for="record in history" :key="record.id" class="history-item">
            <div class="history-info">
              <span class="history-mode">{{ getModeText(record.gameMode) }}</span>
              <span class="history-result" :class="getWinnerClass(record)">
                {{ record.winner }} 胜
              </span>
            </div>
            <div class="history-details">
              <span>{{ record.boardSize }}×{{ record.boardSize }}</span>
              <span>{{ record.totalMoves }}手</span>
              <span>{{ formatTime(record.totalTime) }}</span>
            </div>
            <div class="history-date">{{ record.date }}</div>
            <button class="delete-btn" @click="deleteRecord(record.id)">删除</button>
          </div>
        </div>
      </div>

      <div v-if="history.length > 0" class="panel-footer">
        <button class="btn btn-danger" @click="clearAll">清空记录</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useGame } from '../composables/useGame'

defineEmits(['close'])

const { getHistory, deleteHistory, clearHistory } = useGame()

const history = ref([])

onMounted(() => {
  history.value = getHistory()
})

function getModeText(mode) {
  return mode === 'ai' ? '人机' : '双人对战'
}

function getWinnerClass(record) {
  if (record.winner.includes('玩家') || record.winner === '黑棋') {
    return 'win'
  }
  return 'lose'
}

function formatTime(seconds) {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins}:${secs.toString().padStart(2, '0')}`
}

function deleteRecord(id) {
  deleteHistory(id)
  history.value = getHistory()
}

function clearAll() {
  if (confirm('确定要清空所有对局记录吗？')) {
    clearHistory()
    history.value = []
  }
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px;
}

.history-panel {
  background: white;
  border-radius: 16px;
  width: 100%;
  max-width: 480px;
  max-height: 80vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  border-bottom: 1px solid #eee;
}

.panel-header h2 {
  margin: 0;
  font-size: 18px;
  color: #333;
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  color: #999;
  cursor: pointer;
  padding: 0;
  width: 30px;
  height: 30px;
}

.panel-content {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
}

.empty-state {
  text-align: center;
  padding: 40px 20px;
  color: #999;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.history-item {
  background: #f9f9f9;
  border-radius: 8px;
  padding: 12px;
  position: relative;
}

.history-info {
  display: flex;
  justify-content: space-between;
  margin-bottom: 6px;
}

.history-mode {
  font-weight: 500;
  color: #333;
}

.history-result {
  font-weight: 500;
}

.history-result.win {
  color: #4caf50;
}

.history-result.lose {
  color: #f44336;
}

.history-details {
  display: flex;
  gap: 12px;
  font-size: 12px;
  color: #666;
  margin-bottom: 4px;
}

.history-date {
  font-size: 11px;
  color: #999;
}

.delete-btn {
  position: absolute;
  top: 12px;
  right: 12px;
  background: none;
  border: none;
  color: #f44336;
  font-size: 12px;
  cursor: pointer;
  padding: 4px 8px;
}

.delete-btn:hover {
  text-decoration: underline;
}

.panel-footer {
  padding: 16px 20px;
  border-top: 1px solid #eee;
}

.btn {
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
}

.btn-danger {
  background: #f44336;
  color: white;
}

.btn-danger:hover {
  background: #e53935;
}
</style>
