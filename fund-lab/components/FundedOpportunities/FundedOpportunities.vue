<template>
  <div class="funded-opportunities">
    <div class="section-header mb-24">
      <h2 class="section-title">Funded Research Opportunities</h2>
      <p class="section-description">
        Explore funded research positions from leading institutions and companies. 
        These organizations have secured funding and are actively seeking qualified researchers.
      </p>
    </div>
    
    <div class="filters-section mb-24">
      <div class="filter-controls">
        <el-select v-model="selectedOrganizationType" placeholder="Organization Type" clearable class="org-type-filter">
          <el-option label="All Organizations" value="" />
          <el-option label="University" value="university" />
          <el-option label="Government Agency" value="government" />
          <el-option label="Private Company" value="company" />
          <el-option label="Research Institute" value="institute" />
          <el-option label="Non-Profit" value="nonprofit" />
        </el-select>
        
        <el-select v-model="selectedPositionType" placeholder="Position Type" clearable class="position-filter">
          <el-option label="All Positions" value="" />
          <el-option label="Postdoctoral Fellow" value="postdoc" />
          <el-option label="Research Scientist" value="research-scientist" />
          <el-option label="Principal Investigator" value="pi" />
          <el-option label="Research Associate" value="research-associate" />
          <el-option label="Graduate Student" value="graduate" />
        </el-select>
        
        <el-select v-model="selectedFundingRange" placeholder="Budget Range" clearable class="funding-filter">
          <el-option label="All Budgets" value="" />
          <el-option label="Under $100K" value="under-100k" />
          <el-option label="$100K - $250K" value="100k-250k" />
          <el-option label="$250K - $500K" value="250k-500k" />
          <el-option label="Over $500K" value="over-500k" />
        </el-select>
      </div>
    </div>

    <div class="opportunities-list">
      <funded-opportunity-card 
        v-for="opportunity in filteredOpportunities" 
        :key="opportunity.id"
        :opportunity="opportunity"
      />
    </div>

    <div v-if="filteredOpportunities.length === 0" class="no-results">
      <p>No funded opportunities match your current filters.</p>
    </div>

    <div class="load-more-section" v-if="showLoadMore">
      <el-button type="default" @click="loadMoreOpportunities">
        Load More Opportunities
      </el-button>
    </div>
  </div>
</template>

<script>
import FundedOpportunityCard from '@/components/FundedOpportunityCard/FundedOpportunityCard.vue'

