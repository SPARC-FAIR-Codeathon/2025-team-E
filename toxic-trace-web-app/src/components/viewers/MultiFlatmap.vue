<template>
  <div class="viewer-container">
    <div class="map-selector-container">
    
      <div class="select-label">
        Select Body System:
      </div>

      <el-select 
        v-model="selectedMap" 
        placeholder="Body System" 
        @change="onMapSelectionChange"
        style="width: 200px"
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
          @click="clearHealthDataFromStorage"
          type="warning"
          size="small"
        >
          🔄 Reset Health Data
        </el-button>

        <el-popover
          placement="bottom"
          :teleported="false"
          trigger="manual"
          :width="400"
          :offset="0"
          popper-class="funding-popover"
          :visible="fundingPopupVisible"
        >
          <template #default>
            <div class="funding-content">
              <h3 class="funding-title">Support Our Research</h3>
              <div class="funding-links">
                <div class="funding-item">
                  <a href="https://donate.stripe.com/8x214n9Pq834cUS30T1Nu04" 
                     class="funding-button subscription-button"
                     target="_blank"
                     rel="noopener noreferrer">
                    Subscribe to BPA Research
                  </a>
                </div>
                <div class="funding-item">
                  <a href="https://buy.stripe.com/aFa9ATbXy6Z08EC1WP1Nu03" 
                     class="funding-button support-button"
                     target="_blank"
                     rel="noopener noreferrer">
                    Support to Trace BPA in Food
                  </a>
                </div>
                <div class="funding-item">
                  <a href="https://donate.stripe.com/4gM5kD8Lmers9IGgRJ1Nu05" 
                     class="funding-button subscription-button"
                     target="_blank"
                     rel="noopener noreferrer">
                    Subscribe to PFAS Research
                  </a>
                </div>
                <div class="funding-item">
                  <a href="https://buy.stripe.com/7sYdR92mY1EGbQO0SL1Nu06" 
                     class="funding-button support-button"
                     target="_blank"
                     rel="noopener noreferrer">
                    Support to Trace PFAS in Food
                  </a>
                </div>
                <div class="funding-item">
                  <a href="https://donate.stripe.com/9B65kD5zaabc3kiatl1Nu08" 
                     class="funding-button subscription-button"
                     target="_blank"
                     rel="noopener noreferrer">
                    Subscribe to Phthalates Research
                  </a>
                </div>
                <div class="funding-item">
                  <a href="https://buy.stripe.com/5kQeVd0eQcjk6wubxp1Nu07" 
                     class="funding-button support-button"
                     target="_blank"
                     rel="noopener noreferrer">
                    Support to Trace Phthalates in Food
                  </a>
                </div>
              </div>
            </div>
          </template>
          <template #reference>
            <el-button 
              @click="toggleFundingPopup"
              type="danger"
              size="small"
            >
              ❤️ Support Research
            </el-button>
          </template>
        </el-popover>
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
import Tagging from '../../services/tagging.js';
import ContentMixin from "../../mixins/ContentMixin";
import EventBus from "../EventBus";
import {
  capitalise,
  availableSpecies,
  getBodyScaffoldInfo,
  transformObjToString
} from "../scripts/utilities";
import DyncamicMarkerMixin from "../../mixins/DynamicMarkerMixin";

