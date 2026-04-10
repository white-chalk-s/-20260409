<template>
  <div class="scene-panel panel-area">
    <!-- 5. 工具栏区 - 悬浮在三维场景内部顶部 -->
    <div class="scene-toolbar">
      <!-- 新增按钮 -->
      <div class="tool-group">
        <div class="tool-btn-main" @click="handleNew">
          <span>➕</span> 新增
        </div>
      </div>
      <div class="tool-divider"></div>

      <!-- 撤回/重做 -->
      <div class="tool-group">
        <div class="tool-icon" title="撤回">↩</div>
        <div class="tool-icon" title="重做">↪</div>
      </div>
      <div class="tool-divider"></div>

      <!-- 变换工具 -->
      <div class="tool-group">
        <div
          v-for="tool in transformTools"
          :key="tool.id"
          :class="['tool-icon', { active: activeTool === tool.id }]"
          :title="tool.label"
          @click="activeTool = tool.id"
        >
          {{ tool.icon }}
        </div>
      </div>
      <div class="tool-divider"></div>

      <!-- 步长设置 -->
      <div class="tool-group">
        <div class="tool-input-group" title="平移步长">
          <div class="tool-input-label">✥ 步长</div>
          <input type="text" class="tool-input" v-model="moveStep" />
        </div>
        <div class="tool-input-group" title="旋转步长">
          <div class="tool-input-label">↻ 角度</div>
          <input type="text" class="tool-input" v-model="rotateStep" />
        </div>
        <div class="tool-input-group" title="缩放步长">
          <div class="tool-input-label">⤢ 缩放</div>
          <input type="text" class="tool-input" v-model="scaleStep" />
        </div>
      </div>
      <div class="tool-divider"></div>

      <!-- 默认视图 -->
      <div class="tool-group">
        <div class="tool-icon home-btn" title="默认视图">🏠</div>
      </div>
    </div>

    <!-- 7. 三维场景区 -->
    <div
      class="scene-viewport"
      :class="{
        'has-focus': focusedDevice,
        'status-view': isStatusTab,
        'drag-over': isDragging,
        'tool-view': isToolTab
      }"
      @dragover.prevent="handleDragOver"
      @dragleave="handleDragLeave"
      @drop="handleDrop"
    >
      <div class="scene-placeholder">
        <div v-if="isToolTab" class="grid-overlay"></div>
        <div v-if="isToolTab" class="viewport-label">3D VIEWPORT</div>
        <!-- 默认标注 -->
        <template v-if="!focusedDevice && !isStatusTab && !isToolTab">
          <div class="annotation annotation-1">楼宇 A 栋</div>
          <div class="annotation annotation-2">传感器节点</div>
          <div class="annotation annotation-3">设备区域</div>
        </template>

        <!-- 选中设备标注 -->
        <template v-if="focusedDevice">
          <div class="device-annotation">
            <div class="annotation-box">
              <div class="annotation-title">{{ focusedDevice.name }}</div>
              <div class="annotation-status">
                <span v-if="focusedDevice.locked" class="status-locked">🔒 已锁定</span>
                <span v-if="!focusedDevice.visible" class="status-hidden">👁️ 已隐藏</span>
              </div>
            </div>
          </div>
        </template>

        <!-- 状态tab坐标显示 -->
        <template v-if="isStatusTab">
          <div class="coord-overlay">
            CAM_XYZ: [142.52, 65.88, -10.04] | FOV: 60.0°
          </div>
        </template>

        <!-- 右下角视角控制器 -->
        <div class="camera-controller">
          <button :class="['cam-btn', { active: activeCamera === 'default' }]" @click="setCamera('default')" title="默认视角">
            <div class="cam-icon">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"></path><polyline points="3.27 6.96 12 12.01 20.73 6.96"></polyline><line x1="12" y1="22.08" x2="12" y2="12"></line></svg>
            </div>
            <span class="cam-text">默认视角</span>
          </button>

          <button :class="['cam-btn', { active: activeCamera === 'top' }]" @click="setCamera('top')" title="俯视视角">
            <div class="cam-icon">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="18" height="18" rx="2"></rect><circle cx="12" cy="12" r="3"></circle></svg>
            </div>
            <span class="cam-text">俯视视角</span>
          </button>

          <button :class="['cam-btn', { active: activeCamera === 'ortho' }]" @click="setCamera('ortho')" title="正交视角">
            <div class="cam-icon">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="3 6 9 3 15 6 21 3 21 18 15 21 9 18 3 21"></polygon><line x1="9" y1="3" x2="9" y2="21"></line><line x1="15" y1="3" x2="15" y2="21"></line></svg>
            </div>
            <span class="cam-text">正交视角</span>
          </button>

          <div class="divider-v"></div>

          <!-- 站厅视角下拉 -->
          <div :class="['dropdown-wrapper', { open: openDropdown === 'hall' }]">
            <button class="cam-btn" @click="toggleDropdown('hall')" title="站厅视角">
              <div class="cam-icon">
                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="12 2 2 7 12 12 22 7 12 2"></polygon><polyline points="2 17 12 22 22 17"></polyline><polyline points="2 12 12 17 22 12"></polyline></svg>
              </div>
              <span class="cam-text">站厅视角 <span class="arrow">▼</span></span>
            </button>
            <div class="drawer-panel">
              <ul class="drawer-list">
                <li :class="['drawer-item', { selected: selectedHall === 'center' }]" @click="selectHall('center')">L1 站厅层中心</li>
                <li :class="['drawer-item', { selected: selectedHall === 'east' }]" @click="selectHall('east')">L1 东侧闸机区</li>
                <li :class="['drawer-item', { selected: selectedHall === 'west' }]" @click="selectHall('west')">L1 西侧服务中心</li>
                <li :class="['drawer-item', { selected: selectedHall === 'transfer' }]" @click="selectHall('transfer')">L2 换乘通道</li>
              </ul>
            </div>
          </div>

          <!-- 漫游路径下拉 -->
          <div :class="['dropdown-wrapper', { open: openDropdown === 'path' }]">
            <button class="cam-btn" @click="toggleDropdown('path')" title="漫游路径">
              <div class="cam-icon">
                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="18" cy="5" r="3"></circle><circle cx="6" cy="12" r="3"></circle><circle cx="18" cy="19" r="3"></circle><line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line><line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line></svg>
              </div>
              <span class="cam-text">漫游路径 <span class="arrow">▼</span></span>
            </button>
            <div class="drawer-panel" style="width: 180px;">
              <ul class="drawer-list">
                <li :class="['drawer-item', { selected: selectedPath === 'default' }]" @click="selectPath('default')">乘客进站流线 (默认)</li>
                <li :class="['drawer-item', { selected: selectedPath === 'security' }]" @click="selectPath('security')">安检-闸机快速流线</li>
                <li :class="['drawer-item', { selected: selectedPath === 'fire', 'danger': true }]" @click="selectPath('fire')">火灾应急疏散路径 A</li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, computed } from 'vue'

