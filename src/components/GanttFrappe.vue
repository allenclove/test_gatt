<template>
  <div class="gantt-frappe-container">
    <!-- 工具栏 -->
    <div class="toolbar">
      <div class="date-picker">
        <button @click="changeDate(-1)" class="nav-btn">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polyline points="15 18 9 12 15 6"></polyline>
          </svg>
        </button>
        <input type="date" v-model="currentDate" @change="onDateChange">
        <button @click="changeDate(1)" class="nav-btn">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polyline points="9 18 15 12 9 6"></polyline>
          </svg>
        </button>
      </div>
      <div class="view-controls">
        <button
          v-for="mode in viewModes"
          :key="mode.key"
          @click="changeViewMode(mode.key)"
          :class="['view-btn', { active: viewMode === mode.key }]"
        >
          {{ mode.label }}
        </button>
      </div>
      <div class="today-btn">
        <button @click="goToToday">今天</button>
      </div>
    </div>

    <!-- Frappe Gantt SVG容器 -->
    <div class="gantt-wrapper">
      <svg id="gantt" ref="ganttSvg"></svg>
    </div>

    <!-- 任务统计 -->
    <div class="task-stats">
      <div class="stat-item">
        <span class="stat-label">总任务:</span>
        <span class="stat-value">{{ tasks.length }}</span>
      </div>
      <div class="stat-item">
        <span class="stat-label">进行中:</span>
        <span class="stat-value">{{ inProgressCount }}</span>
      </div>
      <div class="stat-item">
        <span class="stat-label">已完成:</span>
        <span class="stat-value">{{ completedCount }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted, nextTick } from 'vue'

const props = defineProps({
  tasks: {
    type: Array,
    required: true
  },
  viewMode: {
    type: String,
    default: 'day'
  }
})

const emit = defineEmits(['task-click', 'task-updated', 'date-changed'])

const viewModes = [
  { key: 'day', label: '日视图' },
  { key: 'week', label: '周视图' },
  { key: 'month', label: '月视图' }
]

const currentDate = ref(new Date().toISOString().split('T')[0])
const viewMode = ref('day')
const ganttSvg = ref(null)

let ganttInstance = null

// 转换任务格式为frappe-gantt格式
const frappeTasks = computed(() => {
  return props.tasks.map(task => ({
    id: task.id,
    name: task.orderNo || task.name,
    start: task.start.split('T')[0],
    end: task.end.split('T')[0],
    progress: task.progress,
    dependencies: task.dependencies || [],
    customClass: task.customClass || 'order-task'
  }))
})

const inProgressCount = computed(() => props.tasks.filter(t => t.progress > 0 && t.progress < 100).length)
const completedCount = computed(() => props.tasks.filter(t => t.progress === 100).length)

// 初始化Gantt
const initGantt = () => {
  if (!ganttSvg.value) return

  // 清空现有内容
  ganttSvg.value.innerHTML = ''

  // 动态加载frappe-gantt CSS和JS
  if (!window.Gantt) {
    const loadScript = () => {
      return new Promise((resolve, reject) => {
        // 加载CSS
        const cssLink = document.createElement('link')
        cssLink.rel = 'stylesheet'
        cssLink.href = 'https://cdn.jsdelivr.net/npm/frappe-gantt@0.6.1/dist/frappe-gantt.css'
        document.head.appendChild(cssLink)

        // 加载JS
        const script = document.createElement('script')
        script.src = 'https://cdn.jsdelivr.net/npm/frappe-gantt@0.6.1/dist/frappe-gantt.min.js'
        script.onload = resolve
        script.onerror = reject
        document.head.appendChild(script)
      })
    }

    loadScript().then(() => {
      createGantt()
    }).catch(err => {
      console.error('加载frappe-gantt失败:', err)
    })
  } else {
    createGantt()
  }
}

const createGantt = () => {
  if (!window.Gantt || !ganttSvg.value) return

  ganttInstance = new window.Gantt(ganttSvg.value, frappeTasks.value, {
    view_mode: viewMode.value,
    language: 'zh',
    on_click: (task) => {
      const originalTask = props.tasks.find(t => t.id === task.id)
      if (originalTask) {
        emit('task-click', originalTask)
      }
    },
    on_date_change: (task, start, end) => {
      emit('task-updated', {
        id: task.id,
        start,
        end
      })
    },
    on_progress_change: (task, progress) => {
      emit('task-updated', {
        id: task.id,
        progress
      })
    }
  })
}

