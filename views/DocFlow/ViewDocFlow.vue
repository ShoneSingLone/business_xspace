<template>
  <div class="docflow-container">
    <DocFlowHeader />
    <div class="docflow-content">
      <router-view />
    </div>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  const [DocFlowHeader] = await _.$importVue([
    "@/views/DocFlow/components/DocFlowHeader.vue"
  ]);

  return {
    components: {
      DocFlowHeader
    },
    provide() {
      return {
        DocFlow: this
      };
    },
    data() {
      return {
        currentDocument: null
      };
    },
    methods: {
      async createDocument() {
        // 创建新文档的逻辑
        this.$router.push('/docflow/document/new');
      },
      async loadDocument(id) {
        // 加载文档的逻辑
        console.log('加载文档:', id);
      }
    }
  };
}
</script>

<style lang="less">
.docflow-container {
  display: flex;
  flex-direction: column;
  height: 100%;

  .docflow-content {
    flex: 1;
    overflow: auto;
  }
}
</style>