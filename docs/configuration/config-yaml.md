# Full Config Reference

`config/config.yaml` is the single source of truth, loaded once at startup from a fixed path (`<project root>/config/config.yaml`). Every string value supports `${ENV_VAR}` (required) or `${ENV_VAR:default}` (optional) interpolation.

## Minimal Configuration

Only `llm` and `database` are required:

```yaml
llm:
  provider: "anthropic"
  anthropic:
    api_key: "${ANTHROPIC_API_KEY}"
    model: "claude-sonnet-4-20250514"

database:
  provider: "trino"
  trino:
    host: "${TRINO_HOST}"
    port: 443
    user: "${TRINO_USER}"
    password: "${TRINO_PASSWORD}"
    catalog: "hive"
    schema: "default"
    http_scheme: "https"
```

## app

```yaml
app:
  name: "SQLatte"
  host: "0.0.0.0"
  port: 8000
```

## llm

```yaml
llm:
  provider: "anthropic"   # anthropic | gemini | vertexai
  anthropic:
    api_key: "${ANTHROPIC_API_KEY}"
    model: "claude-sonnet-4-20250514"
    max_tokens: 4096
  gemini:
    api_key: "${GEMINI_API_KEY}"
    model: "gemini-pro"
    max_tokens: 1000
  vertexai:
    project_id: "${VERTEXAI_PROJECT_ID}"
    location: "${VERTEXAI_LOCATION:europe-west1}"
    model: "gemini-2.5-pro"
    credentials_path: "${GOOGLE_APPLICATION_CREDENTIALS}"
    # or inline for containers: credentials_json: "${VERTEXAI_CREDENTIALS_JSON}"
    model_routing:
      intent_detection: "gemini-2.5-flash"
      chat: "gemini-2.5-flash"
      sql: "gemini-2.5-pro"
      insights: "gemini-2.5-flash"
      ops_insights: "gemini-2.5-flash"
```

