<template>
  <el-dialog 
    v-model="visible" 
    :title="`${interaction.targetProtein} - Molecular Interaction`"
    width="85%"
    max-width="1000px"
    :before-close="handleClose"
    class="interaction-detail-popup"
  >
    <div class="popup-content">
      <!-- Header Section -->
      <div class="header-section">
        <div class="interaction-overview">
          <h2>{{ interaction.targetProtein }}</h2>
          <div class="overview-badges">
            <el-tag 
              :type="getEvidenceType(interaction.evidence.level)"
              size="large"
            >
              {{ interaction.evidence.level }} Evidence
            </el-tag>
            <el-tag 
              v-if="interaction.evidence.hasStructuralData"
              type="success"
              size="large"
            >
              3D Structure Available
            </el-tag>
            <el-tag 
              type="info"
              size="large"
            >
              {{ interaction.system }}
            </el-tag>
          </div>
        </div>
        
        <div class="quick-stats">
          <div class="stat-item">
            <span class="stat-label">Target System</span>
            <span class="stat-value">{{ interaction.system }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">Toxin</span>
            <span class="stat-value">{{ interaction.toxin }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">Mechanism</span>
            <span class="stat-value">{{ interaction.interaction.mechanism }}</span>
          </div>
        </div>
      </div>

      <!-- Interaction Description -->
      <div class="section">
        <h3><el-icon><Document /></el-icon> Interaction Description</h3>
        <p class="description-text">{{ interaction.interaction.description }}</p>
      </div>

      <!-- 3D Structure Information -->
      <div class="section" v-if="interaction.structures">
        <h3><el-icon><View /></el-icon> 3D Structure Information</h3>
        <div class="structure-cards">
          <!-- Natural Ligand Structure -->
          <div class="structure-card natural">
            <div class="structure-header">
              <el-icon class="structure-icon"><Cpu /></el-icon>
              <h4>Natural Ligand</h4>
              <el-tag type="success" size="small">{{ interaction.structures.naturalLigand.potency }}</el-tag>
            </div>
            <div class="structure-content">
              <div class="pdb-info">
                <span class="pdb-label">PDB ID:</span>
                <code class="pdb-id">{{ interaction.structures.naturalLigand.pdbId }}</code>
              </div>
              <p class="structure-description">{{ interaction.structures.naturalLigand.description || 'Natural ligand structure data' }}</p>
              <el-button 
                type="primary" 
                size="small"
                :icon="Link"
                @click="viewStructure(interaction.structures.naturalLigand.pdbId)"
              >
                View 3D Structure
              </el-button>
            </div>
          </div>

          <!-- Toxin Complex Structure -->
          <div class="structure-card toxin" v-if="interaction.structures.toxinComplex.pdbId">
            <div class="structure-header">
              <el-icon class="structure-icon"><Warning /></el-icon>
              <h4>Toxin Complex</h4>
              <el-tag type="danger" size="small">{{ interaction.structures.toxinComplex.potency }}</el-tag>
            </div>
            <div class="structure-content">
              <div class="pdb-info">
                <span class="pdb-label">PDB ID:</span>
                <code class="pdb-id">{{ interaction.structures.toxinComplex.pdbId }}</code>
              </div>
              <p class="structure-description">{{ interaction.structures.toxinComplex.description || 'Toxin-protein complex structure' }}</p>
              <el-button 
                type="danger" 
                size="small"
                :icon="Link"
                @click="viewStructure(interaction.structures.toxinComplex.pdbId)"
              >
                View 3D Structure
              </el-button>
            </div>
          </div>
        </div>
      </div>

      <!-- Evidence Details -->
      <div class="section">
        <h3><el-icon><Reading /></el-icon> Evidence & Validation</h3>
        <div class="evidence-grid">
          <div class="evidence-card">
            <div class="evidence-header">
              <el-icon><DocumentChecked /></el-icon>
              <h4>Evidence Level</h4>
            </div>
            <div class="evidence-content">
              <el-tag 
                :type="getEvidenceType(interaction.evidence.level)"
                size="large"
              >
                {{ interaction.evidence.level }}
              </el-tag>
              <p>{{ getEvidenceDescription(interaction.evidence.level) }}</p>
            </div>
          </div>
          
          <div class="evidence-card" v-if="interaction.evidence.hasStructuralData">
            <div class="evidence-header">
              <el-icon><View /></el-icon>
              <h4>Structural Data</h4>
            </div>
            <div class="evidence-content">
              <el-tag type="success" size="large">Available</el-tag>
              <p>3D structural data confirms molecular interactions</p>
            </div>
          </div>
        </div>
      </div>

      <!-- Research References -->
      <div class="section" v-if="interaction.references && interaction.references.length > 0">
        <h3><el-icon><Collection /></el-icon> Research References</h3>
        <div class="references-list">
          <div 
            v-for="(reference, index) in interaction.references" 
            :key="index"
            class="reference-card"
          >
            <div class="reference-content">
              <h4>{{ reference.title || `Reference ${index + 1}` }}</h4>
              <p v-if="reference.authors">{{ reference.authors }}</p>
              <p v-if="reference.journal" class="journal-info">{{ reference.journal }}</p>
              <div class="reference-actions">
                <el-button 
                  type="primary" 
                  size="small"
                  :icon="Link"
                  @click="openResearch(reference.url)"
                >
                  View Paper
                </el-button>
                <el-button 
                  v-if="reference.doi"
                  type="text" 
                  size="small"
                  @click="openDOI(reference.doi)"
                >
                  DOI: {{ reference.doi }}
                </el-button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Additional Properties -->
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
          v-if="interaction.structures"
          type="primary" 
          :icon="View"
          @click="viewAllStructures"
        >
          Compare Structures
        </el-button>
        <el-button 
          v-if="interaction.references && interaction.references.length > 0"
          type="success" 
          :icon="Link"
          @click="openResearch(interaction.references[0].url)"
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
  Document, View, Reading, Collection, InfoFilled, Link,
  Cpu, Warning, DocumentChecked
} from '@element-plus/icons-vue'

export default {
  name: 'InteractionDetailPopup',
  components: {
    ElDialog,
    ElButton,
    ElTag,
    ElIcon,
    Document,
    View,
    Reading,
    Collection,
    InfoFilled,
    Link,
    Cpu,
    Warning,
    DocumentChecked
  },
  props: {
    modelValue: {
      type: Boolean,
      default: false
    },
    interaction: {
      type: Object,
      required: true
    }
  },
  emits: ['update:modelValue', 'view-structure', 'open-research'],
  setup(props, { emit }) {
    const visible = computed({
      get: () => props.modelValue,
      set: (value) => emit('update:modelValue', value)
    })

    const additionalProperties = computed(() => {
      const excluded = ['targetProtein', 'system', 'toxin', 'interaction', 'structures', 'evidence', 'references']
      const additional = {}
      
      Object.keys(props.interaction).forEach(key => {
        if (!excluded.includes(key)) {
          additional[key] = props.interaction[key]
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

    const viewStructure = (pdbId) => {
      emit('view-structure', pdbId)
    }

    const viewAllStructures = () => {
      if (props.interaction.structures) {
        const structures = [
          props.interaction.structures.naturalLigand.pdbId,
          props.interaction.structures.toxinComplex.pdbId
        ].filter(Boolean)
        
        emit('view-structure', structures)
      }
    }

    const openResearch = (url) => {
      emit('open-research', url)
    }

    const openDOI = (doi) => {
      const doiUrl = `https://doi.org/${doi}`
      emit('open-research', doiUrl)
    }

    const getEvidenceType = (level) => {
      switch (level?.toLowerCase()) {
        case 'strong': return 'success'
        case 'moderate': return 'warning'
        case 'emerging': return 'info'
        case 'limited': return 'info'
        default: return 'info'
      }
    }

    const getEvidenceDescription = (level) => {
      switch (level?.toLowerCase()) {
        case 'strong': return 'Well-documented with multiple independent studies'
        case 'moderate': return 'Supported by several studies with consistent results'
        case 'emerging': return 'Initial evidence with promising results'
        case 'limited': return 'Limited but relevant evidence available'
        default: return 'Evidence level not specified'
      }
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
      viewStructure,
      viewAllStructures,
      openResearch,
      openDOI,
      getEvidenceType,
      getEvidenceDescription,
      formatKey,
      formatValue
    }
  }
}
</script>

<style scoped lang="scss">
.interaction-detail-popup {
  .popup-content {
    max-height: 75vh;
    overflow-y: auto;
    
    .header-section {
      background: linear-gradient(135deg, #e6f4ff 0%, #f0f8ff 100%);
      border-radius: 12px;
      padding: 24px;
      margin-bottom: 24px;
      
      .interaction-overview {
        margin-bottom: 16px;
        
        h2 {
          margin: 0 0 12px 0;
          color: #1c4e80;
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
          border-left: 4px solid #409eff;
          
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
      
      .structure-cards {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
        gap: 20px;
        
        .structure-card {
          border-radius: 12px;
          padding: 20px;
          transition: all 0.3s ease;
          
          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 25px rgba(0,0,0,0.1);
          }
          
          &.natural {
            background: linear-gradient(135deg, #f6ffed 0%, #e6f7ff 100%);
            border: 2px solid #95de64;
          }
          
          &.toxin {
            background: linear-gradient(135deg, #fff1f0 0%, #ffebe6 100%);
            border: 2px solid #ff7875;
          }
          
          .structure-header {
            display: flex;
            align-items: center;
            gap: 12px;
            margin-bottom: 16px;
            
            .structure-icon {
              font-size: 24px;
            }
            
            .natural & .structure-icon {
              color: #52c41a;
            }
            
            .toxin & .structure-icon {
              color: #ff4d4f;
            }
            
            h4 {
              margin: 0;
              flex: 1;
              font-size: 18px;
              font-weight: 600;
            }
          }
          
          .structure-content {
            .pdb-info {
              display: flex;
              align-items: center;
              gap: 8px;
              margin-bottom: 12px;
              
              .pdb-label {
                font-weight: 500;
                color: #595959;
              }
              
              .pdb-id {
                background: #f0f2f5;
                padding: 4px 8px;
                border-radius: 4px;
                font-family: 'Courier New', monospace;
                font-weight: 600;
                color: #1890ff;
              }
            }
            
            .structure-description {
              color: #8c8c8c;
              font-size: 13px;
              margin-bottom: 16px;
              line-height: 1.4;
            }
          }
        }
      }
      
      .evidence-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
        gap: 16px;
        
        .evidence-card {
          background: linear-gradient(135deg, #f9f0ff 0%, #f0f8ff 100%);
          border: 1px solid #d3adf7;
          border-radius: 12px;
          padding: 20px;
          
          .evidence-header {
            display: flex;
            align-items: center;
            gap: 8px;
            margin-bottom: 12px;
            
            h4 {
              margin: 0;
              color: #531dab;
              font-size: 16px;
              font-weight: 600;
            }
          }
          
          .evidence-content {
            p {
              margin: 8px 0 0 0;
              color: #722ed1;
              font-size: 13px;
              line-height: 1.4;
            }
          }
        }
      }
      
      .references-list {
        display: flex;
        flex-direction: column;
        gap: 16px;
        
        .reference-card {
          background: #fafafa;
          border: 1px solid #d9d9d9;
          border-radius: 8px;
          padding: 16px;
          transition: all 0.3s ease;
          
          &:hover {
            border-color: #409eff;
            box-shadow: 0 2px 8px rgba(64, 158, 255, 0.1);
          }
          
          .reference-content {
            h4 {
              margin: 0 0 8px 0;
              color: #303133;
              font-size: 16px;
              font-weight: 600;
            }
            
            p {
              margin: 0 0 4px 0;
              color: #606266;
              font-size: 13px;
              
              &.journal-info {
                font-style: italic;
                color: #909399;
              }
            }
            
            .reference-actions {
              margin-top: 12px;
              display: flex;
              gap: 8px;
              flex-wrap: wrap;
            }
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
    flex-wrap: wrap;
    gap: 8px;
  }
}

// Responsive design
@media (max-width: 768px) {
  .interaction-detail-popup {
    .popup-content {
      .header-section .quick-stats {
        grid-template-columns: 1fr;
      }
      
      .structure-cards {
        grid-template-columns: 1fr;
      }
      
      .evidence-grid {
        grid-template-columns: 1fr;
      }
    }
    
    .popup-footer {
      flex-direction: column;
      align-items: stretch;
      
      .el-button {
        width: 100%;
      }
    }
  }
}
</style>
