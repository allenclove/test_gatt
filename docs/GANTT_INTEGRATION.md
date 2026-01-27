# 甘特图组件接入指南

本文档说明如何将 `GanttChart` 组件集成到你的项目中。

## 快速开始

### 1. 复制组件文件

将以下文件复制到你的项目：

```
your-project/
└── src/
    └── components/
        └── GanttChart.vue
```

### 2. 基本使用

```vue
<template>
  <div>
    <GanttChart
      :tasks="tasks"
      :view-mode="viewMode"
      @task-click="handleTaskClick"
      @task-updated="handleTaskUpdate"
      @date-changed="handleDateChange"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import GanttChart from '@/components/GanttChart.vue'

const viewMode = ref('hour')

const tasks = ref([
  {
    id: '1',
    name: '任务名称',
    start: '2026-01-28T08:00',
    end: '2026-01-28T12:00',
    progress: 50,
    zoneId: 'zone-1',
    customClass: 'order-task'
  }
])

const handleTaskClick = (task) => {
  console.log('点击任务', task)
}

const handleTaskUpdate = (task) => {
  console.log('任务更新', task)
}

const handleDateChange = (date) => {
  console.log('日期变更', date)
}
</script>
```

## Props

| 属性 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `tasks` | `Array` | 是 | - | 任务数据数组 |
| `viewMode` | `String` | 否 | `'hour'` | 时间视图模式 |

### viewMode 可选值

| 值 | 说明 | 每个时间槽 |
|----|------|-----------|
| `minute` | 30分钟视图 | 30分钟 |
| `hour` | 1小时视图 | 60分钟 |
| `day` | 4小时视图 | 240分钟 |

## Events

| 事件 | 参数 | 说明 |
|------|------|------|
| `task-click` | `(task: Object)` | 点击任务条时触发 |
| `task-updated` | `(task: Object)` | 任务数据更新时触发 |
| `date-changed` | `(date: String)` | 日期切换时触发，格式 `YYYY-MM-DD` |

## 数据格式

### 任务对象

```typescript
interface Task {
  id: string              // 唯一标识
  name: string            // 任务名称
  start: string           // 开始时间，ISO格式：YYYY-MM-DDTHH:mm
  end: string             // 结束时间，ISO格式：YYYY-MM-DDTHH:mm
  progress: number        // 进度 0-100
  zoneId: string          // 所属区域ID
  customClass?: string    // 自定义样式类名（可选）
  orderNo?: string        // 单号，显示在任务条上（可选）
  productName?: string    // 产品名称（可选）
  duration?: number       // 持续时间（分钟），可选
}
```

### 时间格式说明

时间使用 ISO 8601 格式：

```js
// 正确格式
start: '2026-01-28T08:00'   // 2026年1月28日 8:00
end: '2026-01-28T12:00'     // 2026年1月28日 12:00

// 错误格式（不支持）
start: '2026-01-28 08:00'   // 缺少 T 分隔符
start: '2026-01-28'         // 缺少时间部分
```

## 区域（Zone）说明

甘特图的每一行代表一个"区域"（Zone），而不是代表任务。任务通过 `zoneId` 关联到对应的区域行。

### 区域ID命名规范

建议使用 `zone-` 前缀加数字的格式：

```js
const zoneId = 'zone-1'   // 第1行
const zoneId = 'zone-2'   // 第2行
// ...
const zoneId = 'zone-12'  // 第12行
```

组件会自动从任务数据中提取所有使用的 `zoneId`，并按顺序生成区域行。

## 样式定制

### 修改尺寸参数

在 `GanttChart.vue` 中修改以下常量：

```js
const rowHeight = 50      // 每行高度（像素）
const headerHeight = 50   // 时间表头高度（像素）
const columnWidth = 60    // 每个时间槽宽度（像素）
```

### 任务条颜色

通过 `customClass` 设置任务类型，对应不同颜色：

