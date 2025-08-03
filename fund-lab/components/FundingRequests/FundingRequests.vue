<template>
  <div class="funding-requests">
    <div class="section-header mb-24">
      <h2 class="section-title">Active Funding Requests</h2>
      <p class="section-description">
        Browse current research projects seeking funding support. Connect with researchers 
        and discover innovative opportunities in biomedical research.
      </p>
    </div>
    
    <div class="filters-section mb-24">
      <div class="filter-controls">
        <el-select v-model="selectedCategory" placeholder="All Categories" clearable class="category-filter">
          <el-option label="All Categories" value="" />
          <el-option label="Neuroscience" value="neuroscience" />
          <el-option label="Bioengineering" value="bioengineering" />
          <el-option label="Data Science" value="data-science" />
          <el-option label="Medical Devices" value="medical-devices" />
        </el-select>
        
        <el-select v-model="selectedFundingRange" placeholder="Funding Range" clearable class="funding-filter">
          <el-option label="All Amounts" value="" />
          <el-option label="Under $50K" value="under-50k" />
          <el-option label="$50K - $100K" value="50k-100k" />
          <el-option label="$100K - $500K" value="100k-500k" />
          <el-option label="Over $500K" value="over-500k" />
        </el-select>
      </div>
    </div>

    <div class="requests-list">
      <funding-request-card 
        v-for="request in filteredRequests" 
        :key="request.id"
        :request="request"
      />
    </div>

    <div v-if="filteredRequests.length === 0" class="no-results">
      <p>No funding requests match your current filters.</p>
    </div>

    <div class="load-more-section" v-if="showLoadMore">
      <el-button type="default" @click="loadMoreRequests">
        Load More Projects
      </el-button>
    </div>
  </div>
</template>

<script>
import FundingRequestCard from '@/components/FundingRequestCard/FundingRequestCard.vue'

