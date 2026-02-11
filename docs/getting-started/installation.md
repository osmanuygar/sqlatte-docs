# Installation Guide

This comprehensive guide will walk you through installing SQLatte on your system, from basic setup to production deployment.

## Overview

SQLatte can be installed in multiple ways depending on your use case:

- **Quick Start** (5 minutes) - Docker Compose for immediate testing
- **Standard Installation** (10 minutes) - Python installation for development
- **Production Deployment** (30 minutes) - Docker with persistent storage and monitoring
- **Development Setup** (15 minutes) - Full development environment

Choose the method that best fits your needs.

## Prerequisites

### Required

=== "All Installations"
    - **Python 3.8+** (3.10+ recommended for best performance)
    - **Database Access**: One of the following
        - Trino (distributed SQL)
        - PostgreSQL 12+
        - MySQL 8.0+
        - Google BigQuery (with service account)
    - **LLM API Key**: One of the following
        - Anthropic Claude API key (recommended)
        - Google Gemini API key (free tier available)
        - Google Cloud Vertex AI access

=== "Production"
    - **Operating System**: Linux (Ubuntu 20.04+ recommended) or macOS
    - **Docker** (optional but recommended): Docker 20.10+ and Docker Compose 1.29+
    - **Reverse Proxy**: Nginx or Caddy (for HTTPS)
    - **Monitoring**: Prometheus + Grafana (optional)
    - **Email Server**: SMTP access (for scheduled reports)

=== "Development"
    - **Git** for version control
    - **Code Editor**: VS Code, PyCharm, or similar
    - **Virtual Environment**: `venv` or `conda`
    - **Database Client**: pgAdmin, DBeaver, or similar (for testing)

### Optional (for Advanced Features)

- **PostgreSQL 12+** for persistent analytics storage (otherwise uses in-memory)
- **Redis** (for production caching and queue management)
- **SMTP Server** access (Gmail, SendGrid, etc.) for email reports
- **Google Cloud Account** (for BigQuery or Vertex AI)

## Installation Methods

### Method 1: Quick Start with Docker Compose (Recommended for Testing) 

**Time**: ~5 minutes | **Difficulty**: Easy | **Best for**: Testing, demos, quick evaluation

This is the fastest way to get SQLatte running with all features enabled.

#### Step 1: Prerequisites

```bash
# Verify Docker and Docker Compose are installed
docker --version          # Should be 20.10+
docker-compose --version  # Should be 1.29+
```

#### Step 2: Download SQLatte

```bash
# Clone the repository
git clone https://github.com/osmanuygar/sqlatte.git
cd sqlatte
```

#### Step 3: Configure Environment

Create a `.env` file (or edit `config/config.yaml`):

```bash
# Quick config with PostgreSQL example
cat > config/config.yaml << 'EOF'
app:
  name: "SQLatte"
  host: "0.0.0.0"
  port: 8000
  debug: false

database:
  provider: "postgresql"
  postgresql:
    host: "localhost"
    port: 5432
    database: "your_database"
    user: "your_user"
    password: "your_password"
    schema: "public"

llm:
  provider: "anthropic"
  anthropic:
    api_key: "your-anthropic-api-key"
    model: "claude-sonnet-4-20250514"

cors:
  allow_origins:
    - "*"  # For testing only! Restrict in production
  allow_credentials: true
  allow_methods: ["*"]
  allow_headers: ["*"]

# Optional: Enable features
scheduler:
  enabled: false

email:
  enabled: false

insights:
  enabled: true
  mode: "hybrid"

plugins:
  auth:
    enabled: false
EOF
```

#### Step 4: Start SQLatte

```bash
# Using Docker Compose
docker-compose up -d

# Check logs
docker-compose logs -f sqlatte

# Or without Docker (direct Python)
pip install -r requirements.txt
python run.py
```

#### Step 5: Verify Installation

```bash
# Test API health
curl http://localhost:8000/health

# Expected response:
# {
#   "status": "healthy",
#   "version": "1.0.0",
#   "database": "connected",
#   "llm": "available"
# }
```

#### Step 6: Access SQLatte

Open your browser:

- **Main Dashboard**: http://localhost:8000
- **Admin Panel**: http://localhost:8000/admin
- **Demo Page**: http://localhost:8000/demo
- **Analytics**: http://localhost:8000/analytics

✅ **You're done!** SQLatte is now running and ready to use.

---

### Method 2: Standard Python Installation (For Development)

**Time**: ~10 minutes | **Difficulty**: Moderate | **Best for**: Development, customization

This method gives you more control and is ideal for development or when Docker isn't available.

#### Step 1: System Preparation

