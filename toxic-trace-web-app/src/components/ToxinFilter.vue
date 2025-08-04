<template>
  <div class="toxin-filter">
    <div class="filter-header">
      <h4>🔍 Filter Toxins</h4>
      <small>Select filters to narrow down toxins</small>
    </div>
    
    <el-cascader
      v-model="selectedFilters"
      :options="filterOptions"
      :props="cascaderProps"
      clearable
      filterable
      multiple
      collapse-tags
      collapse-tags-tooltip
      placeholder="Choose filters..."
      class="filter-cascader"
      @change="handleFilterChange"
    />
    
    <div class="filter-summary" v-if="activeFilters.length > 0">
      <div class="summary-header">Active Filters:</div>
      <el-tag
        v-for="filter in activeFilters"
        :key="filter.key"
        closable
        @close="removeFilter(filter)"
        class="filter-tag"
        type="info"
      >
        {{ filter.label }}
      </el-tag>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'
import { ElCascader, ElTag } from 'element-plus'

export default {
  name: 'ToxinFilter',
  components: {
    ElCascader,
    ElTag
  },
  emits: ['filter-change'],
  setup(props, { emit }) {
    const selectedFilters = ref([])
    
    const cascaderProps = {
      multiple: true,
      checkStrictly: true,
      emitPath: false,
      label: 'label',
      value: 'value',
      children: 'children'
    }

    const filterOptions = computed(() => [
      {
        label: 'Severity',
        value: 'severity',
        children: [
          { label: 'Moderate', value: 'severity:moderate' },
          { label: 'High', value: 'severity:high' }
        ]
      },
      {
        label: 'Detox Difficulty',
        value: 'detoxDifficulty',
        children: [
          { label: 'Easy', value: 'detoxDifficulty:easy' },
          { label: 'Moderate', value: 'detoxDifficulty:moderate' },
          { label: 'Difficult', value: 'detoxDifficulty:difficult' }
        ]
      },
      {
        label: 'Elimination Speed',
        value: 'eliminationSpeed',
        children: [
          { label: 'Fast', value: 'eliminationSpeed:fast' },
          { label: 'Slow', value: 'eliminationSpeed:slow' },
          { label: 'Very Slow', value: 'eliminationSpeed:very slow' }
        ]
      },
      {
        label: 'Time to Eliminate',
        value: 'timeToEliminate',
        children: [
          { label: 'Days', value: 'timeToEliminate:days' },
          { label: 'Weeks', value: 'timeToEliminate:weeks' },
          { label: 'Years', value: 'timeToEliminate:years' }
        ]
      },
      {
        label: 'Affected Systems',
        value: 'affectedSystems',
        children: [
          { label: 'Endocrine', value: 'affectedSystems:endocrine' },
          { label: 'Reproductive', value: 'affectedSystems:reproductive' },
          { label: 'Cardiovascular', value: 'affectedSystems:cardiovascular' },
          { label: 'Respiratory', value: 'affectedSystems:respiratory' },
          { label: 'Immune', value: 'affectedSystems:immune' },
          { label: 'Liver', value: 'affectedSystems:liver' },
          { label: 'Kidney', value: 'affectedSystems:kidney' },
          { label: 'Digestive', value: 'affectedSystems:digestive' },
          { label: 'Circulatory', value: 'affectedSystems:circulatory' }
        ]
      },
      {
        label: 'Detox Methods',
        value: 'detoxMethods',
        children: [
          { label: 'Dietary', value: 'detoxMethods:dietary' },
          { label: 'Sauna', value: 'detoxMethods:sauna' },
          { label: 'Exercise', value: 'detoxMethods:exercise' },
          { label: 'Supplements', value: 'detoxMethods:supplements' },
          { label: 'Probiotics', value: 'detoxMethods:probiotics' },
          { label: 'Binders', value: 'detoxMethods:binders' },
          { label: 'Antioxidants', value: 'detoxMethods:antioxidants' },
          { label: 'Specialized Binders', value: 'detoxMethods:specialized binders' },
          { label: 'Gut Health', value: 'detoxMethods:gut health' }
        ]
      },
      {
        label: 'Evidence Level',
        value: 'evidenceLevel',
        children: [
          { label: 'Strong', value: 'evidenceLevel:strong' },
          { label: 'Moderate', value: 'evidenceLevel:moderate' },
          { label: 'Emerging', value: 'evidenceLevel:emerging' }
        ]
      },
      {
        label: 'Primary Source',
        value: 'primarySource',
        children: [
          { label: 'Food Packaging', value: 'primarySource:food packaging' },
          { label: 'Plastic Containers', value: 'primarySource:plastic containers' },
          { label: 'Receipts', value: 'primarySource:receipts' },
          { label: 'Personal Care Products', value: 'primarySource:personal care products' },
          { label: 'Vinyl Flooring', value: 'primarySource:vinyl flooring' },
          { label: 'Non-stick Cookware', value: 'primarySource:non-stick cookware' },
          { label: 'Waterproof Clothing', value: 'primarySource:waterproof clothing' },
          { label: 'Firefighting Foam', value: 'primarySource:firefighting foam' },
          { label: 'Bottled Water', value: 'primarySource:bottled water' },
          { label: 'Seafood', value: 'primarySource:seafood' },
          { label: 'Processed Foods', value: 'primarySource:processed foods' }
        ]
      }
    ])

    const activeFilters = computed(() => {
      return selectedFilters.value.map(filterValue => {
        const [category, value] = filterValue.split(':')
        return {
          key: filterValue,
          label: `${category}: ${value}`,
          category,
          value
        }
      })
    })

    const handleFilterChange = (values) => {
      const filters = values.map(value => {
        const [category, filterValue] = value.split(':')
        return { category, value: filterValue }
      })
      emit('filter-change', filters)
    }

    const removeFilter = (filter) => {
      selectedFilters.value = selectedFilters.value.filter(f => f !== filter.key)
    }

    const clearAllFilters = () => {
      selectedFilters.value = []
    }

    watch(selectedFilters, () => {
      handleFilterChange(selectedFilters.value)
    })

    return {
      selectedFilters,
      filterOptions,
      cascaderProps,
      activeFilters,
      handleFilterChange,
      removeFilter,
      clearAllFilters
    }
  }
}
</script>

<style scoped lang="scss">
.toxin-filter {
  margin-bottom: 20px;
  padding: 16px;
  background: #f8f9fa;
  border-radius: 8px;
  border: 1px solid #e9ecef;

  .filter-header {
    margin-bottom: 12px;
    
    h4 {
      margin: 0 0 4px 0;
      color: #303133;
      font-size: 14px;
      font-weight: 600;
    }
    
    small {
      color: #606266;
      font-size: 12px;
    }
  }

  .filter-cascader {
    width: 100%;
    margin-bottom: 12px;
  }

  .filter-summary {
    .summary-header {
      font-size: 12px;
      color: #303133;
      font-weight: 500;
      margin-bottom: 8px;
    }
    
    display: flex;
    flex-direction: column;
    gap: 8px;

    .filter-tag {
      margin: 2px;
      font-size: 11px;
    }
  }
}

// Make sure the cascader is properly styled
:deep(.el-cascader) {
  .el-input {
    background: white;
  }
}
</style>
