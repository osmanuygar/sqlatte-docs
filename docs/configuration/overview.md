# Configuration Overview

SQLatte is configured entirely through one file: `config/config.yaml`, loaded relative to the project root at startup — there's no `--config` flag or `.env` requirement, though every value supports `${ENV_VAR:default}` interpolation if you want secrets sourced from the environment instead of the file.

Only two sections are required to boot: `llm` and `database`. Everything else is optional and off by default.

## Sections at a Glance

| Section | Default | Purpose |
|---|---|---|
| `app` | — | Host, port, app name |
| `llm` | required | LLM provider + model (Anthropic / Gemini / Vertex AI), optional per-task `model_routing` |
| `database` | required | Trino / PostgreSQL / MySQL / BigQuery connection |
| `mcp.sse` | off | Network MCP transport — see [MCP Overview](../mcp/overview.md) |
| `plugins.auth` | off | Multi-tenant per-user DB connections, catalog/schema restrictions, auto-session for widgets |
| `admin` | — | Admin panel username/password, session TTL |
| `config_db` | off | Persist runtime config + audit logs to PostgreSQL instead of memory |
| `analytics` | off | Query history, performance metrics — [details](analytics.md) |
| `scheduler` | off | Cron-based recurring queries — see [Scheduled Queries](../features/schedules.md) |
| `email` | off | SMTP delivery for scheduled reports — [details](email.md) |
| `export` | — | CSV/Excel/HTML export formats and limits |
| `insights` | on | AI insights engine mode (`llm_only` / `statistical_only` / `hybrid`) |
| `ops_agent` | off | BigQuery Ops Console — see [Ops Console](../features/ops-console.md) |
| `alarms` / `jira` | off | Cost alarm scheduling + Jira ticket creation |
| `rate_limiting` | off | Per-endpoint request throttling — see [Security Overview](../security/overview.md) |
| `ui.sections` | — | Toggle visibility of Assistant / Demo / Analytics / Schedules / Dashboards / Ops / Admin tabs |
| `prompts` | — | The four editable AI prompts — see [Runtime Prompts](../features/runtime-prompts.md) |

See the [Full Config Reference](config-yaml.md) for every key, or jump to a specific area: [Database Providers](database-providers.md), [LLM Providers](llm-providers.md), [Analytics](analytics.md), [Dashboard](dashboard.md), [Email](email.md).

## Hot Reload

Most settings changed via the admin panel (`/admin`) apply immediately — no restart. Config edited directly in `config.yaml` requires a restart unless `config_db` is enabled, in which case runtime overrides persist and take precedence over the file on next load.
