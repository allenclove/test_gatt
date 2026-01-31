<template>
  <div class="gantt-vue-echarts-container">
    <!-- 时间轴工具栏 -->
    <div class="time-toolbar">
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
        <span>Vue-ECharts 封装</span>
      </div>

      <div class="view-controls">
        <button
          v-for="mode in viewModes"
          :key="mode.key"
          @click="internalViewMode = mode.key"
          :class="['view-btn', { active: internalViewMode === mode.key }]"
        >
          {{ mode.label }}
        </button>
      </div>
      <div class="today-btn">
        <button @click="goToToday">今天</button>
      </div>
    </div>

    <!-- Vue-ECharts 甘特图主体 -->
    <div class="gantt-body">
      <!-- 时间标尺 -->
      <div class="time-ruler" :style="{ width: contentWidth + 'px' }">
        <div class="time-ruler-header">
          <div v-for="(label, index) in timeLabels" :key="index"
               class="time-label"
               :style="{ left: label.position + 'px' }">
            {{ label.text }}
          </div>
        </div>
      </div>
      <v-chart
        ref="chartRef"
        class="echarts-chart"
        :style="{ height: Math.max(zones.length * rowHeight + 20, 300) + 'px', width: contentWidth + 'px' }"
        :option="chartOption"
        :autoresize="true"
        @mouseover="handleChartMouseover"
        @click="handleChartClick"
      />
    </div>

    <!-- Tooltip -->
    <div
      v-if="tooltip.visible"
      class="task-tooltip"
      :class="{ 'tooltip-hovered': !tooltipPinned && tooltip.visible, 'tooltip-pinned-state': tooltipPinned }"
      :style="{
        left: tooltip.x + 'px',
        top: tooltip.y + 'px'
      }"
      @mouseenter="tooltipHovered = true"
      @mouseleave="handleTooltipMouseLeave"
    >
      <div class="tooltip-header" :class="{ 'tooltip-pinned': tooltipPinned }">
        <span>{{ tooltip.task?.name }}</span>
        <button v-if="tooltipPinned" class="tooltip-close-btn" @click.stop="closeTooltip">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>
      </div>
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
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'
import VChart from 'vue-echarts'
import { use } from 'echarts/core'
import { CanvasRenderer } from 'echarts/renderers'
import { CustomChart, ScatterChart, EffectScatterChart } from 'echarts/charts'
import { GridComponent, DataZoomComponent } from 'echarts/components'

use([
  CanvasRenderer,
  CustomChart,
  ScatterChart,
  EffectScatterChart,
  GridComponent,
  DataZoomComponent
])

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

const internalViewMode = ref(props.viewMode || 'hour')
const zones = ref([])
const taskDataMap = ref([])  // 用于存储任务数据的引用
const chartWidth = ref(800)  // 图表宽度，用于计算时间标签位置

const rowHeight = 50
const headerHeight = 50
const baseColumnWidth = 60  // 30分钟模式的列宽
// 根据视图模式动态计算列宽：30分钟=60px, 1小时=120px, 4小时=480px
const columnWidth = computed(() => {
  const mode = viewModes.find(m => m.key === internalViewMode.value) || viewModes[1]
  return baseColumnWidth * (mode.slotMinutes / 30)  // slotMinutes: 30/60/240
})
const taskBarHeight = 38  // 任务条高度

// Tooltip状态
const tooltip = ref({
  visible: false,
  x: 0,
  y: 0,
  task: null
})
const tooltipHovered = ref(false)
const tooltipPinned = ref(false)
const taskBarHovered = ref(false)
let hideTooltipTimer = null

const chartRef = ref(null)

// 任务类型颜色映射
const taskTypeColors = {
  'order-task': { start: '#2196F3', end: '#1976D2' },
  'maintenance-task': { start: '#FF9800', end: '#F57C00' },
  'quality-task': { start: '#9C27B0', end: '#7B1FA2' }
}

// 全局点击处理
const handleGlobalClick = (event) => {
  if (!tooltipPinned.value || !tooltip.value.visible) return
  const target = event.target
  const isTooltip = target.closest('.task-tooltip')
  const isChart = target.closest('.echarts-chart')
  if (isTooltip || isChart) return
  closeTooltip()
}

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

