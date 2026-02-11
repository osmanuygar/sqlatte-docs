# Analytics & Insights

AI-powered insights and usage analytics for data-driven decision making.

---

## Overview

SQLatte provides two analytics systems:

1. **Query Analytics** - Track usage, performance, history
2. **Insights Engine** - AI-powered data analysis

---

## Insights Engine

### What It Does

Automatically analyzes query results and generates actionable insights.

**Example Query:** "Show daily sales last 7 days"

**Insights Generated:**
- 📈 **Trend:** Sales increased 15% over the week
- ⚠️ **Anomaly:** Thursday had unusually low sales (holiday)
- 💡 **Recommendation:** Focus marketing on slow days

---

### Enable Insights

```yaml
insights:
  enabled: true
  mode: "hybrid"              # llm_only, statistical_only, hybrid
  max_insights: 3             # Per query
  include_statistical: true
```

---

### Insight Modes

**LLM Only** - Pure AI analysis
```yaml
mode: "llm_only"
```
- ✅ Creative, context-aware
- ✅ Natural language explanations
- ⚠️ Higher LLM cost
- ⚠️ Slower (1-2 seconds)

**Statistical Only** - Rule-based
```yaml
mode: "statistical_only"
```
- ✅ Fast (<100ms)
- ✅ No LLM cost
- ✅ Consistent results
- ⚠️ Less context-aware

**Hybrid** - Best of both (Recommended)
```yaml
mode: "hybrid"
```
- ✅ AI + statistical analysis
- ✅ Balanced cost/quality
- ✅ Fallback to statistical if LLM fails

---

### Insight Types

**Trend** - Patterns over time
```
📈 Sales increased 25% compared to last month
```

**Anomaly** - Unexpected values
```
⚠️ Login attempts 10x higher than average (possible attack)
```

**Summary** - Key takeaways
```
📊 Top 3 products account for 60% of revenue
```

**Recommendation** - Action items
```
💡 Consider restocking Product A (low inventory, high demand)
```

---

## Query Analytics

### What Gets Tracked

**Query History:**
- User questions
- Generated SQL
- Execution time
- Rows returned
- Success/failure
- Timestamp

**Performance Metrics:**
- Average execution time
- Query volume
- Success rate
- Popular tables
- User activity

---

### Enable Analytics

```yaml
analytics:
  enabled: true
  storage: "postgresql"       # or "memory"
  
  postgresql:
    host: "localhost"
    port: 5432
    database: "sqlatte_analytics"
    user: "sqlatte"
    password: "password"
  
  query_history_days: 90      # Retention
```

---

### Analytics Dashboard

**Access:** `http://localhost:8000/analytics`

**Metrics Cards:**
- Total queries executed
- Average execution time
- Success rate (%)
- Active users (if auth enabled)

**Charts:**
- Queries per day/hour
- Execution time trends
- Top 10 queries
- Table usage heatmap
- Success vs failure ratio

---

### Example Queries

**Most popular queries:**
```sql
SELECT question, COUNT(*) as count
FROM query_history
WHERE created_at > NOW() - INTERVAL '7 days'
GROUP BY question
ORDER BY count DESC
LIMIT 10;
```

**Slow queries:**
```sql
SELECT question, sql, execution_time_ms
FROM query_history
WHERE execution_time_ms > 5000
ORDER BY execution_time_ms DESC
LIMIT 20;
```

**Daily query volume:**
```sql
SELECT 
  DATE(created_at) as date,
  COUNT(*) as queries,
  AVG(execution_time_ms) as avg_time
FROM query_history
GROUP BY DATE(created_at)
ORDER BY date DESC
LIMIT 30;
```

---

## Insight Examples

### Sales Analysis

**Query:** "Show daily sales last 30 days"

**Insights:**
```
📈 Trend: Sales growing 12% month-over-month
⚠️ Anomaly: Weekend sales 40% lower than weekdays
💡 Recommendation: Run weekend promotions to boost sales
```

---

### User Behavior

**Query:** "Show login failures by IP today"

**Insights:**
```
⚠️ Anomaly: IP 192.168.1.100 has 500 failed attempts (possible brute force)
📊 Summary: 95% of failures from 3 IPs
💡 Recommendation: Block suspicious IPs, enable rate limiting
```

---

### Inventory Management

**Query:** "Show products with low stock"

