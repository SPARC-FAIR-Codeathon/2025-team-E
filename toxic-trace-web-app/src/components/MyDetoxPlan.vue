<template>
  <div class="detox-plan-container">
    <div class="header">
      <h2>🌿 My Detox Plan</h2>
      <p class="subtitle">Select toxins to create your personalized detoxification strategy</p>
    </div>

    <!-- Toxin Selection -->
    <div class="toxin-selection">
      <h3>Select Toxins of Interest:</h3>
      <div class="toxin-toggles">
        <el-button
          v-for="toxin in toxinData"
          :key="toxin.name"
          :type="selectedToxins.includes(toxin.name) ? 'primary' : 'default'"
          :class="{ 'toxin-selected': selectedToxins.includes(toxin.name) }"
          @click="toggleToxin(toxin.name)"
          class="toxin-toggle"
        >
          <span class="toxin-icon">{{ getToxinIcon(toxin.name) }}</span>
          {{ toxin.name }}
        </el-button>
      </div>
    </div>

    <!-- Selected Toxins Information -->
    <ToxinInfoDisplay 
      v-if="selectedToxins.length > 0"
      :selectedToxins="selectedToxins"
      :toxinData="toxinData"
    />

    <!-- No Selection Message -->
    <div v-if="selectedToxins.length === 0" class="no-selection">
      <div class="no-selection-icon">🌱</div>
      <h3>Select toxins above to view your personalized detox plan</h3>
      <p>Choose from BPA, Phthalates, PFAS, or Microplastics to get started</p>
    </div>

    <!-- Action Buttons -->
    <div class="action-buttons">
      <el-button 
        v-if="selectedToxins.length > 0"
        type="primary"
        :loading="exportingPdf"
        @click="exportToPdf"
      >
        <el-icon><Document /></el-icon>
        {{ exportingPdf ? 'Generating PDF...' : 'Export as PDF' }}
      </el-button>
      <el-button 
        v-if="selectedToxins.length > 0"
        type="default" 
        @click="clearSelection"
      >
        🗑️ Clear Selection
      </el-button>
    </div>

    <!-- Metadata -->
    <div v-if="selectedToxins.length > 0" class="metadata">
      <p><small>Data last updated: {{ toxinMetadata.lastUpdated }}</small></p>
      <p><small>Source: {{ toxinMetadata.dataSource }}</small></p>
      <p><small>{{ toxinMetadata.note }}</small></p>
    </div>
  </div>
</template>

<script>

import detoxificationData from './data/detoxification.json';
import ToxinInfoDisplay from './ToxinInfoDisplay.vue';
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';
import { Document } from '@element-plus/icons-vue';
import { ElMessage } from 'element-plus';

