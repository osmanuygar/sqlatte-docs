# Dashboard Customization

Dashboards have no dedicated config block of their own — they inherit their persistence backend from [`analytics`](analytics.md) and their visibility from `ui.sections`.

```yaml
analytics:
  enabled: true   # dashboards persist to PostgreSQL when true, in-memory (lost on restart) when false

ui:
  sections:
    dashboards: true   # show/hide the Dashboards tab in the standalone interface
```

## How Dashboards Are Created

Dashboards are generated from an existing **favorite query** (`POST /api/dashboards/generate` with a `query_id`), not built from a standalone config file — a dashboard is a re-runnable snapshot of a query plus its chart/metric-card layout. See [Dashboard Feature](../features/dashboard.md) for the full workflow: favoriting a query, promoting it to a dashboard, refreshing, and deleting.

## Persistence

| `analytics.enabled` | Storage | Survives restart |
|---|---|---|
| `false` | In-memory | No |
| `true` | PostgreSQL (`analytics.postgresql`) | Yes |

Check current storage mode via `GET /api/dashboards/stats`, which reports `"storage": "postgresql"` or `"storage": "in-memory"`.
