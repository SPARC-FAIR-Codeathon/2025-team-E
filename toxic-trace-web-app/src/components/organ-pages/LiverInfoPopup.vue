<template>
  <el-dialog
    v-model="visible"
    title="🏥 The Liver: Your Body's Detox Powerhouse"
    width="800px"
    :close-on-click-modal="false"
    class="liver-info-dialog"
  >
    <div class="liver-content">
      <h2>🏥 The Liver: Your Body's Detox Powerhouse</h2>
      
      <p>The liver is a vital organ responsible for:</p>
      <ul>
        <li>Filtering blood coming from the digestive tract</li>
        <li>Breaking down toxins, drugs, and hormones</li>
        <li>Producing bile to aid digestion</li>
        <li>Regulating glucose, cholesterol, and nutrient levels</li>
      </ul>

      <p>It hosts powerful enzyme systems like <strong>cytochrome P450</strong>, <strong>UGTs (UDP-glucuronosyltransferases)</strong>, and <strong>SULTs (sulfotransferases)</strong> which modify and neutralize foreign chemicals.</p>

      <h3>❗ Why the Liver Gets Overloaded</h3>
      
      <p>The liver's detox system can be <strong>overwhelmed</strong> or <strong>impaired</strong> due to:</p>

      <h4>🛑 Excessive Toxin Load</h4>
      <ul>
        <li><strong>Chronic exposure</strong> to chemicals like BPA, phthalates, alcohol, drugs, and pesticides</li>
        <li>Too many substrates competing for the <strong>same detox enzymes</strong> (e.g., UGTs)</li>
      </ul>

      <h4>🧬 Genetic Variants</h4>
      <ul>
        <li>Some people have <strong>polymorphisms in detox genes</strong> (like <em>UGT2B15</em>, <em>SULT1A1</em>) that reduce their liver's ability to process toxins efficiently</li>
      </ul>

      <h4>🧒 Developmental Immaturity</h4>
      <ul>
        <li><strong>Infants and young children</strong> have underdeveloped liver enzyme systems</li>
        <li>UGT activity is only ~10% of adult levels in newborns</li>
      </ul>

      <h4>🦠 Gut–Liver Axis Disruption</h4>
      <ul>
        <li><strong>Leaky gut</strong> and <strong>microbiome imbalances</strong> can increase the flow of toxic byproducts into the liver</li>
      </ul>

      <h4>🥴 Poor Nutrition & Lifestyle</h4>
      <ul>
        <li>Deficiencies in detox cofactors (e.g., B vitamins, magnesium, glutathione)</li>
        <li>Fatty liver disease, high sugar intake, and sedentary behavior reduce liver resilience</li>
      </ul>

      <p>
        🧪 <a href="https://pubmed.ncbi.nlm.nih.gov/20020268/" target="_blank" rel="noopener noreferrer">
          Review: Environmental Chemicals and Liver Health (Cave et al., 2010)
        </a>
      </p>

      <h3>♻️ How the Liver Processes BPA</h3>

      <h4>1. Absorption & Entry</h4>
      <ul>
        <li>After ingesting food or drink containing BPA (e.g., from cans or plastic), BPA is absorbed in the <strong>small intestine</strong></li>
        <li>Enters the liver via the <strong>hepatic portal vein</strong></li>
      </ul>

      <h4>2. Detoxification (Phase II)</h4>
      <ul>
        <li>BPA is primarily processed through <strong>glucuronidation</strong> and <strong>sulfation</strong>:
          <ul>
            <li><strong>BPA-glucuronide</strong> (via UGT enzymes)</li>
            <li><strong>BPA-sulfate</strong> (via SULT enzymes)</li>
          </ul>
        </li>
        <li>These conjugated forms are <strong>water-soluble</strong>, <strong>biologically inactive</strong>, and eliminated via:
          <ul>
            <li><strong>Urine</strong> (renal excretion)</li>
            <li><strong>Bile</strong> (fecal excretion)</li>
          </ul>
        </li>
      </ul>

      <p>
        🧪 <a href="https://pubmed.ncbi.nlm.nih.gov/15800095/" target="_blank" rel="noopener noreferrer">
          Study: BPA Metabolism via Glucuronidation in Humans (Teeguarden et al., 2005)
        </a>
      </p>

      <h3>⚠️ What Happens to BPA That Isn't Filtered?</h3>
      
      <p>If liver detox pathways are <strong>overwhelmed</strong> or <strong>impaired</strong>:</p>
      <ul>
        <li>Some BPA remains in its <strong>free (active)</strong> form</li>
        <li>It enters <strong>systemic circulation</strong>, where it:
          <ul>
            <li>Binds to hormone receptors</li>
            <li><strong>Accumulates in fat tissue</strong></li>
            <li>May cross the <strong>blood–brain barrier</strong></li>
          </ul>
        </li>
        <li>Infants, young children, and people with <strong>low UGT activity</strong> have slower BPA clearance</li>
      </ul>

      <p>
        🧪 <a href="https://pubmed.ncbi.nlm.nih.gov/19244986/" target="_blank" rel="noopener noreferrer">
          Study: Reduced BPA Clearance in Neonates (Edginton & Ritter, 2009)
        </a>
      </p>

      <h3>🔗 BPA and Estrogen Receptor Binding</h3>
      
      <p>BPA is a <strong>xenoestrogen</strong> — it mimics natural estrogen by binding to:</p>
      <ul>
        <li><strong>ERα and ERβ (classical estrogen receptors)</strong></li>
        <li><strong>GPER (G protein-coupled estrogen receptor)</strong></li>
      </ul>

      <h4>Where Does This Happen?</h4>
      <ul>
        <li><strong>Liver</strong>: Alters fat/cholesterol metabolism, contributes to fatty liver</li>
        <li><strong>Brain</strong>: Disrupts neurodevelopment and emotional regulation</li>
        <li><strong>Reproductive organs</strong>: Affects breast, testis, uterine tissue</li>
      </ul>

      <h4>🚨 Why It's Harmful When BPA Mimics Estrogen</h4>
      <ul>
        <li>Estrogen regulates <strong>growth, mood, metabolism, fertility, and brain development</strong></li>
        <li>BPA binding <strong>distorts hormonal timing and intensity</strong>, triggering biological responses at the wrong time or place</li>
        <li>In sensitive tissues, even low-dose BPA can promote <strong>tumor growth</strong>, <strong>reproductive disorders</strong>, and <strong>metabolic disruption</strong></li>
        <li>Unlike natural estrogen, BPA is <strong>not properly regulated by feedback loops</strong>, leading to persistent overstimulation</li>
      </ul>

      <p>
        🧪 <a href="https://pubmed.ncbi.nlm.nih.gov/25460323/" target="_blank" rel="noopener noreferrer">
          Study: BPA Activates Estrogen Receptors (Acconcia et al., 2015)
        </a> 
        🧪 <a href="https://pubmed.ncbi.nlm.nih.gov/17890788/" target="_blank" rel="noopener noreferrer">
          Study: GPER Activation by BPA in Liver (Wetherill et al., 2007)
        </a>
      </p>

      <h3>🧠 Pro Tip (Researcher View)</h3>
      <ul>
        <li><strong>UGT2B15 and UGT2B7</strong> are the main enzymes for BPA glucuronidation</li>
        <li>Adult BPA-glucuronide <strong>half-life</strong>: ~6 hours</li>
        <li>Neonatal UGT activity is &lt;10% of adult capacity</li>
        <li>BPA also triggers <strong>non-genomic estrogenic pathways</strong> via GPER in <strong>liver</strong>, <strong>heart</strong>, and <strong>brain</strong></li>
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
  name: 'LiverInfoPopup',
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
.liver-info-dialog {
  --el-dialog-border-radius: 12px;
}