export default {
  name: 'MyDetoxPlan',
  components: {
    ToxinInfoDisplay,
    Document
  },
  data() {
    return {
      selectedToxins: [],
      toxinData: detoxificationData.toxins,
      toxinMetadata: detoxificationData.metadata,
      exportingPdf: false
    };
  },
  computed: {
    // No computed properties needed for save/load
  },
  methods: {
    toggleToxin(toxinName) {
      const index = this.selectedToxins.indexOf(toxinName);
      if (index > -1) {
        this.selectedToxins.splice(index, 1);
      } else {
        this.selectedToxins.push(toxinName);
      }
      
      // Emit event for parent component
      this.$emit('toxins-selected', this.selectedToxins);
    },
    
    getToxinIcon(toxinName) {
      const icons = {
        'BPA': '🧴',
        'Phthalates': '🛢️',
        'PFAS': '⚗️',
        'Microplastics': '🔬'
      };
      return icons[toxinName] || '💧';
    },
    
    clearSelection() {
      this.selectedToxins = [];
      this.$emit('toxins-selected', []);
    },

    async exportToPdf() {
      if (this.selectedToxins.length === 0) {
        ElMessage.warning('Please select at least one toxin before exporting');
        return;
      }

      this.exportingPdf = true;

      try {
        // Create a temporary container with the content to export
        const exportContent = this.createPdfContent();
        
        // Create a temporary element for rendering
        const tempDiv = document.createElement('div');
        tempDiv.innerHTML = exportContent;
        tempDiv.style.position = 'absolute';
        tempDiv.style.left = '-9999px';
        tempDiv.style.top = '-9999px';
        tempDiv.style.width = '800px';
        tempDiv.style.backgroundColor = 'white';
        tempDiv.style.padding = '40px';
        tempDiv.style.fontFamily = 'Arial, sans-serif';
        document.body.appendChild(tempDiv);

        // Generate canvas from HTML content
        const canvas = await html2canvas(tempDiv, {
          scale: 2,
          useCORS: true,
          backgroundColor: '#ffffff',
          width: 880,
          height: tempDiv.scrollHeight + 80
        });

        // Clean up temporary element
        document.body.removeChild(tempDiv);

        // Create PDF
        const pdf = new jsPDF('p', 'mm', 'a4');
        const imgWidth = 210;
        const pageHeight = 295;
        const imgHeight = (canvas.height * imgWidth) / canvas.width;
        let heightLeft = imgHeight;

        let position = 0;

        // Add the image to PDF
        pdf.addImage(canvas.toDataURL('image/png'), 'PNG', 0, position, imgWidth, imgHeight);
        heightLeft -= pageHeight;

        // Add new pages if content is longer than one page
        while (heightLeft >= 0) {
          position = heightLeft - imgHeight;
          pdf.addPage();
          pdf.addImage(canvas.toDataURL('image/png'), 'PNG', 0, position, imgWidth, imgHeight);
          heightLeft -= pageHeight;
        }

        // Generate filename with current date
        const currentDate = new Date();
        const dateString = currentDate.toISOString().split('T')[0];
        const filename = `my-detox-plan-${dateString}.pdf`;

        // Save the PDF
        pdf.save(filename);

        ElMessage.success('PDF exported successfully!');

      } catch (error) {
        console.error('Error generating PDF:', error);
        ElMessage.error('Failed to generate PDF. Please try again.');
      } finally {
        this.exportingPdf = false;
      }
    },

    createPdfContent() {
      const selectedToxinData = this.selectedToxins.map(toxinName => 
        this.toxinData.find(toxin => toxin.name === toxinName)
      );

      let content = `
        <div style="font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px;">
          <div style="text-align: center; margin-bottom: 40px; border-bottom: 3px solid #667eea; padding-bottom: 20px;">
            <h1 style="color: #2c3e50; margin: 0 0 10px 0; font-size: 32px;">🌿 My Personal Detox Plan</h1>
            <p style="color: #7f8c8d; font-size: 16px; margin: 0;">Generated on ${new Date().toLocaleDateString()}</p>
          </div>

          <div style="margin-bottom: 30px;">
            <h2 style="color: #34495e; border-bottom: 2px solid #e9ecef; padding-bottom: 10px;">Selected Toxins:</h2>
            <div style="display: flex; flex-wrap: wrap; gap: 10px; margin-top: 15px;">
              ${this.selectedToxins.map(toxin => `
                <span style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); 
                           color: white; padding: 8px 16px; border-radius: 20px; font-weight: 600;">
                  ${this.getToxinIcon(toxin)} ${toxin}
                </span>
              `).join('')}
            </div>
          </div>
      `;

      // Add detailed information for each selected toxin
      selectedToxinData.forEach(toxin => {
        if (toxin) {
          content += `
            <div style="margin-bottom: 40px; border: 1px solid #e9ecef; border-radius: 12px; padding: 25px; background: #f8f9fc;">
              <h3 style="color: #2c3e50; margin: 0 0 20px 0; font-size: 24px; border-bottom: 2px solid #667eea; padding-bottom: 8px;">
                ${this.getToxinIcon(toxin.name)} ${toxin.name} Detoxification Plan
              </h3>

              <div style="margin-bottom: 20px;">
                <h4 style="color: #34495e; margin: 0 0 10px 0;">🔬 Scientific Evidence</h4>
                <p style="margin: 0 0 10px 0; line-height: 1.6;">${toxin.detoxEvidence.description}</p>
                <p style="margin: 0; font-size: 14px;"><strong>Research:</strong> ${toxin.detoxEvidence.research}</p>
              </div>

              <div style="margin-bottom: 20px;">
                <h4 style="color: #34495e; margin: 0 0 10px 0;">⚡ Natural Elimination</h4>
                <p style="margin: 0 0 5px 0;"><strong>Pathway:</strong> ${toxin.naturalElimination.pathway}</p>
                <p style="margin: 0;"><strong>Timeframe:</strong> ${toxin.naturalElimination.timeframe}</p>
              </div>

              <div style="margin-bottom: 20px;">
                <h4 style="color: #34495e; margin: 0 0 10px 0;">🛡️ Detox Strategies</h4>
                <div style="display: flex; flex-wrap: wrap; gap: 8px;">
                  ${toxin.supportedDetoxStrategies.map(strategy => `
                    <span style="background: #d4edda; color: #155724; padding: 4px 12px; 
                               border-radius: 15px; font-size: 14px; border: 1px solid #c3e6cb;">
                      ${strategy}
                    </span>
                  `).join('')}
                </div>
              </div>

              <div style="margin-bottom: 20px;">
                <h4 style="color: #34495e; margin: 0 0 10px 0;">🥗 Lifestyle & Diet</h4>
                <div style="display: flex; flex-wrap: wrap; gap: 8px;">
                  ${toxin.lifestyleDietIntervention.map(intervention => `
                    <span style="background: #fff3cd; color: #856404; padding: 4px 12px; 
                               border-radius: 15px; font-size: 14px; border: 1px solid #ffeaa7;">
                      ${intervention}
                    </span>
                  `).join('')}
                </div>
              </div>

              <div>
                <h4 style="color: #34495e; margin: 0 0 10px 0;">🧪 Experimental Techniques</h4>
                <div style="display: flex; flex-wrap: wrap; gap: 8px;">
                  ${toxin.experimentalTechniques.map(technique => `
                    <span style="background: #d1ecf1; color: #0c5460; padding: 4px 12px; 
                               border-radius: 15px; font-size: 14px; border: 1px solid #bee5eb;">
                      ${technique}
                    </span>
                  `).join('')}
                </div>
              </div>
            </div>
          `;
        }
      });

      content += `
          <div style="margin-top: 40px; padding-top: 20px; border-top: 1px solid #e9ecef; text-align: center;">
            <p style="color: #95a5a6; font-size: 14px; margin: 0;">
              Generated by Toxic-Trace Platform • Data last updated: ${this.toxinMetadata.lastUpdated}
            </p>
            <p style="color: #95a5a6; font-size: 12px; margin: 5px 0 0 0;">
              ${this.toxinMetadata.note}
            </p>
          </div>
        </div>
      `;

      return content;
    }
  },
  
  mounted() {
    // Component ready - no auto-loading needed
  }
};
</script>

