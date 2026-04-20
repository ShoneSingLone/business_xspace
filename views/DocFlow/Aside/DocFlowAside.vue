<template>
  <div class="docflow-aside">
    <div class="aside-header">
      <h3>文档流程</h3>
    </div>
    <div class="aside-menu">
      <div class="menu-item active">
        <span>流程列表</span>
      </div>
      <div class="menu-item">
        <span>创建流程</span>
      </div>
      <div class="menu-item">
        <span>流程模板</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DocFlowAside'
}
</script>

<style scoped>
.docflow-aside {
  width: 240px;
  background-color: #f5f5f5;
  border-right: 1px solid #e0e0e0;
  padding: 20px;
  height: 100%;
  overflow: auto;
}

.aside-header {
  margin-bottom: 20px;
}

.aside-header h3 {
  margin: 0;
  font-size: 16px;
  font-weight: 600;
}

.aside-menu {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.menu-item {
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
}

.menu-item:hover {
  background-color: #e0e0e0;
}

.menu-item.active {
  background-color: #1890ff;
  color: white;
}
</style>