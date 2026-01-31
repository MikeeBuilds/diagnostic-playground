# Supabase Query Patterns

Example SQL patterns for each diagnostic template. **Adapt these to the actual project schema** — the table and column names below are illustrative placeholders.

Before writing queries, run `mcp__supabase__list_tables` to discover the real schema.

---

## Schema Discovery

Run this first to understand the project's data model:

```sql
-- List all tables and their row counts
SELECT
  schemaname,
  relname AS table_name,
  n_live_tup AS row_count
FROM pg_stat_user_tables
WHERE schemaname = 'public'
ORDER BY n_live_tup DESC;
```

```sql
-- Get columns for a specific table (replace 'table_name')
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_schema = 'public' AND table_name = '{table_name}'
ORDER BY ordinal_position;
```

```sql
-- Find foreign key relationships
SELECT
  tc.table_name AS source_table,
  kcu.column_name AS source_column,
  ccu.table_name AS target_table,
  ccu.column_name AS target_column
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage ccu ON tc.constraint_name = ccu.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY' AND tc.table_schema = 'public';
```

---

## Entity Flow Debugger Patterns

These patterns assume a primary content table (e.g., `orders`, `posts`, `submissions`) linked to users and optionally to categories/items.

### 1. Recent entities with user and context

```sql
-- Adapt: replace {entities}, {users}, {categories} with actual table names
-- Adapt: replace column names to match actual schema
SELECT
  e.id AS entity_id,
  e.created_at,
  e.status,
  e.{media_column},           -- e.g., photo_url, attachment_url, image_path
  e.{target_id_column},       -- e.g., location_id, project_id, channel_id
  t.{target_name_column},     -- e.g., name, title, retailer_name
  e.user_id,
  u.{username_column},        -- e.g., username, display_name, email
  u.{role_column},            -- e.g., role, rank, tier
  c.{category_name_column}    -- e.g., name, title, label
FROM {entities} e
JOIN {users} u ON e.user_id = u.id
LEFT JOIN {targets} t ON e.{target_id_column} = t.id
LEFT JOIN {categories} c ON e.{category_id_column} = c.id
ORDER BY e.created_at DESC
LIMIT 200;
```

### 2. Volume by time bucket (last 7 days)

```sql
SELECT
  date_trunc('hour', created_at) AS time_bucket,
  COUNT(*) AS entity_count,
  COUNT(DISTINCT user_id) AS unique_users,
  COUNT(CASE WHEN {media_column} IS NOT NULL THEN 1 END) AS with_media,
  COUNT(CASE WHEN status = '{success_status}' THEN 1 END) AS successful,
  COUNT(CASE WHEN status = '{failure_status}' THEN 1 END) AS failed
FROM {entities}
WHERE created_at > NOW() - INTERVAL '7 days'
GROUP BY time_bucket
ORDER BY time_bucket DESC;
```

### 3. Side-effect verification (e.g., points/credits awarded for actions)

```sql
-- Check if each entity has a corresponding ledger/transaction entry
SELECT
  e.id AS entity_id,
  e.user_id,
  e.created_at AS entity_time,
  u.{username_column},
  l.id AS ledger_id,
  l.{amount_column},
  l.{reason_column},
  l.created_at AS ledger_time
FROM {entities} e
JOIN {users} u ON e.user_id = u.id
LEFT JOIN {ledger} l ON l.user_id = e.user_id
  AND l.{reason_column} LIKE '%{entity_type}%'
  AND l.created_at BETWEEN e.created_at - INTERVAL '5 minutes' AND e.created_at + INTERVAL '5 minutes'
WHERE e.created_at > NOW() - INTERVAL '7 days'
ORDER BY e.created_at DESC;
```

### 4. Media upload success rate

```sql
SELECT
  COUNT(*) AS total_entities,
  COUNT(CASE WHEN {media_column} IS NOT NULL AND {media_column} != '' THEN 1 END) AS with_media,
  COUNT(CASE WHEN {media_column} IS NULL OR {media_column} = '' THEN 1 END) AS without_media,
  ROUND(
    COUNT(CASE WHEN {media_column} IS NOT NULL AND {media_column} != '' THEN 1 END)::numeric
    / NULLIF(COUNT(*), 0) * 100, 1
  ) AS media_rate_pct
FROM {entities}
WHERE created_at > NOW() - INTERVAL '7 days';
```

---

## User Funnel Patterns

### 1. All users with activity data

```sql
SELECT
  u.id AS user_id,
  u.{username_column},
  u.created_at AS signup_date,
  u.{activity_count_column},    -- e.g., posts_count, orders_count, reports_count
  u.{points_column},            -- e.g., xp, credits, karma (if exists)
  u.{role_column},              -- e.g., role, rank, tier (if exists)
  u.{last_active_column},       -- e.g., last_active, last_seen, updated_at
  u.{avatar_column} IS NOT NULL AS has_avatar,
  u.{bio_column} IS NOT NULL AND u.{bio_column} != '' AS has_bio
FROM {users} u
ORDER BY u.created_at DESC;
```

### 2. Actual activity counts per user

```sql
SELECT
  user_id,
  COUNT(*) AS actual_count,
  MIN(created_at) AS first_action_at,
  MAX(created_at) AS last_action_at
FROM {entities}
GROUP BY user_id;
```

