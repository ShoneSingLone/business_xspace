<template>
  <div class="auth-screen">
    <div class="auth-screen__container">
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
    }
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
