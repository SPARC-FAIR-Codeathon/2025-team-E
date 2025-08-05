<template>
  <div class="viewer-container">
    <div class="map-selector-container">
    
      <div class="select-label">
        Body System:
      </div>

      <el-select 
        v-model="selectedMap" 
        placeholder="Body System" 
        @change="onMapSelectionChange"
        style="width: 140px"
        class="body-system-select"
      >
        <el-option
          v-for="option in mapOptions"
          :key="option.value"
          :label="option.label"
          :value="option.value"
        />
      </el-select>

      <div class="quiz-controls">
        <el-button 
          @click="toggleQuiz"
          :type="showQuiz ? 'primary' : 'default'"
        >
          {{ showQuiz ? '❌ Hide Quiz' : '📝 Take Health Quiz' }}
        </el-button>
        
        <el-button 
          @click="toggleDetoxPlan"
          :type="showDetoxPlan ? 'primary' : 'default'"
        >
          {{ showDetoxPlan ? '❌ Hide Plan' : '🌿 My Detox Plan' }}
        </el-button>
        
        <el-button 
          @click="togglePdbViewer"
          type="default"
        >
          🧬 Open Molecular Viewer
        </el-button>
        
        <el-button 
          @click="clearHealthDataFromStorage"
          type="warning"
          size="small"
        >
          🔄 Reset Health Data
        </el-button>

        <SupportResearchPopover 
          :visible="fundingPopupVisible"
          @toggle="toggleFundingPopup"
        />
      </div>
    </div>
    
    <div class="map-display-area">
      <div v-if="!selectedMap" class="placeholder-text">
        Please select a body system to display
      </div>
      <div v-else class="image-container">
        <object v-if="isSVGFile"
          :data="selectedMapImage"
          type="image/svg+xml"
          class="system-image svg-object"
          ref="svgObject"
          @load="onSVGLoad"
        >
          <!-- Fallback for browsers that don't support object tag -->
          <img :src="selectedMapImage" :alt="`${selectedMapLabel} diagram`" class="system-image" />
        </object>
        
        <img v-else
          :src="selectedMapImage" 
          :alt="`${selectedMapLabel} diagram`"
          @error="onImageError"
          @load="onImageLoad"
          class="system-image"
        />
        
        <div v-if="imageError" class="error-message">
          Image not found for {{ selectedMapLabel }}
          <br>
          <small>Expected: {{ selectedMapImage }}</small>
        </div>
      </div>
    </div>

    <!-- Health Quiz Component in Dialog -->
    <el-dialog
      v-model="showQuiz"
      title="🏥 Health Assessment Quiz"
      width="90%"
      :close-on-click-modal="false"
      :close-on-press-escape="true"
      append-to-body
      class="health-quiz-dialog"
    >
      <HealthQuiz 
        @health-data-updated="onHealthDataUpdated"
        @health-data-loaded="onHealthDataLoaded"
        @quiz-completed="onQuizCompleted"
      />
      
      <template #footer>
        <el-button @click="showQuiz = false" type="info">
          ✕ Close Quiz
        </el-button>
      </template>
    </el-dialog>

    <!-- My Detox Plan Component in Dialog -->
    <el-dialog
      v-model="showDetoxPlan"
      title="🌿 My Detox Plan"
      width="95%"
      :close-on-click-modal="false"
      :close-on-press-escape="true"
      append-to-body
      class="detox-plan-dialog"
    >
      <MyDetoxPlan 
        @toxins-selected="onToxinsSelected"
      />
      
      <template #footer>
        <el-button @click="showDetoxPlan = false" type="info">
          ✕ Close Detox Plan
        </el-button>
      </template>
    </el-dialog>

    <!-- Liver Information Popup -->
    <LiverInfoPopup 
      :show-dialog="showLiverInfo"
      @close="showLiverInfo = false"
    />

    <!-- Lung Information Popup -->
    <LungInfoPopup 
      :show-dialog="showLungInfo"
      @close="showLungInfo = false"
    />

    <!-- Brain Information Popup -->
    <BrainInfoPopup 
      :show-dialog="showBrainInfo"
      @close="showBrainInfo = false"
    />

    <!-- Full-screen PDB Viewer Overlay -->
    <div v-if="showPdbViewer" class="pdb-viewer-overlay" @click="closePdbViewer">
      <div class="pdb-viewer-content" @click.stop>
        <div class="pdb-viewer-header">
          <h3>🧬 Molecular Structure Viewer</h3>
          <button @click="closePdbViewer" class="close-button">✕</button>
        </div>
        <iframe 
          src="/pdb_viewer.html"
          class="pdb-viewer-iframe"
          frameborder="0"
          title="Molecular Structure Viewer"
        ></iframe>
      </div>
    </div>

    <!-- Toxin Sidebar (always rendered so tab is always visible) -->
    <ToxinSidebar
      @close="closeToxinSidebar"
      @toxin-selected="onToxinSelected"
      @view-details="onViewToxinDetails"
    />

    <HelpModeDialog
      v-if="helpMode && useHelpModeDialog"
      ref="multiflatmapHelp"
      :multiflatmapRef="multiflatmapRef"
      :lastItem="helpModeLastItem"
      @show-next="onHelpModeShowNext"
      @finish-help-mode="onFinishHelpMode"
    />
    
    <!-- Chatbase GPT Widget -->
    <div id="chatbase-widget" ref="chatbaseWidget"></div>
  </div>
