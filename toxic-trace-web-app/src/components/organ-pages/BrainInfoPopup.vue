<template>
  <el-dialog
    v-model="visible"
    title="🧠 The Brain: Your Neural Command Center"
    width="800px"
    :close-on-click-modal="false"
    class="brain-info-dialog"
  >
    <div class="brain-content">
      <h2>🧠 The Brain: Your Neural Command Center</h2>
      
      <p>The brain manages memory, mood, and all bodily coordination. It's protected by the <strong>blood-brain barrier (BBB)</strong>, but BPA can breach this defense. Its hormone-disrupting effects reach deep into neural tissue—especially during development.</p>

      <h3>❗ How the Brain Gets Compromised</h3>

      <h4>🧪 BPA in the Brain</h4>
      
      <div class="image-container">
        <img 
          src="/organ-system-images/Brain - BPA.png" 
          alt="BPA in Brain Tissue and Neural Effects" 
          class="section-image"
        />
      </div>
      
      <ul>
        <li>BPA crosses the <strong>blood-brain barrier</strong> and is found in brain tissue and cerebrospinal fluid</li>
        <li>Binds to <strong>estrogen</strong>, <strong>thyroid</strong>, and <strong>androgen receptors</strong> in neurons and glial cells</li>
      </ul>

      <p>
        🧪 <a href="https://doi.org/10.1016/j.toxlet.2005.08.010" target="_blank" rel="noopener noreferrer">
          Study: BPA detected in fetal and adult brain tissue (Nakamura et al., 2006)
        </a>
      </p>

      <h4>🧬 Neuroinflammation & Oxidative Stress</h4>
      <ul>
        <li>Activates <strong>microglia</strong> (brain immune cells), increasing ROS and inflammation</li>
        <li>Leads to <strong>DNA damage</strong> and <strong>neuron degeneration</strong></li>
      </ul>

      <p>
        🧪 <a href="https://doi.org/10.1016/j.tox.2009.12.001" target="_blank" rel="noopener noreferrer">
          Study: BPA increases IL-1β and TNF-α in hippocampus (Xu et al., 2010)
        </a>
      </p>

      <h4>👶 Developmental Sensitivity</h4>
      <ul>
        <li><strong>Fetal and infant brains</strong> are highly sensitive to hormonal interference</li>
        <li>BPA disrupts <strong>neurodevelopment</strong> and <strong>neurotransmitter wiring</strong></li>
      </ul>

      <p>
        🧪 <a href="https://doi.org/10.1289/ehp.1205827" target="_blank" rel="noopener noreferrer">
          Study: Prenatal BPA linked to impaired executive function (Harley et al., 2013)
        </a>
      </p>

      <h4>🧠 Hormonal Hijacking</h4>
      <ul>
        <li>BPA mimics estrogen, altering the <strong>HPA axis</strong> and neurotransmitter systems</li>
        <li>Impacts <strong>dopamine</strong>, <strong>serotonin</strong>, and <strong>GABA</strong> balance</li>
      </ul>

      <p>
        🧪 <a href="https://doi.org/10.1016/j.neuropharm.2010.10.017" target="_blank" rel="noopener noreferrer">
          Study: BPA disrupts prefrontal cortex neurochemistry (Niwa et al., 2011)
        </a>
      </p>

      <h4>🧩 Cognitive & Emotional Effects</h4>
      <ul>
        <li>Impairs <strong>learning and memory</strong> by damaging the hippocampus</li>
        <li>Reduces <strong>dendritic spine density</strong> and <strong>synaptic plasticity</strong></li>
      </ul>

      <p>
        🧪 <a href="https://doi.org/10.1016/j.neuro.2012.03.002" target="_blank" rel="noopener noreferrer">
          Study: BPA impairs memory and synaptic function (Eilam-Stock et al., 2012)
        </a>
      </p>

      <h3>♻️ BPA in the Brain — Entry & Elimination</h3>

      <h4>1. Entry via Circulation</h4>
      <ul>
        <li>BPA enters bloodstream via <strong>ingestion</strong>, <strong>inhalation</strong>, or <strong>skin</strong></li>
        <li>Crosses the BBB by <strong>passive diffusion</strong> or <strong>active transport</strong></li>
      </ul>

      <h4>2. Local Effects & Detox</h4>
      <ul>
        <li>Brain has <strong>low detox enzyme activity</strong> (CYPs, UGTs, SULTs)</li>
        <li>BPA persists in <strong>fatty brain regions</strong></li>
      </ul>

      <p>
        🧪 <a href="https://doi.org/10.1016/S0045-6535(98)00377-3" target="_blank" rel="noopener noreferrer">
          Study: BPA has limited clearance in the CNS (Yokota et al., 1999)
        </a>
      </p>

      <h3>⚠️ What Happens if BPA Persists in the Brain?</h3>
      
      <ul>
        <li>Increases <strong>oxidative stress</strong> and disrupts gene expression</li>
        <li>Weakens <strong>tight junctions</strong> in the BBB</li>
      </ul>

      <p>
        🧪 <a href="https://doi.org/10.1016/j.fct.2016.06.016" target="_blank" rel="noopener noreferrer">
          Study: BPA disrupts BBB integrity (Chen et al., 2016)
        </a>
      </p>

      <h3>🔗 BPA, Estrogen Receptors, and Brain Function</h3>
      
      <ul>
        <li><strong>Estrogen receptors</strong> (ERα, ERβ, GPER) are abundant in brain regions like the <strong>hippocampus</strong> and <strong>cortex</strong></li>
        <li>BPA binding alters <strong>neuroinflammation</strong> and <strong>emotional regulation</strong></li>
      </ul>

      <p>
        🧪 <a href="https://doi.org/10.1016/j.neuro.2022.03.004" target="_blank" rel="noopener noreferrer">
          Study: BPA activates GPER in astrocytes (Quesada-Candela et al., 2022)
        </a>
      </p>

      <h3>🚨 Why This Matters</h3>
      
      <ul>
        <li>BPA is linked to <strong>ADHD</strong>, <strong>anxiety</strong>, <strong>depression</strong>, and <strong>early cognitive decline</strong></li>
      </ul>

      <p>
        🧪 <a href="https://doi.org/10.1289/EHP3791" target="_blank" rel="noopener noreferrer">
          Study: BPA alters amygdala development and emotion processing (Grohs et al., 2019)
        </a>
      </p>

      <h3>🧠 Pro Tip (Researcher View)</h3>
      <ul>
        <li><strong>GPER activation</strong> in brain cells boosts IL-6 and IL-8 production</li>
        <li>BPA may silently <strong>prime the brain for inflammatory disease</strong></li>
        <li>Neurotoxic effects are often <strong>non-linear</strong>—<strong>low doses can still be harmful</strong></li>
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
  name: 'BrainInfoPopup',
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
.brain-info-dialog {
  --el-dialog-border-radius: 12px;
}

