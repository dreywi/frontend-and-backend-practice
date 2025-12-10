<template>
  <div class="palette-generator">
    <div class="controls-section">
      <div class="control-group">
        <button @click="generateRandomPalette" class="btn btn-primary">
          🎲 Случайная палитра
        </button>
        
        <div class="color-count-control">
          <label>Количество цветов:</label>
          <select v-model="colorCount" @change="updatePaletteCount" class="form-select">
            <option value="3">3</option>
            <option value="5">5</option>
            <option value="7">7</option>
          </select>
        </div>
        
        <div class="format-control">
          <label>Формат:</label>
          <div class="format-toggle">
            <button 
              @click="colorFormat = 'hex'" 
              :class="{ active: colorFormat === 'hex' }"
              class="format-btn"
            >
              HEX
            </button>
            <button 
              @click="colorFormat = 'rgb'" 
              :class="{ active: colorFormat === 'rgb' }"
              class="format-btn"
            >
              RGB
            </button>
          </div>
        </div>
      </div>
    </div>
    
    <div class="palette-display">
      <div class="palette-container">
        <div 
          v-for="(color, index) in palette" 
          :key="index"
          class="color-card"
          :style="{ backgroundColor: color.hex }"
          @click="copyColor(color)"
        >
          <div class="color-card-content" :class="{ 'dark-text': isLightColor(color) }">
            <button 
              @click.stop="toggleLock(index)"
              class="lock-btn"
              :title="color.locked ? 'Разблокировать' : 'Заблокировать'"
            >
              {{ color.locked ? '🔒' : '🔓' }}
            </button>
            
            <div class="color-value">
              {{ getFormattedColor(color) }}
            </div>
            
            <div class="color-index">
              Цвет {{ index + 1 }}
            </div>
          </div>
        </div>
      </div>
      
      <div class="palette-info">
        <p>Кликните на цвет, чтобы скопировать {{ colorFormat === 'hex' ? 'HEX' : 'RGB' }} код</p>
        <p>🔒 - цвет заблокирован и не изменится при генерации новой палитры</p>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  name: 'PaletteGenerator',
  emits: ['color-copied', 'palette-updated'],
  setup(props, { emit }) {
    const colorCount = ref(5)
    const colorFormat = ref('hex')
    const palette = ref([])

    // Инициализация палитры
    onMounted(() => {
      if (palette.value.length === 0) {
        generateRandomPalette()
      }
    })

    // Генерация случайного цвета
    const generateRandomColor = () => {
      const hue = Math.floor(Math.random() * 360)
      const saturation = Math.floor(Math.random() * 30) + 70
      const lightness = Math.floor(Math.random() * 30) + 35
      
      return {
        hex: hslToHex(hue, saturation, lightness),
        rgb: hslToRgb(hue, saturation, lightness),
        hsl: { h: hue, s: saturation, l: lightness },
        locked: false
      }
    }

    // Генерация гармоничной палитры
    const generateRandomPalette = () => {
      const baseHue = Math.floor(Math.random() * 360)
      const newPalette = []
      
      for (let i = 0; i < colorCount.value; i++) {
        if (palette.value[i]?.locked) {
          newPalette.push(palette.value[i])
        } else {
          const hue = (baseHue + i * 60) % 360
          const color = generateRandomColor()
          color.hex = hslToHex(hue, color.hsl.s, color.hsl.l)
          color.rgb = hslToRgb(hue, color.hsl.s, color.hsl.l)
          color.hsl.h = hue
          newPalette.push(color)
        }
      }
      
      palette.value = newPalette
      emit('palette-updated', palette.value)
    }

    // Обновление количества цветов
    const updatePaletteCount = () => {
      if (palette.value.length > colorCount.value) {
        palette.value = palette.value.slice(0, colorCount.value)
      } else {
        while (palette.value.length < colorCount.value) {
          palette.value.push(generateRandomColor())
        }
      }
      emit('palette-updated', palette.value)
    }

    // Копирование цвета
    const copyColor = (color) => {
      const text = colorFormat.value === 'hex' 
        ? color.hex 
        : `rgb(${color.rgb.r}, ${color.rgb.g}, ${color.rgb.b})`
      
      navigator.clipboard.writeText(text).then(() => {
        emit('color-copied')
      })
    }

    // Блокировка/разблокировка цвета
    const toggleLock = (index) => {
      palette.value[index].locked = !palette.value[index].locked
      emit('palette-updated', palette.value)
    }

    // Вспомогательные функции
    const getFormattedColor = (color) => {
      if (colorFormat.value === 'rgb') {
        return `rgb(${color.rgb.r}, ${color.rgb.g}, ${color.rgb.b})`
      }
      return color.hex
    }

    const isLightColor = (color) => {
      const brightness = (color.rgb.r * 299 + color.rgb.g * 587 + color.rgb.b * 114) / 1000
      return brightness > 128
    }

    // Функции для работы с цветами
    const hslToHex = (h, s, l) => {
      const rgb = hslToRgb(h, s, l)
      return `#${((1 << 24) + (rgb.r << 16) + (rgb.g << 8) + rgb.b).toString(16).slice(1).toUpperCase()}`
    }

    const hslToRgb = (h, s, l) => {
      s /= 100
      l /= 100
      
      const c = (1 - Math.abs(2 * l - 1)) * s
      const x = c * (1 - Math.abs((h / 60) % 2 - 1))
      const m = l - c / 2
      
      let r, g, b
      
      if (h >= 0 && h < 60) {
        [r, g, b] = [c, x, 0]
      } else if (h >= 60 && h < 120) {
        [r, g, b] = [x, c, 0]
      } else if (h >= 120 && h < 180) {
        [r, g, b] = [0, c, x]
      } else if (h >= 180 && h < 240) {
        [r, g, b] = [0, x, c]
      } else if (h >= 240 && h < 300) {
        [r, g, b] = [x, 0, c]
      } else {
        [r, g, b] = [c, 0, x]
      }
      
      return {
        r: Math.round((r + m) * 255),
        g: Math.round((g + m) * 255),
        b: Math.round((b + m) * 255)
      }
    }

    return {
      colorCount,
      colorFormat,
      palette,
      generateRandomPalette,
      updatePaletteCount,
      copyColor,
      toggleLock,
      getFormattedColor,
      isLightColor
    }
  }
}
</script>

