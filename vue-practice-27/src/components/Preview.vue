<template>
  <div class="preview">
    <h2>Предпросмотр интерфейса</h2>
    
    <div class="preview-controls">
      <button 
        @click="darkMode = !darkMode" 
        class="btn btn-outline"
      >
        {{ darkMode ? '🌙 Тёмный фон' : '☀️ Светлый фон' }}
      </button>
    </div>
    
    <div class="preview-container" :class="{ 'dark-mode': darkMode }">
      <!-- Mockup веб-страницы -->
      <div class="mockup-header">
        <div class="mockup-logo" :style="{ backgroundColor: getColor(0) }"></div>
        <div class="mockup-nav">
          <span 
            v-for="n in 3" 
            :key="n"
            class="mockup-nav-item"
            :style="{ backgroundColor: getColor(n) }"
          ></span>
        </div>
      </div>
      
      <div class="mockup-hero">
        <div 
          class="mockup-title"
          :style="{ color: getColor(1) }"
        >
          Пример интерфейса
        </div>
        <p class="mockup-subtitle">
          Демонстрация использования вашей палитры
        </p>
        <button 
          class="mockup-button"
          :style="{ 
            backgroundColor: getColor(2),
            color: getContrastColor(getColor(2))
          }"
        >
          Пример кнопки
        </button>
      </div>
      
      <div class="mockup-cards">
        <div 
          v-for="n in 3" 
          :key="n"
          class="mockup-card"
        >
          <div 
            class="mockup-card-header"
            :style="{ backgroundColor: getColor(n + 2) }"
          ></div>
          <div class="mockup-card-content">
            <h4>Карточка {{ n }}</h4>
            <p>Пример контента с использованием вашей палитры</p>
          </div>
        </div>
      </div>
    </div>
    
    <div v-if="palette.length === 0" class="empty-preview">
      <p>Для предпросмотра сначала создайте палитру в разделе "Генератор палитр".</p>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue'

export default {
  name: 'Preview',
  props: {
    palette: {
      type: Array,
      default: () => []
    }
  },
  setup(props) {
    const darkMode = ref(false)

    // Получение цвета по индексу или дефолтного
    const getColor = (index) => {
      if (props.palette[index] && props.palette[index].hex) {
        return props.palette[index].hex
      }
      return getDefaultColor(index)
    }

    // Дефолтные цвета для превью
    const getDefaultColor = (index) => {
      const defaults = ['#667eea', '#764ba2', '#4CAF50', '#FF9800', '#E91E63', '#9C27B0']
      return defaults[index % defaults.length]
    }

    // Определение цвета текста для контраста
    const getContrastColor = (hexColor) => {
      if (!hexColor) return '#ffffff'
      
      // Удаляем # если есть
      const hex = hexColor.replace('#', '')
      
      // Конвертируем в RGB
      const r = parseInt(hex.substr(0, 2), 16)
      const g = parseInt(hex.substr(2, 2), 16)
      const b = parseInt(hex.substr(4, 2), 16)
      
      // Вычисляем яркость
      const brightness = (r * 299 + g * 587 + b * 114) / 1000
      
      return brightness > 128 ? '#333333' : '#ffffff'
    }

    return {
      darkMode,
      getColor,
      getContrastColor
    }
  }
}
</script>

<style scoped>
.preview {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
}

.preview h2 {
  color: #333;
  margin-bottom: 1.5rem;
  text-align: center;
}

.preview-controls {
  display: flex;
  justify-content: center;
  margin-bottom: 2rem;
}

.btn-outline {
  padding: 0.75rem 1.5rem;
  border: 2px solid #667eea;
  background: transparent;
  color: #667eea;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.3s;
}

.btn-outline:hover {
  background: #667eea;
  color: white;
}

.preview-container {
  background: #f8f9fa;
  border-radius: 12px;
  padding: 2rem;
  transition: all 0.3s;
}

.preview-container.dark-mode {
  background: #2c3e50;
  color: white;
}

.mockup-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  background: white;
  border-radius: 8px 8px 0 0;
  margin-bottom: 1px;
}

.dark-mode .mockup-header {
  background: #34495e;
}

.mockup-logo {
  width: 60px;
  height: 30px;
  border-radius: 4px;
}

.mockup-nav {
  display: flex;
  gap: 1rem;
}

.mockup-nav-item {
  width: 40px;
  height: 8px;
  border-radius: 4px;
}

.mockup-hero {
  padding: 3rem 2rem;
  text-align: center;
  background: white;
  margin-bottom: 1px;
}

.dark-mode .mockup-hero {
  background: #34495e;
}

.mockup-title {
  font-size: 2rem;
  font-weight: bold;
  margin-bottom: 1rem;
}

.mockup-subtitle {
  font-size: 1.1rem;
  opacity: 0.8;
  margin-bottom: 2rem;
}

.mockup-button {
  padding: 1rem 2rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
  transition: transform 0.3s;
}

.mockup-button:hover {
  transform: translateY(-2px);
}

.mockup-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1.5rem;
  padding: 2rem;
  background: white;
  border-radius: 0 0 8px 8px;
}

.dark-mode .mockup-cards {
  background: #34495e;
}

.mockup-card {
  background: white;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s;
}

.dark-mode .mockup-card {
  background: #4a6572;
}

.mockup-card:hover {
  transform: translateY(-5px);
}

.mockup-card-header {
  height: 100px;
}

.mockup-card-content {
  padding: 1.5rem;
}

.mockup-card-content h4 {
  margin-bottom: 0.5rem;
  font-size: 1.2rem;
}

.mockup-card-content p {
  font-size: 0.9rem;
  opacity: 0.8;
}

.empty-preview {
  text-align: center;
  padding: 3rem 1rem;
  color: #666;
}

.empty-preview p {
  font-size: 1.1rem;
}

@media (max-width: 768px) {
  .mockup-cards {
    grid-template-columns: 1fr;
  }
  
  .mockup-header {
    flex-direction: column;
    gap: 1rem;
  }
}
</style>