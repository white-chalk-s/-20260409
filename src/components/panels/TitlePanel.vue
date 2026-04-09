<template>
  <div class="title-panel panel-area">
    <div class="brand-info">
      <div class="brand-logo">PCI 佳都</div>
      <div class="brand-text">
        <div class="brand-title">三维场景编辑器</div>
        <div class="brand-subtitle">数字孪生平台</div>
      </div>
    </div>
    <div class="status-tag">原型设计中</div>
  </div>
</template>

<script setup>
</script>

<style scoped>
.title-panel {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  background: var(--bg-header);
  border-bottom: 1px solid var(--border-color);
}

.brand-info {
  display: flex;
  align-items: center;
  gap: 12px;
}

.brand-logo {
  font-size: 20px;
  font-weight: 900;
  color: var(--brand-color);
  letter-spacing: 1px;
}

.brand-text {
  display: flex;
  flex-direction: column;
}

.brand-title {
  font-size: 14px;
  font-weight: 600;
  color: var(--text-primary);
}

.brand-subtitle {
  font-size: 11px;
  color: var(--text-secondary);
}

.status-tag {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 4px 10px;
  background: rgba(22, 119, 255, 0.15);
  color: var(--primary-color);
  font-size: 11px;
  border-radius: 12px;
}

.status-tag::before {
  content: "";
  width: 6px;
  height: 6px;
  background: var(--primary-color);
  border-radius: 50%;
}
</style>
