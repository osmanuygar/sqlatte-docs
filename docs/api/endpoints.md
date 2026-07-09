# REST API Endpoints

Auto-generated interactive docs are always available at `/docs` (Swagger) on a running instance — this page is a grouped reference of what exists and why. All endpoints are relative to your configured `app.host:app.port`.

## Core Query (unauthenticated / default widget)

| Endpoint | Method | Purpose |
|---|---|---|
| `/query` | POST | Main NL→SQL query endpoint. Requires `X-Session-ID` if `query.require_session: true` |
| `/tables` | GET | List tables in the configured database |
| `/schema/{table_name}` | GET | Column definitions for a table |
| `/schema/multiple` | POST | Column definitions for several tables at once |
| `/history`, `/history/stats` | GET | Query history and stats |
| `/history/{query_id}` | DELETE | Delete a history entry |
| `/history/clear/{session_id}` | POST | Clear a session's history |
| `/favorites` | GET / POST | List / save favorite queries |
| `/favorites/{query_id}` | DELETE | Remove a favorite |
| `/conversation/stats`, `/conversation/history/{session_id}` | GET | Conversation memory inspection |
| `/conversation/clear/{session_id}`, `/conversation/cleanup` | POST | Reset conversation state |
| `/conversation/{session_id}` | DELETE | Delete a conversation |
| `/health`, `/config`, `/ui-config` | GET | Health check, effective config, UI section visibility |
| `/reload-providers` | POST | Reload DB/LLM providers without a full restart |

## Auth Widget & MCP (`/auth/*`)

Used by the auth widget, and by the MCP server under the hood. See [MCP Overview](../mcp/overview.md) and [Security Overview](../security/overview.md).

| Endpoint | Method | Purpose |
|---|---|---|
| `/auth/login` | POST | Per-user login with DB credentials → session ID |
| `/auth/logout`, `/auth/validate` | POST | End / check a session |
| `/auth/session-info`, `/auth/stats`, `/auth/user-stats` | GET | Session and usage introspection |
| `/auth/config` | GET | Auth plugin config (non-secret) |
| `/auth/tables`, `/auth/schema/{table_name}`, `/auth/schema/multiple` | GET/POST | Scoped schema access |
| `/auth/query` | POST | Governed query endpoint — what MCP tools call, with `bypass_intent` support |
| `/auth/mcp-mask-rules` | GET | Active field-masking rules for the caller |
| `/auth/conversation/history`, `/auth/conversation/clear` | GET/POST | Per-session conversation memory |
| `/auth/token/generate` \| `/validate` \| `/revoke` | POST | [API token lifecycle](../security/tokens.md) |
| `/auth/tokens` | GET | List the caller's own tokens |
| `/auth/auto-session` | POST | Server-credential-backed session for unauthenticated widgets (if `auto_session.enabled`) |

## MCP Server

| Endpoint | Method | Purpose |
|---|---|---|
| `/mcp/sse` | GET | SSE connection for network MCP clients (requires `x-mcp-token`) |
| `/mcp/messages/` | POST | MCP protocol message channel |

Only mounted when `mcp.sse.enabled: true`. See [Network Setup](../mcp/network-setup.md).

## Admin (`/admin/*`, session-cookie gated)

| Endpoint | Method | Purpose |
|---|---|---|
| `/admin/login`, `/admin/login/json`, `/admin/logout` | GET/POST | Admin authentication |
| `/admin/config` | GET/POST | Full effective config |
| `/admin/config/{llm,database,email,scheduler,insights,ops-agent,export}` | PUT | Section-scoped config updates |
| `/admin/config/reset`, `/admin/config/history` | POST/GET | Reset to defaults / view change history |
| `/admin/config/snapshot`, `/admin/config/restore` | POST | Backup and restore config |
| `/admin/test`, `/admin/test/email` | POST | Connection tests (DB/LLM, SMTP) |
| `/admin/reload`, `/admin/info` | POST/GET | Reload providers, instance info |
| `/admin/prompts`, `/admin/prompts/update`, `/admin/prompts/reset` | GET/POST | [Runtime prompt editing](../features/runtime-prompts.md) |
| `/admin/tokens`, `/admin/token/revoke`, `/admin/token-policy`, `/admin/token/set-limit` | GET/POST | Platform-wide [token management](../security/tokens.md) |
| `/admin/mcp-mask-rules` | GET/POST/PUT/DELETE | [Field masking rules](../mcp/field-masking.md) |

