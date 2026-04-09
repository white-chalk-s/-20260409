<template>
  <div class="app-container">
    <!-- 顶部区域：标题 + 导航 + 操作按钮 -->
    <div class="header-area">
      <TitlePanel class="area-title" />
      <TabPanel class="area-tabs" @change="handleTabChange" @sub-change="handleSubTabChange" />
      <SettingsPanel class="area-settings" />
    </div>

    <!-- 内容区域 -->
    <div class="content-area">
      <!-- 3. 对象区 -->
      <ObjectPanel
        class="area-object"
        :tab-id="activeTab"
        :sub-tab-id="activeSubTab"
        @device-change="handleDeviceChange"
      />

      <!-- 5. 工具栏区 + 7. 三维场景区 -->
      <ScenePanel
        class="area-scene"
        :selected-device="selectedDevice"
        :tab-id="activeTab"
        :sub-tab-id="activeSubTab"
        @device-change="handleDeviceChange"
      />

      <!-- 6. 属性配置区 -->
      <PropertyPanel
        class="area-property"
        :tab-id="activeTab"
        :sub-tab-id="activeSubTab"
        :selected-device="selectedDevice"
      />
    </div>

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
  display: flex;
  flex-direction: column;
  height: 100vh;
  background: var(--border-color);
}

.header-area {
  display: grid;
  grid-template-columns: 300px 1fr 300px;
  grid-template-rows: 50px 38px;
  flex-shrink: 0;
}

.content-area {
  display: grid;
  grid-template-columns: 300px 1fr 320px;
  grid-template-rows: 1fr;
  flex: 1;
  overflow: hidden;
}

.area-title {
  grid-column: 1;
  grid-row: 1 / span 2;
}

.area-tabs {
  grid-column: 2;
  grid-row: 1 / span 2;
}

.area-settings {
  grid-column: 3;
  grid-row: 1 / span 2;
}

.area-object {
  grid-column: 1;
  grid-row: 1;
}

.area-scene {
  grid-column: 2;
  grid-row: 1;
}

.area-property {
  grid-column: 3;
  grid-row: 1;
}

.area-footer {
  flex-shrink: 0;
}
</style>
