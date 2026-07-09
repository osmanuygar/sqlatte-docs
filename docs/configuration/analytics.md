# Analytics Setup

`analytics.enabled` controls whether query history, performance metrics, and usage stats persist to PostgreSQL — with a SQLite-backed fallback used when it's off.

```yaml
analytics:
  enabled: false
  backend: "postgresql"
  postgresql:
    host: "localhost"
    port: 5432
    database: "sqlatte_analytics"
    user: "sqlatte"
    password: "your-password"
```

When `enabled: false`, analytics still works locally (query history, favorites) via an embedded SQLite store — no setup required, but it doesn't survive being deployed across multiple instances and isn't meant for long-term retention. Turn on PostgreSQL for anything beyond a single dev instance.

This same flag also determines [Dashboard](dashboard.md) persistence — dashboards save to PostgreSQL when analytics is enabled, and to in-memory storage (cleared on restart) otherwise.

## What Gets Tracked

Exposed via `/api/analytics/*` (see [REST Endpoints](../api/endpoints.md)):

- Query summary stats, hourly usage patterns, error rates
- Standard vs. auth widget usage comparison
- Query performance and complexity distribution
- Top users by query volume

This is distinct from [Audit Logs](../features/audit-logs.md), which track every individual LLM call (tokens, risk score, source) rather than aggregate usage patterns.

## Toggle From the Admin Panel

`ui.sections.analytics` controls whether the Analytics tab appears in the standalone dashboard — independent of whether the `analytics` feature itself is enabled.