</template>

<script>
/* eslint-disable no-alert, no-console */
import ContentMixin from "../../mixins/ContentMixin";
import EventBus from "../EventBus";
import {
  capitalise,
  availableSpecies,
  getBodyScaffoldInfo
} from "../scripts/utilities";
import DyncamicMarkerMixin from "../../mixins/DynamicMarkerMixin";

import YellowStar from "../../icons/yellowstar";
import HealthQuiz from "../HealthQuiz.vue";
import ToxinSidebar from "../ToxinSidebar.vue";
import MyDetoxPlan from "../MyDetoxPlan.vue";
import LiverInfoPopup from "../organ-pages/LiverInfoPopup.vue";
import LungInfoPopup from "../organ-pages/LungInfoPopup.vue";
import BrainInfoPopup from "../organ-pages/BrainInfoPopup.vue";
import SupportResearchPopover from "../SupportResearchPopover.vue";

import { MultiFlatmapVuer } from "@abi-software/flatmapvuer";
import "@abi-software/flatmapvuer/dist/style.css";
import { HelpModeDialog } from '@abi-software/map-utilities'
import '@abi-software/map-utilities/dist/style.css'

const getOpenMapOptions = (species) => {
  const options = [
    {
      display: "Open AC Map",
      key: "AC"
    },
    {
      display: "Open FC Map",
      key: "FC"
    },
    {
      display: "Open 3D Human Map",
      key: "3D"
    },
  ]
  return options;
}

