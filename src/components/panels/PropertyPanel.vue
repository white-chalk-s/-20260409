<template>
  <div class="property-panel panel-area" v-show="!isEffectTab">
    <!-- 灯光tab时显示三维视图配置 -->
    <template v-if="isLightTab">
      <div class="panel-header">
        <div>
          <div class="panel-title">三维视图配置</div>
          <div class="panel-subtitle">场景渲染与后处理设置</div>
        </div>
      </div>
      <div class="property-content">
        <!-- 三维视图配置 -->
        <div class="property-group">
          <div class="group-title">三维视图配置</div>
          <div class="property-item">
            <label class="label">色彩空间</label>
            <select class="select">
              <option>LinearSRGBColorSpace</option>
            </select>
          </div>
          <div class="property-item">
            <label class="label">色调映射</label>
            <select class="select">
              <option>线性</option>
            </select>
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

        <!-- 环境贴图配置 -->
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

        <!-- 后处理配置 -->
        <div class="property-group">
          <div class="group-title">后处理配置</div>
          <div class="property-item switch-item">
            <label class="label">后处理</label>
            <label class="switch">
              <input type="checkbox" v-model="postProcessing.enabled" />
              <span class="slider"></span>
            </label>
            <span class="switch-text">开启</span>
          </div>
          <div class="property-item switch-item">
            <label class="label">AO</label>
            <label class="switch">
              <input type="checkbox" v-model="postProcessing.ao" />
              <span class="slider"></span>
            </label>
            <span class="switch-text">开启</span>
          </div>
          <div class="property-item switch-item">
            <label class="label">SSR</label>
            <label class="switch">
              <input type="checkbox" v-model="postProcessing.ssr" />
              <span class="slider"></span>
            </label>
            <span class="switch-text">开启</span>
          </div>
          <div class="property-item switch-item">
            <label class="label">反锯齿</label>
            <label class="switch">
              <input type="checkbox" v-model="postProcessing.antiAlias" />
              <span class="slider"></span>
            </label>
            <span class="switch-text">开启</span>
          </div>
          <div class="property-item">
            <label class="label">方法</label>
            <select class="select">
              <option>FXAA</option>
            </select>
          </div>
        </div>
      </div>
    </template>

    <!-- 模型-材质tab时显示材质库 -->
    <template v-else-if="isModelMaterialTab">
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
            <div class="mat-preview">
              <div class="mat-sphere" :style="{ background: mat.color }"></div>
            </div>
            <div class="mat-name">{{ mat.name }}</div>
          </div>
        </div>
        <div class="adv-btn-container">
          <button class="btn-advanced" @click="showAdvanced = !showAdvanced">
            高级设置 {{ showAdvanced ? '▼' : '❯' }}
          </button>
        </div>
        <!-- 高级设置抽屉 -->
        <div class="advanced-drawer" v-show="showAdvanced">
          <div class="drawer-header">
            <span class="drawer-title">高级材质配置</span>
            <button class="drawer-close" @click="showAdvanced = false">×</button>
          </div>
          <div class="mat-title">{{ selectedMaterialName }}</div>
          <div class="property-group">
            <div class="property-item">
              <label class="label">类型</label>
              <select class="select">
                <option>单面透明材质</option>
              </select>
            </div>
            <div class="property-item">
              <label class="label">颜色</label>
              <input type="color" class="color-input" v-model="materialColor" />
            </div>
            <div class="property-item">
              <label class="label">粗糙度</label>
              <div class="slider-group">
                <input type="range" v-model="roughness" min="0" max="1" step="0.01" />
                <span class="slider-value">{{ roughness.toFixed(2) }}</span>
              </div>
            </div>
            <div class="property-item">
              <label class="label">金属度</label>
              <div class="slider-group">
                <input type="range" v-model="metallic" min="0" max="1" step="0.01" />
                <span class="slider-value">{{ metallic.toFixed(2) }}</span>
              </div>
            </div>
          </div>
          <div class="tex-section-title">材质贴图</div>
          <div class="property-item">
            <label class="label">颜色贴图</label>
            <div class="tex-btns">
              <button class="tex-btn">配置</button>
              <button class="tex-btn clean">清理</button>
            </div>
          </div>
          <div class="property-item">
            <label class="label">法线贴图</label>
            <button class="tex-btn" style="width: 100%">选择</button>
          </div>
          <div class="mapping-box">
            <div class="mapping-title">贴图配置</div>
            <div class="map-grid">
              <div>重复</div>
              <input type="text" class="map-input" v-model="textureRepeat.x" />
              <div style="text-align: center">×</div>
              <input type="text" class="map-input" v-model="textureRepeat.y" />
            </div>
            <div class="map-grid">
              <div>偏移</div>
              <input type="text" class="map-input" v-model="textureOffset.x" />
              <div style="text-align: center">×</div>
              <input type="text" class="map-input" v-model="textureOffset.y" />
            </div>
          </div>
          <button class="btn-advanced btn-apply" @click="showAdvanced = false">应用并关闭</button>
        </div>
      </div>
    </template>

    <!-- 模型-动画tab时显示动画配置 -->
    <template v-else-if="isModelAnimationTab">
      <div class="panel-header">
        <div>
          <div class="panel-title">动画配置</div>
          <div class="panel-subtitle">动画列表与播放控制</div>
        </div>
      </div>
      <div class="property-content">
        <div class="group-title">动画列表</div>
        <div class="anim-list">
          <div
            v-for="anim in animationList"
            :key="anim.id"
            :class="['anim-row', { active: selectedAnimId === anim.id }]"
            @click="selectedAnimId = anim.id"
          >
            <span>{{ anim.name }}</span>
            <span class="anim-duration">{{ anim.duration }}</span>
          </div>
        </div>
        <div class="pagination-row">
          <span class="page-arrow">❮ PREV</span>
          <span class="page-info">1 / 4</span>
          <span class="page-arrow">NEXT ❯</span>
        </div>

        <div class="group-title">播放属性</div>
        <div class="control-area">
          <div class="property-item">
            <label class="label">循环模式</label>
            <select class="select">
              <option>LoopRepeat (无限循环)</option>
              <option>LoopOnce (播放一次)</option>
              <option>LoopPingPong (往返循环)</option>
            </select>
          </div>
          <div class="property-item">
            <label class="label">播放速度</label>
            <select class="select">
              <option>1x (默认速度)</option>
              <option>2x (快速)</option>
              <option>0.5x (慢放)</option>
            </select>
          </div>
          <div class="btn-grid">
            <button class="btn-action btn-play" @click="playAnim">播放</button>
            <button class="btn-action btn-stop" @click="stopAnim">停止</button>
            <button class="btn-action btn-resume" @click="resumeAnim">继续</button>
            <button class="btn-action btn-pause" @click="pauseAnim">暂停</button>
            <button class="btn-action btn-reset" @click="resetAnim">重置动画状态</button>
          </div>
        </div>
      </div>
    </template>

    <!-- 其他tab时显示设备属性 -->
    <template v-else>
      <div class="panel-header">
        <div>
          <div class="panel-title">{{ panelTitle }}</div>
          <div class="panel-subtitle">
            {{ selectedDevice ? `已选择: ${selectedDevice.name}` : '请选择设备' }}
          </div>
        </div>
      </div>
      <div class="property-content">
        <!-- 有选中设备时显示详细信息 -->
        <template v-if="selectedDevice">
          <!-- 基本信息 -->
          <div class="property-group">
            <div class="group-title">基本信息</div>
            <div class="property-item">
              <label class="label">设备唯一编号</label>
              <div class="val-text">1773978151443</div>
            </div>
            <div class="property-item">
              <label class="label">设备名称</label>
              <input type="text" class="input" v-model="selectedDevice.name" />
            </div>
            <div class="property-item">
              <label class="label">线路名称</label>
              <div class="val-text">4号线二期</div>
            </div>
            <div class="property-item">
              <label class="label">车站名称</label>
              <div class="val-text">两江影视城站</div>
            </div>
            <div class="property-item">
              <label class="label">位置</label>
              <div class="val-text">站厅层-西侧</div>
            </div>
            <div class="property-item">
              <label class="label">设备专业</label>
              <div class="val-text">AFC</div>
            </div>
            <div class="property-item">
              <label class="label">设备类型</label>
              <div class="val-text">AGM</div>
            </div>
            <div class="property-item switch-item">
              <label class="label">是否可见</label>
              <label class="switch">
                <input type="checkbox" v-model="selectedDevice.visible" />
                <span class="slider"></span>
              </label>
            </div>
          </div>

          <!-- 更多详情 -->
          <div class="more-link" @click="showMoreDetail = !showMoreDetail">
            更多详情 >>
          </div>
          <div class="data-view-container" v-show="showMoreDetail">
            <div class="data-view-tabs">
              <div :class="['data-view-tab', { active: dataViewType === 'table' }]" @click="dataViewType = 'table'">表格视图</div>
              <div :class="['data-view-tab', { active: dataViewType === 'json' }]" @click="dataViewType = 'json'">JSON视图</div>
            </div>
            <div class="data-content" v-show="dataViewType === 'table'">
              <table class="data-table">
                <tr><td style="color: #888;">equipmentInfo</td><td style="color: var(--primary-color); cursor: pointer;">查看</td></tr>
                <tr><td style="color: #888;">fileInfo</td><td style="color: var(--primary-color); cursor: pointer;">查看</td></tr>
                <tr><td style="color: #888;">xrayed</td><td>false</td></tr>
              </table>
            </div>
            <div class="data-content" v-show="dataViewType === 'json'">
              <div class="json-box">{"equipmentInfo":{"uniqueId":1773978151443},"xrayed":false}</div>
            </div>
          </div>

          <!-- 变换 -->
          <div class="property-group">
            <div class="group-title">变换</div>
            <div class="property-item">
              <label class="label">位置</label>
              <div class="input-group">
                <div class="input-item">
                  <input type="text" class="input" v-model="position.x" />
                  <div class="input-label">X</div>
                </div>
                <div class="input-item">
                  <input type="text" class="input" v-model="position.y" />
                  <div class="input-label">Y</div>
                </div>
                <div class="input-item">
                  <input type="text" class="input" v-model="position.z" />
                  <div class="input-label">Z</div>
                </div>
              </div>
            </div>
            <div class="property-item">
              <label class="label">旋转</label>
              <div class="input-group">
                <div class="input-item">
                  <input type="text" class="input" v-model="rotation.x" />
                  <div class="input-label">X</div>
                </div>
                <div class="input-item">
                  <input type="text" class="input" v-model="rotation.y" />
                  <div class="input-label">Y</div>
                </div>
                <div class="input-item">
                  <input type="text" class="input" v-model="rotation.z" />
                  <div class="input-label">Z</div>
                </div>
              </div>
            </div>
            <div class="property-item">
              <label class="label">缩放</label>
              <div class="input-group">
                <div class="input-item">
                  <input type="text" class="input" v-model="scale.x" />
                  <div class="input-label">X</div>
                </div>
                <div class="input-item">
                  <input type="text" class="input" v-model="scale.y" />
                  <div class="input-label">Y</div>
                </div>
                <div class="input-item">
                  <input type="text" class="input" v-model="scale.z" />
                  <div class="input-label">Z</div>
                </div>
              </div>
            </div>
          </div>

          <!-- 动画 -->
          <div class="property-group">
            <div class="group-title">动画</div>
            <div class="property-item">
              <label class="label">动画选择</label>
              <select class="select">
                <option>Take 001</option>
              </select>
            </div>
            <div class="property-item">
              <label class="label">循环模式</label>
              <select class="select">
                <option>LoopRepeat</option>
              </select>
            </div>
            <div class="property-item">
              <label class="label">播放控制</label>
              <div class="action-grid">
                <button class="act-btn btn-blue" @click="playAnim">播放</button>
                <button class="act-btn btn-orange" @click="stopAnim">停止</button>
                <button class="act-btn btn-green" @click="resumeAnim">继续</button>
                <button class="act-btn btn-grey" @click="pauseAnim">暂停</button>
                <button class="act-btn btn-red" @click="resetAnim">重置</button>
              </div>
            </div>
          </div>

          <!-- 显示状态 -->
          <div class="property-group">
            <div class="group-title">显示状态</div>
            <div class="property-item switch-item">
              <label class="label">是否显示</label>
              <label class="switch">
                <input type="checkbox" v-model="selectedDevice.visible" />
                <span class="slider"></span>
              </label>
            </div>
            <div class="property-item switch-item">
              <label class="label">是否锁定</label>
              <label class="switch">
                <input type="checkbox" v-model="selectedDevice.locked" />
                <span class="slider"></span>
              </label>
            </div>
          </div>
        </template>

        <!-- 未选中设备时显示空状态 -->
        <template v-else>
          <div class="empty-state">
            <div class="empty-icon">📦</div>
            <div class="empty-text">请在左侧选择设备</div>
            <div class="empty-hint">点击设备节点查看属性</div>
          </div>
        </template>
      </div>
    </template>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  tabId: {
    type: String,
    default: 'scene'
  },
  subTabId: {
    type: String,
    default: 'object'
  },
  selectedDevice: {
    type: Object,
    default: null
  }
})

