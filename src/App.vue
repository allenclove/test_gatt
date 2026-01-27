<template>
  <div class="app">
    <!-- 顶部导航栏 -->
    <header class="header">
      <div class="header-left">
        <div class="logo">
          <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
            <line x1="16" y1="2" x2="16" y2="6"></line>
            <line x1="8" y1="2" x2="8" y2="6"></line>
            <line x1="3" y1="10" x2="21" y2="10"></line>
          </svg>
          <span>排产管理系统</span>
        </div>
      </div>
      <div class="header-right">
        <button class="btn-primary" @click="showAddTask = true">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <line x1="12" y1="5" x2="12" y2="19"></line>
            <line x1="5" y1="12" x2="19" y2="12"></line>
          </svg>
          新增任务
        </button>
      </div>
    </header>

    <!-- 主内容区 -->
    <div class="main-content">
      <!-- 左侧边栏 - 任务列表 -->
      <aside class="sidebar">
        <div class="sidebar-header">
          <h3>任务列表</h3>
          <span class="task-count">{{ tasks.length }} 个任务</span>
        </div>

        <!-- 任务类型筛选 -->
        <div class="filter-section">
          <button
            v-for="filter in filters"
            :key="filter.key"
            @click="activeFilter = filter.key"
            :class="['filter-btn', { active: activeFilter === filter.key }]"
          >
            <span class="filter-dot" :style="{ background: filter.color }"></span>
            {{ filter.label }}
          </button>
        </div>

        <!-- 任务列表 -->
        <div class="task-list">
          <div
            v-for="task in filteredTasks"
            :key="task.id"
            class="task-item"
            :class="{ selected: selectedTaskId === task.id }"
            @click="selectTask(task)"
          >
            <div class="task-item-header">
              <span class="task-name">{{ task.name }}</span>
              <span class="task-type" :style="{ color: getTypeColor(task.customClass) }">
                {{ getTypeLabel(task.customClass) }}
              </span>
            </div>
            <div class="task-item-body">
              <div class="task-dates">
                <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect>
                  <line x1="16" y1="2" x2="16" y2="6"></line>
                  <line x1="8" y1="2" x2="8" y2="6"></line>
                  <line x1="3" y1="10" x2="21" y2="10"></line>
                </svg>
                {{ formatDateRange(task.start, task.end) }}
              </div>
              <div class="task-progress">
                <div class="progress-bar">
                  <div class="progress-fill" :style="{ width: task.progress + '%' }"></div>
                </div>
                <span class="progress-text">{{ task.progress }}%</span>
              </div>
            </div>
            <div class="task-item-footer">
              <button class="btn-icon" @click.stop="editTask(task)" title="编辑">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path>
                  <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path>
                </svg>
              </button>
              <button class="btn-icon btn-danger" @click.stop="deleteTask(task.id)" title="删除">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <polyline points="3 6 5 6 21 6"></polyline>
                  <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path>
                </svg>
              </button>
            </div>
          </div>
        </div>
      </aside>

      <!-- 右侧主区域 -->
      <main class="main">
        <!-- 统计卡片 -->
        <div class="stats-row">
          <div class="stat-card">
            <div class="stat-icon" style="background: #E3F2FD; color: #1976D2;">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M9 11l3 3L22 4"></path>
                <path d="M21 12v7a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11"></path>
              </svg>
            </div>
            <div class="stat-content">
              <div class="stat-label">总任务数</div>
              <div class="stat-value">{{ tasks.length }}</div>
            </div>
          </div>
          <div class="stat-card">
            <div class="stat-icon" style="background: #E8F5E9; color: #388E3C;">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <circle cx="12" cy="12" r="10"></circle>
                <polyline points="12 6 12 12 16 14"></polyline>
              </svg>
            </div>
            <div class="stat-content">
              <div class="stat-label">进行中</div>
              <div class="stat-value">{{ inProgressCount }}</div>
            </div>
          </div>
          <div class="stat-card">
            <div class="stat-icon" style="background: #FFF3E0; color: #F57C00;">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M12 2v4"></path>
                <path d="M12 18v4"></path>
                <path d="M4.93 4.93l2.83 2.83"></path>
                <path d="M16.24 16.24l2.83 2.83"></path>
                <path d="M2 12h4"></path>
                <path d="M18 12h4"></path>
                <path d="M4.93 19.07l2.83-2.83"></path>
                <path d="M16.24 7.76l2.83-2.83"></path>
              </svg>
            </div>
            <div class="stat-content">
              <div class="stat-label">待开始</div>
              <div class="stat-value">{{ pendingCount }}</div>
            </div>
          </div>
          <div class="stat-card">
            <div class="stat-icon" style="background: #F3E5F5; color: #7B1FA2;">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"></path>
                <polyline points="22 4 12 14.01 9 11.01"></polyline>
              </svg>
            </div>
            <div class="stat-content">
              <div class="stat-label">已完成</div>
              <div class="stat-value">{{ completedCount }}</div>
            </div>
          </div>
        </div>

        <!-- 甘特图 -->
        <div class="gantt-wrapper">
          <GanttChart
            v-if="tasks.length > 0"
            :tasks="tasks"
            :view-mode="viewMode"
            @task-updated="handleTaskUpdate"
            @task-click="handleTaskClick"
            @date-changed="handleDateChange"
          />
        </div>
      </main>
    </div>

    <!-- 新增/编辑任务弹窗 -->
    <div v-if="showAddTask || editingTask" class="modal-overlay" @click.self="closeModal">
      <div class="modal">
        <div class="modal-header">
          <h3>{{ editingTask ? '编辑任务' : '新增任务' }}</h3>
          <button class="btn-close" @click="closeModal">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="18" y1="6" x2="6" y2="18"></line>
              <line x1="6" y1="6" x2="18" y2="18"></line>
            </svg>
          </button>
        </div>
        <div class="modal-body">
          <div class="form-group">
            <label>任务名称</label>
            <input v-model="taskForm.name" type="text" placeholder="请输入任务名称">
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>加工区域</label>
              <select v-model="taskForm.zoneId">
                <option v-for="zone in zones" :key="zone.id" :value="zone.id">{{ zone.name }}</option>
              </select>
            </div>
            <div class="form-group">
              <label>任务类型</label>
              <select v-model="taskForm.customClass">
                <option value="order-task">订单任务</option>
                <option value="maintenance-task">设备维护</option>
                <option value="quality-task">质检工序</option>
              </select>
            </div>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>开始时间</label>
              <input v-model="taskForm.start" type="datetime-local">
            </div>
            <div class="form-group">
              <label>结束时间</label>
              <input v-model="taskForm.end" type="datetime-local">
            </div>
          </div>
          <div class="form-group">
            <label>进度 (%)</label>
            <input v-model.number="taskForm.progress" type="number" min="0" max="100">
          </div>
        </div>
        <div class="modal-footer">
          <button class="btn-secondary" @click="closeModal">取消</button>
          <button class="btn-primary" @click="saveTask">保存</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import GanttChart from './components/GanttChart.vue'