.liver-content {
  max-height: 600px;
  overflow-y: auto;
  padding: 0 20px;
  line-height: 1.6;
}

.liver-content h2,
.liver-content h3,
.liver-content h4 {
  color: #7c3aed;
  margin-top: 24px;
  margin-bottom: 16px;
}

.liver-content h2 {
  border-bottom: 2px solid #7c3aed;
  padding-bottom: 8px;
}

.liver-content h3 {
  border-bottom: 1px solid #e1e1e1;
  padding-bottom: 4px;
}

.liver-content ul {
  margin: 16px 0;
  padding-left: 20px;
}

.liver-content li {
  margin: 8px 0;
}

.liver-content strong {
  color: #7c3aed;
  font-weight: 600;
}

.liver-content em {
  color: #666;
  font-style: italic;
}

.liver-content p {
  margin: 12px 0;
}

.liver-content a {
  color: #7c3aed;
  text-decoration: none;
  font-weight: 500;
  border-bottom: 1px solid transparent;
  transition: all 0.3s ease;
}

.liver-content a:hover {
  color: #5b21b6;
  border-bottom-color: #7c3aed;
  text-decoration: none;
}

.liver-content a:visited {
  color: #8b5cf6;
}

/* Scrollbar styling */
.liver-content::-webkit-scrollbar {
  width: 8px;
}

.liver-content::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 4px;
}

.liver-content::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 4px;
}

.liver-content::-webkit-scrollbar-thumb:hover {
  background: #a1a1a1;
}
</style>
