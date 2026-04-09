<template>
  <div class="settings-panel panel-area">
    <button class="btn">
      <span>↺</span> 重置
    </button>
    <button class="btn">
      <span>👁</span> 预览
    </button>
    <button class="btn">
      <span>📤</span> 导出
    </button>
    <button class="btn btn-primary">
      <span>💾</span> 保存
    </button>
  </div>
</template>

<script setup>
</script>

<style scoped>
.settings-panel {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 8px;
  padding: 8px 16px;
  background: var(--bg-header);
  border-bottom: 1px solid var(--border-color);
  border-left: 1px solid var(--border-color);
}
</style>