### 3. Time from signup to first action

```sql
SELECT
  u.id AS user_id,
  u.{username_column},
  u.created_at AS signup_date,
  MIN(e.created_at) AS first_action_date,
  EXTRACT(EPOCH FROM (MIN(e.created_at) - u.created_at)) / 3600 AS hours_to_first_action
FROM {users} u
LEFT JOIN {entities} e ON e.user_id = u.id
GROUP BY u.id, u.{username_column}, u.created_at
ORDER BY u.created_at DESC;
```

### 4. Funnel stage counts

```sql
-- Adapt stages to match the project's activation milestones
SELECT
  COUNT(*) AS total_signups,
  COUNT(CASE WHEN {bio_column} IS NOT NULL AND {bio_column} != '' OR {avatar_column} IS NOT NULL THEN 1 END) AS profile_completed,
  COUNT(CASE WHEN {activity_count_column} > 0 THEN 1 END) AS completed_first_action,
  COUNT(CASE WHEN {activity_count_column} >= 5 THEN 1 END) AS power_users,
  COUNT(CASE WHEN {last_active_column} > NOW() - INTERVAL '7 days' THEN 1 END) AS active_last_7d,
  COUNT(CASE WHEN {last_active_column} > NOW() - INTERVAL '30 days' THEN 1 END) AS active_last_30d
FROM {users};
```

### 5. Signup cohort breakdown (weekly)

```sql
SELECT
  date_trunc('week', u.created_at) AS cohort_week,
  COUNT(*) AS signups,
  COUNT(CASE WHEN u.{activity_count_column} > 0 THEN 1 END) AS converted,
  ROUND(
    COUNT(CASE WHEN u.{activity_count_column} > 0 THEN 1 END)::numeric
    / NULLIF(COUNT(*), 0) * 100, 1
  ) AS conversion_pct
FROM {users} u
GROUP BY cohort_week
ORDER BY cohort_week DESC
LIMIT 12;
```

---

## Data Integrity Checker Patterns

### 1. Denormalized count drift

```sql
-- Compare a cached count column on the user profile with the actual count
SELECT
  u.id AS user_id,
  u.{username_column},
  u.{activity_count_column} AS cached_count,
  COALESCE(actual.cnt, 0) AS actual_count,
  u.{activity_count_column} - COALESCE(actual.cnt, 0) AS drift
FROM {users} u
LEFT JOIN (
  SELECT user_id, COUNT(*) AS cnt
  FROM {entities}
  GROUP BY user_id
) actual ON actual.user_id = u.id
WHERE u.{activity_count_column} != COALESCE(actual.cnt, 0)
ORDER BY ABS(u.{activity_count_column} - COALESCE(actual.cnt, 0)) DESC;
```

### 2. Balance drift (cached balance vs ledger sum)

```sql
-- Compare a cached balance column with the actual ledger sum
SELECT
  u.id AS user_id,
  u.{username_column},
  u.{points_column} AS cached_balance,
  COALESCE(ledger.total, 0) AS ledger_total,
  u.{points_column} - COALESCE(ledger.total, 0) AS drift
FROM {users} u
LEFT JOIN (
  SELECT user_id, SUM({amount_column}) AS total
  FROM {ledger}
  GROUP BY user_id
) ledger ON ledger.user_id = u.id
WHERE u.{points_column} != COALESCE(ledger.total, 0)
ORDER BY ABS(u.{points_column} - COALESCE(ledger.total, 0)) DESC;
```

### 3. Status conflicts (parent status vs latest child)

```sql
-- Compare a parent record's status with its most recent child record's status
SELECT
  p.id AS parent_id,
  p.{parent_name_column},
  p.{parent_status_column} AS parent_status,
  latest.{child_status_column} AS latest_child_status,
  latest.created_at AS latest_child_time
FROM {parents} p
LEFT JOIN LATERAL (
  SELECT {child_status_column}, created_at
  FROM {children}
  WHERE {parent_fk} = p.id
  ORDER BY created_at DESC
  LIMIT 1
) latest ON true
WHERE p.{parent_status_column} IS DISTINCT FROM latest.{child_status_column};
```

### 4. Orphaned records

```sql
-- Find records that reference non-existent foreign keys
-- Adapt each UNION block to the actual foreign key relationships

SELECT 'orphaned_{child}_to_{parent}' AS issue_type, c.id::text AS record_id, c.{fk_column}::text AS ref_id
FROM {children} c
LEFT JOIN {parents} p ON c.{fk_column} = p.id
WHERE p.id IS NULL

UNION ALL

SELECT 'orphaned_{table2}_to_{table3}', t2.id::text, t2.{fk_column}::text
FROM {table2} t2
LEFT JOIN {table3} t3 ON t2.{fk_column} = t3.id
WHERE t2.{fk_column} IS NOT NULL AND t3.id IS NULL;
```

### 5. Summary statistics

```sql
-- Get row counts for all relevant tables
SELECT
  (SELECT COUNT(*) FROM {users}) AS total_users,
  (SELECT COUNT(*) FROM {entities}) AS total_entities,
  (SELECT COUNT(*) FROM {targets}) AS total_targets,
  (SELECT COUNT(*) FROM {ledger}) AS total_ledger_entries;
```