const refreshGantt = () => {
  if (ganttInstance) {
    ganttInstance.refresh(frappeTasks.value)
  }
}

const changeViewMode = (mode) => {
  viewMode.value = mode
  if (ganttInstance) {
    ganttInstance.change_view_mode(mode)
  }
}

const changeDate = (delta) => {
  const date = new Date(currentDate.value)
  date.setDate(date.getDate() + delta)
  currentDate.value = date.toISOString().split('T')[0]
  emit('date-changed', currentDate.value)
}

const onDateChange = () => {
  emit('date-changed', currentDate.value)
}

const goToToday = () => {
  currentDate.value = new Date().toISOString().split('T')[0]
  emit('date-changed', currentDate.value)
}

onMounted(() => {
  nextTick(() => {
    initGantt()
  })
})

onUnmounted(() => {
  ganttInstance = null
})

watch(() => props.tasks, () => {
  refreshGantt()
}, { deep: true })

watch(() => props.viewMode, (newVal) => {
  if (newVal) {
    changeViewMode(newVal)
  }
})
</script>

<style scoped>
.gantt-frappe-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  background: white;
  border-radius: 8px;
  overflow: hidden;
}

.toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  border-bottom: 1px solid #e5e7eb;
  background: #f9fafb;
  gap: 16px;
}

.date-picker {
  display: flex;
  align-items: center;
  gap: 8px;
}

.nav-btn {
  padding: 6px 10px;
  border: 1px solid #d1d5db;
  background: white;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.nav-btn:hover {
  background: #f3f4f6;
}

.date-picker input[type="date"] {
  padding: 6px 10px;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  font-size: 14px;
}

.view-controls {
  display: flex;
  gap: 8px;
}

.view-btn {
  padding: 8px 12px;
  border: 1px solid #d1d5db;
  background: white;
  border-radius: 4px;
  font-size: 13px;
  cursor: pointer;
  transition: all 0.2s;
}

.view-btn:hover {
  background: #f3f4f6;
}

.view-btn.active {
  background: #4CAF50;
  color: white;
  border-color: #4CAF50;
}

.today-btn button {
  padding: 6px 12px;
  border: 1px solid #d1d5db;
  background: white;
  border-radius: 4px;
  font-size: 13px;
  cursor: pointer;
}

.today-btn button:hover {
  background: #f3f4f6;
}

.gantt-wrapper {
  flex: 1;
  overflow: auto;
  padding: 20px;
}

#ggantt {
  width: 100%;
}

.task-stats {
  display: flex;
  justify-content: center;
  gap: 30px;
  padding: 12px 16px;
  border-top: 1px solid #e5e7eb;
  background: #f9fafb;
}

.stat-item {
  display: flex;
  gap: 8px;
  font-size: 13px;
}

.stat-label {
  color: #6b7280;
}

.stat-value {
  font-weight: 600;
  color: #1f2937;
}

/* Frappe Gantt 样式覆盖 */
:deep(.gantt) {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

:deep(.gantt .bar) {
  fill: #4CAF50;
  stroke: #45a049;
  stroke-width: 1px;
}

:deep(.gantt .bar:hover) {
  fill: #45a049;
}

:deep(.gantt .bar-progress) {
  fill: #81C784;
}

:deep(.gantt .bar-wrapper) {
  cursor: pointer;
}

:deep(.gantt .bar-label) {
  fill: #333;
  font-size: 12px;
}

/* 自定义任务类型样式 */
:deep(.gantt .bar.order-task) {
  fill: #2196F3;
  stroke: #1976D2;
}

:deep(.gantt .bar.order-task:hover) {
  fill: #1976D2;
}

:deep(.gantt .bar.order-task .bar-progress) {
  fill: #64B5F6;
}

:deep(.gantt .bar.maintenance-task) {
  fill: #FF9800;
  stroke: #F57C00;
}

:deep(.gantt .bar.maintenance-task:hover) {
  fill: #F57C00;
}

:deep(.gantt .bar.maintenance-task .bar-progress) {
  fill: #FFB74D;
}

:deep(.gantt .bar.quality-task) {
  fill: #9C27B0;
  stroke: #7B1FA2;
}

:deep(.gantt .bar.quality-task:hover) {
  fill: #7B1FA2;
}

:deep(.gantt .bar.quality-task .bar-progress) {
  fill: #BA68C8;
}

:deep(.gantt .grid-line) {
  stroke: #e0e0e0;
}

:deep(.gantt .arrow) {
  stroke: #999;
  stroke-width: 1px;
}
</style>