const props = defineProps({
  selectedDevice: {
    type: Object,
    default: null
  },
  tabId: {
    type: String,
    default: 'scene'
  },
  subTabId: {
    type: String,
    default: 'object'
  }
})

const emit = defineEmits(['device-change'])

const isDragging = ref(false)

// 视角控制器
const activeCamera = ref('default')
const openDropdown = ref('')
const selectedHall = ref('center')
const selectedPath = ref('default')

function setCamera(cam) {
  activeCamera.value = cam
  openDropdown.value = ''
}

function toggleDropdown(id) {
  openDropdown.value = openDropdown.value === id ? '' : id
}

function selectHall(val) {
  selectedHall.value = val
  activeCamera.value = 'hall'
  openDropdown.value = ''
}

function selectPath(val) {
  selectedPath.value = val
  activeCamera.value = 'path'
  openDropdown.value = ''
}

const activeTool = ref('move')
const moveStep = ref('1.0')
const rotateStep = ref('15°')
const scaleStep = ref('0.1')

const focusedDevice = ref(null)

// 是否是状态tab
const isStatusTab = computed(() => props.tabId === 'status')

// 是否是工具tab
const isToolTab = computed(() => props.tabId === 'tool')

const transformTools = [
  { id: 'move', icon: '✥', label: '平移' },
  { id: 'rotate', icon: '↻', label: '旋转' },
  { id: 'autoRotate', icon: '🌀', label: '自动旋转' },
  { id: 'view3d', icon: '3D', label: '2D/3D切换' }
]

