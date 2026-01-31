# Data Integrity Checker Template

Identifies data drift and inconsistencies across Supabase tables. Compares denormalized counts with actual aggregates, checks for orphaned records, and validates cached balances.

This template is generic — it works with any project that has denormalized counts, cached balances, parent-child relationships, or foreign key dependencies.

## When to use

- Suspected data drift (cached counts don't match reality)
- After a migration or bulk operation
- Routine health check before a release
- Investigating balance/points discrepancies

## Data Requirements

Discover the project schema first (see SKILL.md Step 0), then adapt these queries from `references/supabase-queries.md`:

1. **Denormalized count drift** (Data Integrity Pattern #1) → embed as `DATA.countDrift`
2. **Balance drift** (Pattern #2) → embed as `DATA.balanceDrift`
3. **Status conflicts** (Pattern #3) → embed as `DATA.statusConflicts`
4. **Orphaned records** (Pattern #4) → embed as `DATA.orphans`
5. **Summary statistics** (Pattern #5) → embed as `DATA.summary`

**Note:** Not every project will have all 4 check types. Skip patterns that don't apply (e.g., if there's no points/credits system, skip balance drift). Adapt the tabs accordingly.

## Layout

```
+------------------------------------------+------------------+
|  HEADER: Data Integrity Checker          |                  |
|  Queried at: {timestamp}                 |                  |
+------------------------------------------+   FINDINGS       |
|  HEALTH SCORE BAR:                       |   PANEL          |
|  ████████████░░ 78% Healthy              |                  |
+------------------------------------------+   [Add Finding]  |
|  STAT CARDS:                             |                  |
|  [Total Users] [Total Entities]          |   Finding 1...   |
|  [Total Targets] [Total Ledger]          |                  |
+------------------------------------------+                  |
|  TABS:                                   |                  |
|  [Count Drift] [Balance Drift]           |                  |
|  [Status Conflicts] [Orphans]            |   [Copy          |
+------------------------------------------+    Feedback]     |
|  TAB CONTENT (table for active tab)      |                  |
|  Color-coded rows: green=OK, red=bad     |                  |
+------------------------------------------+------------------+
```

## Interactive Features

### Health Score
- Calculate overall health percentage:
  - Start at 100%
  - Subtract points for each issue type:
    - Each count drift: -1 point
    - Each balance drift > 10: -2 points
    - Each status conflict: -1 point
    - Each orphaned record: -3 points
  - Minimum 0%, maximum 100%
- Display as a progress bar with color gradient (success → warning → danger)
- Show fraction: "X issues found across Y records checked"

### Summary Stat Cards
- Show row counts for the main tables from `DATA.summary`
- Each card shows the count in large text with table name label

### Tabbed Interface
Up to 4 tabs (skip any that don't apply to the project):

#### Tab 1: Count Drift
- Table from `DATA.countDrift`
- Columns: User, Cached Count, Actual Count, Drift
- Drift column: green badge if 0, red badge if non-zero
- Positive drift = "cached says more", negative = "cached says fewer"
- Sort by absolute drift descending by default

#### Tab 2: Balance Drift
- Table from `DATA.balanceDrift`
- Columns: User, Cached Balance, Ledger Total, Drift
- Same green/red badge pattern
- Large drifts (>100) get `critical` highlighting
- Show drift direction: "+50 (cached ahead)" or "-30 (ledger ahead)"

#### Tab 3: Status Conflicts
- Table from `DATA.statusConflicts`
- Columns: Parent Name, Parent Status, Latest Child Status, Child Time
- Status columns use badge colors
- "Stale" indicator if latest child is > 7 days old

#### Tab 4: Orphaned Records
- Table from `DATA.orphans`
- Columns: Issue Type, Record ID, Missing Reference ID
- Group by issue type with sub-headers
- Each row highlighted red
- Count summary at top

### Cross-Tab Summary
- Below tabs, show a summary bar:
- "Count drift: X | Balance drift: Y | Status conflicts: Z | Orphans: W"
- Each count clickable to switch to that tab

### Fix SQL Generator
- Each tab has a "Generate Fix SQL" button
- Creates SQL UPDATE statements to resolve the issues
- SQL displayed in a `<pre>` block with "Copy SQL" button
- **Important:** These are suggestions — tester reviews before running

## HTML Generation Notes

### Data Embedding

```javascript
const DATA = Object.freeze({
  countDrift: Object.freeze([
    // Each row from adapted Pattern #1
    { user_id, username, cached_count, actual_count, drift }
  ]),

  balanceDrift: Object.freeze([
    // Each row from adapted Pattern #2 (empty array if N/A)
    { user_id, username, cached_balance, ledger_total, drift }
  ]),

  statusConflicts: Object.freeze([
    // Each row from adapted Pattern #3 (empty array if N/A)
    { parent_id, parent_name, parent_status, latest_child_status, latest_child_time }
  ]),

  orphans: Object.freeze([
    // Each row from adapted Pattern #4
    { issue_type, record_id, ref_id }
  ]),

  summary: Object.freeze({
    // Row counts from adapted Pattern #5
    // Keys will vary per project
  }),

  _meta: Object.freeze({
    queriedAt: "{ISO timestamp}",
    projectId: "{supabase-project-id}",
    template: "data-integrity-checker",
    rowCounts: {
      countDrift: N, balanceDrift: N,
      statusConflicts: N, orphans: N
    }
  })
});
```

### Health Score Calculation

```javascript
function calculateHealthScore() {
  let deductions = 0;
  deductions += DATA.countDrift.length * 1;
  deductions += DATA.balanceDrift.filter(d => Math.abs(d.drift) > 10).length * 2;
  deductions += DATA.statusConflicts.length * 1;
  deductions += DATA.orphans.length * 3;

  const totalChecked = Object.values(DATA.summary).reduce((a, b) => a + b, 0);
  const score = Math.max(0, Math.min(100, 100 - (deductions / Math.max(totalChecked, 1) * 100)));
  return { score: Math.round(score), issues: deductions, checked: totalChecked };
}
```

### Tab Switching

```javascript
let activeTab = 'count-drift';

function switchTab(tabId) {
  activeTab = tabId;
  document.querySelectorAll('.tab').forEach(t =>
    t.classList.toggle('active', t.dataset.tab === tabId)
  );
  document.querySelectorAll('.tab-content').forEach(c =>
    c.style.display = c.id === `tab-${tabId}` ? 'block' : 'none'
  );
}
```

### Fix SQL Generation

```javascript
// Claude should generate project-specific fix SQL based on the actual table/column names
// These are example patterns:

function generateCountDriftFix(tableName, countColumn, entityTable) {
  return DATA.countDrift.map(row =>
    `UPDATE ${tableName} SET ${countColumn} = ${row.actual_count} WHERE id = '${row.user_id}';`
  ).join('\n');
}

function generateBalanceDriftFix(tableName, balanceColumn) {
  return DATA.balanceDrift.map(row =>
    `UPDATE ${tableName} SET ${balanceColumn} = ${row.ledger_total} WHERE id = '${row.user_id}';`
  ).join('\n');
}
```

## CSS Specifics

- Health score bar: CSS gradient from `var(--accent-primary)` to `var(--accent-danger)`
- Tab content transitions: `display: none/block` (no animation needed)
- Drift = 0: `background: rgba(217, 243, 71, 0.1)` (subtle success)
- Drift != 0: `background: rgba(255, 125, 161, 0.1)` (subtle danger)
- Orphaned records: always `background: rgba(255, 125, 161, 0.15)` (stronger danger)
- Fix SQL blocks: `background: var(--bg-tertiary)`, `font-family: monospace`, `white-space: pre`
- Tab badges: show count of issues per tab, e.g., "Count Drift (5)"
