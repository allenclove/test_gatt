# 排产甘特图系统

基于 Vue 3 的生产排产甘特图可视化系统，提供两种实现方案用于技术选型对比。

## 功能特点

- **双实现方案**：自定义Vue组件 + Frappe Gantt库，支持一键切换对比
- **多区域展示**：支持多个加工区域并行排产
- **时间维度切换**：30分钟/1小时/4小时三种时间刻度
- **任务管理**：创建、编辑、删除任务
- **可视化交互**：任务条悬停显示详情，点击查看
- **实时时间线**：当前时间红线标识
- **进度追踪**：任务进度可视化展示
- **类型分类**：订单任务、设备维护、质检工序三种类型

## 技术方案对比

| 特性 | 自定义Vue组件 | Frappe Gantt库 |
|------|--------------|----------------|
| **实现方式** | 纯Vue + Canvas/DOM | SVG + 第三方库 |
| **时间粒度** | 30分钟/1小时/4小时 | 日/周/月 |
| **区域布局** | 横向区域行，任务定位 | 传统任务行布局 |
| **依赖关系** | 不显示（可扩展） | 显示依赖箭头 |
| **自定义性** | 完全可控 | 受限于库API |
| **文件大小** | 自包含，较小 | 需加载外部库 |
| **适用场景** | 复杂业务定制 | 快速原型开发 |

## 技术栈

- Vue 3 (Composition API)
- Vite
- Frappe Gantt (可选方案)
- 纯 JavaScript (无 TypeScript)

## 快速开始

### 安装依赖

```bash
npm install
```

### 启动开发服务器

```bash
npm run dev
```

访问 `http://localhost:5173/`

### 构建生产版本

```bash
npm run build
```

构建产物在 `dist` 目录

## 项目结构

```
test_gatt/
├── src/
│   ├── components/
│   │   ├── GanttCustom.vue    # 自定义甘特图组件
│   │   └── GanttFrappe.vue    # Frappe Gantt库封装
│   ├── App.vue                # 主应用页面（含切换功能）
│   ├── main.js                # 应用入口
│   └── style.css              # 全局样式
├── docs/
│   └── GANTT_INTEGRATION.md   # 组件接入文档
├── CLAUDE.md                  # Claude Code 项目说明
└── package.json
```

## 使用说明

### 切换技术方案

页面顶部右侧有切换按钮，可以随时在两种实现方案之间切换：

```
┌─────────────────────────────────────────┐
│  [自定义Vue组件] [Frappe Gantt库] [新增任务] │
└─────────────────────────────────────────┘
```

点击按钮即可切换，数据自动同步。

## 组件使用

### 基本用法

```vue
<template>
  <GanttChart
    :tasks="tasks"
    :view-mode="viewMode"
    @task-click="handleTaskClick"
  />
</template>

<script setup>
import GanttChart from '@/components/GanttChart.vue'

const tasks = ref([
  {
    id: '1',
    name: '任务名称',
    orderNo: 'SO2026001',
    start: '2026-01-28T08:00',
    end: '2026-01-28T12:00',
    progress: 50,
    zoneId: 'zone-1',
    customClass: 'order-task'
  }
])
</script>
```

详细接入文档请查看 [docs/GANTT_INTEGRATION.md](docs/GANTT_INTEGRATION.md)

## 数据格式

```typescript
interface Task {
  id: string              // 唯一标识
  name: string            // 任务名称
  orderNo: string         // 单号
  productName: string     // 产品名称
  start: string           // 开始时间 ISO格式: YYYY-MM-DDTHH:mm
  end: string             // 结束时间 ISO格式: YYYY-MM-DDTHH:mm
  progress: number        // 进度 0-100
  zoneId: string          // 所属区域ID (zone-1, zone-2, ...)
  customClass: string     // 类型: order-task | maintenance-task | quality-task
}
```

## 配置说明

### 时间视图模式

| 模式 | 说明 | 每个时间槽 |
|------|------|-----------|
| minute | 30分钟视图 | 30分钟 |
| hour | 1小时视图 | 60分钟 |
| day | 4小时视图 | 240分钟 |

### 样式参数

在 `GanttChart.vue` 中可调整：

```js
const rowHeight = 50      // 每行高度（像素）
const headerHeight = 50   // 时间表头高度（像素）
const columnWidth = 60    // 每个时间槽宽度（像素）
```

## 部署说明

### 生产环境部署

```bash
# 1. 构建
npm run build

# 2. 部署 dist 目录到静态服务器
```

### 内网环境部署

由于内网环境无法访问外网，本项目包含 `node_modules` 依赖。

**方式一：直接运行源码**
```bash
# 复制整个项目目录（含node_modules）到内网
npm run dev
```

**方式二：部署构建产物**
```bash
# 在有网环境构建后，只复制 dist 目录到内网
npm run build
```

## License

MIT
