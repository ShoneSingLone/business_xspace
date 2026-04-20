<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    async uploadImage(file) {
      try {
        const formData = new FormData();
        formData.append('file', file);
        const response = await _api.xspace.saveImgByBase64(formData);
        return response.data;
      } catch (error) {
        console.error('上传图片失败:', error);
        return null;
      }
    },

    async uploadFile(file) {
      try {
        const formData = new FormData();
        formData.append('file', file);
        const response = await _api.xspace.apiResourceSingleUpload(formData);
        return response.data;
      } catch (error) {
        console.error('上传文件失败:', error);
        return null;
      }
    },

    async previewFile(fileId) {
      try {
        const response = await _api.xspace.getFileInfo({ fileId });
        return response.data;
      } catch (error) {
        console.error('获取文件信息失败:', error);
        return null;
      }
    },

    async deleteFile(fileId) {
      try {
        const response = await _api.xspace.deleteFile({ fileId });
        return response.data;
      } catch (error) {
        console.error('删除文件失败:', error);
        return false;
      }
    }
  };
}
</script>