## Dashboards (`/api/dashboards`)

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/dashboards` | GET | List all |
| `/api/dashboards/stats` | GET | Count + storage backend |
| `/api/dashboards/{id}` | GET / DELETE | Fetch / remove |
| `/api/dashboards/generate` | POST | Create from a favorite query |
| `/api/dashboards/{id}/refresh` | POST | Re-run and update |

See [Dashboard](../features/dashboard.md).

## Scheduled Queries (`/api/schedules`)

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/schedules` | POST / GET | Create / list |
| `/api/schedules/{id}` | GET / PUT / DELETE | Fetch, update, remove |
| `/api/schedules/{id}/toggle` | POST | Enable/disable |
| `/api/schedules/{id}/run` | POST | Run immediately |
| `/api/schedules/{id}/executions`, `/stats` | GET | History and stats |
| `/api/schedules/admin/all`, `/admin/jobs` | GET | Cross-user view (admin) |

See [Scheduled Queries](../features/schedules.md).

## Semantic Layer (`/api/semantic`)

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/semantic/entities` | GET/POST | List / create entities |
| `/api/semantic/entities/{id}` | GET/PUT/DELETE | Manage one entity |
| `/api/semantic/entities/{id}/columns`, `/columns` | GET/POST | Column display names |
| `/api/semantic/relationships` | GET/POST | JOIN relationships |
| `/api/semantic/metrics` | GET/POST | Calculated metrics |
| `/api/semantic/context` | GET | Assembled context as sent to the LLM |
| `/api/semantic/discover` | POST | Auto-discovery scan |

See [Semantic Layer](../features/semantic-layer.md).

## BigQuery Ops Console (`/ops-agent`)

| Endpoint | Method | Purpose |
|---|---|---|
| `/ops-agent/operations` | GET | List available operations |
| `/ops-agent/projects`, `/switch-project` | GET/POST | Multi-project management |
| `/ops-agent/execute` | POST | Run an operation |
| `/ops-agent/health` | GET | Health check |
| `/ops-agent/alarms` | GET/POST | List / create cost alarms |
| `/ops-agent/alarms/{id}` | PUT/DELETE | Update / remove an alarm |
| `/ops-agent/alarms/{id}/test` | POST | Run condition check on-demand |
| `/ops-agent/alarms/{id}/history`, `/history/all` | GET | Trigger history |
| `/ops-agent/alarms/jira-config`, `/jira-issue-types` | GET | Jira integration status |

See [BigQuery Ops Console](../features/ops-console.md).

## Audit Logs (`/api/audit`)

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/audit/logs` | GET | Filtered log entries |
| `/api/audit/summary`, `/daily-trend`, `/top-users` | GET | Aggregate views |
| `/api/audit/export/csv` | GET | CSV export |

See [Audit Logs](../features/audit-logs.md).

## Usage Analytics (`/api/analytics`)

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/analytics/summary`, `/hourly-stats`, `/errors` | GET | Usage overview |
| `/api/analytics/widget-comparison` | GET | Standard vs. auth widget traffic |
| `/api/analytics/performance`, `/query-complexity` | GET | Execution time and complexity distributions |
| `/api/analytics/top-users` | GET | Highest-volume users |
| `/api/analytics/health` | GET | Health check |

See [Analytics & Insights](../features/analytics.md).

## Demo (`/demo`)

| Endpoint | Method | Purpose |
|---|---|---|
| `/demo`, `/demo/fullscreen`, `/demo/standard`, `/demo/comparison` | GET | Widget embed previews |
| `/demo/health` | GET | Health check |
