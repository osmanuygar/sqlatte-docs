# Scheduled Queries

Automate recurring reports with scheduled query execution and email delivery.

---

## Quick Start

### Enable Scheduler

```yaml
# config.yaml
scheduler:
  enabled: true
  timezone: "UTC"

email:
  enabled: true
  smtp:
    host: "smtp.gmail.com"
    port: 587
    user: "your-email@gmail.com"
    password: "app-password"
```

### Create First Schedule

1. Go to `http://localhost:8000` → **Schedules** tab
2. Click **"+ New Schedule"**
3. Fill in details:
   - Name: "Daily Sales Report"
   - Question: "Show total sales by product today"
   - Frequency: Daily at 9:00 AM
   - Recipients: `team@company.com`
4. Click **Save**

**Done!** Report will be emailed daily at 9 AM.

---

## Creating Schedules

### Via Dashboard

```
Schedules Tab → New Schedule → Fill Form → Save
```

**Form Fields:**

| Field | Description | Example |
|-------|-------------|---------|
| Name | Schedule name | Daily Sales Report |
| Question | Natural language query | Show sales by region |
| Tables | Select table(s) | orders, customers |
| Frequency | Cron expression | `0 9 * * *` (9 AM daily) |
| Recipients | Email addresses | john@company.com, team@company.com |
| Format | Report format | Excel, CSV, HTML |
| Subject | Email subject | Sales Report - {{date}} |

---

### Via API

```bash
curl -X POST http://localhost:8000/api/schedules \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Daily Sales Report",
    "question": "Show total sales by product today",
    "tables": ["orders"],
    "cron": "0 9 * * *",
    "email_recipients": ["team@company.com"],
    "format": "excel"
  }'
```

---

## Cron Expressions

### Common Schedules

| Schedule | Cron Expression | Description |
|----------|----------------|-------------|
| Daily 9 AM | `0 9 * * *` | Every day at 9:00 AM |
| Weekdays 8 AM | `0 8 * * 1-5` | Monday-Friday at 8 AM |
| Weekly Monday | `0 9 * * 1` | Every Monday at 9 AM |
| Monthly 1st | `0 6 1 * *` | 1st of month at 6 AM |
| Hourly | `0 * * * *` | Every hour on the hour |

### Cron Format

```
* * * * *
│ │ │ │ │
│ │ │ │ └─── Day of week (0-7, 0 and 7 = Sunday)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

**Examples:**
```bash
0 9 * * *      # Daily at 9:00 AM
30 8 * * 1     # Every Monday at 8:30 AM
0 6 1 * *      # First day of month at 6:00 AM
*/15 * * * *   # Every 15 minutes
0 9 * * 1-5    # Weekdays at 9:00 AM
```

---

## Email Reports

### Report Formats

**CSV** - Simple, universal
```yaml
format: "csv"
```
- Lightweight
- Opens in Excel/Google Sheets
- Best for data export

**Excel** - Formatted spreadsheet
```yaml
format: "excel"
```
- Professional formatting
- Multiple sheets possible
- Best for business reports

**HTML** - Embedded in email
```yaml
format: "html"
```
- No attachment needed
- Readable in email client
- Best for quick summaries

---

### Email Template

**Default subject:**
```
✅ {{schedule_name}} - {{date}}
```

**Custom subject:**
```yaml
templates:
  success_subject: "📊 {{schedule_name}} - {{date}}"
  failure_subject: "❌ Report Failed: {{schedule_name}}"
```

**Variables:**
- `{{schedule_name}}` - Schedule name
- `{{date}}` - YYYY-MM-DD
- `{{time}}` - HH:MM:SS
- `{{format}}` - csv, excel, html

---

## Managing Schedules

### View All Schedules

```
Dashboard → Schedules Tab
```

Shows:
- Schedule name
- Frequency
- Next run time
- Last run status
- Recipients
- Actions (Edit, Delete, Run Now)

---

### Edit Schedule

```
Schedules Tab → Click schedule → Edit → Save
```

**Editable:**
- Name
- Question
- Frequency (cron)
- Recipients
- Format

**Not editable:**
- Schedule ID
- Creation date

---

### Delete Schedule

```
Schedules Tab → Click schedule → Delete → Confirm
```

⚠️ **Warning:** Deletes all execution history for this schedule.

---

### Run Now

Test schedule without waiting:

```
Schedules Tab → Click schedule → "Run Now"
```

Executes immediately and sends email.

---

## Execution History

View past executions:

```
Schedules Tab → Click schedule → "History"
```

**Details:**
- Execution time
- Success/failure status
- Rows returned
- Execution duration
- Email sent status
- Error messages (if failed)

**Retention:**
```yaml
scheduler:
  keep_history_days: 30  # Keep logs for 30 days
