<template>
  <el-dialog 
    v-model="visible" 
    :title="`${interaction.nerve} - Nerve Interaction`"
    width="85%"
    max-width="1000px"
    :before-close="handleClose"
    class="nerve-detail-popup"
  >
    <div class="popup-content">
      <!-- Header Section -->
      <div class="header-section">
        <div class="nerve-overview">
          <h2>{{ interaction.nerve }}</h2>
          <div class="overview-badges">
            <el-tag 
              :type="getEvidenceType(interaction.evidence.level)"
              size="large"
            >
              {{ interaction.evidence.level }} Evidence
            </el-tag>
            <el-tag 
              type="warning"
              size="large"
            >
              {{ interaction.nerve.split(' ')[0] }}
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
            <span class="stat-label">Target Protein</span>
            <span class="stat-value">{{ interaction.targetProtein }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">Toxin</span>
            <span class="stat-value">{{ interaction.toxin }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">System</span>
            <span class="stat-value">{{ interaction.system }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">Key Functions</span>
            <span class="stat-value">{{ interaction.keyFunctions }}</span>
          </div>
        </div>
      </div>

      <!-- Interaction Description -->
      <div class="section">
        <h3><el-icon><Document /></el-icon> Interaction Description</h3>
        <p class="description-text">{{ interaction.interaction.description }}</p>
      </div>

      <!-- Nerve Function & Location -->
      <div class="section">
        <h3><el-icon><Connection /></el-icon> Nerve Function & Location</h3>
        <div class="nerve-info-grid">
          <div class="nerve-info-card location">
            <div class="info-header">
              <el-icon class="info-icon"><Location /></el-icon>
              <h4>Nerve Location</h4>
            </div>
            <div class="info-content">
              <p><strong>{{ interaction.nerve }}</strong></p>
              <p class="info-description">{{ getNerveLocationDescription(interaction.nerve) }}</p>
            </div>
          </div>
          
          <div class="nerve-info-card functions">
            <div class="info-header">
              <el-icon class="info-icon"><Tools /></el-icon>
              <h4>Key Functions</h4>
            </div>
            <div class="info-content">
              <p><strong>{{ interaction.keyFunctions }}</strong></p>
              <p class="info-description">{{ getFunctionDescription(interaction.keyFunctions) }}</p>
            </div>
          </div>
        </div>
      </div>

      <!-- Pathways -->
      <div class="section" v-if="interaction.pathways && interaction.pathways.length > 0">
        <h3><el-icon><Share /></el-icon> Neural Pathways</h3>
        <div class="pathways-grid">
          <div 
            v-for="(pathway, index) in interaction.pathways" 
            :key="index"
            class="pathway-card"
          >
            <div class="pathway-number">{{ index + 1 }}</div>
            <div class="pathway-content">
              <p>{{ pathway }}</p>
            </div>
          </div>
        </div>
      </div>

      <!-- Consequences -->
      <div class="section" v-if="interaction.consequences && interaction.consequences.length > 0">
        <h3><el-icon><Warning /></el-icon> Physiological Consequences</h3>
        <div class="consequences-grid">
          <div 
            v-for="(consequence, index) in interaction.consequences" 
            :key="index"
            class="consequence-card"
          >
            <div class="consequence-severity">
              <el-icon><WarningFilled /></el-icon>
            </div>
            <div class="consequence-content">
              <div class="consequence-parts">
                <span class="cause">{{ consequence.split('→')[0]?.trim() }}</span>
                <el-icon class="arrow"><Right /></el-icon>
                <span class="effect">{{ consequence.split('→')[1]?.trim() }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Evidence Details -->
      <div class="section">
        <h3><el-icon><Reading /></el-icon> Evidence & Validation</h3>
        <div class="evidence-detailed">
          <div class="evidence-level-card">
            <div class="evidence-header">
              <el-tag 
                :type="getEvidenceType(interaction.evidence.level)"
                size="large"
              >
                {{ interaction.evidence.level }} Evidence
              </el-tag>
            </div>
            <div class="evidence-description">
              <p>{{ getEvidenceDescription(interaction.evidence.level) }}</p>
              <div v-if="interaction.evidence.notes" class="evidence-notes">
                <strong>Additional Notes:</strong>
                <p>{{ interaction.evidence.notes }}</p>
              </div>
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
              <h4>{{ reference.title || `Research Study ${index + 1}` }}</h4>
              <p v-if="reference.authors" class="authors">{{ reference.authors }}</p>
              <p v-if="reference.journal" class="journal-info">{{ reference.journal }}</p>
              <p v-if="reference.year" class="year">Published: {{ reference.year }}</p>
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
          v-if="interaction['3d-viewer']"
          type="success" 
          :icon="View"
          @click="handleView3DStructure"
          class="view-3d-btn"
        >
          🧬 View 3D Structure
        </el-button>
        <el-button 
          v-if="interaction.references && interaction.references.length > 0"
          type="primary" 
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
import EventBus from '../EventBus'
import { 
  Document, Connection, Location, Tools, Share, Warning, Reading, 
  Collection, InfoFilled, Link, WarningFilled, Right, View
} from '@element-plus/icons-vue'

export default {
  name: 'NerveInteractionDetailPopup',
  components: {
    ElDialog,
    ElButton,
    ElTag,
    ElIcon,
    Document,
    Connection,
    Location,
    Tools,
    Share,
    Warning,
    Reading,
    Collection,
    InfoFilled,
    Link,
    WarningFilled,
    Right,
    View
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
  emits: ['update:modelValue', 'open-research'],
  setup(props, { emit }) {
    const visible = computed({
      get: () => props.modelValue,
      set: (value) => emit('update:modelValue', value)
    })

    const additionalProperties = computed(() => {
      const excluded = ['nerve', 'targetProtein', 'toxin', 'system', 'keyFunctions', 'interaction', 'pathways', 'consequences', 'evidence', 'references']
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

    const openResearch = (url) => {
      emit('open-research', url)
    }

    const openDOI = (doi) => {
      const doiUrl = `https://doi.org/${doi}`
      emit('open-research', doiUrl)
    }

    const handleView3DStructure = () => {
      // Map target proteins to molecule keys for MultiFlatmap
      const proteinToMoleculeMap = {
        'Estrogen Receptor': 'estrogenReceptor',
        'Transthyretin (TTR)': 'transthyretin',
        'Transthyretin': 'transthyretin',
        'Androgen Receptor (AR)': 'ppar', // Using PPAR as placeholder for AR
        'Androgen Receptor': 'ppar',
        'PPAR': 'ppar'
      }

      const moleculeKey = proteinToMoleculeMap[props.interaction.targetProtein] || 'estrogenReceptor'
      
      // Emit EventBus event to MultiFlatmap
      EventBus.emit('open3DViewer', {
        moleculeKey: moleculeKey,
        moleculeName: props.interaction.targetProtein,
        viewerData: props.interaction['3d-viewer'],
        interaction: props.interaction
      })

      // Close the popup after opening 3D viewer
      visible.value = false
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
        case 'strong': return 'Well-documented with multiple independent studies and clinical observations'
        case 'moderate': return 'Supported by several studies with consistent neurological findings'
        case 'emerging': return 'Initial evidence with promising neurological research'
        case 'limited': return 'Limited but relevant neurological evidence available'
        default: return 'Evidence level not specified'
      }
    }

    const getNerveLocationDescription = (nerve) => {
      const descriptions = {
        'vagus': 'Major cranial nerve controlling parasympathetic functions',
        'sciatic': 'Largest peripheral nerve in the human body',
        'median': 'Major nerve of the upper limb',
        'ulnar': 'Nerve of the forearm and hand',
        'facial': 'Cranial nerve controlling facial muscles',
        'trigeminal': 'Cranial nerve responsible for facial sensation',
        'optic': 'Cranial nerve responsible for vision',
        'oculomotor': 'Cranial nerve controlling eye movement'
      }
      
      const nerveKey = nerve.toLowerCase().split(' ')[0]
      return descriptions[nerveKey] || 'Important component of the nervous system'
    }

    const getFunctionDescription = (functions) => {
      const descriptions = {
        'motor': 'Controls voluntary muscle movement and coordination',
        'sensory': 'Processes sensory information from the environment',
        'autonomic': 'Regulates involuntary bodily functions',
        'mixed': 'Combines both motor and sensory functions',
        'parasympathetic': 'Controls rest and digest functions',
        'sympathetic': 'Controls fight or flight responses'
      }
      
      const functionKey = functions.toLowerCase().split(' ')[0]
      return descriptions[functionKey] || 'Essential neurological function'
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
      openDOI,
      handleView3DStructure,
      getEvidenceType,
      getEvidenceDescription,
      getNerveLocationDescription,
      getFunctionDescription,
      formatKey,
      formatValue
    }
  }
}
</script>

<style scoped lang="scss">
.nerve-detail-popup {
  .popup-content {
    max-height: 75vh;
    overflow-y: auto;
    
    .header-section {
      background: linear-gradient(135deg, #fff7e6 0%, #fff1f0 100%);
      border-radius: 12px;
      padding: 24px;
      margin-bottom: 24px;
      
      .nerve-overview {
        margin-bottom: 16px;
        
        h2 {
          margin: 0 0 12px 0;
          color: #ad4e00;
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
        grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
        gap: 16px;
        
        .stat-item {
          background: white;
          padding: 12px 16px;
          border-radius: 8px;
          box-shadow: 0 2px 4px rgba(0,0,0,0.05);
          border-left: 4px solid #fa8c16;
          
          .stat-label {
            display: block;
            font-size: 12px;
            color: #909399;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            margin-bottom: 4px;
          }
          
          .stat-value {
            font-size: 14px;
            font-weight: 600;
            color: #303133;
            line-height: 1.3;
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
        border-left: 4px solid #fa8c16;
      }
      
      .nerve-info-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
        gap: 20px;
        
        .nerve-info-card {
          border-radius: 12px;
          padding: 20px;
          transition: all 0.3s ease;
          
          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 25px rgba(0,0,0,0.1);
          }
          
          &.location {
            background: linear-gradient(135deg, #e6f7ff 0%, #f0f8ff 100%);
            border: 2px solid #91d5ff;
          }
          
          &.functions {
            background: linear-gradient(135deg, #f6ffed 0%, #f0f9ff 100%);
            border: 2px solid #95de64;
          }
          
          .info-header {
            display: flex;
            align-items: center;
            gap: 12px;
            margin-bottom: 16px;
            
            .info-icon {
              font-size: 24px;
            }
            
            .location & .info-icon {
              color: #1890ff;
            }
            
            .functions & .info-icon {
              color: #52c41a;
            }
            
            h4 {
              margin: 0;
              font-size: 18px;
              font-weight: 600;
            }
          }
          
          .info-content {
            p {
              margin: 0 0 8px 0;
              
              &:first-child {
                font-size: 16px;
                font-weight: 600;
                color: #303133;
              }
              
              &.info-description {
                color: #8c8c8c;
                font-size: 13px;
                line-height: 1.4;
              }
            }
          }
        }
      }
      
      .pathways-grid {
        display: flex;
        flex-direction: column;
        gap: 12px;
        
        .pathway-card {
          display: flex;
          align-items: flex-start;
          gap: 16px;
          background: linear-gradient(135deg, #f0f8ff 0%, #e6f4ff 100%);
          border: 1px solid #91d5ff;
          border-radius: 8px;
          padding: 16px;
          transition: all 0.3s ease;
          
          &:hover {
            border-color: #40a9ff;
            box-shadow: 0 2px 8px rgba(64, 169, 255, 0.2);
          }
          
          .pathway-number {
            display: flex;
            align-items: center;
            justify-content: center;
            width: 32px;
            height: 32px;
            background: #1890ff;
            color: white;
            border-radius: 50%;
            font-weight: 600;
            font-size: 14px;
            flex-shrink: 0;
          }
          
          .pathway-content {
            flex: 1;
            
            p {
              margin: 0;
              color: #303133;
              font-size: 14px;
              line-height: 1.5;
            }
          }
        }
      }
      
      .consequences-grid {
        display: flex;
        flex-direction: column;
        gap: 12px;
        
        .consequence-card {
          display: flex;
          align-items: flex-start;
          gap: 16px;
          background: linear-gradient(135deg, #fff1f0 0%, #ffebe6 100%);
          border: 1px solid #ffaaa5;
          border-radius: 8px;
          padding: 16px;
          transition: all 0.3s ease;
          
          &:hover {
            border-color: #ff7875;
            box-shadow: 0 2px 8px rgba(255, 120, 117, 0.2);
          }
          
          .consequence-severity {
            display: flex;
            align-items: center;
            justify-content: center;
            width: 32px;
            height: 32px;
            background: #ff4d4f;
            color: white;
            border-radius: 50%;
            flex-shrink: 0;
            
            .el-icon {
              font-size: 16px;
            }
          }
          
          .consequence-content {
            flex: 1;
            
            .consequence-parts {
              display: flex;
              align-items: center;
              gap: 12px;
              flex-wrap: wrap;
              
              .cause {
                color: #cf1322;
                font-weight: 600;
                font-size: 14px;
              }
              
              .arrow {
                color: #ff7875;
                font-size: 16px;
              }
              
              .effect {
                color: #a8071a;
                font-size: 14px;
                font-style: italic;
              }
            }
          }
        }
      }
      
      .evidence-detailed {
        .evidence-level-card {
          background: linear-gradient(135deg, #f9f0ff 0%, #f0f8ff 100%);
          border: 1px solid #d3adf7;
          border-radius: 12px;
          padding: 20px;
          
          .evidence-header {
            margin-bottom: 16px;
          }
          
          .evidence-description {
            p {
              margin: 0 0 12px 0;
              color: #531dab;
              font-size: 14px;
              line-height: 1.5;
            }
            
            .evidence-notes {
              margin-top: 16px;
              padding-top: 16px;
              border-top: 1px solid #d3adf7;
              
              strong {
                color: #722ed1;
                display: block;
                margin-bottom: 8px;
              }
              
              p {
                color: #9254de;
                font-style: italic;
              }
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
            border-color: #fa8c16;
            box-shadow: 0 2px 8px rgba(250, 140, 22, 0.1);
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
              
              &.authors {
                color: #303133;
                font-weight: 500;
              }
              
              &.journal-info {
                font-style: italic;
                color: #909399;
              }
              
              &.year {
                color: #8c8c8c;
                font-size: 12px;
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

    .view-3d-btn {
      background: linear-gradient(135deg, #8E4EC6, #9d4edd);
      border-color: #7b2cbf;
      transition: all 0.3s ease;

      &:hover {
        background: linear-gradient(135deg, #7b2cbf, #8E4EC6);
        border-color: #6f2899;
        transform: translateY(-1px);
        box-shadow: 0 4px 12px rgba(123, 44, 191, 0.3);
      }
    }
  }
}

// Responsive design
@media (max-width: 768px) {
  .nerve-detail-popup {
    .popup-content {
      .header-section .quick-stats {
        grid-template-columns: 1fr;
      }
      
      .nerve-info-grid {
        grid-template-columns: 1fr;
      }
      
      .consequence-card .consequence-parts {
        flex-direction: column;
        align-items: flex-start;
        gap: 8px;
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
