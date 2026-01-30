# 甘特图组件接入指南

本文档说明如何将甘特图组件集成到你的项目中。本项目提供**四种实现方案**可供选择。

## 四种实现方案

### 1. GanttCustom.vue - 自定义Vue组件

**推荐场景**：生产环境，需要深度定制

**特点**：
- 纯Vue + DOM实现
- 文件体积小
- 完全可控

```js
import GanttCustom from '@/components/GanttCustom.vue'
```

### 2. GanttFrappe.vue - Frappe Gantt库

**推荐场景**：快速原型，项目管理

**特点**：
- 开箱即用
- 显示任务依赖关系
- 需要CDN加载

```js
import GanttFrappe from '@/components/GanttFrappe.vue'
```

### 3. GanttEchartsNative.vue - 原生ECharts

**推荐场景**：数据可视化，需要丰富图表功能

**特点**：
- 强大的图表能力
- 丰富的交互功能
- 需要ECharts依赖

```js
import GanttEchartsNative from '@/components/GanttEchartsNative.vue'
```

### 4. GanttVueEcharts.vue - Vue-ECharts封装

**推荐场景**：Vue项目，需要响应式图表

**特点**：
- Vue友好
- 自动resize
- 声明式配置

```js
import GanttVueEcharts from '@/components/GanttVueEcharts.vue'
```

## 快速开始

### 1. 复制组件文件

将所需的组件文件复制到你的项目：

```
your-project/
└── src/
    └── components/
        ├── GanttCustom.vue         # 可选
        ├── GanttFrappe.vue         # 可选
        ├── GanttEchartsNative.vue  # 可选
        └── GanttVueEcharts.vue     # 可选
```

### 2. 安装依赖

根据选择的方案安装相应依赖：

```bash
# 必需（所有方案）
npm install vue@^3.4.0

# 方案2：Frappe Gantt
npm install frappe-gantt@^1.0.4

# 方案3和4：ECharts
npm install echarts@^6.0.0 vue-echarts@^8.0.1
```

### 3. 基本使用

```vue
<template>
  <div>
    <GanttCustom
      :tasks="tasks"
      :view-mode="viewMode"
      :current-date="currentDate"
      :filter-types="filterTypes"
      :filter-statuses="filterStatuses"
      @task-click="handleTaskClick"
      @task-updated="handleTaskUpdate"
      @date-changed="handleDateChange"
      @task-action="handleTaskAction"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import GanttCustom from '@/components/GanttCustom.vue'

const viewMode = ref('hour')
const currentDate = ref(new Date().toISOString().split('T')[0])
const filterTypes = ref(['normal', 'urgent', 'expedited'])
const filterStatuses = ref(['pending', 'in-production', 'completed', 'delayed'])

const tasks = ref([
  {
    id: '1',
    name: '任务名称',
    orderNo: 'SO2026001',
    productName: '产品A',
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
  currentDate.value = date
}

const handleTaskAction = ({ action, task }) => {
  console.log('任务操作', action, task)
}
</script>
```

## Props

| 属性 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `tasks` | `Array` | 是 | - | 任务数据数组 |
| `viewMode` | `String` | 否 | `'hour'` | 时间视图模式 |
| `currentDate` | `String` | 否 | 今天 | 当前日期 YYYY-MM-DD |
| `filterTypes` | `Array` | 否 | `['normal', 'urgent', 'expedited']` | 任务类型筛选 |
| `filterStatuses` | `Array` | 否 | 全部 | 任务状态筛选 |

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
| `task-action` | `({ action: String, task: Object })` | Tooltip操作按钮点击时触发 |

### task-action 的 action 类型

| action | 说明 |
|--------|------|
| `edit` | 编辑按钮 |
| `delete` | 删除按钮 |
| `detail` | 详情按钮 |

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
  orderType?: string      // 订单类型（可选）
  orderStatus?: string    // 订单状态（可选）
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

### customClass 可选值

| 值 | 颜色 | 说明 |
|----|------|------|
| `order-task` | 蓝色 | 订单任务 |
| `maintenance-task` | 橙色 | 设备维护 |
| `quality-task` | 紫色 | 质检工序 |

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

## ECharts方案特殊说明

### 配置参数

```js
const rowHeight = 20       // 任务条高度（像素）
const zoneBaseY = 100      // 每个区域占用空间（像素）
const columnWidth = 60     // 每个时间槽宽度（像素）
```

