# 排产甘特图系统

基于 Vue 3 的生产排产甘特图可视化系统，提供**四种实现方案**用于技术选型对比。

## 功能特点

- **四种实现方案**：自定义Vue组件、Frappe Gantt库、原生ECharts、Vue-ECharts封装，支持一键切换对比
- **多区域展示**：支持多个加工区域并行排产
- **时间维度切换**：30分钟/1小时/4小时三种时间刻度
- **任务管理**：创建、编辑、删除任务
- **可视化交互**：任务条悬停显示详情，点击查看
- **实时时间线**：当前时间红线标识（自定义版本）
- **进度追踪**：任务进度可视化展示
- **类型分类**：订单任务、设备维护、质检工序三种类型
- **功能按钮**：Tooltip中包含编辑、删除、详情操作按钮

## 技术方案对比

| 特性 | 自定义Vue组件 | Frappe Gantt库 | 原生 ECharts | Vue-ECharts 封装 |
|------|--------------|----------------|-------------|-----------------|
| **实现方式** | 纯Vue + DOM | SVG + 第三方库 | Canvas + ECharts | Vue组件封装 |
| **时间粒度** | 30分钟/1小时/4小时 | 日/周/月 | 30分钟/1小时/4小时 | 30分钟/1小时/4小时 |
| **区域布局** | 横向区域行，任务定位 | 传统任务行布局 | 散点图模拟甘特图 | 散点图模拟甘特图 |
| **依赖关系** | 不显示（可扩展） | 显示依赖箭头 | 不显示 | 不显示 |
| **自定义性** | 完全可控 | 受限于库API | 高度可定制 | 高度可定制 |
| **文件大小** | 自包含，较小 | 需加载外部库 | 较大 | 较大 |
| **适用场景** | 复杂业务定制 | 快速原型开发 | 数据可视化展示 | Vue项目集成 |

## 技术栈

- Vue 3 (Composition API)
- Vite
- Frappe Gantt (可选方案)
- ECharts 6.0
- vue-echarts 8.0
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
│   │   ├── GanttCustom.vue         # 自定义甘特图组件
│   │   ├── GanttFrappe.vue         # Frappe Gantt库封装
│   │   ├── GanttEchartsNative.vue  # 原生ECharts甘特图
│   │   └── GanttVueEcharts.vue     # Vue-ECharts封装甘特图
│   ├── App.vue                     # 主应用页面（含切换功能）
│   ├── main.js                     # 应用入口
│   └── style.css                   # 全局样式
├── docs/
│   └── GANTT_INTEGRATION.md        # 组件接入文档
├── CLAUDE.md                       # Claude Code 项目说明
└── package.json
```

## 使用说明

### 切换技术方案

页面顶部右侧有切换按钮，可以随时在四种实现方案之间切换：

```
┌─────────────────────────────────────────────────────────────────────┐
│  [自定义Vue组件] [Frappe Gantt库] [原生 ECharts] [Vue-ECharts 封装] │
└─────────────────────────────────────────────────────────────────────┘
```

点击按钮即可切换，数据自动同步。

## 实现方案详解

### 1. 自定义Vue组件 (GanttCustom.vue)

**技术特点**：
- 纯Vue实现，基于DOM渲染
- 绝对定位布局
- 自定义Tooltip交互

**优势**：
- 完全可控，易于定制
- 文件体积小
- 适合复杂业务逻辑

**适用场景**：生产环境，需要深度定制

### 2. Frappe Gantt库 (GanttFrappe.vue)

**技术特点**：
- 基于Frape Gantt库
- SVG渲染
- 显示任务依赖关系

**优势**：
- 开箱即用
- 显示依赖箭头
- 适合项目管理

**适用场景**：快速原型，项目进度展示

### 3. 原生ECharts (GanttEchartsNative.vue)

**技术特点**：
- 使用ECharts 6.0
- ScatterChart模拟甘特图
- 数值型Y轴，自定义刻度

**优势**：
- 强大的图表能力
- 丰富的交互功能
- 可扩展性强

**适用场景**：数据可视化，需要丰富图表功能

### 4. Vue-ECharts封装 (GanttVueEcharts.vue)

**技术特点**：
- vue-echarts组件封装
- 响应式配置
- Vue风格的事件处理

**优势**：
- Vue友好
- 自动resize
- 声明式配置

**适用场景**：Vue项目，需要响应式图表

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

### ECharts配置参数

```js
const rowHeight = 20       // 任务条高度（像素）
const zoneBaseY = 100      // 每个区域占用空间（像素）
const columnWidth = 60     // 每个时间槽宽度（像素）
```

### 自定义Vue组件参数

```js
const rowHeight = 50       // 每行高度（像素）
const headerHeight = 50    // 时间表头高度（像素）
const columnWidth = 60     // 每个时间槽宽度（像素）
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
