<template>
  <div class="property-panel panel-area" v-show="showPropertyPanel">
    <!-- 灯光tab时显示三维视图配置 -->
    <div v-if="isLightTab" class="light-config">
      <div class="panel-header">
        <div>
          <div class="panel-title">三维视图配置</div>
          <div class="panel-subtitle">场景渲染与后处理设置</div>
        </div>
      </div>
      <div class="property-content">
        <div class="property-group">
          <div class="group-title">三维视图配置</div>
          <div class="property-item">
            <label class="label">色彩空间</label>
            <select class="select"><option>LinearSRGBColorSpace</option></select>
          </div>
          <div class="property-item">
            <label class="label">色调映射</label>
            <select class="select"><option>线性</option></select>
          </div>
          <div class="property-item">
            <label class="label">曝光度</label>
            <div class="stepper">
              <button class="step-btn" @click="exposure = Math.max(0, exposure - 0.1)">−</button>
              <input type="text" class="step-val" v-model="exposure" />
              <button class="step-btn" @click="exposure = Math.min(10, exposure + 0.1)">+</button>
            </div>
          </div>
          <div class="property-item">
            <label class="label">背景色</label>
            <div class="color-picker">
              <div class="color-box" :style="{ background: backgroundColor }"></div>
              <input type="color" v-model="backgroundColor" class="color-input" />
            </div>
          </div>
        </div>
        <div class="property-group">
          <div class="group-title">环境贴图配置</div>
          <div class="property-item">
            <label class="label">环境贴图</label>
            <div class="file-input">
              <input type="text" class="input" :value="envMapPath" readonly />
              <button class="btn-select">选择</button>
            </div>
          </div>
          <div class="property-item">
            <label class="label">强度</label>
            <div class="stepper">
              <button class="step-btn" @click="envIntensity = Math.max(0, envIntensity - 0.1)">−</button>
              <input type="text" class="step-val" v-model="envIntensity" />
              <button class="step-btn" @click="envIntensity = Math.min(5, envIntensity + 0.1)">+</button>
            </div>
          </div>
        </div>
        <div class="property-group">
          <div class="group-title">后处理配置</div>
          <div class="property-item switch-item">
            <label class="label">后处理</label>
            <label class="switch"><input type="checkbox" v-model="postProcessing.enabled" /><span class="slider"></span></label>
            <span class="switch-text">开启</span>
          </div>
          <div class="property-item switch-item">
            <label class="label">AO</label>
            <label class="switch"><input type="checkbox" v-model="postProcessing.ao" /><span class="slider"></span></label>
            <span class="switch-text">开启</span>
          </div>
          <div class="property-item switch-item">
            <label class="label">SSR</label>
            <label class="switch"><input type="checkbox" v-model="postProcessing.ssr" /><span class="slider"></span></label>
            <span class="switch-text">开启</span>
          </div>
          <div class="property-item switch-item">
            <label class="label">反锯齿</label>
            <label class="switch"><input type="checkbox" v-model="postProcessing.antiAlias" /><span class="slider"></span></label>
            <span class="switch-text">开启</span>
          </div>
          <div class="property-item">
            <label class="label">方法</label>
            <select class="select"><option>FXAA</option></select>
          </div>
        </div>
      </div>
    </div>

    <!-- 模型-材质tab时显示材质库 -->
    <div v-else-if="isModelMaterialTab" class="material-config">
      <div class="panel-header">
        <div>
          <div class="panel-title">材质库</div>
          <div class="panel-subtitle">内置材质与自定义材质</div>
        </div>
      </div>
      <div class="property-content">
        <div class="mat-category">📂 所有材质</div>
        <div class="mat-sub-category">📁 测试子目录</div>
        <div class="mat-grid">
          <div class="mat-card" v-for="mat in materials" :key="mat.id" @click="selectMaterial(mat)">
            <div class="mat-preview"><div class="mat-sphere" :style="{ background: mat.color }"></div></div>
            <div class="mat-name">{{ mat.name }}</div>
          </div>
        </div>
        <div class="adv-btn-container">
          <button class="btn-advanced" @click="showAdvanced = !showAdvanced">高级设置 {{ showAdvanced ? '▼' : '❯' }}</button>
        </div>
        <div class="advanced-drawer" v-show="showAdvanced">
          <div class="drawer-header">
            <span class="drawer-title">高级材质配置</span>
            <button class="drawer-close" @click="showAdvanced = false">×</button>
          </div>
          <div class="mat-title">{{ selectedMaterialName }}</div>
          <div class="property-group">
            <div class="property-item"><label class="label">类型</label><select class="select"><option>单面透明材质</option></select></div>
            <div class="property-item"><label class="label">颜色</label><input type="color" class="color-input" v-model="materialColor" /></div>
            <div class="property-item">
              <label class="label">粗糙度</label>
              <div class="slider-group"><input type="range" v-model="roughness" min="0" max="1" step="0.01" /><span class="slider-value">{{ roughness.toFixed(2) }}</span></div>
            </div>
            <div class="property-item">
              <label class="label">金属度</label>
              <div class="slider-group"><input type="range" v-model="metallic" min="0" max="1" step="0.01" /><span class="slider-value">{{ metallic.toFixed(2) }}</span></div>
            </div>
          </div>
          <div class="tex-section-title">材质贴图</div>
          <div class="property-item"><label class="label">颜色贴图</label><div class="tex-btns"><button class="tex-btn">配置</button><button class="tex-btn clean">清理</button></div></div>
          <div class="property-item"><label class="label">法线贴图</label><button class="tex-btn" style="width:100%">选择</button></div>
          <div class="mapping-box">
            <div class="mapping-title">贴图配置</div>
            <div class="map-grid"><div>重复</div><input type="text" class="map-input" v-model="textureRepeat.x" /><div style="text-align:center">×</div><input type="text" class="map-input" v-model="textureRepeat.y" /></div>
            <div class="map-grid"><div>偏移</div><input type="text" class="map-input" v-model="textureOffset.x" /><div style="text-align:center">×</div><input type="text" class="map-input" v-model="textureOffset.y" /></div>
          </div>
          <button class="btn-advanced btn-apply" @click="showAdvanced = false">应用并关闭</button>
        </div>
      </div>
    </div>

    <!-- 模型-动画tab时显示动画配置 -->
    <div v-else-if="isModelAnimationTab" class="animation-config">
      <div class="panel-header">
        <div><div class="panel-title">动画配置</div><div class="panel-subtitle">动画列表与播放控制</div></div>
      </div>
      <div class="property-content">
        <div class="group-title">动画列表</div>
        <div class="anim-list">
          <div v-for="anim in animationList" :key="anim.id" :class="['anim-row', { active: selectedAnimId === anim.id }]" @click="selectedAnimId = anim.id">
            <span>{{ anim.name }}</span><span class="anim-duration">{{ anim.duration }}</span>
          </div>
        </div>
        <div class="pagination-row"><span class="page-arrow">❮ PREV</span><span class="page-info">1 / 4</span><span class="page-arrow">NEXT ❯</span></div>
        <div class="group-title">播放属性</div>
        <div class="control-area">
          <div class="property-item"><label class="label">循环模式</label><select class="select"><option>LoopRepeat (无限循环)</option><option>LoopOnce (播放一次)</option><option>LoopPingPong (往返循环)</option></select></div>
          <div class="property-item"><label class="label">播放速度</label><select class="select"><option>1x (默认速度)</option><option>2x (快速)</option><option>0.5x (慢放)</option></select></div>
          <div class="btn-grid">
            <button class="btn-action btn-play" @click="playAnim">播放</button>
            <button class="btn-action btn-stop" @click="stopAnim">停止</button>
            <button class="btn-action btn-resume" @click="resumeAnim">继续</button>
            <button class="btn-action btn-pause" @click="pauseAnim">暂停</button>
            <button class="btn-action btn-reset" @click="resetAnim">重置动画状态</button>
          </div>
        </div>
      </div>
    </div>

    <!-- 状态-视角tab时显示视角管理 -->
    <div v-else-if="isStatusViewTab" class="view-config">
      <div class="panel-header">
        <div><div class="panel-title">视角管理</div><div class="panel-subtitle">视角锚点配置</div></div>
      </div>
      <div class="property-content">
        <div class="add-viewpoint-container"><button class="btn-primary-add"><i>+</i> 新增视角锚点</button></div>
        <div class="group-title">已保存视角 (4)</div>
        <div class="view-list">
          <div class="view-row active"><span class="view-icon">📍</span><input type="text" class="view-name" value="大厅全局概览" /></div>
          <div class="view-row"><span class="view-icon">📍</span><input type="text" class="view-name" value="进站闸机特写" /></div>
          <div class="view-row"><span class="view-icon">📍</span><input type="text" class="view-name" value="自动售票机区域" /></div>
          <div class="view-row"><span class="view-icon">📍</span><input type="text" class="view-name" value="安检通道监控视角" /></div>
        </div>
        <div class="group-title">空间位置参数</div>
        <div class="control-area">
          <div class="ctrl-group-title">世界坐标 (POSITION)</div>
          <div class="coord-row">
            <div class="coord-item"><span class="coord-label">X-AXIS</span><input type="text" class="coord-input" value="142.52" /></div>
            <div class="coord-item"><span class="coord-label">Y-AXIS</span><input type="text" class="coord-input" value="65.88" /></div>
            <div class="coord-item"><span class="coord-label">Z-AXIS</span><input type="text" class="coord-input" value="-10.04" /></div>
          </div>
          <div class="ctrl-group-title">姿态旋转 (ROTATION)</div>
          <div class="coord-row">
            <div class="coord-item"><span class="coord-label">H (偏航)</span><input type="text" class="coord-input" value="180.00" /></div>
            <div class="coord-item"><span class="coord-label">P (俯仰)</span><input type="text" class="coord-input" value="-30.00" /></div>
            <div class="coord-item"><span class="coord-label">R (翻滚)</span><input type="text" class="coord-input" value="0.00" /></div>
          </div>
          <div class="action-box">
            <button class="btn-secondary">📸 覆盖当前视角坐标</button>
            <button class="btn-secondary" style="color:#ff4d4f">🗑 删除选定视角</button>
          </div>
        </div>
      </div>
    </div>

    <!-- 状态-漫游tab时显示漫游配置 -->
    <div v-else-if="isStatusRoamTab" class="roam-config">
      <div class="panel-header">
        <div><div class="panel-title">漫游配置</div><div class="panel-subtitle">漫游路径与点位管理</div></div>
      </div>
      <div class="property-content roam-content">
        <div class="panel-section">
          <div class="panel-header">
            <div class="panel-title">漫游路径</div>
            <div class="tool-icons"><span title="新增路径">⊕</span><span title="重命名">✎</span><span title="删除路径">🗑</span></div>
          </div>
          <div class="list-container">
            <div class="list-item selected"><span class="item-icon">🛤</span><input type="text" class="item-name" value="核心巡检路径_01" /></div>
            <div class="list-item"><span class="item-icon">🛤</span><input type="text" class="item-name" value="应急疏散演练路径" /></div>
          </div>
        </div>
        <div class="panel-section">
          <div class="panel-header">
            <div class="panel-title">视角点位 (点位流)</div>
            <div class="tool-icons"><span title="播放全部">▶</span><span title="新增点位">⊕</span><span title="重命名点位">✎</span><span title="删除点位">🗑</span></div>
          </div>
          <div class="list-container">
            <div class="list-item selected"><span class="item-icon">📍</span><input type="text" class="item-name" value="点位 1: 站厅入口" /></div>
            <div class="list-item"><span class="item-icon">📍</span><input type="text" class="item-name" value="点位 2: 扶梯侧视" /></div>
            <div class="list-item"><span class="item-icon">📍</span><input type="text" class="item-name" value="点位 3: 站台监控" /></div>
            <div class="list-item"><span class="item-icon">📍</span><input type="text" class="item-name" value="点位 4: 闸机特写" /></div>
          </div>
          <div class="config-detail-area">
            <div class="config-title">参数配置 (选中点位)</div>
            <div class="config-grid">
              <div class="config-row"><span class="config-label">视点(P)</span><div class="config-inputs"><input type="text" class="input-box" value="243.3" /><input type="text" class="input-box" value="198.6" /><input type="text" class="input-box" value="-192" /></div></div>
              <div class="config-row"><span class="config-label">目标(T)</span><div class="config-inputs"><input type="text" class="input-box" value="71.7" /><input type="text" class="input-box" value="-6.3" /><input type="text" class="input-box" value="-49.0" /></div></div>
              <div class="config-row">
                <div class="config-time"><span class="config-label">飞行</span><input type="text" class="input-box small" value="2.5" /><span class="time-unit">s</span></div>
                <div class="config-time"><span class="config-label">驻留</span><input type="text" class="input-box small" value="1.0" /><span class="time-unit">s</span></div>
              </div>
              <div class="config-row"><span class="config-label">视场角</span><input type="range" style="flex:1;height:4px" /><span class="fov-value">60°</span></div>
            </div>
            <button class="btn-full">📸 保存当前相机视角</button>
          </div>
        </div>
      </div>
    </div>

    <!-- 其他tab时显示设备属性 -->
    <div v-else class="device-config">
      <div class="panel-header">
        <div><div class="panel-title">{{ panelTitle }}</div><div class="panel-subtitle">{{ selectedDevice ? `已选择: ${selectedDevice.name}` : '请选择设备' }}</div></div>
      </div>
      <div class="property-content">
        <template v-if="selectedDevice">
          <div class="property-group">
            <div class="group-title">基本信息</div>
            <div class="property-item"><label class="label">设备唯一编号</label><div class="val-text">1773978151443</div></div>
            <div class="property-item"><label class="label">设备名称</label><input type="text" class="input" v-model="selectedDevice.name" /></div>
            <div class="property-item"><label class="label">线路名称</label><div class="val-text">4号线二期</div></div>
            <div class="property-item"><label class="label">车站名称</label><div class="val-text">两江影视城站</div></div>
            <div class="property-item"><label class="label">位置</label><div class="val-text">站厅层-西侧</div></div>
            <div class="property-item"><label class="label">设备专业</label><div class="val-text">AFC</div></div>
            <div class="property-item"><label class="label">设备类型</label><div class="val-text">AGM</div></div>
            <div class="property-item switch-item"><label class="label">是否可见</label><label class="switch"><input type="checkbox" v-model="selectedDevice.visible" /><span class="slider"></span></label></div>
          </div>
          <div class="more-link" @click="showMoreDetail = !showMoreDetail">更多详情 >></div>
          <div class="data-view-container" v-show="showMoreDetail">
            <div class="data-view-tabs">
              <div :class="['data-view-tab', { active: dataViewType === 'table' }]" @click="dataViewType = 'table'">表格视图</div>
              <div :class="['data-view-tab', { active: dataViewType === 'json' }]" @click="dataViewType = 'json'">JSON视图</div>
            </div>
            <div class="data-content" v-show="dataViewType === 'table'">
              <table class="data-table"><tr><td style="color:#888;">equipmentInfo</td><td style="color:var(--primary-color);cursor:pointer;">查看</td></tr><tr><td style="color:#888;">fileInfo</td><td style="color:var(--primary-color);cursor:pointer;">查看</td></tr><tr><td style="color:#888;">xrayed</td><td>false</td></tr></table>
            </div>
            <div class="data-content" v-show="dataViewType === 'json'"><div class="json-box">{"equipmentInfo":{"uniqueId":1773978151443},"xrayed":false}</div></div>
          </div>
          <div class="property-group">
            <div class="group-title">变换</div>
            <div class="property-item"><label class="label">位置</label><div class="input-group"><div class="input-item"><input type="text" class="input" v-model="position.x" /><div class="input-label">X</div></div><div class="input-item"><input type="text" class="input" v-model="position.y" /><div class="input-label">Y</div></div><div class="input-item"><input type="text" class="input" v-model="position.z" /><div class="input-label">Z</div></div></div></div>
            <div class="property-item"><label class="label">旋转</label><div class="input-group"><div class="input-item"><input type="text" class="input" v-model="rotation.x" /><div class="input-label">X</div></div><div class="input-item"><input type="text" class="input" v-model="rotation.y" /><div class="input-label">Y</div></div><div class="input-item"><input type="text" class="input" v-model="rotation.z" /><div class="input-label">Z</div></div></div></div>
            <div class="property-item"><label class="label">缩放</label><div class="input-group"><div class="input-item"><input type="text" class="input" v-model="scale.x" /><div class="input-label">X</div></div><div class="input-item"><input type="text" class="input" v-model="scale.y" /><div class="input-label">Y</div></div><div class="input-item"><input type="text" class="input" v-model="scale.z" /><div class="input-label">Z</div></div></div></div>
          </div>
          <div class="property-group">
            <div class="group-title">动画</div>
            <div class="property-item"><label class="label">动画选择</label><select class="select"><option>Take 001</option></select></div>
            <div class="property-item"><label class="label">循环模式</label><select class="select"><option>LoopRepeat</option></select></div>
            <div class="property-item"><label class="label">播放控制</label><div class="action-grid"><button class="act-btn btn-blue" @click="playAnim">播放</button><button class="act-btn btn-orange" @click="stopAnim">停止</button><button class="act-btn btn-green" @click="resumeAnim">继续</button><button class="act-btn btn-grey" @click="pauseAnim">暂停</button><button class="act-btn btn-red" @click="resetAnim">重置</button></div></div>
          </div>
          <div class="property-group">
            <div class="group-title">显示状态</div>
            <div class="property-item switch-item"><label class="label">是否显示</label><label class="switch"><input type="checkbox" v-model="selectedDevice.visible" /><span class="slider"></span></label></div>
            <div class="property-item switch-item"><label class="label">是否锁定</label><label class="switch"><input type="checkbox" v-model="selectedDevice.locked" /><span class="slider"></span></label></div>
          </div>
        </template>
        <template v-else>
          <div class="empty-state"><div class="empty-icon">📦</div><div class="empty-text">请在左侧选择设备</div><div class="empty-hint">点击设备节点查看属性</div></div>
        </template>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  tabId: { type: String, default: 'scene' },
  subTabId: { type: String, default: 'object' },
  selectedDevice: { type: Object, default: null }
})

