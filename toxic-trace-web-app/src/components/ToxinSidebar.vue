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
        <div class="header-controls">
          <h3>{{ currentMode === 'detox' ? 'Toxin Detox' : 
                 currentMode === 'interactions' ? 'Molecular Interactions' : 
                 'Nerve Interactions' }}</h3>
          <div class="mode-toggle">
            <el-button 
              :type="currentMode === 'detox' ? 'primary' : 'default'"
              size="large"
              @click="setMode('detox')"
              class="mode-button detox-button"
            >
              <el-icon><Aim /></el-icon>
              Detox
            </el-button>
            <el-button 
              :type="currentMode === 'interactions' ? 'primary' : 'default'"
              size="large"
              @click="setMode('interactions')"
              class="mode-button interactions-button"
            >
              <el-icon><Connection /></el-icon>
              Interactions
            </el-button>
            <el-button 
              :type="currentMode === 'nerves' ? 'primary' : 'default'"
              size="large"
              @click="setMode('nerves')"
              class="mode-button nerves-button"
            >
              <el-icon><Lightning /></el-icon>
              Nerves
            </el-button>
          </div>
        </div>
      </div>
      
      <div class="sidebar-content">
        <!-- Filter Component -->
        <ToxinFilter @filter-change="handleFilterChange" />
        
        <!-- Results Count -->
        <div class="results-header">
          <span class="results-count">
            {{ currentMode === 'detox' ? 
                `${filteredToxins.length} of ${toxinData.toxins.length} toxins` :
                currentMode === 'interactions' ?
                `${filteredInteractions.length} of ${interactionData.datasets.length} interactions` :
                `${filteredNerveInteractions.length} of ${nerveInteractionData.datasets.length} nerve interactions`
            }}
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

        <!-- Toxin Cards (Detox Mode) -->
        <div v-if="currentMode === 'detox'" class="toxin-cards">
          <ToxinCard
            v-for="toxin in filteredToxins" 
            :key="toxin.name"
            :toxin="toxin"
            :is-selected="selectedToxins.includes(toxin.name)"
            @select="selectToxin"
            @view-details="viewDetails"
            @open-research="openResearch"
          />
        </div>

        <!-- Interaction Cards (Interactions Mode) -->
        <div v-if="currentMode === 'interactions'" class="interaction-cards">
          <InteractionCard
            v-for="interaction in filteredInteractions" 
            :key="interaction.id"
            :interaction="interaction"
            :is-selected="selectedInteractions.includes(interaction.id)"
            @select="selectInteraction"
            @view-details="viewInteractionDetails"
            @open-research="openResearch"
          />
        </div>

        <!-- Nerve Interaction Cards (Nerves Mode) -->
        <div v-if="currentMode === 'nerves'" class="nerve-interaction-cards">
          <NerveInteractionCard
            v-for="interaction in filteredNerveInteractions" 
            :key="interaction.id"
            :interaction="interaction"
            :is-selected="selectedNerveInteractions.includes(interaction.id)"
            @select="selectNerveInteraction"
            @view-details="viewNerveInteractionDetails"
            @open-research="openResearch"
          />
        </div>

        <!-- Empty State -->
        <div v-if="(currentMode === 'detox' && filteredToxins.length === 0) || 
                   (currentMode === 'interactions' && filteredInteractions.length === 0) ||
                   (currentMode === 'nerves' && filteredNerveInteractions.length === 0)" 
             class="empty-state">
          <el-empty :description="`No ${currentMode === 'detox' ? 'toxins' : 
                                       currentMode === 'interactions' ? 'interactions' : 
                                       'nerve interactions'} match your filters`">
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
import { Close, Aim, Connection, Lightning } from '@element-plus/icons-vue'
import ToxinFilter from './ToxinFilter.vue'
import ToxinCard from './cards/ToxinCard.vue'
import InteractionCard from './cards/InteractionCard.vue'
import NerveInteractionCard from './cards/NerveInteractionCard.vue'
import detoxificationData from './data/detoxification.json'
import molecularInteractionsData from './data/molecular-interactions.json'
import nerveInteractionsData from './data/nerve-interactions.json'

export default {
  name: 'ToxinSidebar',
  components: {
    ElButton,
    ElTag,
    ElIcon,
    ElEmpty,
    ToxinFilter,
    ToxinCard,
    InteractionCard,
    NerveInteractionCard,
    Close,
    Aim,
    Connection,
    Lightning
  },
  emits: ['close', 'toxin-selected', 'view-details', 'interaction-selected', 'view-interaction-details', 'nerve-interaction-selected', 'view-nerve-interaction-details'],
  setup(props, { emit }) {
    const activeFilters = ref([])
    const selectedToxins = ref([])
    const selectedInteractions = ref([])
    const selectedNerveInteractions = ref([])
    const toxinData = ref(detoxificationData)
    const interactionData = ref(molecularInteractionsData)
    const nerveInteractionData = ref(nerveInteractionsData)
    const showSidebar = ref(false) // Start with sidebar closed to show the tab
    const currentMode = ref('detox') // 'detox', 'interactions', or 'nerves'

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

    const filteredInteractions = computed(() => {
      if (activeFilters.value.length === 0) {
        return interactionData.value.datasets
      }

      return interactionData.value.datasets.filter(interaction => {
        return activeFilters.value.every(filter => {
          const { category, value } = filter
          
          // Handle different filter categories for interactions
          switch (category) {
            case 'system':
              return interaction.system.toLowerCase() === value.toLowerCase()
            case 'toxin':
              return interaction.toxin.toLowerCase().includes(value.toLowerCase())
            case 'evidenceLevel':
              return interaction.evidence.level.toLowerCase() === value.toLowerCase()
            case 'hasStructuralData':
              return interaction.evidence.hasStructuralData === (value === 'true')
            case 'comparablePotencies':
              return interaction.evidence.comparablePotencies === (value === 'true')
            default:
              return true
          }
        })
      })
    })

    const filteredNerveInteractions = computed(() => {
      if (activeFilters.value.length === 0) {
        return nerveInteractionData.value.datasets
      }

      return nerveInteractionData.value.datasets.filter(interaction => {
        return activeFilters.value.every(filter => {
          const { category, value } = filter
          
          // Handle different filter categories for nerve interactions
          switch (category) {
            case 'system':
              return interaction.system.toLowerCase() === value.toLowerCase()
            case 'toxin':
              return interaction.toxin.toLowerCase().includes(value.toLowerCase())
            case 'evidenceLevel':
              return interaction.evidence.level.toLowerCase() === value.toLowerCase()
            case 'nerve':
              return interaction.nerve.toLowerCase().includes(value.toLowerCase())
            case 'targetProtein':
              return interaction.targetProtein.toLowerCase().includes(value.toLowerCase())
            default:
              return true
          }
        })
      })
    })

    const setMode = (mode) => {
      currentMode.value = mode
      // Clear filters when switching modes
      activeFilters.value = []
      selectedToxins.value = []
      selectedInteractions.value = []
      selectedNerveInteractions.value = []
    }

    const selectInteraction = (interaction) => {
      const index = selectedInteractions.value.indexOf(interaction.id)
      if (index > -1) {
        selectedInteractions.value.splice(index, 1)
      } else {
        selectedInteractions.value.push(interaction.id)
      }
      emit('interaction-selected', selectedInteractions.value)
    }

    const selectNerveInteraction = (interaction) => {
      const index = selectedNerveInteractions.value.indexOf(interaction.id)
      if (index > -1) {
        selectedNerveInteractions.value.splice(index, 1)
      } else {
        selectedNerveInteractions.value.push(interaction.id)
      }
      emit('nerve-interaction-selected', selectedNerveInteractions.value)
    }

    const viewInteractionDetails = (interaction) => {
      emit('view-interaction-details', interaction)
    }

    const viewNerveInteractionDetails = (interaction) => {
      emit('view-nerve-interaction-details', interaction)
    }

    const getEvidenceType = (level) => {
      switch (level.toLowerCase()) {
        case 'strong': return 'success'
        case 'moderate': return 'warning'
        case 'emerging': return 'info'
        default: return 'info'
      }
    }

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
      currentMode,
      activeFilters,
      selectedToxins,
      selectedInteractions,
      selectedNerveInteractions,
      toxinData,
      interactionData,
      nerveInteractionData,
      filteredToxins,
      filteredInteractions,
      filteredNerveInteractions,
      handleFilterChange,
      clearFilters,
      selectToxin,
      selectInteraction,
      selectNerveInteraction,
      viewDetails,
      viewInteractionDetails,
      viewNerveInteractionDetails,
      openResearch,
      closeSidebar,
      openSidebar,
      setMode,
      capitalizeFirst,
      getSeverityType,
      getDifficultyType,
      getEvidenceType
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
    background: #f5f7fa;
    flex-shrink: 0; // Prevent header from shrinking

    .header-controls {
      display: flex;
      flex-direction: column;
      gap: 12px;

      h3 {
        margin: 0;
        color: #303133;
        font-size: 18px;
        font-weight: 600;
        text-align: center;
        padding-bottom: 8px;
        border-bottom: 1px solid #e4e7ed;
        width: 100%;
        display: block;
      }

      .mode-toggle {
        display: flex;
        gap: 8px;
        justify-content: center;
        width: 100%;

        .mode-button {
          flex: 1;
          padding: 12px 8px;
          font-size: 13px;
          font-weight: 600;
          border-radius: 10px;
          transition: all 0.3s ease;
          display: flex;
          align-items: center;
          justify-content: center;
          gap: 6px;
          min-height: 45px;
          position: relative;
          overflow: hidden;

          .el-icon {
            font-size: 16px;
          }

          &.detox-button {
            background: linear-gradient(135deg, #10b981 0%, #059669 100%);
            border: none;
            color: white;
            
            &:not(.el-button--primary) {
              background: linear-gradient(135deg, #f0f9ff 0%, #e0f2fe 100%);
              color: #0891b2;
              border: 2px solid #0891b2;
            }
          }

          &.interactions-button {
            background: linear-gradient(135deg, #8b5cf6 0%, #7c3aed 100%);
            border: none;
            color: white;
            
            &:not(.el-button--primary) {
              background: linear-gradient(135deg, #faf5ff 0%, #f3e8ff 100%);
              color: #8b5cf6;
              border: 2px solid #8b5cf6;
            }
          }

          &.nerves-button {
            background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);
            border: none;
            color: white;
            
            &:not(.el-button--primary) {
              background: linear-gradient(135deg, #fffbeb 0%, #fef3c7 100%);
              color: #d97706;
              border: 2px solid #d97706;
            }
          }

          &::before {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255,255,255,0.3), transparent);
            transition: left 0.5s ease;
          }

          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.15);

            &::before {
              left: 100%;
            }

            .el-icon {
              transform: scale(1.1);
            }
          }

          &:active {
            transform: translateY(0);
          }
        }
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

  .toxin-cards,
  .interaction-cards,
  .nerve-interaction-cards {
    display: flex;
    flex-direction: column;
    gap: 16px;
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
