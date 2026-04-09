<template>
  <div class="settings-panel panel-area">
    <!-- 场景信息开关 -->
    <button class="btn">
      <span>👁️</span> 场景信息
      <label class="switch-mini">
        <input type="checkbox" checked />
        <span class="slider-mini"></span>
      </label>
    </button>

    <!-- 预览按钮 -->
    <button class="btn">
      <span>▶</span> 预览
    </button>

    <!-- 保存按钮 -->
    <button class="btn btn-primary">
      <span>💾</span> 保存
    </button>

    <!-- 帮助下拉 - 点击触发 -->
    <div class="dropdown" @click="showMenu = !showMenu">
      <button class="icon-btn-header" title="帮助与支持">❓</button>
      <div class="dropdown-menu" v-show="showMenu">
        <div class="dropdown-item">⌨️ 快捷键</div>
        <div class="dropdown-item">📄 操作文档</div>
        <div class="dropdown-item">📖 操作指南</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const showMenu = ref(false)
</script>

<style scoped>
.settings-panel {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 8px;
  padding: 0 16px;
  background: linear-gradient(90deg, #242528 0%, #1a1a1a 100%);
  border-bottom: 1px solid #222;
  height: 100%;
  position: relative;
  z-index: 100;
}

.btn {
  background: #2a2a2a;
  border: 1px solid #3a3a3a;
  color: var(--text-primary);
  padding: 4px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  display: flex;
  align-items: center;
  gap: 4px;
  transition: 0.2s;
}

.btn:hover {
  background: #333;
  border-color: #555;
}

.btn-primary {
  background: var(--primary-color);
  border-color: var(--primary-color);
  color: #fff;
}

.btn-primary:hover {
  background: var(--primary-hover);
  border-color: var(--primary-hover);
}

.icon-btn-header {
  background: transparent;
  border: 1px solid transparent;
  color: var(--text-secondary);
  width: 26px;
  height: 26px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 14px;
  transition: 0.2s;
}

.icon-btn-header:hover {
  background: #2a2a2a;
  color: #fff;
  border-color: #3a3a3a;
}

/* 下拉菜单 */
.dropdown {
  position: relative;
  display: flex;
  align-items: center;
  cursor: pointer;
}

.dropdown-menu {
  position: fixed;
  top: 90px;
  right: 16px;
  background: #1f1f1f;
  border: 1px solid #3a3a3a;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.5);
  flex-direction: column;
  width: 130px;
  padding: 4px 0;
  z-index: 9999;
}

.dropdown-item {
  padding: 8px 12px;
  color: var(--text-main);
  font-size: 12px;
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.dropdown-item:hover {
  background: #2a2a2a;
  color: var(--primary-color);
}

/* 开关控件 */
.switch-mini {
  position: relative;
  display: inline-block;
  width: 24px;
  height: 12px;
  margin-left: 6px;
}

.switch-mini input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider-mini {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #444;
  border-radius: 12px;
  transition: 0.3s;
}

.slider-mini:before {
  position: absolute;
  content: "";
  height: 8px;
  width: 8px;
  left: 2px;
  bottom: 2px;
  background-color: white;
  border-radius: 50%;
  transition: 0.3s;
}

.switch-mini input:checked + .slider-mini {
  background-color: var(--primary-color);
}

.switch-mini input:checked + .slider-mini:before {
  transform: translateX(12px);
}
</style>