const viewMode = ref('day')
const selectedTaskId = ref(null)
const showAddTask = ref(false)
const editingTask = ref(null)
const activeFilter = ref('all')

const viewModes = [
  { key: 'day', label: '日视图' },
  { key: 'week', label: '周视图' },
  { key: 'month', label: '月视图' }
]

const filters = [
  { key: 'all', label: '全部', color: '#666' },
  { key: 'order-task', label: '订单任务', color: '#2196F3' },
  { key: 'maintenance-task', label: '设备维护', color: '#FF9800' },
  { key: 'quality-task', label: '质检工序', color: '#9C27B0' }
]

const taskForm = ref({
  name: '',
  start: '',
  end: '',
  progress: 0,
  customClass: 'order-task',
  zoneId: 'zone-1'
})

// 生成大量模拟任务数据
const generateMockTasks = () => {
  const taskTypes = [
    { type: 'order-task', name: '订单', count: 25 },
    { type: 'maintenance-task', name: '设备维护', count: 5 },
    { type: 'quality-task', name: '质检工序', count: 8 }
  ]

  const tasks = []
  const zoneCount = 12
  let taskId = 1

  // 为每种任务类型生成数据
  taskTypes.forEach(({ type, name: typeName, count }) => {
    for (let i = 1; i <= count; i++) {
      // 随机分配到不同区域
      const zoneId = `zone-${Math.floor(Math.random() * zoneCount) + 1}`

      // 随机开始时间 (8:00 - 16:00)
      const startHour = 8 + Math.floor(Math.random() * 9)
      const startMin = Math.random() > 0.5 ? 0 : 30
      const startTime = `${startHour.toString().padStart(2, '0')}:${startMin.toString().padStart(2, '0')}`

      // 随机持续时间 (1-4小时)
      const duration = 1 + Math.floor(Math.random() * 4)
      const endHour = startHour + duration
      const endTime = `${endHour.toString().padStart(2, '0')}:${startMin.toString().padStart(2, '0')}`

      // 随机进度
      const progress = Math.random() > 0.3 ? Math.floor(Math.random() * 100) : 0

      // 生成单号
      const orderNo = `SO${(2026000 + taskId).toString().slice(-6)}`
      const productName = `产品-${String.fromCharCode(65 + Math.floor(Math.random() * 6))}${Math.floor(Math.random() * 1000)}`

      tasks.push({
        id: String(taskId),
        name: `${typeName}-${orderNo}`,
        orderNo,
        productName,
        start: `2026-01-28T${startTime}`,
        end: `2026-01-28T${endTime}`,
        progress,
        duration: duration * 60,
        dependencies: [],
        customClass: type,
        zoneId
      })
      taskId++
    }
  })

  return tasks
}

