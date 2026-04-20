<template>
  <Transition name="hero">
    <div
      v-show="!window.isMinimized"
      ref="windowRef"
      class="v1-window"
      :class="windowClassList"
      :style="windowStyle"
      @mousedown="system.focusWindow(window.id)">
      <div ref="handleRef" class="v1-window__titlebar">
        <div class="v1-window__title">
          <div class="v1-window__app-icon" :style="{ color: appColor }">
            <div class="v1-window__app-letter">{{ appIcon }}</div>
          </div>
          <div class="v1-window__title-copy">
            <span class="v1-window__title-text">{{ window.title }}</span>
            <span class="v1-window__title-meta">{{ windowSubtitle }}</span>
          </div>
        </div>

        <div class="v1-window__controls">
          <button
            type="button"
            class="v1-window__control-btn"
            title="Pin to Desktop"
            @click.stop="pinToDesktop">
            <xIcon icon="star" size="20" />
          </button>
          <button
            type="button"
            class="v1-window__control-btn"
            title="Minimize"
            @click.stop="handleMinimize">
            <xIcon icon="minus" size="20" />
          </button>
          <button
            type="button"
            class="v1-window__control-btn"
            title="Maximize"
            @click.stop="handleMaximize">
            <xIcon :icon="window.isMaximized ? 'copy-document' : 'scale-to-original'" size="16" />
          </button>
          <button
            type="button"
            class="v1-window__control-btn v1-window__control-btn--close"
            title="Close"
            @click.stop="handleClose">
            <xIcon icon="close" size="20" />
          </button>
        </div>
      </div>

      <div class="v1-window__content">
        <div v-if="isApiManagerApp" class="v1-window__stage">
          <ApiManager :window-data="window.data" />
        </div>
        <div v-else-if="isExploreApp" class="v1-window__stage">
          <Explore />
        </div>
        <div v-else class="v1-window__placeholder">
          <div class="v1-window__placeholder-icon" :style="{ color: appColor }">
            <div class="v1-window__placeholder-letter">{{ appIcon }}</div>
          </div>
          <p class="v1-window__placeholder-text">
            The <strong>{{ window.title }}</strong> module is currently under development.
          </p>
          <button type="button" class="v1-window__placeholder-btn">Request Feature</button>
        </div>
      </div>
    </div>
  </Transition>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    components: {
      ApiManager: () => _.$importVue('./modules/ApiManager.vue'),
      Explore: () => _.$importVue('./modules/Explore.vue')
    },
    props: {
      window: Object
    },
    // system: v1 Desktop Workspace 局部状态（来自 ViewXspace）
    inject: ['system'],
    data() {
      return {
        windowRef: null,
        handleRef: null,
        x: this.window.x,
        y: this.window.y,
        isDragging: false
      };
    },
    computed: {
      isActive() {
        return this.system.activeWindowId === this.window.id;
      },
      appInfo() {
        return this.system.apps.find(a => a.id === this.window.appId) || {};
      },
      appIcon() {
        const name = this.appInfo.name || this.window.title || '?';
        return name.charAt(0) || '?';
      },
      appColor() {
        return this.appInfo.color || 'var(--color-primary)';
      },
      windowSubtitle() {
        if (this.isApiManagerApp) {
          return 'Project workspace';
        }
        if (this.isExploreApp) {
          return 'Explore content';
        }
        return 'Feature preview';
      },
      isApiManagerApp() {
        return [
          'api',
          'group',
          'project',
          'api_folder',
          'doc_folder',
          'folder',
          'api_endpoint',
          'doc',
          'code',
          'member_list',
          'setting',
          'cicd'
        ].includes(this.window.appId);
      },
      isExploreApp() {
        return this.window.appId === 'explore';
      },
      windowClassList() {
        return {
          'v1-window--maximized': this.window.isMaximized,
          'v1-window--active': this.isActive,
          'v1-window--inactive': !this.isActive,
          'v1-window--animating': !this.isDragging
        };
      },
      windowStyle() {
        if (this.window.isMaximized) {
          return {
            zIndex: this.window.zIndex
          };
        }

        return {
          zIndex: this.window.zIndex,
          width: this.window.width + 'px',
          height: this.window.height + 'px',
          transform: `translate(${this.x}px, ${this.y}px)`
        };
      }
    },
    methods: {
      handleClose() {
        this.system.closeWindow(this.window.id);
      },
      handleMinimize() {
        this.system.minimizeWindow(this.window.id);
      },
      handleMaximize() {
        this.system.toggleMaximize(this.window.id);
      },
      pinToDesktop() {
        const app = this.system.apps.find(a => a.id === this.window.appId);
        if (!app) return;
        this.system.addShortcut({
          id: this.window.data ? `shortcut-${this.window.data.id}` : `shortcut-${app.id}`,
          appId: app.id,
          name: this.window.title,
          icon: app.icon,
          color: app.color,
          data: this.window.data
        });
      },
      focusWindow() {
        this.system.focusWindow(this.window.id);
      }
    },
    mounted() {
      // 简单的拖拽实现
      let isDragging = false;
      let startX = 0;
      let startY = 0;
      let startWindowX = 0;
      let startWindowY = 0;

      const handle = this.handleRef;
      if (handle) {
        handle.addEventListener("mousedown", e => {
          if (this.window.isMaximized) return;

          isDragging = true;
          startX = e.clientX;
          startY = e.clientY;
          startWindowX = this.x;
          startWindowY = this.y;
          this.system.focusWindow(this.window.id);
          this.isDragging = true;
        });

        document.addEventListener("mousemove", e => {
          if (!isDragging || this.window.isMaximized) return;

          const deltaX = e.clientX - startX;
          const deltaY = e.clientY - startY;

          this.x = startWindowX + deltaX;
          this.y = startWindowY + deltaY;
          this.window.x = this.x;
          this.window.y = this.y;
        });

        document.addEventListener("mouseup", () => {
          isDragging = false;
          this.isDragging = false;
        });
      }
    }
  };
}
</script>

