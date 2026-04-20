<template>
  <div class="docflow-container">
    <DocFlowAside />
    <div class="docflow-content">
      <DocFlowSection />
    </div>
  </div>
</template>

<script>
import DocFlowAside from './Aside/DocFlowAside.vue'
import DocFlowSection from './Section/DocFlowSection.vue'

export default {
  name: 'DocFlow',
  components: {
    DocFlowAside,
    DocFlowSection
  }
}
</script>

<style scoped>
.docflow-container {
  display: flex;
  height: 100%;
  width: 100%;
}

.docflow-content {
  flex: 1;
  overflow: auto;
  padding: 20px;
}
</style>