.brain-content {
  max-height: 600px;
  overflow-y: auto;
  padding: 0 20px;
  line-height: 1.6;
}

.brain-content h2,
.brain-content h3,
.brain-content h4 {
  color: #7c3aed;
  margin-top: 24px;
  margin-bottom: 16px;
}

.brain-content h2 {
  border-bottom: 2px solid #7c3aed;
  padding-bottom: 8px;
}

.brain-content h3 {
  border-bottom: 1px solid #e1e1e1;
  padding-bottom: 4px;
}

.brain-content ul {
  margin: 16px 0;
  padding-left: 20px;
}

.brain-content li {
  margin: 8px 0;
}

.brain-content strong {
  color: #7c3aed;
  font-weight: 600;
}

.brain-content em {
  color: #666;
  font-style: italic;
}

.brain-content p {
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

.brain-content a {
  color: #7c3aed;
  text-decoration: none;
  font-weight: 500;
  border-bottom: 1px solid transparent;
  transition: all 0.3s ease;
}

.brain-content a:hover {
  color: #5b21b6;
  border-bottom-color: #7c3aed;
  text-decoration: none;
}

.brain-content a:visited {
  color: #8b5cf6;
}

/* Scrollbar styling */
.brain-content::-webkit-scrollbar {
  width: 8px;
}

.brain-content::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 4px;
}

.brain-content::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 4px;
}

.brain-content::-webkit-scrollbar-thumb:hover {
  background: #a1a1a1;
}
</style>
