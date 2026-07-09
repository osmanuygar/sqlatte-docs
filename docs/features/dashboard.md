# Dashboard

Promote any favorited query into a persistent, re-runnable dashboard — chart, metric cards, and all — without writing any visualization code.

## How It Works

1. Run a query and star it as a **favorite**
2. From the favorite, click **Create Dashboard** (or `POST /api/dashboards/generate` with the `query_id`)
3. SQLatte re-runs the query, auto-generates a chart type based on the result shape (line/bar/pie), computes metric cards, and optionally attaches an AI insight
4. The dashboard is saved (PostgreSQL if [`analytics.enabled`](../configuration/dashboard.md), otherwise in-memory) and appears on the Dashboards tab

## Refresh and Delete

- **One-click refresh** (`POST /api/dashboards/{id}/refresh`) re-executes the underlying SQL and updates the chart/metrics with fresh data — the dashboard definition itself doesn't change
- **Delete** (`DELETE /api/dashboards/{id}`) removes it permanently

## Endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/dashboards` | GET | List all dashboards |
| `/api/dashboards/stats` | GET | Count + current storage backend |
| `/api/dashboards/{id}` | GET | Fetch one dashboard |
| `/api/dashboards/generate` | POST | Create from a favorite query |
| `/api/dashboards/{id}/refresh` | POST | Re-run and update |
| `/api/dashboards/{id}` | DELETE | Remove |

Charts render with Chart.js on the frontend; the backend only decides chart type and computes metric-card values from the result set.