// 计算可见任务
const visibleTasks = computed(() => {
  return props.tasks.filter(task => {
    const taskDate = task.start.split('T')[0] || task.start
    if (taskDate !== props.currentDate) return false
    if (!task.zoneId) return false

    if (task.orderType && !props.filterTypes.includes(task.orderType)) return false

    let taskStatus = 'pending'
    if (task.progress > 0 && task.progress < 100) {
      taskStatus = 'in-production'
    } else if (task.progress === 100) {
      taskStatus = 'completed'
    }

    const effectiveStatus = task.orderStatus || taskStatus
    if (!props.filterStatuses.includes(effectiveStatus)) return false

    return true
  })
})

// Custom系列渲染函数 - 绘制精美的甘特图条
const renderGanttBar = (params, api) => {
  const categoryIndex = api.value(0)  // 区域索引
  const startSlot = api.value(1)      // 开始槽位
  const durationSlots = api.value(2)  // 持续槽位数

  // 通过dataIndex从taskDataMap获取任务数据
  const task = taskDataMap.value[params.dataIndex]
  if (!task) return

  const colors = taskTypeColors[task.customClass] || taskTypeColors['order-task']
  const progress = task.progress || 0

  // 计算布局 - Y坐标使用类别索引（ECharts会自动处理category类型的Y轴）
  const layoutY = categoryIndex
  const y = categoryIndex * rowHeight + (rowHeight - taskBarHeight) / 2
  const height = taskBarHeight

  // 转换为像素坐标（使用槽位坐标，Y值用类别索引）
  const startX = api.coord([startSlot, layoutY])[0]
  const endX = api.coord([startSlot + durationSlots, layoutY])[0]
  const width = Math.max(endX - startX, 20)  // 最小宽度20px

  const x = startX

  // 定义任务条图形
  const barPath = {
    type: 'rect',
    shape: {
      x: x,
      y: y,
      width: width,
      height: height,
      r: 6  // 圆角半径
    },
    style: {
      fill: {
        type: 'linear',
        x: 0, y: 0, x2: 1, y2: 0,
        colorStops: [
          { offset: 0, color: colors.start },
          { offset: 1, color: colors.end }
        ]
      },
      shadowBlur: 2,
      shadowColor: 'rgba(0,0,0,0.15)',
      shadowOffsetY: 1
    },
    styleEmphasis: {
      shadowBlur: 8,
      shadowColor: 'rgba(0,0,0,0.3)',
      shadowOffsetY: 2
    }
  }

  // 进度条（底部细条）
  const progressWidth = width * (progress / 100)
  const progressPath = {
    type: 'rect',
    shape: {
      x: x,
      y: y + height - 4,
      width: progressWidth,
      height: 3,
      r: 0
    },
    style: {
      fill: 'rgba(255,255,255,0.7)'
    },
    silent: true
  }

  // 订单号文字
  const textPath = {
    type: 'text',
    style: {
      text: task.orderNo || 'N/A',
      x: x + width / 2,
      y: y + height / 2,
      textAlign: 'center',
      textVerticalAlign: 'middle',
      fontSize: 11,
      fontWeight: '600',
      fill: '#fff',
      textShadowColor: 'rgba(0,0,0,0.3)',
      textShadowBlur: 2
    },
    silent: true
  }

  // 订单类型标识
  let groupChildren = [barPath, progressPath, textPath]

  if (task.orderType && task.orderType !== 'normal') {
    const orderTypeColors = {
      'urgent': '#EF4444',
      'expedited': '#DC2626'
    }
    const badgeText = task.orderType === 'urgent' ? '急' : '特'
    const badgePath = {
      type: 'rect',
      shape: {
        x: x + width - 20,
        y: y + 4,
        width: 16,
        height: 14,
        r: 3
      },
      style: {
        fill: orderTypeColors[task.orderType] || '#EF4444'
      },
      silent: true
    }
    const badgeTextPath = {
      type: 'text',
      style: {
        text: badgeText,
        x: x + width - 12,
        y: y + 11,
        textAlign: 'center',
        textVerticalAlign: 'middle',
        fontSize: 9,
        fontWeight: 'bold',
        fill: '#fff'
      },
      silent: true
    }
    groupChildren.push(badgePath, badgeTextPath)
  }

  // 状态指示点
  if (task.orderStatus || task.progress > 0) {
    const statusColors = {
      'pending': '#9CA3AF',
      'in-production': '#3B82F6',
      'completed': '#10B981',
      'delayed': '#EF4444'
    }
    let statusColor = '#9CA3AF'
    if (task.orderStatus) {
      statusColor = statusColors[task.orderStatus] || '#9CA3AF'
    } else if (task.progress === 100) {
      statusColor = statusColors['completed']
    } else if (task.progress > 0) {
      statusColor = statusColors['in-production']
    }
    const statusDot = {
      type: 'circle',
      shape: {
        cx: x + width - 8,
        cy: y + 8,
        r: 5
      },
      style: {
        fill: statusColor,
        stroke: 'rgba(255,255,255,0.8)',
        lineWidth: 1
      },
      silent: true
    }
    groupChildren.push(statusDot)
  }

  return {
    type: 'group',
    children: groupChildren
  }
}

