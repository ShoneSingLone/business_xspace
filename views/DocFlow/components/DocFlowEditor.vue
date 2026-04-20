<template>
  <div class="docflow-editor">
    <div class="docflow-editor-toolbar">
      <div class="docflow-editor-toolbar-group">
        <xBtn :configs="formatButtons.bold" />
        <xBtn :configs="formatButtons.italic" />
        <xBtn :configs="formatButtons.underline" />
        <xBtn :configs="formatButtons.strike" />
      </div>
      <div class="docflow-editor-toolbar-group">
        <xBtn :configs="blockButtons.heading1" />
        <xBtn :configs="blockButtons.heading2" />
        <xBtn :configs="blockButtons.heading3" />
      </div>
      <div class="docflow-editor-toolbar-group">
        <xBtn :configs="listButtons.bulletList" />
        <xBtn :configs="listButtons.orderedList" />
        <xBtn :configs="listButtons.taskList" />
      </div>
      <div class="docflow-editor-toolbar-group">
        <xBtn :configs="insertButtons.image" />
        <xBtn :configs="insertButtons.table" />
        <xBtn :configs="insertButtons.code" />
      </div>
    </div>
    <div class="docflow-editor-content" ref="editorContainer"></div>
  </div>
</template>

<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    data() {
      const vm = this;
      return {
        editor: null,
        formatButtons: {
          bold: {
            label: 'B',
            onClick: () => {
              if (vm.editor) {
                vm.editor.commands.toggleBold();
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          },
          italic: {
            label: 'I',
            onClick: () => {
              if (vm.editor) {
                vm.editor.commands.toggleItalic();
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          },
          underline: {
            label: 'U',
            onClick: () => {
              if (vm.editor) {
                vm.editor.commands.toggleUnderline();
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          },
          strike: {
            label: 'S',
            onClick: () => {
              if (vm.editor) {
                vm.editor.commands.toggleStrike();
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          }
        },
        blockButtons: {
          heading1: {
            label: 'H1',
            onClick: () => {
              if (vm.editor) {
                vm.editor.commands.toggleHeading({ level: 1 });
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          },
          heading2: {
            label: 'H2',
            onClick: () => {
              if (vm.editor) {
                vm.editor.commands.toggleHeading({ level: 2 });
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          },
          heading3: {
            label: 'H3',
            onClick: () => {
              if (vm.editor) {
                vm.editor.commands.toggleHeading({ level: 3 });
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          }
        },
        listButtons: {
          bulletList: {
            label: '•',
            onClick: () => {
              if (vm.editor) {
                vm.editor.commands.toggleBulletList();
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          },
          orderedList: {
            label: '1.',
            onClick: () => {
              if (vm.editor) {
                vm.editor.commands.toggleOrderedList();
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          },
          taskList: {
            label: '✓',
            onClick: () => {
              if (vm.editor) {
                vm.editor.commands.toggleTaskList();
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          }
        },
        insertButtons: {
          image: {
            label: '图片',
            onClick: () => {
              if (vm.editor) {
                vm.insertImage();
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          },
          table: {
            label: '表格',
            onClick: () => {
              if (vm.editor) {
                vm.insertTable();
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          },
          code: {
            label: '代码',
            onClick: () => {
              if (vm.editor) {
                vm.insertCode();
              } else {
                vm.showEditorNotLoadedMessage();
              }
            }
          }
        }
      };
    },
    mounted() {
      this.initEditor();
    },
    methods: {
      initEditor() {
        // 初始化 Tiptap 编辑器
        // 由于我们使用 CDN 或公共库引入，这里需要确保 Tiptap 已经加载
        if (window.Tiptap && window.TiptapStarterKit) {
          try {
            const { Editor } = window.Tiptap;
            this.editor = new Editor({
              element: this.$refs.editorContainer,
              content: '<p>开始编辑文档...</p>',
              extensions: [
                window.TiptapStarterKit
                // 其他扩展
              ],
              onUpdate: ({ editor }) => {
                console.log('编辑器内容更新:', editor.getHTML());
              }
            });
          } catch (error) {
            console.error('初始化 Tiptap 编辑器失败:', error);
            this.initFallbackEditor();
          }
        } else {
          console.error('Tiptap 未加载，使用备选编辑器');
          this.initFallbackEditor();
        }
      },
      initFallbackEditor() {
        // 备选编辑器：使用简单的 contenteditable 元素
        const container = this.$refs.editorContainer;
        container.innerHTML = '<div contenteditable="true" style="min-height: 400px; padding: 20px; font-size: 16px; line-height: 1.6;">开始编辑文档...</div>';
        const editable = container.querySelector('[contenteditable]');
        if (editable) {
          editable.addEventListener('input', () => {
            console.log('备选编辑器内容更新:', editable.innerHTML);
          });
        }
      },
      insertImage() {
        // 实现图片插入逻辑
        console.log('插入图片');
      },
      insertTable() {
        // 实现表格插入逻辑
        console.log('插入表格');
      },
      insertCode() {
        // 实现代码块插入逻辑
        console.log('插入代码块');
      },
      showEditorNotLoadedMessage() {
        // 显示编辑器未加载的提示信息
        if (window._ && window._.$message) {
          window._.$message({
            type: 'warning',
            message: '编辑器未完全加载，部分功能可能不可用'
          });
        } else {
          alert('编辑器未完全加载，部分功能可能不可用');
        }
      }
    }
  };
}
</script>

<style lang="less">
.docflow-editor {
  border: 1px solid #e4e7ed;
  border-radius: 8px;
  overflow: hidden;

  .docflow-editor-toolbar {
    display: flex;
    gap: 10px;
    padding: 10px;
    background: #f5f7fa;
    border-bottom: 1px solid #e4e7ed;

    .docflow-editor-toolbar-group {
      display: flex;
      gap: 5px;
      padding: 0 10px;
      border-right: 1px solid #e4e7ed;

      &:last-child {
        border-right: none;
      }
    }

    .xBtn {
      min-width: 32px;
      height: 32px;
      padding: 0;
      display: flex;
      align-items: center;
      justify-content: center;
    }
  }

  .docflow-editor-content {
    min-height: 400px;
    padding: 20px;
    font-size: 16px;
    line-height: 1.6;

    :deep(p) {
      margin-bottom: 16px;
    }

    :deep(h1) {
      font-size: 24px;
      font-weight: 600;
      margin: 24px 0 16px;
    }

    :deep(h2) {
      font-size: 20px;
      font-weight: 600;
      margin: 20px 0 16px;
    }

    :deep(h3) {
      font-size: 18px;
      font-weight: 600;
      margin: 18px 0 16px;
    }

    :deep(ul),
    :deep(ol) {
      margin-bottom: 16px;
      padding-left: 24px;
    }

    :deep(li) {
      margin-bottom: 8px;
    }

    :deep(code) {
      background: #f5f7fa;
      padding: 2px 6px;
      border-radius: 4px;
      font-family: 'Courier New', Courier, monospace;
    }

    :deep(pre) {
      background: #f5f7fa;
      padding: 16px;
      border-radius: 8px;
      overflow: auto;
      margin-bottom: 16px;

      code {
        background: none;
        padding: 0;
      }
    }

    :deep(table) {
      border-collapse: collapse;
      width: 100%;
      margin-bottom: 16px;

      th,
      td {
        border: 1px solid #e4e7ed;
        padding: 8px 12px;
        text-align: left;
      }

      th {
        background: #f5f7fa;
        font-weight: 600;
      }
    }

    :deep(img) {
      max-width: 100%;
      height: auto;
      border-radius: 8px;
      margin: 16px 0;
    }
  }
}
</style>