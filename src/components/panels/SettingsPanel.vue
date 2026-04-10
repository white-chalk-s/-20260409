<template>
  <div class="settings-panel panel-area">
    <!-- 场景信息开关 -->
    <div class="switch-group">
      <span class="label">
        <svg viewBox="0 0 24 24" width="14" height="14" stroke="currentColor" stroke-width="2" fill="none">
          <circle cx="12" cy="12" r="10"></circle>
          <line x1="12" y1="16" x2="12" y2="12"></line>
          <line x1="12" y1="8" x2="12.01" y2="8"></line>
        </svg>
        场景信息
      </span>
      <label class="custom-switch">
        <input type="checkbox" checked />
        <span class="switch-slider"></span>
      </label>
    </div>

    <div class="divider"></div>

    <!-- 预览按钮 -->
    <button class="btn btn-secondary">
      <svg viewBox="0 0 24 24" width="14" height="14" fill="currentColor">
        <path d="M8 5v14l11-7z"/>
      </svg>
      预览
    </button>

    <!-- 导出按钮 -->
    <button class="btn btn-secondary">
      <svg viewBox="0 0 24 24" width="14" height="14" stroke="currentColor" stroke-width="2" fill="none">
        <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
        <polyline points="7 10 12 15 17 10"></polyline>
        <line x1="12" y1="15" x2="12" y2="3"></line>
      </svg>
      导出
    </button>

    <!-- 保存按钮 -->
    <button class="btn btn-primary">
      <svg viewBox="0 0 24 24" width="14" height="14" stroke="currentColor" stroke-width="2" fill="none">
        <path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"></path>
        <polyline points="17 21 17 13 7 13 7 21"></polyline>
        <polyline points="7 3 7 8 15 8"></polyline>
      </svg>
      保存
    </button>

    <div class="divider"></div>

    <!-- 帮助菜单 - 点击触发 -->
    <div class="dropdown" @click="toggleMenu">
      <button class="btn-icon" title="帮助及文档">
        <svg viewBox="0 0 24 24" width="16" height="16" stroke="currentColor" stroke-width="2" fill="none">
          <circle cx="12" cy="12" r="10"></circle>
          <path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"></path>
          <line x1="12" y1="17" x2="12.01" y2="17"></line>
        </svg>
      </button>

      <!-- 下拉菜单 -->
      <div class="dropdown-menu" v-show="showMenu">
        <a class="dropdown-item">
          <svg viewBox="0 0 24 24" width="14" height="14" stroke="currentColor" stroke-width="2" fill="none">
            <path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"></path>
            <polyline points="3.27 6.96 12 12.01 20.73 6.96"></polyline>
            <line x1="12" y1="22.08" x2="12" y2="12"></line>
          </svg>
          快捷键
        </a>
        <a class="dropdown-item">
          <svg viewBox="0 0 24 24" width="14" height="14" stroke="currentColor" stroke-width="2" fill="none">
            <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
            <polyline points="14 2 14 8 20 8"></polyline>
            <line x1="16" y1="13" x2="8" y2="13"></line>
            <line x1="16" y1="17" x2="8" y2="17"></line>
          </svg>
          操作文档
        </a>
        <a class="dropdown-item">
          <svg viewBox="0 0 24 24" width="14" height="14" stroke="currentColor" stroke-width="2" fill="none">
            <circle cx="12" cy="12" r="10"></circle>
            <polygon points="16.24 7.76 14.12 14.12 7.76 16.24 9.88 9.88 16.24 7.76"></polygon>
          </svg>
          操作指南
        </a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const showMenu = ref(false)

function toggleMenu() {
  console.log('toggleMenu clicked, showMenu:', !showMenu.value)
  showMenu.value = !showMenu.value
}
</script>

<style scoped>
.settings-panel {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 0 16px;
  background: #1f1f1f;
  border-bottom: 1px solid #303030;
  height: 100%;
  white-space: nowrap;
  position: relative;
  z-index: 1000;
}

.divider {
  width: 1px;
  height: 18px;
  background-color: #434343;
  margin: 0 4px;
}

.switch-group {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 4px 10px;
  border-radius: 4px;
  background: #2b2b2b;
  border: 1px solid transparent;
  transition: all 0.2s;
}

.switch-group:hover {
  background: #3a3a3a;
}

.switch-group .label {
  font-size: 12px;
  color: #e0e0e0;
  display: flex;
  align-items: center;
  gap: 4px;
  white-space: nowrap;
}

.custom-switch {
  position: relative;
  display: inline-block;
  width: 32px;
  height: 18px;
}

.custom-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.switch-slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #434343;
  transition: 0.3s;
  border-radius: 18px;
}

.switch-slider:before {
  position: absolute;
  content: "";
  height: 14px;
  width: 14px;
  left: 2px;
  bottom: 2px;
  background-color: white;
  transition: 0.3s;
  border-radius: 50%;
}

.custom-switch input:checked + .switch-slider {
  background-color: #1890ff;
}

.custom-switch input:checked + .switch-slider:before {
  transform: translateX(14px);
}

.btn {
  height: 28px;
  padding: 0 12px;
  border-radius: 4px;
  font-size: 12px;
  display: flex;
  align-items: center;
  gap: 4px;
  cursor: pointer;
  border: 1px solid transparent;
  transition: all 0.2s;
  color: #e0e0e0;
  white-space: nowrap;
}

.btn svg {
  width: 12px;
  height: 12px;
  flex-shrink: 0;
}

.btn-secondary {
  background: #2b2b2b;
  border-color: #303030;
}

.btn-secondary:hover {
  background: #3a3a3a;
  border-color: #434343;
  color: #fff;
}

.btn-primary {
  background: #1890ff;
  color: #fff;
}

.btn-primary:hover {
  background: #40a9ff;
}

.btn-icon {
  width: 28px;
  height: 28px;
  border-radius: 4px;
  background: transparent;
  border: none;
  color: #8c8c8c;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s;
  flex-shrink: 0;
}

.btn-icon:hover {
  background: #2b2b2b;
  color: #e0e0e0;
}

.dropdown {
  position: relative;
  display: inline-block;
  pointer-events: auto;
}

.dropdown-menu {
  position: absolute;
  top: 100%;
  right: 0;
  margin-top: 8px;
  background: #1f1f1f;
  border: 1px solid #303030;
  border-radius: 6px;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.5);
  min-width: 140px;
  padding: 6px 0;
  z-index: 99999;
  pointer-events: auto;
}

.dropdown-menu::before {
  content: '';
  position: absolute;
  top: -5px;
  right: 11px;
  width: 8px;
  height: 8px;
  background: #1f1f1f;
  border-top: 1px solid #303030;
  border-left: 1px solid #303030;
  transform: rotate(45deg);
}

.dropdown-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 14px;
  color: #e0e0e0;
  font-size: 12px;
  text-decoration: none;
  cursor: pointer;
  transition: background 0.2s;
  white-space: nowrap;
}

.dropdown-item svg {
  width: 12px;
  height: 12px;
  color: #8c8c8c;
  transition: color 0.2s;
  flex-shrink: 0;
}

.dropdown-item:hover {
  background: #3a3a3a;
  color: #fff;
}

.dropdown-item:hover svg {
  color: #1890ff;
}
</style>
