<template>
  <div class="footer-panel panel-area">
    <div class="info-group">
      <div class="info-item">
        <span class="label">三角面:</span>
        <span class="value">{{ stats.triangles }}</span>
      </div>
      <div class="divider"></div>
      <div class="info-item">
        <span class="label">Draw Call:</span>
        <span class="value">{{ stats.drawCalls }}</span>
      </div>
      <div class="divider"></div>
      <div class="info-item">
        <span class="label">纹理占用:</span>
        <span class="value">{{ stats.textureSize }}</span>
      </div>
      <div class="divider"></div>
      <div class="info-item">
        <span class="label">世界坐标:</span>
        <span class="value">X: {{ coords.x }}  Y: {{ coords.y }}  Z: {{ coords.z }}</span>
      </div>
      <div class="divider"></div>
      <div class="info-item">
        <span class="label">相机高度:</span>
        <span class="value">{{ stats.cameraHeight }}</span>
      </div>
      <div class="divider"></div>
      <div class="info-item">
        <span class="label">缩放比例:</span>
        <span class="value">{{ stats.zoom }}</span>
      </div>
    </div>
    <div class="status-message">
      <span class="status-dot"></span>
      {{ statusText }}
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const stats = ref({
  triangles: '12,456',
  drawCalls: '128',
  textureSize: '256.8 MB',
  cameraHeight: '15.0m',
  zoom: '1.0x'
})

const coords = ref({
  x: '0.00',
  y: '0.00',
  z: '0.00'
})

const statusText = ref('就绪')
</script>

<style scoped>
.footer-panel {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  background: var(--bg-header);
  border-top: 1px solid var(--border-color);
}

.info-group {
  display: flex;
  align-items: center;
  gap: 16px;
}

.info-item {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 11px;
}

.info-item .label {
  color: var(--text-muted);
}

.info-item .value {
  color: var(--text-secondary);
  font-family: "SF Mono", "Consolas", monospace;
}

.status-message {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 4px 12px;
  background: rgba(34, 197, 94, 0.1);
  border: 1px solid rgba(34, 197, 94, 0.3);
  border-radius: var(--radius-sm);
  font-size: 11px;
  color: var(--success-color);
}

.status-dot {
  width: 6px;
  height: 6px;
  background: var(--success-color);
  border-radius: 50%;
}
</style>