export default {
  name: 'FundingRequests',
  components: {
    FundingRequestCard
  },
  data() {
    return {
      selectedCategory: '',
      selectedFundingRange: '',
      showLoadMore: true,
      fundingRequests: [
        {
          id: 1,
          title: "Understanding how PFAS binds to Estrogen Receptors",
          ownerName: "Dr. Sarah Chen",
          orcidId: "0000-0002-1825-0097",
          fundingAmount: 125000,
          fundingTimespan: "18 months",
          category: "neuroscience",
          description: `**Background:** Per- and polyfluoroalkyl substances (PFAS) are persistent environmental contaminants that have been linked to endocrine disruption. This research aims to understand the molecular mechanisms by which PFAS compounds interact with estrogen receptors.

**Objectives:**
- Characterize binding affinity of major PFAS compounds to estrogen receptor subtypes
- Investigate downstream signaling pathway alterations
- Develop predictive models for PFAS endocrine disruption potential

**Expected Impact:** Results will inform risk assessment protocols and support development of safer chemical alternatives.`
        },
        {
          id: 2,
          title: "Neural Interface for Spinal Cord Injury Recovery",
          ownerName: "Dr. Michael Rodriguez",
          orcidId: "0000-0003-2156-8745",
          fundingAmount: 450000,
          fundingTimespan: "3 years",
          category: "bioengineering",
          description: `**Project Summary:** Development of a novel brain-computer interface system to restore motor function in individuals with spinal cord injuries.

**Innovation:** Our approach combines advanced neural signal processing with closed-loop stimulation to bypass damaged neural pathways.

**Key Deliverables:**
- Prototype neural interface device
- Clinical trial preparation
- Regulatory pathway documentation

**Clinical Significance:** This technology could potentially restore movement for thousands of paralyzed individuals.`
        },
        {
          id: 3,
          title: "AI-Powered Drug Discovery for Rare Diseases",
          ownerName: "Dr. Emily Watson",
          orcidId: "0000-0001-9834-5672",
          fundingAmount: 275000,
          fundingTimespan: "2 years",
          category: "data-science",
          description: `**Mission:** Leverage machine learning to accelerate drug discovery for rare diseases affecting fewer than 200,000 people in the US.

**Methodology:**
- Deep learning models trained on molecular structure data
- Integration with existing rare disease databases
- Validation through in-vitro screening

**Target Diseases:** Focus on rare neurological and metabolic disorders with no current treatments.

**Expected Outcomes:** Identification of 5-10 promising drug candidates for further development.`
        },
        {
          id: 4,
          title: "Wearable Glucose Monitoring for Diabetic Patients",
          ownerName: "Dr. James Park",
          orcidId: "0000-0004-7123-4567",
          fundingAmount: 85000,
          fundingTimespan: "15 months",
          category: "medical-devices",
          description: `**Problem:** Current continuous glucose monitors are expensive and require frequent calibration.

**Solution:** Development of a low-cost, non-invasive wearable device using novel biosensor technology.

**Technical Approach:**
- Micro-needle array with integrated sensors
- Wireless data transmission to smartphone app
- Machine learning for personalized glucose predictions

**Market Impact:** Potential to make continuous glucose monitoring accessible to underserved populations.`
        },
        {
          id: 5,
          title: "Biomarker Discovery for Early Alzheimer's Detection",
          ownerName: "Dr. Lisa Thompson",
          orcidId: "0000-0002-8901-2345",
          fundingAmount: 320000,
          fundingTimespan: "30 months",
          category: "neuroscience",
          description: `**Objective:** Identify novel blood-based biomarkers for early detection of Alzheimer's disease, years before clinical symptoms appear.

**Approach:**
- Multi-omics analysis of longitudinal cohort data
- Integration of proteomics, metabolomics, and genetic data
- Machine learning model development for risk prediction

**Clinical Translation:** Development of a simple blood test for early Alzheimer's screening in primary care settings.

**Potential Impact:** Earlier intervention could significantly improve patient outcomes and reduce healthcare costs.`
        }
      ]
    }
  },
  computed: {
    filteredRequests() {
      let filtered = this.fundingRequests

      if (this.selectedCategory) {
        filtered = filtered.filter(request => request.category === this.selectedCategory)
      }

      if (this.selectedFundingRange) {
        filtered = filtered.filter(request => {
          const amount = request.fundingAmount
          switch (this.selectedFundingRange) {
            case 'under-50k':
              return amount < 50000
            case '50k-100k':
              return amount >= 50000 && amount <= 100000
            case '100k-500k':
              return amount > 100000 && amount <= 500000
            case 'over-500k':
              return amount > 500000
            default:
              return true
          }
        })
      }

      return filtered
    }
  },
  methods: {
    loadMoreRequests() {
      // Placeholder for loading more requests from API
      this.showLoadMore = false
    }
  }
}
</script>

<style scoped lang="scss">
@import 'sparc-design-system-components-2/src/assets/_variables.scss';

.funding-requests {
  width: 100%;
}

.section-header {
  text-align: center;
  
  .section-title {
    font-size: 2rem;
    font-weight: 600;
    color: $darkBlue;
    margin-bottom: 1rem;
  }
  
  .section-description {
    font-size: 1.125rem;
    color: #666;
    max-width: 600px;
    margin: 0 auto;
    line-height: 1.6;
  }
}

.filters-section {
  .filter-controls {
    display: flex;
    gap: 1rem;
    justify-content: center;
    flex-wrap: wrap;
    
    @media (max-width: 768px) {
      flex-direction: column;
      align-items: center;
    }
  }
  
  .category-filter,
  .funding-filter {
    min-width: 200px;
  }
}

.requests-list {
  max-width: 800px;
  margin: 0 auto;
}

.no-results {
  text-align: center;
  padding: 3rem 1rem;
  color: #666;
  font-size: 1.125rem;
}

.load-more-section {
  text-align: center;
  margin-top: 2rem;
}

:deep(.el-select) {
  .el-input__inner {
    border-radius: 6px;
  }
}

:deep(.el-button) {
  border-radius: 6px;
  font-weight: 500;
}
</style>
