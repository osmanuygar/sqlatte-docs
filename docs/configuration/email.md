# Email Settings

Configure email delivery for scheduled reports and notifications.

---

## Quick Start

### Gmail Setup (Recommended for Testing)

```yaml
email:
  enabled: true
  
  smtp:
    host: "smtp.gmail.com"
    port: 587
    user: "your-email@gmail.com"
    password: "your-app-password"        # NOT your Gmail password!
    from_email: "your-email@gmail.com"
    from_name: "SQLatte Reports"
    use_tls: true
```

### Get Gmail App Password

1. Enable 2FA on your Gmail account
2. Visit [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)
3. Select **Mail** and your device
4. Click **Generate**
5. Copy the 16-character password
6. Use in `config.yaml` (no spaces)

---

## SMTP Providers

### Gmail

```yaml
smtp:
  host: "smtp.gmail.com"
  port: 587                              # TLS
  use_tls: true
```

**Limits:** 500 emails/day (free), 2,000/day (Workspace)

---

### SendGrid

```yaml
smtp:
  host: "smtp.sendgrid.net"
  port: 587
  user: "apikey"                         # Literally "apikey"
  password: "SG.xxxxx"                   # Your SendGrid API key
  use_tls: true
```

**Get API Key:**
1. Sign up at [sendgrid.com](https://sendgrid.com)
2. Settings → API Keys → Create API Key
3. Copy and use as password

**Limits:** 100 emails/day (free), unlimited (paid)

---

### Amazon SES

```yaml
smtp:
  host: "email-smtp.us-east-1.amazonaws.com"
  port: 587
  user: "AKIAXXXXX"                      # SMTP username
  password: "xxxxx"                      # SMTP password
  use_tls: true
```

**Setup:**
1. Enable SES in AWS Console
2. Verify sender email/domain
3. Create SMTP credentials
4. Move out of sandbox (for production)

**Limits:** 62,000 emails/month (free tier)

---

### Microsoft 365

```yaml
smtp:
  host: "smtp.office365.com"
  port: 587
  user: "your-email@company.com"
  password: "your-password"
  use_tls: true
```

**Limits:** 10,000 emails/day

---

### Custom SMTP Server

```yaml
smtp:
  host: "mail.company.com"
  port: 587                              # 25, 465, 587
  user: "username"
  password: "password"
  from_email: "noreply@company.com"
  from_name: "Company Analytics"
  use_tls: true                          # or false for port 25
```

---

## Email Configuration

### Complete Configuration

```yaml
email:
  enabled: true
  
  smtp:
    host: "smtp.gmail.com"
    port: 587
    user: "your-email@gmail.com"
    password: "app-password"
    from_email: "noreply@company.com"    # Sender email
    from_name: "SQLatte Reports"          # Sender name
    use_tls: true
  
  # Email templates
  templates:
    success_subject: "✅ {{schedule_name}} - {{date}}"
    failure_subject: "❌ Failed: {{schedule_name}}"
  
  # Rate limiting
  max_emails_per_day: 1000               # Daily limit
  max_recipients_per_email: 10           # Per email
```

### Template Variables

Available in email subjects/bodies:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{schedule_name}}` | Schedule name | Daily Sales Report |
| `{{date}}` | Date (YYYY-MM-DD) | 2025-02-11 |
| `{{time}}` | Time (HH:MM:SS) | 09:00:00 |
| `{{format}}` | File format | csv, excel, html |

---

## Export Formats

Configure report file formats:

```yaml
export:
  formats:
    - csv
    - excel
    - html
  
  max_rows: 1000                         # Limit rows per export
  max_file_size_mb: 25                   # Attachment size limit
  
  # Filename template
  filename_template: "{{schedule_name}}_{{date}}_{{time}}.{{format}}"
```

**Example filename:** `Daily_Sales_2025-02-11_09-00-00.xlsx`

---

## Testing Email

### Test via Admin Panel

```
http://localhost:8000/admin
→ Email & SMTP tab
→ Fill in SMTP settings
→ Click "Send Test Email"
→ Check inbox
```

### Test via API

```bash
curl -X POST http://localhost:8000/admin/test-email \
  -H "Content-Type: application/json" \
  -d '{
    "to": "test@example.com",
    "subject": "Test Email",
    "body": "This is a test"
  }'
```

---

## Mock Mode (No Email)

Test without sending actual emails:

```yaml
email:
  enabled: false                         # Emails logged but not sent
