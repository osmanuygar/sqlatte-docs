# Analytics Setup

Track query history, performance metrics, and usage patterns.

---

## Quick Start

### Option 1: In-Memory (Default)

No setup needed - analytics work out of the box:

```yaml
analytics:
  enabled: true
  storage: "memory"              # Default
```

**Pros:** Zero configuration, fast  
**Cons:** Data lost on restart, not shared across instances

---

### Option 2: PostgreSQL (Recommended for Production)

Persistent storage with full analytics:

```yaml
analytics:
  enabled: true
  storage: "postgresql"
  
  postgresql:
    host: "localhost"
    port: 5432
    database: "sqlatte_analytics"
    user: "sqlatte"
    password: "analytics_password"
  
  query_history_days: 90         # Retention period
  execution_logs_days: 30
```

**Pros:** Persistent, shared across instances, full history  
**Cons:** Requires PostgreSQL database

---

## PostgreSQL Setup

### 1. Create Database

```bash
# Connect to PostgreSQL
psql -U postgres

# Create database and user
CREATE DATABASE sqlatte_analytics;
CREATE USER sqlatte WITH PASSWORD 'analytics_password';
GRANT ALL PRIVILEGES ON DATABASE sqlatte_analytics TO sqlatte;
```

### 2. Update Config

```yaml
analytics:
  enabled: true
  storage: "postgresql"
  postgresql:
    host: "localhost"
    port: 5432
    database: "sqlatte_analytics"
    user: "sqlatte"
    password: "analytics_password"
```

### 3. Start SQLatte

Tables are created automatically on first run:

```bash
python run.py

# Tables created:
# - query_history
# - execution_logs
# - insights_cache
# - runtime_overrides (for config management)
```

---

## What Gets Tracked

### Query History
- User questions
- Generated SQL
- Execution time
- Row count
- Success/failure
- Timestamps

### Execution Logs
- Scheduled query runs
- Email delivery status
- Error messages
- Performance metrics

### Insights Cache
- AI-generated insights
- Statistical analysis
- Trend detection

---

## Analytics Dashboard

Access analytics at: `http://localhost:8000/analytics`

**Metrics:**
- Total queries executed
- Average execution time
- Success rate
- Most queried tables
- Top users (if auth enabled)
- Query trends over time

**Charts:**
- Queries per day/hour
- Execution time distribution
- Success vs failure ratio
- Table usage heatmap

---

## Configuration Options

```yaml
analytics:
  enabled: true
  storage: "postgresql"        # or "memory"
  
  # Retention policies
  query_history_days: 90       # Keep query logs for 90 days
  execution_logs_days: 30      # Keep execution logs for 30 days
  
  # Performance
  batch_size: 100              # Batch insert size
  flush_interval_seconds: 60   # How often to flush to DB
```

---

## Insights Engine

Enable AI-powered insights:

```yaml
insights:
  enabled: true
  mode: "hybrid"               # llm_only, statistical_only, hybrid
  max_insights: 3              # Per query
  include_statistical: true    # Fallback to statistical
```

**Modes:**

| Mode | Description | Use Case |
|------|-------------|----------|
| `llm_only` | Pure AI insights | Best quality, higher cost |
| `statistical_only` | Rule-based | Fast, no LLM cost |
| `hybrid` | Best of both | Recommended (balanced) |

---

## Querying Analytics

### Via API

```bash
# Get query history
curl http://localhost:8000/analytics/history?limit=10

# Get performance metrics
curl http://localhost:8000/analytics/metrics

# Get top queries
curl http://localhost:8000/analytics/top-queries
```

### Via Database (PostgreSQL)

```sql
-- Most frequent queries
SELECT question, COUNT(*) as count
FROM query_history
WHERE created_at > NOW() - INTERVAL '7 days'
GROUP BY question
ORDER BY count DESC
LIMIT 10;

-- Average execution time by table
SELECT 
  tables,
  AVG(execution_time_ms) as avg_time,
  COUNT(*) as query_count
FROM query_history
WHERE success = true
GROUP BY tables
ORDER BY avg_time DESC;

-- Daily query volume
SELECT 
  DATE(created_at) as date,
  COUNT(*) as queries
FROM query_history
GROUP BY DATE(created_at)
ORDER BY date DESC
LIMIT 30;
```

---

## Data Retention

Automatic cleanup keeps database size manageable:

```yaml
analytics:
  query_history_days: 90       # Delete older than 90 days
  execution_logs_days: 30      # Delete older than 30 days
```

**Manual cleanup:**
```sql
-- Delete old query history
DELETE FROM query_history 
WHERE created_at < NOW() - INTERVAL '90 days';

-- Delete old execution logs
DELETE FROM execution_logs 
WHERE created_at < NOW() - INTERVAL '30 days';
```

---

## Privacy & Compliance

### Anonymization

Anonymize user data:

```yaml
analytics:
  anonymize_users: true        # Hash user IDs
  anonymize_queries: false     # Keep queries readable
```

### Disable Analytics

Completely disable analytics:

```yaml
analytics:
  enabled: false               # No tracking
```

### GDPR Compliance

Delete specific user data:

```sql
-- Delete all data for a user
DELETE FROM query_history WHERE user_id = 'user123';
DELETE FROM execution_logs WHERE user_id = 'user123';
```

---

## Performance Impact

### Memory Mode
- **CPU:** Minimal (<1%)
- **Memory:** ~10MB per 1000 queries
- **Disk:** None

### PostgreSQL Mode
- **CPU:** Minimal (<2%)
- **Network:** ~1KB per query logged
- **DB Storage:** ~100MB per 10,000 queries

---

## Monitoring Analytics DB

### Check Database Size

```sql
-- Total size
SELECT pg_size_pretty(pg_database_size('sqlatte_analytics'));

-- Table sizes
SELECT 
  tablename,
  pg_size_pretty(pg_total_relation_size(tablename::regclass)) as size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(tablename::regclass) DESC;
```

### Check Query Count

```sql
-- Total queries logged
SELECT COUNT(*) FROM query_history;

-- Queries today
SELECT COUNT(*) FROM query_history 
WHERE created_at > CURRENT_DATE;
```

---

## Troubleshooting

### Analytics Not Working

```bash
# Check config
analytics:
  enabled: true  # Must be true

# Check database connection
psql -U sqlatte -d sqlatte_analytics -h localhost

# Check logs
tail -f logs/sqlatte.log | grep analytics
```

### Database Connection Failed

```bash
# Verify credentials
# Check PostgreSQL is running
# Check firewall rules
# Test connection manually
```

### Slow Dashboard

```bash
# Add indexes
CREATE INDEX idx_query_history_created ON query_history(created_at);
CREATE INDEX idx_query_history_user ON query_history(user_id);

# Reduce retention
analytics:
  query_history_days: 30  # Instead of 90
```

---

## Best Practices

### For Development
- ✅ Use in-memory mode
- ✅ Disable insights to save LLM costs
- ✅ Short retention (7 days)

### For Production
- ✅ Use PostgreSQL storage
- ✅ Enable insights (hybrid mode)
- ✅ Set appropriate retention (30-90 days)
- ✅ Monitor database size
- ✅ Create indexes for performance

### For Compliance
- ✅ Enable anonymization
- ✅ Document data retention policy
- ✅ Provide user data deletion capability
- ✅ Encrypt database at rest

---

**Next:** [Email Settings](email.md) | [Full Config Reference](config-yaml.md)