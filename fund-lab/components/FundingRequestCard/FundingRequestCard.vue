<template>
  <div class="funding-request-card">
    <div class="card-header">
      <h3 class="funding-title">{{ request.title }}</h3>
      <div class="funding-amount">
        <span class="amount-label">Requested:</span>
        <span class="amount-value">${{ formatAmount(request.fundingAmount) }}</span>
      </div>
    </div>
    
    <div class="card-body">
      <div class="owner-info mb-16">
        <div class="owner-name">
          <strong>Principal Investigator:</strong> {{ request.ownerName }}
        </div>
        <div class="orcid-id" v-if="request.orcidId">
          <strong>ORCID:</strong> 
          <a :href="`https://orcid.org/${request.orcidId}`" target="_blank" class="orcid-link">
            {{ request.orcidId }}
          </a>
        </div>
        <div class="timespan">
          <strong>Duration:</strong> {{ request.fundingTimespan }}
        </div>
      </div>
      
      <div class="description" v-html="parseMarkdown(request.description)"></div>
    </div>
    
    <div class="card-footer">
      <el-button type="primary" class="contact-button">
        Contact Researcher
      </el-button>
      <el-button type="default" class="learn-more-button">
        Learn More
      </el-button>
    </div>
  </div>
</template>

<script>
import marked from '@/mixins/marked'

export default {
  name: 'FundingRequestCard',
  mixins: [marked],
  props: {
    request: {
      type: Object,
      required: true,
      validator(value) {
        return value && 
               typeof value.title === 'string' && 
               typeof value.ownerName === 'string' &&
               typeof value.fundingAmount === 'number' &&
               typeof value.fundingTimespan === 'string' &&
               typeof value.description === 'string'
      }
    }
  },
  methods: {
    formatAmount(amount) {
      return new Intl.NumberFormat('en-US').format(amount)
    }
  }
}
</script>

<style scoped lang="scss">
@import 'sparc-design-system-components-2/src/assets/_variables.scss';

.funding-request-card {
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

.funding-title {
  font-size: 1.25rem;
  font-weight: 600;
  color: $darkBlue;
  margin: 0;
  flex: 1;
  line-height: 1.4;
}

.funding-amount {
  text-align: right;

  @media (max-width: 768px) {
    text-align: left;
  }

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

.card-body {
  margin-bottom: 1.5rem;
}

.owner-info {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.5rem;

  @media (max-width: 768px) {
    grid-template-columns: 1fr;
  }

  .owner-name,
  .orcid-id,
  .timespan {
    font-size: 0.875rem;
    color: #333;
  }

  .orcid-link {
    color: $purple;
    text-decoration: none;
    
    &:hover {
      text-decoration: underline;
    }
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
  
  @media (max-width: 768px) {
    flex-direction: column;
  }

  .contact-button {
    background-color: $purple;
    border-color: $purple;
    
    &:hover {
      background-color: darken($purple, 10%);
      border-color: darken($purple, 10%);
    }
  }
}
</style>
