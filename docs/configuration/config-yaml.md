# Complete config.yaml Reference

This page provides a comprehensive reference for all configuration options in SQLatte's `config.yaml` file.

!!! tip "Quick Start"
    Looking for a minimal configuration? Jump to [Minimal Configuration](#minimal-configuration) section.

## File Location

SQLatte looks for the configuration file in these locations (in order):

1. Path specified via `--config` argument: `python run.py --config /path/to/config.yaml`
2. Environment variable: `export SQLATTE_CONFIG=/path/to/config.yaml`
3. Default location: `config/config.yaml`

## Complete Configuration Template

Here's the complete `config.yaml` structure with all available options:

```yaml
# ============================================
# APPLICATION SETTINGS
# ============================================
app:
  name: "SQLatte"
  version: "1.0.0"
  host: "0.0.0.0"              # Listen address (0.0.0.0 = all interfaces)
  port: 8000                   # Port number
  debug: false                 # Debug mode (set true for development only)
  log_level: "INFO"            # Logging level: DEBUG, INFO, WARNING, ERROR, CRITICAL

# ============================================
# CORS SETTINGS
# ============================================
cors:
  allow_origins:
    - "*"                      # Allowed origins (* = all, restrict in production!)
    # - "https://yourdomain.com"
    # - "http://localhost:3000"
  allow_credentials: true      # Allow cookies/auth headers
  allow_methods: ["*"]         # HTTP methods (* = all)
  allow_headers: ["*"]         # HTTP headers (* = all)

# ============================================
# DATABASE CONFIGURATION
# ============================================
# Choose ONE provider: trino, postgresql, mysql, or bigquery
database:
  provider: "postgresql"       # Options: trino, postgresql, mysql, bigquery
  
  # PostgreSQL Configuration
  postgresql:
    host: "localhost"
    port: 5432
    database: "your_database"
    user: "your_user"
    password: "your_password"
    schema: "public"           # Default schema
    pool_size: 5               # Connection pool size
    max_overflow: 10           # Max connections beyond pool_size
    
  # Trino Configuration
  trino:
    host: "trino.company.com"
    port: 443
    user: "your_user"
    password: "your_password"  # Optional, depending on auth
    catalog: "hive"            # Trino catalog
    schema: "default"          # Trino schema
    http_scheme: "https"       # http or https
    verify_ssl: true           # SSL verification
    
  # MySQL Configuration
  mysql:
    host: "localhost"
    port: 3306
    database: "your_database"
    user: "your_user"
    password: "your_password"
    charset: "utf8mb4"
    
  # BigQuery Configuration
  bigquery:
    project_id: "your-gcp-project"
    dataset: "your_dataset"
    credentials_path: "/path/to/service-account.json"
    location: "US"             # Data location (US, EU, etc.)

# ============================================
# LLM PROVIDER CONFIGURATION
# ============================================
# Choose ONE provider: anthropic, gemini, or vertex
llm:
  provider: "anthropic"        # Options: anthropic, gemini, vertex
  
  # Anthropic Claude Configuration
  anthropic:
    api_key: "sk-ant-xxxxx"
    model: "claude-sonnet-4-20250514"  # Recommended model
    max_tokens: 4096
    temperature: 0.0           # 0.0 = deterministic, 1.0 = creative
    timeout: 60                # Request timeout in seconds
    
  # Google Gemini Configuration
  gemini:
    api_key: "your-gemini-api-key"
    model: "gemini-2.0-flash-exp"
    max_tokens: 4096
    temperature: 0.0
    
  # Google Vertex AI Configuration
  vertex:
    project_id: "your-gcp-project"
    location: "us-central1"
    model: "gemini-2.0-flash-exp"
    credentials_path: "/path/to/service-account.json"
    max_tokens: 4096
    temperature: 0.0

# ============================================
# SCHEDULED QUERIES & EMAIL REPORTS
# ============================================
scheduler:
  enabled: false               # Enable scheduled queries feature
  timezone: "UTC"              # Default timezone for schedules
  max_concurrent_jobs: 10      # Max jobs running simultaneously
  job_timeout_seconds: 300     # Timeout for each job execution
  
  # Execution History
  keep_history_days: 30        # How long to keep execution logs
  max_executions_per_schedule: 100  # Max execution logs per schedule

# Email Configuration
email:
  enabled: false               # Set to true to enable email reports
  provider: "smtp"             # Currently only SMTP supported
  
  smtp:
    host: "smtp.gmail.com"
    port: 587
    user: "your-email@gmail.com"     # SMTP username
    password: "app-password"         # SMTP password (use app password for Gmail)
    from_email: "noreply@company.com"  # From email address
    from_name: "SQLatte Reports"     # From name
    use_tls: true                    # Use TLS encryption
    
  # Email Templates
  templates:
    success_subject: "✅ {{schedule_name}} - {{date}}"
    failure_subject: "❌ Failed: {{schedule_name}}"
    
  # Rate Limiting
  max_emails_per_day: 1000     # Max emails per day
  max_recipients_per_email: 10  # Max recipients per email

# Export Configuration
export:
  formats:
    - csv
    - excel
    - html
    
  max_rows: 1000               # Max rows to export (prevent huge files)
  max_file_size_mb: 25         # Max attachment size
  
  # File naming template
  filename_template: "{{schedule_name}}_{{date}}_{{time}}.{{format}}"

# ============================================
# INSIGHTS ENGINE
# ============================================
insights:
  enabled: true                # Enable AI-powered insights
  mode: "hybrid"               # Options: llm_only, statistical_only, hybrid
  max_insights: 3              # Maximum insights per query
  include_statistical: true    # Include statistical analysis

# ============================================
# ANALYTICS & STORAGE
# ============================================
# Optional: Store query history, analytics, and configurations
analytics:
  enabled: false               # Enable analytics tracking
  storage: "memory"            # Options: memory, postgresql
  
  # PostgreSQL storage (if storage=postgresql)
  postgresql:
    host: "localhost"
    port: 5432
    database: "sqlatte_analytics"
    user: "sqlatte"
    password: "analytics_password"
    
  # Retention settings
  query_history_days: 90       # Keep query history for 90 days
  execution_logs_days: 30      # Keep execution logs for 30 days

# ============================================
# PLUGIN SYSTEM
# ============================================
plugins:
  # Authentication Plugin (Multi-tenant mode)
  auth:
    enabled: false             # Enable multi-tenant authentication
    session_ttl_minutes: 480   # Session timeout (8 hours)
    max_workers: 40            # Thread pool size for concurrent users
    
    # Server-side database configuration (for auth plugin)
    db_provider: "trino"       # Database type
    db_host: "trino.company.com"
    db_port: 443
    
    # Catalog/Schema Restrictions
    # If empty lists, all catalogs/schemas are allowed
    # If filled, only these will be available in login form
    allowed_catalogs:
      - name: "hive"
        allowed_schemas: ["edr", "hive"]
      - name: "network"
        allowed_schemas: [ "cloudflare" ]
      - name: "mongo_stores_db"
        allowed_schemas: [ "mongo" ]
      
    # Database Type Restrictions
    allowed_db_types:
      - "trino"


# ============================================
# PROMPTS CONFIGURATION
# ============================================
# These prompts control how SQLatte interacts with your data
# You can customize them via Admin Panel or directly here
prompts:
  # Intent Detection: Determines if question needs SQL or is chat
  intent_detection: |
    Analyze this user question and determine if it requires a SQL query or is general conversation.
    
    Available Tables Schema:
    {schema_info}
    
    User Question: {question}
    
    Rules:
    1. If question asks about data, analytics, or queries related to the available tables → intent: "sql"
    2. If question is greeting, general chat, help, or unrelated to tables → intent: "chat"
    3. If tables are not selected but question seems like SQL query → intent: "chat" (explain they need to select tables)
    
    Respond in this exact format:
    INTENT: sql or chat
    CONFIDENCE: 0.0 to 1.0
    REASONING: brief explanation

  # Barista Personality: Chat response personality
  barista_personality: |
    You are SQLatte ☕ - a friendly AI assistant that helps users query their databases with natural language.
    
    Your personality:
    - Helpful and friendly, like a barista serving the perfect drink
    - Knowledgeable about SQL and databases
    - Can have casual conversations too
    - Use coffee/brewing metaphors occasionally when appropriate
    
    When users ask general questions (not about data):
    - Respond naturally and helpfully
    - If they seem lost, guide them on how to use SQLatte
    - Be concise but friendly

  # SQL Generation: Natural language to SQL translation
  sql_generation: |
    You are a SQL expert. Generate a SQL query based on the user's question.
    
    Table Schema(s):
    {schema_info}
    
    User Question: {question}
    
    Rules:
    1. Generate ONLY valid SQL syntax for the target database
    2. If multiple tables are provided, use appropriate JOINs
    3. Infer JOIN conditions from table relationships (common column names)
    4. Use table aliases for readability (e.g., orders o, customers c)
    5. Include LIMIT clause for safety (default 100 rows)
    6. For aggregations, use GROUP BY appropriately
    7. Use explicit JOIN syntax (INNER JOIN, LEFT JOIN, etc.)
    
    ⚡ PERFORMANCE OPTIMIZATION (CRITICAL):
    8. **PARTITION COLUMN**: If schema contains a 'dt' column (VARCHAR format YYYYMMDD, e.g., '20251218'), this is a PARTITION KEY
       - ALWAYS add WHERE clause with 'dt' filter when possible
       - Date filters MUST use dt column in format: dt = '20251218' or dt BETWEEN '20251201' AND '20251218'
       - For "recent", "latest", "today", "last" queries → use last 2 days
       - For "yesterday" → use appropriate date
       - For specific date range → convert to dt format
       - ⚠️ NEVER query without dt filter unless explicitly asked for "all time" data
    9. If 'datetime' column exists alongside 'dt', use 'dt' for filtering (faster) and 'datetime' for display
    10. Example optimized query: 
        SELECT * FROM orders WHERE dt >= '20251201' AND status = 'completed' LIMIT 100
                          
    Format your response as:
    SQL: [your SQL query here]
    EXPLANATION: [brief explanation including JOIN strategy if applicable]

  # Insights Generation: Analyze results and provide insights
  insights_generation: |
    Analyze query results and generate actionable insights.
    
    Available Information:
    - User Question: {user_question}
    - SQL Query: {sql_query}
    - Data Sample: {data_preview}
    - Schema: {schema_info}
    
    Generate 1-3 key insights that are:
    1. Actionable and specific
    2. Based on actual data patterns
    3. Relevant to the user's question
    4. Easy to understand
    
    Identify patterns such as:
    - Trends (increasing/decreasing over time)
    - Anomalies (unexpected values, outliers)
    - Recommendations (what action to take)
    - Summaries (key takeaways)
    
    Format each insight as:
    TYPE: trend/anomaly/recommendation/summary
    SEVERITY: low/medium/high
    MESSAGE: <concise, actionable insight>
```

---

## Minimal Configuration

The absolute minimum needed to run SQLatte:

```yaml
database:
  provider: postgresql
  postgresql:
    host: localhost
    port: 5432
    database: mydb
    user: dbuser
    password: dbpass

llm:
  provider: anthropic
  anthropic:
    api_key: sk-ant-xxxxx
    model: claude-sonnet-4-20250514
```

This gives you basic natural language to SQL conversion without optional features.

---

## Configuration by Section

### App Settings

Controls the FastAPI application server.

```yaml
app:
  name: "SQLatte"              # Application name (appears in UI)
  version: "1.0.0"             # Version number
  host: "0.0.0.0"              # Bind address
  port: 8000                   # Server port
  debug: false                 # Enable debug mode (dev only!)
  log_level: "INFO"            # Logging verbosity
```

**Host Options:**
- `0.0.0.0` - Listen on all network interfaces (production)
- `127.0.0.1` - Listen only on localhost (development)

**Log Levels:**
- `DEBUG` - Verbose logging, includes SQL queries
- `INFO` - Standard logging
- `WARNING` - Warnings only
- `ERROR` - Errors only
- `CRITICAL` - Critical errors only

### CORS Settings

Cross-Origin Resource Sharing configuration for widget embedding.

```yaml
cors:
  allow_origins:
    - "*"                      # Allow all origins (dev only!)
    - "https://myapp.com"      # Specific domain
  allow_credentials: true
  allow_methods: ["*"]         # or ["GET", "POST"]
  allow_headers: ["*"]
```

!!! danger "Production CORS"
    Never use `"*"` for `allow_origins` in production! Specify exact domains.

### Database Providers

SQLatte supports four database providers. Choose one by setting `database.provider`.

#### PostgreSQL

```yaml
database:
  provider: "postgresql"
  postgresql:
    host: "localhost"
    port: 5432
    database: "your_db"
    user: "your_user"
    password: "your_password"
    schema: "public"           # Default schema
    pool_size: 5               # Connection pool
    max_overflow: 10           # Additional connections
```

**Connection String Alternative:**
```yaml
database:
  provider: "postgresql"
  postgresql:
    connection_string: "postgresql://user:password@host:5432/database"
```

#### Trino

```yaml
database:
  provider: "trino"
  trino:
    host: "trino.company.com"
    port: 443
    user: "your_user"
    password: "optional_password"
    catalog: "hive"            # Hive, Iceberg, Delta Lake, etc.
    schema: "default"
    http_scheme: "https"
    verify_ssl: true
```

**Trino-Specific Notes:**
- Catalogs represent different data sources (Hive, Iceberg, PostgreSQL connector, etc.)
- Schema is the namespace within a catalog
- HTTP scheme usually `https` in production

#### MySQL

```yaml
database:
  provider: "mysql"
  mysql:
    host: "localhost"
    port: 3306
    database: "your_db"
    user: "your_user"
    password: "your_password"
    charset: "utf8mb4"         # UTF-8 support
```

#### Google BigQuery

```yaml
database:
  provider: "bigquery"
  bigquery:
    project_id: "my-gcp-project"
    dataset: "my_dataset"
    credentials_path: "/path/to/service-account.json"
    location: "US"             # or "EU", "asia-northeast1", etc.
```

**Service Account Setup:**
1. Create service account in GCP Console
2. Grant `BigQuery Data Viewer` and `BigQuery Job User` roles
3. Download JSON key file
4. Set `credentials_path` to JSON file location

### LLM Providers

SQLatte supports three LLM providers. Choose one by setting `llm.provider`.

#### Anthropic Claude (Recommended)

```yaml
llm:
  provider: "anthropic"
  anthropic:
    api_key: "sk-ant-xxxxx"
    model: "claude-sonnet-4-20250514"  # Most balanced
    # model: "claude-opus-4-20250514"  # Most capable
    # model: "claude-haiku-4-20250316" # Fastest/cheapest
    max_tokens: 4096
    temperature: 0.0           # Deterministic (recommended for SQL)
    timeout: 60
```

**Model Comparison:**
- **Sonnet 4** - Best balance of speed/quality/cost (recommended)
- **Opus 4** - Highest quality, slower, more expensive
- **Haiku 4** - Fastest, cheapest, good for simple queries

#### Google Gemini

```yaml
llm:
  provider: "gemini"
  gemini:
    api_key: "your-gemini-api-key"
    model: "gemini-2.0-flash-exp"
    max_tokens: 4096
    temperature: 0.0
```

**Getting API Key:**
1. Visit https://makersuite.google.com/app/apikey
2. Create new API key
3. Free tier available with rate limits

#### Google Vertex AI

```yaml
llm:
  provider: "vertex"
  vertex:
    project_id: "your-gcp-project"
    location: "us-central1"
    model: "gemini-2.0-flash-exp"
    credentials_path: "/path/to/service-account.json"
    max_tokens: 4096
    temperature: 0.0
```

### Scheduler & Email

Enable automated query execution and email delivery.

```yaml
scheduler:
  enabled: true
  timezone: "America/New_York"  # Use IANA timezone names
  max_concurrent_jobs: 10
  job_timeout_seconds: 300
  keep_history_days: 30

email:
  enabled: true
  smtp:
    host: "smtp.gmail.com"
    port: 587                   # 587 for TLS, 465 for SSL
    user: "your-email@gmail.com"
    password: "app-password"    # Gmail: use App Password, not account password
    from_email: "noreply@company.com"
    from_name: "SQLatte Reports"
    use_tls: true

export:
  formats: [csv, excel, html]
  max_rows: 1000
  max_file_size_mb: 25
  filename_template: "{{schedule_name}}_{{date}}_{{time}}.{{format}}"
```

**Gmail App Password:**
1. Enable 2FA on Gmail account
2. Go to https://myaccount.google.com/apppasswords
3. Generate app password
4. Use generated password in config

**Template Variables:**
- `{{schedule_name}}` - Name of the schedule
- `{{date}}` - Date in YYYY-MM-DD format
- `{{time}}` - Time in HH-MM-SS format
- `{{format}}` - File format (csv, excel, html)

### Insights Engine

AI-powered data analysis and pattern detection.

```yaml
insights:
  enabled: true
  mode: "hybrid"               # llm_only, statistical_only, or hybrid
  max_insights: 3
  include_statistical: true
```

**Modes:**
- `llm_only` - Pure AI-generated insights
- `statistical_only` - Rule-based statistical analysis
- `hybrid` - Best of both (recommended)

### Analytics Storage

Track query history and performance metrics.

```yaml
analytics:
  enabled: true
  storage: "postgresql"        # memory or postgresql
  
  postgresql:
    host: "localhost"
    port: 5432
    database: "sqlatte_analytics"
    user: "sqlatte"
    password: "analytics_pass"
  
  query_history_days: 90
  execution_logs_days: 30
```

**Storage Modes:**
- `memory` - In-memory storage, fast but not persistent
- `postgresql` - Persistent storage in PostgreSQL database

### Plugin Configuration

#### Auth Plugin (Multi-Tenant)

Enable per-user database authentication.

```yaml
plugins:
  auth:
    enabled: true
    session_ttl_minutes: 480    # 8 hours
    max_workers: 40
    
    # Server-side database info (what DB type to connect to)
    db_provider: "trino"
    db_host: "trino.company.com"
    db_port: 443
    
    # Catalog/Schema Restrictions
    # If empty lists, all catalogs/schemas are allowed
    # If filled, only these will be available in login form
    allowed_catalogs:
      - name: "hive"
        allowed_schemas: ["edr", "hive"]
      - name: "network"
        allowed_schemas: [ "cloudflare" ]
      - name: "mongo_stores_db"
        allowed_schemas: [ "mongo" ]

    # Database Type Restrictions
    allowed_db_types:
      - "trino"
```

**When to Enable:**
- Multi-tenant SaaS applications
- Per-user database access control
- Customer-facing analytics

**When to Disable:**
- Internal tools with shared database access
- Single-tenant deployments

### Prompts Configuration

Customize AI behavior through prompts. These can also be edited via Admin Panel.

```yaml
prompts:
  intent_detection: |
    [Your custom intent detection prompt]
    
  barista_personality: |
    [Your custom chat personality]
    
  sql_generation: |
    [Your custom SQL generation rules]
    
  insights_generation: |
    [Your custom insights analysis prompt]
```

**Template Variables Available:**
- `{schema_info}` - Database schema information
- `{question}` - User's natural language question
- `{query_result}` - SQL query execution results
- `{user_question}` - Original user question
- `{sql_query}` - Generated SQL query
- `{data_preview}` - Sample of query results

See [Runtime Prompts](../features/runtime-prompts.md) for detailed customization guide.

---

## Environment Variable Overrides

Any configuration value can be overridden with environment variables using this pattern:

```bash
# Database
export SQLATTE_DB_HOST=production-db.company.com
export SQLATTE_DB_PASSWORD=secure_password

# LLM
export SQLATTE_LLM_PROVIDER=anthropic
export SQLATTE_LLM_API_KEY=sk-ant-xxxxx

# Email
export SQLATTE_SMTP_HOST=smtp.gmail.com
export SQLATTE_SMTP_PASSWORD=app_password

# App
export SQLATTE_PORT=8000
export SQLATTE_DEBUG=false
```

**Naming Convention:**
`SQLATTE_<SECTION>_<KEY>=<value>`

Environment variables take precedence over `config.yaml` values.

---

## Configuration Validation

SQLatte validates configuration on startup and provides helpful error messages.

### Common Validation Errors

**Database Connection Failed:**
```
ERROR: Could not connect to PostgreSQL at localhost:5432
- Check host and port are correct
- Verify credentials
- Ensure database exists
- Check firewall rules
```

**Invalid LLM API Key:**
```
ERROR: Anthropic API authentication failed
- Verify API key starts with 'sk-ant-'
- Check key has not expired
- Ensure sufficient API credits
```

**Missing Required Fields:**
```
ERROR: Missing required field: database.postgresql.database
- Add 'database' field under postgresql section
```

### Manual Validation

Run validation without starting server:

```bash
python -m src.core.config_validator --config config/config.yaml
```

---

## Security Best Practices

!!! danger "Never Commit Secrets"
    Do not commit `config.yaml` with real credentials to version control!

### Recommended Approach

**1. Template Configuration**

Create `config/config.template.yaml`:
```yaml
database:
  provider: postgresql
  postgresql:
    host: ${DB_HOST}
    password: ${DB_PASSWORD}

llm:
  provider: anthropic
  anthropic:
    api_key: ${ANTHROPIC_API_KEY}
```

**2. .gitignore**

```
config/config.yaml
*.env
.env.local
```

**3. Environment Variables**

Use `.env` file (not committed):
```bash
DB_HOST=localhost
DB_PASSWORD=secret
ANTHROPIC_API_KEY=sk-ant-xxxxx
```

**4. Production Secrets Manager**

For production, use:
- AWS Secrets Manager
- Google Cloud Secret Manager
- HashiCorp Vault
- Azure Key Vault

---

## Configuration Examples

### E-Commerce Analytics

```yaml
database:
  provider: postgresql
  postgresql:
    host: analytics-db.company.com
    database: ecommerce_prod
    
analytics:
  enabled: true
  storage: postgresql
  query_history_days: 180
  
scheduler:
  enabled: true
  timezone: "America/Los_Angeles"
  
email:
  enabled: true
  smtp:
    host: smtp.sendgrid.net
    from_email: analytics@company.com
    
insights:
  enabled: true
  mode: hybrid
```

### Multi-Tenant SaaS

```yaml
plugins:
  auth:
    enabled: true
    session_ttl_minutes: 240
    max_workers: 100
    db_provider: trino
    allowed_catalogs:
      - customer_data
      
analytics:
  enabled: true
  storage: postgresql
  
cors:
  allow_origins:
    - "https://app.company.com"
  allow_credentials: true
```

### Data Warehouse Querying

```yaml
database:
  provider: bigquery
  bigquery:
    project_id: company-analytics
    dataset: warehouse
    
llm:
  provider: vertex
  vertex:
    project_id: company-analytics
    location: us-central1
    
analytics:
  enabled: false  # Keep it lightweight
  storage: memory
```

---

## Configuration Tips

### Performance Optimization

```yaml
database:
  postgresql:
    pool_size: 10              # Increase for high concurrency
    max_overflow: 20
    
scheduler:
  max_concurrent_jobs: 20      # Limit parallel scheduled queries
  job_timeout_seconds: 600     # Prevent long-running queries
```

### Development vs Production

=== "Development"
    ```yaml
    app:
      debug: true
      log_level: "DEBUG"
      
    cors:
      allow_origins: ["*"]
      
    analytics:
      enabled: false
      storage: memory
    ```

=== "Production"
    ```yaml
    app:
      debug: false
      log_level: "INFO"
      
    cors:
      allow_origins:
        - "https://analytics.company.com"
      
    analytics:
      enabled: true
      storage: postgresql
    ```

### Resource Limits

```yaml
export:
  max_rows: 10000              # Limit CSV export size
  max_file_size_mb: 50         # Limit email attachments
  
email:
  max_emails_per_day: 5000     # Rate limiting
  max_recipients_per_email: 20
```

---

## Troubleshooting

### Config Not Loading

```bash
# Check file exists
ls -l config/config.yaml

# Check YAML syntax
python -c "import yaml; yaml.safe_load(open('config/config.yaml'))"

# Check permissions
chmod 600 config/config.yaml
```

### Changes Not Taking Effect

Some changes require restart:

```bash
# Stop SQLatte
pkill -f "python run.py"

# Start again
python run.py
```

For hot-reloadable configs (prompts, LLM parameters), use admin panel or reload endpoint:

```bash
curl -X POST http://localhost:8000/admin/reload
```

---

## Next Steps

- **Database Setup**: [Configure your database provider →](database-providers.md)
- **LLM Setup**: [Configure your LLM provider →](llm-providers.md)
- **Runtime Prompts**: [Customize AI behavior →](../features/runtime-prompts.md)
- **Scheduling**: [Set up automated reports →](../features/schedules.md)

---

**Need help?** Check [Configuration Overview](overview.md) or create an issue on [GitHub](https://github.com/osmanuygar/sqlatte/issues).
