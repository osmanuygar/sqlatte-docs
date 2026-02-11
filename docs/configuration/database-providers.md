# Database Providers

SQLatte supports four database providers. Configure one in `config.yaml`:

```yaml
database:
  provider: "postgresql"  # trino, postgresql, mysql, bigquery
```

---

## Trino

**Best for:** Distributed data lakes (Hive, Iceberg, Delta Lake)

```yaml
database:
  provider: "trino"
  trino:
    host: "trino.company.com"
    port: 443
    user: "username"
    password: "password"         # Optional
    catalog: "hive"              # hive, iceberg, delta, etc.
    schema: "default"
    http_scheme: "https"         # http or https
    verify_ssl: true
```

**Tips:**
- Use fully qualified table names: `catalog.schema.table`
- Optimize with partition filters: `WHERE dt >= '20251201'`
- Enable HTTPS in production

---

## PostgreSQL

**Best for:** Relational databases, OLTP workloads

```yaml
database:
  provider: "postgresql"
  postgresql:
    host: "localhost"
    port: 5432
    database: "mydb"
    user: "postgres"
    password: "password"
    schema: "public"             # Default schema
    pool_size: 5                 # Connection pool
    max_overflow: 10             # Extra connections
```

**Connection String Alternative:**
```yaml
database:
  provider: "postgresql"
  postgresql:
    connection_string: "postgresql://user:pass@host:5432/db"
```

**Tips:**
- Use connection pooling for performance
- Set appropriate `pool_size` for concurrent users
- Use schemas to organize tables

---

## MySQL

**Best for:** Web applications, MySQL workloads

```yaml
database:
  provider: "mysql"
  mysql:
    host: "localhost"
    port: 3306
    database: "mydb"
    user: "root"
    password: "password"
    charset: "utf8mb4"           # UTF-8 support
```

**Tips:**
- Use `utf8mb4` for full Unicode support
- Enable SSL for production: `ssl_ca`, `ssl_cert`, `ssl_key`

---

## Google BigQuery

**Best for:** Cloud data warehouse, analytics at scale

```yaml
database:
  provider: "bigquery"
  bigquery:
    project_id: "my-gcp-project"
    dataset: "analytics"
    credentials_path: "/path/to/service-account.json"
    location: "US"               # US, EU, asia-northeast1, etc.
```

### Service Account Setup

1. **Create Service Account:**
   ```bash
   # In GCP Console → IAM & Admin → Service Accounts
   # Click "Create Service Account"
   ```

2. **Grant Permissions:**
   - `BigQuery Data Viewer` - Read data
   - `BigQuery Job User` - Execute queries

3. **Download JSON Key:**
   ```bash
   # Click on service account → Keys → Add Key → JSON
   # Save to: /path/to/service-account.json
   ```

4. **Update Config:**
   ```yaml
   credentials_path: "/path/to/service-account.json"
   ```

**Tips:**
- Use backticks for table names: `` `project.dataset.table` ``
- Leverage partitioning: `WHERE _PARTITIONTIME >= '2025-01-01'`
- Monitor costs with query limits

---

## Testing Connection

Test your database connection:

```bash
# Via SQLatte
curl http://localhost:8000/health

# Via Admin Panel
http://localhost:8000/admin
# → Providers tab → Test Connection button
```

---

## Switching Providers

Change provider without code changes:

1. Edit `config.yaml`
2. Update `database.provider`
3. Add provider-specific config
4. Restart SQLatte (or use admin panel reload)

**Example - Trino to PostgreSQL:**
```yaml
# Before
database:
  provider: "trino"
  trino: {...}

# After
database:
  provider: "postgresql"
  postgresql: {...}
```

---

## Performance Tips

### Trino
- Use partition pruning: `WHERE dt >= '20251201'`
- Prefer `approx_distinct()` over `COUNT(DISTINCT)`
- Use `LIMIT` to control result size

### PostgreSQL
- Create indexes on frequently queried columns
- Use `EXPLAIN ANALYZE` for query optimization
- Increase `pool_size` for concurrent users

### MySQL
- Use `LIMIT` in queries
- Index foreign key columns
- Enable query cache if applicable

### BigQuery
- Use clustering and partitioning
- Avoid `SELECT *` on large tables
- Use `APPROX_COUNT_DISTINCT` for large cardinality

---

## Troubleshooting

### Connection Refused
```bash
# Check host and port
ping your-db-host
telnet your-db-host 5432

# Check firewall rules
# Check database is running
```

### Authentication Failed
```bash
# Verify credentials
# Check user has necessary permissions
# Check password special characters (escape in YAML)
```

### Timeout Errors
```bash
# Increase timeout in config
# Check network latency
# Optimize slow queries
```

---

**Next:** [LLM Providers](llm-providers.md) | [Full Config Reference](config-yaml.md)