```css
/* 在 GanttChart.vue 的 style 标签中修改 */

.task-bar.order-task {
  background: linear-gradient(135deg, #2196F3 0%, #1976D2 100%);
}

.task-bar.maintenance-task {
  background: linear-gradient(135deg, #FF9800 0%, #F57C00 100%);
}

.task-bar.quality-task {
  background: linear-gradient(135deg, #9C27B0 0%, #7B1FA2 100%);
}
```

添加新类型：

```css
/* 1. 添加新样式类 */
.task-bar.custom-type {
  background: linear-gradient(135deg, #4CAF50 0%, #388E3C 100%);
}
```

```js
// 2. 任务数据中使用
{
  customClass: 'custom-type'
}
```

## 完整示例

```vue
<template>
  <div class="page">
    <h1>项目进度管理</h1>

    <div class="controls">
      <button @click="addTask">新增任务</button>
    </div>

    <GanttChart
      :tasks="tasks"
      :view-mode="viewMode"
      @task-click="onTaskClick"
    />

    <!-- 任务详情弹窗 -->
    <div v-if="selectedTask" class="modal">
      <h3>{{ selectedTask.name }}</h3>
      <p>时间: {{ formatTime(selectedTask.start) }} - {{ formatTime(selectedTask.end) }}</p>
      <p>进度: {{ selectedTask.progress }}%</p>
      <button @click="selectedTask = null">关闭</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import GanttChart from '@/components/GanttChart.vue'

const viewMode = ref('hour')
const selectedTask = ref(null)

const tasks = ref([
  {
    id: '1',
    name: '设计阶段',
    start: '2026-01-28T09:00',
    end: '2026-01-28T12:00',
    progress: 100,
    zoneId: 'zone-1',
    customClass: 'order-task'
  },
  {
    id: '2',
    name: '开发阶段',
    start: '2026-01-28T13:00',
    end: '2026-01-28T17:00',
    progress: 60,
    zoneId: 'zone-1',
    customClass: 'order-task'
  },
  {
    id: '3',
    name: '测试阶段',
    start: '2026-01-28T14:00',
    end: '2026-01-28T16:00',
    progress: 0,
    zoneId: 'zone-2',
    customClass: 'quality-task'
  }
])

const onTaskClick = (task) => {
  selectedTask.value = task
}

const formatTime = (isoString) => {
  return isoString.split('T')[1]?.substring(0, 5) || ''
}

const addTask = () => {
  const newId = String(Date.now())
  tasks.value.push({
    id: newId,
    name: '新任务',
    start: '2026-01-28T10:00',
    end: '2026-01-28T11:00',
    progress: 0,
    zoneId: 'zone-1',
    customClass: 'order-task'
  })
}
</script>

<style scoped>
.page {
  padding: 20px;
}

.controls {
  margin-bottom: 20px;
}

.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.2);
}
</style>
```

## 常见问题

### Q: 如何切换日期？

A: 组件内部管理日期状态，通过 `@date-changed` 事件监听日期变化。如需外部控制日期，可修改组件使 `currentDate` 支持 v-model。

### Q: 如何实现拖拽调整任务时间？

A: 当前版本不支持拖拽。如需此功能，可以：
1. 在任务条上添加拖拽事件监听
2. 计算拖拽后的新时间
3. 通过 `@task-updated` 事件更新数据

### Q: 同一区域可以同时有多个任务吗？

A: 可以。每个区域行可以容纳多个任务，只要时间不冲突即可。时间重叠的任务会并排显示。

### Q: 如何隐藏 Tooltip？

A: 在 `GanttChart.vue` 中删除 Tooltip 相关的 template 和 script 代码：
- 删除 `tooltip` 相关的 ref
- 删除 `showTooltip` 和 `hideTooltip` 函数
- 删除模板中的 `.task-tooltip` 元素
- 删除 `@mouseenter` 和 `@mouseleave` 事件绑定