```bash
# Update system packages (Ubuntu/Debian)
sudo apt update && sudo apt upgrade -y

# Install Python 3.10+ (if not already installed)
sudo apt install python3.10 python3.10-venv python3-pip -y

# Verify Python version
python3 --version  # Should be 3.8 or higher
```

#### Step 2: Clone Repository

```bash
git clone https://github.com/osmanuygar/sqlatte.git
cd sqlatte
```

#### Step 3: Create Virtual Environment

```bash
# Create virtual environment
python3 -m venv venv

# Activate it
source venv/bin/activate  # Linux/macOS
# OR
venv\Scripts\activate     # Windows
```

#### Step 4: Install Dependencies

```bash
# Upgrade pip
pip install --upgrade pip

# Install SQLatte dependencies
pip install -r requirements.txt

# Verify installations
pip list | grep -E "anthropic|fastapi|trino|psycopg2"
```

#### Step 3: Configure SQLatte

```bash
# Create config directory if it doesn't exist
mkdir -p config

# Copy example config
cp config/config.example.yaml config/config.yaml

# Edit configuration
nano config/config.yaml  # or use your preferred editor
```

**Minimal config.yaml**:

```yaml
app:
  name: "SQLatte"
  host: "0.0.0.0"
  port: 8000
  debug: true  # Enable for development

database:
  provider: "postgresql"  # or "trino", "mysql", "bigquery"
  postgresql:
    host: "localhost"
    port: 5432
    database: "mydb"
    user: "postgres"
    password: "your_password"
    schema: "public"

llm:
  provider: "anthropic"
  anthropic:
    api_key: "sk-ant-xxxxx"  # Your API key
    model: "claude-sonnet-4-20250514"
    max_tokens: 4096
    temperature: 0.0

cors:
  allow_origins:
    - "http://localhost:3000"
    - "http://localhost:8000"
  allow_credentials: true
```

**With Prompts Customization (Optional)**:

If you want to customize AI behavior from the start:

```yaml
# ... (database and llm config above) ...

# Customize AI prompts
prompts:
  # Intent Detection
  intent_detection: |
    Analyze this user question and determine if it requires a SQL query.
    
    Available Tables: {schema_info}
    Question: {question}
    
    Rules:
    1. Data/analytics questions → intent: "sql"
    2. Greetings/chat → intent: "chat"
    
    Format: INTENT: sql|chat

  # SQL Generation (with your database-specific optimizations)
  sql_generation: |
    Generate SQL for the question.
    
    Schema: {schema_info}
    Question: {question}
    
    Rules:
    1. Use proper JOINs
    2. Add LIMIT 100 for safety
    3. For Trino: use catalog.schema.table format
    4. If 'dt' partition column exists, ALWAYS filter by it
    
    Format:
    SQL: [query]
    EXPLANATION: [brief explanation]

  # Barista Personality
  barista_personality: |
    You are SQLatte ☕ - a friendly data assistant.
    Be helpful, concise, and use coffee metaphors occasionally.

  # Insights Generation
  insights_generation: |
    Analyze results and provide 1-3 actionable insights.
    
    Focus on: trends, anomalies, recommendations
    Format: TYPE: [type] | SEVERITY: [level] | MESSAGE: [insight]
```

!!! tip "Prompts Can Be Edited Later"
    You can skip prompt customization now and edit them later via the Admin Panel at http://localhost:8000/admin

#### Step 6: Initialize Database (Optional - for Analytics)

If you want persistent analytics:

```bash
# Create PostgreSQL database for SQLatte analytics
psql -U postgres -c "CREATE DATABASE sqlatte_analytics;"

# Update config.yaml
analytics:
  enabled: true
  storage: "postgresql"
  postgresql:
    host: "localhost"
    port: 5432
    database: "sqlatte_analytics"
    user: "postgres"
    password: "your_password"
```

#### Step 7: Run SQLatte

```bash
# Start the server
python run.py

# Or with auto-reload for development
uvicorn src.api.app:app --reload --host 0.0.0.0 --port 8000
```

#### Step 8: Test Installation

```bash
# In another terminal
curl http://localhost:8000/health

# Test database connection
curl http://localhost:8000/api/tables

# Test a simple query
curl -X POST http://localhost:8000/api/query \
  -H "Content-Type: application/json" \
  -d '{"question": "show me all tables"}'
```

---

### Method 3: Production Deployment with Docker 

**Time**: ~30 minutes | **Difficulty**: Advanced | **Best for**: Production environments

This method includes SSL, persistent storage, monitoring, and high availability.

#### Architecture Overview

