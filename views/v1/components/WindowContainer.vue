<template>
  <div 
    class="window-container" 
    :style="cptWindowStyle"
    @mousedown="handleFocus">
    
    <!-- Title Bar (Drag Handle) -->
    <div 
      class="window-container__titlebar" 
      @dblclick="handleToggleMaximize">
      <div class="window-container__title">
        <div class="window-container__icon" :style="{ color: appColor }">
          {{ appIcon }}
        </div>
        <div class="window-container__title-text">{{ window.title }}</div>
      </div>
      
      <div class="window-container__controls" @mousedown.stop>
        <button class="window-container__control-btn" @click="handleMinimize" title="Minimize">
          <xIcon icon="minus" size="18" />
        </button>
        <button class="window-container__control-btn" @click="handleToggleMaximize" title="Toggle Maximize">
          <xIcon icon="full-screen" size="16" />
        </button>
        <button class="window-container__control-btn window-container__control-btn--close" @click="handleClose" title="Close">
          <xIcon icon="close" size="18" />
        </button>
      </div>
    </div>

    <!-- Content Area -->
    <div class="window-container__content">
      <!-- 内容由 ModalManager 内部管理 -->
    </div>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    props: {
      window: { type: Object, required: true },
      ModalManager: { type: Object, required: true },
      system: { type: Object, required: true }
    },
    computed: {
      cptWindowStyle() {
        // 适配新的 ModalManager 窗口实例结构
        return {
          zIndex: this.window.viewerZIndex || 1000,
          position: 'fixed'
        };
      },
      appIcon() {
        const app = this.system.apps.find(a => a.id === this.window.appId);
        return app ? app.name.charAt(0) : '?';
      },
      appColor() {
        const app = this.system.apps.find(a => a.id === this.window.appId);
        return app ? app.color : '#3182ce';
      }
    },
    methods: {
      handleFocus() {
        this.ModalManager.toTop(this.window.id);
      },
      handleClose() {
        this.ModalManager.close(this.window.id);
      },
      handleMinimize() {
        this.ModalManager.minimize(this.window.id);
      },
      handleToggleMaximize() {
        this.ModalManager.maximize(this.window.id);
      },
      
      // Drag & Resize Logic
      startDrag(e) {
        // 暂时禁用拖拽功能，因为新的 ModalManager 可能不支持
        return;
      },

      startResize(e, direction) {
        // 暂时禁用 resize 功能，因为新的 ModalManager 可能不支持
        return;
      }
    }
  };
}
</script>

<style lang="less">
.window-container {
  position: fixed;
  display: flex;
  flex-direction: column;
  background: var(--v1-shell-surface);
  border-radius: 12px;
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.15), 0 2px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  border: 1px solid var(--v1-shell-border);
  transition: box-shadow 0.2s ease, transform 0.2s ease;

  &.is-focused {
    box-shadow: 0 20px 48px rgba(0, 0, 0, 0.2), 0 4px 12px rgba(0, 0, 0, 0.1);
    border-color: var(--v1-shell-primary);
  }

  &.is-maximized {
    border-radius: 0;
    border: none;
  }

  &__titlebar {
    height: 44px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 12px;
    background: var(--v1-shell-bg-soft);
    border-bottom: 1px solid var(--v1-shell-border);
    cursor: default;
    user-select: none;
    flex-shrink: 0;
  }

  &__title {
    display: flex;
    align-items: center;
    gap: 10px;
    font-weight: 600;
    font-size: 0.9rem;
    color: var(--v1-shell-text);
  }

  &__icon {
    width: 24px;
    height: 24px;
    background: rgba(255, 255, 255, 0.8);
    border-radius: 6px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.75rem;
    font-weight: 700;
    box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.1);
  }

  &__controls {
    display: flex;
    align-items: center;
    gap: 4px;
  }

  &__control-btn {
    width: 28px;
    height: 28px;
    border: none;
    background: transparent;
    border-radius: 6px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--v1-shell-text-muted);
    cursor: pointer;
    transition: background-color 0.2s, color 0.2s;

    &:hover {
      background: rgba(0, 0, 0, 0.05);
      color: var(--v1-shell-text);
    }

    &--close:hover {
      background: #ef4444;
      color: white;
    }
  }

  &__content {
    flex: 1;
    overflow: auto;
    position: relative;
    background: white;
  }

  &__resize-handle {
    position: absolute;
    z-index: 10;

    &--r {
      top: 0;
      right: 0;
      bottom: 0;
      width: 6px;
      cursor: e-resize;
    }

    &--b {
      left: 0;
      right: 0;
      bottom: 0;
      height: 6px;
      cursor: s-resize;
    }

    &--br {
      right: 0;
      bottom: 0;
      width: 12px;
      height: 12px;
      cursor: se-resize;
    }
  }
}
</style>
