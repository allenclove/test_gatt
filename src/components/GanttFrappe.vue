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
        <input type="date" :value="props.currentDate" @change="(e) => emit('date-changed', e.target.value)">
        <button @click="changeDate(1)" class="nav-btn">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polyline points="9 18 15 12 9 6"></polyline>
          </svg>
        </button>
      </div>

      <!-- 技术方案标签 -->
      <div class="tech-badge">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M12 2L2 7l10 5 10-5-10-5 2 7z"></path>
        </svg>
        <span>Frappe Gantt库</span>
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

    <!-- 甘特图主体 -->
    <div class="gantt-body">
      <!-- 左侧区域列表 -->
      <div class="zones-panel">
        <div class="zone-header">加工区域</div>
        <div
          v-for="zone in zones"
          :key="zone.id"
          class="zone-row"
          :style="{ height: rowHeight + 'px' }"
        >
          <div class="zone-name">{{ zone.name }}</div>
        </div>
      </div>

      <!-- 右侧任务条区域 -->
      <div class="tasks-panel">
        <!-- 时间刻度 -->
        <div class="time-header" :style="{ height: headerHeight + 'px' }">
          <div
            v-for="(time, index) in timeSlots"
            :key="index"
            class="time-slot"
            :style="{ width: columnWidth + 'px' }"
          >
            {{ time.label }}
          </div>
        </div>

        <!-- 任务网格 -->
        <div class="tasks-grid" :style="{ height: (zones.length * rowHeight) + 'px' }">
          <!-- 背景网格 -->
          <div class="grid-background">
            <div
              v-for="(zone, zoneIndex) in zones"
              :key="zone.id"
              class="grid-row"
              :style="{
                top: (zoneIndex * rowHeight) + 'px',
                height: rowHeight + 'px',
                width: (timeSlots.length * columnWidth) + 'px'
              }"
            >
              <div
                v-for="(time, timeIndex) in timeSlots"
                :key="timeIndex"
                class="grid-cell"
                :style="{
                  left: (timeIndex * columnWidth) + 'px',
                  width: columnWidth + 'px',
                  height: rowHeight + 'px'
                }"
              ></div>
            </div>
          </div>

          <!-- 当前时间线 -->
          <div
            v-if="currentTimePosition > 0"
            class="current-time-line"
            :style="{
              left: currentTimePosition + 'px',
              height: (zones.length * rowHeight) + 'px'
            }"
          ></div>

          <!-- 任务条 -->
          <div
            v-for="task in visibleTasks"
            :key="task.id"
            class="task-bar"
            :class="task.customClass"
            :style="getTaskStyle(task)"
            @mouseenter="showTooltip($event, task)"
            @mouseleave="hideTooltip"
            @click="$emit('task-click', task)"
          >
            <div class="task-content">
              <span class="task-order-no">{{ task.orderNo || 'N/A' }}</span>
              <!-- 订单类型标识 -->
              <span v-if="task.orderType && task.orderType !== 'normal'" class="order-type-badge" :style="{ background: orderTypeMap[task.orderType]?.color }">
                {{ orderTypeMap[task.orderType]?.label }}
              </span>
            </div>
            <!-- 订单状态指示点 -->
            <div v-if="task.orderStatus || task.progress > 0" class="status-indicator" :style="{ background: getStatusColor(task) }" :title="getStatusTitle(task)"></div>
            <div class="task-progress" :style="{ width: task.progress + '%' }"></div>
          </div>
        </div>
      </div>
    </div>

    <!-- Tooltip -->
    <div
      v-if="tooltip.visible"
      class="task-tooltip"
      :style="{
        left: tooltip.x + 'px',
        top: tooltip.y + 'px'
      }"
      @mouseenter="tooltipHovered = true"
      @mouseleave="handleTooltipMouseLeave"
    >
      <div class="tooltip-header">{{ tooltip.task?.name }}</div>
      <div class="tooltip-body">
        <div class="tooltip-row">
          <span class="tooltip-label">单号:</span>
          <span class="tooltip-value">{{ tooltip.task?.orderNo }}</span>
        </div>
        <div class="tooltip-row">
          <span class="tooltip-label">产品:</span>
          <span class="tooltip-value">{{ tooltip.task?.productName }}</span>
        </div>
        <div class="tooltip-row">
          <span class="tooltip-label">时间:</span>
          <span class="tooltip-value">{{ getTaskTimeRange(tooltip.task) }}</span>
        </div>
        <div class="tooltip-row">
          <span class="tooltip-label">进度:</span>
          <span class="tooltip-value">{{ tooltip.task?.progress }}%</span>
        </div>
        <div class="tooltip-row">
          <span class="tooltip-label">状态:</span>
          <span class="tooltip-value" :class="getStatusClass(tooltip.task?.progress)">
            {{ getStatusText(tooltip.task?.progress) }}
          </span>
        </div>
        <!-- 自定义功能按钮 -->
        <div class="tooltip-actions">
          <button class="tooltip-btn tooltip-btn-primary" @click.stop="handleTooltipAction('edit', tooltip.task)">
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path>
              <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path>
            </svg>
            编辑
          </button>
          <button class="tooltip-btn tooltip-btn-danger" @click.stop="handleTooltipAction('delete', tooltip.task)">
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="3 6 5 6 21 6"></polyline>
              <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path>
            </svg>
            删除
          </button>
          <button class="tooltip-btn tooltip-btn-success" @click.stop="handleTooltipAction('detail', tooltip.task)">
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
              <polyline points="14 2 14 8 20 8"></polyline>
              <line x1="16" y1="13" x2="8" y2="13"></line>
              <line x1="16" y1="17" x2="8" y2="17"></line>
              <polyline points="10 9 9 9 8 9"></polyline>
            </svg>
            详情
          </button>
        </div>
      </div>
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
    default: 'hour'
  },
  currentDate: {
    type: String,
    default: () => new Date().toISOString().split('T')[0]
  },
  filterTypes: {
    type: Array,
    default: () => ['normal', 'urgent', 'expedited']
  },
  filterStatuses: {
    type: Array,
    default: () => ['pending', 'in-production', 'completed', 'delayed']
  }
})

