<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  return {
    async aiRequest(type, text) {
      try {
        const response = await _api.xspace.aiRequest({ type, text });
        return response.data;
      } catch (error) {
        console.error('AI 请求失败:', error);
        return null;
      }
    },

    async brainstorm(prompt) {
      return this.aiRequest('brainstorm', prompt);
    },

    async polish(text) {
      return this.aiRequest('polish', text);
    },

    async continueWriting(text) {
      return this.aiRequest('continue', text);
    },

    async summarize(text) {
      return this.aiRequest('summarize', text);
    },

    async translate(text, targetLanguage) {
      return this.aiRequest('translate', `${text}|||${targetLanguage}`);
    }
  };
}
</script>