<template>
  <div class="auth-screen">
    <!-- Loading State -->
    <div v-if="isAuthenticating" class="auth-screen__loading">
      <div class="auth-screen__loading-container">
        <div class="auth-screen__loading-ring">
          <div class="auth-screen__loading-ring-inner"></div>
          <div class="auth-screen__loading-ring-outer"></div>
        </div>
        <div class="auth-screen__loading-dots">
          <span class="auth-screen__loading-dot"></span>
          <span class="auth-screen__loading-dot"></span>
          <span class="auth-screen__loading-dot"></span>
        </div>
        <p class="auth-screen__loading-text">正在验证身份...</p>
        <p class="auth-screen__loading-hint">请稍候，我们正在检查您的登录状态</p>
      </div>
    </div>

    <!-- Login Form -->
    <div v-else class="auth-screen__container">
      <div class="auth-screen__brand">
        <span class="auth-screen__brand-dot"></span>
        <span class="auth-screen__brand-text">xspace</span>
      </div>
      <h1 class="auth-screen__title">欢迎登录工作台</h1>
      <p class="auth-screen__subtitle">使用现有账号进入 v1 桌面壳层。</p>
      <LoginForm class="auth-screen__form" />
    </div>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    components: {
      LoginForm: () => _.$importVue('@/views/Login/LoginForm.vue')
    },
    data() {
      return {
        isAuthenticating: true
      };
    },
    async mounted() {
      await this.checkAuthStatus();
    },
    methods: {
      async checkAuthStatus() {
        try {
          // 模拟认证检查延迟，实际应用中应该调用 API 检查登录状态
          await new Promise(resolve => setTimeout(resolve, 2000));
          
          // 检查用户是否已经登录
          // 如果已登录，v1 主界面会自动切换显示，这里不需要手动跳转
          if (!this.APP?.user?.isLogin) {
            // 用户未登录，显示登录表单
            this.isAuthenticating = false;
          }
        } catch (error) {
          console.error('认证检查失败:', error);
          // 检查失败时显示登录表单
          this.isAuthenticating = false;
        }
      }
    },
    inject: ['APP']
  };
}
</script>

<style lang="less">
.auth-screen {
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  background:
    radial-gradient(circle at 12% 12%, rgba(49, 130, 206, 0.16) 0%, rgba(49, 130, 206, 0) 34%),
    radial-gradient(circle at 84% 10%, rgba(99, 179, 237, 0.14) 0%, rgba(99, 179, 237, 0) 28%),
    linear-gradient(180deg, rgba(255, 255, 255, 0.72) 0%, rgba(255, 255, 255, 0.2) 100%),
    var(--body-bg-color, #f4f9fd);

  &__loading {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  &__loading-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 24px;
  }

  &__loading-ring {
    position: relative;
    width: 120px;
    height: 120px;

    &-inner,
    &-outer {
      position: absolute;
      inset: 0;
      border-radius: 50%;
      border: 3px solid transparent;
    }

    &-inner {
      border-top-color: var(--el-color-primary, #3182ce);
      border-right-color: var(--el-color-primary, #3182ce);
      animation: spinInner 1.5s linear infinite;
    }

    &-outer {
      border-bottom-color: var(--el-color-primary-light-5, #93c5fd);
      border-left-color: var(--el-color-primary-light-5, #93c5fd);
      animation: spinOuter 2s linear infinite;
    }
  }

  &__loading-dots {
    display: flex;
    gap: 8px;

    & > span {
      width: 8px;
      height: 8px;
      border-radius: 50%;
      background: var(--el-color-primary, #3182ce);
      animation: bounceDot 1.4s ease-in-out infinite both;

      &:nth-child(1) {
        animation-delay: -0.32s;
      }

      &:nth-child(2) {
        animation-delay: -0.16s;
      }

      &:nth-child(3) {
        animation-delay: 0s;
      }
    }
  }

  &__loading-text {
    font-size: 1.125rem;
    font-weight: 600;
    color: var(--el-text-color-primary, #303133);
    margin: 0;
  }

  &__loading-hint {
    font-size: 0.875rem;
    color: var(--el-text-color-secondary, #909399);
    margin: 0;
  }
}

@keyframes spinInner {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

@keyframes spinOuter {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(-360deg);
  }
}

@keyframes bounceDot {
  0%,
  80%,
  100% {
    transform: scale(0);
    opacity: 0.5;
  }
  40% {
    transform: scale(1);
    opacity: 1;
  }
}

.auth-screen__container {
  width: 100%;
  max-width: 420px;
  padding: 32px;
  background: rgba(255, 255, 255, 0.92);
  border: 1px solid var(--el-border-color-lighter, #ebeef5);
  border-radius: 20px;
  box-shadow: var(--el-box-shadow);
  backdrop-filter: blur(20px);
}

.auth-screen__brand {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 16px;
}

.auth-screen__brand-dot {
  width: 10px;
  height: 10px;
  border-radius: 999px;
  background: var(--el-color-primary, #3182ce);
  box-shadow: 0 0 0 6px var(--el-color-primary-light-9, #eff6ff);
}

.auth-screen__brand-text {
  font-size: 0.875rem;
  font-weight: 600;
  letter-spacing: 0.04em;
  color: var(--el-text-color-regular, #606266);
}

.auth-screen__title {
  font-size: 1.75rem;
  font-weight: 700;
  margin-bottom: 8px;
  color: var(--el-text-color-primary, #303133);
}

.auth-screen__subtitle {
  margin: 0 0 24px;
  font-size: 0.9375rem;
  line-height: 1.6;
  color: var(--el-text-color-secondary, #909399);
}

.auth-screen__form {
  display: flex;
  flex-direction: column;
}

// 移动端适配
body.app-mobile {
  .auth-screen__container {
    padding: 24px;
  }
}
</style>