const emit = defineEmits(['task-click', 'task-updated', 'date-changed', 'task-action'])

const viewModes = [
  { key: 'minute', label: '30分钟', slotMinutes: 30 },
  { key: 'hour', label: '1小时', slotMinutes: 60 },
  { key: 'day', label: '4小时', slotMinutes: 240 }
]

const viewMode = ref('hour')
const zones = ref([])

const rowHeight = 50
const headerHeight = 50
const columnWidth = 60

// Tooltip状态
const tooltip = ref({
  visible: false,
  x: 0,
  y: 0,
  task: null
})
const tooltipHovered = ref(false)
let hideTooltipTimer = null

// 从父组件获取zones
watch(() => props.tasks, () => {
  if (props.tasks.length > 0) {
    const zoneSet = new Set(props.tasks.map(t => t.zoneId).filter(Boolean))
    zones.value = Array.from(zoneSet).map((id, index) => ({
      id,
      name: `加工区域 ${id.replace('zone-', '').toUpperCase()}`
    }))
  }
}, { immediate: true })

// 计算时间槽
const timeSlots = computed(() => {
  const mode = viewModes.find(m => m.key === viewMode.value) || viewModes[1]
  const slotsPerDay = 1440 / mode.slotMinutes
  const slots = []

  for (let i = 0; i < slotsPerDay; i++) {
    const minutes = i * mode.slotMinutes
    const hours = Math.floor(minutes / 60)
    const mins = minutes % 60

    let label = ''
    if (viewMode.value === 'minute') {
      label = `${hours}:${mins.toString().padStart(2, '0')}`
    } else if (viewMode.value === 'hour') {
      label = `${hours}:00`
    } else {
      const endHour = hours + 4
      label = `${hours}:00-${endHour}:00`
    }

    slots.push({
      index: i,
      label,
      startMinutes: minutes,
      endMinutes: minutes + mode.slotMinutes
    })
  }

  return slots
})

// 订单类型映射
const orderTypeMap = {
  'normal': { label: '普通', color: '#3B82F6' },
  'urgent': { label: '加急', color: '#EF4444' },
  'expedited': { label: '特急', color: '#DC2626' }
}

// 订单状态映射
const orderStatusMap = {
  'pending': { label: '待开始', color: '#9CA3AF' },
  'in-production': { label: '生产中', color: '#3B82F6' },
  'completed': { label: '已完成', color: '#10B981' },
  'delayed': { label: '已延期', color: '#EF4444' }
}

