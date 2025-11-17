<template>
  <div class="container">
    <!-- 头部标题 -->
    <header class="header">
      <h1 class="title">Todo List</h1>
      <p class="subtitle">简洁高效的个人任务管理工具</p>
    </header>

    <!-- 筛选与批量操作栏 -->
    <div class="toolbar">
      <!-- 分类筛选 -->
      <select v-model="selectedCategory" class="category-select">
        <option value="all">全部分类</option>
        <option value="工作">工作</option>
        <option value="学习">学习</option>
        <option value="生活">生活</option>
        <option value="其他">其他</option>
      </select>

      <!-- 批量操作按钮 -->
      <div class="batch-buttons">
        <button class="btn btn-success" @click="batchComplete">批量完成</button>
        <button class="btn btn-warning" @click="batchUncomplete">批量未完成</button>
        <button class="btn btn-danger" @click="batchDelete">批量删除</button>
      </div>
    </div>

    <!-- 任务列表区域 -->
    <div class="task-section">
      <div class="task-header">
        <h2 class="task-title">任务列表 ({{ filteredTodos.length }}个任务)</h2>
        <button class="sort-btn" @click="toggleSortOrder">
          <span>时间排序</span>
          <span :class="isDescending ? 'arrow-down' : 'arrow-up'"></span>
        </button>
      </div>

      <!-- 全选与选中计数 -->
      <div class="select-all">
        <input type="checkbox" v-model="isAllSelected" class="select-all-checkbox" />
        <label>全选</label>
        <span class="selected-count">已选 {{ selectedCount }}/{{ filteredTodos.length }}</span>
      </div>

      <!-- 任务列表 -->
      <ul class="task-list">
        <li 
          v-for="todo in filteredTodos" 
          :key="todo.id" 
          :class="['task-item', { 'completed': todo.status === 'completed' }]"
        >
          <input 
            type="checkbox" 
            v-model="todo.status" 
            class="task-checkbox"
            @change="updateTodo(todo)"
          />
          <div class="task-content">
            <h3 class="task-title">{{ todo.title }}</h3>
            <p v-if="todo.description" class="task-desc">{{ todo.description }}</p>
            <div class="task-meta">
              <span :class="['category-tag', `tag-${todo.category.toLowerCase()}`]">
                {{ todo.category }}
              </span>
              <span class="task-time">{{ formatTime(todo.createdAt) }}</span>
            </div>
          </div>
          <button class="delete-btn" @click="deleteTodo(todo.id)">🗑️</button>
        </li>
      </ul>
    </div>

    <!-- 悬浮添加按钮 -->
    <button class="add-btn" @click="isModalShow = true">+</button>

    <!-- 引入弹窗组件 -->
    <TodoModal
      :is-show="isModalShow"
      @close="isModalShow = false"
      @submit="addTodo"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import TodoModal from './components/TodoModal.vue';

// 分类配置
const categoryMap = {
  工作: 'blue',
  学习: 'green',
  生活: 'yellow',
  其他: 'purple'
};

// 响应式数据
const todos = ref([]); // 所有任务
const selectedCategory = ref('all'); // 选中的分类
const isDescending = ref(true); // 时间排序方向（降序）
const isModalShow = ref(false); // 弹窗显示状态
const selectedTodos = ref([]); // 选中的任务（用于批量操作）

// 从LocalStorage读取任务
onMounted(() => {
  const storedTodos = localStorage.getItem('todoList');
  if (storedTodos) {
    todos.value = JSON.parse(storedTodos);
  }
});

// 筛选+排序后的任务列表
const filteredTodos = computed(() => {
  let result = todos.value;
  // 按分类筛选
  if (selectedCategory.value !== 'all') {
    result = result.filter(todo => todo.category === selectedCategory.value);
  }
  // 按状态和时间排序（未完成在前，已完成在后；同状态下按时间排序）
  result = result.sort((a, b) => {
    // 一级排序：未完成在前
    if (a.status === 'pending' && b.status === 'completed') return -1;
    if (a.status === 'completed' && b.status === 'pending') return 1;
    // 二级排序：时间排序
    const timeA = new Date(a.createdAt).getTime();
    const timeB = new Date(b.createdAt).getTime();
    return isDescending.value ? (timeB - timeA) : (timeA - timeB);
  });
  return result;
});

// 选中的任务数量
const selectedCount = computed(() => {
  return filteredTodos.value.filter(todo => todo.status === 'completed').length;
});

// 全选状态
const isAllSelected = computed({
  get() {
    return filteredTodos.value.length > 0 && 
           filteredTodos.value.every(todo => todo.status === 'completed');
  },
  set(val) {
    filteredTodos.value.forEach(todo => {
      todo.status = val ? 'completed' : 'pending';
      updateTodo(todo);
    });
  }
});

// 格式化时间
const formatTime = (timeStr) => {
  const date = new Date(timeStr);
  return date.toLocaleString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  }).replace(/\//g, '-');
};