export default {
  name: "MultiFlatmap",
  mixins: [ContentMixin, DyncamicMarkerMixin],
  components: {
    MultiFlatmapVuer,
    HelpModeDialog,
    HealthQuiz,
    ToxinSidebar,
    MyDetoxPlan,
    LiverInfoPopup,
    LungInfoPopup,
    BrainInfoPopup,
    SupportResearchPopover,
  },
  data: function () {
    return {
      zoomLevel: 6,
      flatmapReady: false,
      availableSpecies: availableSpecies(),
      scaffoldResource: { },
      showStarInLegend: false,
      openMapOptions: getOpenMapOptions("Human Male"),
      selectedMap: 'full-body', // Default to full-body on page load
      imageError: false,
      showQuiz: false,
      showToxinSidebar: false,
      showDetoxPlan: false,
      showPdbViewer: false,
      showLiverInfo: false,
      showLungInfo: false,
      showBrainInfo: false,
      fundingPopupVisible: false,
      // Health percentages for each organ (constants for now)
      organHealthData: {
          'brain': { health: -1, x: 150, y: 80 },
          'heart': { health: -1, x: 150, y: 180 },
          'lungs': { health: -1, x: 150, y: 150 },
          'liver': { health: -1, x: 140, y: 230 },
          'stomach': { health: -1, x: 170, y: 260 },
          'intestine-small': { health: -1, x: 130, y: 330 },
          'intestine-large': { health: -1, x: 150, y: 345 }
      },
      mapOptions: [
        { value: 'full-body', label: 'Full Body', image: '/body-system-images/full-body.svg' },
        { value: 'digestive', label: 'Digestive System', image: '/body-system-images/digestive.png'},
        { value: 'respiratory', label: 'Respiratory System', image: '/body-system-images/respiratory.png'},
        { value: 'lymphatic', label: 'Lymphatic System', image: '/body-system-images/lymphatic.png'},
        { value: 'musculoskeletal', label: 'Musculoskeletal System', image: '/body-system-images/musculoskeletal.png'},
        { value: 'central-nervous', label: 'Central Nervous System', image: '/body-system-images/central-nervous.png'},
        { value: 'circulatory', label: 'Circulatory System', image: '/body-system-images/circulatory.png'},
        { value: 'endocrine', label: 'Endocrine System', image: '/body-system-images/endocrine.png'},
        { value: 'neuron', label: 'Neuron', image: '/body-system-images/neuron.png'},
        { value: 'urinary', label: 'Urinary System', image: '/body-system-images/urinary.png'},
        { value: 'synapse', label: 'Synapse', image: '/body-system-images/synapse.png'},
        { value: 'skin', label: 'Skin', image: '/body-system-images/skin.png'},
      ]
    }
  },
  methods: {
    getState: function () {
      return {};
    },
    flatmapPanZoomCallback: function (payload) {
      return null;
    },
    /**
     * Perform a local search on this contentvuer
     */
    search: function (term) {
      return false;
    },
    /**
     * Append the list of suggested terms to suggestions
     */
    searchSuggestions: function (term, suggestions) {
      return null;
    },
    flatmaprResourceSelected: function (type, resource) {
      if (resource.eventType === 'click' && resource.feature.type === 'feature') {
        console.log('Flatmap resource selected:', {
          label: resource.label || '',
          id: resource.feature.id || '',
          featureId: resource.feature.featureId || '',
          taxonomy: resource.taxonomy || '',
          resources: resource.resource.join(', ')
        });
      }
    },
    onPathwaySelectionChanged: function (data) {
      const { label, property, checked, selectionsTitle } = data;
      console.log('Pathway selection changed:', { label, property, checked, selectionsTitle });
    },
    onSidebarAnnotationClose: function() {
      if (this.flatmapReady) {
  
      }
    },
    onOpenPubmedUrl: function (url) {
      console.log('Opening PubMed URL:', url);
    },
    displayTooltip: function (info) {
      if (info) {
        let name = info.name;
        if (name) {
          this.search(name);
        } else {
          const flatmap = this.$refs.multiflatmap.getCurrentFlatmap();
          flatmap.mapImp.clearSearchResults();
        }
      }
    },
    zoomToFeatures: function (info, forceSelect) {
      return null;
    },
    highlightFeatures: function (info) {
      return null;
    },
    updateProvCard: function() {
      return null;
    },
    flatmapChanged: async function (activeSpecies) {

    },
    multiFlatmapReady: function (flatmap) {
      if (flatmap) {
        this.flatmapReady = true;
        const flatmapImp = undefined
        EventBus.emit("mapLoaded", flatmap);
      }
    },
    getFlatmapImp: function () {
      if (this.entry.type === "MultiFlatmap" && this.flatmapReady && this.$refs.multiflatmap) {
        return this.$refs.multiflatmap.getCurrentFlatmap()["mapImp"];
      } else {
        return undefined;
      }
    },
    flatmapAreaSearch() {
      return null;
    },
    restoreFeaturedMarkers: function (flatmap) {

      return null;
    },
    // updateFeaturedMarkers will step through the featured markers and add them to the map
    updateFeaturedMarkers: function (markers, flatmap) {
      return null;
    },
    // addFeaturedMarker: add a featured marker to the map at the specified uberon location
    addFeaturedMarker: function (marker, index, flatmap) {
      return false;
    },
    /**
     * Change the view mode of the current flatmap
     */
    changeViewingMode: function (modeName) {
      return null;
    },
    showConnectivity: function (payload) {
      if (this?.alive && this.flatmapReady && this.$refs.multiflatmap) {
        const { featureIds, offset } = payload;
        const currentFlatmap = this.$refs.multiflatmap.getCurrentFlatmap();
        if (currentFlatmap) {
          currentFlatmap.moveMap(featureIds, {
            offsetX: offset ? -150 : 0,
            zoom: 4,
          });
        }
      }
    },
    showConnectivityTooltips: function (payload) {
      if (this?.alive && this.flatmapReady) {
        const flatmap = this.$refs.multiflatmap.getCurrentFlatmap();
        flatmap.showConnectivityTooltips(payload);
      }
    },
    showConnectivitiesByReference: function (payload) {
      if (this?.alive && this.flatmapReady && this.$refs.multiflatmap) {
        const currentFlatmap = this.$refs.multiflatmap.getCurrentFlatmap();
        if (currentFlatmap) {
          currentFlatmap.showConnectivitiesByReference(payload);
        }
      }
    },
    changeConnectivitySource: function (payload) {
      return null;
    },
    updateViewerSettings: function () {
      return null;
    },
    setVisibilityFilter: function (payload) {
      return null;
    },
    onMapSelectionChange: function(value) {
      console.log('Selected map:', value);
      this.selectedMap = value;
      this.imageError = false; // Reset error state when changing maps
      // You can emit an event or call other methods when the map selection changes
      this.$emit('map-changed', value);
    },
    onImageError: function() {
      this.imageError = true;
      console.warn(`Image not found: ${this.selectedMapImage}`);
    },
    onImageLoad: function() {
      this.imageError = false;
      console.log(`Image loaded successfully: ${this.selectedMapImage}`);
    },
    onSVGLoad: function() {
      this.imageError = false;
      console.log(`SVG loaded successfully: ${this.selectedMapImage}`);
      
      // Load any saved health data first
      this.loadHealthDataFromStorage();
      
      // Set up click handlers for SVG
      this.setupSVGClickHandlers();
    },
    setupSVGClickHandlers: function() {
      this.$nextTick(() => {
        const svgObject = this.$refs.svgObject;
        if (!svgObject) return;

        // Wait a bit for the SVG to fully load
        setTimeout(() => {
          const svgDoc = svgObject.contentDocument;
          
          if (svgDoc) {
            // Add click handlers for each organ class
            const organClasses = ['brain', 'heart', 'lungs', 'liver', 'stomach', 'intestine-small', 'intestine-large'];
            
            organClasses.forEach(className => {
              const elements = svgDoc.querySelectorAll(`.${className}`);
              elements.forEach((element, index) => {
                element.style.cursor = 'pointer';
                element.addEventListener('click', () => {
                  this.onOrganClick(className, this.getOrganName(className, index));
                });
                
                // Add hover effects
                element.addEventListener('mouseenter', () => {
                  element.style.strokeWidth = '3';
                  element.style.filter = 'brightness(1.1)';
                });
                
                element.addEventListener('mouseleave', () => {
                  element.style.strokeWidth = '1';
                  element.style.filter = 'none';
                });
              });
            });

            // Add health percentage text overlays
            this.addHealthPercentageOverlays(svgDoc);
          } else {
            console.warn('Could not access SVG document');
          }
        }, 100);
      });
    },
    addHealthPercentageOverlays: function(svgDoc) {
      // Create a group for health overlays
      const overlayGroup = svgDoc.createElementNS('http://www.w3.org/2000/svg', 'g');
      overlayGroup.setAttribute('id', 'health-overlays');
      
      Object.keys(this.organHealthData).forEach(organClass => {
        if (this.organHealthData[organClass].health == -1) return; // Skip if health data is not set
        const healthData = this.organHealthData[organClass];
        const organElements = svgDoc.querySelectorAll(`.${organClass}`);
        
        if (organElements.length > 0) {
          // Create background circle for the percentage
          const bgCircle = svgDoc.createElementNS('http://www.w3.org/2000/svg', 'circle');
          bgCircle.setAttribute('cx', healthData.x);
          bgCircle.setAttribute('cy', healthData.y);
          bgCircle.setAttribute('r', '18');
          bgCircle.setAttribute('fill', 'rgba(255, 255, 255, 0.9)');
          bgCircle.setAttribute('stroke', '#333');
          bgCircle.setAttribute('stroke-width', '1');
          bgCircle.style.cursor = 'pointer';
          
          // Create text element for the percentage
          const textElement = svgDoc.createElementNS('http://www.w3.org/2000/svg', 'text');
          textElement.setAttribute('x', healthData.x);
          textElement.setAttribute('y', healthData.y + 4); // Offset for center alignment
          textElement.setAttribute('text-anchor', 'middle');
          textElement.setAttribute('font-family', 'Arial, sans-serif');
          textElement.setAttribute('font-size', '12');
          textElement.setAttribute('font-weight', 'bold');
          textElement.setAttribute('fill', this.getHealthColor(healthData.health));
          textElement.textContent = `${healthData.health}%`;
          textElement.style.cursor = 'pointer';
          textElement.style.pointerEvents = 'none'; // Let clicks pass through to background
          
          // Add click handler to the background circle
          bgCircle.addEventListener('click', () => {
            this.onHealthPercentageClick(organClass, healthData.health);
          });
          
          overlayGroup.appendChild(bgCircle);
          overlayGroup.appendChild(textElement);
        }
      });
      
      // Add the overlay group to the SVG
      const svgRoot = svgDoc.querySelector('svg');
      if (svgRoot) {
        svgRoot.appendChild(overlayGroup);
      }
    },
    getHealthColor: function(percentage) {
      if (percentage >= 90) return '#22c55e'; // Green
      if (percentage >= 75) return '#f59e0b'; // Yellow/Orange
      if (percentage >= 60) return '#f97316'; // Orange
      return '#ef4444'; // Red
    },
    onHealthPercentageClick: function(organClass, healthPercentage) {
      console.log(`Clicked on ${organClass} health: ${healthPercentage}%`);
      
      // Emit an event for health percentage clicks
      this.$emit('health-clicked', {
        organ: organClass,
        health: healthPercentage,
        timestamp: new Date().toISOString()
      });
    },
    // Centralized popup toggle handler
    togglePopup: function(popupType) {
      // First, close all popups
      this.closeAllPopups();
      
      // Then toggle the requested popup
      switch(popupType) {
        case 'quiz':
          this.showQuiz = !this.showQuiz;
          break;
        case 'detoxPlan':
          this.showDetoxPlan = !this.showDetoxPlan;
          break;
        case 'toxinSidebar':
          this.showToxinSidebar = !this.showToxinSidebar;
          break;
        case 'fundingPopup':
          this.fundingPopupVisible = !this.fundingPopupVisible;
          break;
        default:
          console.warn(`Unknown popup type: ${popupType}`);
      }
    },
    
    // Helper function to close all popups
    closeAllPopups: function() {
      this.showQuiz = false;
      this.showDetoxPlan = false;
      this.showToxinSidebar = false;
      this.fundingPopupVisible = false;
    },
    
    // Individual toggle methods for template binding
    toggleQuiz: function() {
      this.togglePopup('quiz');
    },
    toggleToxinSidebar: function() {
      this.togglePopup('toxinSidebar');
    },
    toggleDetoxPlan: function() {
      this.togglePopup('detoxPlan');
    },
    togglePdbViewer: function() {
      this.showPdbViewer = !this.showPdbViewer;
    },
    closePdbViewer: function() {
      this.showPdbViewer = false;
    },
    closeToxinSidebar: function() {
      this.showToxinSidebar = false;
    },
    onToxinSelected: function(selectedToxins) {
      // Handle toxin selection from sidebar
      console.log('Toxins selected:', selectedToxins);
    },
    onToxinsSelected: function(selectedToxins) {
      // Handle toxin selection from MyDetoxPlan component
      console.log('Toxins selected from detox plan:', selectedToxins);
    },
    onViewToxinDetails: function(toxin) {
      // Handle viewing toxin details (could open MyDetoxPlan component)
      console.log('View toxin details:', toxin);
    },
    toggleFundingPopup: function() {
      // Toggle the funding popup visibility
      this.fundingPopupVisible = !this.fundingPopupVisible;
    },
    openChatbot: function() {
      // Programmatically open the Chatbase chat widget
      console.log('Opening chatbot...');
      
      // Try to click the chatbase button to open the chat
      const chatButton = document.getElementById('chatbase-bubble-button');
      if (chatButton) {
        chatButton.click();
        console.log('Chatbot opened by clicking the button');
      } else {
        console.log('Chatbase button not found, trying to trigger via window.chatbase');
        
        // Alternative method: try to use the chatbase API if available
        if (window.chatbase) {
          try {
            window.chatbase('open');
          } catch (error) {
            console.log('Failed to open chatbot via API:', error);
          }
        }
        
        // Retry after a short delay in case the button wasn't loaded yet
        setTimeout(() => {
          const retryButton = document.getElementById('chatbase-bubble-button');
          if (retryButton) {
            retryButton.click();
            console.log('Chatbot opened on retry');
          }
        }, 500);
      }
    },
    onHealthDataUpdated: function(healthData) {
      // Update organ health data with quiz results
      Object.keys(healthData).forEach(organ => {
        if (this.organHealthData[organ]) {
          this.organHealthData[organ].health = healthData[organ];
        }
      });
      
      // Always refresh the health overlays when data is updated
      this.$nextTick(() => {
        if (this.isSVGFile && this.$refs.svgObject) {
          this.refreshHealthOverlays();
        }
      });
      
      console.log('Health data updated:', healthData);
    },
    onHealthDataLoaded: function(healthData) {
      // Load saved health data from local storage
      Object.keys(healthData).forEach(organ => {
        if (this.organHealthData[organ]) {
          this.organHealthData[organ].health = healthData[organ];
        }
      });
      
      // Always refresh the health overlays when data is loaded
      this.$nextTick(() => {
        if (this.isSVGFile && this.$refs.svgObject) {
          this.refreshHealthOverlays();
        }
      });
      
      console.log('Health data loaded from storage:', healthData);
    },
    onQuizCompleted: function(quizData) {
      console.log('Quiz completed:', quizData);
      
      // Emit event for parent components
      this.$emit('quiz-completed', quizData);
    },
    refreshHealthOverlays: function() {
      this.$nextTick(() => {
        const svgObject = this.$refs.svgObject;
        if (!svgObject) return;

        setTimeout(() => {
          const svgDoc = svgObject.contentDocument;
          if (svgDoc) {
            // Remove existing overlays
            const existingOverlays = svgDoc.getElementById('health-overlays');
            if (existingOverlays) {
              existingOverlays.remove();
            }
            
            // Add updated overlays with new health data
            this.addHealthPercentageOverlays(svgDoc);
            console.log('Health overlays refreshed with updated data');
          }
        }, 100);
      });
    },
    loadHealthDataFromStorage: function() {
      try {
        const savedData = localStorage.getItem('healthQuizResults');
        if (savedData) {
          const healthData = JSON.parse(savedData);
          if (healthData.organHealth) {
            // Update organ health data with saved results
            Object.keys(healthData.organHealth).forEach(organ => {
              if (this.organHealthData[organ]) {
                this.organHealthData[organ].health = healthData.organHealth[organ];
              }
            });
            
            console.log('Loaded health data from storage on mount:', healthData.organHealth);
            return healthData.organHealth;
          }
        }
      } catch (error) {
        console.error('Failed to load health data from storage:', error);
      }
      return null;
    },
    clearHealthDataFromStorage: function() {
      try {
        localStorage.removeItem('healthQuizResults');
        
        // Reset to default health values
        this.organHealthData = {
          'brain': { health: -1, x: 150, y: 80 },
          'heart': { health: -1, x: 150, y: 180 },
          'lungs': { health: -1, x: 150, y: 150 },
          'liver': { health: -1, x: 140, y: 230 },
          'stomach': { health: -1, x: 170, y: 260 },
          'intestine-small': { health: -1, x: 130, y: 330 },
          'intestine-large': { health: -1, x: 150, y: 345 }
        };
        
        // Refresh overlays if SVG is loaded
        if (this.isSVGFile && this.$refs.svgObject) {
          this.refreshHealthOverlays();
        }
        
        console.log('Health data cleared from storage and reset to defaults');
      } catch (error) {
        console.error('Failed to clear health data from storage:', error);
      }
    },
    initializeChatbase: function() {
      // Initialize Chatbase GPT widget
      if (!window.chatbase || window.chatbase("getState") !== "initialized") {
        window.chatbase = (...args) => {
          if (!window.chatbase.q) {
            window.chatbase.q = [];
          }
          window.chatbase.q.push(args);
        };
        
        window.chatbase = new Proxy(window.chatbase, {
          get(target, prop) {
            if (prop === "q") {
              return target.q;
            }
            return (...args) => target(prop, ...args);
          }
        });
      }
      
      // Add global CSS for Chatbase styling
      this.addChatbaseStyles();
      
      // Load the Chatbase script
      const script = document.createElement("script");
      script.src = "https://www.chatbase.co/embed.min.js";
      script.id = "MdW4jU0fFrfgF7QWDZFZs";
      script.domain = "www.chatbase.co";
      script.defer = true;
      
      // Apply styles after script loads
      script.onload = () => {
        setTimeout(() => {
          this.applyChatbaseStyles();
        }, 1000);
      };
      
      // Add to head instead of body for better performance
      document.head.appendChild(script);
      
      console.log('Chatbase GPT widget initialized');
    },
    addChatbaseStyles: function() {
      // Add global CSS styles that will override Chatbase defaults
      const style = document.createElement('style');
      style.textContent = `
        #chatbase-bubble-button {
          background: linear-gradient(135deg, #8E4EC6, #9d4edd) !important;
          border: 2px solid #7b2cbf !important;
          box-shadow: 0 8px 20px rgba(123, 44, 191, 0.4) !important;
          transition: all 0.3s ease !important;
        }
        
        #chatbase-bubble-button:hover {
          background: linear-gradient(135deg, #7b2cbf, #8E4EC6) !important;
          transform: scale(1.05) !important;
          box-shadow: 0 12px 24px rgba(123, 44, 191, 0.5) !important;
        }
      `;
      document.head.appendChild(style);
    },
    applyChatbaseStyles: function() {
      // Apply styles directly to the chatbase button
      const chatbaseButton = document.getElementById('chatbase-bubble-button');
      if (chatbaseButton) {
        chatbaseButton.style.background = 'linear-gradient(135deg, #8E4EC6, #9d4edd)';
        chatbaseButton.style.border = '2px solid #7b2cbf';
        chatbaseButton.style.boxShadow = '0 8px 20px rgba(123, 44, 191, 0.4)';
        chatbaseButton.style.transition = 'all 0.3s ease';
        
        // Add hover effect
        chatbaseButton.addEventListener('mouseenter', () => {
          chatbaseButton.style.background = 'linear-gradient(135deg, #7b2cbf, #8E4EC6)';
          chatbaseButton.style.transform = 'scale(1.05)';
          chatbaseButton.style.boxShadow = '0 12px 24px rgba(123, 44, 191, 0.5)';
        });
        
        chatbaseButton.addEventListener('mouseleave', () => {
          chatbaseButton.style.background = 'linear-gradient(135deg, #8E4EC6, #9d4edd)';
          chatbaseButton.style.transform = 'scale(1)';
          chatbaseButton.style.boxShadow = '0 8px 20px rgba(123, 44, 191, 0.4)';
        });
        
      } else {
        // Try again after a delay
        setTimeout(() => {
          this.applyChatbaseStyles();
        }, 2000);
      }
    },
    getOrganName: function(className, index) {
      const organNames = {
        'brain': 'Brain',
        'heart': 'Heart',
        'lungs': index === 0 ? 'Left Lung' : 'Right Lung',
        'liver': 'Liver',
        'stomach': 'Stomach',
        'intestine-small': 'Small Intestine',
        'intestine-large': 'Large Intestine'
      };
      return organNames[className] || className;
    },
    onOrganClick: function(organId, organName) {
      console.log(`Clicked on ${organName} (${organId})`);
      
      // Show liver information popup when liver is clicked
      if (organId === 'liver') {
        this.showLiverInfo = true;
      }
      // Show lung information popup when lungs are clicked
      else if (organId === 'lungs') {
        this.showLungInfo = true;
      }
      // Show brain information popup when brain is clicked
      else if (organId === 'brain') {
        this.showBrainInfo = true;
      }
    },
  },
  computed: {
    facetSpecies() {
      return this.settingsStore.facets.species;
    },
    featuredMarkers() {
      return this.settingsStore.featuredMarkers;
    },
    selectedMapImage() {
      const option = this.mapOptions.find(opt => opt.value === this.selectedMap);
      return option ? option.image : '';
    },
    selectedMapLabel() {
      const option = this.mapOptions.find(opt => opt.value === this.selectedMap);
      return option ? option.label : this.selectedMap;
    },
    isSVGFile() {
      return this.selectedMapImage.toLowerCase().endsWith('.svg');
    }
  },
  watch: {
    featuredMarkers: function (markers) {
      if (!this.flatmapReady) {
        return;
      }
    },
  },
  mounted: function () {
    this.multiFlatmapReady();
    
    // Load saved health data from local storage
    this.loadHealthDataFromStorage();
    
    // Initialize Chatbase GPT widget
    this.$nextTick(() => {
      this.initializeChatbase();
    });
    
    // Listen for chatbot open event
    EventBus.on("openChatbot", this.openChatbot);
    
    // Listen for toxin sidebar open event
    EventBus.on("openToxinSidebar", this.toggleToxinSidebar);
  },
  unmounted: function () {
    // Clean up EventBus listeners
    EventBus.off("openChatbot", this.openChatbot);
    EventBus.off("openToxinSidebar", this.toggleToxinSidebar);
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">

.viewer-container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.map-selector-container {
  padding: 16px;
  background-color: #f8f9fa;
  border-bottom: 1px solid #e9ecef;
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.select-label {
  font-weight: 600;
  color: #2c3e50;
  font-size: 14px;
  white-space: nowrap;
}

.quiz-controls {
  margin-left: auto;
  display: flex;
  gap: 6px;
  align-items: center;
  
  .el-button {
    font-size: 13px;
    padding: 8px 12px;
    white-space: nowrap;
  }
  
  .el-button--small {
    padding: 6px 10px;
    font-size: 12px;
  }
}

.map-display-area {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  overflow: hidden;
}

.placeholder-text {
  font-size: 18px;
  color: #666;
  text-align: center;
}

.image-container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.system-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.svg-object {
  border: none;
  outline: none;
}

.error-message {
  text-align: center;
  color: #e74c3c;
  font-size: 16px;
  
  small {
    color: #666;
    font-size: 12px;
  }
}

:deep(.health-quiz-dialog) {
  .el-dialog {
    border-radius: 12px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  }
  
  .el-dialog__header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border-radius: 12px 12px 0 0;
    padding: 20px 24px;
    
    .el-dialog__title {
      font-size: 20px;
      font-weight: 600;
      color: white;
    }
    
    .el-dialog__headerbtn {
      .el-dialog__close {
        color: white;
        font-size: 18px;
        
        &:hover {
          color: #f0f0f0;
        }
      }
    }
  }
  
  .el-dialog__body {
    padding: 24px;
    max-height: 70vh;
    overflow-y: auto;
    
    /* Custom scrollbar */
    &::-webkit-scrollbar {
      width: 8px;
    }
    
    &::-webkit-scrollbar-track {
      background: #f1f1f1;
      border-radius: 4px;
    }
    
    &::-webkit-scrollbar-thumb {
      background: #c1c1c1;
      border-radius: 4px;
      
      &:hover {
        background: #a8a8a8;
      }
    }
  }
  
  .el-dialog__footer {
    padding: 16px 24px;
    background-color: #f8f9fa;
    border-radius: 0 0 12px 12px;
    border-top: 1px solid #e9ecef;
    text-align: center;
  }
}

// Detox Plan Dialog Styles
:deep(.detox-plan-dialog) {
  .el-dialog {
    border-radius: 12px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  }
  
  .el-dialog__header {
    background: linear-gradient(135deg, #27ae60 0%, #2ecc71 100%);
    color: white;
    border-radius: 12px 12px 0 0;
    padding: 20px 24px;
    
    .el-dialog__title {
      font-size: 20px;
      font-weight: 600;
      color: white;
    }
    
    .el-dialog__headerbtn {
      .el-dialog__close {
        color: white;
        font-size: 18px;
        
        &:hover {
          color: #f0f0f0;
        }
      }
    }
  }
  
  .el-dialog__body {
    padding: 0; // Let the component handle its own padding
    max-height: 75vh;
    overflow-y: auto;
    
    /* Custom scrollbar */
    &::-webkit-scrollbar {
      width: 8px;
    }
    
    &::-webkit-scrollbar-track {
      background: #f1f1f1;
      border-radius: 4px;
    }
    
    &::-webkit-scrollbar-thumb {
      background: #c1c1c1;
      border-radius: 4px;
      
      &:hover {
        background: #a8a8a8;
      }
    }
  }
  
  .el-dialog__footer {
    padding: 16px 24px;
    background-color: #f8f9fa;
    border-radius: 0 0 12px 12px;
    border-top: 1px solid #e9ecef;
    text-align: center;
  }
}

// Make sure the dialog appears above other elements
:deep(.el-overlay-dialog) {
  z-index: 2000;
}

:deep(.maplibregl-popup) {
  z-index: 11;
}

:deep(.maplibregl-marker) {
  &.standard-marker {
    cursor: pointer !important;
    z-index: 2;
  }
  &.highlight-marker {
    visibility: visible !important;
    cursor: pointer !important;
    z-index: 1;
    div {
      scale: 0.5;
      width: 0;
    }
  }
}

.pdb-viewer-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.8);
  z-index: 10000;
  display: flex;
  align-items: center;
  justify-content: center;
}

.pdb-viewer-content {
  width: 95vw;
  height: 95vh;
  background: white;
  border-radius: 8px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.pdb-viewer-header {
  background: linear-gradient(135deg, #7b2cbf, #9d4edd);
  color: white;
  padding: 15px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-shrink: 0;
}

.pdb-viewer-header h3 {
  margin: 0;
  font-size: 18px;
}

.pdb-viewer-close {
  background: none;
  border: none;
  color: white;
  font-size: 24px;
  cursor: pointer;
  padding: 0;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: background-color 0.2s;
}

.pdb-viewer-close:hover {
  background-color: rgba(255, 255, 255, 0.2);
}

.pdb-viewer-iframe {
  width: 100%;
  height: 100%;
  border: none;
  flex: 1;
}

// Chatbase GPT Widget Styling - SPARC Purple Theme
// Target the actual chatbase button and elements that get injected
:deep(#chatbase-bubble-button) {
  background-color: #8E4EC6 !important;
  background: linear-gradient(135deg, #8E4EC6, #9d4edd) !important;
  border: 2px solid #7b2cbf !important;
  box-shadow: 0 8px 20px rgba(123, 44, 191, 0.4) !important;
  
  &:hover {
    background: linear-gradient(135deg, #7b2cbf, #8E4EC6) !important;
    transform: scale(1.05) !important;
    transition: all 0.3s ease !important;
    box-shadow: 0 12px 24px rgba(123, 44, 191, 0.5) !important;
  }
}



// Additional global styles to ensure SPARC purple theme
#chatbase-widget {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 9999;
}

// Responsive adjustments for 1080px and below
@media (max-width: 1080px) {
  .map-selector-container {
    gap: 8px;
    padding: 12px 16px;
  }
  
  .quiz-controls {
    gap: 4px;
    
    .el-button {
      font-size: 12px;
      padding: 6px 10px;
    }
    
    .el-button--small {
      padding: 4px 8px;
      font-size: 11px;
    }
  }
  
  .select-label {
    font-size: 13px;
  }
}

// Body System Select Purple Theme
:deep(.body-system-select) {
  .el-input__wrapper {
    transition: all 0.3s ease;
    
    &:hover {
      box-shadow: 0 0 0 1px #8E4EC6 !important;
    }
    
    &.is-focus {
      box-shadow: 0 0 0 1px #8E4EC6 !important;
    }
  }
  
  .el-input__inner {
    &:focus {
      border-color: #8E4EC6 !important;
    }
  }
}

</style>

<!-- Global styles for Element Plus dropdown (unscoped) -->
<style lang="scss">
// Element Plus Select Dropdown Purple Theme - Global styles
.el-select-dropdown {
  .el-select-dropdown__item {
    transition: all 0.2s ease;
    
    &:hover {
      background-color: rgba(142, 78, 198, 0.1) !important;
      color: #8E4EC6 !important;
      font-weight: 600 !important;
    }
    
    &.selected {
      background-color: transparent !important;
      color: #8E4EC6 !important;
      font-weight: 700 !important;
    }
    
    &.is-selected {
      background-color: transparent !important;
      color: #8E4EC6 !important;
      font-weight: 700 !important;
    }
    
    &.selected:hover,
    &.is-selected:hover {
      background-color: rgba(142, 78, 198, 0.1) !important;
      color: #7b2cbf !important;
      font-weight: 700 !important;
    }
  }
}

// Alternative class names that Element Plus might use
.el-popper {
  .el-select-dropdown__item {
    &:hover {
      background-color: rgba(142, 78, 198, 0.1) !important;
      color: #8E4EC6 !important;
      font-weight: 600 !important;
    }
    
    &.selected,
    &.is-selected {
      background-color: transparent !important;
      color: #8E4EC6 !important;
      font-weight: 700 !important;
    }
    
    &.selected:hover,
    &.is-selected:hover {
      background-color: rgba(142, 78, 198, 0.1) !important;
      color: #7b2cbf !important;
      font-weight: 700 !important;
    }
  }
}
</style>

<style src="../../assets/mapicon-species-style.css"></style>