**Insights:**
```
⚠️ Anomaly: 15 products below reorder threshold
💡 Recommendation: Restock top 5 sellers immediately (high demand)
📊 Summary: $50K potential lost revenue if out of stock
```

---

## Configuration

### Full Configuration

```yaml
insights:
  enabled: true
  mode: "hybrid"
  max_insights: 3
  include_statistical: true
  
  # Advanced settings
  min_confidence: 0.7         # Filter low-confidence insights
  context_window: 100         # Rows to analyze
  llm_timeout: 10             # Seconds

analytics:
  enabled: true
  storage: "postgresql"
  
  postgresql:
    host: "postgres"
    port: 5432
    database: "sqlatte_analytics"
    user: "sqlatte"
    password: "${ANALYTICS_PASSWORD}"
  
  # Retention
  query_history_days: 90
  execution_logs_days: 30
  
  # Performance
  batch_size: 100
  flush_interval_seconds: 60
```

---

## Best Practices

### Insights
- ✅ Enable hybrid mode for best results
- ✅ Set `max_insights: 3` to avoid overwhelming users
- ✅ Review insights for accuracy before relying on them
- ⚠️ Don't use insights as sole decision-making tool

### Analytics
- ✅ Use PostgreSQL storage for production
- ✅ Set appropriate retention (30-90 days)
- ✅ Monitor database size regularly
- ✅ Create indexes for performance
- ⚠️ Don't track sensitive data (enable anonymization if needed)

### Performance
- ✅ Use statistical mode for real-time dashboards
- ✅ Cache insights for frequently-run queries
- ✅ Limit context window for large result sets
- ⚠️ LLM insights add 1-2 seconds to response time

---

## Privacy & Compliance

### Anonymize Data

```yaml
analytics:
  anonymize_users: true       # Hash user IDs
  anonymize_queries: false    # Keep queries readable
  anonymize_results: false    # Keep results readable
```

### GDPR Compliance

**Delete user data:**
```sql
DELETE FROM query_history WHERE user_id = 'user123';
DELETE FROM execution_logs WHERE user_id = 'user123';
```

**Export user data:**
```sql
SELECT * FROM query_history 
WHERE user_id = 'user123'
ORDER BY created_at DESC;
```

---

## Troubleshooting

### No Insights Generated

```bash
# Check insights are enabled
insights:
  enabled: true

# Check LLM provider is working
curl http://localhost:8000/health

# Check logs for errors
docker-compose logs -f sqlatte | grep insights
```

### Analytics Dashboard Empty

```bash
# Verify analytics enabled
analytics:
  enabled: true

# Check database connection
docker-compose exec postgres psql -U sqlatte -d sqlatte_analytics

# Check tables exist
\dt

# Check data exists
SELECT COUNT(*) FROM query_history;
```

### Slow Performance

```bash
# Add database indexes
CREATE INDEX idx_query_history_created ON query_history(created_at);
CREATE INDEX idx_query_history_user ON query_history(user_id);

# Reduce retention
analytics:
  query_history_days: 30  # Instead of 90

# Use statistical mode only
insights:
  mode: "statistical_only"
```

---

## Monitoring

### Database Size

```sql
-- Total size
SELECT pg_size_pretty(pg_database_size('sqlatte_analytics'));

-- Table sizes
SELECT 
  tablename,
  pg_size_pretty(pg_total_relation_size(tablename::regclass))
FROM pg_tables
WHERE schemaname = 'public';
```

### Query Volume

```sql
-- Today's queries
SELECT COUNT(*) FROM query_history 
WHERE created_at > CURRENT_DATE;

-- Last 7 days by day
SELECT 
  DATE(created_at) as date,
  COUNT(*) as queries
FROM query_history
WHERE created_at > CURRENT_DATE - INTERVAL '7 days'
GROUP BY DATE(created_at);
```

---

## Limitations

**Insights Engine:**
- ⚠️ Works best with time-series data
- ⚠️ Limited to numeric/date columns for trends
- ⚠️ May miss complex patterns
- ⚠️ Requires sufficient data (minimum 5-10 rows)

**Analytics:**
- ⚠️ PostgreSQL required for persistence
- ⚠️ Not shared across multiple instances (use shared DB)
- ⚠️ No real-time streaming (batch updates)

---

**Next:** [Runtime Prompts](runtime-prompts.md) | [Configuration](../configuration/analytics.md)