// 计算可见任务
const visibleTasks = computed(() => {
  return props.tasks.filter(task => {
    const taskDate = task.start.split('T')[0] || task.start
    if (taskDate !== props.currentDate) return false
    if (!task.zoneId) return false

    // 订单类型筛选
    if (task.orderType && !props.filterTypes.includes(task.orderType)) return false

    // 订单状态筛选 - 根据进度判断状态
    let taskStatus = 'pending'
    if (task.progress > 0 && task.progress < 100) {
      taskStatus = 'in-production'
    } else if (task.progress === 100) {
      taskStatus = 'completed'
    }

    // 如果有orderStatus字段，使用它，否则根据进度推断
    const effectiveStatus = task.orderStatus || taskStatus
    if (!props.filterStatuses.includes(effectiveStatus)) return false

    return true
  })
})

const inProgressCount = computed(() => props.tasks.filter(t => t.progress > 0 && t.progress < 100).length)
const completedCount = computed(() => props.tasks.filter(t => t.progress === 100).length)

// 计算当前时间线位置
const currentTimePosition = ref(0)

const updateCurrentTimeLine = () => {
  const now = new Date()
  const today = now.toISOString().split('T')[0]

  if (today === props.currentDate) {
    const mode = viewModes.find(m => m.key === viewMode.value) || viewModes[1]
    const currentMinutes = now.getHours() * 60 + now.getMinutes()
    const slotIndex = Math.floor(currentMinutes / mode.slotMinutes)
    const positionInSlot = (currentMinutes % mode.slotMinutes) / mode.slotMinutes
    currentTimePosition.value = (slotIndex + positionInSlot) * columnWidth
  } else {
    currentTimePosition.value = 0
  }
}

// 获取任务样式
const getTaskStyle = (task) => {
  const mode = viewModes.find(m => m.key === viewMode.value) || viewModes[1]

  let taskStartDate, taskStartTime
  if (task.start.includes('T')) {
    [taskStartDate, taskStartTime] = task.start.split('T')
  } else {
    taskStartDate = task.start
    taskStartTime = '00:00'
  }

  if (taskStartDate !== props.currentDate) {
    return { display: 'none' }
  }

  const [startHour, startMin] = taskStartTime.split(':').map(Number)
  const startMinutes = startHour * 60 + startMin

  let endMinutes
  if (task.end && task.end.includes('T')) {
    const [, endTime] = task.end.split('T')
    const [endHour, endMin] = endTime.split(':').map(Number)
    endMinutes = endHour * 60 + endMin
  } else if (task.duration) {
    endMinutes = startMinutes + task.duration
  } else {
    endMinutes = startMinutes + 60
  }

  const duration = endMinutes - startMinutes
  const left = (startMinutes / mode.slotMinutes) * columnWidth
  const width = (duration / mode.slotMinutes) * columnWidth

  const zoneIndex = zones.value.findIndex(z => z.id === task.zoneId)
  const top = zoneIndex >= 0 ? zoneIndex * rowHeight + 6 : 0

  return {
    left: left + 'px',
    top: top + 'px',
    width: Math.max(width, 40) + 'px',
    height: (rowHeight - 12) + 'px'
  }
}

const getStatusColor = (task) => {
  // 如果有orderStatus字段，使用它
  if (task.orderStatus) {
    return orderStatusMap[task.orderStatus]?.color || '#9CA3AF'
  }
  // 否则根据进度推断
  if (task.progress === 0) return orderStatusMap['pending'].color
  if (task.progress === 100) return orderStatusMap['completed'].color
  return orderStatusMap['in-production'].color
}

const getStatusTitle = (task) => {
  if (task.orderStatus) {
    return orderStatusMap[task.orderStatus]?.label || '未知状态'
  }
  if (task.progress === 0) return orderStatusMap['pending'].label
  if (task.progress === 100) return orderStatusMap['completed'].label
  return orderStatusMap['in-production'].label
}

const getTaskTimeRange = (task) => {
  if (!task) return ''
  const startTime = task.start.split('T')[1]?.substring(0, 5) || ''
  const endTime = task.end.split('T')[1]?.substring(0, 5) || ''
  return `${startTime} - ${endTime}`
}

const getStatusText = (progress) => {
  if (progress === 0) return '待开始'
  if (progress === 100) return '已完成'
  return '进行中'
}

const getStatusClass = (progress) => {
  if (progress === 0) return 'status-pending'
  if (progress === 100) return 'status-completed'
  return 'status-progress'
}

