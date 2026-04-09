<template>
  <div class="object-panel panel-area">
    <!-- 车站/设备切换（仅在场景-对象子Tab时显示） -->
    <div class="left-sub-tabs" v-if="tabId === 'scene' && subTabId === 'object'">
      <div
        :class="['left-sub-tab', { active: activeLeftTab === 'station' }]"
        @click="activeLeftTab = 'station'"
      >
        车站
      </div>
      <div
        :class="['left-sub-tab', { active: activeLeftTab === 'device' }]"
        @click="activeLeftTab = 'device'"
      >
        设备
      </div>
    </div>

    <!-- 模型-材质子Tab：节点树 -->
    <div v-if="tabId === 'model' && subTabId === 'material'" class="view-content model-view">
      <div class="node-panel-header">
        <div class="panel-title">节点设置</div>
        <div class="header-actions">░ ◎</div>
      </div>
      <div class="node-tree">
        <div
          v-for="node in modelNodes"
          :key="node.id"
          :class="['node-item', { selected: selectedNodeId === node.id, folder: node.isFolder }]"
          :style="{ paddingLeft: node.level ? node.level * 24 + 'px' : '12px' }"
          @click="selectNode(node)"
        >
          <span class="node-icon">{{ node.isFolder ? (node.collapsed ? '+' : '−') : '' }}</span>
          <span class="node-name">{{ node.isFolder ? '📁 ' + node.name : node.name }}</span>
          <div class="node-actions">
            <span v-if="node.target" class="action-icon active-target">🎯</span>
            <span :class="['action-icon', { 'active-eye': node.visible }]">👁</span>
            <span :class="['action-icon', { 'active-lock': node.locked }]">🔒</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 模型-动画子Tab：设备列表 -->
    <div v-if="tabId === 'model' && subTabId === 'animation'" class="view-content model-view">
      <div class="model-tree-header">
        <div class="toggle-row">
          <button :class="['toggle-btn', { active: activeDimension === 'space' }]" @click="activeDimension = 'space'">按空间</button>
          <button :class="['toggle-btn', { active: activeDimension === 'system' }]" @click="activeDimension = 'system'">按系统</button>
        </div>
        <select class="system-select" v-model="selectedFloor">
          <option v-for="floor in floors" :key="floor.value" :value="floor.value">
            {{ floor.label }}
          </option>
        </select>
      </div>
      <div class="device-tree">
        <!-- 空间维度 -->
        <template v-if="activeDimension === 'space'">
          <div
            v-for="group in spaceGroups"
            :key="group.id"
            :class="['tree-node', { folder: true, collapsed: group.collapsed }]"
            @click="group.collapsed = !group.collapsed"
          >
            <span class="node-arrow">{{ group.collapsed ? '▶' : '▼' }}</span>
            <span class="node-icon">📁</span>
            <span class="node-name">{{ group.name }}</span>
          </div>
          <template v-if="!spaceGroups[0].collapsed">
            <div
              v-for="device in devices"
              :key="device.id"
              :class="['tree-node', { selected: selectedDeviceId === device.id }]"
              @click="selectDevice(device)"
            >
              <span class="node-icon">💠</span>
              <span class="node-name">{{ device.name }}</span>
              <div class="node-actions">
                <span class="action-icon">🎯</span>
                <span :class="['action-icon', { 'active-lock': device.locked }]">🔒</span>
                <span :class="['action-icon', { 'active-eye': device.visible }]">👁</span>
              </div>
            </div>
          </template>
          <div
            v-for="group in spaceGroups.slice(1)"
            :key="group.id"
            :class="['tree-node', { folder: true, collapsed: group.collapsed }]"
            @click="group.collapsed = !group.collapsed"
          >
            <span class="node-arrow">{{ group.collapsed ? '▶' : '▼' }}</span>
            <span class="node-icon">📁</span>
            <span class="node-name">{{ group.name }}</span>
          </div>
          <template v-if="!spaceGroups[1].collapsed">
            <div class="tree-node">
              <span class="node-icon">📷</span>
              <span class="node-name">监控摄像头_Cam01</span>
            </div>
            <div class="tree-node">
              <span class="node-icon">📷</span>
              <span class="node-name">监控摄像头_Cam02</span>
            </div>
          </template>
          <div class="tree-node folder">
            <span class="node-arrow">▶</span>
            <span class="node-icon">📁</span>
            <span class="node-name">自动售票设备</span>
          </div>
          <div class="tree-node">
            <span class="node-icon">🎫</span>
            <span class="node-name">售票机_TVM_01</span>
          </div>
        </template>
        <!-- 系统维度 -->
        <template v-else>
          <div
            v-for="group in systemGroups"
            :key="group.id"
            :class="['tree-node', { folder: true, collapsed: group.collapsed }]"
            @click="group.collapsed = !group.collapsed"
          >
            <span class="node-arrow">{{ group.collapsed ? '▶' : '▼' }}</span>
            <span class="node-icon">📁</span>
            <span class="node-name">{{ group.name }}</span>
          </div>
          <template v-if="!systemGroups[0].collapsed">
            <div class="tree-node">
              <span class="node-icon">🎫</span>
              <span class="node-name">售票机_TVM_01</span>
            </div>
          </template>
          <template v-if="!systemGroups[1].collapsed">
            <div class="tree-node">
              <span class="node-icon">📷</span>
              <span class="node-name">站台球机_01</span>
            </div>
          </template>
        </template>
      </div>
    </div>

    <!-- 车站视图 -->
    <div v-if="tabId === 'scene' && subTabId === 'object' && activeLeftTab === 'station'" class="view-content">
      <div class="station-list">
        <div
          v-for="station in stations"
          :key="station.id"
          :class="['station-card', { active: selectedStation === station.id }]"
          @click="selectStation(station)"
        >
          <div class="station-name">{{ station.name }}</div>
          <div class="station-info">{{ station.info }}</div>
        </div>
      </div>
    </div>

    <!-- 设备视图 -->
    <div v-if="tabId === 'scene' && subTabId === 'object' && activeLeftTab === 'device'" class="view-content">
      <!-- 维度筛选 -->
      <div class="filter-wrapper">
        <div class="dimension-filter">
          <div
            :class="['dim-btn', { active: activeDimension === 'space' }]"
            @click="activeDimension = 'space'"
          >
            按空间
          </div>
          <div
            :class="['dim-btn', { active: activeDimension === 'system' }]"
            @click="activeDimension = 'system'"
          >
            按系统
          </div>
        </div>
        <select class="system-select" v-model="currentSelect">
          <option v-for="opt in currentOptions" :key="opt.value" :value="opt.value">
            {{ opt.label }}
          </option>
        </select>
      </div>

      <!-- 设备工具栏 -->
      <div class="obj-toolbar">
        <div class="icon-btn" title="删除">🗑️</div>
        <div class="icon-btn" title="重命名">✏️</div>
        <div class="icon-btn" title="克隆">📄</div>
        <div class="icon-btn" title="上移">⬆️</div>
        <div class="icon-btn" title="下移">⬇️</div>
        <div class="icon-btn" title="置顶">⏫</div>
        <div class="icon-btn" title="置底">⏬</div>
        <div class="icon-btn" title="编组">🔗</div>
        <div class="icon-btn" title="解组">✂️</div>
      </div>

      <!-- 设备列表 -->
      <div class="device-tree">
        <template v-if="activeDimension === 'space'">
          <div
            v-for="group in spaceGroups"
            :key="group.id"
            :class="['tree-node', { folder: true, collapsed: group.collapsed }]"
            @click="group.collapsed = !group.collapsed"
          >
            <span class="node-arrow">{{ group.collapsed ? '▶' : '▼' }}</span>
            <span class="node-icon">{{ group.icon }}</span>
            <span class="node-name">{{ group.name }}</span>
          </div>
          <div
            v-for="device in devices"
            :key="device.id"
            :class="['tree-node', { selected: selectedDeviceId === device.id }]"
            @click="selectDevice(device)"
          >
            <span class="node-icon">{{ device.icon }}</span>
            <span class="node-name">{{ device.name }}</span>
            <div class="node-actions">
              <span class="action-icon" title="聚焦">🎯</span>
              <span
                :class="['action-icon', { active: device.locked }]"
                title="锁定"
                @click.stop="toggleLock(device)"
              >
                {{ device.locked ? '🔒' : '🔓' }}
              </span>
              <span
                :class="['action-icon', { active: !device.visible }]"
                title="隐藏/显示"
                @click.stop="toggleVisible(device)"
              >
                {{ device.visible ? '👁️' : '👁️‍🗨️' }}
              </span>
            </div>
          </div>
        </template>

        <template v-if="activeDimension === 'system'">
          <div
            v-for="group in systemGroups"
            :key="group.id"
            :class="['tree-node', { folder: true, collapsed: group.collapsed }]"
            @click="group.collapsed = !group.collapsed"
          >
            <span class="node-arrow">{{ group.collapsed ? '▶' : '▼' }}</span>
            <span class="node-icon">{{ group.icon }}</span>
            <span class="node-name">{{ group.name }}</span>
          </div>
          <div
            v-for="device in systemDevices"
            :key="device.id"
            :class="['tree-node', { selected: selectedDeviceId === device.id }]"
            @click="selectDevice(device)"
          >
            <span class="node-icon">{{ device.icon }}</span>
            <span class="node-name">{{ device.name }}</span>
            <div class="node-actions">
              <span class="action-icon" title="聚焦">🎯</span>
              <span
                :class="['action-icon', { active: device.locked }]"
                title="锁定"
                @click.stop="toggleLock(device)"
              >
                {{ device.locked ? '🔒' : '🔓' }}
              </span>
              <span
                :class="['action-icon', { active: !device.visible }]"
                title="隐藏/显示"
                @click.stop="toggleVisible(device)"
              >
                {{ device.visible ? '👁️' : '👁️‍🗨️' }}
              </span>
            </div>
          </div>
        </template>
      </div>
    </div>

    <!-- 灯光视图 -->
    <div v-if="subTabId === 'light'" class="view-content">
      <div class="light-tree">
        <div class="tree-node">
          <span class="node-icon">☀️</span>
          <span class="node-name">环境光 (AmbientLight)</span>
        </div>
        <div class="tree-node selected">
          <span class="node-icon">🔦</span>
          <span class="node-name">聚光灯_闸机区_01</span>
        </div>
        <div class="tree-node">
          <span class="node-icon">💡</span>
          <span class="node-name">平行光_全局</span>
        </div>
      </div>
    </div>

    <!-- 特效视图 -->
    <div v-if="subTabId === 'effect'" class="view-content">
      <div class="empty-state">
        <div class="empty-icon">✨</div>
        <div class="empty-text">特效配置</div>
        <div class="empty-hint">暂无特效数据</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
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

