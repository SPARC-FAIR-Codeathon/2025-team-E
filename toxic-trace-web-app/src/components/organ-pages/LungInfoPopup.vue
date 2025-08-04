<template>
  <el-dialog
    v-model="visible"
    title="🌬️ The Lungs: Your Respiratory Interface"
    width="800px"
    :close-on-click-modal="false"
    class="lung-info-dialog"
  >
    <div class="lung-content">
      <h2>🌬️ The Lungs: Your Respiratory Interface</h2>
      
      <p>The lungs are essential for gas exchange, delivering oxygen to the bloodstream and removing carbon dioxide. Beyond this, they also act as a frontline filter for airborne particles, chemicals, and pathogens.</p>
      
      <p>They contain metabolically active cells and enzymes (like <strong>CYP450s</strong>, <strong>monooxygenases</strong>, and <strong>glutathione S-transferases</strong>) that respond to inhaled or circulating toxins—including BPA.</p>

      <h3>❗ How the Lungs Get Compromised</h3>
      
      <p>Lung detox pathways can be weakened by:</p>

      <h4>🛑 Airborne Toxicants</h4>
      
      <div class="image-container">
        <img 
          src="/organ-system-images/Lungs - BPA IntakePM.png" 
          alt="BPA Intake via Particulate Matter in Lungs" 
          class="section-image"
        />
      </div>
      
      <ul>
        <li>Inhalation of <strong>BPA-laden dust</strong>, <strong>microplastics</strong>, and indoor air pollution</li>
        <li><strong>Occupational exposure</strong> (e.g., factory, packaging, plastics)</li>
      </ul>

      <h4>🧬 Inflammatory Overload</h4>
      <ul>
        <li>BPA can trigger <strong>oxidative stress</strong>, leading to chronic inflammation, especially in airways</li>
        <li>Alters immune responses, increasing sensitivity to allergens and infections</li>
      </ul>

      <h4>🧒 Developmental Sensitivity</h4>
      <ul>
        <li><strong>Infants and children</strong> breathe faster, increasing dose per body weight</li>
        <li>Lung tissue and immune defenses are still maturing</li>
      </ul>

      <h4>🫁 Surfactant Disruption</h4>
      <ul>
        <li>BPA interferes with <strong>surfactant proteins</strong>, impairing lung elasticity and gas exchange</li>
      </ul>

      <p>
        🧪 <a href="https://pubmed.ncbi.nlm.nih.gov/28302478/" target="_blank" rel="noopener noreferrer">
          Study: BPA inhalation triggers pulmonary inflammation (Liu et al., 2017)
        </a>
      </p>

      <h3>♻️ BPA in the Lungs — Entry & Elimination</h3>

      <div class="image-container">
        <img 
          src="/organ-system-images/Lung - Meabolism.png" 
          alt="Lung Metabolism and BPA Processing" 
          class="section-image"
        />
      </div>

      <h4>1. Inhalation & Absorption</h4>
      <ul>
        <li>BPA enters via <strong>inhaled air bound to dust</strong> or aerosolized plastic particles</li>
        <li>Can be absorbed directly by <strong>bronchial epithelium</strong> into circulation</li>
      </ul>

      <h4>2. Local Effects & Clearance</h4>
      <ul>
        <li>BPA accumulates in lung tissue, promoting <strong>oxidative stress</strong> and <strong>fibrosis</strong></li>
        <li>Detoxification via <strong>CYP1A1</strong>, <strong>GSTs</strong>, and <strong>peroxidases</strong> in lung cells</li>
        <li>Partial clearance through <strong>mucociliary elevator</strong> and <strong>macrophage engulfment</strong></li>
      </ul>

      <p>
        🧪 <a href="https://pubmed.ncbi.nlm.nih.gov/29247866/" target="_blank" rel="noopener noreferrer">
          Study: BPA exposure affects lung antioxidant defenses (Chen et al., 2018)
        </a>
      </p>

      <h3>⚠️ What Happens if BPA Persists in Lung Tissue?</h3>
      
      <p>If detox fails or exposure is chronic:</p>
      <ul>
        <li>BPA binds to receptors in <strong>airway and alveolar cells</strong></li>
        <li>Induces <strong>inflammatory gene expression</strong> (e.g., IL-6, TNF-α)</li>
        <li>Weakens <strong>barrier integrity</strong>, increasing vulnerability to airborne toxins</li>
      </ul>

      <p>
        🧪 <a href="https://pubmed.ncbi.nlm.nih.gov/32505661/" target="_blank" rel="noopener noreferrer">
          Study: BPA compromises lung barrier and immune balance (Dong et al., 2020)
        </a>
      </p>

      <h3>🔗 BPA and Estrogen Receptors in the Lungs</h3>
      
      <div class="image-container">
        <img 
          src="/organ-system-images/Lungs-BPA-Estrogen.png" 
          alt="BPA and Estrogen Receptor Interactions in Lungs" 
          class="section-image"
        />
      </div>
      
      <p>Lung cells express <strong>ERα</strong>, <strong>ERβ</strong>, and <strong>GPER</strong>, making them responsive to xenoestrogens like BPA:</p>
      <ul>
        <li>Alters <strong>airway remodeling</strong> and <strong>fibrosis</strong></li>
        <li>Influences <strong>asthma severity</strong>, especially in women and children</li>
        <li><strong>Estrogenic pathways</strong> amplify inflammatory responses</li>
      </ul>

      <h4>🚨 Why This Matters</h4>
      <ul>
        <li>BPA can act like <strong>hormonal fuel</strong> in lung tissue, promoting chronic lung issues</li>
        <li>Linked to higher rates of <strong>asthma</strong>, <strong>wheezing</strong>, and <strong>pulmonary hypersensitivity</strong></li>
      </ul>

      <p>
        🧪 <a href="https://pubmed.ncbi.nlm.nih.gov/17507688/" target="_blank" rel="noopener noreferrer">
          Study: Estrogen signaling in lung disease (Carey et al., 2007)
        </a>
      </p>

      <h3>🧠 Pro Tip (Researcher View)</h3>
      <ul>
        <li><strong>Lung-specific CYP1A1</strong> expression rises with BPA exposure</li>
        <li>BPA-induced airway inflammation often <strong>precedes observable lung damage</strong></li>
        <li><strong>GPER activation</strong> in lung epithelial cells linked to IL-8 overproduction</li>
        <li>BPA + allergens can <strong>synergistically worsen asthma</strong></li>
      </ul>
    </div>
    
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="closeDialog" type="primary">
          Close
        </el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script>
