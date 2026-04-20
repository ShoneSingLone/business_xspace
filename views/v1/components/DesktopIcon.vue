<template>
  <div
    class="desktop-icon"
    draggable="true"
    @dragstart="onDragStart"
    @click="handleClick">
    <div class="desktop-icon__content">
      <div class="desktop-icon__icon-shell">
        <div class="desktop-icon__icon" :style="{ '--desktop-icon-color': app.color }">
          <xIcon :icon="app.icon || app.name.charAt(0)" size="24" />
        </div>
      </div>
      <span class="desktop-icon__label">{{ app.name }}</span>
      <button
        @click="handleUnpin"
        class="desktop-icon__remove"
        title="从桌面移除">
        <xIcon icon="close" size="12" />
      </button>
    </div>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    props: {
      app: Object
    },
    emits: ["click", "unpin"],
    methods: {
      onDragStart(e) {
        if (e.dataTransfer) {
          e.dataTransfer.setData("text/plain", this.app.appId);
          e.dataTransfer.effectAllowed = "copyMove";
        }
      },
      handleClick() {
        this.$emit("click");
      },
      handleUnpin(event) {
        event.stopPropagation();
        this.$emit("unpin");
      }
    }
  };
}
</script>

<style lang="less">
.desktop-icon {
  position: relative;
  display: flex;
  width: 92px;
  min-height: 112px;
  border-radius: 18px;
  cursor: pointer;
  transition:
    transform 180ms cubic-bezier(0.4, 0, 0.2, 1),
    background-color 180ms cubic-bezier(0.4, 0, 0.2, 1),
    box-shadow 180ms cubic-bezier(0.4, 0, 0.2, 1);
  background: transparent;

  &:hover {
    transform: translateY(-2px);
    background: rgba(255, 255, 255, 0.46);
    box-shadow: 0 12px 24px rgba(49, 130, 206, 0.1);
  }

  &:active {
    transform: translateY(0) scale(0.98);
  }

  &__content {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    gap: 10px;
    width: 100%;
    min-height: inherit;
    padding: 10px 8px 12px;
  }

  &__icon-shell {
    width: 64px;
    height: 64px;
    border-radius: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    background:
      linear-gradient(180deg, rgba(255, 255, 255, 0.92) 0%, rgba(245, 248, 247, 0.92) 100%);
    border: 1px solid var(--v1-shell-border, var(--el-border-color-lighter));
    box-shadow:
      inset 0 1px 0 rgba(255, 255, 255, 0.85),
      0 10px 24px rgba(49, 130, 206, 0.12);
    flex-shrink: 0;
  }

  &__icon {
    width: 50px;
    height: 50px;
    border-radius: 16px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: var(--desktop-icon-color, var(--v1-shell-primary, var(--el-color-primary)));
    color: #ffffff;
    transition:
      transform 180ms cubic-bezier(0.4, 0, 0.2, 1),
      box-shadow 180ms cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 0 10px 18px rgba(49, 130, 206, 0.18);

    .desktop-icon:hover & {
      transform: translateY(-1px) scale(1.04);
      box-shadow: 0 14px 24px rgba(49, 130, 206, 0.22);
    }
  }

  &__label {
    display: -webkit-box;
    font-size: 12px;
    font-weight: 600;
    line-height: 1.35;
    color: var(--v1-shell-text, var(--el-text-color-primary));
    overflow: hidden;
    text-overflow: ellipsis;
    text-align: center;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
  }

  &__remove {
    position: absolute;
    top: 4px;
    right: 4px;
    transform: scale(0.9);
    width: 24px;
    height: 24px;
    padding: 0;
    background: rgba(255, 255, 255, 0.82);
    color: #f56c6c;
    border-radius: 999px;
    border: 1px solid var(--v1-shell-border, var(--el-border-color-lighter));
    box-shadow: 0 8px 18px rgba(48, 49, 51, 0.12);
    opacity: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    transition:
      opacity 150ms cubic-bezier(0.4, 0, 0.2, 1),
      transform 150ms cubic-bezier(0.4, 0, 0.2, 1),
      background-color 150ms cubic-bezier(0.4, 0, 0.2, 1);

    .desktop-icon:hover & {
      opacity: 1;
      transform: scale(1);
    }

    &:hover {
      background: #ffffff;
    }

    &:active {
      transform: scale(0.96);
    }
  }
}
</style>