watch(() => props.selectedDevice, (newDevice) => {
  focusedDevice.value = newDevice
}, { immediate: true })

function handleNew() {
  // 新增功能 - 跳转到资产库
  console.log('点击新增按钮')
}

function handleDragOver(event) {
  event.dataTransfer.dropEffect = 'copy'
  isDragging.value = true
}

function handleDragLeave() {
  isDragging.value = false
}

function handleDrop(event) {
  isDragging.value = false
  const deviceData = event.dataTransfer.getData('device')
  if (deviceData) {
    const device = JSON.parse(deviceData)
    focusedDevice.value = device
    emit('device-change', device)
  }
}
</script>

<style scoped>
.scene-panel {
  position: relative;
  background: var(--bg-dark);
  display: flex;
  flex-direction: column;
  overflow: visible;
  pointer-events: none;
}

/* 5. 工具栏 - 悬浮在场景内部顶部 */
.scene-toolbar {
  position: absolute;
  top: 16px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 10;
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 6px 12px;
  background: rgba(30, 41, 59, 0.95);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: var(--radius-lg);
  backdrop-filter: blur(8px);
  pointer-events: auto;
}

.tool-group {
  display: flex;
  align-items: center;
  gap: 4px;
}

.tool-divider {
  width: 1px;
  height: 16px;
  background: rgba(255, 255, 255, 0.2);
  margin: 0 4px;
}

.tool-btn-main {
  background: var(--primary-color);
  color: #fff;
  padding: 4px 10px;
  border-radius: 4px;
  font-size: 12px;
  cursor: pointer;
  border: none;
  display: flex;
  align-items: center;
  gap: 4px;
  transition: 0.2s;
}

.tool-btn-main:hover {
  background: var(--primary-hover);
}

.tool-icon {
  width: 28px;
  height: 28px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 4px;
  cursor: pointer;
  color: #ccc;
  font-size: 14px;
  background: transparent;
  transition: 0.2s;
}

.tool-icon:hover {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
}

.tool-icon.active {
  background: rgba(22, 119, 255, 0.2);
  color: var(--primary-color);
}

.home-btn {
  font-size: 18px;
}

.tool-input-group {
  display: flex;
  align-items: center;
  background: #111;
  border: 1px solid #333;
  border-radius: 4px;
  overflow: visible;
}

.tool-input-label {
  background: #222;
  padding: 0 6px;
  font-size: 11px;
  color: #aaa;
  border-right: 1px solid #333;
  height: 24px;
  line-height: 24px;
}

.tool-input {
  width: 35px;
  background: transparent;
  border: none;
  color: #fff;
  font-size: 11px;
  padding: 0 4px;
  height: 24px;
  outline: none;
  text-align: center;
}

/* 7. 三维场景区 */
.scene-viewport {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  pointer-events: auto;
  background: url('https://minimax-algeng-chat-tts.oss-cn-wulanchabu.aliyuncs.com/ccv2%2F2026-04-09%2FMiniMax-M2.5%2F2031966486800633944%2F7d5cd02b21c1ebf6634589a79125d08a004d30d45a455b927db18a3aa67239f4..png?Expires=1775800594&OSSAccessKeyId=LTAI5tGLnRTkBjLuYPjNcKQ8&Signature=SXjAUIy6N7Jy7044AOWwcQfxVEg%3D') center/cover no-repeat;
}

.scene-viewport.has-focus {
  background: url('https://minimax-algeng-chat-tts.oss-cn-wulanchabu.aliyuncs.com/ccv2%2F2026-04-09%2FMiniMax-M2.5%2F2031966486800633944%2F7d5cd02b21c1ebf6634589a79125d08a004d30d45a455b927db18a3aa67239f4..png?Expires=1775800594&OSSAccessKeyId=LTAI5tGLnRTkBjLuYPjNcKQ8&Signature=SXjAUIy6N7Jy7044AOWwcQfxVEg%3D') center/cover no-repeat, rgba(26, 42, 58, 0.5);
}

