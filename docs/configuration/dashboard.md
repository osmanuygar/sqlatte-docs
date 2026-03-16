# 📊 Dashboard Feature

SQLatte's Dashboard feature automatically generates visual reports from your favorite queries — with charts, metric cards, and AI insights — in a single click.

---

## Overview

Dashboards are built on top of **Favorite Queries**. Once a query is starred, it can be promoted to a full dashboard with:

- 📈 **Auto-generated charts** (line, bar, pie — based on column types)
- 📐 **Metric cards** (sum, avg, min, max per numeric column)
- 💡 **AI Insights** (if Insights Engine is enabled)
- 📋 **Sortable data table** with search
- 🔗 **Shareable link** per dashboard
- 🔄 **One-click data refresh** (re-runs the SQL live)

---

## Prerequisites

Before generating a dashboard, a query must be saved to **Favorites**:

1. Ask a question in the SQLatte widget
2. When results appear, click the ⭐ star icon to save it as a favorite
3. Optionally give it a custom name (this becomes the dashboard title)

---

## Generating a Dashboard

### Via the UI

1. Navigate to `http://localhost:8000` → **Dashboards** tab
2. Click **✨ New Dashboard**
3. Select a favorite query from the list
4. (Optional) Enter a custom title
5. Click **✨ Generate**

The dashboard opens automatically after creation.

### Via API

```http
POST /api/dashboards/generate
Content-Type: application/json

{
  "query_id": "<favorite_query_id>",
  "title": "My Dashboard",       // optional
  "description": "Daily report"  // optional
}
```

**Title resolution priority:**
1. `title` field in the request body
2. Custom name set when starring the query
3. First 60 characters of the natural language question
4. Fallback: `"Dashboard"`

---

## Dashboard Page

Each dashboard is available at:

```
http://localhost:8000/dashboard.html?id=<dashboard_id>
```

### Sections

| Section | Description |
|---|---|
| **Insights** | AI-generated observations (if insights enabled) |
| **Metric Cards** | Key numeric summaries (sum / avg / min / max) |
| **Charts** | Auto-selected chart types based on data shape |
| **Data Table** | Full result set with sorting and search |
| **SQL** | The underlying SQL query for transparency |

### Header Actions

| Button | Action |
|---|---|
| 🔗 Share Link | Copy the public URL to clipboard |
| 🔄 Refresh Data | Re-run SQL and regenerate charts + insights |
| 🗑️ Delete | Permanently remove the dashboard |

---

## Chart Types

Charts are auto-selected based on column analysis:

| Data Shape | Chart Type |
|---|---|
| Date/time column + numeric | Line chart |
| Categorical column + numeric (≤ 10 categories) | Pie / Doughnut |
| Categorical column + numeric (> 10 categories) | Bar chart |
| Single-row aggregate (SUM, COUNT, AVG…) | Grouped bar (Metrics Overview) |

All charts use Chart.js with the SQLatte dark coffee theme.

---

## Storage

| Mode | Condition | Behaviour |
|---|---|---|
| **PostgreSQL** | `analytics.enabled: true` | Dashboards persisted in `dashboards` table, survive restarts |
| **In-memory** | `analytics.enabled: false` | Dashboards lost on server restart |

> **Recommended:** Enable analytics + PostgreSQL for production use.

Cached data is capped at **500 rows** per dashboard to keep storage efficient. The full result set is still shown in the data table (fetched live on refresh).

---

## Configuration

### `config/config.yaml`

#### Analytics (required for persistence)

```yaml
# ============================================
# ANALYTICS — enables PostgreSQL persistence
# ============================================
analytics:
  enabled: true          # false = in-memory only (dashboards lost on restart)
  postgresql:
    host: "localhost"
    port: 5432
    database: "sqlatte_analytics"
    user: "postgres"
    password: "your_password"
```

#### Insights Engine (optional — enriches dashboards with AI observations)

```yaml
# ============================================
# INSIGHTS ENGINE — AI-generated observations
# ============================================
insights:
  enabled: true                 # false = dashboards skip the insights section
  mode: hybrid                  # Options: llm_only | statistical_only | hybrid
  max_insights: 3               # Max number of insight bullets per dashboard
  include_statistical: true     # Fallback to statistical analysis when LLM is unavailable
```

#### Query limits (affects data available to dashboards)

```yaml
# ============================================
# QUERY SETTINGS
# ============================================
query:
  default_limit: 100    # Rows returned per query (used when generating dashboard data)
  max_limit: 1000
  timeout: 300
```

> **Tip:** Increase `default_limit` or add explicit `LIMIT` to your SQL if you want more data points in charts.

---

## API Reference

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/dashboards` | List all dashboards (summary cards) |
| `GET` | `/api/dashboards/stats` | Storage stats and total count |
| `GET` | `/api/dashboards/{id}` | Get full dashboard (charts, data, insights) |
| `POST` | `/api/dashboards/generate` | Generate dashboard from a favorite query |
| `POST` | `/api/dashboards/{id}/refresh` | Re-run SQL and regenerate everything |
| `DELETE` | `/api/dashboards/{id}` | Delete a dashboard |

---

## Troubleshooting

**Dashboard shows no charts**
- Ensure the query returns at least one numeric column alongside a string or date column.
- Pure `SELECT *` on wide tables with only text columns will skip chart generation.

**"Favorite query not found" error**
- The `query_id` must match an entry in Favorites. Re-star the query if needed.

**Non-serializable value error (date, Decimal, bytes)**
- The Dashboard Manager sanitizes these automatically. If you see this error, check server logs for the exact column causing the issue.

**Dashboards disappear after restart**
- Analytics persistence is not enabled. Set `analytics.enabled: true` in `config.yaml` and configure the PostgreSQL connection.

**Insights section is empty**
- Verify `insights.enabled: true` in `config.yaml`.
- Check that the LLM provider is reachable and the API key is valid.