# Dashboard Theme (Brutalist Dark)

CSS foundation for all diagnostic playground dashboards. A bold, high-contrast dark theme with hard borders and offset shadows. Clean, developer-focused aesthetic that works for any project.

## Color Palette

```css
:root {
  /* Accent colors */
  --accent-primary: #D9F347;     /* Bright lime — CTAs, success, active states */
  --accent-danger: #FF7DA1;      /* Pink — errors, mismatches, alerts */
  --accent-info: #E0D4FC;        /* Lilac — info badges, secondary highlights */
  --accent-secondary: #FCE0EF;   /* Blush — tertiary highlights */
  --accent-warning: #f0b429;     /* Amber — warnings, medium severity */
  --ink: #0C0C0C;                /* Near-black — borders, shadows, heavy text */

  /* Dashboard dark theme */
  --bg-primary: #0d1117;
  --bg-secondary: #161b22;
  --bg-tertiary: #1c2128;
  --border-color: #30363d;
  --border-heavy: #484f58;
  --text-primary: #e6edf3;
  --text-secondary: #8b949e;
  --text-muted: #6e7681;

  /* Semantic aliases */
  --success: var(--accent-primary);
  --danger: var(--accent-danger);
  --warning: var(--accent-warning);
  --info: var(--accent-info);

  /* Brutalist tokens */
  --border-width: 2.5px;
  --radius: 12px;
  --shadow-offset: 4px;
}
```

## Base Styles

```css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'SF Mono', 'Segoe UI', system-ui, sans-serif;
  background: var(--bg-primary);
  color: var(--text-primary);
  min-height: 100vh;
  line-height: 1.5;
}
```

## Brutalist Card

The primary container for data sections:

```css
.neo-card {
  background: var(--bg-secondary);
  border: var(--border-width) solid var(--ink);
  border-radius: var(--radius);
  box-shadow: var(--shadow-offset) var(--shadow-offset) 0 var(--ink);
  padding: 16px;
  transition: transform 0.1s ease;
}

.neo-card:hover {
  transform: translate(-1px, -1px);
  box-shadow: calc(var(--shadow-offset) + 1px) calc(var(--shadow-offset) + 1px) 0 var(--ink);
}
```

## Data Table

```css
.data-table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  border: var(--border-width) solid var(--ink);
  border-radius: var(--radius);
  overflow: hidden;
}

.data-table th {
  background: var(--bg-tertiary);
  color: var(--accent-primary);
  font-weight: 700;
  text-transform: uppercase;
  font-size: 11px;
  letter-spacing: 0.5px;
  padding: 10px 12px;
  text-align: left;
  border-bottom: var(--border-width) solid var(--ink);
  cursor: pointer;
  user-select: none;
}

.data-table th:hover {
  background: var(--border-color);
}

.data-table th .sort-arrow {
  margin-left: 4px;
  opacity: 0.5;
}

.data-table th .sort-arrow.active {
  opacity: 1;
  color: var(--accent-primary);
}

.data-table td {
  padding: 8px 12px;
  border-bottom: 1px solid var(--border-color);
  font-size: 13px;
  font-family: 'SF Mono', 'Fira Code', monospace;
}

.data-table tr:hover td {
  background: var(--bg-tertiary);
}

.data-table tr:last-child td {
  border-bottom: none;
}
```

## Status Badges

```css
.badge {
  display: inline-block;
  padding: 2px 8px;
  border-radius: 6px;
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  border: 1.5px solid var(--ink);
}

.badge--match, .badge--success {
  background: var(--accent-primary);
  color: var(--ink);
}

.badge--mismatch, .badge--danger {
  background: var(--accent-danger);
  color: var(--ink);
}

.badge--warning {
  background: var(--accent-warning);
  color: var(--ink);
}

.badge--info {
  background: var(--accent-info);
  color: var(--ink);
}
```

## Dashboard Layout

