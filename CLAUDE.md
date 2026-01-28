# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Vue 3 production scheduling Gantt chart application (排产管理系统) designed for technology comparison. It provides **two implementation approaches** that can be switched via UI buttons:

1. **GanttCustom.vue** - Fully custom Vue component with zone-based layout
2. **GanttFrappe.vue** - Wrapper around the Frappe Gantt library

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
- No TypeScript (plain JavaScript)
- Scoped CSS in components

### Project Structure
```
src/
├── main.js               # App entry point
├── style.css             # Global styles
├── App.vue               # Main app with tech switcher
└── components/
    ├── GanttCustom.vue   # Custom zone-based Gantt
    └── GanttFrappe.vue   # Frappe Gantt wrapper
```

### Application Layout
- **Header**: Logo, tech switcher buttons, "New Task" button
- **Left Sidebar**: Task list with filtering by task type
- **Main Area**: Statistics cards and Gantt chart (switches between two implementations)

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
  zoneId: String          // "zone-1", "zone-2", etc. (used by GanttCustom)
}
```

### State Management (App.vue)

All state managed in App.vue using Vue refs:
- `tasks`: Array of all tasks
- `ganttType`: Current implementation (`'custom'` | `'frappe'`)
- `zones`: Array of work zones
- `viewMode`: Time granularity
- `taskForm`: Form state for create/edit modal

### Color Coding (Both Components)

Task types use gradient backgrounds:
- `order-task`: Blue (#2196F3 → #1976D2)
- `maintenance-task`: Orange (#FF9800 → #F57C00)
- `quality-task`: Purple (#9C27B0 → #7B1FA2)

## Important Notes

- **Dual Implementation**: App switches between components via `ganttType` state
- **Shared Data**: Both components consume the same `tasks` array
- **Frappe CDN**: GanttFrappe loads library from CDN on mount
- **Mock Data**: `generateMockTasks()` creates sample production data
- **Time Format**: ISO 8601 with 'T' separator
- **Tooltip**: Custom component has hover tooltip; Frappe shows task info on click
- `taskForm`: Form state for create/edit modal

### Color Coding

Task types use gradient backgrounds defined in GanttChart.vue styles:
- `order-task`: Blue (#2196F3 → #1976D2)
- `maintenance-task`: Orange (#FF9800 → #F57C00)
- `quality-task`: Purple (#9C27B0 → #7B1FA2)

### Key Configuration Values (GanttChart.vue)

```js
rowHeight = 50      // Height of each zone row in pixels
headerHeight = 50   // Height of time header
columnWidth = 60    // Width of each time slot in pixels
```

Adjust these to change the visual density of the chart.

## Important Notes

- **No Dependencies**: The project intentionally avoids external Gantt libraries for customizability
- **Mock Data Generation**: `generateMockTasks()` creates sample production data for testing
- **Time Format**: All times use ISO 8601 format with 'T' separator between date and time
- **Tooltip**: Task bars show only order number; hover for full details in tooltip
- **Current Time Line**: Red vertical line updates every 60 seconds when viewing today's date