.scene-placeholder {
  width: 100%;
  height: 100%;
  position: relative;
}

.focused-model {
  position: relative;
  z-index: 1;
}

.model-box {
  width: 120px;
  height: 160px;
  border: 2px solid var(--primary-color);
  box-shadow: 0 0 20px rgba(22, 119, 255, 0.4);
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding-top: -20px;
  position: relative;
  background: rgba(22, 119, 255, 0.1);
}

.model-box::before {
  content: attr(data-name);
  position: absolute;
  top: -24px;
  left: 50%;
  transform: translateX(-50%);
  background: var(--primary-color);
  color: #fff;
  padding: 2px 8px;
  border-radius: 10px;
  font-size: 11px;
  white-space: nowrap;
}

.model-box.locked {
  border-color: var(--warning-color);
  box-shadow: 0 0 20px rgba(245, 158, 11, 0.4);
}

.annotation {
  position: absolute;
  padding: 6px 12px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: var(--radius-sm);
  font-size: 12px;
  color: var(--text-primary);
  box-shadow: var(--shadow-md);
  z-index: 2;
}

.annotation-1 {
  top: 30%;
  left: 25%;
}

.annotation-2 {
  top: 45%;
  right: 20%;
}

.annotation-3 {
  bottom: 25%;
  left: 40%;
}

.device-annotation {
  position: absolute;
  z-index: 10;
}

.annotation-box {
  background: rgba(255, 255, 255, 0.95);
  border-radius: var(--radius-sm);
  padding: 8px 12px;
  box-shadow: var(--shadow-md);
}

.annotation-title {
  font-size: 12px;
  font-weight: 600;
  color: var(--text-primary);
}

.annotation-status {
  display: flex;
  gap: 8px;
  margin-top: 4px;
  font-size: 11px;
}

.status-locked {
  color: var(--warning-color);
}

.status-hidden {
  color: var(--text-muted);
}

/* 状态tab视图 */
.scene-viewport.status-view {
  background: url('https://minimax-algeng-chat-tts.oss-cn-wulanchabu.aliyuncs.com/ccv2%2F2026-04-09%2FMiniMax-M2.5%2F2031966486800633944%2F7d5cd02b21c1ebf6634589a79125d08a004d30d45a455b927db18a3aa67239f4..png?Expires=1775800594&OSSAccessKeyId=LTAI5tGLnRTkBjLuYPjNcKQ8&Signature=SXjAUIy6N7Jy7044AOWwcQfxVEg%3D') center/cover no-repeat;
}

/* 工具tab视图 */
.scene-viewport.tool-view {
  background: url('https://minimax-algeng-chat-tts.oss-cn-wulanchabu.aliyuncs.com/ccv2%2F2026-04-09%2FMiniMax-M2.5%2F2031966486800633944%2F7d5cd02b21c1ebf6634589a79125d08a004d30d45a455b927db18a3aa67239f4..png?Expires=1775800594&OSSAccessKeyId=LTAI5tGLnRTkBjLuYPjNcKQ8&Signature=SXjAUIy6N7Jy7044AOWwcQfxVEg%3D') center/cover no-repeat, #08080a;
}

.scene-viewport.tool-view .scene-placeholder {
  display: flex;
  align-items: center;
  justify-content: center;
}

