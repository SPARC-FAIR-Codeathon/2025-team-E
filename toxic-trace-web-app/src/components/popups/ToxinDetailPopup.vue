<template>
  <el-dialog 
    v-model="visible" 
    :title="toxin.name"
    width="80%"
    max-width="900px"
    :before-close="handleClose"
    class="toxin-detail-popup"
  >
    <div class="popup-content">
      <!-- Header Section with Key Info -->
      <div class="header-section">
        <div class="toxin-overview">
          <h2>{{ toxin.name }}</h2>
          <div class="overview-badges">
            <el-tag 
              :type="getSeverityType(toxin.filters.severity)"
              size="large"
            >
              {{ capitalizeFirst(toxin.filters.severity) }} Severity
            </el-tag>
            <el-tag 
              :type="getDifficultyType(toxin.filters.detoxDifficulty)"
              size="large"
            >
              {{ capitalizeFirst(toxin.filters.detoxDifficulty) }} Detox
            </el-tag>
            <el-tag 
              type="info"
              size="large"
            >
              {{ toxin.filters.eliminationSpeed }} Elimination
            </el-tag>
          </div>
        </div>
        
        <div class="quick-stats">
          <div class="stat-item">
            <span class="stat-label">Time to Eliminate</span>
            <span class="stat-value">{{ toxin.filters.timeToEliminate }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">Evidence Level</span>
            <span class="stat-value">{{ toxin.filters.evidenceLevel }}</span>
          </div>
        </div>
      </div>

      <!-- Description Section -->
      <div class="section">
        <h3><el-icon><Document /></el-icon> Description</h3>
        <p class="description-text">{{ toxin.detoxEvidence.description }}</p>
      </div>

      <!-- Affected Systems -->
      <div class="section">
        <h3><el-icon><User /></el-icon> Affected Systems</h3>
        <div class="system-grid">
          <div 
            v-for="system in toxin.filters.affectedSystems" 
            :key="system"
            class="system-card"
          >
            <el-icon class="system-icon"><Monitor /></el-icon>
            <span>{{ capitalizeFirst(system) }}</span>
          </div>
        </div>
      </div>

      <!-- Primary Sources -->
      <div class="section">
        <h3><el-icon><Location /></el-icon> Common Sources</h3>
        <div class="sources-list">
          <el-tag 
            v-for="source in toxin.filters.primarySource" 
            :key="source"
            type="warning"
            size="large"
            class="source-tag"
          >
            {{ source }}
          </el-tag>
        </div>
      </div>

      <!-- Detoxification Methods -->
      <div class="section">
        <h3><el-icon><Tools /></el-icon> Detoxification Methods</h3>
        <div class="methods-grid">
          <div 
            v-for="method in toxin.filters.detoxMethods" 
            :key="method"
            class="method-card"
          >
            <el-icon class="method-icon"><CircleCheck /></el-icon>
            <div class="method-content">
              <h4>{{ capitalizeFirst(method) }}</h4>
              <p>Recommended detoxification approach</p>
            </div>
          </div>
        </div>
      </div>

      <!-- Evidence & Research -->
      <div class="section" v-if="toxin.detoxEvidence">
        <h3><el-icon><Reading /></el-icon> Evidence & Research</h3>
        <div class="evidence-card">
          <div class="evidence-header">
            <el-tag 
              :type="getEvidenceTypeForLevel(toxin.filters.evidenceLevel)"
              size="large"
            >
              {{ toxin.filters.evidenceLevel }} Evidence
            </el-tag>
          </div>
          <div class="evidence-content">
            <p><strong>Research Status:</strong> {{ toxin.detoxEvidence.description }}</p>
            <div v-if="toxin.detoxEvidence.link" class="research-link">
              <el-button 
                type="primary" 
                :icon="Link"
                @click="openResearch(toxin.detoxEvidence.link)"
              >
                View Research Paper
              </el-button>
            </div>
          </div>
        </div>
      </div>

      <!-- Additional Properties (if any exist) -->
      <div class="section" v-if="hasAdditionalProperties">
        <h3><el-icon><InfoFilled /></el-icon> Additional Information</h3>
        <div class="additional-info">
          <div v-for="(value, key) in additionalProperties" :key="key" class="info-row">
            <span class="info-key">{{ formatKey(key) }}:</span>
            <span class="info-value">{{ formatValue(value) }}</span>
          </div>
        </div>
      </div>
    </div>

    <template #footer>
      <div class="popup-footer">
        <el-button @click="handleClose">Close</el-button>
        <el-button 
          v-if="toxin.detoxEvidence.link"
          type="primary" 
          :icon="Link"
          @click="openResearch(toxin.detoxEvidence.link)"
        >
          View Research
        </el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script>
import { computed } from 'vue'
import { ElDialog, ElButton, ElTag, ElIcon } from 'element-plus'
import { 
  Document, User, Location, Tools, Reading, InfoFilled, Link,
  Monitor, CircleCheck
} from '@element-plus/icons-vue'

export default {
  name: 'ToxinDetailPopup',
  components: {
    ElDialog,
    ElButton,
    ElTag,
    ElIcon,
    Document,
    User,
    Location,
    Tools,
    Reading,
    InfoFilled,
    Link,
    Monitor,
    CircleCheck
  },
  props: {
    modelValue: {
      type: Boolean,
      default: false
    },
    toxin: {
      type: Object,
      required: true
    }
  },
  emits: ['update:modelValue', 'open-research'],
  setup(props, { emit }) {
    const visible = computed({
      get: () => props.modelValue,
      set: (value) => emit('update:modelValue', value)
    })

    const additionalProperties = computed(() => {
      const excluded = ['name', 'filters', 'detoxEvidence']
      const additional = {}
      
      Object.keys(props.toxin).forEach(key => {
        if (!excluded.includes(key)) {
          additional[key] = props.toxin[key]
        }
      })
      
      return additional
    })

    const hasAdditionalProperties = computed(() => {
      return Object.keys(additionalProperties.value).length > 0
    })

    const handleClose = () => {
      visible.value = false
    }

    const openResearch = (url) => {
      emit('open-research', url)
    }

    const getSeverityType = (severity) => {
      switch (severity?.toLowerCase()) {
        case 'high': return 'danger'
        case 'moderate': return 'warning'
        case 'low': return 'success'
        default: return 'info'
      }
    }

    const getDifficultyType = (difficulty) => {
      switch (difficulty?.toLowerCase()) {
        case 'hard': return 'danger'
        case 'moderate': return 'warning'
        case 'easy': return 'success'
        default: return 'info'
      }
    }

    const getEvidenceTypeForLevel = (level) => {
      switch (level?.toLowerCase()) {
        case 'strong': return 'success'
        case 'moderate': return 'warning'
        case 'limited': return 'info'
        case 'emerging': return 'info'
        default: return 'info'
      }
    }

    const capitalizeFirst = (str) => {
      if (!str) return ''
      return str.charAt(0).toUpperCase() + str.slice(1)
    }

    const formatKey = (key) => {
      return key.replace(/([A-Z])/g, ' $1').replace(/^./, str => str.toUpperCase())
    }

    const formatValue = (value) => {
      if (Array.isArray(value)) {
        return value.join(', ')
      }
      if (typeof value === 'object') {
        return JSON.stringify(value, null, 2)
      }
      return String(value)
    }

    return {
      visible,
      additionalProperties,
      hasAdditionalProperties,
      handleClose,
      openResearch,
      getSeverityType,
      getDifficultyType,
      getEvidenceTypeForLevel,
      capitalizeFirst,
      formatKey,
      formatValue
    }
  }
}
</script>

<style scoped lang="scss">
.toxin-detail-popup {
  .popup-content {
    max-height: 70vh;
    overflow-y: auto;
    
    .header-section {
      background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
      border-radius: 12px;
      padding: 24px;
      margin-bottom: 24px;
      
      .toxin-overview {
        margin-bottom: 16px;
        
        h2 {
          margin: 0 0 12px 0;
          color: #303133;
          font-size: 28px;
          font-weight: 700;
        }
        
        .overview-badges {
          display: flex;
          gap: 8px;
          flex-wrap: wrap;
        }
      }
      
      .quick-stats {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 16px;
        
        .stat-item {
          background: white;
          padding: 12px 16px;
          border-radius: 8px;
          box-shadow: 0 2px 4px rgba(0,0,0,0.05);
          
          .stat-label {
            display: block;
            font-size: 12px;
            color: #909399;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            margin-bottom: 4px;
          }
          
          .stat-value {
            font-size: 16px;
            font-weight: 600;
            color: #303133;
          }
        }
      }
    }
    
    .section {
      margin-bottom: 32px;
      
      h3 {
        display: flex;
        align-items: center;
        gap: 8px;
        margin: 0 0 16px 0;
        color: #303133;
        font-size: 18px;
        font-weight: 600;
        padding-bottom: 8px;
        border-bottom: 2px solid #e4e7ed;
      }
      
      .description-text {
        font-size: 14px;
        line-height: 1.6;
        color: #606266;
        background: #f8f9fc;
        padding: 16px;
        border-radius: 8px;
        border-left: 4px solid #409eff;
      }
      
      .system-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
        gap: 12px;
        
        .system-card {
          display: flex;
          align-items: center;
          gap: 8px;
          padding: 12px 16px;
          background: linear-gradient(135deg, #fff1f0 0%, #fff7e6 100%);
          border: 1px solid #ffd591;
          border-radius: 8px;
          transition: all 0.3s ease;
          
          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(255, 213, 145, 0.3);
          }
          
          .system-icon {
            color: #fa8c16;
            font-size: 18px;
          }
          
          span {
            font-weight: 500;
            color: #d4380d;
          }
        }
      }
      
      .sources-list {
        display: flex;
        flex-wrap: wrap;
        gap: 8px;
        
        .source-tag {
          font-size: 13px;
        }
      }
      
      .methods-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
        gap: 16px;
        
        .method-card {
          display: flex;
          align-items: flex-start;
          gap: 12px;
          padding: 16px;
          background: linear-gradient(135deg, #f6ffed 0%, #f0f9ff 100%);
          border: 1px solid #95de64;
          border-radius: 12px;
          transition: all 0.3s ease;
          
          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(149, 222, 100, 0.2);
          }
          
          .method-icon {
            color: #52c41a;
            font-size: 24px;
            margin-top: 2px;
          }
          
          .method-content {
            flex: 1;
            
            h4 {
              margin: 0 0 4px 0;
              color: #389e0d;
              font-size: 16px;
              font-weight: 600;
            }
            
            p {
              margin: 0;
              font-size: 13px;
              color: #73d13d;
            }
          }
        }
      }
      
      .evidence-card {
        background: linear-gradient(135deg, #f9f0ff 0%, #f0f8ff 100%);
        border: 1px solid #d3adf7;
        border-radius: 12px;
        padding: 20px;
        
        .evidence-header {
          margin-bottom: 16px;
        }
        
        .evidence-content {
          p {
            margin: 0 0 12px 0;
            color: #531dab;
            font-size: 14px;
            line-height: 1.5;
          }
          
          .research-link {
            margin-top: 16px;
          }
        }
      }
      
      .additional-info {
        background: #fafafa;
        border: 1px solid #d9d9d9;
        border-radius: 8px;
        padding: 16px;
        
        .info-row {
          display: flex;
          margin-bottom: 8px;
          
          &:last-child {
            margin-bottom: 0;
          }
          
          .info-key {
            font-weight: 600;
            color: #303133;
            min-width: 150px;
            margin-right: 12px;
          }
          
          .info-value {
            color: #606266;
            flex: 1;
          }
        }
      }
    }
  }
  
  .popup-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
}

// Responsive design
@media (max-width: 768px) {
  .toxin-detail-popup {
    .popup-content {
      .header-section .quick-stats {
        grid-template-columns: 1fr;
      }
      
      .system-grid {
        grid-template-columns: 1fr;
      }
      
      .methods-grid {
        grid-template-columns: 1fr;
      }
    }
  }
}
</style>