// 左侧子Tab
const activeLeftTab = ref('station')

// 维度切换
const activeDimension = ref('space')

// 选中状态
const selectedStation = ref('station-1')
const selectedDeviceId = ref(null)

// 车站数据
const stations = ref([
  { id: 'station-1', name: '南京路站 (L2/L10)', info: '已布放 452 个设备模块' },
  { id: 'station-2', name: '世纪大道站 (L2/L4/L6/L9)', info: '枢纽级场景，多层级复杂结构' },
  { id: 'station-3', name: '徐家汇站 (L1/L9/L11)', info: '标准换乘站，部分数据已同步' }
])

// 楼层数据
const floors = ref([
  { value: '1f', label: '站厅层 (1F)' },
  { value: 'b1', label: '站台层 (B1)' },
  { value: 'b2', label: '设备层 (B2)' },
  { value: 'g', label: '地面出口 (G)' }
])
const selectedFloor = ref('1f')

// 系统数据
const systems = ref([
  { value: 'afc', label: 'AFC 自动售检票系统' },
  { value: 'cctv', label: 'CCTV 闭路监控系统' },
  { value: 'pis', label: 'PIS 乘客信息系统' },
  { value: 'bas', label: 'BAS 环境监控系统' }
])
const selectedSystem = ref('afc')