// 是否是灯光tab
const isLightTab = computed(() => props.tabId === 'scene' && props.subTabId === 'light')

// 是否是特效tab
const isEffectTab = computed(() => props.tabId === 'scene' && props.subTabId === 'effect')

// 是否是模型-材质tab
const isModelMaterialTab = computed(() => props.tabId === 'model' && props.subTabId === 'material')

// 是否是模型-动画tab
const isModelAnimationTab = computed(() => props.tabId === 'model' && props.subTabId === 'animation')

const panelTitle = computed(() => '属性配置')

// 更多详情
const showMoreDetail = ref(false)
const dataViewType = ref('table')

// 灯光配置数据
const exposure = ref(1)
const backgroundColor = ref('#000000')
const envMapPath = ref('/attachment...')
const envIntensity = ref(0.85)
const postProcessing = ref({
  enabled: true,
  ao: true,
  ssr: false,
  antiAlias: true
})

// 设备属性数据
const position = ref({ x: '0.00', y: '0.00', z: '0.00' })
const rotation = ref({ x: '0', y: '0', z: '0' })
const scale = ref({ x: '1.0', y: '1.0', z: '1.0' })

// 材质库数据
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

function selectMaterial(mat) {
  selectedMaterialId.value = mat.id
  materialColor.value = mat.color
}

