<template>
  <AuthScreen v-if="!isAuthenticated" />
  <div v-else class="v1-workspace">
    <!-- Top Bar -->
    <TopBar class="v1-workspace__topbar" />

    <!-- Desktop Area -->
    <div class="v1-workspace__desktop" @dragover="onDragOver" @drop="onDrop">
      <div class="v1-workspace__backdrop" aria-hidden="true">
        <div class="v1-workspace__glow v1-workspace__glow--left"></div>
        <div class="v1-workspace__glow v1-workspace__glow--right"></div>
        <div class="v1-workspace__glow v1-workspace__glow--bottom"></div>
      </div>

      <!-- Desktop Icons Grid -->
      <div class="v1-workspace__icons">
        <DesktopIcon 
          v-for="shortcut in shortcuts" 
          :key="shortcut.id" 
          :app="shortcut" 
          @click="openApp(shortcut.appId, false, shortcut.data)"
          @unpin="removeShortcut(shortcut.id)"
        />
      </div>

      <!-- Windows Layer -->
      <WindowContainer 
        v-for="win in windowManager.state.windows" 
        :key="win.id"
        :window="win"
        :window-manager="windowManager"
        :system="systemForWindows"
      />

      <!-- Bottom Dock -->
      <div class="v1-workspace__dock-wrapper">
        <Dock />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  const windowManager = await _.$importVue('@/utils/windowManager.vue');

  return {
    inject: ['APP'],
    components: {
      AuthScreen: () => _.$importVue('@/views/v1/components/AuthScreen.vue'),
      TopBar: () => _.$importVue('@/views/v1/components/TopBar.vue'),
      DesktopIcon: () => _.$importVue('@/views/v1/components/DesktopIcon.vue'),
      Dock: () => _.$importVue('@/views/v1/components/Dock.vue'),
      WindowContainer: () => _.$importVue('@/views/v1/components/WindowContainer.vue')
    },
    provide() {
      return {
        windowManager: this.windowManager,
        system: {
          apps: this.apps,
          shortcuts: this.shortcuts,
          openWindows: this.windowManager.state.windows,
          activeWindowId: this.windowManager.state.activeId,
          pinnedApps: this.pinnedApps,
          pinApp: this.pinApp,
          unpinApp: this.unpinApp,
          addShortcut: this.addShortcut,
          removeShortcut: this.removeShortcut,
          openApp: this.openApp,
          focusWindow: (id) => this.windowManager.focus(id),
          closeWindow: (id) => this.windowManager.close(id),
          minimizeWindow: (id) => this.windowManager.minimize(id),
          toggleMaximize: (id) => this.windowManager.toggleMaximize(id)
        }
      };
    },
    data() {
      return {
        windowManager,
        apps: [
          { id: 'api', name: 'API Manager', icon: '_database', color: '#6750A4', component: '@/views/Api/Project/Project.vue' },
          { id: 'cicd', name: 'CI/CD', icon: '_ci', color: '#625B71', component: '@/views/CiCd/CiCd.vue' },
          { id: 'note', name: 'Documents', icon: '_article', color: '#7D5260', component: '@/views/Note/Note.vue' },
          { id: 'im', name: 'Chat', icon: '_contact', color: '#006A6A', component: '@/views/Im/Im.vue' },
          { id: 'rtc', name: 'Meeting', icon: '_webrtc', color: '#B3261E', component: '@/views/Rtc/Rtc.vue' },
          { id: 'office', name: 'Cloud Storage', icon: '_cloud_o', color: '#0061A4', component: '@/views/CloudDisk/CloudDisk.vue' },
          { id: 'explore', name: 'Explore', icon: 'search', color: '#984061', component: '@/views/Explore/Explore.vue' },
          { id: 'user', name: 'User', icon: 'user', color: '#006874', component: '@/views/User/User.vue' },
        ],
        shortcuts: [
          { id: 'api', appId: 'api', name: 'API Manager', icon: '_database', color: '#6750A4' },
          { id: 'explore', appId: 'explore', name: 'Explore', icon: 'search', color: '#984061' },
          { id: 'note', appId: 'note', name: 'Documents', icon: '_article', color: '#7D5260' }
        ],
        theme: 'light',
        lastOpenedAppId: null,
        pinnedApps: ['api', 'explore']
      };
    },
    computed: {
      isAuthenticated() {
        return this.APP?.user?.isLogin;
      },
      systemForWindows() {
        return {
          apps: this.apps
        };
      }
    },
    methods: {
      onDragOver(e) {
        e.preventDefault();
        if (e.dataTransfer) {
          e.dataTransfer.dropEffect = 'move';
        }
      },
      onDrop(e) {
        e.preventDefault();
        const action = e.dataTransfer?.getData('action');
        const appId = e.dataTransfer?.getData('text/plain');
        
        if (action === 'unpin' && appId) {
          this.unpinApp(appId);
        }
      },
      pinApp(appId) {
        if (!this.pinnedApps.includes(appId)) {
          this.pinnedApps.push(appId);
        }
      },
      unpinApp(appId) {
        this.pinnedApps = this.pinnedApps.filter(id => id !== appId);
      },
      addShortcut(shortcut) {
        if (!this.shortcuts.find(s => s.id === shortcut.id)) {
          this.shortcuts.push(shortcut);
        }
      },
      removeShortcut(id) {
        const index = this.shortcuts.findIndex(s => s.id === id);
        if (index > -1) {
          this.shortcuts.splice(index, 1);
        }
      },
      async openApp(appId, forceNew = false, data) {
        const app = this.apps.find(a => a.id === appId);
        if (!app) return;

        // 加载组件
        const component = () => _.$importVue(app.component);

        this.windowManager.open({
          appId,
          title: data?.name || app.name,
          component,
          data,
          singleton: !forceNew
        });
      }
    }
  };
}
</script>

