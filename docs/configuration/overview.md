# Configuration Overview

SQLatte uses a YAML-based configuration system that follows the "optional complexity" philosophy - start with minimal settings and add features as needed.

## Configuration File

SQLatte loads configuration from `config.yaml` in the root directory. You can also specify a custom path:

```bash
python app.py --config /path/to/custom-config.yaml
```

Or use environment variable:
```bash
export SQLATTE_CONFIG=/path/to/custom-config.yaml
```

## Minimal Configuration

The absolute minimum to get started:

```yaml
database:
  provider: postgresql
  host: localhost
  database: mydb
  user: dbuser
  password: dbpass

llm:
  provider: anthropic
  api_key: sk-ant-xxxxx
```

This gives you basic natural language querying without any optional features.

## Full Configuration Structure

Here's the complete configuration structure:

```yaml
app:
  host: 0.0.0.0
  port: 5001
  debug: false
  cors:
    enabled: true
    origins: ["*"]

database:
  provider: postgresql  # postgresql, mysql, trino, bigquery
  host: localhost
  port: 5432
  database: mydb
  user: dbuser
  password: dbpass
  # Provider-specific options...

llm:
  provider: anthropic  # anthropic, gemini, vertex
  api_key: your-key
  model: claude-sonnet-4-5-20250929
  temperature: 0.0
  max_tokens: 4096

analytics:
  enabled: true
  provider: postgresql
  host: localhost
  port: 5432
  database: sqlatte_analytics
  user: analytics_user
  password: analytics_pass

session:
  enabled: true
  secret_key: your-secret-key
  max_age: 86400

scheduling:
  enabled: true
  email:
    smtp_host: smtp.gmail.com
    smtp_port: 587
    smtp_user: your-email@gmail.com
    smtp_password: your-app-password
    from_address: sqlatte@yourcompany.com

prompts:
  system_prompt: |
    You are a helpful data analyst assistant...
  # Runtime prompts loaded from database when enabled
```

## Configuration Sections

### App Settings

Controls the Flask application:

```yaml
app:
  host: 0.0.0.0          # Listen address
  port: 5001             # Port number
  debug: false           # Debug mode (dev only!)
  cors:
    enabled: true        # Enable CORS
    origins: ["*"]       # Allowed origins
```

