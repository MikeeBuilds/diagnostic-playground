---
name: diagnostic-playground
description: Generates interactive HTML dashboards using real Supabase data to debug app issues. Testers interact with live data, flag problems, and copy structured feedback back to Claude.
user-invocable: true
---

# Diagnostic Playground

Build interactive HTML dashboards that embed **real Supabase data** so testers can explore, filter, flag issues, and copy structured feedback back to Claude for action.

This plugin works with **any Supabase-backed project**. Claude discovers the project's schema, writes appropriate queries, and adapts the templates to the actual data model.

## When to use this skill

Use when debugging requires seeing **real data in context** — not just raw SQL output. Examples:

- "Why aren't users completing the core action?" → `user-funnel` template
- "Show me recent submission issues" → `entity-flow-debugger` template
- "Check data integrity across tables" → `data-integrity-checker` template
- Any time you need a tester to visually validate data before you act on it

## How to use this skill

### Step 0: Discover the project schema

Before anything else:

1. Identify the Supabase project ID from the codebase (check environment files, config, or CLAUDE.md)
2. Run `mcp__supabase__list_tables` to see available tables
3. Identify the key entities: users table, primary content table, any ledger/transaction tables, any status/category tables
4. Map the project's schema to the template's generic concepts:
   - **Users table** → the table with user profiles (e.g., `user_profiles`, `profiles`, `users`)
   - **Primary content table** → the main user-generated content (e.g., `orders`, `posts`, `reports`, `submissions`, `listings`)
   - **Ledger/points table** → any table tracking points, credits, or transactions (e.g., `xp_ledger`, `credits_log`, `transactions`)
   - **Categories/items table** → reference data (e.g., `products`, `categories`, `tags`)
   - **Locations/targets table** → what the content relates to (e.g., `locations`, `projects`, `channels`)

### Step 1: Pick a template

Read the appropriate template file for the diagnostic type:

- `templates/entity-flow-debugger.md` — Debug the primary user action flow (creation, processing, side-effects, volume trends)
- `templates/user-funnel.md` — Signup → first action conversion analysis (cohort breakdown, drop-off points)
- `templates/data-integrity-checker.md` — Cross-table consistency checks (count mismatches, balance drift, orphaned records)

### Step 2: Run ALL Supabase queries FIRST

**This is mandatory.** Before writing any HTML:

1. Read the template's **Data Requirements** section
2. Read `references/supabase-queries.md` for query patterns
3. **Adapt the example SQL** to the actual project schema discovered in Step 0
4. Execute every query using `mcp__supabase__execute_sql`
5. Store the results — you will embed them in the HTML

**Never generate HTML with placeholder or mock data.** The entire value of this plugin is real data.

### Step 3: Generate the HTML dashboard

1. Read `references/dashboard-theme.md` for the CSS foundation
2. Read `references/feedback-schema.md` for the feedback panel structure
3. Follow the template's **Layout** and **Interactive Features** sections
4. Embed query results as JavaScript constants:

```html
<script>
// Real data from Supabase — queried at {timestamp}
const DATA = {
  entities: [/* actual rows */],
  summary: {/* actual aggregates */},
  queriedAt: "{ISO timestamp}"
};
</script>
```

5. Include the feedback panel (every dashboard gets one)
6. Save as `diagnostic-{template-name}.html` in the project root
7. Open with `open diagnostic-{template-name}.html`

### Step 4: Handle feedback

When the tester pastes back structured feedback:

1. Parse the feedback block
2. For each finding:
   - **bug**: Investigate the code path, propose a fix
   - **data-issue**: Write a migration or SQL fix, run it
   - **missing-analytics**: Add the tracking event to the relevant view/component
   - **question**: Research and answer with data
3. If the finding references specific IDs, query them for more context
4. After acting, offer to regenerate the dashboard with fresh data

## Core requirements (every dashboard)

- **Single HTML file** — inline all CSS and JS, no external dependencies
- **Real data only** — all data comes from Supabase queries run before generation
- **Filterable tables** — click column headers to sort, use search/filter inputs
- **Feedback panel** — sidebar or bottom panel where testers flag findings
- **Copy Feedback button** — copies structured text to clipboard with "Copied!" confirmation
- **Timestamp** — show when the data was queried (so testers know freshness)
- **Dark theme** — uses the Brutalist dark theme from references/dashboard-theme.md

## Data embedding pattern

```javascript
// Embed as frozen constants — never mutate source data
const DATA = Object.freeze({
  // Template-specific data sections
  entities: Object.freeze([...rows]),
  summary: Object.freeze({...aggregates}),

  // Always include metadata
  _meta: Object.freeze({
    queriedAt: "2026-01-31T12:00:00Z",
    projectId: "{supabase-project-id}",
    template: "{template-name}",
    rowCounts: { entities: 150, users: 42 }
  })
});
```

## Feedback panel pattern

Every dashboard includes a findings panel where testers can:
1. Click "Add Finding" to open a form
2. Select a category (bug, data-issue, missing-analytics, question)
3. Add a description and optionally reference specific row IDs
4. All findings accumulate in a list
5. "Copy Feedback" serializes everything as structured text for Claude

See `references/feedback-schema.md` for the exact format.

## Common mistakes to avoid

- **Querying inside the HTML** — all data must be pre-embedded, not fetched at runtime
- **Using mock data** — if a query fails, report the error instead of faking it
- **Skipping the feedback panel** — it's the whole point; testers need to flag issues
- **External dependencies** — no CDN links, no fetch calls, everything is self-contained
- **Forgetting the timestamp** — stale data without context causes confusion
- **Over-filtering** — show all data by default, let testers narrow down
- **Hardcoding table names** — always discover the schema first and adapt queries
