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
    <div v-if="selectedToxins.length > 0" class="detox-information">
      <h3>Your Detox Plan:</h3>
      
      <div v-for="toxinName in selectedToxins" :key="toxinName" class="toxin-info-card">
        <div class="toxin-header">
          <h4>{{ getToxinIcon(toxinName) }} {{ toxinName }} Detoxification</h4>
          <el-tag :type="getEfficiencyTag(getToxinInfo(toxinName).removalEfficiency)" size="small">
            {{ getToxinInfo(toxinName).removalEfficiency }}
          </el-tag>
        </div>

        <div class="info-sections">
          <!-- Evidence Section -->
          <div class="info-section">
            <h5>🔬 Scientific Evidence</h5>
            <p>{{ getToxinInfo(toxinName).detoxEvidence.description }}</p>
            <div class="research-info">
              <small><strong>Research:</strong> {{ getToxinInfo(toxinName).detoxEvidence.research }}</small>
              <a v-if="getToxinInfo(toxinName).detoxEvidence.link" 
                 :href="getToxinInfo(toxinName).detoxEvidence.link" 
                 target="_blank" 
                 rel="noopener noreferrer"
                 class="research-link">
                📄 View Study
              </a>
            </div>
          </div>

          <!-- Natural Elimination -->
          <div class="info-section">
            <h5>⚡ Natural Elimination</h5>
            <p><strong>Pathway:</strong> {{ getToxinInfo(toxinName).naturalElimination.pathway }}</p>
            <p><strong>Timeframe:</strong> {{ getToxinInfo(toxinName).naturalElimination.timeframe }}</p>
          </div>

          <!-- Detox Strategies -->
          <div class="info-section">
            <h5>🛡️ Supported Detox Strategies</h5>
            <div class="strategy-tags">
              <el-tag
                v-for="strategy in getToxinInfo(toxinName).supportedDetoxStrategies"
                :key="strategy"
                type="success"
                size="small"
                class="strategy-tag"
              >
                {{ strategy }}
              </el-tag>
            </div>
          </div>

          <!-- Lifestyle & Diet -->
          <div class="info-section">
            <h5>🥗 Lifestyle & Diet Interventions</h5>
            <div class="intervention-tags">
              <el-tag
                v-for="intervention in getToxinInfo(toxinName).lifestyleDietIntervention"
                :key="intervention"
                type="warning"
                size="small"
                class="intervention-tag"
              >
                {{ intervention }}
              </el-tag>
            </div>
          </div>

          <!-- Experimental Techniques -->
          <div class="info-section">
            <h5>🧪 Experimental Techniques</h5>
            <div class="experimental-tags">
              <el-tag
                v-for="technique in getToxinInfo(toxinName).experimentalTechniques"
                :key="technique"
                type="info"
                size="small"
                class="experimental-tag"
              >
                {{ technique }}
              </el-tag>
            </div>
          </div>
        </div>
      </div>
    </div>

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

export default {
  name: 'MyDetoxPlan',
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
    
    getToxinInfo(toxinName) {
      return this.toxinData.find(toxin => toxin.name === toxinName);
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
    
    getEfficiencyTag(efficiency) {
      if (!efficiency) return 'info';
      
      const lowerEfficiency = efficiency.toLowerCase();
      if (lowerEfficiency.includes('70') || lowerEfficiency.includes('90')) {
        return 'success';
      } else if (lowerEfficiency.includes('50') || lowerEfficiency.includes('60')) {
        return 'warning';
      } else if (lowerEfficiency.includes('not specified') || lowerEfficiency.includes('slow')) {
        return 'danger';
      }
      return 'info';
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

.detox-information {
  margin-bottom: 32px;
  
  h3 {
    color: #27ae60;
    margin-bottom: 20px;
    font-size: 22px;
    text-align: center;
  }
}

.toxin-info-card {
  background: white;
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 24px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  border: 1px solid #e9ecef;
}

.toxin-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 12px;
  border-bottom: 2px solid #f8f9fa;
  
  h4 {
    color: #2c3e50;
    margin: 0;
    font-size: 20px;
    font-weight: 700;
  }
}

.info-sections {
  display: grid;
  gap: 20px;
}

.info-section {
  h5 {
    color: #34495e;
    margin: 0 0 12px 0;
    font-size: 16px;
    font-weight: 600;
  }
  
  p {
    margin: 0 0 8px 0;
    line-height: 1.6;
    color: #5a6c7d;
  }
  
  small {
    color: #7f8c8d;
    font-style: italic;
  }
}

.research-info {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}

.research-link {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 4px 8px;
  background-color: #f8f9fa;
  border: 1px solid #e9ecef;
  border-radius: 6px;
  color: #0066cc;
  text-decoration: none;
  font-size: 12px;
  font-weight: 500;
  transition: all 0.2s ease;
  
  &:hover {
    background-color: #e9ecef;
    color: #004499;
    text-decoration: none;
    transform: translateY(-1px);
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }
  
  &:visited {
    color: #6666aa;
  }
}

.strategy-tags, .intervention-tags, .experimental-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 8px;
}

.strategy-tag, .intervention-tag, .experimental-tag {
  margin: 2px;
  font-weight: 500;
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
  
  .toxin-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }
  
  .action-buttons {
    flex-direction: column;
    align-items: center;
  }
}
</style>
