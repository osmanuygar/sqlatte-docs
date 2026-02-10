# Installation

This guide will help you install and set up SQLatte on your system.

## Prerequisites

- Python 3.8 or higher
- Access to a database (Trino, PostgreSQL, MySQL, or BigQuery)
- API key for an LLM provider (Anthropic, Gemini, or Vertex AI)

## Installation Methods

=== "Standard Installation"

    ### Clone the Repository

    ```bash
    git clone https://github.com/osmanuygar/sqlatte.git
    cd sqlatte
    ```

    ### Install Dependencies

    ```bash
    pip install -r requirements.txt
    ```

    ### Create Configuration File

    Create a `config.yaml` file in the config/ directory:

    ```yaml
    app:
      host: 0.0.0.0
      port: 8000
      debug: false
    
    database:
      provider: postgresql
      host: localhost
      port: 5432
      database: mydb
      user: dbuser
      password: dbpassword
    
    llm:
      provider: anthropic
      api_key: your-api-key-here
      model: claude-sonnet-4-5-20250929
    ```

    ### Run SQLatte

    ```bash
    python app.py
    ```

    SQLatte will start at `http://localhost:8000`

=== "Docker"

    ### Using Docker Compose

    Create a `docker-compose.yml`:

    ```yaml
    version: '3.8'
    
    services:
      sqlatte:
        image: sqlatte:latest
        ports:
          - "5001:5001"
        volumes:
          - ./config.yaml:/app/config/config.yaml
        environment:
          - SQLATTE_CONFIG=/app/config/config.yaml
    ```

    Build and run:

    ```bash
    docker-compose up -d
    ```

    See [Docker Deployment](docker.md) for more details.

=== "Development"

    ### Development Setup

    ```bash
    # Clone repository
    git clone https://github.com/osmanuygar/sqlatte.git
    cd sqlatte
    
    # Create virtual environment
    python -m venv venv
    source venv/bin/activate  # Windows: venv\Scripts\activate
    
    # Install dependencies with dev tools
    pip install -r requirements.txt
    pip install -r requirements-dev.txt  # if available
    
    # Run in debug mode
    python app.py --debug
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
    - Review [Security Best Practices](../architecture/security.md)

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