const isLightTab = computed(() => props.tabId === 'scene' && props.subTabId === 'light')
const isEffectTab = computed(() => props.tabId === 'scene' && props.subTabId === 'effect')
const showPropertyPanel = computed(() => {
  if (props.tabId === 'scene' && props.subTabId === 'effect') return false
  if (props.tabId === 'tool') return false
  return true
})
const isModelMaterialTab = computed(() => props.tabId === 'model' && props.subTabId === 'material')
const isModelAnimationTab = computed(() => props.tabId === 'model' && props.subTabId === 'animation')
const isStatusViewTab = computed(() => props.tabId === 'status' && props.subTabId === 'view')
const isStatusRoamTab = computed(() => props.tabId === 'status' && props.subTabId === 'roam')
const panelTitle = computed(() => '属性配置')

const showMoreDetail = ref(false)
const dataViewType = ref('table')
const exposure = ref(1)
const backgroundColor = ref('#000000')
const envMapPath = ref('/attachment...')
const envIntensity = ref(0.85)
const postProcessing = ref({ enabled: true, ao: true, ssr: false, antiAlias: true })
const position = ref({ x: '0.00', y: '0.00', z: '0.00' })
const rotation = ref({ x: '0', y: '0', z: '0' })
const scale = ref({ x: '1.0', y: '1.0', z: '1.0' })

