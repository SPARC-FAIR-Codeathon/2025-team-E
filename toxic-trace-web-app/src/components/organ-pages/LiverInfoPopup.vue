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

      <h3>♻️ How the Liver Processes BPA</h3>

      <div class="image-container">
        <img 
          src="/organ-system-images/Liver - BPA Detox.png" 
          alt="Liver BPA Detox Process" 
          class="section-image"
        />
      </div>

      <h4>1. Absorption & Entry</h4>
      <ul>
        <li>After ingesting food or drink containing BPA (e.g., from cans or plastic), BPA is absorbed in the <strong>small intestine</strong></li>
        <li>Enters the liver via the <strong>hepatic portal vein</strong></li>
      </ul>

      <h4>2. Detoxification (Phase II)</h4>
      
      <div class="image-container">
        <img 
          src="/organ-system-images/Liver - BPA Metabolism.png" 
          alt="Liver BPA Metabolism Pathways" 
          class="section-image"
        />
      </div>

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

      <h3>🔗 BPA and Estrogen Receptor Binding</h3>
      
      <div class="image-container">
        <img 
          src="/organ-system-images/Liver-BPA-Estrogen.png" 
          alt="BPA and Estrogen Receptor Interactions" 
          class="section-image"
        />
      </div>
      
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
  color: #2c5aa0;
  margin-top: 24px;
  margin-bottom: 16px;
}

.liver-content h2 {
  border-bottom: 2px solid #2c5aa0;
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
  color: #2c5aa0;
  font-weight: 600;
}

.liver-content em {
  color: #666;
  font-style: italic;
}

.liver-content p {
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