// 显示tooltip
const showTooltip = (event, task) => {
  const rect = event.target.getBoundingClientRect()
  tooltip.value = {
    visible: true,
    x: rect.right + 10,
    y: rect.top,
    task
  }
}

// 隐藏tooltip（延迟执行，允许鼠标移动到tooltip上）
const hideTooltip = () => {
  // 清除之前的定时器
  if (hideTooltipTimer) {
    clearTimeout(hideTooltipTimer)
  }
  // 延迟100ms隐藏，给用户时间移动到tooltip上
  hideTooltipTimer = setTimeout(() => {
    if (!tooltipHovered.value) {
      tooltip.value.visible = false
    }
  }, 100)
}

// tooltip鼠标离开处理
const handleTooltipMouseLeave = () => {
  tooltipHovered.value = false
  tooltip.value.visible = false
  if (hideTooltipTimer) {
    clearTimeout(hideTooltipTimer)
  }
}

// 处理tooltip中的操作按钮点击
const handleTooltipAction = (action, task) => {
  hideTooltip()
  emit('task-action', { action, task })
}

const changeViewMode = (mode) => {
  viewMode.value = mode
}

const changeDate = (delta) => {
  const date = new Date(props.currentDate)
  date.setDate(date.getDate() + delta)
  emit('date-changed', date.toISOString().split('T')[0])
}

const goToToday = () => {
  emit('date-changed', new Date().toISOString().split('T')[0])
}

let timeUpdateInterval = null

onMounted(() => {
  updateCurrentTimeLine()
  timeUpdateInterval = setInterval(updateCurrentTimeLine, 60000)
})

onUnmounted(() => {
  if (timeUpdateInterval) {
    clearInterval(timeUpdateInterval)
  }
})

watch(() => props.viewMode, (newVal) => {
  viewMode.value = newVal
})