```

**Output in logs:**
```
📧 Mock: Would send to john@example.com
📧 Subject: Daily Sales Report - 2025-02-11
📧 Attachment: report.xlsx (2.5MB)
```

**Use case:** Development, testing, debugging

---

## Email Security

### Use Environment Variables

Don't store passwords in config.yaml:

```bash
# .env file
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password
```

```yaml
# config.yaml
smtp:
  user: "${SMTP_USER}"
  password: "${SMTP_PASSWORD}"
```

### TLS/SSL

Always use encryption in production:

```yaml
smtp:
  port: 587                              # TLS (recommended)
  use_tls: true

  # OR for SSL
  port: 465
  use_ssl: true
```

**Never use port 25 without encryption in production!**

---

## Troubleshooting

### Authentication Failed

```bash
# Error: 535 Authentication failed
# - Verify username/password
# - For Gmail: Use App Password (not account password)
# - For SendGrid: Use "apikey" as username
```

### Connection Refused

```bash
# Error: Connection refused
# - Check host and port
# - Verify firewall allows outbound SMTP
# - Test with telnet: telnet smtp.gmail.com 587
```

### TLS/SSL Error

```bash
# Error: SSL handshake failed
# - Use port 587 with use_tls: true
# - Or port 465 with use_ssl: true
# - Don't mix TLS and SSL settings
```

### Rate Limit Exceeded

```bash
# Error: Too many emails
# - Check provider limits (Gmail: 500/day)
# - Reduce max_emails_per_day in config
# - Upgrade to paid plan
```

### Email Not Received

```bash
# Check spam folder
# Verify sender email is verified (SES, SendGrid)
# Check email logs in admin panel
# Test with simple email first
```

---

## Best Practices

### For Development
- ✅ Use Gmail with app password
- ✅ Enable mock mode for testing
- ✅ Send to your own email
- ✅ Use small `max_recipients_per_email`

### For Production
- ✅ Use dedicated email service (SendGrid, SES)
- ✅ Verify sender domain (SPF, DKIM, DMARC)
- ✅ Store credentials in environment variables
- ✅ Enable TLS/SSL
- ✅ Monitor email delivery rates
- ✅ Set appropriate rate limits

### For Compliance
- ✅ Include unsubscribe link
- ✅ Add company contact info
- ✅ Follow CAN-SPAM Act / GDPR
- ✅ Don't send to unverified emails

---

## Email Templates

### Customize Email Content

Create custom email templates:

```yaml
templates:
  success_subject: "📊 {{schedule_name}} Ready"
  success_body: |
    Hi there,
    
    Your report "{{schedule_name}}" is ready!
    
    Generated: {{date}} at {{time}}
    
    Please find the {{format}} file attached.
    
    Best regards,
    SQLatte Team
  
  failure_subject: "⚠️ Report Failed: {{schedule_name}}"
  failure_body: |
    Hi there,
    
    Unfortunately, your report "{{schedule_name}}" failed to generate.
    
    Error: {{error_message}}
    
    Please check your query and try again.
    
    Best regards,
    SQLatte Team
```

---

## Advanced Configuration

### Multiple Recipients

```yaml
# In schedule configuration
recipients:
  - team@company.com
  - manager@company.com
  - analytics@company.com
```

**Note:** Limited by `max_recipients_per_email`

### Custom Headers

```yaml
smtp:
  custom_headers:
    X-Mailer: "SQLatte v1.0"
    X-Priority: "1"                      # High priority
```

### Reply-To Address

```yaml
smtp:
  from_email: "noreply@company.com"
  reply_to: "support@company.com"        # Where replies go
```

---

## Monitoring Email Delivery

### Via Admin Panel

```
http://localhost:8000/admin
→ Scheduler tab
→ View execution history
→ Check "Email Sent" status
```

### Via Database (if analytics enabled)

```sql
SELECT 
  schedule_name,
  execution_time,
  email_sent,
  email_error
FROM execution_logs
WHERE execution_time > NOW() - INTERVAL '7 days'
ORDER BY execution_time DESC;
```

---

## Cost Optimization

| Provider | Free Tier | Paid |
|----------|-----------|------|
| Gmail | 500/day | 2,000/day (Workspace) |
| SendGrid | 100/day | $15/mo (40,000/mo) |
| Amazon SES | 62,000/mo | $0.10/1,000 emails |
| Mailgun | 5,000/mo | $35/mo (50,000/mo) |

**Recommendation:** Start with Gmail, upgrade to SendGrid/SES for production.

---

**Next:** [Scheduled Queries](../features/schedules.md) | [Full Config Reference](config-yaml.md)