<style lang="less">
.v1-window {
  position: absolute;
  inset: auto;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  pointer-events: auto;
  background: var(--color-surface);
  border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
  border-radius: 24px;
  box-shadow: var(--el-box-shadow-dark);

  &--active {
    box-shadow: 0 20px 48px rgba(15, 23, 42, 0.18), 0 10px 24px rgba(15, 23, 42, 0.12);
  }

  &--inactive {
    box-shadow: 0 14px 32px rgba(15, 23, 42, 0.1), 0 6px 16px rgba(15, 23, 42, 0.08);
  }

  &--animating {
    transition: width 0.3s ease-out, height 0.3s ease-out, transform 0.3s ease-out,
      opacity 0.3s ease-out;
  }

  &--maximized {
    inset: 0 !important;
    width: 100% !important;
    height: 100% !important;
    transform: translate(0, 0) !important;
    border-radius: 0;
    border-width: 0;
  }

  &__titlebar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 16px;
    min-height: 64px;
    padding: 0 20px 0 22px;
    background: color-mix(in srgb, var(--color-surface-container) 92%, white 8%);
    border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 72%, transparent);
    backdrop-filter: blur(14px);
    cursor: default;
    user-select: none;
  }

  &__title {
    display: flex;
    align-items: center;
    gap: 14px;
    min-width: 0;
  }

  &__app-icon,
  &__placeholder-icon {
    display: flex;
    align-items: center;
    justify-content: center;
    background: color-mix(in srgb, var(--color-surface-container-highest) 90%, white 10%);
    box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.7);
  }

  &__app-icon {
    width: 40px;
    height: 40px;
    border-radius: 14px;
  }

  &__app-letter,
  &__placeholder-letter {
    font-weight: 700;
    line-height: 1;
  }

  &__app-letter {
    font-size: 18px;
  }

  &__title-copy {
    display: flex;
    flex-direction: column;
    gap: 2px;
    min-width: 0;
  }

  &__title-text {
    overflow: hidden;
    color: var(--color-on-surface);
    font-size: 15px;
    font-weight: 600;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  &__title-meta {
    color: var(--color-on-surface-variant);
    font-size: 12px;
    font-weight: 500;
  }

  &__controls {
    display: flex;
    align-items: center;
    gap: 6px;
    flex-shrink: 0;
  }

  &__control-btn {
    width: 36px;
    height: 36px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    border: 0;
    border-radius: 999px;
    background: transparent;
    color: var(--color-on-surface-variant);
    cursor: pointer;
    transition: background-color 0.2s ease, color 0.2s ease, transform 0.2s ease;

    &:hover {
      background: rgba(0, 0, 0, 0.06);
    }

    &:active {
      background: rgba(0, 0, 0, 0.1);
      transform: scale(0.96);
    }

    &--close:hover {
      background: color-mix(in srgb, var(--el-color-danger-light-9) 92%, white 8%);
      color: var(--el-color-danger);
    }

    &--close:active {
      background: color-mix(in srgb, var(--el-color-danger-light-8) 92%, white 8%);
    }
  }

  &__content {
    flex: 1;
    min-height: 0;
    overflow: auto;
    background: var(--color-surface);
  }

  &__stage {
    height: 100%;
  }

  &__placeholder {
    height: 100%;
    min-height: 320px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 48px 24px;
    text-align: center;
  }

  &__placeholder-icon {
    width: 88px;
    height: 88px;
    border-radius: 28px;
    margin-bottom: 24px;
  }

  &__placeholder-letter {
    font-size: 30px;
  }

  &__placeholder-text {
    max-width: 480px;
    margin: 0 0 24px;
    color: var(--color-on-surface-variant);
    font-size: 16px;
    line-height: 1.7;
  }

  &__placeholder-btn {
    min-width: 148px;
    min-height: 44px;
    padding: 0 20px;
    border: 0;
    border-radius: 999px;
    background: var(--el-color-primary);
    color: var(--color-on-primary);
    cursor: pointer;
    font-size: 14px;
    font-weight: 600;
    transition: background-color 0.2s ease, box-shadow 0.2s ease, transform 0.2s ease;

    &:hover {
      background: var(--el-color-primary-hover);
      box-shadow: var(--el-box-shadow-light);
    }

    &:active {
      background: var(--el-color-primary-active);
      transform: scale(0.98);
    }
  }
}

/* 窗口过渡动画 */
.hero-enter-active,
.hero-leave-active {
  transition: all 0.3s ease;
}

.hero-enter-from,
.hero-leave-to {
  opacity: 0;
  transform: scale(0.95);
}
</style>
