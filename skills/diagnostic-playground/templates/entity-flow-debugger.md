# Entity Flow Debugger Template

Diagnoses issues in the primary user action pipeline: entity creation, side-effect processing (points/credits), media uploads, and volume patterns. Features both a **visual flow diagram** showing the pipeline architecture and a **data table** for detailed inspection.

This template is generic — "entity" means whatever the core user-generated content is in the project (orders, posts, submissions, reports, listings, etc.).

## When to use

- Users create entities but side-effects (points, notifications, status updates) aren't firing
- Media uploads are failing silently
- Submission volume has dropped unexpectedly
- Testers need to verify the full action → side-effect → ledger flow
- You need to understand WHERE in the pipeline failures occur

## Data Requirements

Discover the project schema first (see SKILL.md Step 0), then:

### Pipeline Discovery (NEW)

Before running data queries, analyze the project code to discover the pipeline:

1. **Find the main action entry point** — the function that handles entity creation (e.g., `submitReport()`, `createOrder()`, `postComment()`)
2. **Trace the sequential steps** — auth checks, validation, database writes, side-effects
3. **Identify decision points** — where the flow can fail (auth, geofence, rate limit, validation)
4. **Map to step types**: `action`, `decision`, `success`, `failure`

Embed this as `DATA.pipeline` (see structure below).

### Data Queries

Adapt these queries from `references/supabase-queries.md`:

