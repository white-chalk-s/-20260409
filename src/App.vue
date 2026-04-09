<template>
  <div class="app-container">
    <!-- 1. 标题展示区 -->
    <TitlePanel class="area-title" />

    <!-- 2. 状态 Tab 切换区 -->
    <TabPanel
      class="area-tabs"
      @change="handleTabChange"
      @sub-change="handleSubTabChange"
    />

    <!-- 3. 对象区 -->
    <ObjectPanel
      class="area-object"
      :tab-id="activeTab"
      :sub-tab-id="activeSubTab"
      @device-change="handleDeviceChange"
    />

    <!-- 4. 默认设置区 -->
    <SettingsPanel class="area-settings" />

    <!-- 5. 工具栏区 + 7. 三维场景区 -->
    <ScenePanel
      class="area-scene"
      :selected-device="selectedDevice"
    />

    <!-- 6. 属性配置区 -->
    <PropertyPanel
      class="area-property"
      :tab-id="activeTab"
      :sub-tab-id="activeSubTab"
      :selected-device="selectedDevice"
    />

    <!-- 8. 杂七杂八信息区 -->
    <FooterPanel class="area-footer" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import TitlePanel from './components/panels/TitlePanel.vue'
import TabPanel from './components/panels/TabPanel.vue'
import ObjectPanel from './components/panels/ObjectPanel.vue'
import SettingsPanel from './components/panels/SettingsPanel.vue'
import ScenePanel from './components/panels/ScenePanel.vue'
import PropertyPanel from './components/panels/PropertyPanel.vue'
import FooterPanel from './components/panels/FooterPanel.vue'

const activeTab = ref('scene')
const activeSubTab = ref('object')
const selectedDevice = ref(null)

function handleTabChange(tabId) {
  activeTab.value = tabId
}

function handleSubTabChange(subTabId) {
  activeSubTab.value = subTabId
}

function handleDeviceChange(device) {
  selectedDevice.value = device
}
</script>

<style scoped>
.app-container {
  display: grid;
  grid-template-columns: 300px 1fr 320px;
  grid-template-rows: 50px 38px 1fr 32px;
  grid-template-areas:
    "title   tabs    settings"
    "title   tabs    settings"
    "object  scene   property"
    "footer  footer  footer";
  height: 100vh;
  background: var(--border-color);
  gap: 1px;
}

.area-title { grid-area: title; }
.area-tabs { grid-area: tabs; }
.area-object { grid-area: object; }
.area-settings { grid-area: settings; }
.area-scene { grid-area: scene; }
.area-property { grid-area: property; }
.area-footer { grid-area: footer; }
</style>
