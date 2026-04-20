<template>
  <div
    class="dock">
    <div class="dock__left">
      <xDropdown trigger="click" placement="top-start">
        <button
          class="dock__launcher"
          type="button"
          title="打开应用台">
          <span class="dock__launcher-orb"></span>
          <xIcon icon="grid" size="18" />
        </button>
        <xDropdownMenu slot="dropdown">
          <div class="dock__launcher-menu">
            <xBtn preset="text" class="dock__launcher-menu-btn" @click="openAppById('api')">
              <span class="dock__launcher-menu-btn-content">
                <span class="dock__launcher-menu-icon" :style="{ color: getAppColor('api') }">
                  <xIcon :icon="getAppIcon('api')" size="16" />
                </span>
                <span class="dock__launcher-menu-label">API 管理</span>
              </span>
            </xBtn>
            <xBtn preset="text" class="dock__launcher-menu-btn" @click="openAppById('explore')">
              <span class="dock__launcher-menu-btn-content">
                <span class="dock__launcher-menu-icon" :style="{ color: getAppColor('explore') }">
                  <xIcon :icon="getAppIcon('explore')" size="16" />
                </span>
                <span class="dock__launcher-menu-label">资源管理</span>
              </span>
            </xBtn>
          </div>
        </xDropdownMenu>
      </xDropdown>

      <div class="dock__separator" aria-hidden="true"></div>

      <div class="dock__items">
        <xPopover
          v-for="app in dockApps"
          :key="app.id"
          trigger="hover"
          placement="top"
          :openDelay="120"
          :closeDelay="240"
          :width="320"
          :visibleArrow="false"
          popperClass="dock__preview-popper">
          <div class="dock__preview">
            <div class="dock__preview-header">
              <div class="dock__preview-title">
                <span>{{ app.name }}</span>
                <small>{{ getAppWindows(app.id).length ? getAppWindows(app.id).length + ' 个窗口' : '已固定应用' }}</small>
              </div>
              <button
                class="dock__pin-toggle"
                :title="isPinned(app.id) ? '从 Dock 取消固定' : '固定到 Dock'"
                @click.stop="togglePinned(app.id)">
                <xIcon icon="star" size="14" />
              </button>
            </div>

            <div v-if="getAppWindows(app.id).length" class="dock__preview-list">
              <div
                v-for="win in getAppWindows(app.id)"
                :key="win.id"
                class="dock__preview-item"
                :class="{ 'dock__preview-item--active': system.activeWindowId === win.id && !win.isMinimized }"
                role="button"
                tabindex="0"
                @click.stop="activateWindow(win.id)">
                @keydown.enter.stop.prevent="activateWindow(win.id)"
                @keydown.space.stop.prevent="activateWindow(win.id)">
                <span class="dock__preview-item-meta">
                  <span class="dock__preview-item-title">{{ win.title }}</span>
                  <span class="dock__preview-item-subtitle">
                    {{ win.isMinimized ? '最小化' : (system.activeWindowId === win.id ? '当前窗口' : '已打开') }}
                  </span>
                </span>
                <span class="dock__preview-item-actions">
                  <button
                    class="dock__preview-action"
                    title="最小化"
                    @click.stop="minimizeWindow(win.id)">
                    <xIcon icon="minus" size="14" />
                  </button>
                  <button
                    class="dock__preview-action dock__preview-action--danger"
                    title="关闭"
                    @click.stop="closeWindow(win.id)">
                    <xIcon icon="close" size="14" />
                  </button>
                </span>
              </div>
            </div>

            <button
              v-else
              class="dock__preview-empty"
              @click.stop="system.openApp(app.id)">
              <span class="dock__preview-empty-title">启动 {{ app.name }}</span>
              <span class="dock__preview-empty-subtitle">当前没有活动窗口</span>
            </button>
          </div>

          <div slot="reference" class="dock__item-wrapper">
            <xTooltip effect="dark" placement="top">
              <div slot="content">{{ app.name }}</div>
              <button
                class="dock__item"
                :class="itemClass(app)"
                :title="app.name"
                @click="handleAppClick(app)">
                <span class="dock__item-icon">
                  <xIcon :icon="app.icon || 'grid'" size="22" />
                </span>
                <span
                  v-if="getAppWindows(app.id).length"
                  class="dock__indicator"
                  :class="{ 'dock__indicator--active': isAppActive(app.id) }"></span>
                <span
                  v-if="getAppWindows(app.id).length > 1"
                  class="dock__window-count">
                  {{ getAppWindows(app.id).length }}
                </span>
              </button>
            </xTooltip>
          </div>
        </xPopover>
      </div>
    </div>

    <div class="dock__right">
      <xDropdown trigger="click" placement="top-end">
        <button class="dock__tray" type="button" title="任务栏菜单">
          <span class="dock__tray-time">{{ cptTime }}</span>
          <span class="dock__tray-date">{{ cptDate }}</span>
        </button>
        <xDropdownMenu slot="dropdown">
          <div class="dock__tray-menu">
            <div class="dock__tray-menu-title">{{ cptDate }}</div>
            <div class="dock__tray-menu-time">{{ cptTime }}</div>
            <div class="dock__tray-menu-actions">
              <xBtn preset="text" @click="handleSettingsClick">设置</xBtn>
              <xBtn preset="text" @click="handleLauncherClick">应用台</xBtn>
            </div>
          </div>
        </xDropdownMenu>
      </xDropdown>

      <xTooltip effect="dark" placement="top">
        <div slot="content">设置</div>
        <button
          class="dock__settings"
          title="打开设置"
          @click="handleSettingsClick">
          <xIcon icon="_setting_outlined" size="18" />
        </button>
      </xTooltip>
    </div>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    inject: ['system'],
    data() {
      return {
        nowTs: Date.now(),
        nowTimer: null
      };
    },
    mounted() {
      this.nowTimer = setInterval(() => {
        this.nowTs = Date.now();
      }, 30 * 1000);
    },
    beforeDestroy() {
      if (this.nowTimer) {
        clearInterval(this.nowTimer);
        this.nowTimer = null;
      }
    },
    computed: {
      cptTime() {
        const date = new Date(this.nowTs);
        const hour = this.pad2(date.getHours());
        const minute = this.pad2(date.getMinutes());
        return `${hour}:${minute}`;
      },
      cptDate() {
        const date = new Date(this.nowTs);
        const month = this.pad2(date.getMonth() + 1);
        const day = this.pad2(date.getDate());
        return `${month}/${day}`;
      },
      dockApps() {
        const apps = this.system && this.system.apps ? this.system.apps : [];
        const pinnedApps = this.system && this.system.pinnedApps ? this.system.pinnedApps : [];
        const openWindows = this.system && this.system.openWindows ? this.system.openWindows : [];
        const seen = {};
        const orderedIds = [];

        pinnedApps.forEach(appId => {
          if (!seen[appId]) {
            seen[appId] = true;
            orderedIds.push(appId);
          }
        });

        openWindows.forEach(win => {
          if (!seen[win.appId]) {
            seen[win.appId] = true;
            orderedIds.push(win.appId);
          }
        });

        return orderedIds
          .map(appId => apps.find(app => app.id === appId))
          .filter(app => app && !app.hidden);
      }
    },
    methods: {
      pad2(value) {
        const number = Number(value) || 0;
        return number < 10 ? `0${number}` : `${number}`;
      },
      getAppById(appId) {
        const apps = this.system && this.system.apps ? this.system.apps : [];
        return apps.find(app => app.id === appId) || null;
      },
      getAppIcon(appId) {
        const app = this.getAppById(appId);
        if (app && app.icon) return app.icon;
        return 'grid';
      },
      getAppColor(appId) {
        const app = this.getAppById(appId);
        if (app && app.color) return app.color;
        return 'var(--v1-shell-text-secondary, var(--el-text-color-regular))';
      },
      openAppById(appId) {
        if (!appId) return;
        this.system.openApp(appId);
      },
      itemClass(app) {
        return {
          'dock__item--active': this.isAppActive(app.id),
          'dock__item--running': this.getAppWindows(app.id).length > 0,
          'dock__item--pinned': this.isPinned(app.id)
        };
      },
      getAppWindows(appId) {
        const openWindows = this.system && this.system.openWindows ? this.system.openWindows : [];
        return openWindows
          .filter(win => win.appId === appId)
          .slice()
          .sort((a, b) => b.zIndex - a.zIndex);
      },
      isPinned(appId) {
        const pinnedApps = this.system && this.system.pinnedApps ? this.system.pinnedApps : [];
        return pinnedApps.includes(appId);
      },
      isAppActive(appId) {
        const activeWindowId = this.system ? this.system.activeWindowId : null;
        const activeWindow = this.getAppWindows(appId).find(win => win.id === activeWindowId);
        return !!activeWindow && !activeWindow.isMinimized;
      },
      handleAppClick(app) {
        const windows = this.getAppWindows(app.id);

        if (!windows.length) {
          this.system.openApp(app.id);
          return;
        }

        const win = windows[0];
        if (this.system.activeWindowId === win.id && !win.isMinimized) {
          this.system.minimizeWindow(win.id);
        } else {
          this.activateWindow(win.id);
        }
      },
      activateWindow(windowId) {
        const win = (this.system.openWindows || []).find(item => item.id === windowId);
        if (!win) return;

        win.isMinimized = false;
        this.system.focusWindow(windowId);
      },
      minimizeWindow(windowId) {
        this.system.minimizeWindow(windowId);
      },
      closeWindow(windowId) {
        this.system.closeWindow(windowId);
      },
      togglePinned(appId) {
        if (this.isPinned(appId)) {
          this.system.unpinApp(appId);
        } else {
          this.system.pinApp(appId);
        }
      },
      handleLauncherClick() {
        this.openAppById('explore');
      },
      handleSettingsClick() {
        this.openAppById('setting');
      }
    }
  };
}
</script>

