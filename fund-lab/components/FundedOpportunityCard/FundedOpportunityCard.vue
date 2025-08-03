<template>
  <div class="funded-opportunity-card">
    <div class="card-header">
      <div class="organization-info">
        <h3 class="opportunity-title">{{ opportunity.title }}</h3>
        <div class="organization-name">{{ opportunity.organizationName }}</div>
        <div class="organization-type">{{ opportunity.organizationType }}</div>
      </div>
      <div class="funding-details">
        <div class="funding-amount">
          <span class="amount-label">Available Budget:</span>
          <span class="amount-value">${{ formatAmount(opportunity.fundingAmount) }}</span>
        </div>
        <div class="position-type">{{ opportunity.positionType }}</div>
      </div>
    </div>
    
    <div class="card-body">
      <div class="opportunity-meta mb-16">
        <div class="duration">
          <strong>Duration:</strong> {{ opportunity.duration }}
        </div>
        <div class="location">
          <strong>Location:</strong> {{ opportunity.location }}
        </div>
        <div class="application-deadline" v-if="opportunity.applicationDeadline">
          <strong>Application Deadline:</strong> {{ formatDate(opportunity.applicationDeadline) }}
        </div>
        <div class="required-qualifications">
          <strong>Required Qualifications:</strong> {{ opportunity.requiredQualifications }}
        </div>
      </div>
      
      <div class="description" v-html="parseMarkdown(opportunity.description)"></div>
    </div>
    
    <div class="card-footer">
      <el-button type="primary" class="apply-button">
        Apply for Position
      </el-button>
      <el-button type="default" class="learn-more-button">
        View Details
      </el-button>
      <el-button type="text" class="contact-button">
        Contact Organization
      </el-button>
    </div>
  </div>
</template>

<script>
import marked from '@/mixins/marked'
import { format } from 'date-fns'

export default {
  name: 'FundedOpportunityCard',
  mixins: [marked],
  props: {
    opportunity: {
      type: Object,
      required: true,
      validator(value) {
        return value && 
               typeof value.title === 'string' && 
               typeof value.organizationName === 'string' &&
               typeof value.organizationType === 'string' &&
               typeof value.fundingAmount === 'number' &&
               typeof value.duration === 'string' &&
               typeof value.description === 'string'
      }
    }
  },
  methods: {
    formatAmount(amount) {
      return new Intl.NumberFormat('en-US').format(amount)
    },
    formatDate(dateString) {
      if (!dateString) return ''
      return format(new Date(dateString), 'MMM dd, yyyy')
    }
  }
}
</script>

<style scoped lang="scss">
@import 'sparc-design-system-components-2/src/assets/_variables.scss';

.funded-opportunity-card {
  background: white;
  border: 1px solid #e4e7ed;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: box-shadow 0.3s ease;

  &:hover {
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  }
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid #f0f0f0;

  @media (max-width: 768px) {
    flex-direction: column;
    gap: 1rem;
  }
}

.organization-info {
  flex: 1;

  .opportunity-title {
    font-size: 1.25rem;
    font-weight: 600;
    color: $darkBlue;
    margin: 0 0 0.5rem 0;
    line-height: 1.4;
  }

  .organization-name {
    font-size: 1rem;
    font-weight: 500;
    color: $purple;
    margin-bottom: 0.25rem;
  }

  .organization-type {
    font-size: 0.875rem;
    color: #666;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }
}

.funding-details {
  text-align: right;

  @media (max-width: 768px) {
    text-align: left;
  }

  .funding-amount {
    margin-bottom: 0.5rem;

    .amount-label {
      display: block;
      font-size: 0.875rem;
      color: #666;
      margin-bottom: 0.25rem;
    }

    .amount-value {
      font-size: 1.5rem;
      font-weight: 700;
      color: $purple;
    }
  }

  .position-type {
    font-size: 0.875rem;
    font-weight: 500;
    color: $darkBlue;
    background-color: #f0f4ff;
    padding: 0.25rem 0.75rem;
    border-radius: 12px;
    display: inline-block;
  }
}

.card-body {
  margin-bottom: 1.5rem;
}

.opportunity-meta {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.5rem;

  @media (max-width: 768px) {
    grid-template-columns: 1fr;
  }

  .duration,
  .location,
  .application-deadline,
  .required-qualifications {
    font-size: 0.875rem;
    color: #333;
  }

  .application-deadline {
    color: #e74c3c;
    font-weight: 500;
  }
}

.description {
  color: #555;
  line-height: 1.6;
  
  :deep(p) {
    margin-bottom: 0.75rem;
    
    &:last-child {
      margin-bottom: 0;
    }
  }

  :deep(ul) {
    margin-left: 1.5rem;
    margin-bottom: 0.75rem;
  }

  :deep(strong) {
    color: #333;
  }
}

.card-footer {
  display: flex;
  gap: 1rem;
  align-items: center;
  
  @media (max-width: 768px) {
    flex-direction: column;
    align-items: stretch;
  }

  .apply-button {
    background-color: $purple;
    border-color: $purple;
    
    &:hover {
      background-color: darken($purple, 10%);
      border-color: darken($purple, 10%);
    }
  }

  .contact-button {
    color: $purple;
    
    &:hover {
      color: darken($purple, 10%);
    }
  }
}
</style>
