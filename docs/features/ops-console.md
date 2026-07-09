# BigQuery Ops Console

Operational automation for BigQuery environments — cost, security, and performance diagnostics that would otherwise mean hand-writing `INFORMATION_SCHEMA` queries. Available at `/ops-agent`.

```yaml
ops_agent:
  enabled: true
  ai_insights: true
  ai_insights_max: 5
  provider: "bigquery_ops"
  config:
    projects:
      - project_id: "my-project"
        region: "europe-west1"
        credentials_path: "/path/to/service-account.json"
      # additional projects can be listed and switched between at runtime
    allowed_operations:
      - get_expensive_queries
      - forecast_monthly_costs
      # ... see full list below
```

This uses its own service-account credential (`ops_agent.config.projects[].credentials_path`), separate from the main [`database.bigquery`](../configuration/database-providers.md#bigquery) connection used for natural-language queries — you can scope ops/cost-analysis access differently from query access.

## Operations

Grouped by category, gated individually via `allowed_operations`:

| Category | Operations |
|---|---|
| **Cost** | `get_expensive_queries`, `forecast_monthly_costs`, `get_time_travel_consumers`, `analyze_storage_compression`, `identify_large_unpartitioned_tables`, `get_on_demand_vs_flat_rate_analysis` |
| **Security** | `find_public_datasets`, `check_table_permissions`, `audit_service_account_usage` |
| **Performance** | `get_slow_queries`, `analyze_data_skew`, `check_slot_saturation`, `get_partition_recommendations`, `get_query_errors`, `find_full_table_scans` |
| **Governance** | `find_unused_tables`, `get_recent_table_users` |

Run one via `POST /ops-agent/execute` with `{"operation": "...", "params": {...}}`, or discover the full catalog (with parameter schemas and estimated duration) via `GET /ops-agent/operations`.

## Multi-Project Support

Multiple GCP projects can be configured; switch the active one at runtime with `POST /ops-agent/switch-project` — this updates both credentials and region without a restart.

## AI Insights

When `ai_insights: true`, each operation's raw result set is optionally summarized by the LLM using the `ops_insights_generation` prompt (see [Runtime Prompts](runtime-prompts.md)) — capped at `ai_insights_max` insights per run, surfacing cost/security/performance findings and concrete next steps rather than just a data dump.

## Cost Alarms

Schedule threshold-based alarms on top of any ops operation: run `get_expensive_queries` daily, alert when `total_tb_processed > 0.5`, notify by email and/or create a Jira ticket automatically.

```yaml
alarms:
  enabled: true
  storage_path: "config/alarms.json"
  default_recipients: ["team@company.com"]

jira:
  enabled: true
  url: "${JIRA_URL}"
  email: "${JIRA_EMAIL}"
  api_token: "${JIRA_API_TOKEN}"
  project_key: "DATA"
  issue_type: "Bug"
  priority: "High"
```

An alarm definition combines an operation, a condition (`field` / `operator` / `threshold`), a cron `schedule`, recipients, and which notification channels to fire:

```json
{
  "name": "Daily cost spike check",
  "operation": "get_expensive_queries",
  "params": {"days": 1},
  "condition": {"field": "total_tb_processed", "operator": "gt", "threshold": 0.5},
  "notifications": {"email": true, "jira": false},
  "schedule": "0 8 * * *",
  "recipients": ["team@company.com"]
}
```

| Endpoint | Method | Purpose |
|---|---|---|
| `/ops-agent/alarms` | GET / POST | List / create alarms |
| `/ops-agent/alarms/{id}` | PUT / DELETE | Update / remove |
| `/ops-agent/alarms/{id}/test` | POST | Run the condition check on-demand, without waiting for the schedule |
| `/ops-agent/alarms/{id}/history` | GET | Past trigger history for one alarm |
| `/ops-agent/alarms/history/all` | GET | Trigger history across all alarms |
| `/ops-agent/alarms/jira-config`, `/jira-issue-types` | GET | Jira connection status and available issue types (no secrets exposed) |

Alarms are backed by APScheduler cron triggers and persisted to `alarms.storage_path` (JSON file by default), so they survive restarts independent of `config_db`.