export default {
  name: 'LungInfoPopup',
  props: {
    showDialog: {
      type: Boolean,
      default: false
    }
  },
  emits: ['close'],
  computed: {
    visible: {
      get() {
        return this.showDialog
      },
      set(value) {
        if (!value) {
          this.$emit('close')
        }
      }
    }
  },
  methods: {
    closeDialog() {
      this.$emit('close')
    }
  }
}
</script>

<style scoped>
.lung-info-dialog {
  --el-dialog-border-radius: 12px;
}

.lung-content {
  max-height: 600px;
  overflow-y: auto;
  padding: 0 20px;
  line-height: 1.6;
}

.lung-content h2,
.lung-content h3,
.lung-content h4 {
  color: #7c3aed;
  margin-top: 24px;
  margin-bottom: 16px;
}

.lung-content h2 {
  border-bottom: 2px solid #7c3aed;
  padding-bottom: 8px;
}

.lung-content h3 {
  border-bottom: 1px solid #e1e1e1;
  padding-bottom: 4px;
}

.lung-content ul {
  margin: 16px 0;
  padding-left: 20px;
}

.lung-content li {
  margin: 8px 0;
}

.lung-content strong {
  color: #7c3aed;
  font-weight: 600;
}

.lung-content em {
  color: #666;
  font-style: italic;
}

.lung-content p {
  margin: 12px 0;
}

.image-container {
  text-align: center;
  margin: 20px 0;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 8px;
  border: 1px solid #e9ecef;
  min-height: 100px;
}

.section-image {
  max-width: 100%;
  height: auto;
  border-radius: 6px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
  display: block;
  margin: 0 auto;
}

.section-image:hover {
  transform: scale(1.02);
}

.lung-content a {
  color: #7c3aed;
  text-decoration: none;
  font-weight: 500;
  border-bottom: 1px solid transparent;
  transition: all 0.3s ease;
}

.lung-content a:hover {
  color: #5b21b6;
  border-bottom-color: #7c3aed;
  text-decoration: none;
}

.lung-content a:visited {
  color: #8b5cf6;
}

/* Scrollbar styling */
.lung-content::-webkit-scrollbar {
  width: 8px;
}

.lung-content::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 4px;
}

.lung-content::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 4px;
}

.lung-content::-webkit-scrollbar-thumb:hover {
  background: #a1a1a1;
}
</style>
