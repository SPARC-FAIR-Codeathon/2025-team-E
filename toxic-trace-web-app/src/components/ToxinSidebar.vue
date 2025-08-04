<template>
  <div>
    <!-- Sidebar Tab (when sidebar is closed) -->
    <div 
      v-if="!showSidebar" 
      class="sidebar-tab"
      @click="openSidebar"
    >
      &lt;
    </div>

    <!-- Main Sidebar -->
    <div v-if="showSidebar" class="toxin-sidebar">
      <!-- Close Tab (when sidebar is open) -->
      <div 
        class="sidebar-close-tab"
        @click="closeSidebar"
      >
        &gt;
      </div>
      
      <div class="sidebar-header">
        <h3>Toxin Database</h3>
        <el-button 
          type="text" 
          @click="closeSidebar"
          class="close-button"
        >
          <el-icon><Close /></el-icon>
        </el-button>
      </div>
      
      <div class="sidebar-content">
        <!-- Filter Component -->
        <ToxinFilter @filter-change="handleFilterChange" />
        
        <!-- Results Count -->
        <div class="results-header">
          <span class="results-count">
            {{ filteredToxins.length }} of {{ toxinData.toxins.length }} toxins
          </span>
          <el-button 
            v-if="activeFilters.length > 0"
            type="text" 
            @click="clearFilters"
            class="clear-filters"
          >
            Clear All
          </el-button>
        </div>

        <!-- Toxin Cards -->
        <div class="toxin-cards">
          <div 
            v-for="toxin in filteredToxins" 
            :key="toxin.name"
            class="toxin-card"
            @click="selectToxin(toxin)"
            :class="{ 'selected': selectedToxins.includes(toxin.name) }"
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
                @click.stop="viewDetails(toxin)"
              >
                View Details
              </el-button>
              <el-button 
                v-if="toxin.detoxEvidence.link"
                type="text" 
                size="small"
                @click.stop="openResearch(toxin.detoxEvidence.link)"
              >
                Research
              </el-button>
            </div>
          </div>
        </div>

        <!-- Empty State -->
        <div v-if="filteredToxins.length === 0" class="empty-state">
          <el-empty description="No toxins match your filters">
            <el-button type="primary" @click="clearFilters">
              Clear Filters
            </el-button>
          </el-empty>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue'
import { ElButton, ElTag, ElIcon, ElEmpty } from 'element-plus'
import { Close } from '@element-plus/icons-vue'
import ToxinFilter from './ToxinFilter.vue'
import detoxificationData from './data/detoxification.json'

export default {
  name: 'ToxinSidebar',
  components: {
    ElButton,
    ElTag,
    ElIcon,
    ElEmpty,
    ToxinFilter,
    Close
  },
  emits: ['close', 'toxin-selected', 'view-details'],
  setup(props, { emit }) {
    const activeFilters = ref([])
    const selectedToxins = ref([])
    const toxinData = ref(detoxificationData)
    const showSidebar = ref(false) // Start with sidebar closed to show the tab

    const filteredToxins = computed(() => {
      if (activeFilters.value.length === 0) {
        return toxinData.value.toxins
      }

      return toxinData.value.toxins.filter(toxin => {
        return activeFilters.value.every(filter => {
          const { category, value } = filter
          const toxinFilterValue = toxin.filters[category]

          if (Array.isArray(toxinFilterValue)) {
            return toxinFilterValue.includes(value)
          }
          return toxinFilterValue === value
        })
      })
    })

    const handleFilterChange = (filters) => {
      activeFilters.value = filters
    }

    const clearFilters = () => {
      activeFilters.value = []
    }

    const selectToxin = (toxin) => {
      const index = selectedToxins.value.indexOf(toxin.name)
      if (index > -1) {
        selectedToxins.value.splice(index, 1)
      } else {
        selectedToxins.value.push(toxin.name)
      }
      emit('toxin-selected', selectedToxins.value)
    }

    const viewDetails = (toxin) => {
      emit('view-details', toxin)
    }

    const openResearch = (link) => {
      window.open(link, '_blank')
    }

    const closeSidebar = () => {
      showSidebar.value = false
      // Emit close event to parent when sidebar is closed via close button
      emit('close')
    }

    const openSidebar = () => {
      showSidebar.value = true
    }

    const capitalizeFirst = (str) => {
      return str.charAt(0).toUpperCase() + str.slice(1)
    }

    const getSeverityType = (severity) => {
      switch (severity) {
        case 'high': return 'danger'
        case 'moderate': return 'warning'
        default: return 'info'
      }
    }

    const getDifficultyType = (difficulty) => {
      switch (difficulty) {
        case 'difficult': return 'danger'
        case 'moderate': return 'warning'
        case 'easy': return 'success'
        default: return 'info'
      }
    }

    return {
      showSidebar,
      activeFilters,
      selectedToxins,
      toxinData,
      filteredToxins,
      handleFilterChange,
      clearFilters,
      selectToxin,
      viewDetails,
      openResearch,
      closeSidebar,
      openSidebar,
      capitalizeFirst,
      getSeverityType,
      getDifficultyType
    }
  }
}
</script>