// 模拟排产任务数据
const tasks = ref(generateMockTasks())

// 加工区域列表
const zones = [
  { id: 'zone-1', name: 'A01-数控车床' },
  { id: 'zone-2', name: 'A02-数控车床' },
  { id: 'zone-3', name: 'A03-数控铣床' },
  { id: 'zone-4', name: 'A04-数控铣床' },
  { id: 'zone-5', name: 'B01-加工中心' },
  { id: 'zone-6', name: 'B02-加工中心' },
  { id: 'zone-7', name: 'B03-线切割' },
  { id: 'zone-8', name: 'B04-电火花' },
  { id: 'zone-9', name: 'C01-热处理' },
  { id: 'zone-10', name: 'C02-喷涂线' },
  { id: 'zone-11', name: 'C03-组装区' },
  { id: 'zone-12', name: 'C04-质检区' }
]

const filteredTasks = computed(() => {
  if (activeFilter.value === 'all') return tasks.value
  return tasks.value.filter(t => t.customClass === activeFilter.value)
})

const inProgressCount = computed(() => tasks.value.filter(t => t.progress > 0 && t.progress < 100).length)
const pendingCount = computed(() => tasks.value.filter(t => t.progress === 0).length)
const completedCount = computed(() => tasks.value.filter(t => t.progress === 100).length)

const selectTask = (task) => {
  selectedTaskId.value = task.id
}

const handleTaskUpdate = (task) => {
  const index = tasks.value.findIndex(t => t.id === task.id)
  if (index !== -1) {
    tasks.value[index] = { ...tasks.value[index], ...task }
  }
}

const handleTaskClick = (task) => {
  selectedTaskId.value = task.id
}

const handleDateChange = (date) => {
  console.log('日期变更:', date)
}

const editTask = (task) => {
  editingTask.value = task
  taskForm.value = {
    name: task.name,
    start: task.start,
    end: task.end,
    progress: task.progress,
    customClass: task.customClass,
    zoneId: task.zoneId || 'zone-1'
  }
}

const deleteTask = (id) => {
  if (confirm('确定要删除这个任务吗？')) {
    tasks.value = tasks.value.filter(t => t.id !== id)
  }
}

const saveTask = () => {
  if (editingTask.value) {
    // 编辑
    const index = tasks.value.findIndex(t => t.id === editingTask.value.id)
    if (index !== -1) {
      tasks.value[index] = {
        ...tasks.value[index],
        name: taskForm.value.name,
        start: taskForm.value.start,
        end: taskForm.value.end,
        progress: taskForm.value.progress,
        customClass: taskForm.value.customClass,
        zoneId: taskForm.value.zoneId
      }
    }
  } else {
    // 新增
    const newId = String(Date.now())
    tasks.value.push({
      id: newId,
      ...taskForm.value
    })
  }
  closeModal()
}

const closeModal = () => {
  showAddTask.value = false
  editingTask.value = null
  taskForm.value = {
    name: '',
    start: '',
    end: '',
    progress: 0,
    customClass: 'order-task',
    zoneId: 'zone-1'
  }
}

const getTypeColor = (type) => {
  const colors = {
    'order-task': '#2196F3',
    'maintenance-task': '#FF9800',
    'quality-task': '#9C27B0'
  }
  return colors[type] || '#666'
}

const getTypeLabel = (type) => {
  const labels = {
    'order-task': '订单',
    'maintenance-task': '维护',
    'quality-task': '质检'
  }
  return labels[type] || '其他'
}

