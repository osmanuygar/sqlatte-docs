# REST API Endpoints

Complete reference for SQLatte's REST API.

---

## Base URL

```
http://localhost:8000
```

---

## Authentication

### Standard Widget (No Auth)

No authentication required - uses backend config credentials.

```bash
curl http://localhost:8000/api/query
```

---

### Auth Widget (Session-Based)

Requires login and session ID in headers.

**1. Login:**
```bash
curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "user@company.com",
    "password": "password",
    "database_type": "trino",
    "host": "trino.company.com",
    "catalog": "hive"
  }'

# Response:
{
  "session_id": "abc123...",
  "username": "user@company.com"
}
```

**2. Use session ID:**
```bash
curl http://localhost:8000/auth/query \
  -H "X-Session-ID: abc123..."
```

---

## Core Endpoints

### Query Execution

#### POST /api/query

Execute natural language query (standard widget).

**Request:**
```bash
curl -X POST http://localhost:8000/api/query \
  -H "Content-Type: application/json" \
  -d '{
    "question": "Show me top 10 customers by revenue",
    "schema": "Table: customers\nColumns: id, name, revenue",
    "session_id": "optional-session-id"
  }'
```

**Response:**
```json
{
  "sql": "SELECT * FROM customers ORDER BY revenue DESC LIMIT 10",
  "columns": ["id", "name", "revenue"],
  "data": [
    [1, "Acme Corp", 50000],
    [2, "TechCo", 45000]
  ],
  "insights": [
    {
      "type": "trend",
      "severity": "medium",
      "message": "Top customer revenue increased 15% from last month"
    }
  ],
  "session_id": "abc123..."
}
```

---

#### POST /auth/query

Execute query with user credentials (auth widget).

**Request:**
```bash
curl -X POST http://localhost:8000/auth/query \
  -H "Content-Type: application/json" \
  -H "X-Session-ID: abc123..." \
  -d '{
    "question": "Show total sales today",
    "schema": "Table: orders..."
  }'
```

**Response:** Same as `/api/query`

---

### Table Management

#### GET /api/tables

List available tables.

**Request:**
```bash
curl http://localhost:8000/api/tables
```

**Response:**
```json
{
  "tables": ["customers", "orders", "products", "sessions"]
}
```

---

#### POST /api/schema

Get schema for specific tables.

**Request:**
```bash
curl -X POST http://localhost:8000/api/schema \
  -H "Content-Type: application/json" \
  -d '{
    "tables": ["customers", "orders"]
  }'
```

**Response:**
```json
{
  "combined_schema": "Table: customers\nColumns: id (INT), name (VARCHAR), revenue (DECIMAL)\n\nTable: orders\nColumns: order_id (INT), customer_id (INT), amount (DECIMAL), order_date (DATE)"
}
```

---

### Health Check

#### GET /health

System health status.

**Request:**
```bash
curl http://localhost:8000/health
```

**Response:**
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "database": "connected",
  "llm": "available",
  "analytics": "enabled",
  "scheduler": "running"
}
```

---

## Auth Widget Endpoints

### POST /auth/login

Authenticate user and create session.

**Request:**
```json
{
  "username": "user@company.com",
  "password": "password",
  "database_type": "trino",
  "host": "trino.company.com",
  "port": 443,
  "catalog": "hive",
  "schema": "default"
}
```

**Response:**
```json
{
  "session_id": "abc123...",
  "username": "user@company.com",
  "expires_in": 28800
}
```

---

### POST /auth/logout

End user session.

**Request:**
```bash
curl -X POST http://localhost:8000/auth/logout \
  -H "X-Session-ID: abc123..."
```

**Response:**
```json
{
  "message": "Logged out successfully"
}
```

---

### GET /auth/tables

List tables for authenticated user.

**Request:**
```bash
curl http://localhost:8000/auth/tables \
  -H "X-Session-ID: abc123..."
