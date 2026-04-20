<template>
  <div class="docflow-header">
    <div class="docflow-header-logo">
      <h1>DocFlow</h1>
    </div>
    <div class="docflow-header-nav">
      <router-link to="/docflow/list" class="docflow-header-nav-item">
        文档列表
      </router-link>
      <router-link to="/docflow/document/new" class="docflow-header-nav-item">
        新建文档
      </router-link>
    </div>
    <div class="docflow-header-actions">
      <xBtn :configs="importButton" />
      <xBtn :configs="exportButton" />
    </div>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    data() {
      const vm = this;
      return {
        importButton: {
          label: '导入',
          onClick: async () => {
            await vm.importDocument();
          }
        },
        exportButton: {
          label: '导出',
          onClick: async () => {
            await vm.exportDocument();
          }
        }
      };
    },
    methods: {
      async importDocument() {
        // 导入文档逻辑
        console.log('导入文档');
      },
      async exportDocument() {
        // 导出文档逻辑
        console.log('导出文档');
      }
    }
  };
}
</script>

<style lang="less">
.docflow-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
  height: 60px;
  background: #fff;
  border-bottom: 1px solid #e4e7ed;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);

  .docflow-header-logo {
    h1 {
      margin: 0;
      font-size: 20px;
      font-weight: 600;
      color: #1890ff;
    }
  }

  .docflow-header-nav {
    display: flex;
    gap: 20px;

    .docflow-header-nav-item {
      padding: 8px 16px;
      border-radius: 4px;
      color: #303133;
      text-decoration: none;
      font-size: 14px;
      font-weight: 500;

      &:hover {
        background: #f5f7fa;
        color: #1890ff;
      }

      &.router-link-active {
        background: #e6f7ff;
        color: #1890ff;
      }
    }
  }

  .docflow-header-actions {
    display: flex;
    gap: 10px;
  }
}
</style>