.grid-overlay {
  position: absolute;
  inset: 0;
  background-image: radial-gradient(circle, #222 1px, transparent 1px);
  background-size: 24px 24px;
  opacity: 0.5;
}

.viewport-label {
  position: relative;
  z-index: 1;
  font-size: 48px;
  font-weight: 900;
  color: #1a1a1e;
  letter-spacing: 10px;
  pointer-events: none;
}

.scene-viewport.drag-over {
  background: url('https://minimax-algeng-chat-tts.oss-cn-wulanchabu.aliyuncs.com/ccv2%2F2026-04-09%2FMiniMax-M2.5%2F2031966486800633944%2F7d5cd02b21c1ebf6634589a79125d08a004d30d45a455b927db18a3aa67239f4..png?Expires=1775800594&OSSAccessKeyId=LTAI5tGLnRTkBjLuYPjNcKQ8&Signature=SXjAUIy6N7Jy7044AOWwcQfxVEg%3D') center/cover no-repeat, rgba(22, 119, 255, 0.2);
  box-shadow: inset 0 0 30px rgba(22, 119, 255, 0.3);
  border: 2px dashed var(--primary-color);
}

.scene-viewport.status-view .scene-placeholder::after {
  content: '3D REALTIME VIEWPORT';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: #333;
  font-weight: bold;
  font-size: 24px;
  letter-spacing: 2px;
}

.scene-viewport.status-view .coord-overlay {
  position: absolute;
  bottom: 20px;
  left: 20px;
  background: rgba(0, 0, 0, 0.7);
  padding: 10px 16px;
  border-radius: var(--radius-sm);
  font-family: 'Courier New', monospace;
  font-size: 12px;
  color: var(--success-color);
  border: 1px solid #444;
  backdrop-filter: blur(4px);
}

/* 右下角视角控制器 */
.camera-controller {
  position: absolute;
  bottom: 24px;
  right: 24px;
  background: rgba(26, 30, 38, 0.85);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 8px;
  padding: 6px;
  display: flex;
  gap: 6px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
  z-index: 50;
}

.cam-btn {
  background: transparent;
  border: 1px solid transparent;
  color: #9ca3af;
  height: 36px;
  padding: 0 10px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: visible;
}

.cam-btn:hover {
  background: rgba(255, 255, 255, 0.08);
  color: #e5e7eb;
}

.cam-btn.active {
  background: rgba(24, 144, 255, 0.15);
  color: #1890ff;
  border-color: rgba(24, 144, 255, 0.3);
}

.cam-icon {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.cam-icon svg {
  width: 18px;
  height: 18px;
}

.cam-text {
  max-width: 0;
  opacity: 0;
  white-space: nowrap;
  font-size: 13px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  align-items: center;
}

.cam-btn.active .cam-text {
  max-width: 80px;
  opacity: 1;
  margin-left: 8px;
}

.divider-v {
  width: 1px;
  background: #2d3340;
  margin: 4px 2px;
}

.dropdown-wrapper {
  position: relative;
}

.arrow {
  font-size: 10px;
  margin-left: 4px;
  transition: transform 0.2s;
}

.dropdown-wrapper.open .cam-btn {
  background: rgba(24, 144, 255, 0.15);
  color: #1890ff;
  border-color: rgba(24, 144, 255, 0.3);
}

.dropdown-wrapper.open .arrow {
  transform: rotate(180deg);
}

.drawer-panel {
  position: absolute;
  bottom: calc(100% + 10px);
  left: 50%;
  transform: translateX(-50%) translateY(10px);
  background: #1a1e26;
  border: 1px solid #2d3340;
  border-radius: 6px;
  width: 160px;
  box-shadow: 0 -4px 16px rgba(0, 0, 0, 0.3);
  opacity: 0;
  visibility: hidden;
  transition: all 0.2s cubic-bezier(0.16, 1, 0.3, 1);
  z-index: 60;
}

.dropdown-wrapper.open .drawer-panel {
  opacity: 1;
  visibility: visible;
  transform: translateX(-50%) translateY(0);
}

.drawer-panel::after {
  content: '';
  position: absolute;
  bottom: -5px;
  left: 50%;
  transform: translateX(-50%) rotate(45deg);
  width: 10px;
  height: 10px;
  background: #1a1e26;
  border-right: 1px solid #2d3340;
  border-bottom: 1px solid #2d3340;
}

.drawer-list {
  list-style: none;
  padding: 4px 0;
  max-height: 200px;
  overflow-y: auto;
}

.drawer-item {
  padding: 10px 16px;
  font-size: 13px;
  color: #9ca3af;
  cursor: pointer;
  transition: all 0.1s;
}

.drawer-item:hover {
  background: rgba(255, 255, 255, 0.05);
  color: #e5e7eb;
}

.drawer-item.selected {
  color: #1890ff;
  background: rgba(24, 144, 255, 0.05);
}

.drawer-item.danger {
  color: #ef4444;
}
</style>