<style lang="less">
.v1-workspace {
  --v1-shell-bg: var(--body-bg-color, #f4f9fd);
  --v1-shell-bg-soft: var(--bg-color, #f5f8f7);
  --v1-shell-surface: var(--el-fill-color-blank, #ffffff);
  --v1-shell-surface-muted: var(--dialog-bg-color, #ffffff);
  --v1-shell-border: var(--el-border-color-lighter, #ebeef5);
  --v1-shell-border-strong: var(--el-border-color, #dcdfe6);
  --v1-shell-text: var(--el-text-color-primary, #303133);
  --v1-shell-text-secondary: var(--el-text-color-regular, #606266);
  --v1-shell-text-muted: var(--el-text-color-secondary, #909399);
  --v1-shell-primary: var(--el-color-primary, #3182ce);
  --v1-shell-primary-soft: var(--el-color-primary-light-9, #eff6ff);
  --v1-shell-shadow: var(--el-box-shadow);
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background:
    radial-gradient(circle at 10% 6%, color-mix(in srgb, var(--v1-shell-primary) 14%, transparent) 0%, transparent 34%),
    radial-gradient(circle at 84% 0%, color-mix(in srgb, var(--el-color-primary-hover, var(--v1-shell-primary)) 12%, transparent) 0%, transparent 28%),
    linear-gradient(180deg, rgba(255, 255, 255, 0.58) 0%, rgba(255, 255, 255, 0.18) 100%),
    var(--v1-shell-bg);
  color: var(--v1-shell-text);
  user-select: none;
  display: flex;
  flex-direction: column;
  position: relative;

  &__topbar {
    z-index: 100;
  }

  &__desktop {
    flex: 1;
    position: relative;
    overflow: hidden;
    background:
      linear-gradient(180deg, rgba(255, 255, 255, 0.26) 0%, rgba(255, 255, 255, 0.52) 100%),
      linear-gradient(90deg, color-mix(in srgb, var(--v1-shell-primary) 3%, transparent) 1px, transparent 1px),
      linear-gradient(color-mix(in srgb, var(--v1-shell-primary) 3%, transparent) 1px, transparent 1px),
      radial-gradient(circle at 16% 12%, color-mix(in srgb, var(--v1-shell-primary) 12%, transparent) 0%, transparent 32%),
      radial-gradient(circle at 82% 16%, color-mix(in srgb, var(--el-color-primary-hover, var(--v1-shell-primary)) 12%, transparent) 0%, transparent 24%),
      var(--v1-shell-bg-soft);
    background-size:
      auto,
      64px 64px,
      64px 64px,
      auto,
      auto,
      auto;
    background-position:
      0 0,
      0 0,
      0 0,
      0 0,
      0 0,
      0 0;
  }

  &__backdrop {
    position: absolute;
    inset: 0;
    z-index: 0;
    pointer-events: none;
    overflow: hidden;
  }

  &__glow {
    position: absolute;
    border-radius: 999px;
    filter: blur(32px);
    opacity: 0.72;
    transform: translateZ(0);

    &--left {
      top: -120px;
      left: -80px;
      width: 420px;
      height: 420px;
      background: radial-gradient(circle, color-mix(in srgb, var(--v1-shell-primary) 22%, transparent) 0%, color-mix(in srgb, var(--v1-shell-primary) 8%, transparent) 38%, transparent 72%);
    }

    &--right {
      top: 48px;
      right: -140px;
      width: 420px;
      height: 420px;
      background: radial-gradient(circle, rgba(99, 179, 237, 0.22) 0%, rgba(99, 179, 237, 0.08) 38%, rgba(99, 179, 237, 0) 72%);
    }

    &--bottom {
      left: 50%;
      bottom: -240px;
      width: 760px;
      height: 420px;
      background: radial-gradient(circle, color-mix(in srgb, var(--v1-shell-primary) 18%, transparent) 0%, color-mix(in srgb, var(--v1-shell-primary) 5%, transparent) 32%, transparent 72%);
      transform: translateX(-50%);
      filter: blur(40px);
      opacity: 0.56;
    }
  }

  &__bg-logo {
    display: none;
  }

  &__icons {
    position: absolute;
    top: 0;
    left: 0;
    bottom: 0;
    width: min(520px, 100%);
    z-index: 10;
    padding:
      28px
      24px
      calc(112px + env(safe-area-inset-bottom, 0px))
      calc(28px + env(safe-area-inset-left, 0px));
    display: grid;
    grid-template-columns: repeat(auto-fill, 92px);
    grid-auto-rows: 112px;
    gap: 18px 12px;
    align-content: start;
    pointer-events: auto;
  }

  &__dock-wrapper {
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 50;
    display: flex;
    justify-content: stretch;
    align-items: stretch;
    width: 100%;
    height: calc(56px + env(safe-area-inset-bottom, 0px));
    padding-bottom: env(safe-area-inset-bottom, 0px);
    background: rgba(255, 255, 255, 0.6);
    pointer-events: none;
    backdrop-filter: blur(18px);

    & > * {
      pointer-events: auto;
      width: 100%;
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