`model_routing` (top-level, works across providers) routes each task type to a different model/provider to balance cost and accuracy — see [LLM Providers](llm-providers.md#task-based-model-routing).

## database

```yaml
database:
  provider: "trino"   # trino | postgresql | mysql | bigquery
  trino:
    host: "${TRINO_HOST}"
    port: 443
    user: "${TRINO_USER}"
    password: "${TRINO_PASSWORD}"
    catalog: "${TRINO_CATALOG:hive}"
    schema: "${TRINO_SCHEMA:default}"
    http_scheme: "https"
  postgresql:
    host: "${POSTGRES_HOST}"
    port: 5432
    database: "${POSTGRES_DB}"
    user: "${POSTGRES_USER}"
    password: "${POSTGRES_PASSWORD}"
    schema: "${POSTGRES_SCHEMA:public}"
    min_connections: 1
    max_connections: 10
  mysql:
    host: "${MYSQL_HOST}"
    port: 3306
    database: "${MYSQL_DB}"
    user: "${MYSQL_USER}"
    password: "${MYSQL_PASSWORD}"
    autocommit: true
  bigquery:
    project_id: "${BQ_PROJECT_ID}"
    dataset: ""
    location: "US"
    credentials_path: "${GOOGLE_APPLICATION_CREDENTIALS}"
    # or: credentials_json: ""
    timeout: 300
    max_results: 10000
```

See [Database Providers](database-providers.md) for field-by-field notes.

## mcp

```yaml
mcp:
  sse:
    enabled: true   # exposes GET /mcp/sse + POST /mcp/messages/
```

See [MCP Network Setup](../mcp/network-setup.md). Local (stdio) MCP mode needs no config here — it authenticates via `SQLATTE_TOKEN`/env vars passed to `sqlatte_mcp_server.py` directly.

## admin

```yaml
admin:
  enabled: true
  username: "${ADMIN_USERNAME:admin}"
  password: "${ADMIN_PASSWORD}"
```

Fails closed if `password` is empty — see [Admin Panel Authentication](../security/tokens.md#admin-panel-authentication).

## config_db

```yaml
config_db:
  enabled: false
  type: postgresql
  postgresql:
    host: "${CONFIG_DB_HOST:localhost}"
    database: "${CONFIG_DB_NAME:sqlatte_config}"
    user: "${CONFIG_DB_USER:sqlatte}"
    password: "${CONFIG_DB_PASSWORD}"
  # Generate with: python3 -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
  encryption_key: "${CONFIG_DB_ENCRYPTION_KEY}"
```

When enabled, runtime config changes, API tokens, mask rules, and audit logs persist to PostgreSQL instead of process memory — required for anything beyond single-instance/dev use. `encryption_key` encrypts stored credentials at rest.

## plugins.auth

```yaml
plugins:
  auth:
    enabled: false
    session_ttl_minutes: 480
    max_workers: 40
    db_provider: "trino"
    db_host: "${TRINO_HOST}"
    db_port: 443
    allowed_catalogs:
      - name: "hadoop"
        allowed_schemas: ["hive"]
    allowed_db_types: ["trino"]
    auto_session:
      enabled: false
      ttl_hours: 1
      label: "auto-session"
      # allowed_origins: []
```

Multi-tenant per-user database connections. See [Security Overview](../security/overview.md#multi-tenant-auth-plugin).

## analytics

```yaml
analytics:
  enabled: false
  backend: "postgresql"
  postgresql:
    host: "${ANALYTICS_DB_HOST:localhost}"
    port: 5432
    database: "${ANALYTICS_DB_NAME:sqlatte_analytics}"
    user: "${ANALYTICS_DB_USER:sqlatte}"
    password: "${ANALYTICS_DB_PASSWORD}"
```

See [Analytics Setup](analytics.md).

## scheduler / email / export

```yaml
scheduler:
  enabled: false
  timezone: "UTC"
  max_concurrent_jobs: 10
  job_timeout_seconds: 300
  keep_history_days: 30
  max_executions_per_schedule: 100

email:
  enabled: false
  provider: "smtp"
  smtp:
    host: "${SMTP_HOST}"
    port: 587
    user: "${SMTP_USER:}"
    password: "${SMTP_PASSWORD:}"
    from_email: "${SMTP_FROM_EMAIL}"
    from_name: "SQLatte Reports"
  templates:
    success_subject: "✅ {{schedule_name}} - {{date}}"
    failure_subject: "❌ Failed: {{schedule_name}}"
  max_emails_per_day: 1000
  max_recipients_per_email: 10

export:
  formats: [csv, excel, html]
  max_rows: 10000
  max_file_size_mb: 25
  filename_template: "{{schedule_name}}_{{date}}_{{time}}.{{format}}"
```

See [Scheduled Queries](../features/schedules.md) and [Email Settings](email.md).

## insights

```yaml
insights:
  enabled: true
  mode: hybrid   # llm_only | statistical_only | hybrid
  max_insights: 3
  include_statistical: true
```

## ops_agent / alarms / jira

```yaml
ops_agent:
  enabled: false
  ai_insights: true
  ai_insights_max: 5
  provider: "bigquery_ops"
  config:
    projects:
      - project_id: "${BQ_OPS_PROJECT_ID}"
        region: "${BQ_OPS_REGION:europe-west1}"
        credentials_path: "${BQ_OPS_CREDENTIALS_PATH}"
    allowed_operations:
      - get_expensive_queries
      - forecast_monthly_costs
      - find_public_datasets
      - check_slot_saturation
      # ... see Ops Console page for the full list

alarms:
  enabled: false
  storage_path: "config/alarms.json"
  default_recipients: ["${ALARM_DEFAULT_RECIPIENT}"]

jira:
  enabled: false
  url: "${JIRA_URL}"
  email: "${JIRA_EMAIL}"
  api_token: "${JIRA_API_TOKEN}"
  project_key: "${JIRA_PROJECT_KEY:DATA}"
  issue_type: "Bug"
  priority: "High"
```

See [BigQuery Ops Console](../features/ops-console.md) for the full `allowed_operations` list and alarm behavior.

## rate_limiting

```yaml
rate_limiting:
  enabled: false
  strategy: sliding_window   # sliding_window | token_bucket | fixed_window
  key_type: session_id       # session_id | ip | user_id
  requests_per_window: 30
  window_seconds: 60
  protected_paths: ["/query", "/auth/query"]
  exclude_paths: ["/health", "/static", "/admin"]
```

## query

```yaml
query:
  default_limit: 100
  max_limit: 1000
  timeout: 300
  require_session: true   # 401 unauthenticated callers on the legacy /query endpoint
```

## cors

```yaml
cors:
  allow_origins: ["*"]
  allow_credentials: true
  allow_methods: ["*"]
  allow_headers: ["*"]
```

Restrict `allow_origins` to specific domains before embedding widgets in production.

## ui.sections

```yaml
ui:
  sections:
    assistant: true
    demo: true
    analytics: false
    schedules: true
    dashboards: true
    bigquery_ops: true
    admin: true
    file_analyzer: true
```

Toggles which tabs appear in the standalone dashboard interface, independent of whether the underlying feature is enabled.

## prompts

Four prompts — `intent_detection`, `barista_personality`, `sql_generation`, `insights_generation` (plus `ops_insights_generation` when the Ops Console is on) — are defined here as the defaults, and editable at runtime from the admin panel. See [Runtime Prompts](../features/runtime-prompts.md).
