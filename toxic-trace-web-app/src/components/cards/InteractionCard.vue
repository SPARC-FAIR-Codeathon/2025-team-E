<template>
  <div 
    class="interaction-card"
    @click="$emit('select', interaction)"
    :class="{ 'selected': isSelected }"
  >
    <div class="card-header">
      <h4>{{ interaction.targetProtein }}</h4>
      <div class="interaction-badges">
        <el-tag 
          :type="getEvidenceType(interaction.evidence.level)"
          size="small"
        >
          {{ interaction.evidence.level }}
        </el-tag>
        <el-tag 
          v-if="interaction.evidence.hasStructuralData"
          type="success"
          size="small"
        >
          3D Structure
        </el-tag>
      </div>
    </div>
    
    <div class="card-content">
      <div class="interaction-info">
        <div class="info-item">
          <strong>System:</strong> {{ interaction.system }}
        </div>
        <div class="info-item">
          <strong>Toxin:</strong> {{ interaction.toxin }}
        </div>
        <div class="info-item">
          <strong>Mechanism:</strong> {{ interaction.interaction.mechanism }}
        </div>
      </div>

      <p class="interaction-description">
        {{ interaction.interaction.description }}
      </p>

      <div class="structure-info" v-if="interaction.structures">
        <div class="structure-row">
          <strong>Natural:</strong> 
          <span class="pdb-id">{{ interaction.structures.naturalLigand.pdbId }}</span>
          <span class="potency">({{ interaction.structures.naturalLigand.potency }})</span>
        </div>
        <div class="structure-row" v-if="interaction.structures.toxinComplex.pdbId">
          <strong>Toxin:</strong> 
          <span class="pdb-id">{{ interaction.structures.toxinComplex.pdbId }}</span>
          <span class="potency">({{ interaction.structures.toxinComplex.potency }})</span>
        </div>
      </div>
    </div>

    <div class="card-footer">
      <el-button 
        type="primary" 
        size="small"
        @click.stop="showDetailPopup = true"
      >
        View Details
      </el-button>
      <el-button 
        v-if="interaction.references && interaction.references.length > 0"
        type="text" 
        size="small"
        @click.stop="$emit('open-research', interaction.references[0].url)"
      >
        Research
      </el-button>
      <el-button 
        v-if="interaction['3d-viewer']"
        type="success" 
        size="small"
        @click.stop="handleView3DStructure"
        class="view-3d-btn"
      >
        🧬 View 3D
      </el-button>
    </div>

    <!-- Detail Popup -->
    <InteractionDetailPopup
      v-model="showDetailPopup"
      :interaction="interaction"
      @view-structure="handleViewStructure"
      @open-research="handleOpenResearch"
    />
  </div>
</template>

<script>
import { ref } from 'vue'
import { ElButton, ElTag } from 'element-plus'
import EventBus from '../EventBus'
import InteractionDetailPopup from '../popups/InteractionDetailPopup.vue'

export default {
  name: 'InteractionCard',
  components: {
    ElButton,
    ElTag,
    InteractionDetailPopup
  },
  props: {
    interaction: {
      type: Object,
      required: true
    },
    isSelected: {
      type: Boolean,
      default: false
    }
  },
  emits: ['select', 'view-details', 'open-research', 'view-structure', 'view-3d-structure'],
  setup(props, { emit }) {
    const showDetailPopup = ref(false)

    const handleViewStructure = (pdbId) => {
      emit('view-structure', pdbId)
    }

    const handleOpenResearch = (url) => {
      emit('open-research', url)
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

      // Also emit local event for parent components
      emit('view-3d-structure', {
        moleculeKey: moleculeKey,
        moleculeName: props.interaction.targetProtein,
        viewerData: props.interaction['3d-viewer']
      })
    }

    const getEvidenceType = (level) => {
      switch (level.toLowerCase()) {
        case 'strong': return 'success'
        case 'moderate': return 'warning'
        case 'emerging': return 'info'
        default: return 'info'
      }
    }

    return {
      showDetailPopup,
      handleViewStructure,
      handleOpenResearch,
      handleView3DStructure,
      getEvidenceType
    }
  }
}
</script>

<style scoped lang="scss">
.interaction-card {
  border: 1px solid #e4e7ed;
  border-radius: 8px;
  padding: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
  background: white;

  &:hover {
    border-color: #409eff;
    box-shadow: 0 2px 8px rgba(64, 158, 255, 0.1);
  }

  &.selected {
    border-color: #409eff;
    background: #f0f8ff;
    box-shadow: 0 2px 8px rgba(64, 158, 255, 0.2);
  }

  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 12px;

    h4 {
      margin: 0;
      color: #303133;
      font-size: 16px;
      font-weight: 600;
      flex: 1;
      line-height: 1.3;
    }

    .interaction-badges {
      display: flex;
      gap: 4px;
      flex-wrap: wrap;
      margin-left: 8px;
    }
  }

  .card-content {
    .interaction-description {
      font-size: 13px;
      color: #606266;
      margin-bottom: 12px;
      line-height: 1.4;
    }

    .interaction-info {
      margin-bottom: 12px;

      .info-item {
        font-size: 12px;
        color: #606266;
        margin-bottom: 4px;

        strong {
          color: #303133;
        }
      }
    }

    .structure-info {
      margin-bottom: 12px;
      padding: 8px;
      background: #f8f9fc;
      border-radius: 4px;
      border-left: 3px solid #409eff;

      .structure-row {
        font-size: 11px;
        margin-bottom: 4px;

        strong {
          color: #303133;
        }

        .pdb-id {
          font-family: 'Courier New', monospace;
          background: #e6f7ff;
          padding: 1px 4px;
          border-radius: 3px;
          color: #1890ff;
          margin: 0 4px;
        }

        .potency {
          color: #666;
          font-style: italic;
        }
      }
    }
  }

  .card-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 12px;
    padding-top: 12px;
    border-top: 1px solid #ebeef5;
    gap: 8px;
    flex-wrap: wrap;

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
</style>