<style scoped lang="scss">
// Sidebar Tab (when closed)
.sidebar-tab {
  position: fixed;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 30px;
  height: 60px;
  background: #409eff;
  border-radius: 8px 0 0 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 9999; // Very high z-index to ensure it's always visible
  box-shadow: -2px 0 8px rgba(0, 0, 0, 0.3);
  transition: all 0.3s ease;
  color: white;
  font-size: 16px;
  font-weight: bold;
  border: 2px solid #fff; // Add white border to make it more visible

  &:hover {
    background: #66b1ff;
    width: 35px;
    box-shadow: -4px 0 12px rgba(0, 0, 0, 0.4);
  }
}

.toxin-sidebar {
  width: 400px;
  height: 100vh;
  background: white;
  border-left: 1px solid #e4e7ed;
  display: flex;
  flex-direction: column;
  position: fixed;
  right: 0;
  top: 0;
  z-index: 9998; // High z-index but slightly lower than tab
  box-shadow: -2px 0 8px rgba(0, 0, 0, 0.1);

  // Close Tab (when sidebar is open)
  .sidebar-close-tab {
    position: absolute;
    left: -30px;
    top: 50%;
    transform: translateY(-50%);
    width: 30px;
    height: 60px;
    background: #409eff;
    border-radius: 8px 0 0 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    z-index: 10000; // Higher than sidebar itself
    box-shadow: -2px 0 8px rgba(0, 0, 0, 0.3);
    transition: all 0.3s ease;
    color: white;
    font-size: 16px;
    font-weight: bold;
    border: 2px solid #fff; // Add white border to make it more visible

    &:hover {
      background: #66b1ff;
      left: -35px;
      box-shadow: -4px 0 12px rgba(0, 0, 0, 0.4);
    }
  }

  .sidebar-header {
    padding: 20px;
    border-bottom: 1px solid #e4e7ed;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #f5f7fa;
    flex-shrink: 0; // Prevent header from shrinking

    h3 {
      margin: 0;
      color: #303133;
      font-size: 18px;
      font-weight: 600;
    }

    .close-button {
      padding: 4px;
      color: #909399;
      
      &:hover {
        color: #606266;
      }
    }
  }

  .sidebar-content {
    flex: 1;
    padding: 20px;
    overflow-y: auto;
    min-height: 0; // Allow flex item to shrink below content size

    .results-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 16px;
      padding-bottom: 8px;
      border-bottom: 1px solid #ebeef5;

      .results-count {
        font-size: 14px;
        color: #606266;
        font-weight: 500;
      }

      .clear-filters {
        font-size: 12px;
        padding: 4px 8px;
      }
    }
  }

  .toxin-cards {
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

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
      }

      .toxin-badges {
        display: flex;
        gap: 4px;
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

  .empty-state {
    text-align: center;
    padding: 40px 20px;
  }
}

// Responsive adjustments
@media (max-width: 768px) {
  .toxin-sidebar {
    width: 100vw;
    right: 0;
  }
  
  .sidebar-tab {
    width: 25px;
    height: 50px;
    
    &:hover {
      width: 30px;
    }
  }
}
</style>
