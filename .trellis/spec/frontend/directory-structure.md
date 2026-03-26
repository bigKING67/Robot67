# Directory Structure

> How frontend code is organized in this project.

---

## Overview

The frontend is a **Vite + React SPA** (`web/` directory) providing Chat UI, Dashboard, and Report Generator for the data analysis Agent.

---

## Directory Layout

```
web/
├── src/
│   ├── pages/                   # Route-level page components
│   │   ├── chat/                    # Chat UI (streaming conversation)
│   │   │   ├── ChatPage.tsx
│   │   │   ├── MessageList.tsx
│   │   │   └── InputBar.tsx
│   │   ├── dashboard/               # Dashboard + chat sidebar
│   │   │   ├── DashboardPage.tsx
│   │   │   ├── MetricGrid.tsx
│   │   │   └── ChatSidebar.tsx
│   │   └── reports/                 # Report generator
│   │       ├── ReportPage.tsx
│   │       ├── TemplateSelector.tsx
│   │       └── ReportViewer.tsx     # HTML report with ECharts
│   │
│   ├── components/              # Reusable components
│   │   ├── charts/                  # ECharts wrapper components
│   │   │   ├── LineChart.tsx
│   │   │   ├── BarChart.tsx
│   │   │   ├── FunnelChart.tsx
│   │   │   ├── HeatMap.tsx
│   │   │   └── ChartRenderer.tsx    # Generic chart from Agent data
│   │   ├── ui/                      # Base UI primitives
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Input.tsx
│   │   │   └── Loading.tsx
│   │   └── shared/                  # Composite components
│   │       ├── MetricCard.tsx       # KPI display card
│   │       ├── DataTable.tsx        # Sortable data table
│   │       ├── ActionList.tsx       # Executable action checklist
│   │       └── MarkdownRenderer.tsx # Render Agent markdown output
│   │
│   ├── hooks/                   # Custom React hooks
│   │   ├── use-agent-chat.ts        # Chat with streaming
│   │   ├── use-websocket.ts         # WebSocket connection
│   │   ├── use-sessions.ts          # Session management
│   │   ├── use-dashboard-data.ts    # Dashboard metrics
│   │   └── use-report.ts            # Report generation
│   │
│   ├── lib/                     # Utilities
│   │   ├── api-client.ts            # REST API client
│   │   ├── ws-client.ts             # WebSocket client
│   │   ├── chart-config.ts          # ECharts theme and defaults
│   │   └── report-template.ts       # HTML report template engine
│   │
│   ├── types/                   # TypeScript types
│   │   ├── index.ts
│   │   ├── agent.ts                 # Agent response types
│   │   ├── chart.ts                 # Chart data types
│   │   └── report.ts               # Report types
│   │
│   ├── App.tsx                  # Root + Router
│   └── main.tsx                 # Entry point
│
├── public/
├── index.html
├── package.json
├── tsconfig.json
├── vite.config.ts
└── tailwind.config.ts
```

---

## Naming Conventions

| Entity | Convention | Example |
|--------|-----------|---------|
| Component files | PascalCase `.tsx` | `ChatPage.tsx`, `MetricCard.tsx` |
| Hook files | kebab-case `use-*.ts` | `use-agent-chat.ts` |
| Utility files | kebab-case `.ts` | `api-client.ts` |
| Page directories | kebab-case | `pages/chat/`, `pages/dashboard/` |
| Types files | kebab-case `.ts` | `types/agent.ts` |

---

## Key Component: ChartRenderer

The `ChartRenderer` component takes structured data from the Agent and renders ECharts:

```tsx
// Agent returns structured chart data
interface ChartData {
  type: 'line' | 'bar' | 'pie' | 'funnel' | 'heatmap';
  title: string;
  data: Record<string, unknown>;
  options?: Record<string, unknown>;
}

// ChartRenderer maps to ECharts options
export function ChartRenderer({ chart }: { chart: ChartData }) {
  const option = buildEChartsOption(chart);
  return <ReactECharts option={option} style={{ height: 400 }} />;
}
```

This is the core pattern: **Agent produces data, frontend renders visuals**.
