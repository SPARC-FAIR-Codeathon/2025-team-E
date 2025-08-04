<template>
  <div 
    class="toxin-card"
    @click="$emit('select', toxin)"
    :class="{ 'selected': isSelected }"
  >
    <div class="card-header">
      <h4>{{ toxin.name }}</h4>
      <div class="toxin-badges">
        <el-tag 
          :type="getSeverityType(toxin.filters.severity)"
          size="small"
        >
          {{ capitalizeFirst(toxin.filters.severity) }}
        </el-tag>
        <el-tag 
          :type="getDifficultyType(toxin.filters.detoxDifficulty)"
          size="small"
        >
          {{ capitalizeFirst(toxin.filters.detoxDifficulty) }}
        </el-tag>
      </div>
    </div>
    
    <div class="card-content">
      <p class="toxin-description">
        {{ toxin.detoxEvidence.description }}
      </p>
      
      <div class="toxin-info">
        <div class="info-item">
          <strong>Elimination:</strong> {{ toxin.filters.eliminationSpeed }}
        </div>
        <div class="info-item">
          <strong>Time Frame:</strong> {{ toxin.filters.timeToEliminate }}
        </div>
        <div class="info-item">
          <strong>Evidence:</strong> {{ toxin.filters.evidenceLevel }}
        </div>
      </div>

      <div class="affected-systems">
        <strong>Affected Systems:</strong>
        <div class="system-tags">
          <el-tag 
            v-for="system in toxin.filters.affectedSystems" 
            :key="system"
            size="mini"
            class="system-tag"
          >
            {{ capitalizeFirst(system) }}
          </el-tag>
        </div>
      </div>

      <div class="detox-methods">
        <strong>Detox Methods:</strong>
        <div class="method-tags">
          <el-tag 
            v-for="method in toxin.filters.detoxMethods" 
            :key="method"
            type="success"
            size="mini"
            class="method-tag"
          >
            {{ capitalizeFirst(method) }}
          </el-tag>
        </div>
      </div>

      <div class="primary-sources">
        <strong>Common Sources:</strong>
        <span class="sources-text">
          {{ toxin.filters.primarySource.join(', ') }}
        </span>
      </div>
    </div>

    <div class="card-footer">
      <el-button 
        type="primary" 
        size="small"
        @click.stop="$emit('view-details', toxin)"
      >
        View Details
      </el-button>
      <el-button 
        v-if="toxin.detoxEvidence.link"
        type="text" 
        size="small"
        @click.stop="$emit('open-research', toxin.detoxEvidence.link)"
      >
        Research
      </el-button>
    </div>
  </div>
</template>

<script>
import { ElButton, ElTag } from 'element-plus'

export default {
  name: 'ToxinCard',
  components: {
    ElButton,
    ElTag
  },
  props: {
    toxin: {
      type: Object,
      required: true
    },
    isSelected: {
      type: Boolean,
      default: false
    }
  },
  emits: ['select', 'view-details', 'open-research'],
  methods: {
    capitalizeFirst(str) {
      return str.charAt(0).toUpperCase() + str.slice(1)
    },
    getSeverityType(severity) {
      switch (severity) {
        case 'high': return 'danger'
        case 'moderate': return 'warning'
        default: return 'info'
      }
    },
    getDifficultyType(difficulty) {
      switch (difficulty) {
        case 'difficult': return 'danger'
        case 'moderate': return 'warning'
        case 'easy': return 'success'
        default: return 'info'
      }
    }
  }
}
</script>

<style scoped lang="scss">
.toxin-card {
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

    .toxin-badges {
      display: flex;
      gap: 4px;
      flex-wrap: wrap;
      margin-left: 8px;
    }
  }

  .card-content {
    .toxin-description {
      font-size: 13px;
      color: #606266;
      margin-bottom: 12px;
      line-height: 1.4;
    }

    .toxin-info {
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

    .affected-systems,
    .detox-methods {
      margin-bottom: 12px;

      strong {
        display: block;
        font-size: 12px;
        color: #303133;
        margin-bottom: 6px;
      }

      .system-tags,
      .method-tags {
        display: flex;
        flex-wrap: wrap;
        gap: 4px;
      }

      .system-tag,
      .method-tag {
        font-size: 11px;
      }
    }

    .primary-sources {
      strong {
        display: block;
        font-size: 12px;
        color: #303133;
        margin-bottom: 4px;
      }

      .sources-text {
        font-size: 11px;
        color: #606266;
        line-height: 1.3;
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