// 当前下拉选项（根据维度动态切换）
const currentOptions = computed(() => {
  return activeDimension.value === 'space' ? floors.value : systems.value
})
const currentSelect = computed({
  get() {
    return activeDimension.value === 'space' ? selectedFloor.value : selectedSystem.value
  },
  set(val) {
    if (activeDimension.value === 'space') {
      selectedFloor.value = val
    } else {
      selectedSystem.value = val
    }
  }
})

// 空间维度设备数据
const spaceGroups = ref([
  { id: 'group-1', name: '进站闸机组', icon: '📁', collapsed: false },
  { id: 'group-2', name: '安防监控设备', icon: '📁', collapsed: false }
])

const devices = ref([
  { id: 'device-1', name: '闸机_IN_01', icon: '🧊', locked: false, visible: true, type: 'gate' },
  { id: 'device-2', name: '闸机_IN_02', icon: '🧊', locked: false, visible: true, type: 'gate' },
  { id: 'device-3', name: '监控摄像头_Cam01', icon: '📷', locked: false, visible: true, type: 'camera' },
  { id: 'device-4', name: '监控摄像头_Cam02', icon: '📷', locked: false, visible: true, type: 'camera' }
])

// 系统维度数据
const systemGroups = ref([
  { id: 'sys-1', name: 'AFC 自动售检票系统', icon: '📁', collapsed: false },
  { id: 'sys-2', name: 'CCTV 闭路监控系统', icon: '📁', collapsed: false }
])

