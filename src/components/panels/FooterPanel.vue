<template>
  <div class="footer-panel panel-area">
    <div class="info-group">
      <div class="info-item">
        <span>Render:</span>
        <span class="value">WebGL2</span>
      </div>
      <div class="divider"></div>
      <div class="info-item">
        <span>FPS:</span>
        <span class="value">60</span>
      </div>
      <div class="divider"></div>
      <div class="info-item">
        <span>三角面:</span>
        <span class="value">{{ stats.triangles }}</span>
      </div>
      <div class="divider"></div>
      <div class="info-item">
        <span>Draw Call:</span>
        <span class="value">{{ stats.drawCalls }}</span>
      </div>
      <div class="divider"></div>
      <div class="info-item">
        <span>纹理占用:</span>
        <span class="value">{{ stats.textureSize }}</span>
      </div>
      <div class="divider"></div>
      <div class="info-item">
        <span>世界坐标:</span>
        <span class="value">X: {{ coords.x }}  Y: {{ coords.y }}  Z: {{ coords.z }}</span>
      </div>
    </div>
    <div class="status-message">
      <span class="status-dot"></span>
      <span>场景状态: {{ statusText }}</span>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const stats = ref({
  triangles: '12,456',
  drawCalls: '128',
  textureSize: '256.8 MB'
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
  height: 32px;
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
  color: var(--text-secondary);
}

.info-item .value {
  color: var(--text-muted);
  font-family: "SF Mono", "Consolas", monospace;
}

.divider {
  width: 1px;
  height: 12px;
  background: var(--border-color);
}

.status-message {
  display: flex;
  align-items: center;
  gap: 6px;
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