```css
.dashboard {
  display: grid;
  grid-template-columns: 1fr 320px;
  grid-template-rows: auto 1fr;
  height: 100vh;
  gap: 1px;
  background: var(--border-color);
}

.dashboard-header {
  grid-column: 1 / -1;
  background: var(--bg-secondary);
  padding: 16px 24px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: var(--border-width) solid var(--ink);
}

.dashboard-main {
  background: var(--bg-primary);
  padding: 20px;
  overflow-y: auto;
}

.dashboard-sidebar {
  background: var(--bg-secondary);
  padding: 16px;
  overflow-y: auto;
  border-left: var(--border-width) solid var(--ink);
}
```

## Stat Cards Row

```css
.stat-row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 12px;
  margin-bottom: 20px;
}

.stat-card {
  background: var(--bg-secondary);
  border: var(--border-width) solid var(--ink);
  border-radius: var(--radius);
  padding: 16px;
  box-shadow: 3px 3px 0 var(--ink);
}

.stat-card .stat-value {
  font-size: 32px;
  font-weight: 900;
  color: var(--accent-primary);
  line-height: 1;
}

.stat-card .stat-label {
  font-size: 12px;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-top: 4px;
}
```

## Filter Bar

```css
.filter-bar {
  display: flex;
  gap: 8px;
  margin-bottom: 16px;
  flex-wrap: wrap;
}

.filter-input {
  background: var(--bg-tertiary);
  border: var(--border-width) solid var(--ink);
  border-radius: var(--radius);
  color: var(--text-primary);
  padding: 8px 12px;
  font-size: 13px;
  outline: none;
  transition: border-color 0.15s;
}

.filter-input:focus {
  border-color: var(--accent-primary);
}

.filter-input::placeholder {
  color: var(--text-muted);
}

.filter-btn {
  background: var(--bg-tertiary);
  border: var(--border-width) solid var(--ink);
  border-radius: var(--radius);
  color: var(--text-primary);
  padding: 8px 16px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.1s;
}

.filter-btn:hover {
  background: var(--border-color);
}

.filter-btn.active {
  background: var(--accent-primary);
  color: var(--ink);
}
```

## Feedback Panel

```css
.feedback-panel {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.feedback-panel h3 {
  font-size: 14px;
  color: var(--accent-primary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 12px;
}

.finding-card {
  background: var(--bg-tertiary);
  border: 1.5px solid var(--border-heavy);
  border-radius: 8px;
  padding: 10px;
  margin-bottom: 8px;
}

.finding-card[data-severity="critical"] {
  border-color: var(--accent-danger);
}

.finding-card[data-severity="high"] {
  border-color: var(--accent-warning);
}

.finding-header {
  display: flex;
  gap: 6px;
  margin-bottom: 6px;
}

.finding-badge {
  font-size: 10px;
  font-weight: 700;
  padding: 2px 6px;
  border-radius: 4px;
  text-transform: uppercase;
}

.finding-badge--bug { background: var(--accent-danger); color: var(--ink); }
.finding-badge--data-issue { background: var(--accent-warning); color: var(--ink); }
.finding-badge--missing-analytics { background: var(--accent-info); color: var(--ink); }
.finding-badge--question { background: var(--accent-secondary); color: var(--ink); }

.finding-desc {
  font-size: 12px;
  color: var(--text-secondary);
  line-height: 1.4;
}

.finding-ids {
  font-size: 11px;
  color: var(--text-muted);
  font-family: monospace;
  margin-top: 4px;
}

.add-finding-btn {
  background: var(--accent-primary);
  color: var(--ink);
  border: var(--border-width) solid var(--ink);
  border-radius: var(--radius);
  padding: 10px;
  font-weight: 700;
  font-size: 13px;
  cursor: pointer;
  margin-bottom: 8px;
  box-shadow: 3px 3px 0 var(--ink);
  transition: all 0.1s;
}

.add-finding-btn:hover {
  transform: translate(-1px, -1px);
  box-shadow: 4px 4px 0 var(--ink);
}

.add-finding-btn:active {
  transform: translate(1px, 1px);
  box-shadow: 2px 2px 0 var(--ink);
}

.copy-btn {
  background: var(--bg-tertiary);
  color: var(--text-primary);
  border: var(--border-width) solid var(--ink);
  border-radius: var(--radius);
  padding: 10px;
  font-weight: 600;
  font-size: 13px;
  cursor: pointer;
  margin-top: auto;
  transition: all 0.15s;
}

.copy-btn:hover {
  background: var(--accent-primary);
  color: var(--ink);
}
```

