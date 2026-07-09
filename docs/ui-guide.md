# UI Guide

!!! note "About screenshots"
    This page previously carried screenshots from an older UI version. Rather than keep images that no longer match the current interface, this page describes each screen in text — accurate screenshots can be dropped into `docs/assets/images/ui/` and referenced from here once available.

SQLatte's frontend is a set of server-rendered pages plus two embeddable widget scripts. All routes below are served from the FastAPI app at your configured host/port.

## Main Interface (`/`)

The unified dashboard. Tabs are shown/hidden per the `ui.sections` config block — a full deployment shows: **Home**, **Assistant** (chat), **Demo** (widget preview), **Analytics**, **Schedules**, **Dashboards**, **BigQuery Ops**, and **Admin**. The chat/Assistant tab is where natural-language queries happen: pick a table, ask a question, get generated SQL, results table, and an optional AI insight — with a **🔑 API Tokens** entry point for generating MCP credentials (see [MCP Local Setup](mcp/local-setup.md)).

## Admin Panel (`/admin`)

Gated by [admin login](security/tokens.md#admin-panel-authentication) (`/admin/login` redirects unauthenticated browsers here). Sections, matching the sidebar:

| Section | What it does |
|---|---|
| Dashboard | Overview stats, quick actions |
| Prompts | Edit the five AI prompts — see [Runtime Prompts](features/runtime-prompts.md) |
| Tables | Browse connected database schema |
| Semantic Layer | Entity/relationship/metric builder, 5 sub-tabs — see [Semantic Layer](features/semantic-layer.md) |
| Email & SMTP | Configure delivery — see [Email Settings](configuration/email.md) |
| Scheduler | Manage scheduled queries (also its own page, see below) |
| Insights | Insights engine mode and limits |
| Ops Agent | BigQuery Ops Console toggles + cost alarm configuration |
| Export | CSV/Excel/HTML export settings |
| History | Configuration change log |
| Snapshots | Backup and restore config |
| Audit Logs | LLM call history, filterable by widget source — see [Audit Logs](features/audit-logs.md) |
| MCP Masking | Field-masking rules for MCP output — see [Field Masking](mcp/field-masking.md) |
| Tokens | Issue/revoke API tokens platform-wide, set token policy |

## Dedicated Pages

| Route | Purpose |
|---|---|
| `/dashboards.html` | Dashboard gallery — browse, refresh, delete saved dashboards ([details](features/dashboard.md)) |
| `/admin/schedules` (scheduler-admin) | Cross-user schedule management, execution history, email delivery logs |
| `/ops-agent` | BigQuery Ops Console — run cost/security/performance operations, manage alarms ([details](features/ops-console.md)) |
| `/tokens` | Standalone API token management page (same functionality as the widget's 🔑 button) |

## Embeddable Widgets

Two JS files, loaded via `<script>` tag into any external page — see the [main README](https://github.com/osmanuygar/sqlatte#-embedding-in-your-website) for embed snippets:

| Script | Auth model | Use case |
|---|---|---|
| `static/js/sqlatte-badge.js` | Shared backend credentials, no login | Internal tools, single-tenant dashboards |
| `static/js/sqlatte-badge-auth.js` | Per-user login, isolated sessions | Multi-tenant SaaS |

Both support `position`, `fullscreen`, and `apiBase` configuration options and are demo-able at `/demo`.

## Demo Pages (`/demo`)

`/demo/fullscreen`, `/demo/standard`, and `/demo/comparison` render the widgets in different embed configurations for evaluation before wiring them into a real site.
