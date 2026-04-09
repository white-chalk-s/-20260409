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
    <div v-if="tabId === 'scene' && subTabId === 'light'" class="view-content">
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
    <div v-if="tabId === 'scene' && subTabId === 'effect'" class="view-content">
      <div class="empty-state">
        <div class="empty-icon">✨</div>
        <div class="empty-text">特效配置</div>
        <div class="empty-hint">暂无特效数据</div>
      </div>
    </div>

    <!-- 状态-图层视图 -->
    <div v-if="tabId === 'status' && subTabId === 'layer'" class="view-content">
      <div class="empty-state">
        <div class="empty-icon">📊</div>
        <div class="empty-text">图层状态配置</div>
        <div class="empty-hint">暂无图层数据</div>
      </div>
    </div>

    <!-- 资产tab内容 -->
    <div v-if="tabId === 'asset'" class="view-content asset-view">
      <!-- 设备子Tab -->
      <div class="asset-tabs" v-if="subTabId === 'device'">
        <div
          v-for="devTab in deviceSubTabs"
          :key="devTab.id"
          :class="['asset-tab-item', { active: activeAssetTab === devTab.id }]"
          @click="activeAssetTab = devTab.id"
        >
          {{ devTab.label }}
        </div>
      </div>

      <!-- 设备视图 -->
      <div v-if="subTabId === 'device'" class="asset-device-content">
        <div class="asset-section-title">设备模型 (拖拽到场景)</div>
        <div class="device-grid">
          <div
            v-for="dev in deviceModels"
            :key="dev.id"
            class="device-card"
            draggable="true"
            @dragstart="handleDragStart($event, dev)"
          >
            <div class="device-preview">
              <div class="device-icon">{{ dev.icon }}</div>
            </div>
            <div class="device-name">{{ dev.name }}</div>
          </div>
        </div>
      </div>

      <!-- 车站视图 -->
      <div v-if="subTabId === 'station'" class="asset-content">
        <div class="empty-state">
          <div class="empty-icon">🏢</div>
          <div class="empty-text">车站资产</div>
          <div class="empty-hint">车站数据</div>
        </div>
      </div>

      <!-- 材质视图 -->
      <div v-if="subTabId === 'material'" class="asset-content">
        <div class="empty-state">
          <div class="empty-icon">🎨</div>
          <div class="empty-text">材质资产</div>
          <div class="empty-hint">材质数据</div>
        </div>
      </div>

      <!-- 几何体视图 -->
      <div v-if="subTabId === 'geometry'" class="asset-content">
        <div class="empty-state">
          <div class="empty-icon">📐</div>
          <div class="empty-text">几何体资产</div>
          <div class="empty-hint">几何体数据</div>
        </div>
      </div>

      <!-- 图标视图 -->
      <div v-if="subTabId === 'icon'" class="asset-content">
        <div class="empty-state">
          <div class="empty-icon">🏷️</div>
          <div class="empty-text">图标资产</div>
          <div class="empty-hint">图标数据</div>
        </div>
      </div>

      <!-- 测试模型视图 -->
      <div v-if="subTabId === 'testModel'" class="asset-content">
        <div class="empty-state">
          <div class="empty-icon">🧪</div>
          <div class="empty-text">测试模型</div>
          <div class="empty-hint">测试模型数据</div>
        </div>
      </div>

      <!-- 测试文件库视图 -->
      <div v-if="subTabId === 'testFile'" class="asset-content">
        <div class="empty-state">
          <div class="empty-icon">📁</div>
          <div class="empty-text">测试文件库</div>
          <div class="empty-hint">测试文件数据</div>
        </div>
      </div>

      <!-- 工具tab内容 -->
      <div v-if="tabId === 'tool'" class="view-content tool-view">
        <!-- 路径规划 -->
        <div v-if="subTabId === 'path'" class="tool-path-content">
          <div class="panel-header">
            <div class="panel-title">路径规划属性</div>
            <div class="version-tag">V1.0</div>
          </div>
          <div class="config-group">
            <div class="instruction">
              <p style="color: #bbb; margin-bottom: 4px;">操作指引：</p>
              1. 在场景中点击放置路径点<br>
              2. 自动生成最短巡检路径<br>
              3. 支持设置节点停顿时间
            </div>
          </div>
        </div>

        <!-- 编辑器 -->
        <div v-if="subTabId === 'editor'" class="tool-editor-content">
          <div class="empty-state">
            <div class="empty-icon">🛠️</div>
            <div class="empty-text">编辑器工具</div>
            <div class="empty-hint">编辑器功能开发中</div>
          </div>
        </div>
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

// 资产模块相关
const activeAssetTab = ref('doorAccess') // 默认门禁系统

