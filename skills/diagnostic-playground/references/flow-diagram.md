# Flow Diagram Reference

SVG patterns for rendering interactive pipeline flow diagrams. Use these patterns to visualize the sequential steps in any entity creation flow, with real data annotations.

## When to Use

Add a flow diagram when the dashboard needs to show WHERE in a pipeline issues occur, not just WHAT the data shows. The flow diagram complements the data table by providing architectural context.

## SVG Container

```html
<svg id="flow-diagram" viewBox="0 0 900 400" class="flow-svg">
  <!-- Defs for markers and gradients -->
  <defs>
    <marker id="arrow-success" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto">
      <path d="M0,0 L0,6 L9,3 z" fill="var(--accent-primary)" />
    </marker>
    <marker id="arrow-failure" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto">
      <path d="M0,0 L0,6 L9,3 z" fill="var(--accent-danger)" />
    </marker>
    <marker id="arrow-neutral" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto">
      <path d="M0,0 L0,6 L9,3 z" fill="var(--text-secondary)" />
    </marker>
  </defs>

  <!-- Connections layer (render first, behind nodes) -->
  <g class="connections"></g>

  <!-- Nodes layer -->
  <g class="nodes"></g>

  <!-- Failure paths layer -->
  <g class="failure-paths"></g>
</svg>
```

## CSS for Flow Diagram

```css
.flow-svg {
  width: 100%;
  height: auto;
  min-height: 300px;
  background: var(--bg-secondary);
  border: var(--border-width) solid var(--ink);
  border-radius: var(--radius);
}

.flow-node {
  cursor: pointer;
  transition: transform 0.1s ease;
}

.flow-node:hover {
  transform: translateY(-2px);
}

.flow-node.selected .node-bg {
  stroke: var(--accent-info);
  stroke-width: 3px;
}

.flow-node.has-issue .node-bg {
  stroke: var(--accent-danger);
  stroke-width: 3px;
}

.node-bg {
  fill: var(--bg-tertiary);
  stroke: var(--ink);
  stroke-width: 2px;
  rx: 8;
  ry: 8;
}

.node-bg--action { fill: #1a3a2f; } /* Greenish for action steps */
.node-bg--decision { fill: #3a3a1a; } /* Yellowish for decision points */
.node-bg--success { fill: #1a4a2a; } /* Bright green for success */
.node-bg--failure { fill: #4a1a1a; } /* Reddish for failure states */

.node-badge {
  fill: var(--accent-primary);
  stroke: var(--ink);
  stroke-width: 1.5px;
}

.node-badge-text {
  fill: var(--ink);
  font-size: 11px;
  font-weight: 700;
  text-anchor: middle;
  dominant-baseline: central;
}

.node-title {
  fill: var(--text-primary);
  font-size: 13px;
  font-weight: 600;
}

.node-subtitle {
  fill: var(--text-secondary);
  font-size: 11px;
}

.node-count {
  fill: var(--accent-primary);
  font-size: 14px;
  font-weight: 700;
  font-family: 'SF Mono', monospace;
}

.node-count--zero {
  fill: var(--accent-danger);
}

.connection {
  fill: none;
  stroke: var(--text-secondary);
  stroke-width: 2px;
}

.connection--success {
  stroke: var(--accent-primary);
  marker-end: url(#arrow-success);
}

.connection--failure {
  stroke: var(--accent-danger);
  stroke-dasharray: 6 4;
  marker-end: url(#arrow-failure);
}

.connection--optional {
  stroke-dasharray: 4 4;
}

.failure-box {
  fill: var(--bg-tertiary);
  stroke: var(--accent-danger);
  stroke-width: 2px;
  rx: 6;
  ry: 6;
}

.failure-text {
  fill: var(--accent-danger);
  font-size: 11px;
}

.failure-count {
  fill: var(--text-primary);
  font-size: 12px;
  font-weight: 600;
  font-family: 'SF Mono', monospace;
}
```

## Node Pattern

Each pipeline step is a node with:
- Background rectangle with step-type color
- Numbered badge (step order)
- Title and subtitle
- Data count annotation

