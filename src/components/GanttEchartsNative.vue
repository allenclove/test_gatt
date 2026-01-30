<template>
  <div class="gantt-echarts-container">
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
        <span>原生 ECharts</span>
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

    <!-- ECharts 甘特图主体 -->
    <div class="gantt-body">
      <div ref="chartRef" class="echarts-chart" :style="{ height: Math.max(zones.length * 100 + 50, 300) + 'px' }"></div>
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
import { ref, computed, watch, onMounted, onUnmounted, nextTick } from 'vue'
import * as echarts from 'echarts/core'
import { CanvasRenderer } from 'echarts/renderers'
import { ScatterChart } from 'echarts/charts'
import { GridComponent, TooltipComponent } from 'echarts/components'

echarts.use([CanvasRenderer, ScatterChart, GridComponent, TooltipComponent])

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
const chartRef = ref(null)
let chartInstance = null

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
const tooltipPinned = ref(false)
const taskBarHovered = ref(false)
let hideTooltipTimer = null

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
  nextTick(() => {
    updateChart()
  })
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

// 任务类型颜色映射
const taskTypeColors = {
  'order-task': '#2196F3',
  'maintenance-task': '#FF9800',
  'quality-task': '#9C27B0'
}

// 初始化图表
const initChart = () => {
  if (!chartRef.value) return

  try {
    chartInstance = echarts.init(chartRef.value, null)
    updateChart()
  } catch (error) {
    console.error('Failed to initialize ECharts:', error)
  }
}

