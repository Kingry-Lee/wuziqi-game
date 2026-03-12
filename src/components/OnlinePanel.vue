<template>
  <div class="online-panel">
    <!-- 连接状态 -->
    <div v-if="!connected" class="connect-section">
      <h3>在线对战</h3>
      <div class="server-input">
        <input 
          v-model="serverUrl" 
          placeholder="服务器地址"
          class="input"
        />
        <button @click="handleConnect" class="btn btn-primary" :disabled="connecting">
          {{ connecting ? '连接中...' : '连接' }}
        </button>
      </div>
      <p v-if="error" class="error">{{ error }}</p>
    </div>

    <!-- 已连接但未在房间 -->
    <div v-else-if="!roomId" class="room-section">
      <h3>在线对战</h3>
      
      <!-- 创建房间 -->
      <div class="create-room">
        <button @click="handleCreateRoom" class="btn btn-primary" :disabled="creating">
          {{ creating ? '创建中...' : '创建房间' }}
        </button>
      </div>
      
      <div class="divider">或</div>
      
      <!-- 加入房间 -->
      <div class="join-room">
        <input 
          v-model="joinRoomId" 
          placeholder="输入房间号"
          class="input"
          @keyup.enter="handleJoinRoom"
        />
        <button @click="handleJoinRoom" class="btn btn-secondary" :disabled="joining || !joinRoomId">
          {{ joining ? '加入中...' : '加入' }}
        </button>
      </div>
      
      <p v-if="error" class="error">{{ error }}</p>
    </div>

    <!-- 在房间中 -->
    <div v-else class="in-room-section">
      <div class="room-info">
        <div class="room-id">
          房间号: <strong>{{ roomId }}</strong>
        </div>
        <div class="player-info">
          <span :class="['player-badge', player === 1 ? 'black' : 'white']">
            {{ player === 1 ? '黑棋 (你)' : '白棋 (你)' }}
          </span>
        </div>
        <div v-if="!gameStarted" class="waiting">
          <p v-if="player === 1">等待对手加入...</p>
          <p v-else>已进入房间</p>
        </div>
      </div>

      <!-- 游戏状态 -->
      <div v-if="gameStarted" class="game-status">
        <div v-if="gameOver" class="status-gameover">
          <span class="winner-text">
            {{ winner === player ? '你赢了!' : '你输了' }}
          </span>
        </div>
        <div v-else class="status-playing">
          <span :class="['current-player', isMyTurn ? 'your-turn' : '']">
            {{ isMyTurn ? '你的回合' : '对方回合' }}
          </span>
        </div>
      </div>

      <!-- 操作按钮 -->
      <div class="action-buttons">
        <button 
          v-if="gameStarted && isMyTurn && !gameOver" 
          @click="requestUndo" 
          class="btn btn-secondary"
        >
          请求悔棋
        </button>
        
        <button 
          v-if="gameStarted && !gameOver" 
          @click="resign" 
          class="btn btn-danger"
        >
          认输
        </button>
        
        <button 
          v-if="gameStarted && gameOver" 
          @click="requestRestart" 
          class="btn btn-primary"
        >
          重新开始
        </button>
        
        <button @click="leaveRoom" class="btn btn-menu">
          离开房间
        </button>
      </div>

      <!-- 对方请求悔棋 -->
      <div v-if="waitingForUndo" class="confirm-dialog">
        <p>对方请求悔棋</p>
        <div class="dialog-buttons">
          <button @click="acceptUndo" class="btn btn-primary">同意</button>
          <button @click="rejectUndo" class="btn btn-secondary">拒绝</button>
        </div>
      </div>

      <!-- 对方请求重新开始 -->
      <div v-if="waitingForRestart" class="confirm-dialog">
        <p>对方请求重新开始</p>
        <div class="dialog-buttons">
          <button @click="acceptRestart" class="btn btn-primary">同意</button>
          <button @click="rejectRestart" class="btn btn-secondary">拒绝</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useOnlineGame } from '../composables/useOnlineGame'

