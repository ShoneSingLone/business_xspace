<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    async getDocuments() {
      try {
        const response = await _api.xspace.getDocuments();
        return response.data;
      } catch (error) {
        console.error('获取文档列表失败:', error);
        return [];
      }
    },

    async getDocument(id) {
      try {
        const response = await _api.xspace.getDocument({ id });
        return response.data;
      } catch (error) {
        console.error('获取文档失败:', error);
        return null;
      }
    },

    async createDocument(data) {
      try {
        const response = await _api.xspace.createDocument(data);
        return response.data;
      } catch (error) {
        console.error('创建文档失败:', error);
        return null;
      }
    },

    async updateDocument(id, data) {
      try {
        const response = await _api.xspace.updateDocument({ id, ...data });
        return response.data;
      } catch (error) {
        console.error('更新文档失败:', error);
        return null;
      }
    },

    async deleteDocument(id) {
      try {
        const response = await _api.xspace.deleteDocument({ id });
        return response.data;
      } catch (error) {
        console.error('删除文档失败:', error);
        return false;
      }
    }
  };
}
</script>