```javascript
function renderNode(step, index, x, y) {
  const width = 140;
  const height = 80;

  return `
    <g class="flow-node" data-step="${step.id}" transform="translate(${x}, ${y})">
      <!-- Background -->
      <rect class="node-bg node-bg--${step.type}" width="${width}" height="${height}" />

      <!-- Step badge -->
      <circle class="node-badge" cx="15" cy="15" r="12" />
      <text class="node-badge-text" x="15" y="15">${index + 1}</text>

      <!-- Title -->
      <text class="node-title" x="35" y="20">${step.title}</text>

      <!-- Subtitle -->
      <text class="node-subtitle" x="35" y="36">${step.subtitle}</text>

      <!-- Count annotation -->
      <text class="node-count ${step.count === 0 ? 'node-count--zero' : ''}" x="${width / 2}" y="${height - 12}">
        ${step.count.toLocaleString()}
      </text>

      <!-- Warning indicator if has failures -->
      ${step.failureCount > 0 ? `
        <text x="${width - 20}" y="20" fill="var(--accent-danger)" font-size="14">⚠️</text>
      ` : ''}
    </g>
  `;
}
```

## Connection Pattern

Bezier curves between nodes with directional arrows:

```javascript
function renderConnection(from, to, type = 'success') {
  const startX = from.x + from.width;
  const startY = from.y + from.height / 2;
  const endX = to.x;
  const endY = to.y + to.height / 2;

  // Control points for smooth curve
  const cx1 = startX + 30;
  const cx2 = endX - 30;

  return `
    <path
      class="connection connection--${type}"
      d="M ${startX} ${startY} C ${cx1} ${startY}, ${cx2} ${endY}, ${endX} ${endY}"
    />
  `;
}
```

## Failure Path Pattern

Vertical drop from decision node to failure box:

```javascript
function renderFailurePath(node, failureData) {
  const dropY = node.y + node.height + 60;

  return `
    <g class="failure-path" data-step="${node.id}">
      <!-- Vertical drop line -->
      <path
        class="connection connection--failure"
        d="M ${node.x + node.width / 2} ${node.y + node.height}
           L ${node.x + node.width / 2} ${dropY}"
      />

      <!-- Failure box -->
      <rect class="failure-box" x="${node.x - 20}" y="${dropY}" width="${node.width + 40}" height="50" />

      <!-- Failure message -->
      <text class="failure-text" x="${node.x + node.width / 2}" y="${dropY + 18}" text-anchor="middle">
        ${failureData.message}
      </text>

      <!-- Failure count -->
      <text class="failure-count" x="${node.x + node.width / 2}" y="${dropY + 36}" text-anchor="middle">
        ${failureData.count.toLocaleString()} failed
      </text>
    </g>
  `;
}
```

## Data Structure

The `DATA.pipeline` object should contain:

```javascript
const DATA = Object.freeze({
  // ... other data ...

  pipeline: Object.freeze({
    steps: [
      {
        id: 'auth_check',
        title: 'Auth Check',
        subtitle: 'Is user signed in?',
        type: 'decision', // action, decision, success, failure
        count: 323,       // Total that entered this step
        passCount: 310,   // Passed to next step
        failureCount: 13, // Failed at this step
        failureMessage: 'Not authenticated'
      },
      {
        id: 'geofence',
        title: 'Geofence Check',
        subtitle: 'Within 500m?',
        type: 'decision',
        count: 310,
        passCount: 298,
        failureCount: 12,
        failureMessage: 'Too far from location'
      },
      // ... more steps
    ],

    // Summary metrics
    totalEntered: 323,
    totalCompleted: 277,
    conversionRate: 85.7
  })
});
```

## Interactive Selection