[Learn more →](config-yaml.md#app-settings)

### Database Provider

Your data source connection:

```yaml
database:
  provider: postgresql
  # Connection details...
```

Supports: PostgreSQL, MySQL, Trino, BigQuery

[Learn more →](database-providers.md)

### LLM Provider

AI provider for natural language processing:

```yaml
llm:
  provider: anthropic
  api_key: sk-ant-xxxxx
  model: claude-sonnet-4-5-20250929
```

Supports: Anthropic Claude, Google Gemini, Vertex AI

[Learn more →](llm-providers.md)

### Analytics (Optional)

Track queries, insights, and usage:

```yaml
analytics:
  enabled: true
  provider: postgresql
  # Database connection...
```

[Learn more →](analytics.md)

### Session Management (Optional)

Enable user sessions for auth widget:

```yaml
session:
  enabled: true
  secret_key: your-secret-key
  max_age: 86400  # 24 hours
```

Required for: Auth widget, scheduled queries per user

### Scheduling (Optional)

Enable automated query execution:

```yaml
scheduling:
  enabled: true
  email:
    smtp_host: smtp.gmail.com
    smtp_port: 587
    # Email credentials...
```

[Learn more →](../features/schedules.md)

### Runtime Prompts (Optional)

Customize AI behavior through admin panel:

```yaml
prompts:
  system_prompt: |
    Default system prompt...
```

When analytics is enabled, prompts can be managed via admin UI.

[Learn more →](../features/runtime-prompts.md)

## Configuration Patterns

### Development vs Production

=== "Development"

    ```yaml
    app:
      host: 127.0.0.1
      port: 5001
      debug: true
      cors:
        enabled: true
        origins: ["http://localhost:3000"]
    
    database:
      provider: postgresql
      host: localhost
      # Local dev database
    
    analytics:
      enabled: false  # Keep it simple
    
    session:
      enabled: false  # Config-based widget only
    ```

=== "Production"

    ```yaml
    app:
      host: 0.0.0.0
      port: 5001
      debug: false
      cors:
        enabled: true
        origins: ["https://yourapp.com"]
    
    database:
      provider: postgresql
      host: prod-db.company.com
      # Production database
    
    analytics:
      enabled: true
      provider: postgresql
      # Separate analytics database
    
    session:
      enabled: true
      secret_key: ${SESSION_SECRET}  # From env var
    
    scheduling:
      enabled: true
      # Email configuration
    ```

### Feature Progression

Start simple, add features incrementally:

**Phase 1: Basic Querying**
```yaml
database: { ... }
llm: { ... }
```

**Phase 2: Add Analytics**
```yaml
database: { ... }
llm: { ... }
analytics:
  enabled: true
  # ...
```

**Phase 3: Add Sessions**
```yaml
database: { ... }
llm: { ... }
analytics: { ... }
session:
  enabled: true
  # ...
```

**Phase 4: Add Scheduling**
```yaml
database: { ... }
llm: { ... }
analytics: { ... }
session: { ... }
scheduling:
  enabled: true
  # ...
```

## Environment Variable Overrides

Any configuration value can be overridden with environment variables:

```bash
# Database
export SQLATTE_DB_HOST=prod-db.company.com
export SQLATTE_DB_PASSWORD=secure-password

# LLM
export SQLATTE_LLM_API_KEY=sk-ant-xxxxx

# Session
export SQLATTE_SESSION_SECRET=random-secret-key

# Email
export SQLATTE_SMTP_PASSWORD=app-password
```

Environment variables take precedence over `config.yaml`.

## Configuration Validation

SQLatte validates configuration on startup:

```python
# Run validation manually
python -m sqlatte.config validate
```

Common validation errors:

- Missing required fields
- Invalid provider names
- Unreachable database
- Invalid API keys
- Port already in use

## Security Best Practices

!!! danger "Credentials in Config"
    Never commit `config.yaml` with real credentials to version control!

**Recommended approach:**

1. Create `config.template.yaml` with placeholder values
2. Add `config.yaml` to `.gitignore`
3. Use environment variables for secrets in production
4. Store secrets in a secrets manager (AWS Secrets, Vault, etc.)

**Example `.gitignore`:**
```
config.yaml
*.env
.env.local
```

**Example `config.template.yaml`:**
```yaml
database:
  provider: postgresql
  host: ${DB_HOST}
  password: ${DB_PASSWORD}  # Set via environment variable

llm:
  api_key: ${LLM_API_KEY}  # Set via environment variable
```

## Hot Reload

Some configuration changes don't require restart:

**Requires Restart:**
- Database provider change
- Port change
- Analytics enable/disable
- Session enable/disable

**Hot Reloadable (with reload endpoint):**
- Runtime prompts (via admin UI)
- LLM temperature/parameters
- CORS settings

Use the reload endpoint:
```bash
curl -X POST http://localhost:5001/api/admin/reload-config
```

## Configuration Examples

### E-commerce Setup

```yaml
database:
  provider: postgresql
  host: prod-db.company.com
  database: ecommerce_prod
  
analytics:
  enabled: true
  # Separate analytics DB
  
scheduling:
  enabled: true
  # Morning reports to team
```

### Multi-tenant SaaS

```yaml
session:
  enabled: true
  # User isolation required
  
analytics:
  enabled: true
  # Track per-tenant usage
  
database:
  # Shared database with tenant_id
```

### Data Warehouse Queries

```yaml
database:
  provider: bigquery
  project_id: my-project
  
llm:
  provider: vertex
  # Same GCP project
  
analytics:
  enabled: false
  # Keep it lightweight
```

## Next Steps

- **Database Setup**: [Configure your database provider →](database-providers.md)
- **LLM Setup**: [Configure your LLM provider →](llm-providers.md)
- **Full Reference**: [Complete config.yaml reference →](config-yaml.md)
- **Advanced Features**: [Enable analytics, scheduling, etc. →](analytics.md)

## Troubleshooting

### Configuration not loading

Check file location and permissions:
```bash
ls -l config.yaml
```

### Environment variables not working

Verify variable names match pattern:
```bash
env | grep SQLATTE
```

### Validation errors

Run explicit validation:
```bash
python -m sqlatte.config validate --verbose
```

### Changes not taking effect

Some changes require restart:
```bash
# Stop and restart
pkill -f "python app.py"
python app.py
```
