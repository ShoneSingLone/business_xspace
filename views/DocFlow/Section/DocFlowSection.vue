<template>
  <div class="docflow-section">
    <div class="section-header">
      <h2>流程列表</h2>
      <button class="create-btn">创建流程</button>
    </div>
    <div class="section-content">
      <div class="flow-card">
        <div class="flow-card-header">
          <h3>API文档审批流程</h3>
          <span class="flow-status">进行中</span>
        </div>
        <div class="flow-card-body">
          <p>流程描述：API文档的审批流程，包括创建、审核、发布等环节</p>
          <div class="flow-progress">
            <div class="progress-step active">
              <span>创建</span>
            </div>
            <div class="progress-step active">
              <span>审核</span>
            </div>
            <div class="progress-step">
              <span>发布</span>
            </div>
          </div>
        </div>
        <div class="flow-card-footer">
          <span>创建时间：2026-04-07</span>
          <button class="detail-btn">查看详情</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DocFlowSection'
}
</script>

<style scoped>
.docflow-section {
  width: 100%;
  height: 100%;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.section-header h2 {
  margin: 0;
  font-size: 20px;
  font-weight: 600;
}

.create-btn {
  padding: 8px 16px;
  background-color: #1890ff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
}

.create-btn:hover {
  background-color: #40a9ff;
}

.section-content {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}

.flow-card {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 20px;
  transition: all 0.3s;
}

.flow-card:hover {
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}

.flow-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.flow-card-header h3 {
  margin: 0;
  font-size: 16px;
  font-weight: 600;
}

.flow-status {
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 12px;
  background-color: #f6ffed;
  color: #52c41a;
}

.flow-card-body {
  margin-bottom: 15px;
}

.flow-card-body p {
  margin: 0 0 15px 0;
  font-size: 14px;
  color: #666;
}

.flow-progress {
  display: flex;
  gap: 10px;
}

.progress-step {
  flex: 1;
  padding: 8px 12px;
  border-radius: 4px;
  text-align: center;
  font-size: 12px;
  background-color: #f5f5f5;
  color: #666;
}

.progress-step.active {
  background-color: #e6f7ff;
  color: #1890ff;
}

.flow-card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 12px;
  color: #999;
}

.detail-btn {
  padding: 4px 12px;
  background-color: transparent;
  color: #1890ff;
  border: 1px solid #1890ff;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
}

.detail-btn:hover {
  background-color: #e6f7ff;
}
</style>