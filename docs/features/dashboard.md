# Dashboard

SQLatte's unified dashboard brings all features together in one interface.

---

## Overview

The dashboard provides a single-page interface for:
- Natural language queries
- Scheduled reports
- Analytics metrics
- Admin configuration

**Access:** `http://localhost:8000`

---

## Tabs

### 1. Home Tab

Landing page with quick links and status overview.

**Features:**
- System health indicators (DB, LLM, services)
- Recent activity summary
- Quick action buttons
- Feature highlights

---

### 2. Assistant Tab

Embedded standard widget for natural language queries.

**Features:**
- Full widget functionality
- Query history
- Favorites
- Visualizations
- Same as standalone widget

**Use:** Click "Assistant" tab and start asking questions.

---

### 3. Demo Tab

Interactive examples and tutorials.

**Features:**
- Sample queries to try
- Tutorial walkthroughs
- Feature showcase
- Example database (optional)

**Use:** Learn SQLatte by trying pre-built examples.

---

### 4. Analytics Tab

Query performance and usage metrics dashboard.

**Requires:** `analytics.enabled: true` in config

**Metrics:**
- Total queries executed
- Average execution time
- Success rate
- Top queries
- Table usage
- Insights generated

**Charts:**
- Query volume trends
- Performance over time
- Success/failure ratio
- User activity (if auth enabled)

**Use:** Monitor system performance and user behavior.

---

### 5. Schedules Tab

Manage scheduled queries and automated reports.

**Requires:** `scheduler.enabled: true` in config

**Features:**
- Create new schedules
- View all schedules
- Edit/delete schedules
- Execution history
- Email delivery status

**Use:** Automate recurring reports without manual queries.

---

### 6. Configuration Tab

Admin panel embedded in dashboard.

**Features:**
- All admin panel functionality
- Provider settings
- Prompts editor
- Email/SMTP config
- System info

**Use:** Configure SQLatte without leaving dashboard.

---

## Navigation

### Sidebar Navigation

```
┌─────────────────┐
│ ☕ SQLatte      │
├─────────────────┤
│ 🏠 Home         │
│ 💬 Assistant    │
│ 🎯 Demo         │
│ 📊 Analytics    │
│ 📅 Schedules    │
│ ⚙️  Config      │
└─────────────────┘
```

**Responsive:** Collapses to hamburger menu on mobile.

---

## User Experience

### First-Time User

1. **Home** - See what SQLatte can do
2. **Demo** - Try sample queries
3. **Assistant** - Ask your own questions
4. **Schedules** - Automate reports (optional)

### Daily User

1. **Assistant** - Quick queries
2. **Analytics** - Check performance
3. **Schedules** - Manage reports

### Admin

1. **Configuration** - Update settings
2. **Analytics** - Monitor usage
3. **Schedules** - Review automations

---

## Deployment Modes

### Standalone Dashboard

Full dashboard with all tabs:

```yaml
# config.yaml
app:
  host: 0.0.0.0
  port: 8000

# Access: http://localhost:8000
```

**Best for:** Internal tools, team analytics

---

### Widget-Only

Embed widget without dashboard:

```html
<!-- No dashboard, just widget -->
<script src="http://localhost:8000/static/js/sqlatte-badge.js"></script>
```

**Best for:** External sites, customer-facing

---

### Hybrid

Dashboard for admins, widget for users:

```
Dashboard (http://localhost:8000) → Admin access
Widget (embedded) → User access
```

**Best for:** SaaS products, enterprise

---

## Customization

### Disable Tabs

Hide unnecessary tabs:

```yaml
# config.yaml
dashboard:
  enabled_tabs:
    - home
    - assistant
    # - demo      # Disabled
    # - analytics # Disabled
    - schedules
    - config
```

---

## Tips

### Performance
- Analytics tab may be slow with large datasets
- Use filters to limit date range
- Enable PostgreSQL storage for better performance

### Workflow
- Pin frequently used queries in Assistant tab
- Use Schedules for recurring reports
- Check Analytics weekly for insights

### Organization
- Use descriptive schedule names
- Tag queries with keywords (future feature)
- Archive old schedules

---

## Troubleshooting

### Tab Not Loading

```bash
# Check feature is enabled
# Example: Analytics tab requires analytics.enabled: true

# Check logs
docker-compose logs -f sqlatte
```

### Slow Performance

```bash
# Enable PostgreSQL for analytics
analytics:
  storage: "postgresql"

# Add database indexes
# Reduce retention period
```

### Empty Dashboard

```bash
# Verify config.yaml is loaded
# Check database connection
# Review browser console for errors
```

---

**Next:** [Scheduled Queries](schedules.md) | [Analytics & Insights](analytics.md)