const materials = ref([
  { id: 'mat-1', name: '装修地面', color: '#cccccc' },
  { id: 'mat-2', name: '地砖', color: '#555555' },
  { id: 'mat-3', name: '墙面涂料', color: '#eeeeee' },
  { id: 'mat-4', name: '金属板', color: '#888888' }
])
const selectedMaterialId = ref(null)
const selectedMaterialName = computed(() => selectedMaterialId.value ? materials.value.find(m => m.id === selectedMaterialId.value)?.name || '未选择' : '未选择')
const showAdvanced = ref(false)
const materialColor = ref('#cccccc')
const roughness = ref(0.8)
const metallic = ref(0.2)
const textureRepeat = ref({ x: '1', y: '1' })
const textureOffset = ref({ x: '0', y: '0' })

function selectMaterial(mat) { selectedMaterialId.value = mat.id; materialColor.value = mat.color }

const animationList = ref([
  { id: 'anim-1', name: 'Take 001: Gate_Open', duration: '01.2s' },
  { id: 'anim-2', name: 'Take 002: Gate_Close', duration: '01.2s' },
  { id: 'anim-3', name: 'Take 003: Emergency_Alarm', duration: '00.5s' },
  { id: 'anim-4', name: 'Take 004: Pass_Validation', duration: '02.0s' },
  { id: 'anim-5', name: 'Take 005: Access_Denied', duration: '01.8s' },
  { id: 'anim-6', name: 'Take 006: Standby_Idle', duration: '05.0s' },
  { id: 'anim-7', name: 'Take 007: Indicator_Blink', duration: '03.0s' },
  { id: 'anim-8', name: 'Take 008: Hardware_Reset', duration: '00.8s' }
])
const selectedAnimId = ref('anim-1')

