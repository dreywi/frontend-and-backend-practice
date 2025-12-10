<template>
  <div class="color-management">
    <h2>Управление цветами</h2>
    
    <div class="management-controls">
      <button @click="lockAllColors" class="btn btn-secondary">
        🔒 Заблокировать все
      </button>
      <button @click="unlockAllColors" class="btn btn-secondary">
        🔓 Разблокировать все
      </button>
    </div>
    
    <div class="colors-list">
      <div v-for="(color, index) in palette" :key="index" class="color-item">
        <div class="color-preview" :style="{ backgroundColor: color.hex }"></div>
        
        <div class="color-details">
          <div class="color-values">
            <div class="color-hex">
              <strong>HEX:</strong> {{ color.hex }}
            </div>
            <div class="color-rgb">
              <strong>RGB:</strong> rgb({{ color.rgb.r }}, {{ color.rgb.g }}, {{ color.rgb.b }})
            </div>
          </div>
          
          <div class="color-actions">
            <button 
              @click="toggleLock(index)"
              class="btn-action"
              :class="{ locked: color.locked }"
            >
              {{ color.locked ? '🔒 Разблокировать' : '🔓 Заблокировать' }}
            </button>
            <button @click="copyToClipboard(color.hex)" class="btn-action">
              📋 Копировать HEX
            </button>
          </div>
        </div>
      </div>
    </div>
    
    <div v-if="palette.length === 0" class="empty-state">
      <p>Палитра пуста. Сначала создайте палитру в разделе "Генератор палитр".</p>
    </div>
  </div>
</template>

<script>
import { ref, watch } from 'vue'

export default {
  name: 'ColorManagement',
  props: {
    palette: {
      type: Array,
      required: true
    }
  },
  emits: ['palette-updated'],
  setup(props, { emit }) {
    const localPalette = ref([...props.palette])

    // Следим за изменениями пропса
    watch(() => props.palette, (newPalette) => {
      localPalette.value = [...newPalette]
    }, { deep: true })

    const toggleLock = (index) => {
      localPalette.value[index].locked = !localPalette.value[index].locked
      emit('palette-updated', localPalette.value)
    }

    const lockAllColors = () => {
      localPalette.value.forEach(color => {
        color.locked = true
      })
      emit('palette-updated', localPalette.value)
    }

    const unlockAllColors = () => {
      localPalette.value.forEach(color => {
        color.locked = false
      })
      emit('palette-updated', localPalette.value)
    }

    const copyToClipboard = (text) => {
      navigator.clipboard.writeText(text)
    }

    return {
      localPalette,
      toggleLock,
      lockAllColors,
      unlockAllColors,
      copyToClipboard
    }
  }
}
</script>

<style scoped>
.color-management {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
}

.color-management h2 {
  color: #333;
  margin-bottom: 1.5rem;
  text-align: center;
}

.management-controls {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 2rem;
}

.btn-secondary {
  padding: 0.75rem 1.5rem;
  border: 2px solid #667eea;
  background: transparent;
  color: #667eea;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.3s;
}

.btn-secondary:hover {
  background: #667eea;
  color: white;
}

.colors-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.color-item {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  padding: 1.5rem;
  background: #f8f9fa;
  border-radius: 10px;
  border: 1px solid #e9ecef;
}

.color-preview {
  width: 80px;
  height: 80px;
  border-radius: 8px;
  border: 2px solid #dee2e6;
  flex-shrink: 0;
}

.color-details {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.color-values {
  display: flex;
  gap: 2rem;
  font-family: 'Courier New', monospace;
}

.color-hex,
.color-rgb {
  background: white;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  border: 1px solid #dee2e6;
}

.color-actions {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.btn-action {
  padding: 0.5rem 1rem;
  border: 1px solid #667eea;
  background: transparent;
  color: #667eea;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.3s;
}

.btn-action:hover {
  background: #667eea;
  color: white;
}

.btn-action.locked {
  background: #667eea;
  color: white;
}

.empty-state {
  text-align: center;
  padding: 3rem 1rem;
  color: #666;
}

.empty-state p {
  font-size: 1.1rem;
}

@media (max-width: 768px) {
  .color-item {
    flex-direction: column;
    text-align: center;
  }
  
  .color-values {
    flex-direction: column;
    gap: 1rem;
  }
  
  .color-actions {
    justify-content: center;
  }
  
  .management-controls {
    flex-direction: column;
  }
}
</style>