const systemDevices = ref([
  { id: 'sys-device-1', name: '售票机_TVM_01', icon: '🎫', locked: false, visible: true, type: 'tvm' },
  { id: 'sys-device-2', name: '站台球机_01', icon: '📷', locked: false, visible: true, type: 'camera' }
])

// 模型-材质 节点数据
const modelNodes = ref([
  { id: 'node-1', name: 'A-Ground001', isFolder: true, collapsed: false, level: 0, visible: true, locked: false },
  { id: 'node-2', name: 'MainTxt', isFolder: true, collapsed: false, level: 0, target: true, visible: true, locked: false },
  { id: 'node-3', name: 'ZhuZi', isFolder: false, level: 1, visible: true, locked: false },
  { id: 'node-4', name: 'Floor01', isFolder: false, level: 1, visible: true, locked: false },
  { id: 'node-5', name: 'WallGroup', isFolder: true, collapsed: false, level: 0, visible: true, locked: false },
  { id: 'node-6', name: 'Wall_Back', isFolder: false, level: 1, visible: true, locked: false },
  { id: 'node-7', name: 'Wall_Side', isFolder: false, level: 1, visible: true, locked: true }
])

const selectedNodeId = ref(null)

function selectNode(node) {
  if (!node.isFolder) {
    selectedNodeId.value = node.id
  } else {
    node.collapsed = !node.collapsed
  }
  emit('device-change', node)
}

function selectStation(station) {
  selectedStation.value = station.id
}

function selectDevice(device) {
  selectedDeviceId.value = device.id
  emit('device-change', device)
}

function toggleLock(device) {
  device.locked = !device.locked
  emit('device-change', device)
}

function toggleVisible(device) {
  device.visible = !device.visible
  emit('device-change', device)
}
</script>

<style scoped>
.object-panel {
  display: flex;
  flex-direction: column;
  border-right: 1px solid var(--border-color);
}

/* 车站/设备Tab */
.left-sub-tabs {
  display: flex;
  border-bottom: 1px solid var(--border-color);
  background: #1a1a1a;
}

.left-sub-tab {
  flex: 1;
  text-align: center;
  padding: 10px 0;
  font-size: 12px;
  color: var(--text-secondary);
  cursor: pointer;
  background: #1a1a1a;
  transition: 0.2s;
}

.left-sub-tab.active {
  background: var(--bg-panel);
  color: var(--text-primary);
  font-weight: 500;
  border-top: 2px solid var(--primary-color);
}

