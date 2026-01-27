# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Vue 3 production scheduling Gantt chart application (排产管理系统). It displays manufacturing tasks across work zones with time-based scheduling, using a custom-built Gantt chart component rather than external libraries.

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
- No TypeScript (plain JavaScript)
- Scoped CSS in components

### Project Structure
```
src/
├── main.js           # App entry point
├── style.css         # Global styles
├── App.vue           # Main application layout and state management
└── components/
    └── GanttChart.vue # Custom Gantt chart component
```

### Application Layout
The app uses a fixed-layout design:
- **Header**: Logo and "New Task" button
- **Left Sidebar**: Task list with filtering by task type
- **Main Area**: Statistics cards and Gantt chart

### Gantt Chart Component

The GanttChart component is fully custom-built (no external Gantt library). Key architectural concepts:

**Time System**: Uses a slot-based time system where each "slot" represents a configurable duration (30min/1hr/4hr). The `viewModes` array defines these slots.

**Zone-Based Rows**: Unlike traditional Gantt charts where rows = tasks, this chart has:
- Rows = Work zones (加工区域)
- Tasks positioned within zone rows based on start time
- Multiple tasks can occupy the same zone at different times

**Task Positioning**: Tasks are positioned absolutely using `getTaskStyle()`:
- Horizontal position = start time converted to slot-based pixels
- Vertical position = zone index × rowHeight
- Width = duration converted to slot-based pixels

**Data Flow**: Zones are dynamically extracted from tasks via `watch(() => props.tasks)` - the component inspects `task.zoneId` across all tasks to build the zone list.

### Task Data Structure

```js
{
  id: String,
  name: String,           // Display name
  orderNo: String,        // Order number (shown on task bar)
  productName: String,    // Product name
  start: String,          // ISO format: "2026-01-28T08:00"
  end: String,            // ISO format: "2026-01-28T12:00"
  progress: Number,       // 0-100
  customClass: String,    // "order-task" | "maintenance-task" | "quality-task"
  zoneId: String          // "zone-1", "zone-2", etc.
}
```

### State Management (App.vue)

All state is managed in App.vue using Vue refs:
- `tasks`: Array of all tasks
- `zones`: Array of available work zones (for dropdown)
- `viewMode`: Current time granularity (passed to GanttChart)
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
