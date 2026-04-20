<template>
  <div class="top-bar">
    <div class="top-bar__left">
      <div class="top-bar__brand">
        <span class="top-bar__brand-dot"></span>
        <span class="top-bar__title">xspace</span>
      </div>
    </div>

    <div class="top-bar__center">
      <div class="top-bar__search">
        <xIcon icon="search" size="12" class="top-bar__search-icon" />
        <input
          v-model="searchKeyword"
          type="text"
          placeholder="搜索工作台"
          class="top-bar__search-input"
        />
      </div>
    </div>

    <div class="top-bar__right">
      <div class="top-bar__user" :title="userDisplayName">
        <span class="top-bar__user-avatar">{{ userInitial }}</span>
        <span class="top-bar__user-name">{{ userDisplayName }}</span>
      </div>
      <div class="top-bar__datetime">
        <span class="top-bar__date">{{ dateLabel }}</span>
        <span class="top-bar__time">{{ time }}</span>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    // APP: 全局状态（来自 entry.vue）
    // system: v1 Desktop Workspace 局部状态（来自 ViewXspace）
    inject: ['APP', 'system'],
    data() {
      return {
        searchKeyword: '',
        dateLabel: '',
        time: '',
        timer: null
      };
    },
    computed: {
      userDisplayName() {
        const user = this.APP && this.APP.user ? this.APP.user : {};
        return user.username || user.name || user.email || 'Guest';
      },
      userInitial() {
        return String(this.userDisplayName).trim().charAt(0).toUpperCase() || 'G';
      }
    },
    methods: {
      updateTime() {
        const now = new Date();
        this.time = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
        this.dateLabel = now.toLocaleDateString([], {
          month: '2-digit',
          day: '2-digit'
        });
      }
    },
    mounted() {
      this.updateTime();
      this.timer = setInterval(this.updateTime, 1000);
    },
    beforeDestroy() {
      clearInterval(this.timer);
    }
  };
}
</script>

<style lang="less">
.top-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  height: 42px;
  padding: 0 18px;
  font-size: 0.6875rem;
  font-weight: 500;
  color: var(--v1-shell-text-secondary, var(--el-text-color-regular));
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.92) 0%, rgba(255, 255, 255, 0.72) 100%);
  border-bottom: 1px solid var(--v1-shell-border, var(--el-border-color-lighter));
  box-shadow: 0 1px 0 rgba(255, 255, 255, 0.5);
  backdrop-filter: blur(12px);
  user-select: none;

  &__left {
    display: flex;
    align-items: center;
    min-width: 0;
    flex: 1;
  }

  &__center {
    display: flex;
    justify-content: center;
    flex: 0 1 220px;
    min-width: 120px;
  }

  &__right {
    display: flex;
    align-items: center;
    justify-content: flex-end;
    gap: 12px;
    flex: 1;
    min-width: 0;
  }

  &__brand {
    display: flex;
    align-items: center;
    gap: 8px;
    min-width: 0;
  }

  &__brand-dot {
    width: 8px;
    height: 8px;
    border-radius: 999px;
    background: var(--v1-shell-primary, var(--el-color-primary));
    box-shadow: 0 0 0 4px var(--v1-shell-primary-soft, var(--el-color-primary-light-9));
    flex-shrink: 0;
  }

  &__title {
    font-weight: 600;
    letter-spacing: 0.04em;
    white-space: nowrap;
    color: var(--v1-shell-text, var(--el-text-color-primary));
  }

  &__search {
    display: flex;
    align-items: center;
    gap: 6px;
    width: 100%;
    max-width: 220px;
    padding: 6px 12px;
    background: rgba(255, 255, 255, 0.78);
    border: 1px solid var(--v1-shell-border, var(--el-border-color-lighter));
    border-radius: 9999px;
    color: var(--v1-shell-text-muted, var(--el-text-color-secondary));
    opacity: 1;
    transition: opacity 0.2s ease, background-color 0.2s ease, border-color 0.2s ease;
    box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.85);

    &:focus-within {
      background: var(--v1-shell-surface, var(--el-fill-color-blank));
      border-color: var(--v1-shell-primary, var(--el-color-primary));
    }

    &-icon {
      flex-shrink: 0;
    }

    &-input {
      border: none;
      background-color: transparent;
      outline: none;
      width: 100%;
      min-width: 0;
      font-size: inherit;
      color: inherit;

      &::placeholder {
        color: var(--v1-shell-text-muted, var(--el-text-color-secondary));
      }
    }
  }

  &__user {
    display: flex;
    align-items: center;
    gap: 8px;
    min-width: 0;
    padding: 3px 8px;
    border-radius: 999px;
    background: rgba(255, 255, 255, 0.42);
    border: 1px solid rgba(255, 255, 255, 0.5);
  }

  &__user-avatar {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: var(--v1-shell-primary-soft, var(--el-color-primary-light-9));
    color: var(--v1-shell-primary, var(--el-color-primary));
    font-size: 11px;
    font-weight: 700;
    flex-shrink: 0;
  }

  &__user-name {
    max-width: 120px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    color: var(--v1-shell-text-secondary, var(--el-text-color-regular));
  }

  &__datetime {
    display: flex;
    align-items: center;
    gap: 8px;
    color: var(--v1-shell-text-secondary, var(--el-text-color-regular));
    flex-shrink: 0;
  }

  &__date {
    opacity: 0.72;
  }

  &__time {
    flex-shrink: 0;
    font-variant-numeric: tabular-nums;
  }
}
</style>