/* 视图内容 */
.view-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* 车站列表 */
.station-list {
  flex: 1;
  overflow-y: auto;
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.station-card {
  background: #262626;
  border: 1px solid #333;
  border-radius: 6px;
  padding: 12px;
  cursor: pointer;
  transition: 0.2s;
  position: relative;
}

.station-card:hover {
  border-color: #555;
  background: #2d2d2d;
}

.station-card.active {
  border-color: var(--primary-color);
  background: rgba(22, 119, 255, 0.05);
}

.station-card.active::after {
  content: '当前场景';
  position: absolute;
  top: 0;
  right: 0;
  background: var(--primary-color);
  color: #fff;
  font-size: 10px;
  padding: 2px 6px;
  border-bottom-left-radius: 6px;
}

.station-name {
  font-weight: bold;
  font-size: 13px;
  color: #fff;
  margin-bottom: 4px;
}

.station-info {
  font-size: 11px;
  color: var(--text-secondary);
}

/* 筛选器 */
.filter-wrapper {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 8px 12px;
  border-bottom: 1px solid #2a2a2a;
}

.dimension-filter {
  display: flex;
  gap: 8px;
}

.dim-btn {
  flex: 1;
  text-align: center;
  background: #141414;
  border: 1px solid #333;
  padding: 5px 0;
  border-radius: 4px;
  cursor: pointer;
  color: var(--text-secondary);
  font-size: 12px;
  transition: 0.2s;
}

.dim-btn.active {
  background: #2a2a2a;
  color: #fff;
  border-color: #555;
  font-weight: bold;
}

.system-select {
  background: #141414;
  border: 1px solid #333;
  color: var(--text-primary);
  padding: 4px 8px;
  border-radius: 4px;
  outline: none;
  height: 28px;
  font-size: 12px;
}

/* 设备工具栏 */
.obj-toolbar {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 6px;
  padding: 8px 12px;
  border-bottom: 1px solid var(--border-color);
  background: #1c1c1c;
}

.icon-btn {
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #2a2a2a;
  border: 1px solid #3a3a3a;
  border-radius: 4px;
  cursor: pointer;
  color: var(--text-main);
  font-size: 13px;
  transition: 0.2s;
}

.icon-btn:hover {
  background: #3a3a3a;
  color: var(--primary-color);
  border-color: #555;
}

/* 设备列表 */
.device-tree,
.light-tree {
  flex: 1;
  overflow-y: auto;
  padding: 8px 0;
}

.tree-node {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  font-size: 13px;
  color: var(--text-secondary);
  cursor: pointer;
  transition: 0.1s;
}

.tree-node:hover {
  background: #2a2a2a;
  color: var(--text-primary);
}

.tree-node.selected {
  background: rgba(22, 119, 255, 0.15);
  color: var(--primary-color);
  border-right: 2px solid var(--primary-color);
}

.tree-node.folder {
  color: var(--text-primary);
  font-weight: 500;
}

.tree-node.folder .node-icon {
  color: var(--warning-color);
}

.node-arrow {
  font-size: 10px;
  color: var(--text-muted);
  width: 12px;
}

.node-icon {
  font-size: 14px;
}

.node-name {
  flex: 1;
}

.node-actions {
  display: none;
  gap: 4px;
}

.tree-node:hover .node-actions,
.tree-node.selected .node-actions {
  display: flex;
}

.action-icon {
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #333;
  border-radius: 3px;
  font-size: 11px;
  color: #bbb;
  cursor: pointer;
  transition: 0.2s;
}

.action-icon:hover {
  background: var(--primary-color);
  color: #fff;
}

.action-icon.active {
  background: var(--primary-color);
  color: #fff;
}

/* 空状态 */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 200px;
  color: var(--text-muted);
}

.empty-icon {
  font-size: 48px;
  margin-bottom: 16px;
  opacity: 0.5;
}

.empty-text {
  font-size: 14px;
  color: var(--text-secondary);
}

.empty-hint {
  font-size: 12px;
  margin-top: 8px;
}

/* 模型树头部 */
.model-tree-header {
  padding: 8px;
  border-bottom: 1px solid var(--border-color);
}

.toggle-row {
  display: flex;
  gap: 4px;
  margin-bottom: 8px;
}

.toggle-btn {
  flex: 1;
  height: 28px;
  background: #2a2a2a;
  border: 1px solid #3a3a3a;
  color: #fff;
  border-radius: 2px;
  cursor: pointer;
  font-size: 12px;
}

.toggle-btn.active {
  background: #383838;
  border-color: #555;
  font-weight: bold;
}

/* 节点设置面板 */
.node-panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 12px;
  border-bottom: 1px solid #2a2a2a;
}

.node-panel-header .panel-title {
  font-size: 12px;
  font-weight: bold;
  border-left: 3px solid var(--primary-color);
  padding-left: 8px;
}

.header-actions {
  color: #888;
  font-size: 12px;
}

.node-tree {
  flex: 1;
  overflow-y: auto;
  padding-top: 4px;
}

.node-item {
  display: flex;
  align-items: center;
  padding: 6px 12px;
  cursor: pointer;
  font-size: 12px;
  color: #ccc;
}

.node-item:hover {
  background: var(--bg-hover);
}

.node-item.selected {
  background: rgba(22, 119, 255, 0.2);
  color: #fff;
}

.node-icon {
  margin-right: 6px;
  width: 14px;
  text-align: center;
  color: #888;
  font-size: 10px;
}

.node-name {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.node-actions {
  display: flex;
  gap: 10px;
  color: #666;
  font-size: 14px;
}

.node-actions i.active {
  color: #00e5ff;
}

.node-item.folder {
  color: #f1c40f;
  font-weight: 500;
}
</style>
