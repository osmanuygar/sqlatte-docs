# Analytics & Insights

Two related but distinct capabilities: usage **analytics** (what's being queried, by whom, how fast) and the AI **insights engine** (what the data itself is telling you).

## Usage Analytics

Requires [`analytics.enabled: true`](../configuration/analytics.md). Exposed via `/api/analytics/*`:

| Endpoint | Reports |
|---|---|
| `/summary` | Total queries, success rate, overview stats |
| `/hourly-stats` | Query volume by hour |
| `/errors` | Failed queries and error patterns |
| `/widget-comparison` | Standard widget vs. auth widget usage |
| `/performance` | Query execution time distribution |
| `/top-users` | Highest-volume users/sessions |
| `/query-complexity` | JOIN count, subquery depth distribution |

This tracks *usage patterns*. For per-call token cost and risk scoring, see [Audit Logs](audit-logs.md) instead — the two are separate systems (analytics is aggregate/optional, audit logging runs for every LLM call regardless of the `analytics` flag).

## AI Insights Engine

Runs after query execution to surface patterns a user might not think to ask about:

```yaml
insights:
  enabled: true
  mode: hybrid          # llm_only | statistical_only | hybrid
  max_insights: 3
  include_statistical: true
```

| Mode | Behavior |
|---|---|
| `llm_only` | Insight generation delegated entirely to the LLM (uses the `insights_generation` prompt) |
| `statistical_only` | Pure statistical analysis (trend/anomaly detection), no LLM call — cheaper, faster |
| `hybrid` | Statistical signals feed into the LLM prompt for richer, still-grounded insights |

Insights are context-aware: they account for temporal patterns (daily/weekly/monthly cycles) and flag when a query touches incomplete real-time data (e.g. "today" partition still filling in) rather than misreading a partial day as a trend.

Each insight carries a `type` (`trend` / `anomaly` / `recommendation` / `summary`) and `severity` (`low` / `medium` / `high`), and the prompt driving generation is fully editable — see [Runtime Prompts](runtime-prompts.md).
