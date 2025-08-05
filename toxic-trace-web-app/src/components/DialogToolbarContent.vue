<template>
  <div class="header">
    <map-svg-sprite-color />
    
    <h1 class="toxic-trace-title">toxic-trace</h1>

    <el-row class="icon-group">
      
    

      <el-popover class="tooltip" content="Toxin Database" placement="bottom-end" :show-after="helpDelay"
        :teleported=false trigger="hover" popper-class="header-popper" >
        <template #reference>
          <el-icon class="header-icon" @click="openToxinSidebar()">
            <ElIconFilter />
          </el-icon>
        </template>
      </el-popover>
      <el-popover class="tooltip" content="Help" placement="bottom-end" :show-after="helpDelay"
        :teleported=false trigger="hover" popper-class="header-popper" >
        <template #reference>
          <map-svg-icon icon="tooltips" class="header-icon" @click="startHelp()"/>
        </template>
      </el-popover>
      <el-popover class="tooltip"
        content="Fullscreen" placement="bottom-end" :show-after="helpDelay"
        :teleported=false trigger="hover" popper-class="header-popper">
        <template #reference>
          <map-svg-icon v-show="!isFullscreen" icon="fullScreen" class="header-icon" @click="onFullscreen"/>
        </template>
      </el-popover>
      <el-popover class="tooltip"
        content="Exit fullscreen" placement="bottom-end" :show-after="helpDelay"
        :teleported=false trigger="hover" popper-class="header-popper">
        <template #reference>
          <map-svg-icon v-show="isFullscreen" icon="closeFullScreen" class="header-icon"
            @click="onFullscreen"/>
        </template>
      </el-popover>
      <el-popover
        v-if="permalinkRef"
        ref="linkPopover"
        :virtual-ref="permalinkRef"
        placement="bottom-end"
        width="400"
        :teleported=false
        trigger="click"
        popper-class="link-popover"
        virtual-triggering
      >
        <template v-if="displayShareOptions">
          <el-row>
            <el-col :offset="3" :span="8">
              <el-button
                type="primary"
                size="small"
                @click="getShareLink(exportAnnotation)"
                class="share-options"
              >
                Create Permalink
              </el-button>
            </el-col>
            <el-col :span="10">
              <el-popover class="tooltip"
                placement="bottom-end"
                :show-after="helpDelay" :teleported=false trigger="hover"
                popper-class="header-popper"
              >
                <template #reference>
                  <el-checkbox
                    v-model="exportAnnotation"
                    size="small"
                  >
                    Export Annotations
                  </el-checkbox>
                </template>
                <template #default>
                  Create a permalink with anonymous annotations.
                  <br>
                  NOTE: Annotations will only be stored for
                  <br>
                  30 days on the server.
                </template>
              </el-popover>
            </el-col>
        </el-row>
        </template>
        <template v-else>
          <el-row :gutter="20"
            v-loading="loadingLink"
            element-loading-text="Creating link...">
            <el-col :span="20">
              <el-input
                class="link-input"
                size="small"
                placeholder="Permanant Link Here"
                :readonly=true
                v-model="shareLink"
                ref="linkInput">
              </el-input>
            </el-col>
            <el-col :span="4">
              <el-popover class="tooltip" content="Copy link" placement="bottom-end"
                :show-after="helpDelay" :teleported=false trigger="hover"
                popper-class="header-popper">
                <template #reference>
                  <el-button class="copy-button"
                    :icon="ElIconCopyDocument" size="small"
                    @click="copyShareLink"></el-button>
                </template>
              </el-popover>
            </el-col>
          </el-row>
        </template>
      </el-popover>
      
      <el-popover class="tooltip" content="Close" placement="bottom-end" :show-after="helpDelay"
        :teleported=false trigger="hover" popper-class="header-popper">
        <template #reference>
          <map-svg-icon icon="close" class="header-icon" @click="close" v-show="showIcons"/>
        </template>
      </el-popover>

      <el-popover
        v-if="globalSettingRef"
        :virtual-ref="globalSettingRef"
        ref="settingPopover"
        placement="bottom"
        width="230"
        :teleported=false
        trigger="click"
        popper-class="setting-popover"
        virtual-triggering
        :disabled="!mapLoaded"
        >
        <div class="setting-popover-inner">
          <h4>Display options:</h4>

          <div class="setting-popover-block" v-if="'flightPathDisplay' in globalSettings">
            <h5>Flight path</h5>
            <el-radio-group
              v-model="globalSettings.flightPathDisplay"
              @change="updateGlobalSettings"
            >
              <el-radio :value="false">2D</el-radio>
              <el-radio :value="true">3D</el-radio>
            </el-radio-group>
          </div>
          <div class="setting-popover-block" v-if="'organsDisplay' in globalSettings">
            <h5>Organs</h5>
            <el-radio-group
              v-model="globalSettings.organsDisplay"
              @change="updateGlobalSettings"
            >
              <el-radio :value="true">Color</el-radio>
              <el-radio :value="false">Grayscale</el-radio>
            </el-radio-group>
          </div>
          <div class="setting-popover-block" v-if="'outlinesDisplay' in globalSettings">
            <h5>Apply outlines</h5>
            <el-radio-group
              v-model="globalSettings.outlinesDisplay"
              @change="updateGlobalSettings"
            >
              <el-radio :value="true">Show</el-radio>
              <el-radio :value="false">Hide</el-radio>
            </el-radio-group>
          </div>
          <div class="setting-popover-block" v-if="'backgroundDisplay' in globalSettings">
            <h5>Background color</h5>
            <el-radio-group
              class="bg-color-radio-group"
              v-model="globalSettings.backgroundDisplay"
              @change="updateGlobalSettings"
            >
              <el-radio value="white" class="bg-color-radio">
                <span style="--bg-color: white">white</span>
              </el-radio>
              <el-radio value="lightskyblue" class="bg-color-radio">
                <span style="--bg-color: lightskyblue">lightskyblue</span>
              </el-radio>
              <el-radio value="black" class="bg-color-radio">
                <span style="--bg-color: black">black</span>
              </el-radio>
            </el-radio-group>
          </div>
        </div>
      </el-popover>
      <el-popover
        v-if="showGlobalSettings"
        class="tooltip"
        content="Global Settings"
        placement="bottom-end"
        :show-after="helpDelay"
        :teleported=false
        trigger="hover"
        popper-class="header-popper"
        :disabled="!mapLoaded"
      >
        <template #reference>
          <el-icon
            ref="globalSettingRef"
            :disabled="!mapLoaded"
            class="header-icon"
            :class="{'disabled': !mapLoaded}"
          >
            <el-icon-more-filled />
          </el-icon>
        </template>
      </el-popover>

    </el-row>
  </div>
