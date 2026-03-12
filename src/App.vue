<script setup>
import { ref, onMounted, provide } from 'vue'
import GameBoard from './components/GameBoard.vue'
import ControlPanel from './components/ControlPanel.vue'
import OnlinePanel from './components/OnlinePanel.vue'
import SettingsPanel from './components/SettingsPanel.vue'
import HistoryPanel from './components/HistoryPanel.vue'
import { useGame } from './composables/useGame'
import { useOnlineGame } from './composables/useOnlineGame'

const { initGame, setConfig } = useGame()
const onlineGame = useOnlineGame()

const showSettings = ref(false)
const showHistory = ref(false)
const gameMode = ref('ai') // 'ai', 'local', 'online'

// 提供给子组件使用
provide('gameMode', gameMode)
provide('onlineGame', onlineGame)

onMounted(() => {
  initGame()
})

function handleModeChange(mode) {
  gameMode.value = mode
  // 将游戏模式传递给 ConfigManager
  if (mode === 'ai' || mode === 'local') {
    setConfig('gameMode', mode)
    initGame()
  }
}
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <h1>五子棋</h1>
      <!-- 模式选择 -->
      <div class="mode-selector">
        <button 
          :class="['mode-btn', { active: gameMode === 'ai' }]"
          @click="handleModeChange('ai')"
        >
          人机对战
        </button>
        <button 
          :class="['mode-btn', { active: gameMode === 'local' }]"
          @click="handleModeChange('local')"
        >
          双人模式
        </button>
        <button 
          :class="['mode-btn', { active: gameMode === 'online' }]"
          @click="handleModeChange('online')"
        >
          在线对战
        </button>
      </div>
    </header>
    
    <main class="game-container">
      <div class="board-wrapper">
        <GameBoard :mode="gameMode" :onlineGame="gameMode === 'online' ? onlineGame : null" />
      </div>
      
      <aside class="sidebar">
        <!-- AI/本地模式 -->
        <ControlPanel 
          v-if="gameMode !== 'online'"
          @open-settings="showSettings = true"
          @open-history="showHistory = true"
        />
        
        <!-- 在线模式 -->
        <OnlinePanel v-else />
      </aside>
    </main>

    <!-- 设置面板 -->
    <SettingsPanel 
      v-if="showSettings && gameMode !== 'online'" 
      @close="showSettings = false"
      @apply="initGame"
    />

    <!-- 历史面板 -->
    <HistoryPanel 
      v-if="showHistory" 
      @close="showHistory = false"
    />
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

#app {
  min-height: 100vh;
}
</style>

<style scoped>
.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  text-align: center;
  padding: 16px 0;
  margin-bottom: 20px;
}

.app-header h1 {
  color: white;
  font-size: 32px;
  font-weight: 600;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  margin-bottom: 16px;
}

.mode-selector {
  display: flex;
  justify-content: center;
  gap: 8px;
  flex-wrap: wrap;
}

.mode-btn {
  padding: 8px 16px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.1);
  color: white;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s;
}

.mode-btn:hover {
  background: rgba(255, 255, 255, 0.2);
}

.mode-btn.active {
  background: #4caf50;
  border-color: #4caf50;
}

.game-container {
  display: flex;
  gap: 24px;
  flex: 1;
  align-items: flex-start;
  justify-content: center;
}

.board-wrapper {
  flex: 1;
  max-width: 600px;
  width: 100%;
}

.sidebar {
  width: 320px;
  flex-shrink: 0;
}

/* 响应式设计 */
@media (max-width: 1100px) {
  .game-container {
    flex-direction: column;
    align-items: center;
  }

  .board-wrapper {
    width: 100%;
    max-width: min(600px, 90vw);
  }

  .sidebar {
    width: 100%;
    max-width: 600px;
  }
}

@media (max-width: 480px) {
  .app-container {
    padding: 8px;
  }

  .app-header h1 {
    font-size: 24px;
  }

  .mode-btn {
    padding: 6px 12px;
    font-size: 12px;
  }

  .board-wrapper {
    width: 95vw;
    max-width: 95vw;
  }

  .sidebar {
    width: 95vw;
  }
}
</style>