```
Internet → Nginx (SSL) → SQLatte Container(s)
                              ↓
                         PostgreSQL (Analytics)
                              ↓
                         Your Database (Trino/PG/MySQL)
```

#### Step 1: Server Preparation

```bash
# Install Docker and Docker Compose
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify
docker --version
docker-compose --version
```

#### Step 2: Clone and Configure

```bash
# Clone repository
git clone https://github.com/osmanuygar/sqlatte.git
cd sqlatte

# Create production config
cp config/config.example.yaml config/config.production.yaml
```

#### Step 3: Production config.yaml

```yaml
app:
  name: "SQLatte Production"
  host: "0.0.0.0"
  port: 8000
  debug: false
  log_level: "INFO"

database:
  provider: "trino"  # Your production database
  trino:
    host: "trino.company.com"
    port: 443
    user: "sqlatte_service"
    password: "${TRINO_PASSWORD}"  # From environment
    catalog: "hive"
    schema: "analytics"
    http_scheme: "https"

llm:
  provider: "anthropic"
  anthropic:
    api_key: "${ANTHROPIC_API_KEY}"
    model: "claude-sonnet-4-20250514"
    max_tokens: 4096

# Enable all production features
scheduler:
  enabled: true
  timezone: "UTC"
  max_concurrent_jobs: 20
  job_timeout_seconds: 600

email:
  enabled: true
  smtp:
    host: "smtp.gmail.com"
    port: 587
    user: "${SMTP_USER}"
    password: "${SMTP_PASSWORD}"
    from_email: "sqlatte@company.com"
    from_name: "SQLatte Reports"

analytics:
  enabled: true
  storage: "postgresql"
  postgresql:
    host: "postgres"
    port: 5432
    database: "sqlatte_analytics"
    user: "sqlatte"
    password: "${ANALYTICS_DB_PASSWORD}"

insights:
  enabled: true
  mode: "hybrid"

cors:
  allow_origins:
    - "https://analytics.company.com"
    - "https://dashboard.company.com"
  allow_credentials: true

plugins:
  auth:
    enabled: true  # For multi-tenant deployments
    session_ttl_minutes: 480
    max_workers: 100
```

#### Step 4: Docker Compose for Production

Create `docker-compose.production.yml`:

```yaml
version: '3.8'

services:
  sqlatte:
    build: .
    container_name: sqlatte_prod
    restart: unless-stopped
    ports:
      - "8000:8000"
    volumes:
      - ./config:/app/config
      - ./logs:/app/logs
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - TRINO_PASSWORD=${TRINO_PASSWORD}
      - SMTP_USER=${SMTP_USER}
      - SMTP_PASSWORD=${SMTP_PASSWORD}
      - ANALYTICS_DB_PASSWORD=${ANALYTICS_DB_PASSWORD}
    env_file:
      - .env.production
    depends_on:
      - postgres
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - sqlatte_network

  postgres:
    image: postgres:15
    container_name: sqlatte_postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=sqlatte
      - POSTGRES_PASSWORD=${ANALYTICS_DB_PASSWORD}
      - POSTGRES_DB=sqlatte_analytics
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - sqlatte_network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U sqlatte"]
      interval: 10s
      timeout: 5s
      retries: 5

  nginx:
    image: nginx:alpine
    container_name: sqlatte_nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - sqlatte
    networks:
      - sqlatte_network

volumes:
  postgres_data:

networks:
  sqlatte_network:
    driver: bridge
```

#### Step 5: Nginx Configuration

Create `nginx.conf`:

```nginx
events {
    worker_connections 1024;
}

http {
    upstream sqlatte {
        server sqlatte:8000;
    }

    server {
        listen 80;
        server_name analytics.company.com;
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name analytics.company.com;

        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;

        client_max_body_size 50M;

        location / {
            proxy_pass http://sqlatte;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # WebSocket support
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
    }
}
```

#### Step 6: Environment Variables

Create `.env.production`:

```bash
# LLM API Keys
ANTHROPIC_API_KEY=sk-ant-xxxxx

# Database Credentials
TRINO_PASSWORD=your_trino_password

# Email Configuration
SMTP_USER=your_email@gmail.com
SMTP_PASSWORD=your_app_password

# Analytics Database
ANALYTICS_DB_PASSWORD=strong_postgres_password

# Application
SQLATTE_ENV=production
LOG_LEVEL=INFO
```

#### Step 7: SSL Certificates

```bash
# Create SSL directory
mkdir -p ssl

# Option 1: Let's Encrypt (recommended)
sudo apt install certbot
sudo certbot certonly --standalone -d analytics.company.com
sudo cp /etc/letsencrypt/live/analytics.company.com/fullchain.pem ssl/cert.pem
sudo cp /etc/letsencrypt/live/analytics.company.com/privkey.pem ssl/key.pem

# Option 2: Self-signed (for testing)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout ssl/key.pem -out ssl/cert.pem
```