// 动画列表数据
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

// 动画控制
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
.property-panel {
  display: flex;
  flex-direction: column;
  border-left: 1px solid var(--border-color);
}

.panel-header {
  padding: 14px 16px;
  border-bottom: 1px solid var(--border-color);
}

.property-content {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
}

.property-group {
  margin-bottom: 20px;
}

.group-title {
  font-size: 11px;
  font-weight: 600;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 12px;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border-light);
}

.property-item {
  margin-bottom: 14px;
}

.property-item .label {
  display: block;
  font-size: 12px;
  color: var(--text-secondary);
  margin-bottom: 6px;
}

/* 纯文本值 */
.val-text {
  font-size: 12px;
  color: var(--text-primary);
}

.input-group {
  display: flex;
  gap: 8px;
}

.input-item {
  flex: 1;
}

.input-item .input {
  text-align: center;
}

.input-label {
  font-size: 10px;
  color: var(--text-muted);
  text-align: center;
  margin-top: 4px;
}

.switch-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.switch-item .label {
  margin-bottom: 0;
}

/* 更多详情 */
.more-link {
  color: var(--primary-color);
  cursor: pointer;
  font-size: 12px;
  text-decoration: underline;
  margin: 4px 0 10px 0;
}

.data-view-container {
  border: 1px solid var(--border-color);
  border-radius: var(--radius-sm);
  margin-bottom: 10px;
  overflow: hidden;
}