```

---

## Configuration

### Full Configuration

```yaml
scheduler:
  enabled: true
  timezone: "America/New_York"     # IANA timezone
  max_concurrent_jobs: 10          # Parallel executions
  job_timeout_seconds: 300         # 5 minutes per job
  keep_history_days: 30            # Log retention

email:
  enabled: true
  smtp:
    host: "smtp.gmail.com"
    port: 587
    user: "your-email@gmail.com"
    password: "app-password"
    from_email: "noreply@company.com"
    from_name: "SQLatte Reports"
    use_tls: true
  
  templates:
    success_subject: "✅ {{schedule_name}} - {{date}}"
    failure_subject: "❌ Failed: {{schedule_name}}"
  
  max_emails_per_day: 1000
  max_recipients_per_email: 10

export:
  formats: [csv, excel, html]
  max_rows: 1000                   # Limit per report
  max_file_size_mb: 25             # Attachment limit
```

---

## Best Practices

### Scheduling
- ✅ Use descriptive names
- ✅ Test with "Run Now" before scheduling
- ✅ Avoid peak hours for heavy queries
- ✅ Spread schedules throughout the day
- ⚠️ Don't schedule too frequently (respect DB load)

### Email
- ✅ Use distribution lists (team@company.com)
- ✅ Keep recipient count reasonable (<10)
- ✅ Choose appropriate format (CSV for data, HTML for summaries)
- ✅ Include date in subject line
- ⚠️ Don't spam users with too many reports

### Queries
- ✅ Use LIMIT to control result size
- ✅ Add date filters (e.g., "today", "this week")
- ✅ Test queries manually first
- ⚠️ Avoid expensive aggregations in hourly schedules

---

## Examples

### Daily Sales Summary

```yaml
Name: Daily Sales Summary
Question: Show total sales by product category today
Tables: orders, products
Cron: 0 9 * * *          # 9 AM daily
Recipients: sales@company.com
Format: excel
```

---

### Weekly KPI Report

```yaml
Name: Weekly KPI Dashboard
Question: Show weekly sales, top customers, and product performance
Tables: orders, customers, products
Cron: 0 8 * * 1          # Monday 8 AM
Recipients: executives@company.com
Format: html
```

---

### Monthly Revenue Report

```yaml
Name: Monthly Revenue Report
Question: Show monthly revenue by region and product line
Tables: orders, regions
Cron: 0 6 1 * *          # 1st of month 6 AM
Recipients: finance@company.com
Format: excel
```

---

## Troubleshooting

### Schedule Not Running

```bash
# Check scheduler is enabled
scheduler:
  enabled: true

# Check cron expression
# Use crontab.guru to validate

# Check logs
docker-compose logs -f sqlatte | grep scheduler
```

### Email Not Sent

```bash
# Check email is enabled
email:
  enabled: true

# Test SMTP settings
Dashboard → Email & SMTP → Send Test Email

# Check email logs
SELECT * FROM execution_logs WHERE email_sent = false;
```

### Query Failed

```bash
# Check execution history for error
# Test query manually in Assistant tab
# Verify tables still exist
# Check database permissions
```

### Too Many Emails

```bash
# Reduce frequency
# Combine multiple schedules
# Use distribution lists instead of individual emails
```

---

## Monitoring

### Via Dashboard

```
Schedules Tab → View execution history
Analytics Tab → Schedule performance metrics
```

### Via Database

```sql
-- Failed executions today
SELECT schedule_name, error_message
FROM execution_logs
WHERE execution_time > CURRENT_DATE
  AND success = false;

-- Most executed schedules
SELECT schedule_name, COUNT(*) as executions
FROM execution_logs
WHERE execution_time > NOW() - INTERVAL '30 days'
GROUP BY schedule_name
ORDER BY executions DESC;
```

---

## Limitations

- ⚠️ **Single instance only** - Schedules run on one SQLatte instance
- ⚠️ **No distributed execution** - Not shared across multiple instances
- ⚠️ **In-memory jobs** - Restart clears pending jobs
- ⚠️ **Email rate limits** - Respect SMTP provider limits

**Roadmap:**
- Redis-backed job queue
- Distributed scheduler
- Webhook notifications
- Slack/Teams integration

---

**Next:** [Analytics & Insights](analytics.md) | [Email Settings](../configuration/email.md)