<style scoped>
.palette-generator {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
}

.controls-section {
  margin-bottom: 2rem;
}

.control-group {
  display: flex;
  flex-wrap: wrap;
  gap: 1.5rem;
  align-items: center;
  margin-bottom: 1rem;
}

.btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

.color-count-control,
.format-control {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.form-select {
  padding: 0.5rem 1rem;
  border-radius: 8px;
  border: 2px solid #ddd;
  background: white;
  font-size: 1rem;
  cursor: pointer;
}

.format-toggle {
  display: flex;
  gap: 0.5rem;
}

.format-btn {
  padding: 0.5rem 1rem;
  border: 2px solid #ddd;
  background: white;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
}

.format-btn.active {
  background: #667eea;
  color: white;
  border-color: #667eea;
}

.palette-display {
  margin-top: 2rem;
}

.palette-container {
  display: flex;
  gap: 1rem;
  margin-bottom: 1.5rem;
  min-height: 250px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 8px 25px rgba(0,0,0,0.1);
}

.color-card {
  flex: 1;
  cursor: pointer;
  transition: transform 0.3s;
  position: relative;
  overflow: hidden;
}

.color-card:hover {
  transform: translateY(-5px);
}

.color-card-content {
  height: 100%;
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.color-card-content.dark-text {
  color: #333;
}

.lock-btn {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  border-radius: 50%;
  width: 36px;
  height: 36px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s;
  backdrop-filter: blur(5px);
  align-self: flex-end;
}

.lock-btn:hover {
  background: rgba(255, 255, 255, 0.3);
}

.color-value {
  font-family: 'Courier New', monospace;
  font-weight: bold;
  font-size: 1.1rem;
  text-align: center;
  padding: 0.5rem;
  background: rgba(0, 0, 0, 0.1);
  border-radius: 6px;
  backdrop-filter: blur(5px);
}

.dark-text .color-value {
  background: rgba(255, 255, 255, 0.2);
}

.color-index {
  font-size: 0.9rem;
  opacity: 0.9;
  text-align: center;
  padding-top: 0.5rem;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
}

.dark-text .color-index {
  border-top-color: rgba(0, 0, 0, 0.2);
}

.palette-info {
  text-align: center;
  color: #666;
  font-size: 0.9rem;
  margin-top: 1rem;
}

.palette-info p {
  margin-bottom: 0.5rem;
}

@media (max-width: 768px) {
  .palette-container {
    flex-direction: column;
    min-height: auto;
  }
  
  .color-card {
    min-height: 120px;
  }
  
  .control-group {
    flex-direction: column;
    align-items: stretch;
  }
}
</style>