// 计算当前时间线位置
const getCurrentTimePosition = () => {
  const now = new Date()
  const today = now.toISOString().split('T')[0]

  if (today !== props.currentDate) return null

  const currentMinutes = now.getHours() * 60 + now.getMinutes()
  return currentMinutes
}

// 计算时间标签
const timeLabels = computed(() => {
  const mode = viewModes.find(m => m.key === internalViewMode.value) || viewModes[1]
  const slotMinutes = mode.slotMinutes
  const slotsPerDay = 1440 / slotMinutes  // 根据模式计算的槽数

  const labels = []

  for (let i = 0; i < slotsPerDay; i++) {
    // 根据视图模式计算实际分钟数
    const minutes = i * slotMinutes
    const hours = Math.floor(minutes / 60)
    const mins = minutes % 60
    let labelText = ''
    if (internalViewMode.value === 'minute') {
      labelText = `${hours}:${mins.toString().padStart(2, '0')}`
    } else if (internalViewMode.value === 'hour') {
      labelText = `${hours}:00`
    } else {
      const endHour = hours + 4
      labelText = `${hours}:00-${endHour}:00`
    }

    // 使用固定的列宽度计算位置
    labels.push({
      text: labelText,
      position: 120 + i * columnWidth.value
    })
  }
  return labels
})

// 计算内容宽度：槽位数 * 列宽 + 左侧区域宽度
const contentWidth = computed(() => {
  const mode = viewModes.find(m => m.key === internalViewMode.value) || viewModes[1]
  const slotsPerDay = 1440 / mode.slotMinutes
  return 120 + slotsPerDay * columnWidth.value + 30  // left margin + content + right margin
})

