# Scheduled Queries

Turn any favorite query into a recurring report, delivered by email on a cron schedule.

```yaml
scheduler:
  enabled: true
  timezone: "UTC"
  max_concurrent_jobs: 10
  job_timeout_seconds: 300
  keep_history_days: 30
  max_executions_per_schedule: 100
```

Requires [`email`](../configuration/email.md) configured for actual delivery — without SMTP it falls back to mock mode (schedule still runs and logs, no message sent).

## Creating a Schedule

`POST /api/schedules` from an existing favorite query:

```json
{
  "query_id": "<favorite query id>",
  "name": "Weekly Revenue Report",
  "frequency": "weekly",
  "cron_expression": "0 9 * * 1",
  "timezone": "UTC",
  "email_recipients": ["analytics-team@company.com"],
  "format": "excel"
}
```

| Field | Options |
|---|---|
| `frequency` | `daily`, `weekly`, `monthly`, `custom` (validated against `cron_expression`) |
| `format` | `csv`, `excel`, `html`, `pdf` |
| `email_recipients` | one or more, validated as email addresses |

## Endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/schedules` | POST / GET | Create / list schedules |
| `/api/schedules/{id}` | GET / PUT / DELETE | Fetch, update, or remove |
| `/api/schedules/{id}/toggle` | POST | Enable/disable without deleting |
| `/api/schedules/{id}/run` | POST | Run immediately, outside the cron trigger |
| `/api/schedules/{id}/executions` | GET | Execution history |
| `/api/schedules/{id}/stats` | GET | Run count, success rate |
| `/api/schedules/admin/all` | GET | Cross-user schedule list (admin) |
| `/api/schedules/admin/jobs` | GET | Live scheduler job state (admin) |

## AI-Generated Insights in Reports

If [insights](../configuration/config-yaml.md#insights) are enabled, the scheduled report attaches AI-generated analysis of the result set alongside the raw data — same insights engine used interactively, just embedded in the delivered file.

## Limits

`max_concurrent_jobs` caps parallel execution across all schedules; `job_timeout_seconds` kills a stuck run; `keep_history_days` and `max_executions_per_schedule` bound how much execution history is retained per schedule.