.data-view-tabs {
  display: flex;
  background: #1a1a1a;
  border-bottom: 1px solid var(--border-color);
}

.data-view-tab {
  flex: 1;
  padding: 8px;
  text-align: center;
  font-size: 11px;
  cursor: pointer;
  color: var(--text-secondary);
}

.data-view-tab.active {
  color: #fff;
  background: var(--primary-color);
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 11px;
  table-layout: fixed;
}

.data-table td {
  border: 1px solid var(--border-color);
  padding: 6px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.json-box {
  background: #000;
  color: #ce9178;
  padding: 8px;
  font-family: monospace;
  font-size: 11px;
  height: 100px;
  overflow-y: auto;
  white-space: pre-wrap;
  word-break: break-all;
}

/* 动画按钮 */
.action-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
  margin-top: 4px;
  width: 100%;
}

.act-btn {
  border: none;
  border-radius: 3px;
  height: 28px;
  color: #fff;
  font-size: 12px;
  cursor: pointer;
}

.btn-blue { background: #1677ff; }
.btn-orange { background: #e6a23c; }
.btn-green { background: #67c23a; }
.btn-grey { background: #606266; }
.btn-red {
  background: #f56c6c;
  width: calc(50% - 4px);
  margin-top: 8px;
}

/* 步进器 */
.stepper {
  display: flex;
  width: 100%;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-sm);
  overflow: hidden;
}

.step-btn {
  width: 28px;
  height: 26px;
  background: #262626;
  border: none;
  color: #888;
  cursor: pointer;
  font-size: 14px;
}

.step-btn:hover {
  background: #333;
  color: #fff;
}

.step-val {
  flex: 1;
  background: var(--input-bg);
  border: none;
  color: var(--text-primary);
  text-align: center;
  font-size: 12px;
  border-left: 1px solid var(--border-color);
  border-right: 1px solid var(--border-color);
}

/* 颜色选择器 */
.color-picker {
  display: flex;
  align-items: center;
  gap: 8px;
}

.color-box {
  width: 28px;
  height: 28px;
  border: 1px solid var(--border-color);
  border-radius: var(--radius-sm);
}

.color-input {
  width: 28px;
  height: 28px;
  padding: 0;
  border: none;
  background: transparent;
  cursor: pointer;
}

/* 文件输入 */
.file-input {
  display: flex;
  width: 100%;
}

.file-input .input {
  flex: 1;
  border-radius: var(--radius-sm) 0 0 var(--radius-sm);
  text-align: left;
  padding-left: 8px;
  font-size: 11px;
}

.btn-select {
  background: var(--primary-color);
  border: none;
  color: #fff;
  padding: 0 12px;
  border-radius: 0 var(--radius-sm) var(--radius-sm) 0;
  cursor: pointer;
  font-size: 12px;
}

/* 开关带文字 */
.switch-text {
  margin-left: 8px;
  color: var(--primary-color);
  font-size: 12px;
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

/* 材质库 */
.mat-category {
  font-size: 12px;
  color: #f1c40f;
  display: flex;
  align-items: center;
  gap: 6px;
  margin-bottom: 10px;
}

.mat-sub-category {
  padding-left: 20px;
  font-size: 12px;
  color: #ccc;
  margin-bottom: 6px;
  cursor: pointer;
}

.mat-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}

.mat-card {
  background: #161616;
  border: 1px solid #333;
  border-radius: var(--radius-sm);
  overflow: hidden;
  cursor: pointer;
  transition: 0.2s;
}

.mat-card:hover {
  border-color: var(--primary-color);
}

.mat-preview {
  width: 100%;
  aspect-ratio: 1;
  background: radial-gradient(circle, #444 0%, #111 100%);
  display: flex;
  align-items: center;
  justify-content: center;
}

.mat-sphere {
  width: 70%;
  height: 70%;
  border-radius: 50%;
  box-shadow: inset -10px -10px 20px rgba(0,0,0,0.8);
}

.mat-name {
  padding: 6px;
  font-size: 11px;
  text-align: center;
  color: #aaa;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* 高级设置按钮 */
.adv-btn-container {
  padding: 12px;
  border-top: 1px solid #333;
  background: #1a1a1a;
}

.btn-advanced {
  width: 100%;
  padding: 8px;
  background: #333;
  border: 1px solid #444;
  color: #fff;
  border-radius: var(--radius-sm);
  cursor: pointer;
  font-size: 12px;
  font-weight: bold;
}

.btn-advanced:hover {
  background: #444;
}

/* 高级设置抽屉 */
.advanced-drawer {
  padding: 12px;
  background: var(--bg-panel);
  border-top: 1px solid var(--border-color);
}

.drawer-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.drawer-title {
  font-weight: bold;
  color: var(--primary-color);
  font-size: 12px;
}

.drawer-close {
  background: none;
  border: none;
  color: #888;
  cursor: pointer;
  font-size: 18px;
}

.mat-title {
  font-size: 12px;
  color: #fff;
  line-height: 1.5;
  margin-bottom: 15px;
  border-bottom: 1px solid #333;
  padding-bottom: 10px;
}

.slider-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.slider-group input[type="range"] {
  flex: 1;
  accent-color: var(--primary-color);
}

.slider-value {
  width: 35px;
  text-align: right;
  color: var(--text-secondary);
  font-size: 11px;
}

.tex-section-title {
  background: #222;
  padding: 4px 8px;
  font-size: 11px;
  text-align: center;
  margin: 15px 0;
  border-radius: 2px;
}

.tex-btns {
  display: flex;
  gap: 5px;
}

.tex-btn {
  background: var(--primary-color);
  border: none;
  color: #fff;
  padding: 4px 10px;
  border-radius: 3px;
  font-size: 11px;
  cursor: pointer;
}

.tex-btn.clean {
  background: #f56c6c;
}

.mapping-box {
  background: #161616;
  padding: 12px;
  border-radius: var(--radius-sm);
  border: 1px solid #2a2a2a;
  margin-top: 10px;
}

.mapping-title {
  text-align: center;
  font-size: 11px;
  color: #fff;
  margin-bottom: 10px;
  border-bottom: 1px solid #333;
  padding-bottom: 5px;
}

.map-grid {
  display: grid;
  grid-template-columns: 40px 1fr 15px 1fr;
  gap: 8px;
  align-items: center;
  margin-bottom: 8px;
  font-size: 11px;
}

.map-input {
  background: transparent;
  border: 1px solid #444;
  border-radius: 3px;
  width: 100%;
  text-align: center;
  color: #fff;
  height: 24px;
  font-size: 11px;
}

.btn-apply {
  margin-top: 20px;
  background: var(--primary-color);
}

/* 动画列表 */
.anim-list {
  background: #111;
  border: 1px solid #333;
  border-radius: var(--radius-sm);
  overflow: hidden;
}

.anim-row {
  padding: 10px 12px;
  font-size: 12px;
  border-bottom: 1px solid #222;
  display: flex;
  justify-content: space-between;
  cursor: pointer;
}

.anim-row:last-child {
  border-bottom: none;
}

.anim-row:hover {
  background: #1a1a1a;
}

.anim-row.active {
  background: rgba(22, 119, 255, 0.2);
  color: #fff;
  font-weight: bold;
  border-left: 2px solid var(--primary-color);
}

.anim-duration {
  color: #555;
  font-size: 11px;
}

.pagination-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 8px 0 24px 0;
  padding: 4px;
}

.page-arrow {
  cursor: pointer;
  color: var(--primary-color);
  font-size: 12px;
  user-select: none;
  font-weight: bold;
}

.page-info {
  color: #666;
  font-size: 11px;
}

/* 控制区域 */
.control-area {
  background: #181818;
  padding: 16px;
  border-radius: var(--radius-sm);
  border: 1px solid #262626;
}

.control-area .property-item {
  margin-bottom: 12px;
}

.control-area .label {
  margin-bottom: 6px;
}

/* 动画按钮组 */
.btn-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
  margin-top: 10px;
}

.btn-action {
  border: none;
  height: 34px;
  border-radius: 3px;
  color: #fff;
  font-size: 12px;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.2s;
}

.btn-play { background: #1677ff; }
.btn-stop { background: #e6a23c; }
.btn-resume { background: #67c23a; }
.btn-pause { background: #606266; }
.btn-reset {
  background: #f56c6c;
  grid-column: span 2;
  margin-top: 4px;
}

/* 模型树样式 */
.model-view {
  display: flex;
  flex-direction: column;
  flex: 1;
  overflow: hidden;
}

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

.tree-list {
  flex: 1;
  overflow-y: auto;
  padding: 8px 0;
}

.tree-node {
  padding: 8px 4px;
  display: flex;
  align-items: center;
  cursor: pointer;
  font-size: 13px;
  color: #ccc;
}

.tree-node:hover {
  background: var(--tree-hover, #262626);
}

.tree-node.selected {
  background: #0d3d6b;
  color: #4facfe;
  border-right: 2px solid var(--primary-color);
}

.tree-node.folder {
  color: #f1c40f;
  font-weight: 500;
}

.node-arrow {
  margin-right: 8px;
  font-size: 10px;
  color: #666;
  width: 12px;
}

.node-icon {
  margin-right: 8px;
  opacity: 0.8;
}

.node-name {
  flex: 1;
}

.node-actions {
  display: flex;
  gap: 8px;
  padding-right: 4px;
}

.action-icon {
  font-size: 12px;
  opacity: 0.5;
}

.action-icon.active-target {
  color: #ff4d4f;
  opacity: 1;
}

.action-icon.active-lock {
  color: #f1c40f;
  opacity: 1;
}

.action-icon.active-eye {
  color: #fff;
  opacity: 1;
}
</style>