#### Step 8: Deploy

```bash
# Build and start services
docker-compose -f docker-compose.production.yml up -d

# Check logs
docker-compose -f docker-compose.production.yml logs -f

# Verify health
curl https://analytics.company.com/health
```

#### Step 9: Monitoring (Optional)

Add Prometheus and Grafana:

```yaml
# Add to docker-compose.production.yml
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"
    networks:
      - sqlatte_network

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - sqlatte_network
```

---

### Method 4: Development Setup 

**Time**: ~15 minutes | **Difficulty**: Moderate | **Best for**: Contributing, customization

Full development environment with hot reload, testing, and debugging.

#### Step 1: Fork and Clone

```bash
# Fork on GitHub first, then:
git clone https://github.com/YOUR_USERNAME/sqlatte.git
cd sqlatte

# Add upstream remote
git remote add upstream https://github.com/osmanuygar/sqlatte.git
```

#### Step 2: Development Environment

```bash
# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install development dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt  # Includes pytest, black, flake8

# Install pre-commit hooks
pre-commit install
```

#### Step 3: IDE Setup

For **VS Code**, create `.vscode/settings.json`:

```json
{
  "python.linting.enabled": true,
  "python.linting.flake8Enabled": true,
  "python.formatting.provider": "black",
  "python.testing.pytestEnabled": true,
  "python.testing.unittestEnabled": false,
  "editor.formatOnSave": true
}
```

#### Step 4: Run with Hot Reload

```bash
# Start with auto-reload
uvicorn src.api.app:app --reload --host 0.0.0.0 --port 8000

# Or with debugging
python -m debugpy --listen 5678 run.py
```

#### Step 5: Run Tests

```bash
# Run all tests
pytest

# With coverage
pytest --cov=src --cov-report=html

# Run specific test
pytest tests/test_query_engine.py -v
```

## Verify Installation

### Test the API

```bash
curl http://localhost:8000/health
```

Expected response:
```json
{
  "status": "healthy",
  "database": "connected",
  "llm": "available"
}
```

### Test Widget

Open your browser and navigate to:
```
http://localhost:8000/demo.html
```

You should see the SQLatte widget interface.

## Next Steps

Now that SQLatte is installed:

1. **Configure your database** - See [Database Providers](../configuration/database-providers.md)
2. **Configure your LLM** - See [LLM Providers](../configuration/llm-providers.md)
3. **Try your first query** - See [Quick Start](quickstart.md)
4. **Enable optional features** - See [Configuration Overview](../configuration/overview.md)

## Troubleshooting

### Database Connection Issues

If you see database connection errors:

```bash
# Test database connectivity
python -c "from sqlatte.db import test_connection; test_connection()"
```

Check:
- Database host and port are correct
- User credentials are valid
- Database exists and is accessible
- Firewall rules allow connections

### LLM API Issues

If LLM queries fail:

- Verify API key is correct
- Check API key has sufficient credits/quota
- Ensure model name matches provider's offerings
- Test with a simple query first

### Port Already in Use

If port 5001 is already in use:

```bash
# Change port in config.yaml
app:
  port: 8000

# Or use environment variable
export SQLATTE_PORT=8001
```

## Upgrading

To upgrade to the latest version:

```bash
cd sqlatte
git pull origin main
pip install -r requirements.txt --upgrade
```

Check the [Changelog](https://github.com/osmanuygar/sqlatte/blob/main/CHANGELOG.md) for breaking changes.

## System Requirements

### Minimum Requirements
- **CPU**: 2 cores
- **RAM**: 2 GB
- **Storage**: 500 MB

### Recommended for Production
- **CPU**: 4+ cores
- **RAM**: 4+ GB
- **Storage**: 2+ GB (if using PostgreSQL analytics)
- **Database**: Separate database server
- **Network**: Low latency to database and LLM API

## Security Considerations

!!! warning "Production Deployment"
    Before deploying to production:
    
    - Use HTTPS/TLS for all connections
    - Store credentials in environment variables or secrets manager
    - Enable authentication if exposing publicly
    - Configure CORS appropriately

## Getting Help

If you encounter issues:

1. Check the [Troubleshooting](#troubleshooting) section
2. Search [GitHub Issues](https://github.com/osmanuygar/sqlatte/issues)
3. Create a new issue with:
    - SQLatte version
    - Python version
    - Database provider and version
    - Error messages and logs
    - Steps to reproduce