watch([viewMode, () => props.currentDate], () => {
  nextTick(() => {
    updateCurrentTimeLine()
  })
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
  padding: 10px 16px;
  border-bottom: 1px solid #e5e7eb;
  background: #f9fafb;
  gap: 16px;
}

.date-picker {
  display: flex;
  align-items: center;
  gap: 6px;
}

.nav-btn {
  padding: 5px 8px;
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
  padding: 5px 8px;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  font-size: 13px;
}

.view-controls {
  display: flex;
  gap: 6px;
}

.view-btn {
  padding: 5px 10px;
  border: 1px solid #d1d5db;
  background: white;
  border-radius: 4px;
  font-size: 12px;
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
  padding: 5px 10px;
  border: 1px solid #d1d5db;
  background: white;
  border-radius: 4px;
  font-size: 12px;
  cursor: pointer;
}

.today-btn button:hover {
  background: #f3f4f6;
}

.gantt-body {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.zones-panel {
  width: 120px;
  flex-shrink: 0;
  border-right: 2px solid #e5e7eb;
  background: #f9fafb;
  overflow-y: auto;
}

.zone-header {
  height: 50px;
  padding: 10px 8px;
  font-weight: 600;
  font-size: 12px;
  color: #374151;
  border-bottom: 1px solid #e5e7eb;
  background: #f3f4f6;
  position: sticky;
  top: 0;
  z-index: 5;
}

.zone-row {
  border-bottom: 1px solid #e5e7eb;
  display: flex;
  align-items: center;
}

.zone-name {
  padding: 0 8px;
  font-size: 11px;
  color: #4b5563;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.tasks-panel {
  flex: 1;
  overflow: auto;
}

.time-header {
  display: flex;
  border-bottom: 1px solid #e5e7eb;
  background: #f3f4f6;
  position: sticky;
  top: 0;
  z-index: 10;
}

.time-slot {
  flex-shrink: 0;
  padding: 10px 4px;
  text-align: center;
  font-size: 11px;
  color: #6b7280;
  border-right: 1px solid #e5e7eb;
  font-weight: 500;
}

.tasks-grid {
  position: relative;
}

.grid-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.grid-row {
  position: absolute;
  left: 0;
  border-bottom: 1px solid #e5e7eb;
}

.grid-cell {
  position: absolute;
  top: 0;
  border-right: 1px solid #f3f4f6;
}

.current-time-line {
  position: absolute;
  top: 0;
  width: 2px;
  background: #ef4444;
  z-index: 20;
  pointer-events: none;
}

.current-time-line::before {
  content: '';
  position: absolute;
  top: -4px;
  left: -4px;
  width: 10px;
  height: 10px;
  background: #ef4444;
  border-radius: 50%;
}

.task-bar {
  position: absolute;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.15s;
  overflow: hidden;
  background: #2196F3;
  box-shadow: 0 1px 2px rgba(0,0,0,0.1);
}

.task-bar:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
  filter: brightness(1.1);
  z-index: 100;
}

.task-content {
  padding: 4px 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}

.task-order-no {
  font-size: 10px;
  font-weight: 600;
  color: white;
  white-space: nowrap;
}

.task-progress {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 2px;
  background: rgba(255,255,255,0.6);
  border-radius: 0 0 4px 4px;
}

/* 订单类型标识 */
.order-type-badge {
  padding: 1px 4px;
  border-radius: 3px;
  font-size: 9px;
  font-weight: 600;
  color: white;
  margin-left: 4px;
  text-transform: uppercase;
}

/* 订单状态指示点 */
.status-indicator {
  position: absolute;
  top: 4px;
  right: 4px;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  border: 1px solid rgba(255,255,255,0.5);
  box-shadow: 0 1px 2px rgba(0,0,0,0.2);
}

/* 技术标签 */
.tech-badge {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  background: linear-gradient(135deg, #f97316 0%, #ea580c 100%);
  color: white;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 500;
  box-shadow: 0 2px 4px rgba(249, 115, 22, 0.3);
}

.task-bar.order-task {
  background: linear-gradient(135deg, #2196F3 0%, #1976D2 100%);
}

.task-bar.maintenance-task {
  background: linear-gradient(135deg, #FF9800 0%, #F57C00 100%);
}

.task-bar.quality-task {
  background: linear-gradient(135deg, #9C27B0 0%, #7B1FA2 100%);
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

.tasks-panel::-webkit-scrollbar,
.zones-panel::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

.tasks-panel::-webkit-scrollbar-track,
.zones-panel::-webkit-scrollbar-track {
  background: #f3f4f6;
}

.tasks-panel::-webkit-scrollbar-thumb,
.zones-panel::-webkit-scrollbar-thumb {
  background: #d1d5db;
  border-radius: 3px;
}

.tasks-panel::-webkit-scrollbar-thumb:hover,
.zones-panel::-webkit-scrollbar-thumb:hover {
  background: #9ca3af;
}

/* Tooltip */
.task-tooltip {
  position: fixed;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.15);
  padding: 0;
  z-index: 1000;
  min-width: 200px;
  border: 1px solid #e5e7eb;
  animation: tooltipFadeIn 0.15s ease-out;
}

@keyframes tooltipFadeIn {
  from {
    opacity: 0;
    transform: translateY(-4px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.tooltip-header {
  padding: 10px 12px;
  background: #f3f4f6;
  border-bottom: 1px solid #e5e7eb;
  font-size: 13px;
  font-weight: 600;
  color: #1f2937;
  border-radius: 8px 8px 0 0;
}

.tooltip-body {
  padding: 10px 12px;
}

.tooltip-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 6px;
  font-size: 12px;
}

.tooltip-row:last-child {
  margin-bottom: 0;
}

.tooltip-label {
  color: #6b7280;
  margin-right: 16px;
}

.tooltip-value {
  color: #1f2937;
  font-weight: 500;
}

.tooltip-value.status-pending {
  color: #F57C00;
}

.tooltip-value.status-progress {
  color: #1976D2;
}

.tooltip-value.status-completed {
  color: #388E3C;
}

.tooltip-actions {
  display: flex;
  gap: 6px;
  margin-top: 10px;
  padding-top: 10px;
  border-top: 1px solid #e5e7eb;
}

.tooltip-btn {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  padding: 6px 8px;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  font-size: 11px;
  cursor: pointer;
  transition: all 0.2s;
  background: white;
  color: #4b5563;
}

.tooltip-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.tooltip-btn-primary {
  border-color: #3b82f6;
  color: #3b82f6;
}

.tooltip-btn-primary:hover {
  background: #3b82f6;
  color: white;
}

.tooltip-btn-danger {
  border-color: #ef4444;
  color: #ef4444;
}

.tooltip-btn-danger:hover {
  background: #ef4444;
  color: white;
}

.tooltip-btn-success {
  border-color: #10b981;
  color: #10b981;
}

.tooltip-btn-success:hover {
  background: #10b981;
  color: white;
}
</style>