import YellowStar from "../../icons/yellowstar";
import HealthQuiz from "../HealthQuiz.vue";
import MyDetoxPlan from "../MyDetoxPlan.vue";

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
    MyDetoxPlan,
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
      showDetoxPlan: false,
      fundingPopupVisible: false,
      // Health percentages for each organ (constants for now)
      organHealthData: {
        'brain': { health: 95, x: 150, y: 80 },
        'heart': { health: 87, x: 150, y: 180 },
        'lungs': { health: 92, x: 150, y: 150 },
        'liver': { health: 78, x: 140, y: 230 },
        'stomach': { health: 83, x: 170, y: 260 },
        'intestine-small': { health: 89, x: 130, y: 330 },
        'intestine-large': { health: 76, x: 150, y: 345 }
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
        const eventData = {
          label: resource.label || '',
          id: resource.feature.id || '',
          featureId: resource.feature.featureId || '',
          taxonomy: resource.taxonomy || '',
          resources: resource.resource.join(', ')
        };
        const paramString = transformObjToString(eventData);
        // `transformStringToObj` function can be used to change it back to object
        Tagging.sendEvent({
          'event': 'interaction_event',
          'event_name': 'portal_maps_connectivity',
          'category': paramString,
          "location": type + ' ' + map.viewingMode
        });
      }
    },
    onPathwaySelectionChanged: function (data) {
      const { label, property, checked, selectionsTitle } = data;
      // GA Tagging
      // Event tracking for maps' pathway selection change
      Tagging.sendEvent({
        'event': 'interaction_event',
        'event_name': 'portal_maps_pathway_change',
        'category': label + ' [' + property + '] ' + checked,
        'location': selectionsTitle
      });
    },
    onSidebarAnnotationClose: function() {
      if (this.flatmapReady) {
  
      }
    },
    onOpenPubmedUrl: function (url) {
      // GA Tagging
      // Event tracking for open pubmed url from popup
      Tagging.sendEvent({
        'event': 'interaction_event',
        'event_name': 'portal_maps_pubmed_url',
        'file_path': url,
        'location': 'map_popup_button',
      });
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
      
      // Optional: Track analytics for health clicks
      if (Tagging && Tagging.sendEvent) {
        Tagging.sendEvent({
          'event': 'interaction_event',
          'event_name': 'portal_health_percentage_click',
          'category': `${organClass}_${healthPercentage}%`,
          'location': 'health_overlay'
        });
      }
    },
    toggleQuiz: function() {
      this.showQuiz = !this.showQuiz;
      
      // Close funding popup and detox plan if quiz is being opened
      if (this.showQuiz) {
        this.fundingPopupVisible = false;
        this.showDetoxPlan = false;
      }
      
      // Track quiz dialog opening/closing
      if (Tagging && Tagging.sendEvent) {
        Tagging.sendEvent({
          'event': 'interaction_event',
          'event_name': this.showQuiz ? 'portal_health_quiz_opened' : 'portal_health_quiz_closed',
          'category': 'health_assessment_dialog',
          'location': 'quiz_toggle_button'
        });
      }
    },
    toggleDetoxPlan: function() {
      this.showDetoxPlan = !this.showDetoxPlan;
      
      // Close quiz and funding popup if detox plan is being opened
      if (this.showDetoxPlan) {
        this.showQuiz = false;
        this.fundingPopupVisible = false;
      }
      
      // Track detox plan dialog opening/closing
      if (Tagging && Tagging.sendEvent) {
        Tagging.sendEvent({
          'event': 'interaction_event',
          'event_name': this.showDetoxPlan ? 'portal_detox_plan_opened' : 'portal_detox_plan_closed',
          'category': 'detox_plan_dialog',
          'location': 'detox_plan_toggle_button'
        });
      }
    },
    toggleFundingPopup: function() {
      this.fundingPopupVisible = !this.fundingPopupVisible;
      // Close quiz and detox plan when opening funding popup for better UX
      if (this.fundingPopupVisible) {
        this.showQuiz = false;
        this.showDetoxPlan = false;
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
      
      // Optional: Track quiz completion
      if (Tagging && Tagging.sendEvent) {
        Tagging.sendEvent({
          'event': 'interaction_event',
          'event_name': 'portal_health_quiz_completed',
          'category': 'health_assessment',
          'location': 'health_quiz'
        });
      }
      
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
          'brain': { health: 95, x: 150, y: 80 },
          'heart': { health: 87, x: 150, y: 180 },
          'lungs': { health: 92, x: 150, y: 150 },
          'liver': { health: 78, x: 140, y: 230 },
          'stomach': { health: 83, x: 170, y: 260 },
          'intestine-small': { health: 89, x: 130, y: 330 },
          'intestine-large': { health: 76, x: 150, y: 345 }
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
      
      // Load the Chatbase script
      const script = document.createElement("script");
      script.src = "https://www.chatbase.co/embed.min.js";
      script.id = "MdW4jU0fFrfgF7QWDZFZs";
      script.domain = "www.chatbase.co";
      script.defer = true;
      
      // Add to head instead of body for better performance
      document.head.appendChild(script);
      
      console.log('Chatbase GPT widget initialized');
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
      
      const eventData = {
        label: organName|| '',
        id: "UBERON:0000948" || '',
        featureId: "UBERON:0000948" || '',
        taxonomy: "NCBITaxon:9606" || '',
        resources: ""
      };
      const paramString = transformObjToString(eventData);
      // `transformStringToObj` function can be used to change it back to object
      Tagging.sendEvent({
        'event': 'interaction_event',
        'event_name': 'portal_maps_connectivity',
        'category': paramString,
        "location": ""
      });
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
  gap: 12px;
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
  gap: 8px;
  align-items: center;
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

:deep(.funding-popover.el-popover.el-popper) {
  padding: 20px;
  background: #fff!important;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 1000;
}

.funding-content {
  .funding-title {
    margin: 0 0 16px 0;
    color: #333;
    font-size: 18px;
    font-weight: 600;
    text-align: center;
  }

  .funding-links {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .funding-item {
    display: flex;
    justify-content: center;
  }

  .funding-button {
    display: inline-block;
    color: white;
    padding: 12px 20px;
    text-align: center;
    text-decoration: none;
    font-size: 14px;
    font-weight: 500;
    border-radius: 6px;
    transition: all 0.3s ease;
    min-width: 200px;
    
    &:hover {
      transform: translateY(-1px);
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
      text-decoration: none;
      color: white;
    }
    
    &.subscription-button {
      background-color: #003366;
      
      &:hover {
        background-color: #004488;
      }
    }
    
    &.support-button {
      background-color: #0B3D91;
      
      &:hover {
        background-color: #0D4BA3;
      }
    }
  }
}

</style>

<style src="../../assets/mapicon-species-style.css"></style>