// ECharts 配置
const chartOption = computed(() => {
  const mode = viewModes.find(m => m.key === internalViewMode.value) || viewModes[1]

  // 根据视图模式设置不同的槽位数量和宽度
  const slotMinutes = mode.slotMinutes
  const slotsPerDay = 1440 / slotMinutes  // 根据模式计算的槽位数

  // 准备数据：使用当前模式的槽位系统
  taskDataMap.value = []
  const data = visibleTasks.value.map((task) => {
    const zoneIndex = zones.value.findIndex(z => z.id === task.zoneId)
    if (zoneIndex === -1) return null

    const startTime = task.start.split('T')[1] || '00:00'
    const [startHour, startMin] = startTime.split(':').map(Number)
    const startMinutes = startHour * 60 + startMin

    const endTime = task.end.split('T')[1] || '01:00'
    const [endHour, endMin] = endTime.split(':').map(Number)
    const endMinutes = endHour * 60 + endMin

    const durationMinutes = Math.max(endMinutes - startMinutes, 30)  // 最小30分钟

    // 使用当前模式的槽位
    const startSlot = startMinutes / slotMinutes
    const durationSlots = durationMinutes / slotMinutes

    // 存储任务引用
    taskDataMap.value.push(task)

    return {
      value: [zoneIndex, startSlot, durationSlots]
    }
  }).filter(d => d !== null)

  // 生成网格线（基于当前模式槽位）
  const gridLines = []
  for (let i = 0; i <= slotsPerDay; i++) {
    gridLines.push([i, 0])
  }

  // 当前时间线（转换为当前模式槽位）
  const currentTimeMinutes = getCurrentTimePosition()
  const currentTimeSlot = currentTimeMinutes !== null ? currentTimeMinutes / slotMinutes : null
  let currentTimeLine = null
  if (currentTimeSlot !== null) {
    currentTimeLine = {
      xAxis: currentTimeSlot,
      label: { show: false },
      lineStyle: { color: '#ef4444', width: 2, type: 'solid' }
    }
  }

  return {
    grid: {
      left: 120,
      right: 30,
      top: 50,
      bottom: 30
    },
    xAxis: {
      type: 'value',
      min: 0,
      max: slotsPerDay,  // 使用当前模式的槽位数量
      axisLabel: { show: false },
      axisLine: { show: false },
      axisTick: { show: false },
      splitLine: { show: false },
      ...(currentTimeLine ? { markLine: { data: [currentTimeLine], symbol: ['none', 'circle'], symbolSize: 8 } } : {})
    },
    yAxis: {
      type: 'category',
      data: zones.value.map(z => z.name),
      axisLine: {
        show: true,
        lineStyle: { color: '#e5e7eb', width: 2 }
      },
      axisTick: { show: false },
      axisLabel: {
        fontSize: 11,
        color: '#4b5563',
        margin: 8
      },
      splitLine: {
        show: true,
        lineStyle: { color: '#e5e7eb' }
      }
    },
    tooltip: {
      show: false
    },
    series: [{
      type: 'custom',
      renderItem: renderGanttBar,
      data: data,
      coordinateSystem: 'cartesian2d',
      encode: {
        x: [1, 2],
        y: 0
      },
      z: 10,
      cursor: 'pointer'
    }, {
      // 垂直网格线
      type: 'effectScatter',
      data: gridLines,
      symbolSize: 0,
      z: 0,
      markLine: {
        silent: true,
        symbol: ['none', 'none'],
        lineStyle: { color: '#f3f4f6', width: 1 },
        label: { show: false },
        data: gridLines.map(([x]) => ({ coord: [x, 0] }))
      }
    }]
  }
})

// 显示tooltip
const showTooltip = (event, task) => {
  if (tooltipPinned.value) {
    return
  }

  if (hideTooltipTimer) {
    clearTimeout(hideTooltipTimer)
    hideTooltipTimer = null
  }

  taskBarHovered.value = true

  const x = event.clientX
  const y = event.clientY

  tooltip.value = {
    visible: true,
    x: x + 10,
    y: y - 10,
    task
  }
}

// 处理任务条点击
const handleTaskClick = (event, task) => {
  const x = event.clientX
  const y = event.clientY

  tooltip.value = {
    visible: true,
    x: x + 10,
    y: y - 10,
    task
  }
  tooltipPinned.value = true

  emit('task-click', task)
}

// 图表鼠标悬停事件
const handleChartMouseover = (params) => {
  if (params.componentType === 'series' && params.seriesType === 'custom') {
    const task = taskDataMap.value[params.dataIndex]
    if (task) {
      showTooltip({
        clientX: params.event.event.clientX,
        clientY: params.event.event.clientY
      }, task)
    }
  }
}

// 图表点击事件
const handleChartClick = (params) => {
  if (params.componentType === 'series' && params.seriesType === 'custom') {
    const task = taskDataMap.value[params.dataIndex]
    if (task) {
      handleTaskClick({
        clientX: params.event.event.clientX,
        clientY: params.event.event.clientY,
        stopPropagation: () => {}
      }, task)
    }
  }
}

// 获取任务时间范围
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

// 隐藏tooltip
const hideTooltip = () => {
  taskBarHovered.value = false

  if (tooltipPinned.value) {
    return
  }

  if (hideTooltipTimer) {
    clearTimeout(hideTooltipTimer)
  }

  hideTooltipTimer = setTimeout(() => {
    if (!tooltipHovered.value && !taskBarHovered.value) {
      tooltip.value.visible = false
    }
  }, 200)
}

