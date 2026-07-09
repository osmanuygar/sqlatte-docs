# Runtime Prompts

Every prompt that drives SQLatte's AI behavior is editable from the admin panel, in plain text, with no code changes or restart — edits apply immediately to all sessions.

## The Prompts

| Prompt | Controls |
|---|---|
| `intent_detection` | Whether a message is a data question (→ generate SQL) or general chat |
| `barista_personality` | Tone and style of non-SQL conversational replies — see [Barista Personality](barista.md) |
| `sql_generation` | The actual SQL-writing rules: JOIN strategy, partition-column filtering, LIMIT defaults, dialect quirks |
| `insights_generation` | How AI insights are derived from query results — see [Analytics & Insights](analytics.md) |
| `ops_insights_generation` | Same idea, scoped to BigQuery Ops Console findings — only relevant if [`ops_agent`](ops-console.md) is enabled |

Defaults for all five live in `config.yaml` under `prompts:` and are used until overridden at runtime.

## Editing

`/admin` → **Prompts**, or directly via API:

| Endpoint | Method | Purpose |
|---|---|---|
| `/admin/prompts` | GET | Current effective prompts (override or default) |
| `/admin/prompts/update` | POST | Save a new prompt for one or more of the five types |
| `/admin/prompts/reset` | POST | Revert to the shipped default |

## Example: SQL Generation Rules

The shipped default already encodes performance rules specific to Trino/partitioned tables — e.g. always filtering on a `dt` (`YYYYMMDD`) partition column when present, using explicit `JOIN` syntax, and capping unbounded results with `LIMIT`. If your schema uses a different partitioning convention, edit `sql_generation` directly rather than trying to work around it with follow-up questions — it applies to every query going forward.

## Persistence and Hot Reload

Overrides persist to `config_db` if enabled (survives restarts, shared across instances); otherwise they live in memory for the current process. Either way, changes take effect on the very next LLM call — no cache to invalidate, no deploy step.
