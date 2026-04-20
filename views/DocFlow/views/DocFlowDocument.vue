<template>
  <div class="docflow-document">
    <div class="docflow-document-header">
      <xItem :configs="documentForm.title" />
      <div class="docflow-document-actions">
        <xBtn :configs="saveButton" />
        <xBtn :configs="shareButton" />
      </div>
    </div>
    <div class="docflow-document-content">
      <DocFlowEditor />
    </div>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  const [DocFlowEditor] = await _.$importVue([
    "@/views/DocFlow/components/DocFlowEditor.vue"
  ]);

  return {
    components: {
      DocFlowEditor
    },
    data() {
      const vm = this;
      return {
        documentForm: {
          title: {
            label: '文档标题',
            value: '新建文档',
            placeholder: '请输入文档标题'
          }
        },
        saveButton: {
          label: '保存',
          preset: 'blue',
          onClick: async () => {
            await vm.saveDocument();
          }
        },
        shareButton: {
          label: '分享',
          onClick: async () => {
            await vm.shareDocument();
          }
        }
      };
    },
    mounted() {
      // 加载文档数据
      this.loadDocument();
    },
    methods: {
      async loadDocument() {
        const id = this.$route.params.id;
        if (id && id !== 'new') {
          // 加载现有文档
          // 这里可以调用 API 获取文档数据
        }
      },
      async saveDocument() {
        // 保存文档逻辑
        console.log('保存文档:', this.documentForm.title.value);
      },
      async shareDocument() {
        // 分享文档逻辑
        console.log('分享文档');
      }
    }
  };
}
</script>

<style lang="less">
.docflow-document {
  padding: 20px;

  .docflow-document-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    gap: 20px;

    .xItem {
      flex: 1;
    }

    .docflow-document-actions {
      display: flex;
      gap: 10px;
    }
  }

  .docflow-document-content {
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
    overflow: hidden;
  }
}
</style>