// tooltip鼠标离开处理
const handleTooltipMouseLeave = () => {
  tooltipHovered.value = false

  if (tooltipPinned.value) {
    return
  }

  if (hideTooltipTimer) {
    clearTimeout(hideTooltipTimer)
  }

  hideTooltipTimer = setTimeout(() => {
    if (!tooltipHovered.value && !taskBarHovered.value) {
      tooltip.value.visible = false
    }
  }, 200)
}

// 关闭tooltip
const closeTooltip = () => {
  tooltipPinned.value = false
  tooltipHovered.value = false
  taskBarHovered.value = false
  tooltip.value.visible = false
  if (hideTooltipTimer) {
    clearTimeout(hideTooltipTimer)
  }
}

// 处理tooltip中的操作按钮点击
const handleTooltipAction = (action, task) => {
  tooltipPinned.value = false
  tooltipHovered.value = false
  taskBarHovered.value = false
  tooltip.value.visible = false

  emit('task-action', { action, task })
}

const changeDate = (delta) => {
  const date = new Date(props.currentDate)
  date.setDate(date.getDate() + delta)
  emit('date-changed', date.toISOString().split('T')[0])
}

const goToToday = () => {
  emit('date-changed', new Date().toISOString().split('T')[0])
}

onMounted(() => {
  document.addEventListener('click', handleGlobalClick)
})

onUnmounted(() => {
  document.removeEventListener('click', handleGlobalClick)
  if (hideTooltipTimer) {
    clearTimeout(hideTooltipTimer)
  }
})

watch(() => props.viewMode, (newVal) => {
  internalViewMode.value = newVal
})
</script>

<style scoped>
.gantt-vue-echarts-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  background: white;
  border-radius: 8px;
  overflow: hidden;
}

/* 工具栏 */
.time-toolbar {
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
  background: #f59e0b;
  color: white;
  border-color: #f59e0b;
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

/* 技术标签 */
.tech-badge {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);
  color: white;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 500;
  box-shadow: 0 2px 4px rgba(245, 158, 11, 0.3);
}

/* 甘特图主体 */
.gantt-body {
  flex: 1;
  overflow-x: auto;
  overflow-y: hidden;
  display: flex;
  flex-direction: column;
}

/* 时间标尺 */
.time-ruler {
  background: #f3f4f6;
  border-bottom: 1px solid #e5e7eb;
  height: 40px;
  position: relative;
  flex-shrink: 0;
  min-width: fit-content;
}

.echarts-chart {
  min-width: fit-content;
}

.time-ruler-header {
  position: relative;
  height: 100%;
  width: 100%;
}

.time-label {
  position: absolute;
  top: 12px;
  font-size: 11px;
  font-weight: 500;
  color: #6b7280;
  transform: translateX(-50%);
  white-space: nowrap;
  user-select: none;
}

.time-label::after {
  content: '';
  position: absolute;
  left: 50%;
  top: 18px;
  width: 1px;
  height: 8px;
  background: #e5e7eb;
  transform: translateX(-50%);
}

.echarts-chart {
  min-width: fit-content;
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
  padding: 10px 0 10px 15px;
}

.task-tooltip::before {
  content: '';
  position: absolute;
  left: -20px;
  top: 10%;
  width: 20px;
  height: 80%;
  background: transparent;
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

.task-tooltip.tooltip-hovered {
  z-index: 10001;
  box-shadow: 0 6px 24px rgba(0,0,0,0.2);
}

.task-tooltip.tooltip-pinned-state {
  z-index: 10002;
  box-shadow: 0 8px 30px rgba(0,0,0,0.25);
}

.tooltip-header {
  padding: 10px 12px;
  background: #f3f4f6;
  border-bottom: 1px solid #e5e7eb;
  font-size: 13px;
  font-weight: 600;
  color: #1f2937;
  border-radius: 8px 8px 0 0;
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin: -10px 0 0 -15px;
  padding: 10px 12px;
}

.tooltip-header.tooltip-pinned {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.tooltip-close-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  border: none;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  padding: 0;
}

.tooltip-close-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: scale(1.1);
}

.tooltip-body {
  padding: 10px 12px;
  margin: 0 0 -10px -15px;
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