## Tabs (for multi-section dashboards)

```css
.tab-bar {
  display: flex;
  border-bottom: var(--border-width) solid var(--ink);
  margin-bottom: 16px;
}

.tab {
  padding: 10px 20px;
  font-size: 13px;
  font-weight: 600;
  color: var(--text-secondary);
  cursor: pointer;
  border-bottom: 3px solid transparent;
  transition: all 0.15s;
}

.tab:hover {
  color: var(--text-primary);
}

.tab.active {
  color: var(--accent-primary);
  border-bottom-color: var(--accent-primary);
}
```

## Chart Area (CSS bar charts, no external libraries)

```css
.chart-container {
  background: var(--bg-secondary);
  border: var(--border-width) solid var(--ink);
  border-radius: var(--radius);
  padding: 16px;
  margin-bottom: 16px;
}

.bar-chart {
  display: flex;
  align-items: flex-end;
  gap: 4px;
  height: 120px;
  padding-top: 8px;
}

.bar {
  flex: 1;
  background: var(--accent-primary);
  border: 1px solid var(--ink);
  border-radius: 3px 3px 0 0;
  min-width: 8px;
  position: relative;
  transition: opacity 0.15s;
}

.bar:hover {
  opacity: 0.8;
}

.bar .bar-tooltip {
  display: none;
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  background: var(--ink);
  color: var(--text-primary);
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 11px;
  white-space: nowrap;
  margin-bottom: 4px;
}

.bar:hover .bar-tooltip {
  display: block;
}
```

## Responsive Utilities

```css
.text-mono { font-family: 'SF Mono', 'Fira Code', monospace; }
.text-xs { font-size: 11px; }
.text-sm { font-size: 13px; }
.text-muted { color: var(--text-muted); }
.text-accent { color: var(--accent-primary); }
.text-danger { color: var(--accent-danger); }
.mt-8 { margin-top: 8px; }
.mt-16 { margin-top: 16px; }
.mb-8 { margin-bottom: 8px; }
.mb-16 { margin-bottom: 16px; }
.gap-8 { gap: 8px; }
.gap-12 { gap: 12px; }
.flex { display: flex; }
.flex-col { flex-direction: column; }
.items-center { align-items: center; }
.justify-between { justify-content: space-between; }
```

## Modal Overlay (for Add Finding form)

```css
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
}

.modal {
  background: var(--bg-secondary);
  border: var(--border-width) solid var(--ink);
  border-radius: var(--radius);
  box-shadow: 8px 8px 0 var(--ink);
  padding: 24px;
  width: 90%;
  max-width: 480px;
}

.modal h3 {
  font-size: 16px;
  margin-bottom: 16px;
}

.modal select,
.modal textarea,
.modal input {
  width: 100%;
  background: var(--bg-tertiary);
  border: var(--border-width) solid var(--ink);
  border-radius: 8px;
  color: var(--text-primary);
  padding: 8px 12px;
  font-size: 13px;
  margin-bottom: 12px;
  outline: none;
}

.modal select:focus,
.modal textarea:focus,
.modal input:focus {
  border-color: var(--accent-primary);
}

.modal textarea {
  min-height: 80px;
  resize: vertical;
  font-family: inherit;
}

.modal-actions {
  display: flex;
  gap: 8px;
  justify-content: flex-end;
}
```