const deviceSubTabs = ref([
  { id: 'doorAccess', label: '门禁系统' },
  { id: 'fireProtection', label: '消防系统' },
  { id: 'power', label: '供配电系统' },
  { id: 'lighting', label: '照明系统' },
  { id: 'ventilation', label: '通风空调系统' },
  { id: 'elevator', label: '电梯系统' }
])

// 设备模型数据（支持拖拽）
const deviceModels = ref([
  { id: 'dm-1', name: 'AGM闸机', icon: '🚪' },
  { id: 'dm-2', name: 'TVM售票机', icon: '🎫' },
  { id: 'dm-3', name: '球机摄像头', icon: '📷' },
  { id: 'dm-4', name: '枪机摄像头', icon: '📹' },
  { id: 'dm-5', name: '温湿度传感器', icon: '🌡️' },
  { id: 'dm-6', name: '烟感探测器', icon: '🚬' },
  { id: 'dm-7', name: '紧急按钮', icon: '🔴' },
  { id: 'dm-8', name: '声光报警器', icon: '🔔' },
  { id: 'dm-9', name: '疏散指示灯', icon: '➡️' },
  { id: 'dm-10', name: '消防泵', icon: '🔧' },
  { id: 'dm-11', name: '配电箱', icon: '⚡' },
  { id: 'dm-12', name: '照明灯', icon: '💡' }
])

function handleDragStart(event, device) {
  event.dataTransfer.setData('device', JSON.stringify(device))
  event.dataTransfer.effectAllowed = 'copy'
}

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

/* 状态视图 */
.status-view {
  display: flex;
  flex-direction: column;
  flex: 1;
  overflow: hidden;
}

.status-view .section-title {
  font-size: 12px;
  color: #888;
  margin-bottom: 12px;
  border-left: 3px solid var(--primary-color);
  padding-left: 10px;
  font-weight: bold;
  text-transform: uppercase;
  margin: 16px 12px 12px 12px;
}

.status-view .view-list {
  background: #111;
  border: 1px solid #333;
  border-radius: var(--radius-sm);
  margin: 0 12px 16px 12px;
  max-height: 200px;
  overflow-y: auto;
}

.status-view .view-row {
  padding: 12px;
  font-size: 13px;
  border-bottom: 1px solid #222;
  display: flex;
  align-items: center;
  cursor: pointer;
  transition: background 0.2s;
}

.status-view .view-row:last-child {
  border-bottom: none;
}

.status-view .view-row:hover {
  background: #1a1a1a;
}

.status-view .view-row.active {
  background: rgba(22, 119, 255, 0.15);
  color: #fff;
  font-weight: bold;
  border-left: 3px solid var(--primary-color);
}

.status-view .view-icon {
  margin-right: 10px;
}

.status-view .view-name {
  flex: 1;
  background: transparent;
  border: none;
  color: inherit;
  font-size: 13px;
  outline: none;
}

.status-view .control-area {
  background: #181818;
  padding: 16px;
  border-radius: var(--radius-sm);
  border: 1px solid #262626;
  margin: 0 12px 16px 12px;
}

.status-view .ctrl-group-title {
  font-size: 11px;
  color: #555;
  margin-bottom: 12px;
  text-align: center;
  border-bottom: 1px solid #262626;
  padding-bottom: 8px;
  font-weight: bold;
}

.status-view .coord-row {
  display: flex;
  gap: 10px;
  margin-bottom: 12px;
}

.status-view .coord-item {
  flex: 1;
}

.status-view .coord-label {
  display: block;
  font-size: 10px;
  color: #666;
  margin-bottom: 4px;
  text-indent: 2px;
}

.status-view .coord-input {
  width: 100%;
  background: #000;
  border: 1px solid #333;
  color: var(--success-color);
  height: 30px;
  padding: 0 8px;
  border-radius: 2px;
  font-size: 12px;
  font-family: 'Courier New', monospace;
  outline: none;
}

.status-view .coord-input:focus {
  border-color: var(--primary-color);
}

.status-view .action-box {
  margin-top: 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.status-view .btn-secondary {
  width: 100%;
  height: 36px;
  background: #2a2a2a;
  border: 1px solid #3a3a3a;
  color: #ccc;
  border-radius: var(--radius-sm);
  cursor: pointer;
  font-size: 12px;
}

.status-view .btn-secondary:hover {
  background: #333;
  color: #fff;
}

/* 漫游路径面板 */
.status-view .panel-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
  border-bottom: 1px solid #333;
}

.status-view .panel-header {
  padding: 10px 14px;
  background: #252526;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #111;
}

.status-view .panel-title {
  font-size: 12px;
  font-weight: bold;
  color: #aaa;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.status-view .tool-icons {
  display: flex;
  gap: 12px;
  color: #888;
  cursor: pointer;
  font-size: 14px;
}