const formatDateRange = (start, end) => {
  const startDate = new Date(start)
  const endDate = new Date(end)
  const formatDate = (date) => `${date.getMonth() + 1}/${date.getDate()}`
  return `${formatDate(startDate)} - ${formatDate(endDate)}`
}
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.app {
  min-height: 100vh;
  background: #f5f7fa;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

/* 头部 */
.header {
  height: 60px;
  background: white;
  border-bottom: 1px solid #e5e7eb;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 100;
}

.header-left .logo {
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 18px;
  font-weight: 600;
  color: #1f2937;
}

.btn-primary {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-primary:hover {
  background: #45a049;
}

/* 主内容 */
.main-content {
  display: flex;
  margin-top: 60px;
  min-height: calc(100vh - 60px);
}

/* 侧边栏 */
.sidebar {
  width: 320px;
  background: white;
  border-right: 1px solid #e5e7eb;
  display: flex;
  flex-direction: column;
  position: fixed;
  left: 0;
  top: 60px;
  bottom: 0;
}

.sidebar-header {
  padding: 20px;
  border-bottom: 1px solid #e5e7eb;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.sidebar-header h3 {
  font-size: 16px;
  font-weight: 600;
  color: #1f2937;
  margin: 0;
}

.task-count {
  font-size: 12px;
  color: #6b7280;
  background: #f3f4f6;
  padding: 4px 10px;
  border-radius: 12px;
}

.filter-section {
  padding: 16px 20px;
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  border-bottom: 1px solid #e5e7eb;
}

.filter-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  border: 1px solid #e5e7eb;
  background: white;
  border-radius: 16px;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s;
}

.filter-btn:hover {
  background: #f9fafb;
}

.filter-btn.active {
  background: #f3f4f6;
  border-color: #d1d5db;
}

.filter-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

.task-list {
  flex: 1;
  overflow-y: auto;
  padding: 12px;
}

.task-item {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 12px;
  margin-bottom: 8px;
  cursor: pointer;
  transition: all 0.2s;
}

.task-item:hover {
  border-color: #d1d5db;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
}

.task-item.selected {
  border-color: #4CAF50;
  background: #f0fdf4;
}

.task-item-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.task-name {
  font-size: 14px;
  font-weight: 500;
  color: #1f2937;
}

.task-type {
  font-size: 11px;
  padding: 2px 8px;
  background: #f3f4f6;
  border-radius: 4px;
}

.task-item-body {
  margin-bottom: 8px;
}

.task-dates {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  color: #6b7280;
  margin-bottom: 8px;
}

.task-progress {
  display: flex;
  align-items: center;
  gap: 8px;
}

.progress-bar {
  flex: 1;
  height: 6px;
  background: #e5e7eb;
  border-radius: 3px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: #4CAF50;
  border-radius: 3px;
  transition: width 0.3s;
}

.progress-text {
  font-size: 12px;
  color: #6b7280;
  min-width: 35px;
  text-align: right;
}

.task-item-footer {
  display: flex;
  gap: 8px;
}

.btn-icon {
  padding: 4px 8px;
  border: 1px solid #e5e7eb;
  background: white;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.btn-icon:hover {
  background: #f9fafb;
  border-color: #d1d5db;
}

.btn-icon.btn-danger:hover {
  background: #fef2f2;
  border-color: #fca5a5;
  color: #dc2626;
}

/* 主区域 */
.main {
  flex: 1;
  margin-left: 320px;
  padding: 24px;
}

.stats-row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  margin-bottom: 24px;
}

.stat-card {
  background: white;
  border-radius: 12px;
  padding: 20px;
  display: flex;
  align-items: center;
  gap: 16px;
  border: 1px solid #e5e7eb;
}

.stat-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.stat-content {
  flex: 1;
}

.stat-label {
  font-size: 12px;
  color: #6b7280;
  margin-bottom: 4px;
}

.stat-value {
  font-size: 24px;
  font-weight: 700;
  color: #1f2937;
}

/* 甘特图容器 */
.gantt-wrapper {
  background: white;
  border-radius: 12px;
  border: 1px solid #e5e7eb;
  overflow: hidden;
  height: calc(100vh - 280px);
  min-height: 400px;
}

/* 弹窗 */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal {
  background: white;
  border-radius: 12px;
  width: 500px;
  max-width: 90vw;
  max-height: 90vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.modal-header {
  padding: 20px;
  border-bottom: 1px solid #e5e7eb;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.modal-header h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: #1f2937;
}

.btn-close {
  padding: 4px;
  border: none;
  background: none;
  cursor: pointer;
  color: #6b7280;
  border-radius: 4px;
}

.btn-close:hover {
  background: #f3f4f6;
}

.modal-body {
  padding: 20px;
  overflow-y: auto;
}

.form-group {
  margin-bottom: 16px;
}

.form-group label {
  display: block;
  font-size: 13px;
  font-weight: 500;
  color: #374151;
  margin-bottom: 6px;
}

.form-group input,
.form-group select {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 14px;
  transition: border-color 0.2s;
}

.form-group input:focus,
.form-group select:focus {
  outline: none;
  border-color: #4CAF50;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.dependencies-checklist {
  display: flex;
  flex-direction: column;
  gap: 8px;
  max-height: 150px;
  overflow-y: auto;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  padding: 12px;
}

.checkbox-label {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  color: #374151;
  cursor: pointer;
}

.checkbox-label input[type="checkbox"] {
  width: 16px;
  height: 16px;
}

.modal-footer {
  padding: 16px 20px;
  border-top: 1px solid #e5e7eb;
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

.btn-secondary {
  padding: 10px 20px;
  border: 1px solid #d1d5db;
  background: white;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-secondary:hover {
  background: #f9fafb;
}
</style>