</template>


<script>
/* eslint-disable no-alert, no-console */
import { shallowRef } from 'vue';
import EventBus from './EventBus';
import { mapStores } from 'pinia';
import { useEntriesStore } from '../stores/entries';
import { useSettingsStore } from '../stores/settings';
import { useSplitFlowStore } from '../stores/splitFlow';
import { MapSvgIcon, MapSvgSpriteColor } from '@abi-software/svg-sprite';
import SearchControls from './SearchControls.vue';
import {
  CopyDocument as ElIconCopyDocument,
  MoreFilled as ElIconMoreFilled,
  Filter as ElIconFilter,
} from '@element-plus/icons-vue';
import {
  ElButton as Button,
  ElCheckbox as Checkbox,
  ElCol as Col,
  ElIcon as Icon,
  ElInput as Input,
  ElPopover as Popover,
  ElRadio as Radio,
  ElRadioGroup as RadioGroup,
  ElRow as Row,
} from "element-plus";

/**
 * Cmponent for the header of differnt vuers.
 */
export default {
  name: "DialogToolbarContent",
  components: {
    Button,
    Checkbox,
    Col,
    Icon,
    Input,
    Popover,
    Radio,
    RadioGroup,
    Row,
    MapSvgIcon,
    MapSvgSpriteColor,
    SearchControls,
    ElIconCopyDocument,
    ElIconFilter,
  },
  props: {
    /**
     * Array of titles.
     */
    numberOfEntries: {
      type: Number,
      default: 0
    },
    /**
     * Display icons for docking, undocking and etc.
     */
    showIcons: {
      type: Boolean,
      default: false

    },
  },
  inject: ['showGlobalSettings'],
  computed: {
    ...mapStores(useEntriesStore, useSettingsStore, useSplitFlowStore),
    activeView() {
      return this.splitFlowStore.activeView;
    },
    helpDelay() {
      return this.settingsStore.helpDelay;
    },
    shareLink() {
      return this.settingsStore.shareLink;
    },
    offlineAnnotationEnabled() {
      return this.settingsStore.offlineAnnotationEnabled;
    },
    viewIcons() {
      return this.splitFlowStore.viewIcons;
    },
  },
  watch: {
    shareLink: function() {
      this.loadingLink = false;
    },
  },
  data: function() {
    return {
      activeViewRef: undefined,
      displayShareOptions: false,
      ElIconCopyDocument: shallowRef(ElIconCopyDocument),
      exportAnnotation: false,
      failedSearch: undefined,
      globalSettings: {},
      globalSettingRef: undefined,
      isFullscreen: false,
      loadingLink: true,
      permalinkRef: undefined,
      viewingModes: {
        'Exploration': 'Find relevant research and view detail of neural pathways by selecting a pathway to view its connections and data sources',
        'Neuron Connection': 'Discover Neuron connections by selecting a neuron and viewing its associated network connections',
        'Annotation': ['View feature annotations', 'Add, comment on and view feature annotations']
      },
      authorisedUser: false,
      mapLoaded: false,
    }
  },
  methods: {
    loadGlobalSettings: function () {
      this.globalSettings = {
        ...this.globalSettings,
        ...this.settingsStore.globalSettings
      };
    },
    updateViewingMode: function (event, value) {
      const { target } = event;

      // prevent clicking inner checkbox
      if (!target.closest('.el-checkbox')) {
        this.globalSettings.viewingMode = value;

        if (value === 'Exploration') {
          this.globalSettings.displayMarkers = true;
          this.globalSettings.interactiveMode = 'dataset';
        } else if (value === 'Annotation') {
          this.globalSettings.displayMarkers = false;
          this.globalSettings.interactiveMode = 'dataset';
        } else {
          this.globalSettings.displayMarkers = false;
          this.globalSettings.interactiveMode = 'connectivity';
        }

        this.updateGlobalSettings();
      }
    },
    updateGlobalSettings: function() {
      const updatedSettings = this.settingsStore.getUpdatedGlobalSettingsKey(this.globalSettings);
      this.settingsStore.updateGlobalSettings(this.globalSettings);

      // display marker update
      if (updatedSettings.includes('displayMarkers')) {
        EventBus.emit('markerUpdate');
      }
      if (updatedSettings.includes('interactiveMode')) {
        EventBus.emit('modeUpdate', this.globalSettings.interactiveMode);
      }
      // viewing mode update
      if (updatedSettings.includes('viewingMode') ||
        updatedSettings.includes('flightPathDisplay') ||
        updatedSettings.includes('organsDisplay') ||
        updatedSettings.includes('outlinesDisplay') ||
        updatedSettings.includes('backgroundDisplay')) {
        EventBus.emit('globalViewerSettingsUpdate');
      }
    },
    titleClicked: function(id) {
      this.$emit("titleClicked", id);
    },
    startHelp: function(){
      EventBus.emit("openChatbot");
    },
    onFullscreen: function() {
      this.$emit("onFullscreen");
      this.isFullscreen = !this.isFullscreen;
    },
    onFullscreenEsc: function () {
      if (!document.fullscreenElement) {
        this.isFullscreen = false;
      }
    },
    close: function() {
      this.$emit("close");
    },
    openToxinSidebar: function() {
      EventBus.emit("openToxinSidebar");
    },
    copyShareLink: function() {
      if (document) {
        this.$refs.linkInput.$el.querySelector("input").select();
        document.execCommand('copy');
      }
    },
    setFailedSearch: function(result) {
      this.failedSearch = result;
    },
    requestShareLink: function() {
      if (sessionStorage.getItem('anonymous-annotation')) {
        this.displayShareOptions = true;
      } else {
        this.getShareLink(false);
      }
    },
    getShareLink: function(withAnnotation) {
      this.displayShareOptions = false;
      this.loadingLink = true;
      EventBus.emit("updateShareLinkRequested", withAnnotation);
    },
    viewClicked: function(view) {
      this.splitFlowStore.updateActiveView({
        view,
        entries: this.entriesStore.entries,
      });

      if (this.$refs.viewPopover) {
        this.$refs.viewPopover.hide();
      }
    }
  },
  mounted: function () {
    this.activeViewRef = shallowRef(this.$refs.activeViewRef);
    this.permalinkRef = shallowRef(this.$refs.permalinkRef);
    this.globalSettingRef = shallowRef(this.$refs.globalSettingRef);

    EventBus.on("mapLoaded", (map) => {
      this.mapLoaded = true;
    });

    document.addEventListener('fullscreenchange', this.onFullscreenEsc);

    this.loadGlobalSettings();
  },
  unmounted: function () {
    document.removeEventListener('fullscreenchange', this.onFullscreenEsc);
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
@use "../assets/header-icon.scss";

.icon-group {
  :deep(.el-button--text) {
    color:#606266;
    font-size: 1.5em;
    &:hover {
      color: $app-primary-color;
    }
  }
}

.icon-transform {
  -moz-transform: rotate(90deg);
  -webkit-transform: rotate(90deg);
  -o-transform: rotate(90deg);
  -ms-transform: rotate(90deg);
  transform: rotate(90deg);
  padding:10px;
}

.header {
  height:32px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 1rem;
}

.toxic-trace-title {
  font-size: 24px;
  font-weight: 900;
  color: $app-primary-color;
  margin: 0;
  padding: 0;
  letter-spacing: 1px;
  text-shadow: 2px 2px 4px rgba(123, 44, 191, 0.3);
  background: linear-gradient(135deg, #7b2cbf 0%, #9d4edd 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-family: 'Arial Black', 'Helvetica Neue', Arial, sans-serif;
}

.share-options.el-button {
  font-family: inherit;

  &:hover,
  &:focus {
    background: $app-primary-color;
    box-shadow: -3px 2px 4px #00000040;
    color: #fff;
  }
}

:deep(.header-popper.el-popover.el-popper) {
  padding: 6px 4px;
  font-size:12px;
  color: rgb(48, 49, 51);
  background-color: #f3ecf6;
  border: 1px solid $app-primary-color;
  white-space: nowrap;
  min-width: unset;

  .el-popper__arrow {
    &:before {
      border-color: $app-primary-color;
      background-color: #f3ecf6;
    }
  }
}

:deep(.link-popover) {
  border: 1px solid $app-primary-color;
}

:deep(.el-loading-spinner) {
  .path {
    stroke: $app-primary-color;
  }
  .el-loading-text {
    color: $app-primary-color;
  }
}

.copy-button {
  color:#FFFFFF;
  background-color:$app-primary-color;
  &:hover, &:focus {
    color:#FFFFFF;
    background-color:$app-primary-color;
    box-shadow: -3px 2px 4px #000000;
  }
}

.link-input {
  :deep(.el-input__inner) {
    color:#303133;
    &:focus {
      border-color:$app-primary-color;
    }
  }
}

.view-icon-row {
  height: 32px;
  width: 133px;
  border-radius: 4px;
  border: 1px solid rgb(151, 151, 151);
  font-size: 14px;
  margin:8px 0px 0px 0px!important;
  cursor: pointer;

  &.active {
    border: 1px solid $app-primary-color;
    background: rgba(131, 0, 191, 0.1);
  }
}

.view-icon {
  font-size: 1.7em;
  height: 24px!important;
  width: 24px!important;
  color: $app-primary-color;
  padding-top:3px;
}

.view-text {
  letter-spacing:0px;
  font-size:11px;
  line-height:14px;
  font-family:'Asap', 'Avenir',  Arial, sans-serif;
  font-weight:550;
  padding-top:7px;
}

:deep(.view-icon-popover.el-popper),
:deep(.setting-popover.el-popper) {
  border: 1px solid $app-primary-color;
  box-shadow: 0px 2px 12px 0px rgba(0, 0, 0, 0.06);
  padding: 4px 8px 12px 8px;
  min-width:unset!important;
  width:unset!important;
  background-color: #f3ecf6;
  cursor:default;
  .el-popper__arrow {
    &:before {
      border-color: $app-primary-color;
      background-color: #f3ecf6;
    }
  }
}

:deep(.setting-popover.el-popper) {
  min-width: 230px !important;
}

.viewing-mode-selector {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 0.5rem;
}

:deep(.toolbar-dropdown-dropdown.el-popper) {
  border: 1px solid $app-primary-color;
  box-shadow: 0px 2px 12px 0px rgba(0, 0, 0, 0.06);
  background-color: #f3ecf6;
  width: 300px;

  .el-popper__arrow {
    &:before {
      border-color: $app-primary-color;
      background-color: #f3ecf6;
    }
  }

  .el-dropdown-menu {
    background-color: #f3ecf6;
  }

  .el-dropdown-menu__item {
    padding: 0.5rem;
    height: auto;
    color: $app-primary-color;
    flex-direction: column;
    align-items: start;
    gap: 0.5rem;
    position: relative;
    transition: all 0.3s ease;

    + .el-dropdown-menu__item {
      &::before {
        content: "";
        display: block;
        width: calc(100% - 1rem);
        border-top: 1px solid var(--el-border-color);
        position: absolute;
        top: 0;
        left: 0.5rem;
      }
    }

    .el-icon {
      display: inline;
      vertical-align: middle;
    }

    > span,
    > small {
      display: block;
    }

    > span {
      line-height: 1.5;
      font-weight: 500;
    }

    > small {
      height: auto;
      line-height: 1.2;
      font-weight: normal;
      color: gray;
      white-space: normal;
    }

    &.is-selected,
    &:hover {
      background-color: #f1e4f6;
    }

    &.is-selected {
      > span {
        font-weight: 700;
      }
    }
  }

  &:hover {
    .el-dropdown-menu__item {
      opacity: 1;

      &:not(:hover) {
        opacity: 0.5;
      }
    }
  }
}

:deep(.setting-popover.el-popper) {
  padding: 1px !important;
}

.setting-popover-inner {
  padding: 0.5rem 0.75rem;
  max-height: calc(100vh - 135px);
  overflow-y: auto;
  border-radius: var(--el-popover-border-radius);
  scrollbar-width: thin;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;

  > h4 {
    margin: 0;
    padding: 0;
    font-size: 16px;
    color: $app-primary-color;
    text-align: center;
  }
}

.setting-popover-block {
  background-color: rgba(0, 0, 0, 0.05);
  padding: 0.5rem 0.75rem;
  border-radius: 4px;

  h5 {
    margin: 0;
    padding: 0;
    font-size: 14px;
    font-weight: 500;
    line-height: 32px;
    color: #303133;
  }
}

:deep(.el-loading-spinner) {
  top: 0px;
  scale: 0.7;
}

.el-radio__description {
  font-size: 12px;
}

.bg-color-radio-group {
  display: flex;
  gap: 0.5rem;
  margin-top: 0.25rem;

  .el-radio {
    margin-right: 0;
    line-height: 0px;
  }
}

.bg-color-radio {
  position: relative;

  :deep(.el-radio__input) {
    display: none;
  }

  :deep(.el-radio__label) {
    height: auto;
    line-height: 0;
    padding: 4px;

    > span {
      display: block;
      width: 20px;
      height: 20px;
      background-color: var(--bg-color);
      box-shadow: 0px 0px 0px 1px rgb(144, 147, 153);
      font-size: 0px;
      line-height: 0;
      transition: box-shadow 0.3s ease;
    }
  }

  &.is-checked {
    :deep(.el-radio__label) > span {
      background-color: var(--bg-color);
      box-shadow: 0px 0px 0px 2px $app-primary-color;
    }
  }
}

.toolbar-dropdown {
  .el-dropdown-link {
    cursor: pointer;
    color: $app-primary-color;
    display: flex;
    align-items: center;
    height: 24px;
    font-weight: 500;
    outline: none;
  }

  &.is-disabled {
    opacity: 0.5;

    .el-dropdown-link {
      cursor: default;
    }
  }

  :deep(.el-icon) {
    color: $app-primary-color;
  }
}
</style>