<style scoped lang="scss">
.detox-plan-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 24px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.header {
  text-align: center;
  margin-bottom: 32px;
  
  h2 {
    color: #2c3e50;
    margin: 0 0 8px 0;
    font-size: 28px;
    font-weight: 700;
  }
  
  .subtitle {
    color: #7f8c8d;
    font-size: 16px;
    margin: 0;
  }
}

.toxin-selection {
  margin-bottom: 32px;
  
  h3 {
    color: #34495e;
    margin-bottom: 16px;
    font-size: 20px;
  }
}

.toxin-toggles {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  justify-content: center;
}

.toxin-toggle {
  padding: 12px 24px;
  border-radius: 25px;
  font-weight: 600;
  transition: all 0.3s ease;
  border: 2px solid #e9ecef;
  
  .toxin-icon {
    margin-right: 8px;
    font-size: 18px;
  }
  
  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  }
  
  &.toxin-selected {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border-color: #667eea;
    color: white;
  }
}

.no-selection {
  text-align: center;
  padding: 40px 20px;
  color: #7f8c8d;
  
  .no-selection-icon {
    font-size: 64px;
    margin-bottom: 16px;
  }
  
  h3 {
    color: #95a5a6;
    margin-bottom: 8px;
  }
  
  p {
    color: #bdc3c7;
  }
}

.action-buttons {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin: 32px 0;
  flex-wrap: wrap;

  .el-button {
    font-weight: 600;
    padding: 12px 24px;
    border-radius: 25px;
    transition: all 0.3s ease;

    &:hover {
      transform: translateY(-2px);
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
    }

    &.el-button--primary {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      border: none;

      &:hover {
        background: linear-gradient(135deg, #5a67d8 0%, #6b46c1 100%);
      }
    }
  }
}

.metadata {
  text-align: center;
  margin-top: 32px;
  padding-top: 20px;
  border-top: 1px solid #e9ecef;
  
  p {
    margin: 4px 0;
    color: #95a5a6;
  }
}

// Element Plus overrides
:deep(.el-button) {
  font-weight: 600;
}

:deep(.el-tag) {
  font-weight: 500;
  border-radius: 6px;
}

// Responsive design
@media (max-width: 768px) {
  .detox-plan-container {
    padding: 16px;
  }
  
  .toxin-toggles {
    justify-content: center;
  }
  
  .toxin-toggle {
    padding: 8px 16px;
    font-size: 14px;
  }
  
  .action-buttons {
    flex-direction: column;
    align-items: center;
  }
}
</style>
