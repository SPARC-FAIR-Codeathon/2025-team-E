<template>
  <Head>
    <Title>{{ pageTitle }}</Title>
    <Meta name="og:title" hid="og:title" :content="pageTitle" />
    <Meta name="twitter:title" :content="pageTitle" />
    <Meta name="description" hid="description" :content="pageDescription" />
    <Meta name="og:description" hid="og:description" :content="pageDescription" />
    <Meta name="twitter:description" :content="pageDescription" />
  </Head>
  <div class="fundlab-page pb-16">
    <Breadcrumb :breadcrumb="breadcrumb" title="FundLab" />
    <page-hero class="py-24">
      <h1>{{ pageTitle }}</h1>
      <p>{{ pageDescription }}</p>
    </page-hero>
    <div class="container">
      <div class="content-section p-24 mt-32">
        <div class="heading2 mb-16">Welcome to FundLab</div>
        <p class="mb-16">
          Discover funding opportunities and connect with innovative research projects in the SPARC ecosystem. 
          Our platform facilitates collaboration between researchers seeking funding and institutions with funded positions.
        </p>
        <p class="mb-16">
          Choose your view below to explore either funded research positions or funding requests from researchers.
        </p>
        
        <!-- Toggle Buttons -->
        <div class="view-toggle-section">
          <div class="toggle-buttons">
            <button 
              :class="['toggle-btn', { active: activeView === 'opportunities' }]"
              @click="setActiveView('opportunities')"
            >
              <div class="btn-icon">🏢</div>
              <div class="btn-content">
                <span>Funded Positions</span>
                <span class="btn-subtitle">Find your next role</span>
              </div>
            </button>
            <button 
              :class="['toggle-btn', { active: activeView === 'requests' }]"
              @click="setActiveView('requests')"
            >
              <div class="btn-icon">💡</div>
              <div class="btn-content">
                <span>Funding Requests</span>
                <span class="btn-subtitle">Support research</span>
              </div>
            </button>
          </div>
        </div>
      </div>

      <!-- Funded Opportunities Section -->
      <div v-if="activeView === 'opportunities'" class="funded-opportunities-section mt-32">
        <funded-opportunities />
      </div>

      <!-- Funding Requests Section -->
      <div v-if="activeView === 'requests'" class="funding-requests-section mt-32">
        <funding-requests />
      </div>

      <div class="additional-resources p-24 mt-32">
        <div class="heading2 mb-16">Additional Resources</div>
        <div class="coming-soon-notice p-16">
          <h3>Coming Soon</h3>
          <ul>
            <li>Grant application templates and guides</li>
            <li>Funding opportunity alerts</li>
            <li>Partnership matching system</li>
            <li>Research collaboration tools</li>
            <li>Success stories and case studies</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import FundingRequests from '@/components/FundingRequests/FundingRequests.vue'
import FundedOpportunities from '@/components/FundedOpportunities/FundedOpportunities.vue'

export default {
  name: 'FundLabPage',
  components: {
    FundingRequests,
    FundedOpportunities
  },
  data() {
    return {
      pageTitle: 'FundLab',
      pageDescription: 'Discover funding opportunities and resources for biomedical research and innovation.',
      activeView: 'opportunities', // Default to showing funded opportunities
      breadcrumb: [
        {
          to: {
            name: 'index'
          },
          label: 'Home'
        },
        {
          label: 'FundLab'
        }
      ]
    }
  },
  methods: {
    setActiveView(view) {
      this.activeView = view
    }
  },
  head() {
    return {
      title: this.pageTitle,
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: this.pageDescription
        },
        {
          hid: 'og:description',
          property: 'og:description',
          content: this.pageDescription
        },
        {
          hid: 'twitter:description',
          name: 'twitter:description',
          content: this.pageDescription
        }
      ]
    }
  }
}
</script>

<style scoped lang="scss">
@import 'sparc-design-system-components-2/src/assets/_variables.scss';

.fundlab-page {
  min-height: 70vh;
}

.content-section {
  background-color: #f9f9f9;
  border-radius: 8px;
}

.view-toggle-section {
  margin-top: 2rem;
  margin-bottom: 1rem;

  .toggle-buttons {
    display: flex;
    gap: 1rem;
    justify-content: center;
    flex-wrap: wrap;

    @media (max-width: 768px) {
      flex-direction: column;
      align-items: center;
    }
  }

  .toggle-btn {
    display: flex;
    align-items: center;
    padding: 1.5rem 2rem;
    background: white;
    border: 2px solid #e4e7ed;
    border-radius: 12px;
    cursor: pointer;
    transition: all 0.3s ease;
    min-width: 200px;
    text-align: left;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    gap: 1rem;

    &:hover {
      border-color: $purple;
      transform: translateY(-2px);
      box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
    }

    &.active {
      border-color: $purple;
      background: linear-gradient(135deg, $purple, darken($purple, 10%));
      color: white;
      
      .btn-icon {
        background-color: rgba(255, 255, 255, 0.2);
      }
      
      .btn-subtitle {
        color: rgba(255, 255, 255, 0.9);
      }
    }

    .btn-icon {
      font-size: 1.5rem;
      background-color: #f0f4ff;
      border-radius: 8px;
      padding: 0.5rem;
      min-width: 3rem;
      min-height: 3rem;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: background-color 0.3s ease;
    }

    .btn-content {
      display: flex;
      flex-direction: column;
      flex: 1;

      span:first-child {
        font-size: 1.125rem;
        font-weight: 600;
        margin-bottom: 0.25rem;
      }

      .btn-subtitle {
        font-size: 0.875rem;
        color: #666;
        font-weight: 400;
        transition: color 0.3s ease;
      }
    }

    @media (max-width: 768px) {
      min-width: 250px;
      justify-content: center;
    }
  }
}

.additional-resources {
  background-color: #f9f9f9;
  border-radius: 8px;
}

.funding-requests-section {
  background-color: white;
  border-radius: 8px;
  padding: 2rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.funded-opportunities-section {
  background-color: white;
  border-radius: 8px;
  padding: 2rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.heading2 {
  font-size: 1.5rem;
  font-weight: 600;
  color: $darkBlue;
  margin-bottom: 1rem;
}

.coming-soon-notice {
  background-color: #e8f4fd;
  border: 1px solid #b3d9ff;
  border-radius: 8px;
  
  h3 {
    color: $darkBlue;
    margin-bottom: 1rem;
    font-weight: 600;
  }
  
  ul {
    list-style-type: disc;
    margin-left: 1.5rem;
    
    li {
      margin-bottom: 0.5rem;
      color: #333;
    }
  }
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 2rem;
}

@media (max-width: 768px) {
  .container {
    padding: 0 1rem;
  }
  
  .content-section {
    padding: 1rem;
  }
}
</style>
