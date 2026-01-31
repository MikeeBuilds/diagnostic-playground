# diagnostic-playground

A Claude Code plugin that generates interactive HTML dashboards using **real Supabase data** to debug app issues. Testers interact with the dashboard, flag problems, and copy structured feedback back to Claude for action.

Works with **any Supabase-backed project** — Claude discovers your schema and adapts the queries automatically.

## How it works

```
You describe the issue → Claude discovers your schema → runs Supabase queries
    → embeds real data in a self-contained HTML dashboard → opens in browser
        → tester filters data, flags issues, adds notes
            → copies structured feedback → pastes back to Claude
                → Claude acts (fixes code, writes migrations, files issues)
```

## Install

```bash
# Add the marketplace
/plugin marketplace add MikeeBuilds/diagnostic-playground

# Install the plugin
/plugin install diagnostic-playground
```

Or install directly from GitHub:

```bash
/plugin install https://github.com/MikeeBuilds/diagnostic-playground
```

## Usage

Invoke the skill in Claude Code:

```
/diagnostic-playground
```

Then describe what you want to diagnose:

- "Debug the order submission flow" → generates `diagnostic-entity-flow-debugger.html`
- "Show me the user conversion funnel" → generates `diagnostic-user-funnel.html`
- "Check data integrity across tables" → generates `diagnostic-data-integrity-checker.html`

## Templates

| Template | Purpose | What it checks |
|----------|---------|----------------|
| `entity-flow-debugger` | Primary user action pipeline | Entity creation, side-effects (points/credits), media uploads, volume trends |
| `user-funnel` | Signup → first action conversion | User profiles, cohorts, time-to-first-action, drop-off points |
| `data-integrity-checker` | Cross-table consistency | Denormalized count drift, balance drift, status conflicts, orphaned records |

## Requirements

- **Claude Code** with plugin support
- **Supabase MCP** configured (for `mcp__supabase__execute_sql`)
- Claude queries your Supabase project directly — no mock data, no manual configuration

## How feedback works

Each dashboard includes a **Findings Panel** where testers can:

1. Click **Add Finding** to log an issue
2. Select category: `bug`, `data-issue`, `missing-analytics`, or `question`
3. Select severity: `critical`, `high`, `medium`, or `low`
4. Add description and reference specific data rows
5. Click **Copy Feedback** to get structured text
6. Paste back into Claude Code → Claude investigates and acts

## Example use cases

- **Debugging user drop-off**: Why are 77% of signups not completing their first action?
- **Finding data bugs**: Do all orders have matching payment ledger entries?
- **Investigating reports**: Which submissions are failing silently?
- **Auditing consistency**: Do cached counts match actual row counts?

## Plugin structure

```
diagnostic-playground/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── README.md
└── skills/
    └── diagnostic-playground/
        ├── SKILL.md
        ├── templates/
        │   ├── entity-flow-debugger.md
        │   ├── user-funnel.md
        │   └── data-integrity-checker.md
        └── references/
            ├── supabase-queries.md
            ├── feedback-schema.md
            └── dashboard-theme.md
```

## Design

Dashboards use a Brutalist dark theme optimized for data debugging:
- Hard borders (2.5px), offset shadows, high contrast
- Clean monospace data tables with sortable columns
- Color-coded badges for status and severity
- Self-contained single HTML file — no external dependencies
- Works offline once generated

## Contributing

PRs welcome! Ideas for new templates:

- `error-log-viewer` — Visualize Supabase edge function logs
- `rls-policy-tester` — Test RLS policies with different user contexts
- `migration-diff` — Compare schema before/after migrations

## License

MIT