// 新增任务
const addTodo = (newTodo) => {
  todos.value.push(newTodo);
  saveToLocalStorage();
};

// 更新任务（状态变化）
const updateTodo = (todo) => {
  saveToLocalStorage();
};

// 删除任务
const deleteTodo = (todoId) => {
  if (confirm('确定删除该任务吗？')) {
    todos.value = todos.value.filter(todo => todo.id !== todoId);
    saveToLocalStorage();
  }
};

// 批量完成
const batchComplete = () => {
  if (selectedCount.value === 0) {
    alert('请先选择任务');
    return;
  }
  filteredTodos.value.forEach(todo => {
    todo.status = 'completed';
  });
  saveToLocalStorage();
};

// 批量未完成
const batchUncomplete = () => {
  if (selectedCount.value === 0) {
    alert('请先选择任务');
    return;
  }
  filteredTodos.value.forEach(todo => {
    todo.status = 'pending';
  });
  saveToLocalStorage();
};

// 批量删除
const batchDelete = () => {
  if (selectedCount.value === 0) {
    alert('请先选择任务');
    return;
  }
  if (confirm(`确定删除选中的 ${selectedCount.value} 个任务吗？`)) {
    todos.value = todos.value.filter(todo => {
      return !filteredTodos.value.includes(todo) || todo.status === 'pending';
    });
    saveToLocalStorage();
  }
};

// 切换时间排序方向
const toggleSortOrder = () => {
  isDescending.value = !isDescending.value;
};

// 保存到LocalStorage
const saveToLocalStorage = () => {
  localStorage.setItem('todoList', JSON.stringify(todos.value));
};
</script>

<style scoped>
.container {
  margin: 0 auto;
  padding: 50px 100px;
  min-height: 100vh;
  background-color: #f3f4f6;
}

.header {
  text-align: center;
  margin-bottom: 30px;
}

.title {
  font-size: 28px;
  color: #2563eb;
  margin-bottom: 8px;
}

.subtitle {
  font-size: 14px;
  color: #6b7280;
}

.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.category-select-wrapper {
  position: relative;
  margin-right: 20px;
}

.category-select {
  padding: 8px 30px 8px 12px;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  background-color: #fff;
  appearance: none;
  cursor: pointer;
}

.select-arrow {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  width: 12px;
  height: 12px;
  pointer-events: none;
}

.batch-buttons {
  display: flex;
  gap: 8px;
}

.btn {
  padding: 8px 16px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  font-size: 14px;
}

.btn-primary {
  background-color: #3b82f6;
  color: #fff;
}

.btn-success {
  background-color: #22c55e;
  color: #fff;
}

.btn-warning {
  background-color: #eab308;
  color: #fff;
}

.btn-danger {
  background-color: #ef4444;
  color: #fff;
}

.badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 18px;
  height: 18px;
  border-radius: 9px;
  background-color: #fff;
  color: #ef4444;
  font-size: 12px;
  font-weight: 600;
  margin-left: 6px;
}

.btn-secondary {
  background-color: #6b7280;
  color: #fff;
}

.task-section {
  margin-bottom: 20px;
}

.task-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.task-title {
  font-size: 18px;
  color: #1f2937;
}

.sort-btn {
  display: flex;
  align-items: center;
  gap: 4px;
  background: none;
  border: none;
  color: #6b7280;
  cursor: pointer;
  font-size: 14px;
}

.sort-icon {
  width: 14px;
  height: 14px;
}

.select-all {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 16px;
  color: #6b7280;
  font-size: 14px;
}

.select-all-checkbox {
  cursor: pointer;
}

.selected-count {
  margin-left: 8px;
}

.task-list {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  padding: 0;
}

.task-item {
  flex: 1 1 calc(50% - 16px);
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  padding: 16px;
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

.task-item:last-child {
  border-bottom: none;
}

.task-item.completed .task-title {
  text-decoration: line-through;
  color: #6b7280;
}

.task-checkbox {
  width: 16px;
  height: 16px;
  cursor: pointer;
  margin-top: 4px;
}

.task-content {
  flex: 1;
}

.task-content .task-title {
  font-size: 16px;
  margin-bottom: 4px;
}

.task-desc {
  font-size: 14px;
  color: #6b7280;
  margin-bottom: 4px;
}

.task-meta {
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 12px;
  color: #6b7280;
}

.category-tag {
  padding: 2px 8px;
  border-radius: 999px;
  color: #fff;
  font-size: 10px;
}

.task-time {
  color: #6b7280;
}

.delete-btn {
  background: none;
  border: none;
  cursor: pointer;
  margin-top: 4px;
}

.delete-icon {
  width: 16px;
  height: 16px;
}

.add-btn {
  position: fixed;
  right: 20px;
  bottom: 20px;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #2563eb;
  color: #fff;
  border: none;
  font-size: 20px;
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}
</style>