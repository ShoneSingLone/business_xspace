<template>
  <div class="docflow-list">
    <div class="docflow-list-header">
      <h2>文档列表</h2>
      <xBtn :configs="createButton" />
    </div>
    <div class="docflow-list-content">
      <div v-for="document in documents" :key="document.id" class="docflow-list-item">
        <div class="docflow-list-item-title" @click="openDocument(document.id)">
          {{ document.title }}
        </div>
        <div class="docflow-list-item-meta">
          <span>{{ document.createdAt }}</span>
          <span>{{ document.author }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    data() {
      const vm = this;
      return {
        documents: [
          {
            id: 1,
            title: '测试文档 1',
            createdAt: '2024-01-01',
            author: 'admin'
          },
          {
            id: 2,
            title: '测试文档 2',
            createdAt: '2024-01-02',
            author: 'user'
          }
        ],
        createButton: {
          label: '创建文档',
          preset: 'blue',
          onClick: async () => {
            await vm.createDocument();
          }
        }
      };
    },
    methods: {
      async createDocument() {
        // 创建新文档的逻辑
        this.$router.push('/docflow/document/new');
      },
      openDocument(id) {
        this.$router.push(`/docflow/document/${id}`);
      }
    }
  };
}
</script>

<style lang="less">
.docflow-list {
  padding: 20px;

  .docflow-list-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;

    h2 {
      margin: 0;
      font-size: 24px;
      font-weight: 600;
    }
  }

  .docflow-list-content {
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
    overflow: hidden;
  }

  .docflow-list-item {
    padding: 16px;
    border-bottom: 1px solid #e4e7ed;
    cursor: pointer;

    &:hover {
      background: #f5f7fa;
    }

    &:last-child {
      border-bottom: none;
    }

    .docflow-list-item-title {
      font-size: 16px;
      font-weight: 500;
      margin-bottom: 8px;
    }

    .docflow-list-item-meta {
      display: flex;
      gap: 16px;
      font-size: 14px;
      color: #909399;
    }
  }
}
</style>