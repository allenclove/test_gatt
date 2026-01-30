# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Vue 3 production scheduling Gantt chart application (排产管理系统) designed for technology comparison. It provides **four implementation approaches** that can be switched via UI buttons:

1. **GanttCustom.vue** - Fully custom Vue component with zone-based layout
2. **GanttFrappe.vue** - Wrapper around the Frappe Gantt library
3. **GanttEchartsNative.vue** - Native ECharts implementation
4. **GanttVueEcharts.vue** - Vue-ECharts wrapper implementation

## Commands

```bash
npm run dev      # Start development server
npm run build    # Build for production
npm run preview  # Preview production build
```

## Architecture

### Technology Stack
- Vue 3 with Composition API (script setup)
- Vite for build tooling
- Frappe Gantt (optional, CDN-loaded)
- ECharts 6.0
- vue-echarts 8.0
- No TypeScript (plain JavaScript)
- Scoped CSS in components

### Project Structure
```
src/
├── main.js                     # App entry point
├── style.css                   # Global styles
├── App.vue                     # Main app with tech switcher
└── components/
    ├── GanttCustom.vue         # Custom zone-based Gantt
    ├── GanttFrappe.vue         # Frappe Gantt wrapper
    ├── GanttEchartsNative.vue  # Native ECharts Gantt
    └── GanttVueEcharts.vue     # Vue-ECharts wrapper Gantt
```

### Application Layout
- **Header**: Logo, tech switcher buttons (4 options), "New Task" button
- **Left Sidebar**: Task list with filtering by task type
- **Main Area**: Statistics cards and Gantt chart (switches between four implementations)

### Component Comparison

#### GanttCustom.vue (Custom Implementation)

**Architecture Concepts:**
- **Zone-Based Rows**: Rows = Work zones, tasks positioned within rows
- **Time System**: Slot-based (30min/1hr/4hr per slot)
- **Task Positioning**: Absolute positioning via `getTaskStyle()`
  - Horizontal: start time → slot pixels
  - Vertical: zone index × rowHeight
  - Width: duration → slot pixels
- **Zones Extraction**: Dynamic via `watch(() => props.tasks)` inspecting `task.zoneId`

**Configuration:**
```js
rowHeight = 50      // Zone row height in pixels
headerHeight = 50   // Time header height
columnWidth = 60    // Time slot width
```

**View Modes:** `minute` (30min), `hour` (1hr), `day` (4hr)

#### GanttFrappe.vue (Library Implementation)

**Architecture:**
- Wraps Frappe Gantt library (loaded via CDN)
- Converts task data format for frappe compatibility
- Standard Gantt layout: rows = tasks
- Shows dependency arrows between tasks

**View Modes:** `day` (日), `week` (周), `month` (月)

#### GanttEchartsNative.vue (Native ECharts)

**Architecture:**
- Uses ECharts 6.0 with ScatterChart
- Cartesian coordinate system with value-type Y-axis
- Each zone creates a separate series to avoid overlap
- Y-axis labels formatted to show zone names

**Configuration:**
```js
rowHeight = 20       // Task bar height in pixels
zoneBaseY = 100      // Each zone occupies 100 pixels vertically
columnWidth = 60     // Time slot width
```

**Key Implementation Details:**
- Y-axis type: `value` (not category)
- Y-position calculation: `zoneIdx * 100 + 20 + row * rowHeight`
- Symbol: `rect` for bar-like appearance
- SymbolSize: dynamic array `[width, height]` based on duration

**View Modes:** `minute` (30min), `hour` (1hr), `day` (4hr)

#### GanttVueEcharts.vue (Vue-ECharts Wrapper)

**Architecture:**
- Uses vue-echarts component wrapper
- Computed chartOption for reactive updates
- Same scatter chart approach as native version
- Autoresize enabled

**Configuration:** Same as GanttEchartsNative.vue

**View Modes:** `minute` (30min), `hour` (1hr), `day` (4hr)

### Task Data Structure

```js
{
  id: String,
  name: String,           // Display name
  orderNo: String,        // Order number (shown on task bar)
  productName: String,    // Product name
  start: String,          // ISO: "2026-01-28T08:00"
  end: String,            // ISO: "2026-01-28T12:00"
  progress: Number,       // 0-100
  customClass: String,    // "order-task" | "maintenance-task" | "quality-task"
  zoneId: String          // "zone-1", "zone-2", etc. (used by all components)
}
```

### State Management (App.vue)

All state managed in App.vue using Vue refs:
- `tasks`: Array of all tasks
- `ganttType`: Current implementation (`'custom'` | `'frappe'` | `'echarts-native'` | `'vue-echarts'`)
- `zones`: Array of work zones
- `viewMode`: Time granularity
- `taskForm`: Form state for create/edit modal

### Color Coding (All Components)

Task types use different colors:
- `order-task`: Blue (#2196F3)
- `maintenance-task`: Orange (#FF9800)
- `quality-task`: Purple (#9C27B0)

### Tooltip Behavior

All four components support:
- Hover tooltip with task details
- Click to pin tooltip
- Action buttons (Edit, Delete, Detail) in pinned tooltip
- Global click handler to close pinned tooltip

## Important Notes

- **Four Implementations**: App switches between components via `ganttType` state
- **Shared Data**: All components consume the same `tasks` array
- **ECharts Y-Axis**: Uses `value` type, not `category`, with custom formatter for zone labels
- **Frappe CDN**: GanttFrappe loads library from CDN on mount
- **Mock Data**: `generateMockTasks()` creates sample production data
- **Time Format**: ISO 8601 with 'T' separator
- **Order Numbers**: Displayed on task bars via label configuration (ECharts) or direct content (Custom)
- **Tooltip Action Buttons**: All components support edit, delete, and detail actions

## ECharts-Specific Notes

### Scatter Chart for Gantt

The ECharts implementations use ScatterChart with rectangular symbols to simulate Gantt bars:

```js
{
  type: 'scatter',
  symbol: 'rect',
  symbolSize: (val, params) => params.data.symbolSize,  // [width, height]
  label: {
    show: true,
    position: 'inside',
    formatter: (params) => params.data.orderNo || ''
  }
}
```

### Y-Axis Configuration

```js
yAxis: {
  type: 'value',
  min: 0,
  max: zones.length * 100,
  axisLabel: {
    formatter: (value) => {
      const zoneIdx = Math.floor(value / 100)
      return zones[zoneIdx]?.name || ''
    }
  },
  interval: 100
}
```

### Data Point Structure

```js
{
  name: task.name,
  value: [startMinutes, y],  // [x, y] coordinates
  task: task,                 // Full task object for tooltip
  itemStyle: { color: taskColor },
  orderNo: task.orderNo,
  duration: durationMinutes,
  symbolSize: [width, height]
}
```