function playAnim() { console.log('播放动画') }
function stopAnim() { console.log('停止动画') }
function resumeAnim() { console.log('继续动画') }
function pauseAnim() { console.log('暂停动画') }
function resetAnim() { console.log('重置动画') }

watch(() => props.selectedDevice, (newDevice) => {
  if (newDevice) {
    position.value = { x: '-25.92', y: '182.15', z: '-1.94' }
    rotation.value = { x: '0', y: '270.0', z: '0' }
    scale.value = { x: '1.0', y: '1.0', z: '1.0' }
  }
}, { immediate: true })
</script>

<style scoped>
.property-panel { display: flex; flex-direction: column; border-left: 1px solid var(--border-color); }
.panel-header { padding: 14px 16px; border-bottom: 1px solid var(--border-color); }
.property-content { flex: 1; overflow-y: auto; padding: 16px; }
.property-group { margin-bottom: 20px; }
.group-title { font-size: 11px; font-weight: 600; color: var(--text-secondary); text-transform: uppercase; letter-spacing: 0.5px; margin-bottom: 12px; padding-bottom: 8px; border-bottom: 1px solid var(--border-light); }
.property-item { margin-bottom: 14px; }
.property-item .label { display: block; font-size: 12px; color: var(--text-secondary); margin-bottom: 6px; }
.val-text { font-size: 12px; color: var(--text-primary); }
.input-group { display: flex; gap: 8px; }
.input-item { flex: 1; }
.input-item .input { text-align: center; }
.input-label { font-size: 10px; color: var(--text-muted); text-align: center; margin-top: 4px; }
.switch-item { display: flex; align-items: center; justify-content: space-between; }
.switch-item .label { margin-bottom: 0; }
.more-link { color: var(--primary-color); cursor: pointer; font-size: 12px; text-decoration: underline; margin: 4px 0 10px 0; }
.data-view-container { border: 1px solid var(--border-color); border-radius: var(--radius-sm); margin-bottom: 10px; overflow: hidden; }
.data-view-tabs { display: flex; background: #1a1a1a; border-bottom: 1px solid var(--border-color); }
.data-view-tab { flex: 1; padding: 8px; text-align: center; font-size: 11px; cursor: pointer; color: var(--text-secondary); }
.data-view-tab.active { color: #fff; background: var(--primary-color); }
.data-table { width: 100%; border-collapse: collapse; font-size: 11px; table-layout: fixed; }
.data-table td { border: 1px solid var(--border-color); padding: 6px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.json-box { background: #000; color: #ce9178; padding: 8px; font-family: monospace; font-size: 11px; height: 100px; overflow-y: auto; white-space: pre-wrap; word-break: break-all; }
.action-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-top: 4px; width: 100%; }
.act-btn { border: none; border-radius: 3px; height: 28px; color: #fff; font-size: 12px; cursor: pointer; }
.btn-blue { background: #1677ff; }
.btn-orange { background: #e6a23c; }
.btn-green { background: #67c23a; }
.btn-grey { background: #606266; }
.btn-red { background: #f56c6c; width: calc(50% - 4px); margin-top: 8px; }
.stepper { display: flex; width: 100%; border: 1px solid var(--border-color); border-radius: var(--radius-sm); overflow: hidden; }
.step-btn { width: 28px; height: 26px; background: #262626; border: none; color: #888; cursor: pointer; font-size: 14px; }
.step-btn:hover { background: #333; color: #fff; }
.step-val { flex: 1; background: var(--input-bg); border: none; color: var(--text-primary); text-align: center; font-size: 12px; border-left: 1px solid var(--border-color); border-right: 1px solid var(--border-color); }
.color-picker { display: flex; align-items: center; gap: 8px; }
.color-box { width: 28px; height: 28px; border: 1px solid var(--border-color); border-radius: var(--radius-sm); }
.color-input { width: 28px; height: 28px; padding: 0; border: none; background: transparent; cursor: pointer; }
.file-input { display: flex; width: 100%; }
.file-input .input { flex: 1; border-radius: var(--radius-sm) 0 0 var(--radius-sm); text-align: left; padding-left: 8px; font-size: 11px; }
.btn-select { background: var(--primary-color); border: none; color: #fff; padding: 0 12px; border-radius: 0 var(--radius-sm) var(--radius-sm) 0; cursor: pointer; font-size: 12px; }
.switch-text { margin-left: 8px; color: var(--primary-color); font-size: 12px; }
.empty-state { display: flex; flex-direction: column; align-items: center; justify-content: center; height: 200px; color: var(--text-muted); }
.empty-icon { font-size: 48px; margin-bottom: 16px; opacity: 0.5; }
.empty-text { font-size: 14px; color: var(--text-secondary); }
.empty-hint { font-size: 12px; margin-top: 8px; }

.mat-category { font-size: 12px; color: #f1c40f; display: flex; align-items: center; gap: 6px; margin-bottom: 10px; }
.mat-sub-category { padding-left: 20px; font-size: 12px; color: #ccc; margin-bottom: 6px; cursor: pointer; }
.mat-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; }
.mat-card { background: #161616; border: 1px solid #333; border-radius: var(--radius-sm); overflow: hidden; cursor: pointer; transition: 0.2s; }
.mat-card:hover { border-color: var(--primary-color); }
.mat-preview { width: 100%; aspect-ratio: 1; background: radial-gradient(circle, #444 0%, #111 100%); display: flex; align-items: center; justify-content: center; }
.mat-sphere { width: 70%; height: 70%; border-radius: 50%; box-shadow: inset -10px -10px 20px rgba(0,0,0,0.8); }
.mat-name { padding: 6px; font-size: 11px; text-align: center; color: #aaa; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.adv-btn-container { padding: 12px; border-top: 1px solid #333; background: #1a1a1a; }
.btn-advanced { width: 100%; padding: 8px; background: #333; border: 1px solid #444; color: #fff; border-radius: var(--radius-sm); cursor: pointer; font-size: 12px; font-weight: bold; }
.btn-advanced:hover { background: #444; }
.advanced-drawer { padding: 12px; background: var(--bg-panel); border-top: 1px solid var(--border-color); }
.drawer-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
.drawer-title { font-weight: bold; color: var(--primary-color); font-size: 12px; }
.drawer-close { background: none; border: none; color: #888; cursor: pointer; font-size: 18px; }
.mat-title { font-size: 12px; color: #fff; line-height: 1.5; margin-bottom: 15px; border-bottom: 1px solid #333; padding-bottom: 10px; }
.slider-group { display: flex; align-items: center; gap: 8px; }
.slider-group input[type="range"] { flex: 1; accent-color: var(--primary-color); }
.slider-value { width: 35px; text-align: right; color: var(--text-secondary); font-size: 11px; }
.tex-section-title { background: #222; padding: 4px 8px; font-size: 11px; text-align: center; margin: 15px 0; border-radius: 2px; }
.tex-btns { display: flex; gap: 5px; }
.tex-btn { background: var(--primary-color); border: none; color: #fff; padding: 4px 10px; border-radius: 3px; font-size: 11px; cursor: pointer; }
.tex-btn.clean { background: #f56c6c; }
.mapping-box { background: #161616; padding: 12px; border-radius: var(--radius-sm); border: 1px solid #2a2a2a; margin-top: 10px; }
.mapping-title { text-align: center; font-size: 11px; color: #fff; margin-bottom: 10px; border-bottom: 1px solid #333; padding-bottom: 5px; }
.map-grid { display: grid; grid-template-columns: 40px 1fr 15px 1fr; gap: 8px; align-items: center; margin-bottom: 8px; font-size: 11px; }
.map-input { background: transparent; border: 1px solid #444; border-radius: 3px; width: 100%; text-align: center; color: #fff; height: 24px; font-size: 11px; }
.btn-apply { margin-top: 20px; background: var(--primary-color); }

.anim-list { background: #111; border: 1px solid #333; border-radius: var(--radius-sm); overflow: hidden; }
.anim-row { padding: 10px 12px; font-size: 12px; border-bottom: 1px solid #222; display: flex; justify-content: space-between; cursor: pointer; }
.anim-row:last-child { border-bottom: none; }
.anim-row:hover { background: #1a1a1a; }
.anim-row.active { background: rgba(22, 119, 255, 0.2); color: #fff; font-weight: bold; border-left: 2px solid var(--primary-color); }
.anim-duration { color: #555; font-size: 11px; }
.pagination-row { display: flex; justify-content: space-between; align-items: center; margin: 8px 0 24px 0; padding: 4px; }
.page-arrow { cursor: pointer; color: var(--primary-color); font-size: 12px; user-select: none; font-weight: bold; }
.page-info { color: #666; font-size: 11px; }
.control-area { background: #181818; padding: 16px; border-radius: var(--radius-sm); border: 1px solid #262626; }
.btn-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-top: 10px; }
.btn-action { border: none; height: 34px; border-radius: 3px; color: #fff; font-size: 12px; font-weight: bold; cursor: pointer; transition: background 0.2s; }
.btn-play { background: #1677ff; }
.btn-stop { background: #e6a23c; }
.btn-resume { background: #67c23a; }
.btn-pause { background: #606266; }
.btn-reset { background: #f56c6c; grid-column: span 2; margin-top: 4px; }

.add-viewpoint-container { margin-bottom: 20px; }
.btn-primary-add { width: 100%; height: 40px; background: var(--primary-color); border: none; color: #fff; font-weight: bold; border-radius: var(--radius-sm); cursor: pointer; display: flex; align-items: center; justify-content: center; gap: 8px; font-size: 14px; }
.btn-primary-add:hover { opacity: 0.9; }
.btn-primary-add i { font-style: normal; font-size: 20px; line-height: 1; margin-top: -2px; }
.view-list { background: #111; border: 1px solid #333; border-radius: var(--radius-sm); margin-bottom: 16px; max-height: 200px; overflow-y: auto; }
.view-row { padding: 12px; font-size: 13px; border-bottom: 1px solid #222; display: flex; align-items: center; cursor: pointer; }
.view-row:last-child { border-bottom: none; }
.view-row:hover { background: #1a1a1a; }
.view-row.active { background: rgba(22, 119, 255, 0.15); color: #fff; font-weight: bold; border-left: 3px solid var(--primary-color); }
.view-icon { margin-right: 10px; }
.view-name { flex: 1; background: transparent; border: none; color: inherit; font-size: 13px; outline: none; }
.ctrl-group-title { font-size: 11px; color: #555; margin-bottom: 12px; text-align: center; border-bottom: 1px solid #262626; padding-bottom: 8px; font-weight: bold; }
.coord-row { display: flex; gap: 10px; margin-bottom: 12px; }
.coord-item { flex: 1; }
.coord-label { display: block; font-size: 10px; color: #666; margin-bottom: 4px; text-indent: 2px; }
.coord-input { width: 100%; background: #000; border: 1px solid #333; color: var(--success-color); height: 30px; padding: 0 8px; border-radius: 2px; font-size: 12px; font-family: 'Courier New', monospace; outline: none; }
.coord-input:focus { border-color: var(--primary-color); }
.action-box { margin-top: 16px; display: flex; flex-direction: column; gap: 10px; }
.btn-secondary { width: 100%; height: 36px; background: #2a2a2a; border: 1px solid #3a3a3a; color: #ccc; border-radius: var(--radius-sm); cursor: pointer; font-size: 12px; }
.btn-secondary:hover { background: #333; color: #fff; }

.roam-content { display: flex; flex-direction: column; flex: 1; }
.panel-section { flex: 1; display: flex; flex-direction: column; min-height: 0; border-bottom: 1px solid #333; }
.panel-header { padding: 10px 14px; background: #252526; display: flex; align-items: center; justify-content: space-between; border-bottom: 1px solid #111; }
.panel-title { font-size: 12px; font-weight: bold; color: #aaa; text-transform: uppercase; letter-spacing: 1px; }
.tool-icons { display: flex; gap: 12px; color: #888; cursor: pointer; font-size: 14px; }
.tool-icons span:hover { color: var(--primary-color); }
.list-container { flex: 1; overflow-y: auto; background: #181818; padding: 2px 0; }
.list-item { padding: 8px 14px; display: flex; align-items: center; cursor: pointer; border-bottom: 1px solid #222; }
.list-item:hover { background: #222; }
.list-item.selected { background: #0d3d6b; border-left: 3px solid var(--primary-color); }
.item-name { flex: 1; background: transparent; border: none; color: #ddd; font-size: 12px; outline: none; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.item-icon { margin-right: 8px; font-size: 14px; opacity: 0.7; }
.config-detail-area { flex-shrink: 0; background: #1e1e1e; border-top: 2px solid #333; padding: 14px; }
.config-title { margin-bottom: 10px; font-size: 11px; color: #666; font-weight: bold; }
.config-grid { display: grid; gap: 10px; }
.config-row { display: flex; align-items: center; gap: 8px; }
.config-label { width: 55px; font-size: 12px; color: #888; flex-shrink: 0; }
.config-inputs { flex: 1; display: flex; gap: 4px; overflow: hidden; }
.input-box { flex: 1; min-width: 0; background: #111; border: 1px solid #333; color: #fff; height: 24px; padding: 0 4px; font-size: 11px; border-radius: 2px; font-family: monospace; }
.input-box:focus { border-color: var(--primary-color); outline: none; }
.config-time { flex: 1; display: flex; align-items: center; }
.input-box.small { width: 45px; flex: none; }
.time-unit { font-size: 10px; color: #444; margin-left: 2px; }
.fov-value { font-size: 11px; color: #555; width: 24px; }
.btn-full { width: 100%; padding: 8px; background: #722ed1; border: none; color: #fff; border-radius: var(--radius-sm); cursor: pointer; font-weight: bold; margin-top: 8px; font-size: 12px; transition: background 0.2s; }
.btn-full:hover { background: #8247d5; }
</style>