1. **Recent entities with user and context** (Entity Flow Pattern #1) → embed as `DATA.entities`
2. **Volume by time bucket** (Pattern #2) → embed as `DATA.volumeByHour`
3. **Side-effect verification** (Pattern #3) → embed as `DATA.sideEffects`
4. **Media upload rate** (Pattern #4) → embed as `DATA.mediaStats`

### Pipeline Counts (if analytics available)

If the project has analytics (PostHog, Amplitude, etc.), query for funnel counts:
- How many users reached each pipeline step?
- How many passed vs failed at each decision point?

If no analytics, estimate from database queries (e.g., auth failures = users with 0 entities, geofence failures inferred from app logs if available).

## Layout

```
+------------------------------------------+------------------+
|  HEADER: {Entity} Flow Debugger          |                  |
|  Queried at: {timestamp}                 |                  |
+------------------------------------------+   FINDINGS       |
|  TAB BAR: [Flow Diagram] [Data Table]    |   PANEL          |
+------------------------------------------+                  |
|  FLOW DIAGRAM (when tab active):         |   [Add Finding]  |
|  ┌────┐   ┌────┐   ┌────┐   ┌────┐      |                  |
|  │ 1  │──▶│ 2  │──▶│ 3  │──▶│ ✓  │      |   Finding 1...   |
|  │323 │   │310 │   │298 │   │277 │      |   Finding 2...   |
|  └────┘   └──┬─┘   └──┬─┘   └────┘      |                  |
|              │        │                  |                  |
|              ▼        ▼                  |                  |
|           ┌────┐   ┌────┐               |                  |
|           │ 13 │   │ 12 │               |                  |
|           │fail│   │fail│               |                  |
|           └────┘   └────┘               |                  |
+------------------------------------------+                  |
|  STAT CARDS:                             |                  |
|  [Total] [Completed] [Conversion Rate]   |                  |
|  [With Media] [Side-Effects OK]          |                  |
+------------------------------------------+   [Copy          |
|  DATA TABLE (when tab active):           |    Feedback]     |
|  Time | User | Target | Status |         |                  |
|  Category | Media | Side-Effect          |                  |
|  (sortable columns, scrollable)          |                  |
+------------------------------------------+------------------+
```

## Tab Bar

Toggle between Flow Diagram and Data Table views:

```html
<div class="tab-bar">
  <button class="tab active" data-tab="flow">Flow Diagram</button>
  <button class="tab" data-tab="data">Data Table</button>
</div>

<div id="flow-view" class="tab-content active">
  <!-- SVG flow diagram -->
</div>

<div id="data-view" class="tab-content" style="display: none;">
  <!-- Data table -->
</div>
```

```javascript
document.querySelectorAll('.tab').forEach(tab => {
  tab.addEventListener('click', () => {
    document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
    document.querySelectorAll('.tab-content').forEach(c => c.style.display = 'none');

    tab.classList.add('active');
    document.getElementById(tab.dataset.tab + '-view').style.display = 'block';
  });
});
```

## Flow Diagram

Use patterns from `references/flow-diagram.md` to render the pipeline.

### Key Features

1. **Nodes with real counts** — Each step shows how many entities entered that step
2. **Failure paths** — Decision nodes show red paths to failure boxes with failure counts
3. **Click to inspect** — Clicking a node shows step details in a panel
4. **Flag from diagram** — "Flag This Step" button adds the step as a finding
5. **Conversion funnel** — Visual representation of drop-off at each step

### Step Detail Panel

When a node is clicked, show:

```html
<div id="step-detail-panel" class="neo-card" style="display: none;">
  <h4 class="text-accent" id="step-title"></h4>
  <p class="text-muted" id="step-subtitle"></p>

  <div class="stat-row mt-8">
    <div class="mini-stat">
      <span class="value" id="step-entered">-</span>
      <span class="label">Entered</span>
    </div>
    <div class="mini-stat">
      <span class="value text-accent" id="step-passed">-</span>
      <span class="label">Passed</span>
    </div>
    <div class="mini-stat">
      <span class="value text-danger" id="step-failed">-</span>
      <span class="label">Failed</span>
    </div>
  </div>

  <div id="step-failure-info" class="mt-8" style="display: none;">
    <p class="text-danger text-sm" id="step-failure-message"></p>
  </div>

  <button class="add-finding-btn mt-8" onclick="flagStep(selectedStep)">
    Flag This Step
  </button>
</div>
```

## Interactive Features

### Stat Cards
- Calculate from embedded data on load
- Show: total entities, completed count, conversion rate %, entities with media, side-effect matched count

### Volume Chart
- CSS bar chart (no external library)
- One bar per time bucket from `DATA.volumeByHour`
- Bar height proportional to `entity_count`
- Hover tooltip shows: time, count, unique users
- Color-code: accent-primary (normal), warning (low), danger (zero)

### Filterable Data Table
- Default: show all entities, newest first
- Search box: filters across username, target name, category
- Status dropdown: filter by entity status values
- Side-effect dropdown: filter to show only matched, only missing, or all
- Click column header to sort ascending/descending
- Missing side-effect rows highlighted with pink background
- Media column: green checkmark if present, red X if not

### Row Details
- Click a table row to expand inline detail panel
- Shows: full entity ID, related IDs, exact timestamps, side-effect entry details
- "Flag this row" button adds the row's data as a finding

### Feedback Panel
- Fixed right sidebar (320px)
- "Add Finding" button opens modal (see `references/feedback-schema.md`)
- Category dropdown: bug, data-issue, missing-analytics, question
- Severity dropdown: critical, high, medium, low
- Description textarea
- "Affected IDs" auto-populates if flagged from a row or step
- Findings list with color-coded badges
- "Copy Feedback" button at bottom

## HTML Generation Notes

### Data Embedding

```javascript
const DATA = Object.freeze({
  // Pipeline structure (from code analysis)
  pipeline: Object.freeze({
    steps: [
      {
        id: 'step_id',
        title: 'Step Title',
        subtitle: 'What happens here',
        type: 'action|decision|success|failure',
        count: 323,        // How many entered this step
        passCount: 310,    // How many passed to next step
        failureCount: 13,  // How many failed here
        failureMessage: 'Error description if applicable'
      },
      // ... more steps
    ],
    totalEntered: 323,
    totalCompleted: 277,
    conversionRate: 85.7
  }),

  // Entity data (from database queries)
  entities: Object.freeze([
    // Each row from adapted Pattern #1
    // Fields will vary per project schema
  ]),

  volumeByHour: Object.freeze([
    // Each row from adapted Pattern #2
    { time_bucket, entity_count, unique_users, with_media, successful, failed }
  ]),

  sideEffects: Object.freeze([
    // Each row from adapted Pattern #3
    // Links entities to their expected ledger/transaction entries
  ]),

  mediaStats: Object.freeze({
    // Single row from adapted Pattern #4
    total_entities, with_media, without_media, media_rate_pct
  }),

  _meta: Object.freeze({
    queriedAt: "{ISO timestamp}",
    projectId: "{supabase-project-id}",
    template: "entity-flow-debugger",
    entityType: "{actual entity name, e.g. 'orders', 'reports'}",
    rowCounts: { entities: N, volumeBuckets: N, sideEffects: N }
  })
});
```

### Side-Effect Match Logic

An entity has a "matched" side-effect if there exists a corresponding row in `DATA.sideEffects` where the ledger/transaction ID is not null:

```javascript
function isSideEffectMatched(entityId) {
  return DATA.sideEffects.some(s => s.entity_id === entityId && s.ledger_id !== null);
}
```

### Sort Implementation

```javascript
let sortColumn = 'created_at';
let sortDir = 'desc';

function sortData(data, column, direction) {
  return [...data].sort((a, b) => {
    const va = a[column], vb = b[column];
    if (va == null) return 1;
    if (vb == null) return -1;
    const cmp = va < vb ? -1 : va > vb ? 1 : 0;
    return direction === 'asc' ? cmp : -cmp;
  });
}
```

## CSS Specifics

- Use classes from `references/dashboard-theme.md` and `references/flow-diagram.md`
- Missing side-effect rows: `background: rgba(255, 125, 161, 0.1)` (translucent pink)
- Media present: `color: var(--accent-primary)` with checkmark
- Media missing: `color: var(--accent-danger)` with X
- Volume chart bars: `background: var(--accent-primary)` default, `background: var(--accent-danger)` for zero-activity hours
- Flow diagram nodes: follow color patterns in `references/flow-diagram.md`