.status-view .tool-icons span:hover {
  color: var(--primary-color);
}

.status-view .list-container {
  flex: 1;
  overflow-y: auto;
  background: #181818;
  padding: 2px 0;
}

.status-view .list-item {
  padding: 8px 14px;
  display: flex;
  align-items: center;
  cursor: pointer;
  border-bottom: 1px solid #222;
}

.status-view .list-item:hover {
  background: #222;
}

.status-view .list-item.selected {
  background: #0d3d6b;
  border-left: 3px solid var(--primary-color);
}

.status-view .item-name {
  flex: 1;
  background: transparent;
  border: none;
  color: #ddd;
  font-size: 12px;
  outline: none;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.status-view .item-icon {
  margin-right: 8px;
  font-size: 14px;
  opacity: 0.7;
}

.status-view .config-detail-area {
  flex-shrink: 0;
  background: #1e1e1e;
  border-top: 2px solid #333;
  padding: 14px;
}

.status-view .config-title {
  margin-bottom: 10px;
  font-size: 11px;
  color: #666;
  font-weight: bold;
}

.status-view .config-grid {
  display: grid;
  gap: 10px;
}

.status-view .config-row {
  display: flex;
  align-items: center;
  gap: 8px;
}

.status-view .config-label {
  width: 55px;
  font-size: 12px;
  color: #888;
  flex-shrink: 0;
}

.status-view .config-inputs {
  flex: 1;
  display: flex;
  gap: 4px;
  overflow: hidden;
}

.status-view .input-box {
  flex: 1;
  min-width: 0;
  background: #111;
  border: 1px solid #333;
  color: #fff;
  height: 24px;
  padding: 0 4px;
  font-size: 11px;
  border-radius: 2px;
  font-family: monospace;
}

.status-view .input-box:focus {
  border-color: var(--primary-color);
  outline: none;
}

.status-view .config-time {
  flex: 1;
  display: flex;
  align-items: center;
}

.status-view .input-box.small {
  width: 45px;
  flex: none;
}

.status-view .time-unit {
  font-size: 10px;
  color: #444;
  margin-left: 2px;
}

.status-view .fov-value {
  font-size: 11px;
  color: #555;
  width: 24px;
}

.status-view .btn-full {
  width: 100%;
  padding: 8px;
  background: #722ed1;
  border: none;
  color: #fff;
  border-radius: var(--radius-sm);
  cursor: pointer;
  font-weight: bold;
  margin-top: 8px;
  font-size: 12px;
  transition: background 0.2s;
}

.status-view .btn-full:hover {
  background: #8247d5;
}

/* 资产模块样式 */
.asset-view {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.asset-tabs {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  padding: 12px;
  border-bottom: 1px solid var(--border-color);
  background: var(--bg-panel);
}

.asset-tab-item {
  padding: 6px 12px;
  font-size: 12px;
  color: var(--text-secondary);
  background: var(--bg-base);
  border: 1px solid var(--border-light);
  border-radius: var(--radius-sm);
  cursor: pointer;
  transition: all 0.2s;
}

.asset-tab-item:hover {
  background: var(--bg-hover);
  color: var(--text-primary);
}

.asset-tab-item.active {
  background: var(--primary-color);
  color: #fff;
  border-color: var(--primary-color);
}

.asset-device-content {
  flex: 1;
  overflow-y: auto;
  padding: 12px;
}

.asset-section-title {
  font-size: 11px;
  font-weight: 600;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 12px;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border-light);
}

.device-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}

.device-card {
  background: var(--bg-panel);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-sm);
  overflow: hidden;
  cursor: grab;
  transition: all 0.2s;
}

.device-card:hover {
  border-color: var(--primary-color);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(22, 119, 255, 0.2);
}

.device-card:active {
  cursor: grabbing;
}

.device-preview {
  width: 100%;
  aspect-ratio: 1;
  background: radial-gradient(circle, #333 0%, #111 100%);
  display: flex;
  align-items: center;
  justify-content: center;
}

.device-icon {
  font-size: 32px;
}

.device-name {
  padding: 8px;
  font-size: 11px;
  text-align: center;
  color: var(--text-secondary);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.asset-content {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 工具模块样式 */
.tool-view {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.tool-path-content {
  display: flex;
  flex-direction: column;
}

.tool-path-content .panel-header {
  padding: 16px;
  border-bottom: 1px solid var(--border-color);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.tool-path-content .panel-title {
  font-size: 14px;
  font-weight: bold;
}

.tool-path-content .version-tag {
  color: var(--primary-color);
  font-size: 12px;
}

.tool-path-content .config-group {
  padding: 20px;
}

.tool-path-content .instruction {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 6px;
  padding: 12px;
  line-height: 1.6;
  color: #888;
  border: 1px solid #222;
}

.tool-editor-content {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
