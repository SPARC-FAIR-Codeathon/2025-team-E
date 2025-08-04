<template>
  <div class="detox-information">
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
</template>

<script>
export default {
  name: 'ToxinInfoDisplay',
  props: {
    selectedToxins: {
      type: Array,
      required: true,
      default: () => []
    },
    toxinData: {
      type: Array,
      required: true,
      default: () => []
    }
  },
  methods: {
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
    }
  }
};
</script>

<style scoped lang="scss">
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

// Element Plus overrides
:deep(.el-tag) {
  font-weight: 500;
  border-radius: 6px;
}

// Responsive design
@media (max-width: 768px) {
  .toxin-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }
}
</style>
