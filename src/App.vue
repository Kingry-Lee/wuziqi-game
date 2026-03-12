<script setup>
import { ref, onMounted } from 'vue'
import GameBoard from './components/GameBoard.vue'
import ControlPanel from './components/ControlPanel.vue'
import SettingsPanel from './components/SettingsPanel.vue'
import HistoryPanel from './components/HistoryPanel.vue'
import { useGame } from './composables/useGame'

const { initGame } = useGame()

const showSettings = ref(false)
const showHistory = ref(false)

onMounted(() => {
  initGame()
})
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <h1>五子棋</h1>
    </header>
    
    <main class="game-container">
      <div class="board-wrapper">
        <GameBoard />
      </div>
      
      <aside class="sidebar">
        <ControlPanel 
          @open-settings="showSettings = true"
          @open-history="showHistory = true"
        />
      </aside>
    </main>

    <!-- 设置面板 -->
    <SettingsPanel 
      v-if="showSettings" 
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
  width: 280px;
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

  .board-wrapper {
    width: 95vw;
    max-width: 95vw;
  }

  .sidebar {
    width: 95vw;
  }
}
</style>