// 更新图表 - 使用简化的散点图方案
const updateChart = () => {
  if (!chartInstance || !zones.value.length) return

  try {
    const mode = viewModes.find(m => m.key === internalViewMode.value) || viewModes[1]
    const slotsPerDay = 1440 / mode.slotMinutes

    // 生成Y轴数据（区域）
    const yAxisData = zones.value.map(z => z.name)

    // 为每个区域创建数据系列
    const series = zones.value.map((zone, zoneIdx) => {
      // 获取该区域的任务
      const zoneTasks = visibleTasks.value.filter(task => {
        const idx = zones.value.findIndex(z => z.id === task.zoneId)
        return idx === zoneIdx
      })

      // 计算每个任务的Y位置（区域内分层避免重叠）
      const data = zoneTasks.map((task, taskIdx) => {
        const startMinutes = parseInt(task.start.split('T')[1]?.substring(0, 2) || '0') * 60 + parseInt(task.start.split('T')[1]?.substring(3, 5) || '0')
        const endMinutes = parseInt(task.end.split('T')[1]?.substring(0, 2) || '0') * 60 + parseInt(task.end.split('T')[1]?.substring(3, 5) || '0')
        const duration = endMinutes - startMinutes

        // Y位置计算：每个区域占用100像素，区域内任务分层显示
        const zoneBaseY = zoneIdx * 100
        const rowHeight = 20
        const row = Math.floor(taskIdx / 4)
        const col = taskIdx % 4
        const y = zoneBaseY + 20 + row * rowHeight

        return {
          name: task.name,
          value: [startMinutes, y],
          task: task,
          itemStyle: { color: taskTypeColors[task.customClass] || '#2196F3' },
          orderNo: task.orderNo,
          duration: duration,
          symbolSize: [
            Math.max(duration * (columnWidth / mode.slotMinutes), 30), // 宽度
            rowHeight - 2  // 高度
          ]
        }
      })

      return {
        type: 'scatter',
        name: zone.name,
        coordinateSystem: 'cartesian2d',
        data: data,
        symbol: 'rect',
        symbolSize: (val, params) => {
          return params.data.symbolSize
        },
        label: {
          show: true,
          position: 'inside',
          formatter: (params) => {
            return params.data.orderNo || ''
          },
          fontSize: 10,
          color: 'white',
          fontWeight: 'bold'
        },
        itemStyle: {
          opacity: 0.9
        },
        emphasis: {
          itemStyle: {
            opacity: 1,
            shadowBlur: 10,
            shadowColor: 'rgba(0,0,0,0.3)'
          }
        },
        z: 10
      }
    })

    const option = {
      grid: {
        left: 120,
        right: 20,
        top: headerHeight,
        bottom: 20
      },
      xAxis: {
        type: 'value',
        min: 0,
        max: 1440,
        axisLabel: {
          formatter: (value) => {
            const hours = Math.floor(value / 60)
            const mins = value % 60
            return `${hours.toString().padStart(2, '0')}:${mins.toString().padStart(2, '0')}`
          }
        },
        axisLine: { show: true, lineStyle: { color: '#e5e7eb' } },
        axisTick: { show: false },
        splitLine: { show: true, lineStyle: { color: '#f3f4f6' } }
      },
      yAxis: {
        type: 'value',
        min: 0,
        max: zones.value.length * 100,
        axisLabel: {
          fontSize: 11,
          color: '#4b5563',
          formatter: (value) => {
            const zoneIdx = Math.floor(value / 100)
            if (zoneIdx >= 0 && zoneIdx < zones.value.length) {
              return zones.value[zoneIdx].name
            }
            return ''
          }
        },
        axisLine: { show: true, lineStyle: { color: '#e5e7eb' } },
        axisTick: { show: false },
        splitLine: {
          show: true,
          lineStyle: { color: '#e5e7eb' }
        },
        interval: 100
      },
      tooltip: {
        show: false
      },
      series: series
    }

    chartInstance.setOption(option, true)

    // 添加事件监听
    chartInstance.off('mouseover')
    chartInstance.off('click')

    chartInstance.on('mouseover', (params) => {
      if (params.componentType === 'series') {
        const task = params.data.task
        showTooltip({
          clientX: params.event.event.clientX,
          clientY: params.event.event.clientY,
          currentTarget: chartRef.value
        }, task)
      }
    })

    chartInstance.on('click', (params) => {
      if (params.componentType === 'series') {
        const task = params.data.task
        handleTaskClick({
          currentTarget: chartRef.value,
          stopPropagation: () => {},
          event: { event: { clientX: params.event.event.clientX, clientY: params.event.event.clientY } }
        }, task)
      }
    })

  } catch (error) {
    console.error('Failed to update chart:', error)
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

  const x = event.clientX || (event.event?.event?.clientX || 0)
  const y = event.clientY || (event.event?.event?.clientY || 0)

  tooltip.value = {
    visible: true,
    x: x - 5,
    y: y,
    task
  }
}

// 处理任务条点击
const handleTaskClick = (event, task) => {
  event.stopPropagation()

  const x = event.clientX || (event.event?.event?.clientX || 0)
  const y = event.clientY || (event.event?.event?.clientY || 0)

  tooltip.value = {
    visible: true,
    x: x - 5,
    y: y,
    task
  }
  tooltipPinned.value = true

  emit('task-click', task)
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
  }, 300)
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
  }, 300)
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
  nextTick(() => {
    initChart()
  })
  document.addEventListener('click', handleGlobalClick)
  window.addEventListener('resize', () => {
    if (chartInstance) {
      chartInstance.resize()
    }
  })
})

onUnmounted(() => {
  if (chartInstance) {
    chartInstance.dispose()
    chartInstance = null
  }
  document.removeEventListener('click', handleGlobalClick)
  if (hideTooltipTimer) {
    clearTimeout(hideTooltipTimer)
  }
})

watch(() => props.viewMode, (newVal) => {
  internalViewMode.value = newVal
  nextTick(() => {
    updateChart()
  })
})

watch(internalViewMode, () => {
  nextTick(() => {
    updateChart()
  })
})

watch([() => props.currentDate, () => props.filterTypes, () => props.filterStatuses, zones], () => {
  nextTick(() => {
    updateChart()
  })
})
</script>

<style scoped>
.gantt-echarts-container {
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

/* 技术标签 */
.tech-badge {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  color: white;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 500;
  box-shadow: 0 2px 4px rgba(16, 185, 129, 0.3);
}

/* 甘特图主体 */
.gantt-body {
  flex: 1;
  overflow: auto;
}

.echarts-chart {
  min-width: 100%;
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
  margin-left: -15px;
  margin-top: -10px;
  margin-bottom: -10px;
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
