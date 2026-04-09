<template>
  <div class="tab-panel panel-area">
    <!-- 主Tab（居中） -->
    <div class="main-tabs">
      <div
        v-for="tab in mainTabs"
        :key="tab.id"
        :class="['tab-item', { active: activeMainTab === tab.id }]"
        @click="handleTabClick(tab.id)"
      >
        {{ tab.label }}
      </div>
    </div>

    <!-- 场景子Tab -->
    <div class="sub-tabs" v-if="activeMainTab === 'scene'">
      <div
        v-for="subTab in sceneSubTabs"
        :key="subTab.id"
        :class="['sub-tab-item', { active: activeSubTab === subTab.id }]"
        @click="handleSubTabClick(subTab.id)"
      >
        {{ subTab.label }}
      </div>
    </div>

    <!-- 模型子Tab -->
    <div class="sub-tabs" v-if="activeMainTab === 'model'">
      <div
        v-for="subTab in modelSubTabs"
        :key="subTab.id"
        :class="['sub-tab-item', { active: activeSubTab === subTab.id }]"
        @click="handleSubTabClick(subTab.id)"
      >
        {{ subTab.label }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['change', 'sub-change'])

const mainTabs = [
  { id: 'scene', label: '场景' },
  { id: 'model', label: '模型' },
  { id: 'status', label: '状态' },
  { id: 'asset', label: '资产' },
  { id: 'tool', label: '工具' }
]

// 场景子Tab
const sceneSubTabs = [
  { id: 'object', label: '对象' },
  { id: 'light', label: '灯光' },
  { id: 'effect', label: '特效' }
]

// 模型子Tab
const modelSubTabs = [
  { id: 'material', label: '材质' },
  { id: 'animation', label: '动画' }
]

const activeMainTab = ref('scene')
const activeSubTab = ref('object')

function handleTabClick(tabId) {
  activeMainTab.value = tabId
  // 切换主Tab时设置默认子Tab
  if (tabId === 'scene') {
    activeSubTab.value = 'object'
  } else if (tabId === 'model') {
    activeSubTab.value = 'material'
  }
  emit('change', tabId)
}

function handleSubTabClick(subTabId) {
  activeSubTab.value = subTabId
  emit('sub-change', subTabId)
}
</script>

<style scoped>
.tab-panel {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 8px 16px;
  background: var(--bg-header);
  border-bottom: 1px solid var(--border-color);
}

.main-tabs {
  display: flex;
  gap: 4px;
  background: var(--bg-base);
  padding: 3px;
  border-radius: var(--radius-md);
  border: 1px solid var(--border-light);
}

.tab-item {
  padding: 5px 18px;
  font-size: 13px;
  color: var(--text-secondary);
  border-radius: var(--radius-sm);
  cursor: pointer;
  transition: all 0.2s;
}

.tab-item:hover {
  background: var(--bg-hover);
  color: var(--text-primary);
}

.tab-item.active {
  background: var(--bg-hover);
  color: var(--text-primary);
  font-weight: 500;
  position: relative;
}

.tab-item.active::after {
  content: '';
  position: absolute;
  bottom: -3px;
  left: 50%;
  transform: translateX(-50%);
  width: 20px;
  height: 2px;
  background: var(--primary-color);
  border-radius: 1px;
}

/* 子Tab定位到主Tab下方居中 */
.sub-tabs {
  display: flex;
  gap: 20px;
  margin-top: 8px;
}

.sub-tab-item {
  padding: 8px 4px;
  font-size: 13px;
  color: var(--text-secondary);
  cursor: pointer;
  position: relative;
  transition: 0.2s;
}

.sub-tab-item:hover {
  color: var(--text-primary);
}

.sub-tab-item.active {
  color: var(--primary-color);
  font-weight: bold;
}

.sub-tab-item.active::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 2px;
  background: var(--primary-color);
}
</style>
