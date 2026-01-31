# Entity Flow Debugger Template

Diagnoses issues in the primary user action pipeline: entity creation, side-effect processing (points/credits), media uploads, and volume patterns.

This template is generic — "entity" means whatever the core user-generated content is in the project (orders, posts, submissions, reports, listings, etc.).

## When to use

- Users create entities but side-effects (points, notifications, status updates) aren't firing
- Media uploads are failing silently
- Submission volume has dropped unexpectedly
- Testers need to verify the full action → side-effect → ledger flow

## Data Requirements

Discover the project schema first (see SKILL.md Step 0), then adapt these queries from `references/supabase-queries.md`:

1. **Recent entities with user and context** (Entity Flow Pattern #1) → embed as `DATA.entities`
2. **Volume by time bucket** (Pattern #2) → embed as `DATA.volumeByHour`
3. **Side-effect verification** (Pattern #3) → embed as `DATA.sideEffects`
4. **Media upload rate** (Pattern #4) → embed as `DATA.mediaStats`

## Layout

```
+------------------------------------------+------------------+
|  HEADER: {Entity} Flow Debugger          |                  |
|  Queried at: {timestamp}                 |                  |
+------------------------------------------+   FINDINGS       |
|  STAT CARDS:                             |   PANEL          |
|  [Total] [With Media] [Side-Effect OK]  |                  |
|  [Side-Effect Missing] [Avg/Day]        |   [Add Finding]  |
+------------------------------------------+                  |
|  VOLUME CHART (bar chart, last 7 days)   |   Finding 1...   |
|  - Bars colored by status                |   Finding 2...   |
+------------------------------------------+                  |
|  FILTER BAR:                             |                  |
|  [Search] [Status ▼] [Side-Effect ▼]    |                  |
+------------------------------------------+   [Copy          |
|  DATA TABLE:                             |    Feedback]     |
|  Time | User | Target | Status |         |                  |
|  Category | Media | Side-Effect          |                  |
|  (sortable columns, scrollable)          |                  |
+------------------------------------------+------------------+
```

## Interactive Features

### Stat Cards
- Calculate from embedded data on load
- Show: total entities, entities with media, side-effect matched count, side-effect missing count, average per day

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
- "Affected IDs" auto-populates if flagged from a row
- Findings list with color-coded badges
- "Copy Feedback" button at bottom

## HTML Generation Notes

### Data Embedding

```javascript
const DATA = Object.freeze({
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

- Use classes from `references/dashboard-theme.md`
- Missing side-effect rows: `background: rgba(255, 125, 161, 0.1)` (translucent pink)
- Media present: `color: var(--accent-primary)` with checkmark
- Media missing: `color: var(--accent-danger)` with X
- Volume chart bars: `background: var(--accent-primary)` default, `background: var(--accent-danger)` for zero-activity hours
