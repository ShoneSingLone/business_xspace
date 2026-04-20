<template>
  <div class="docflow-index">
    <h1>DocFlow 文档管理系统</h1>
    <p>欢迎使用 DocFlow，一个现代化的文档编辑和协作平台。</p>
    <div class="docflow-index-actions">
      <xBtn :configs="listButton" />
      <xBtn :configs="createButton" />
    </div>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    data() {
      const vm = this;
      return {
        listButton: {
          label: '查看文档列表',
          preset: 'blue',
          onClick: async () => {
            vm.$router.push('/docflow/list');
          }
        },
        createButton: {
          label: '创建新文档',
          preset: 'green',
          onClick: async () => {
            vm.$router.push('/docflow/document/new');
          }
        }
      };
    }
  };
}
</script>

<style lang="less">
.docflow-index {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  padding: 40px;
  text-align: center;

  h1 {
    font-size: 32px;
    font-weight: 600;
    margin-bottom: 16px;
    color: #1890ff;
  }

  p {
    font-size: 18px;
    margin-bottom: 32px;
    color: #606266;
  }

  .docflow-index-actions {
    display: flex;
    gap: 16px;
  }
}
</style>