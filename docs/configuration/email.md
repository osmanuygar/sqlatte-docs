# Email Settings

Configures SMTP delivery for [scheduled query](../features/schedules.md) reports.

```yaml
email:
  enabled: false
  provider: "smtp"
  smtp:
    host: "smtp.gmail.com"
    port: 587
    user: "your-email@gmail.com"
    password: "your-app-password"
    from_email: "your-email@gmail.com"
    from_name: "SQLatte Reports"
  templates:
    success_subject: "✅ {{schedule_name}} - {{date}}"
    failure_subject: "❌ Failed: {{schedule_name}}"
  max_emails_per_day: 1000
  max_recipients_per_email: 10
```

## Graceful Degradation

When `enabled: false`, or SMTP is unreachable, email delivery falls back to **mock mode** — reports still generate and log as if sent, but no actual message goes out. This keeps scheduled queries and dashboard tests working in dev environments without a real mail server.

## Common SMTP Hosts

| Provider | Host | Port | Notes |
|---|---|---|---|
| Gmail | `smtp.gmail.com` | 587 | Requires an [App Password](https://myaccount.google.com/apppasswords), not your regular password |
| Outlook/Office365 | `smtp.office365.com` | 587 | — |
| SendGrid | `smtp.sendgrid.net` | 587 | `user: "apikey"`, password = your SendGrid API key |
| Self-hosted/internal | — | 25 or 587 | Matches the shipped `config/config.yaml` default (`port: 25`, no auth) |

## Templates and Limits

- `success_subject`/`failure_subject` support `{{schedule_name}}` and `{{date}}` placeholders
- `max_emails_per_day` and `max_recipients_per_email` are hard caps to prevent a misconfigured schedule from spamming — exceeding them fails the send rather than partially delivering
- Export attachment formats (CSV/Excel/HTML) and size limits are controlled separately under `export` — see [Full Config Reference](config-yaml.md#scheduler-email-export)