```

**Response:**
```json
{
  "tables": ["orders", "customers"]
}
```

---

## Schedule Endpoints

### GET /api/schedules

List all schedules.

**Request:**
```bash
curl http://localhost:8000/api/schedules
```

**Response:**
```json
{
  "schedules": [
    {
      "id": "schedule_1",
      "name": "Daily Sales Report",
      "question": "Show sales by product",
      "cron": "0 9 * * *",
      "next_run": "2025-02-12T09:00:00Z",
      "enabled": true
    }
  ]
}
```

---

### POST /api/schedules

Create new schedule.

**Request:**
```json
{
  "name": "Weekly Revenue Report",
  "question": "Show weekly revenue by region",
  "tables": ["orders", "regions"],
  "cron": "0 8 * * 1",
  "email_recipients": ["team@company.com"],
  "format": "excel"
}
```

**Response:**
```json
{
  "schedule_id": "schedule_2",
  "message": "Schedule created successfully",
  "next_run": "2025-02-17T08:00:00Z"
}
```

---

### DELETE /api/schedules/{id}

Delete schedule.

**Request:**
```bash
curl -X DELETE http://localhost:8000/api/schedules/schedule_1
```

**Response:**
```json
{
  "message": "Schedule deleted successfully"
}
```

---

## Analytics Endpoints

### GET /analytics/metrics

Query performance metrics.

**Request:**
```bash
curl http://localhost:8000/analytics/metrics
```

**Response:**
```json
{
  "total_queries": 1250,
  "avg_execution_time_ms": 145,
  "success_rate": 0.95,
  "queries_today": 87
}
```

---

### GET /analytics/history

Query history with filters.

**Request:**
```bash
curl "http://localhost:8000/analytics/history?limit=10&success=true"
```

**Response:**
```json
{
  "queries": [
    {
      "id": "query_123",
      "question": "Show top customers",
      "sql": "SELECT * FROM customers...",
      "execution_time_ms": 120,
      "success": true,
      "created_at": "2025-02-11T14:30:00Z"
    }
  ],
  "total": 1250
}
```

---

## Admin Endpoints

### POST /admin/reload

Reload configuration without restart.

**Request:**
```bash
curl -X POST http://localhost:8000/admin/reload
```

**Response:**
```json
{
  "message": "Configuration reloaded successfully",
  "changes": ["llm.temperature", "prompts.sql_generation"]
}
```

---

### GET /admin/prompts

Get all runtime prompts.

**Request:**
```bash
curl http://localhost:8000/admin/prompts
```

**Response:**
```json
{
  "prompts": {
    "intent_detection": "Analyze user question...",
    "sql_generation": "Generate SQL based on...",
    "barista_personality": "You are SQLatte...",
    "insights_generation": "Analyze results..."
  }
}
```

---

### PUT /admin/prompts/{type}

Update specific prompt.

**Request:**
```bash
curl -X PUT http://localhost:8000/admin/prompts/sql_generation \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Custom SQL generation rules..."
  }'
```

**Response:**
```json
{
  "message": "Prompt updated successfully",
  "type": "sql_generation"
}
```

---

## Error Responses

### Standard Error Format

```json
{
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": "Additional details"
}
```

---

### Common Error Codes

| Status | Code | Description |
|--------|------|-------------|
| 400 | INVALID_REQUEST | Missing or invalid parameters |
| 401 | UNAUTHORIZED | Session expired or invalid |
| 404 | NOT_FOUND | Resource not found |
| 429 | RATE_LIMIT | Too many requests |
| 500 | INTERNAL_ERROR | Server error |

---

## Rate Limiting

**Limits:**
- 100 requests/minute per IP
- 1000 requests/hour per session

**Headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1644591600
```

---

## Examples

### Complete Workflow

**1. Get tables:**
```bash
curl http://localhost:8000/api/tables
```

**2. Get schema:**
```bash
curl -X POST http://localhost:8000/api/schema \
  -H "Content-Type: application/json" \
  -d '{"tables": ["orders"]}'
```

**3. Execute query:**
```bash
curl -X POST http://localhost:8000/api/query \
  -H "Content-Type: application/json" \
  -d '{
    "question": "Show total sales today",
    "schema": "Table: orders..."
  }'
```

---

### With Authentication

**1. Login:**
```bash
SESSION=$(curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "user@company.com",
    "password": "password",
    "database_type": "trino",
    "host": "trino.company.com"
  }' | jq -r '.session_id')
```

**2. Query with session:**
```bash
curl -X POST http://localhost:8000/auth/query \
  -H "Content-Type: application/json" \
  -H "X-Session-ID: $SESSION" \
  -d '{
    "question": "Show my orders"
  }'
```

**3. Logout:**
```bash
curl -X POST http://localhost:8000/auth/logout \
  -H "X-Session-ID: $SESSION"
```

---

**Next:** [WebSocket API](websocket.md) 