<style lang="less">
.dock {
  position: relative;
  display: flex;
  align-items: center;
  width: 100%;
  height: 56px;
  padding: 0 12px;
  background: rgba(255, 255, 255, 0.78);
  border-top: 1px solid var(--v1-shell-border, var(--el-border-color-lighter));
  backdrop-filter: blur(18px);
  pointer-events: auto;

  &__left {
    display: flex;
    align-items: center;
    gap: 10px;
    min-width: 0;
    flex: 1;
  }

  &__right {
    display: flex;
    align-items: center;
    gap: 10px;
    flex-shrink: 0;
  }

  &__tray {
    border: 0;
    background: transparent;
    padding: 0 10px;
    height: 44px;
    border-radius: 12px;
    cursor: pointer;
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    justify-content: center;
    gap: 2px;
    color: var(--v1-shell-text-secondary, var(--el-text-color-regular));
    transition: background-color 160ms ease;

    &:hover {
      background: rgba(49, 130, 206, 0.08);
    }
  }

  &__tray-time {
    font-size: 12px;
    font-weight: 700;
    line-height: 1;
    color: var(--v1-shell-text, var(--el-text-color-primary));
  }

  &__tray-date {
    font-size: 11px;
    font-weight: 600;
    line-height: 1;
    color: var(--v1-shell-text-muted, var(--el-text-color-secondary));
  }

  &__tray-menu {
    min-width: 220px;
    padding: 10px 12px;
    background: rgba(255, 255, 255, 0.94);
  }

  &__tray-menu-title {
    font-size: 12px;
    font-weight: 600;
    color: var(--v1-shell-text-muted, var(--el-text-color-secondary));
    margin-bottom: 2px;
  }

  &__tray-menu-time {
    font-size: 18px;
    font-weight: 800;
    color: var(--v1-shell-text, var(--el-text-color-primary));
    margin-bottom: 8px;
  }

  &__tray-menu-actions {
    display: flex;
    gap: 8px;
    justify-content: flex-end;
    border-top: 1px solid var(--v1-shell-border, var(--el-border-color-lighter));
    padding-top: 8px;
  }

  &__items {
    display: flex;
    align-items: center;
    gap: 4px;
    min-width: 0;
  }

  &__item-wrapper {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  &__item,
  &__launcher,
  &__settings {
    position: relative;
    width: 44px;
    height: 44px;
    padding: 0;
    border: 0;
    border-radius: 14px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    background: transparent;
    color: var(--v1-shell-text-secondary, var(--el-text-color-regular));
    cursor: pointer;
    transition:
      transform 180ms cubic-bezier(0.4, 0, 0.2, 1),
      background-color 180ms ease,
      box-shadow 180ms ease;

    &:hover {
      transform: translateY(-1px);
      background: var(--v1-shell-primary-soft, var(--el-color-primary-light-9));
    }

    &:active {
      transform: scale(0.98);
      background: rgba(49, 130, 206, 0.12);
    }
  }

  &__item {
    &--running {
      background: rgba(49, 130, 206, 0.08);
    }

    &--active {
      background: var(--v1-shell-primary, var(--el-color-primary));
      color: #ffffff;
      box-shadow:
        inset 0 1px 0 rgba(255, 255, 255, 0.16),
        0 10px 22px rgba(49, 130, 206, 0.26);
    }

    &--pinned::after {
      content: '';
      position: absolute;
      inset: 1px;
      border-radius: 13px;
      border: 1px solid var(--v1-shell-border, var(--el-border-color-lighter));
      pointer-events: none;
    }
  }

  &__item-icon {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
    color: inherit;
  }

  &__launcher {
    overflow: hidden;
    background: rgba(49, 130, 206, 0.08);
  }

  &__launcher-menu {
    min-width: 180px;
    padding: 8px 10px;
    display: flex;
    flex-direction: column;
    gap: 4px;
    background: rgba(255, 255, 255, 0.94);
  }

  &__launcher-menu-btn {
    width: 100%;
    justify-content: flex-start;
  }

  &__launcher-menu-btn-content {
    display: flex;
    align-items: center;
    gap: 10px;
    width: 100%;
  }

  &__launcher-menu-icon {
    width: 28px;
    height: 28px;
    border-radius: 10px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    background: rgba(49, 130, 206, 0.08);
  }

  &__launcher-menu-label {
    font-size: 14px;
    font-weight: 600;
    color: var(--v1-shell-text, var(--el-text-color-primary));
  }

  &__launcher-orb {
    position: absolute;
    inset: auto auto 8px 8px;
    width: 14px;
    height: 14px;
    border-radius: 999px;
    background: linear-gradient(135deg, var(--v1-shell-primary, var(--el-color-primary)) 0%, var(--el-color-primary-light-5, #93c5fd) 100%);
    filter: blur(1px);
    opacity: 0.78;
  }

  &__settings {
    background: transparent;
  }

  &__indicator {
    position: absolute;
    left: 50%;
    bottom: -4px;
    width: 4px;
    height: 4px;
    border-radius: 999px;
    background: rgba(96, 98, 102, 0.38);
    transform: translateX(-50%);
    transition: all 180ms ease;

    &--active {
      width: 16px;
      background: var(--v1-shell-primary, var(--el-color-primary));
      box-shadow: 0 0 10px rgba(49, 130, 206, 0.3);
    }
  }

  &__window-count {
    position: absolute;
    top: -4px;
    right: -2px;
    min-width: 18px;
    height: 18px;
    padding: 0 5px;
    border-radius: 999px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    background: var(--v1-shell-primary, var(--el-color-primary));
    color: #ffffff;
    font-size: 10px;
    font-weight: 700;
    line-height: 1;
    box-shadow: 0 6px 12px rgba(49, 130, 206, 0.2);
  }

  &__separator {
    width: 1px;
    height: 28px;
    align-self: center;
    background: linear-gradient(180deg, rgba(220, 223, 230, 0) 0%, rgba(220, 223, 230, 1) 50%, rgba(220, 223, 230, 0) 100%);
  }

  &__preview {
    width: 100%;
    padding: 10px;
  }

  &__preview-popper {
    padding: 0;
    border-radius: 18px;
    border: 1px solid var(--v1-shell-border, var(--el-border-color-lighter));
    background: rgba(255, 255, 255, 0.94);
    box-shadow: var(--v1-shell-shadow, var(--el-box-shadow));
    backdrop-filter: blur(20px);
  }

  &__preview-header {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 12px;
    margin-bottom: 6px;
    padding: 4px 8px 8px;
    border-bottom: 1px solid var(--v1-shell-border, var(--el-border-color-lighter));
  }

  &__preview-title {
    display: flex;
    flex-direction: column;
    gap: 3px;
    min-width: 0;
    color: var(--v1-shell-text, var(--el-text-color-primary));
    font-size: 12px;
    font-weight: 500;

    small {
      color: var(--v1-shell-text-muted, var(--el-text-color-secondary));
      font-size: 11px;
      font-weight: 500;
    }
  }

  &__pin-toggle,
  &__preview-action {
    width: 28px;
    height: 28px;
    padding: 0;
    border: 0;
    border-radius: 999px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    background: transparent;
    color: var(--v1-shell-text-secondary, var(--el-text-color-regular));
    cursor: pointer;
    transition: background-color 160ms ease, transform 160ms ease;

    &:hover {
      background: var(--v1-shell-primary-soft, var(--el-color-primary-light-9));
      transform: translateY(-1px);
    }
  }

  &__preview-list {
    display: flex;
    flex-direction: column;
    gap: 4px;
    max-height: 250px;
    overflow-y: auto;
    padding-right: 4px;

    &::-webkit-scrollbar {
      width: 3px;
    }

    &::-webkit-scrollbar-track {
      background: transparent;
    }

    &::-webkit-scrollbar-thumb {
      background: rgba(144, 147, 153, 0.3);
      border-radius: 6px;
    }
  }

  &__preview-item,
  &__preview-empty {
    width: 100%;
    border: 1px solid transparent;
    border-radius: 10px;
    background: transparent;
    color: var(--v1-shell-text, var(--el-text-color-primary));
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
    padding: 6px;
    cursor: pointer;
    transition: background-color 160ms ease, border-color 160ms ease, transform 160ms ease;

    &:hover {
      background: rgba(49, 130, 206, 0.06);
      transform: translateY(-1px);
    }
  }

  &__preview-item {
    &--active {
      background: var(--v1-shell-primary-soft, var(--el-color-primary-light-9));
      border-color: rgba(49, 130, 206, 0.12);
    }
  }

  &__preview-item-meta {
    min-width: 0;
    display: flex;
    flex-direction: column;
    gap: 3px;
    text-align: left;
  }

  &__preview-item-title,
  &__preview-empty-title {
    font-size: 14px;
    font-weight: 500;
    color: var(--v1-shell-text, var(--el-text-color-primary));
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  &__preview-item-subtitle,
  &__preview-empty-subtitle {
    font-size: 12px;
    color: var(--v1-shell-text-muted, var(--el-text-color-secondary));
  }

  &__preview-item-actions {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    flex-shrink: 0;
  }

  &__preview-action--danger:hover {
    background: rgba(245, 108, 108, 0.12);
    color: var(--el-color-danger, #f56c6c);
  }

  &__preview-empty {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>
