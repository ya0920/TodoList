<template>
  <!-- 遮罩层 -->
  <div class="modal-mask" v-if="isShow" @click="closeModal"></div>
  <!-- 弹窗主体 -->
  <div class="modal-container" v-if="isShow">
    <div class="modal-header">
      <h3>添加新任务</h3>
      <button class="close-btn" @click="closeModal">×</button>
    </div>
    <div class="modal-body">
      <form @submit.prevent="handleSubmit">
        <!-- 任务标题 -->
        <div class="form-item">
          <label>任务标题 *</label>
          <input
            type="text"
            v-model="title"
            placeholder="输入任务标题"
            required
            class="input-field"
          />
        </div>
        <!-- 任务描述 -->
        <div class="form-item">
          <label>任务描述</label>
          <textarea
            v-model="description"
            placeholder="输入任务详情"
            rows="3"
            class="input-field"
          ></textarea>
        </div>
        <!-- 分类选择 -->
        <div class="form-item">
          <label>分类</label>
          <select v-model="selectedCategory" class="input-field">
            <option value="工作">工作</option>
            <option value="学习">学习</option>
            <option value="生活">生活</option>
            <option value="其他">其他</option>
          </select>
        </div>
        <div class="form-footer">
          <button type="button" class="btn cancel-btn" @click="closeModal">取消</button>
          <button type="submit" class="btn confirm-btn">添加任务</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits } from 'vue';

// 接收父组件的显示状态
const props = defineProps(['isShow']);
// 向父组件发射事件
const emit = defineEmits(['close', 'submit']);

// 表单数据
const title = ref('');
const description = ref('');
const selectedCategory = ref('工作');

// 关闭弹窗
const closeModal = () => {
  emit('close');
  // 重置表单
  title.value = '';
  description.value = '';
};

// 提交任务
const handleSubmit = () => {
  if (!title.value.trim()) {
    alert('任务标题不能为空！');
    return;
  }
  // 向父组件传递新任务数据
  emit('submit', {
    id: Date.now(), // 生成唯一ID
    title: title.value.trim(),
    description: description.value.trim(),
    category: selectedCategory.value,
    status: 'pending', // 初始状态为未完成
    createdAt: new Date().toISOString() // 记录创建时间
  });
  closeModal();
};
</script>

<style scoped>
.modal-mask {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 999;
}

.modal-container {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #fff;
  border-radius: 8px;
  width: 90%;
  max-width: 400px;
  z-index: 1000;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px;
  border-bottom: 1px solid #e5e7eb;
}

.modal-header h3 {
  font-size: 18px;
  font-weight: 600;
  color: #1f2937;
}

.close-btn {
  background: none;
  border: none;
  font-size: 20px;
  color: #6b7280;
  cursor: pointer;
}

.modal-body {
  padding: 16px;
}

.form-item {
  margin-bottom: 16px;
}

.form-item label {
  display: block;
  margin-bottom: 8px;
  font-size: 14px;
  color: #1f2937;
}

.input-field {
  width: 100%;
  padding: 10px;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  font-size: 14px;
}

.form-footer {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  margin-top: 20px;
}

.btn {
  padding: 8px 16px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  font-size: 14px;
}

.cancel-btn {
  background-color: #f3f4f6;
  color: #6b7280;
}

.confirm-btn {
  background-color: #2563eb;
  color: #fff;
}
</style>