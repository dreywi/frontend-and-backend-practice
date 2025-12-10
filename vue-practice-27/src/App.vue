<template>
  <div id="app">
    <header class="app-header">
      <h1>🎨 Генератор цветовых палитр</h1>
      <p>Создавайте гармоничные цветовые схемы для дизайна</p>
    </header>

    <!-- Навигация между примерами -->
    <nav class="navigation">
      <button
        @click="currentDemo = 'palette'"
        :class="{ active: currentDemo === 'palette' }"
        class="nav-button"
      >
        Генератор палитр
      </button>
      <button
        @click="currentDemo = 'manage'"
        :class="{ active: currentDemo === 'manage' }"
        class="nav-button"
      >
        Управление цветами
      </button>
      <button
        @click="currentDemo = 'preview'"
        :class="{ active: currentDemo === 'preview' }"
        class="nav-button"
      >
        Предпросмотр
      </button>
    </nav>

    <!-- Отображаем выбранный компонент -->
    <main class="main-content">
      <!-- Компонент PaletteGenerator -->
      <PaletteGenerator v-if="currentDemo === 'palette'" 
        @color-copied="showCopyNotification"
        @palette-updated="updateCurrentPalette" />
      
      <!-- Компонент ColorManagement -->
      <ColorManagement v-else-if="currentDemo === 'manage'"
        :palette="currentPalette"
        @palette-updated="updateCurrentPalette" />
      
      <!-- Компонент Preview -->
      <Preview v-else-if="currentDemo === 'preview'"
        :palette="currentPalette" />
      
      <!-- Сообщение если ничего не выбрано -->
      <div v-else class="welcome-message">
        <h2>Добро пожаловать в Генератор палитр!</h2>
        <p>Выберите раздел из навигации выше.</p>
      </div>
    </main>

    <footer class="app-footer">
      <p>Vue 3 + Vite • Генератор цветовых палитр</p>
    </footer>

    <!-- Уведомление о копировании -->
    <div v-if="showCopySuccess" class="copy-notification">
      ✓ HEX-код скопирован в буфер обмена!
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import PaletteGenerator from './components/PaletteGenerator.vue'
import ColorManagement from './components/ColorManagement.vue'
import Preview from './components/Preview.vue'

export default {
  name: 'App',
  components: {
    PaletteGenerator,
    ColorManagement,
    Preview
  },
  setup() {
    const currentDemo = ref('palette')
    const currentPalette = ref([])
    const showCopySuccess = ref(false)

    // Загрузка сохраненной палитры при запуске
    onMounted(() => {
      loadPaletteFromStorage()
    })

    // Функция для показа уведомления о копировании
    const showCopyNotification = () => {
      showCopySuccess.value = true
      setTimeout(() => {
        showCopySuccess.value = false
      }, 2000)
    }

    // Обновление текущей палитры
    const updateCurrentPalette = (newPalette) => {
      currentPalette.value = newPalette
      savePaletteToStorage()
    }

    // Сохранение в localStorage
    const savePaletteToStorage = () => {
      if (currentPalette.value.length > 0) {
        localStorage.setItem('colorPalette', JSON.stringify(currentPalette.value))
      }
    }

    // Загрузка из localStorage
    const loadPaletteFromStorage = () => {
      const saved = localStorage.getItem('colorPalette')
      if (saved) {
        currentPalette.value = JSON.parse(saved)
      }
    }

    return {
      currentDemo,
      currentPalette,
      showCopySuccess,
      showCopyNotification,
      updateCurrentPalette
    }
  }
}
</script>

<style>
/* Глобальные стили */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f5f5;
  color: #333;
  line-height: 1.6;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Стили шапки */
.app-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 2rem;
  text-align: center;
}

.app-header h1 {
  margin-bottom: 0.5rem;
  font-size: 2.5rem;
}

.app-header p {
  opacity: 0.9;
  font-size: 1.1rem;
}

/* Навигация */
.navigation {
  display: flex;
  justify-content: center;
  gap: 1rem;
  padding: 1.5rem;
  background-color: white;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  flex-wrap: wrap;
}

.nav-button {
  padding: 0.75rem 1.5rem;
  border: 2px solid #667eea;
  background-color: transparent;
  color: #667eea;
  border-radius: 25px;
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.nav-button:hover {
  background-color: #667eea;
  color: white;
  transform: translateY(-2px);
}

.nav-button.active {
  background-color: #667eea;
  color: white;
}

/* Основное содержимое */
.main-content {
  flex: 1;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

.welcome-message {
  text-align: center;
  padding: 4rem 2rem;
  color: #666;
}

.welcome-message h2 {
  margin-bottom: 1rem;
  color: #333;
}

/* Подвал */
.app-footer {
  background-color: #333;
  color: white;
  text-align: center;
  padding: 1rem;
  margin-top: auto;
}

/* Уведомление о копировании */
.copy-notification {
  position: fixed;
  top: 20px;
  right: 20px;
  background: #4CAF50;
  color: white;
  padding: 1rem 1.5rem;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
  animation: slideIn 0.3s ease, fadeOut 0.3s ease 1.7s forwards;
  z-index: 1000;
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes fadeOut {
  to {
    opacity: 0;
  }
}

/* Адаптивность */
@media (max-width: 768px) {
  .app-header h1 {
    font-size: 2rem;
  }
  
  .navigation {
    flex-direction: column;
    align-items: center;
  }
  
  .nav-button {
    width: 100%;
    max-width: 300px;
  }
  
  .main-content {
    padding: 1rem;
  }
}
</style>