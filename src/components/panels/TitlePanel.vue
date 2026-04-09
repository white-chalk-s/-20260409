<template>
  <div class="title-panel panel-area">
    <div class="brand-info">
      <div class="brand-logo">PCI 佳都</div>
      <div class="brand-text">
        <div class="brand-title">三维场景编辑器</div>
        <div class="brand-subtitle">两江影视城站AFC智能运维</div>
      </div>
    </div>
  </div>
</template>

<script setup>
</script>

<style scoped>
.title-panel {
  display: flex;
  align-items: center;
  padding: 0 16px 0 8px;
  background: linear-gradient(90deg, #242528 0%, #1a1a1a 100%);
  border-bottom: 1px solid var(--border-color);
  height: 100%;
}

.brand-info {
  display: flex;
  align-items: center;
  gap: 12px;
}

.brand-logo {
  font-size: 22px;
  font-weight: 900;
  color: var(--brand-color);
  font-family: "Arial Black", Impact, sans-serif;
  letter-spacing: 1px;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.6);
}

.brand-text {
  display: flex;
  flex-direction: column;
}

.brand-title {
  font-size: 15px;
  font-weight: 900;
  color: #ffffff;
  letter-spacing: 1px;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}

.brand-subtitle {
  font-size: 11px;
  color: #cccccc;
  letter-spacing: 0.5px;
}
</style>
