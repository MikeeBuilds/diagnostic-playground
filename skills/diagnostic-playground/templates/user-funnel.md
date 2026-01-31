# User Funnel Template

Analyzes the signup → first action conversion funnel. Identifies where users drop off and which cohorts perform best.

This template is generic — "first action" means whatever the project's core activation event is (first order, first post, first submission, first listing, etc.).

## When to use

- "Why do most users never complete the core action?"
- Investigating onboarding friction
- Comparing conversion rates across signup cohorts
- Identifying what separates active users from churned users

## Data Requirements

Discover the project schema first (see SKILL.md Step 0), then adapt these queries from `references/supabase-queries.md`:

1. **All users with activity data** (User Funnel Pattern #1) → embed as `DATA.users`
2. **Actual activity counts per user** (Pattern #2) → embed as `DATA.activityCounts`
3. **Time from signup to first action** (Pattern #3) → embed as `DATA.timeToFirst`
4. **Funnel stage counts** (Pattern #4) → embed as `DATA.funnelCounts`
5. **Signup cohort breakdown** (Pattern #5) → embed as `DATA.cohorts`

## Layout

```
+------------------------------------------+------------------+
|  HEADER: User Funnel Analysis            |                  |
|  Queried at: {timestamp}                 |                  |
+------------------------------------------+   FINDINGS       |
|  FUNNEL VISUALIZATION:                   |   PANEL          |
|  Signup (100%) → Profile (X%) →          |                  |
|  First Action (Y%) → Power User (Z%)    |   [Add Finding]  |
+------------------------------------------+                  |
|  STAT CARDS:                             |   Finding 1...   |
|  [Total Users] [Converted] [Conv %]     |                  |
|  [Median Time to 1st] [Active 7d]       |                  |
+------------------------------------------+                  |
|  COHORT TABLE:                           |   [Copy          |
|  Week | Signups | Converted | Conv %     |    Feedback]     |
+------------------------------------------+                  |
|  FILTER BAR: [Search] [Cohort ▼]        |                  |
|  [Status: All/Never/Converted/Active]    |                  |
+------------------------------------------+                  |
|  USER TABLE:                             |                  |
|  Username | Signup | Actions | First     |                  |
|  Action | Hours to 1st | Last Active     |                  |
+------------------------------------------+------------------+
```

## Interactive Features

### Funnel Visualization
- Horizontal funnel with 4 stages, narrowing bars
- Each bar width proportional to percentage of total signups
- Stages from `DATA.funnelCounts`:
  1. **Signed Up** → `total_signups` (always 100%)
  2. **Profile Completed** → `profile_completed` (has avatar OR bio)
  3. **First Action** → `completed_first_action` (activity count > 0)
  4. **Power User** → `power_users` (activity count >= 5)
- Show count and percentage on each bar
- Drop-off percentage between each stage shown as annotation
- Colors: accent-primary for bars, accent-danger for drop-off annotations

### Stat Cards
Calculate from embedded data:
- Total users: `DATA.funnelCounts.total_signups`
- Converted (1+ action): `DATA.funnelCounts.completed_first_action`
- Conversion rate: percentage
- Median time to first action (calculate from `DATA.timeToFirst`)
- Active last 7 days: `DATA.funnelCounts.active_last_7d`

### Cohort Table
- One row per week from `DATA.cohorts`
- Columns: Week, Signups, Converted, Conversion %
- Conversion % column has inline bar chart (CSS width percentage)
- Color-code: green if conversion > 30%, yellow 15-30%, red < 15%
- Click a cohort row to filter the user table to that cohort

### User Table
- All users from `DATA.users` joined with `DATA.timeToFirst`
- Columns: Username, Signup Date, Actions (actual), First Action Date, Hours to First, Last Active, Status
- Status badge: "never" (0 actions, danger), "converted" (1-4 actions, success), "active" (5+, bright success), "churned" (last_active > 30 days ago, muted)
- Sortable columns
- Search filters across username
- Status filter buttons: All / Never Acted / Converted / Active / Churned
- Rows with 0 actions highlighted with subtle pink background
- Click row to expand: shows full user profile, action timeline

### Time-to-First Distribution
- Simple histogram showing hours-to-first-action distribution
- Buckets: <1h, 1-6h, 6-24h, 1-3d, 3-7d, 7-30d, 30d+
- Helps identify if there's a critical activation window

## HTML Generation Notes

### Data Embedding

```javascript
const DATA = Object.freeze({
  users: Object.freeze([
    // Each row from adapted Pattern #1
    // Fields will vary per project schema
  ]),

  activityCounts: Object.freeze([
    // Each row from adapted Pattern #2
    { user_id, actual_count, first_action_at, last_action_at }
  ]),

  timeToFirst: Object.freeze([
    // Each row from adapted Pattern #3
    { user_id, username, signup_date, first_action_date, hours_to_first_action }
  ]),

  funnelCounts: Object.freeze({
    // Single row from adapted Pattern #4
    total_signups, profile_completed, completed_first_action,
    power_users, active_last_7d, active_last_30d
  }),

  cohorts: Object.freeze([
    // Each row from adapted Pattern #5
    { cohort_week, signups, converted, conversion_pct }
  ]),

  _meta: Object.freeze({
    queriedAt: "{ISO timestamp}",
    projectId: "{supabase-project-id}",
    template: "user-funnel",
    rowCounts: { users: N, cohorts: N }
  })
});
```

### User Status Logic

```javascript
function getUserStatus(user) {
  const ac = DATA.activityCounts.find(a => a.user_id === user.user_id);
  const actual = ac ? ac.actual_count : 0;
  const lastActive = new Date(user.last_active);
  const daysSinceActive = (Date.now() - lastActive) / (1000 * 60 * 60 * 24);

  if (actual === 0) return 'never';
  if (daysSinceActive > 30 && actual < 5) return 'churned';
  if (actual >= 5) return 'active';
  return 'converted';
}
```

### Funnel Bar Widths

```javascript
function renderFunnel() {
  const fc = DATA.funnelCounts;
  const stages = [
    { label: 'Signed Up', count: fc.total_signups },
    { label: 'Profile Complete', count: fc.profile_completed },
    { label: 'First Action', count: fc.completed_first_action },
    { label: 'Power User (5+)', count: fc.power_users }
  ];
  const maxCount = stages[0].count;
  stages.forEach(s => {
    s.pct = ((s.count / maxCount) * 100).toFixed(1);
    s.width = Math.max((s.count / maxCount) * 100, 8); // min 8% width for visibility
  });
  return stages;
}
```

## CSS Specifics

- Funnel bars: `background: var(--accent-primary)`, height 48px, width proportional
- Drop-off annotations: `color: var(--accent-danger)`, positioned between bars
- "Never acted" rows: `background: rgba(255, 125, 161, 0.08)`
- Cohort conversion bars: inline `<div>` with percentage width, 6px height, accent-primary background
- Status badges use the standard `.badge` classes from dashboard-theme.md