const {
  connected,
  roomId,
  player,
  gameStarted,
  isMyTurn,
  gameOver,
  winner,
  waitingForUndo,
  waitingForRestart,
  error,
  connect,
  disconnect,
  createRoom,
  joinRoom,
  placePiece,
  requestUndo,
  acceptUndo,
  rejectUndo,
  resign,
  requestRestart,
  acceptRestart,
  rejectRestart,
} = useOnlineGame()

const serverUrl = ref('http://localhost:3001')
const joinRoomId = ref('')
const connecting = ref(false)
const creating = ref(false)
const joining = ref(false)

async function handleConnect() {
  connecting.value = true
  error.value = null
  connect(serverUrl.value)
  
  // 等待连接
  await new Promise(resolve => setTimeout(resolve, 1000))
  connecting.value = false
}

function handleCreateRoom() {
  creating.value = true
  error.value = null
  createRoom((response) => {
    creating.value = false
    if (!response.success) {
      error.value = response.error
    }
  })
}

function handleJoinRoom() {
  if (!joinRoomId.value) return
  joining.value = true
  error.value = null
  joinRoom(joinRoomId.value, (response) => {
    joining.value = false
    if (!response.success) {
      error.value = response.error
    }
  })
}

function leaveRoom() {
  disconnect()
  joinRoomId.value = ''
}

// 暴露给父组件使用
defineExpose({
  placePiece,
  getBoard: () => null, // 在线模式不使用本地棋盘
  isOnlineMode: true,
  player,
  isMyTurn,
  gameOver,
  winner,
})
</script>

<style scoped>
.online-panel {
  padding: 16px;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
}

h3 {
  margin: 0 0 16px 0;
  text-align: center;
  color: #333;
}

.connect-section, .room-section, .in-room-section {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.input {
  flex: 1;
  padding: 10px 12px;
  border: 2px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
}

.input:focus {
  outline: none;
  border-color: #4caf50;
}

.server-input, .create-room, .join-room {
  display: flex;
  gap: 8px;
}

.divider {
  text-align: center;
  color: #999;
  font-size: 12px;
}

.room-info {
  text-align: center;
  padding: 12px;
  background: #f5f5f5;
  border-radius: 8px;
}

.room-id {
  font-size: 16px;
  margin-bottom: 8px;
}

.room-id strong {
  font-size: 20px;
  color: #4caf50;
  letter-spacing: 2px;
}

.player-badge {
  display: inline-block;
  padding: 4px 12px;
  border-radius: 4px;
  font-size: 14px;
}

.player-badge.black {
  background: #333;
  color: #fff;
}

.player-badge.white {
  background: #fff;
  color: #333;
  border: 1px solid #ddd;
}

.waiting {
  margin-top: 8px;
  color: #666;
}

.game-status {
  text-align: center;
  padding: 12px;
  background: #f5f5f5;
  border-radius: 8px;
}

.winner-text {
  font-size: 18px;
  font-weight: bold;
  color: #e74c3c;
}

.current-player {
  font-size: 16px;
  color: #666;
}

.current-player.your-turn {
  color: #4caf50;
  font-weight: bold;
}

.action-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.btn {
  flex: 1;
  min-width: 80px;
  padding: 10px 12px;
  border: none;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.btn:hover:not(:disabled) {
  transform: translateY(-1px);
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

.error {
  color: #f44336;
  font-size: 13px;
  text-align: center;
}

.confirm-dialog {
  margin-top: 12px;
  padding: 12px;
  background: #fff3e0;
  border-radius: 8px;
  text-align: center;
}

.confirm-dialog p {
  margin: 0 0 8px 0;
  color: #e65100;
}

.dialog-buttons {
  display: flex;
  gap: 8px;
}

.dialog-buttons .btn {
  flex: 1;
}
</style>
