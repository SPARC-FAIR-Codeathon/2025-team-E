<template>
  <div class="detox-plan-container">
    <div class="header">
      <h2>🌿 My Detox Plan</h2>
      <p class="subtitle">Select toxins to create your personalized detoxification strategy</p>
    </div>

    <!-- Toxin Selection -->
    <div class="toxin-selection">
      <h3>Select Toxins of Interest:</h3>
      <div class="toxin-toggles">
        <el-button
          v-for="toxin in toxinData"
          :key="toxin.name"
          :type="selectedToxins.includes(toxin.name) ? 'primary' : 'default'"
          :class="{ 'toxin-selected': selectedToxins.includes(toxin.name) }"
          @click="toggleToxin(toxin.name)"
          class="toxin-toggle"
        >
          <span class="toxin-icon">{{ getToxinIcon(toxin.name) }}</span>
          {{ toxin.name }}
        </el-button>
      </div>
    </div>

    <!-- Selected Toxins Information -->
    <ToxinInfoDisplay 
      v-if="selectedToxins.length > 0"
      :selectedToxins="selectedToxins"
      :toxinData="toxinData"
    />

    <!-- No Selection Message -->
    <div v-if="selectedToxins.length === 0" class="no-selection">
      <div class="no-selection-icon">🌱</div>
      <h3>Select toxins above to view your personalized detox plan</h3>
      <p>Choose from BPA, Phthalates, PFAS, or Microplastics to get started</p>
    </div>

    <!-- Action Buttons -->
    <div class="action-buttons">
      <el-button 
        v-if="selectedToxins.length > 0"
        type="default" 
        @click="clearSelection"
      >
        🗑️ Clear Selection
      </el-button>
    </div>

    <!-- Metadata -->
    <div v-if="selectedToxins.length > 0" class="metadata">
      <p><small>Data last updated: {{ toxinMetadata.lastUpdated }}</small></p>
      <p><small>Source: {{ toxinMetadata.dataSource }}</small></p>
      <p><small>{{ toxinMetadata.note }}</small></p>
    </div>
  </div>
</template>

<script>

import detoxificationData from './data/detoxification.json';
import ToxinInfoDisplay from './ToxinInfoDisplay.vue';

export default {
  name: 'MyDetoxPlan',
  components: {
    ToxinInfoDisplay
  },
  data() {
    return {
      selectedToxins: [],
      toxinData: detoxificationData.toxins,
      toxinMetadata: detoxificationData.metadata
    };
  },
  computed: {
    // No computed properties needed for save/load
  },
  methods: {
    toggleToxin(toxinName) {
      const index = this.selectedToxins.indexOf(toxinName);
      if (index > -1) {
        this.selectedToxins.splice(index, 1);
      } else {
        this.selectedToxins.push(toxinName);
      }
      
      // Emit event for parent component
      this.$emit('toxins-selected', this.selectedToxins);
    },
    
    getToxinIcon(toxinName) {
      const icons = {
        'BPA': '🧴',
        'Phthalates': '🛢️',
        'PFAS': '⚗️',
        'Microplastics': '🔬'
      };
      return icons[toxinName] || '💧';
    },
    
    clearSelection() {
      this.selectedToxins = [];
      this.$emit('toxins-selected', []);
    }
  },
  
  mounted() {
    // Component ready - no auto-loading needed
  }
};
</script>

<style scoped lang="scss">
.detox-plan-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 24px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.header {
  text-align: center;
  margin-bottom: 32px;
  
  h2 {
    color: #2c3e50;
    margin: 0 0 8px 0;
    font-size: 28px;
    font-weight: 700;
  }
  
  .subtitle {
    color: #7f8c8d;
    font-size: 16px;
    margin: 0;
  }
}

.toxin-selection {
  margin-bottom: 32px;
  
  h3 {
    color: #34495e;
    margin-bottom: 16px;
    font-size: 20px;
  }
}

.toxin-toggles {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  justify-content: center;
}

.toxin-toggle {
  padding: 12px 24px;
  border-radius: 25px;
  font-weight: 600;
  transition: all 0.3s ease;
  border: 2px solid #e9ecef;
  
  .toxin-icon {
    margin-right: 8px;
    font-size: 18px;
  }
  
  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  }
  
  &.toxin-selected {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border-color: #667eea;
    color: white;
  }
}

.no-selection {
  text-align: center;
  padding: 40px 20px;
  color: #7f8c8d;
  
  .no-selection-icon {
    font-size: 64px;
    margin-bottom: 16px;
  }
  
  h3 {
    color: #95a5a6;
    margin-bottom: 8px;
  }
  
  p {
    color: #bdc3c7;
  }
}

.action-buttons {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin: 32px 0;
  flex-wrap: wrap;
}

.metadata {
  text-align: center;
  margin-top: 32px;
  padding-top: 20px;
  border-top: 1px solid #e9ecef;
  
  p {
    margin: 4px 0;
    color: #95a5a6;
  }
}

// Element Plus overrides
:deep(.el-button) {
  font-weight: 600;
}

:deep(.el-tag) {
  font-weight: 500;
  border-radius: 6px;
}

// Responsive design
@media (max-width: 768px) {
  .detox-plan-container {
    padding: 16px;
  }
  
  .toxin-toggles {
    justify-content: center;
  }
  
  .toxin-toggle {
    padding: 8px 16px;
    font-size: 14px;
  }
  
  .action-buttons {
    flex-direction: column;
    align-items: center;
  }
}
</style>