```javascript
let selectedStep = null;

function selectStep(stepId) {
  // Remove previous selection
  document.querySelectorAll('.flow-node').forEach(n => n.classList.remove('selected'));

  // Add new selection
  const node = document.querySelector(`[data-step="${stepId}"]`);
  if (node) {
    node.classList.add('selected');
    selectedStep = stepId;
    showStepDetails(stepId);
  }
}

function showStepDetails(stepId) {
  const step = DATA.pipeline.steps.find(s => s.id === stepId);
  if (!step) return;

  const detailPanel = document.getElementById('step-detail-panel');
  detailPanel.innerHTML = `
    <h4>${step.title}</h4>
    <p class="text-muted">${step.subtitle}</p>
    <div class="stat-row mt-8">
      <div class="mini-stat">
        <span class="value">${step.count.toLocaleString()}</span>
        <span class="label">Entered</span>
      </div>
      <div class="mini-stat">
        <span class="value text-accent">${step.passCount.toLocaleString()}</span>
        <span class="label">Passed</span>
      </div>
      ${step.failureCount > 0 ? `
        <div class="mini-stat">
          <span class="value text-danger">${step.failureCount.toLocaleString()}</span>
          <span class="label">Failed</span>
        </div>
      ` : ''}
    </div>
    <button class="add-finding-btn mt-8" onclick="flagStep('${stepId}')">
      Flag This Step
    </button>
  `;
  detailPanel.style.display = 'block';
}

// Event listeners
document.querySelectorAll('.flow-node').forEach(node => {
  node.addEventListener('click', () => selectStep(node.dataset.step));
});
```

## Generating Pipeline from Code Analysis

When Claude discovers the pipeline, it should:

1. **Read the main action entry point** (e.g., `submitReport()` in `ReportStockSheet.swift`)
2. **Identify sequential checks** — auth guards, validation, geofencing, etc.
3. **Identify side-effects** — XP awards, notifications, cache invalidation
4. **Map to step types**:
   - `action` — User-initiated action (tap, select, submit)
   - `decision` — Validation or guard check (auth, geofence, rate limit)
   - `success` — Completion state
   - `failure` — Error state

5. **Query for counts at each step** — Use analytics events or infer from database queries

## Example: Stock Report Pipeline

```javascript
const STOCK_REPORT_PIPELINE = {
  steps: [
    { id: 'map_tap', title: 'Map Tap', subtitle: 'User taps pin', type: 'action' },
    { id: 'location_sheet', title: 'Location Sheet', subtitle: 'Detail sheet opens', type: 'action' },
    { id: 'auth_check', title: 'Auth Check', subtitle: 'Is user signed in?', type: 'decision' },
    { id: 'geofence', title: 'Geofence Check', subtitle: 'Within 500m?', type: 'decision' },
    { id: 'report_sheet', title: 'Report Sheet', subtitle: 'Stock report UI opens', type: 'action' },
    { id: 'select_status', title: 'Select Status', subtitle: 'In Stock / Low / Empty', type: 'action' },
    { id: 'select_products', title: 'Select Products', subtitle: 'Choose seen products', type: 'action' },
    { id: 'photo', title: 'Photo (Optional)', subtitle: 'Take/select photo', type: 'action' },
    { id: 'submit', title: 'Submit Report', subtitle: 'POST to Supabase', type: 'action' },
    { id: 'success', title: 'Success!', subtitle: '+XP awarded', type: 'success' }
  ]
};
```

## Legend

Include a legend explaining the visual encoding:

```html
<div class="flow-legend">
  <div class="legend-item">
    <svg width="20" height="10"><path d="M0,5 L20,5" stroke="var(--accent-primary)" stroke-width="2" marker-end="url(#arrow-success)"/></svg>
    <span>Success path</span>
  </div>
  <div class="legend-item">
    <svg width="20" height="10"><path d="M0,5 L20,5" stroke="var(--accent-danger)" stroke-width="2" stroke-dasharray="4 2"/></svg>
    <span>Failure path</span>
  </div>
  <div class="legend-item">
    <svg width="20" height="10"><path d="M0,5 L20,5" stroke="var(--text-secondary)" stroke-width="2" stroke-dasharray="2 2"/></svg>
    <span>Optional step</span>
  </div>
  <div class="legend-item">
    <span style="color: var(--accent-danger)">⚠️</span>
    <span>Has failures</span>
  </div>
</div>
```
