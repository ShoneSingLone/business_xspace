<template>
  <div 
    v-show="!window.isMinimized"
    class="window-container" 
    :class="{ 
      'is-focused': window.isFocused,
      'is-maximized': window.isMaximized 
    }"
    :style="cptWindowStyle"
    @mousedown="handleFocus">
    
    <!-- Title Bar (Drag Handle) -->
    <div 
      class="window-container__titlebar" 
      @mousedown="startDrag"
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
          <xIcon :icon="window.isMaximized ? 'copy-document' : 'full-screen'" size="16" />
        </button>
        <button class="window-container__control-btn window-container__control-btn--close" @click="handleClose" title="Close">
          <xIcon icon="close" size="18" />
        </button>
      </div>
    </div>

    <!-- Content Area -->
    <div class="window-container__content">
      <component 
        :is="window.component" 
        v-bind="window.props" 
        :window-data="window.data"
        @close="handleClose" />
    </div>

    <!-- Resize Handles (Only if not maximized) -->
    <template v-if="!window.isMaximized">
      <div class="window-container__resize-handle window-container__resize-handle--r" @mousedown.stop="startResize($event, 'r')"></div>
      <div class="window-container__resize-handle window-container__resize-handle--b" @mousedown.stop="startResize($event, 'b')"></div>
      <div class="window-container__resize-handle window-container__resize-handle--br" @mousedown.stop="startResize($event, 'br')"></div>
    </template>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    props: {
      window: { type: Object, required: true },
      windowManager: { type: Object, required: true },
      system: { type: Object, required: true }
    },
    computed: {
      cptWindowStyle() {
        if (this.window.isMaximized) {
          return {
            zIndex: this.window.zIndex,
            top: '0',
            left: '0',
            width: '100%',
            height: '100%',
            borderRadius: '0'
          };
        }
        return {
          zIndex: this.window.zIndex,
          top: `${this.window.y}px`,
          left: `${this.window.x}px`,
          width: `${this.window.width}px`,
          height: `${this.window.height}px`
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
        this.windowManager.focus(this.window.id);
      },
      handleClose() {
        this.windowManager.close(this.window.id);
      },
      handleMinimize() {
        this.windowManager.minimize(this.window.id);
      },
      handleToggleMaximize() {
        this.windowManager.toggleMaximize(this.window.id);
      },
      
      // Drag & Resize Logic
      startDrag(e) {
        if (this.window.isMaximized) return;
        this.handleFocus();
        
        const startX = e.clientX;
        const startY = e.clientY;
        const initialX = this.window.x;
        const initialY = this.window.y;

        const onMouseMove = (moveEvent) => {
          const deltaX = moveEvent.clientX - startX;
          const deltaY = moveEvent.clientY - startY;
          
          let nextX = initialX + deltaX;
          let nextY = initialY + deltaY;

          // Boundary checks (ensure titlebar is accessible)
          const winWidth = window.innerWidth;
          const winHeight = window.innerHeight;
          nextX = Math.max(-this.window.width + 100, Math.min(nextX, winWidth - 100));
          nextY = Math.max(0, Math.min(nextY, winHeight - 40));

          this.windowManager.updateRect(this.window.id, { x: nextX, y: nextY });
        };

        const onMouseUp = () => {
          window.removeEventListener('mousemove', onMouseMove);
          window.removeEventListener('mouseup', onMouseUp);
          document.body.style.userSelect = '';
        };

        document.body.style.userSelect = 'none';
        window.addEventListener('mousemove', onMouseMove);
        window.addEventListener('mouseup', onMouseUp);
      },

      startResize(e, direction) {
        this.handleFocus();
        
        const startX = e.clientX;
        const startY = e.clientY;
        const initialWidth = this.window.width;
        const initialHeight = this.window.height;

        const onMouseMove = (moveEvent) => {
          const deltaX = moveEvent.clientX - startX;
          const deltaY = moveEvent.clientY - startY;
          
          const updates = {};
          if (direction.includes('r')) {
            updates.width = initialWidth + deltaX;
          }
          if (direction.includes('b')) {
            updates.height = initialHeight + deltaY;
          }

          this.windowManager.updateRect(this.window.id, updates);
        };

        const onMouseUp = () => {
          window.removeEventListener('mousemove', onMouseMove);
          window.removeEventListener('mouseup', onMouseUp);
          document.body.style.userSelect = '';
        };

        document.body.style.userSelect = 'none';
        window.addEventListener('mousemove', onMouseMove);
        window.addEventListener('mouseup', onMouseUp);
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
