<template>
  <div 
    class="nerve-interaction-card"
    @click="$emit('select', interaction)"
    :class="{ 'selected': isSelected }"
  >
    <div class="card-header">
      <h4>{{ interaction.targetProtein }}</h4>
      <div class="nerve-interaction-badges">
        <el-tag 
          :type="getEvidenceType(interaction.evidence.level)"
          size="small"
        >
          {{ interaction.evidence.level }}
        </el-tag>
        <el-tag 
          type="warning"
          size="small"
        >
          {{ interaction.nerve.split(' ')[0] }}
        </el-tag>
      </div>
    </div>
    
    <div class="card-content">
      <div class="nerve-interaction-info">
        <div class="info-item">
          <strong>Nerve:</strong> {{ interaction.nerve }}
        </div>
        <div class="info-item">
          <strong>Toxin:</strong> {{ interaction.toxin }}
        </div>
        <div class="info-item">
          <strong>System:</strong> {{ interaction.system }}
        </div>
        <div class="info-item">
          <strong>Functions:</strong> {{ interaction.keyFunctions }}
        </div>
      </div>

      <p class="nerve-interaction-description">
        {{ interaction.interaction.description }}
      </p>

      <div class="pathways-info" v-if="interaction.pathways && interaction.pathways.length > 0">
        <strong>Key Pathways:</strong>
        <ul class="pathway-list">
          <li v-for="(pathway, index) in interaction.pathways.slice(0, 2)" :key="index">
            {{ pathway }}
          </li>
        </ul>
      </div>

      <div class="consequences-info" v-if="interaction.consequences && interaction.consequences.length > 0">
        <strong>Consequences:</strong>
        <div class="consequence-tags">
          <el-tag 
            v-for="(consequence, index) in interaction.consequences.slice(0, 3)" 
            :key="index"
            type="danger"
            size="mini"
            class="consequence-tag"
          >
            {{ consequence.split('→')[0].trim() }}
          </el-tag>
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
    </div>

    <!-- Detail Popup -->
    <NerveInteractionDetailPopup
      v-model="showDetailPopup"
      :interaction="interaction"
      @open-research="handleOpenResearch"
    />
  </div>
</template>

<script>
import { ref } from 'vue'
import { ElButton, ElTag } from 'element-plus'
import NerveInteractionDetailPopup from '../popups/NerveInteractionDetailPopup.vue'

export default {
  name: 'NerveInteractionCard',
  components: {
    ElButton,
    ElTag,
    NerveInteractionDetailPopup
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
  emits: ['select', 'view-details', 'open-research'],
  setup(props, { emit }) {
    const showDetailPopup = ref(false)

    const handleOpenResearch = (url) => {
      emit('open-research', url)
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
      handleOpenResearch,
      getEvidenceType
    }
  }
}
</script>

<style scoped lang="scss">
.nerve-interaction-card {
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

    .nerve-interaction-badges {
      display: flex;
      gap: 4px;
      flex-wrap: wrap;
      margin-left: 8px;
    }
  }

  .card-content {
    .nerve-interaction-description {
      font-size: 13px;
      color: #606266;
      margin-bottom: 12px;
      line-height: 1.4;
    }

    .nerve-interaction-info {
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

    .pathways-info,
    .consequences-info {
      margin-bottom: 12px;

      strong {
        display: block;
        font-size: 12px;
        color: #303133;
        margin-bottom: 6px;
      }

      .pathway-list {
        margin: 0;
        padding-left: 16px;
        
        li {
          font-size: 11px;
          color: #606266;
          line-height: 1.3;
          margin-bottom: 2px;
        }
      }

      .consequence-tags {
        display: flex;
        flex-wrap: wrap;
        gap: 4px;

        .consequence-tag {
          font-size: 10px;
          max-width: 120px;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
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
  }
}
</style>