### Y轴说明

ECharts方案使用数值型Y轴，每个区域占用100像素空间：
- 区域1：0-100
- 区域2：100-200
- 区域3：200-300

Y轴标签通过formatter格式化显示区域名称。

## 样式定制

### 修改尺寸参数

**自定义Vue组件 (GanttCustom.vue)**：
```js
const rowHeight = 50      // 每行高度（像素）
const headerHeight = 50   // 时间表头高度（像素）
const columnWidth = 60    // 每个时间槽宽度（像素）
```

**ECharts组件**：
```js
const rowHeight = 20       // 任务条高度（像素）
const zoneBaseY = 100      // 每个区域占用空间（像素）
const columnWidth = 60     // 每个时间槽宽度（像素）
```

### 任务条颜色

通过 `customClass` 设置任务类型，对应不同颜色：

**自定义Vue组件**：
```css
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

**ECharts组件**：
```js
const taskTypeColors = {
  'order-task': '#2196F3',
  'maintenance-task': '#FF9800',
  'quality-task': '#9C27B0'
}
```

添加新类型请修改对应组件的配置。

## 完整示例

```vue
<template>
  <div class="page">
    <h1>项目进度管理</h1>

    <div class="controls">
      <button @click="addTask">新增任务</button>
      <select v-model="viewMode">
        <option value="minute">30分钟</option>
        <option value="hour">1小时</option>
        <option value="day">4小时</option>
      </select>
    </div>

    <GanttCustom
      :tasks="tasks"
      :view-mode="viewMode"
      @task-click="onTaskClick"
      @task-action="onTaskAction"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import GanttCustom from '@/components/GanttCustom.vue'

const viewMode = ref('hour')

const tasks = ref([
  {
    id: '1',
    name: '设计阶段',
    orderNo: 'SO2026001',
    productName: '产品A',
    start: '2026-01-28T09:00',
    end: '2026-01-28T12:00',
    progress: 100,
    zoneId: 'zone-1',
    customClass: 'order-task'
  },
  {
    id: '2',
    name: '开发阶段',
    orderNo: 'SO2026002',
    productName: '产品B',
    start: '2026-01-28T13:00',
    end: '2026-01-28T17:00',
    progress: 60,
    zoneId: 'zone-1',
    customClass: 'order-task'
  },
  {
    id: '3',
    name: '设备维护',
    orderNo: 'MT001',
    productName: '设备C',
    start: '2026-01-28T14:00',
    end: '2026-01-28T16:00',
    progress: 0,
    zoneId: 'zone-2',
    customClass: 'maintenance-task'
  }
])

const onTaskClick = (task) => {
  console.log('点击任务', task)
}

const onTaskAction = ({ action, task }) => {
  switch (action) {
    case 'edit':
      console.log('编辑任务', task)
      break
    case 'delete':
      console.log('删除任务', task)
      break
    case 'detail':
      console.log('查看详情', task)
      break
  }
}

const addTask = () => {
  const newId = String(Date.now())
  tasks.value.push({
    id: newId,
    name: '新任务',
    orderNo: 'SO' + Date.now(),
    productName: '新产品',
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
  display: flex;
  gap: 10px;
}
</style>
```

## 常见问题

### Q: 如何切换日期？

A: 组件内部管理日期状态，通过 `@date-changed` 事件监听日期变化。如需外部控制日期，可传入 `currentDate` prop 并监听变化。

### Q: 如何实现拖拽调整任务时间？

A: 当前版本不支持拖拽。如需此功能，可以：
1. 在任务条上添加拖拽事件监听
2. 计算拖拽后的新时间
3. 通过 `@task-updated` 事件更新数据

### Q: 同一区域可以同时有多个任务吗？

A: 可以。每个区域行可以容纳多个任务，ECharts方案会在垂直方向分层显示避免重叠。

### Q: 如何隐藏 Tooltip？

A: 删除组件模板中的 Tooltip 相关代码：
- 删除 `.task-tooltip` 元素
- 删除 `tooltip` 相关的 ref 和函数
- 删除鼠标事件绑定

### Q: ECharts方案和自定义方案如何选择？

A:
- **ECharts方案**：适合需要数据可视化、图表交互的场景
- **自定义方案**：适合需要深度定制、轻量级的场景