export default {
  name: 'FundedOpportunities',
  components: {
    FundedOpportunityCard
  },
  data() {
    return {
      selectedOrganizationType: '',
      selectedPositionType: '',
      selectedFundingRange: '',
      showLoadMore: true,
      fundedOpportunities: [
        {
          id: 1,
          title: "AI-Driven Biomarker Discovery for Neurological Disorders",
          organizationName: "Stanford University Medical Center",
          organizationType: "university",
          fundingAmount: 350000,
          duration: "3 years",
          location: "Stanford, CA (Hybrid)",
          positionType: "postdoc",
          requiredQualifications: "PhD in Computational Biology, Machine Learning, or related field",
          applicationDeadline: "2025-09-15",
          description: `**Position Overview:** Join our cutting-edge research team developing AI algorithms for early detection of neurological disorders using multi-omics data.

**Key Responsibilities:**
- Develop machine learning models for biomarker identification
- Collaborate with clinical teams to validate findings
- Publish research in high-impact journals
- Mentor graduate students

**What We Offer:**
- Competitive salary and benefits
- Access to state-of-the-art computing resources
- Collaborative research environment
- Career development opportunities

**Required Skills:** Python/R programming, deep learning frameworks (TensorFlow/PyTorch), statistical analysis, data visualization.`
        },
        {
          id: 2,
          title: "CRISPR Gene Therapy Development for Rare Diseases",
          organizationName: "Broad Institute",
          organizationType: "institute",
          fundingAmount: 275000,
          duration: "2 years",
          location: "Cambridge, MA",
          positionType: "research-scientist",
          requiredQualifications: "PhD in Molecular Biology, Genetics, or Bioengineering with 3+ years experience",
          applicationDeadline: "2025-08-30",
          description: `**Research Focus:** Lead the development of next-generation CRISPR-based therapeutics for treating rare genetic disorders.

**Project Goals:**
- Engineer improved CRISPR delivery systems
- Conduct preclinical efficacy and safety studies
- Advance therapies toward clinical trials

**Ideal Candidate:**
- Expertise in gene editing technologies
- Experience with in vivo models
- Strong publication record
- Excellent communication skills

**Benefits Package:** Comprehensive health insurance, retirement plan, flexible work arrangements, professional development budget.`
        },
        {
          id: 3,
          title: "Quantum Sensing for Medical Diagnostics",
          organizationName: "IBM Research",
          organizationType: "company",
          fundingAmount: 180000,
          duration: "18 months",
          location: "Yorktown Heights, NY",
          positionType: "research-associate",
          requiredQualifications: "MS/PhD in Physics, Quantum Computing, or related field",
          applicationDeadline: "2025-09-01",
          description: `**Innovation Opportunity:** Develop quantum sensing technologies for ultra-sensitive medical diagnostic applications.

**Research Areas:**
- Quantum magnetometry for cardiac monitoring
- Single-molecule detection systems
- Integration with existing medical devices

**Technical Requirements:**
- Background in quantum mechanics and sensing
- Experience with laboratory instrumentation
- Programming skills (Python, MATLAB, C++)
- Data analysis and modeling capabilities

**Company Benefits:** Stock options, health/dental/vision coverage, on-site fitness center, continuing education support.`
        },
        {
          id: 4,
          title: "Microbiome Therapeutics Research Program",
          organizationName: "National Institutes of Health",
          organizationType: "government",
          fundingAmount: 420000,
          duration: "4 years",
          location: "Bethesda, MD",
          positionType: "pi",
          requiredQualifications: "PhD/MD with 7+ years research experience and PI track record",
          applicationDeadline: "2025-10-15",
          description: `**Program Leadership:** Direct a multi-disciplinary research program investigating microbiome-based therapeutics for inflammatory diseases.

**Responsibilities:**
- Lead a team of 8-12 researchers
- Coordinate with clinical collaborators
- Oversee budget and project management
- Drive translation from bench to bedside

**Research Priorities:**
- Identify therapeutic microbiome targets
- Develop novel delivery mechanisms
- Conduct Phase I/II clinical trials

**Federal Benefits:** Competitive salary, excellent health benefits, retirement savings plan, professional development opportunities, job security.`
        },
        {
          id: 5,
          title: "Bioengineered Organs and Tissue Regeneration",
          organizationName: "Organovo Holdings Inc.",
          organizationType: "company",
          fundingAmount: 220000,
          duration: "2.5 years",
          location: "San Diego, CA",
          positionType: "research-scientist",
          requiredQualifications: "PhD in Bioengineering, Tissue Engineering, or related field",
          applicationDeadline: "2025-08-25",
          description: `**Breakthrough Technology:** Lead development of 3D bioprinted tissues and organs for therapeutic applications.

**Key Projects:**
- Engineer functional liver tissue models
- Develop vascularized tissue constructs
- Optimize bioprinting protocols and materials

**Technical Expertise Needed:**
- 3D bioprinting and tissue engineering
- Cell culture and stem cell biology
- Biomaterials science
- Regulatory pathway knowledge

**Startup Environment:** Equity participation, flexible PTO, health benefits, innovation-focused culture, direct impact on product development.`
        },
        {
          id: 6,
          title: "Digital Health and Wearable Technology Research",
          organizationName: "Scripps Research Institute",
          organizationType: "institute",
          fundingAmount: 195000,
          duration: "2 years",
          location: "La Jolla, CA",
          positionType: "postdoc",
          requiredQualifications: "PhD in Computer Science, Biomedical Engineering, or related field",
          applicationDeadline: "2025-09-10",
          description: `**Digital Medicine Focus:** Develop next-generation wearable devices and AI algorithms for continuous health monitoring.

**Research Objectives:**
- Create novel biosensing technologies
- Develop predictive health analytics
- Validate devices in clinical settings
- Advance precision medicine applications

**Skills Required:**
- Signal processing and machine learning
- Mobile health application development
- Biosensor design and validation
- Clinical study design

**Research Environment:** Access to clinical data, industry partnerships, international collaborations, publication and patent opportunities.`
        }
      ]
    }
  },
  computed: {
    filteredOpportunities() {
      let filtered = this.fundedOpportunities

      if (this.selectedOrganizationType) {
        filtered = filtered.filter(opp => opp.organizationType === this.selectedOrganizationType)
      }

      if (this.selectedPositionType) {
        filtered = filtered.filter(opp => opp.positionType === this.selectedPositionType)
      }

      if (this.selectedFundingRange) {
        filtered = filtered.filter(opp => {
          const amount = opp.fundingAmount
          switch (this.selectedFundingRange) {
            case 'under-100k':
              return amount < 100000
            case '100k-250k':
              return amount >= 100000 && amount <= 250000
            case '250k-500k':
              return amount > 250000 && amount <= 500000
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
    loadMoreOpportunities() {
      // Placeholder for loading more opportunities from API
      this.showLoadMore = false
    }
  }
}
</script>

<style scoped lang="scss">
@import 'sparc-design-system-components-2/src/assets/_variables.scss';

.funded-opportunities {
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
  
  .org-type-filter,
  .position-filter,
  .funding-filter {
    min-width: 200px;
  }
}

.opportunities-list {
  max-width: 900px;
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
