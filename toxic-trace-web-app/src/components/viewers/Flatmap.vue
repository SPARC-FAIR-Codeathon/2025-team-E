<template>
  <div class="viewer-container">
    HELOLOLOLO
    KASDFASD
    HLEOELEHLOE
    HELO

    <HelpModeDialog
      v-if="helpMode && useHelpModeDialog"
      ref="flatmapHelp"
      :flatmapRef="flatmapRef"
      :lastItem="helpModeLastItem"
      @show-next="onHelpModeShowNext"
      @finish-help-mode="onFinishHelpMode"
    />
  </div>
</template>

<script>
/* eslint-disable no-alert, no-console */
import Tagging from '../../services/tagging.js';
import EventBus from "../EventBus";
import ContentMixin from "../../mixins/ContentMixin";
import DynamicMarkerMixin from "../../mixins/DynamicMarkerMixin";
import { transformObjToString } from '../scripts/utilities';

import { FlatmapVuer } from "@abi-software/flatmapvuer";
import "@abi-software/flatmapvuer/dist/style.css";
import { HelpModeDialog } from '@abi-software/map-utilities'
import '@abi-software/map-utilities/dist/style.css'

export default {
  name: "Flatmap",
  mixins: [ ContentMixin, DynamicMarkerMixin ],
  components: {
    FlatmapVuer,
    HelpModeDialog,
  },
  data: function () {
    return {
      flatmapReady: true,
    }
  },
  methods: {
    getState: function () {
      return {}
    },
    /**
     * Perform a local search on this contentvuer
     */
    search: function (term) {
      return false
    },
    getFlatmapImp() {
      return null
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
          "location": type + ' ' + this.$refs.flatmap.viewingMode
        });
      }
    },
    flatmapReadyCall: function () {
      this.flatmapReady = true;
      let provClone = {id: this.entry?.id, prov: null}; //create clone of provenance and add id
      const flatmapImp = null
      EventBus.emit("mapImpProv", provClone); // send clone to context card
      this.$emit("flatmap-provenance-ready", provClone);
      this.updateViewerSettings();
      EventBus.emit("mapLoaded", flatmap);
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
      return null;
    },
    highlightFeatures: function(info) {
      return null;
    },
    /**
     * Append the list of suggested terms to suggestions
     */
    searchSuggestions: function (term, suggestions) {
      return null;
    },
    showConnectivity: function (payload) {
      return null;
    },
    showConnectivitiesByReference: function (payload) {
      return null;
    },
    zoomToFeatures: function(info, forceSelect) {
      return null;
    },
    changeViewingMode: function (modeName) {
      return null;
    },
    updateViewerSettings: function () {
      return null;
    },
    setVisibilityFilter: function (payload) {
      return null;
    },
  },
  computed: {
    facetSpecies() {
      return 'pig'
    },
  },
  onMounted() {
    this.flatmapReadyCall()
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.viewer-container {
  width: 100%;
  height: 100%;
}

:deep(.maplibregl-popup) {
  z-index: 11;
}

:deep(.flatmapvuer-popover) {
  .maplibregl-popup-content {
    border-radius: 4px;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
    pointer-events: auto;
    width: 25em;
    background: #fff;
  }
}

:deep(.maplibregl-marker) {
  &.standard-marker {
    z-index: 2;
  }
  &.highlight-marker {
    z-index: 1;
  }
}
</style>
