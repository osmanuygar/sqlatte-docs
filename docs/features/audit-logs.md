# Audit Logs

Every LLM call SQLatte makes — intent detection, SQL generation, chat, insights — is recorded, independent of whether [usage `analytics`](analytics.md) is enabled. This is the observability trail for cost tracking, compliance, and answering "what did this AI agent actually query."

## What's Logged

- Intent type (`intent_detection` / `sql_generation` / `chat` / `insights`)
- Input/output token counts
- The generated SQL and its [risk score](../security/overview.md#sql-injection-protection) (0–100, informational — never blocks execution)
- **Source**: `ui` (default chat), `widget` (embedded), or `mcp` — so you can see exactly how much traffic is coming from AI agents versus human users
- Session and user identifiers, timestamp

## Endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/audit/logs` | GET | Paginated log entries — filter by session, intent type, date range, or user |
| `/api/audit/summary` | GET | Aggregated token usage and call counts |
| `/api/audit/daily-trend` | GET | Call volume over time |
| `/api/audit/top-users` | GET | Highest token/call volume by user |
| `/api/audit/export/csv` | GET | Download filtered logs for billing or compliance review |

All audit routes require admin authentication.

## Admin Panel

`/admin` → **Audit Logs**, filterable by widget source — useful for answering "how much is the MCP integration costing us" separately from interactive chat usage, since both hit the same LLM billing but originate from very different traffic patterns.

## Risk Score Reference

Risk scoring never blocks a query — `is_select_only()` does that, unconditionally, before the score is even computed. The score is purely a signal logged alongside every call for monitoring:

| Range | Meaning |
|---|---|
| 0–9 | Clean SELECT |
| 10–24 | Minor exposure signals (`SELECT *`, no `LIMIT`) |
| 25–44 | Multiple exposure signals (set operations, deep subqueries, cross joins) |
| 45–79 | Dangerous function detected — already blocked |
| 80–100 | Multiple dangerous factors — already blocked |

See [Security Overview](../security/overview.md) for how the score is computed and why blocking and scoring are deliberately separate concerns.
