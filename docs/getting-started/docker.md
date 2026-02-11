# Docker Deployment

Deploy SQLatte with Docker for production-ready, containerized environments.

---

## Quick Start (5 Minutes)

### 1. Prerequisites

```bash
# Verify Docker is installed
docker --version          # 20.10+
docker-compose --version  # 1.29+
```

### 2. Clone & Configure

```bash
# Clone repository
git clone https://github.com/osmanuygar/sqlatte.git
cd sqlatte

# Create config file
cp config/config.example.yaml config/config.yaml
nano config/config.yaml
```

### 3. Start SQLatte

```bash
# Start with Docker Compose
docker-compose up -d

# Check logs
docker-compose logs -f

# Access SQLatte
http://localhost:8000
```

**Done!** SQLatte is running in Docker.

---

## Docker Compose (Recommended)

### Basic Setup

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  sqlatte:
    build: .
    container_name: sqlatte
    restart: unless-stopped
    ports:
      - "8000:8000"
    volumes:
      - ./config:/app/config
      - ./logs:/app/logs
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - DB_PASSWORD=${DB_PASSWORD}
    env_file:
      - .env
```

**Start:**
```bash
docker-compose up -d
```

---

### With PostgreSQL Analytics

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  sqlatte:
    build: .
    container_name: sqlatte
    restart: unless-stopped
    ports:
      - "8000:8000"
    volumes:
      - ./config:/app/config
      - ./logs:/app/logs
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - ANALYTICS_DB_PASSWORD=${ANALYTICS_DB_PASSWORD}
    depends_on:
      - postgres
    networks:
      - sqlatte_network

  postgres:
    image: postgres:15-alpine
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

volumes:
  postgres_data:

networks:
  sqlatte_network:
    driver: bridge
```

**config.yaml:**
```yaml
analytics:
  enabled: true
  storage: "postgresql"
  postgresql:
    host: "postgres"              # Container name
    port: 5432
    database: "sqlatte_analytics"
    user: "sqlatte"
    password: "${ANALYTICS_DB_PASSWORD}"
```

**Start:**
```bash
# Set password in .env
echo "ANALYTICS_DB_PASSWORD=strong_password" >> .env

# Start services
docker-compose up -d

# Check health
docker-compose ps
```

---

## Production Deployment

### With Nginx Reverse Proxy

**docker-compose.production.yml:**
```yaml
version: '3.8'

services:
  sqlatte:
    build: .
    container_name: sqlatte_prod
    restart: unless-stopped
    expose:
      - "8000"
    volumes:
      - ./config:/app/config
      - ./logs:/app/logs
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - DB_PASSWORD=${DB_PASSWORD}
      - ANALYTICS_DB_PASSWORD=${ANALYTICS_DB_PASSWORD}
    env_file:
      - .env.production
    depends_on:
      - postgres
    networks:
      - sqlatte_network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  postgres:
    image: postgres:15-alpine
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

**nginx.conf:**
```nginx
events {
    worker_connections 1024;
}

http {
    upstream sqlatte {
        server sqlatte:8000;
    }

    # Redirect HTTP to HTTPS
    server {
        listen 80;
        server_name analytics.company.com;
        return 301 https://$server_name$request_uri;
    }

    # HTTPS Server
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

**Deploy:**
```bash
# Create SSL directory
mkdir -p ssl

# Get SSL certificate (Let's Encrypt)
sudo certbot certonly --standalone -d analytics.company.com
sudo cp /etc/letsencrypt/live/analytics.company.com/fullchain.pem ssl/cert.pem
sudo cp /etc/letsencrypt/live/analytics.company.com/privkey.pem ssl/key.pem

# Start production stack
docker-compose -f docker-compose.production.yml up -d

# Verify
curl https://analytics.company.com/health
```

---

## Dockerfile

SQLatte includes a production-ready Dockerfile:

```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# Run application
CMD ["python", "run.py"]
```

**Build manually:**
```bash
docker build -t sqlatte:latest .
docker run -d -p 8000:8000 --name sqlatte sqlatte:latest
```

---

## Environment Variables

**.env.production:**
```bash
# LLM Provider
ANTHROPIC_API_KEY=sk-ant-xxxxx

# Database Credentials
DB_HOST=trino.company.com
DB_PORT=443
DB_USER=sqlatte_service
DB_PASSWORD=secure_password

# Analytics Database
ANALYTICS_DB_PASSWORD=analytics_password

# Email/SMTP
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=app_password

# Application
SQLATTE_ENV=production
LOG_LEVEL=INFO
```

**Load in Docker Compose:**
```yaml
services:
  sqlatte:
    env_file:
      - .env.production
```

---

## Docker Commands

### Start/Stop

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# Restart
docker-compose restart

# Stop and remove volumes (⚠️ deletes data!)
docker-compose down -v
```

### Logs

```bash
# Follow all logs
docker-compose logs -f

# SQLatte logs only
docker-compose logs -f sqlatte

# Last 100 lines
docker-compose logs --tail=100 sqlatte

# PostgreSQL logs
docker-compose logs -f postgres
```

### Execute Commands

```bash
# Access SQLatte container
docker-compose exec sqlatte bash

# Run Python command
docker-compose exec sqlatte python -c "from src.core.config import load_config; print(load_config())"

# Access PostgreSQL
docker-compose exec postgres psql -U sqlatte -d sqlatte_analytics
```

### Health Checks

```bash
# Check container health
docker-compose ps

# Check SQLatte health endpoint
curl http://localhost:8000/health

# Check PostgreSQL
docker-compose exec postgres pg_isready -U sqlatte
```

---

## Volumes & Persistence

### Important Volumes

```yaml
volumes:
  # Config files (editable from host)
  - ./config:/app/config
  
  # Logs (accessible from host)
  - ./logs:/app/logs
  
  # PostgreSQL data (persistent)
  - postgres_data:/var/lib/postgresql/data
```

### Backup PostgreSQL

```bash
# Backup
docker-compose exec postgres pg_dump -U sqlatte sqlatte_analytics > backup.sql

# Restore
docker-compose exec -T postgres psql -U sqlatte sqlatte_analytics < backup.sql
```

---

## Scaling & Performance

### Horizontal Scaling (Multiple Instances)

```yaml
version: '3.8'

services:
  sqlatte:
    build: .
    restart: unless-stopped
    deploy:
      replicas: 3              # Run 3 instances
    # ... rest of config

  nginx:
    # Load balancer
    # ... upstream config
```

**Note:** Requires Redis for shared session storage (future feature)

### Resource Limits

```yaml
services:
  sqlatte:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
```

---

## Monitoring

### With Prometheus & Grafana

**docker-compose.monitoring.yml:**
```yaml
version: '3.8'

services:
  # ... (sqlatte, postgres, nginx)

  prometheus:
    image: prom/prometheus
    container_name: prometheus
    restart: unless-stopped
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    networks:
      - sqlatte_network

  grafana:
    image: grafana/grafana
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - sqlatte_network

volumes:
  prometheus_data:
  grafana_data:
```

**prometheus.yml:**
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'sqlatte'
    static_configs:
      - targets: ['sqlatte:8000']
```

---

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker-compose logs sqlatte

# Check config
docker-compose config

# Validate Dockerfile
docker build -t sqlatte:test .
```

### Connection Issues

```bash
# Check network
docker network ls
docker network inspect sqlatte_sqlatte_network

# Ping containers
docker-compose exec sqlatte ping postgres
```

### Database Connection Failed

```bash
# Check PostgreSQL is running
docker-compose ps postgres

# Test connection
docker-compose exec postgres psql -U sqlatte -d sqlatte_analytics

# Check config.yaml
docker-compose exec sqlatte cat /app/config/config.yaml
```

### Port Already in Use

```bash
# Find process using port
lsof -i :8000
# or
netstat -tulpn | grep 8000

# Kill process or change port in docker-compose.yml
ports:
  - "8001:8000"  # Use 8001 on host
```

---

## Security Best Practices

### 1. Use Secrets

```bash
# Don't commit .env files
echo ".env" >> .gitignore
echo ".env.production" >> .gitignore

# Use Docker secrets (Swarm mode)
echo "sk-ant-xxxxx" | docker secret create anthropic_key -
```

### 2. Non-Root User

**Dockerfile:**
```dockerfile
# Create non-root user
RUN useradd -m -u 1000 sqlatte
USER sqlatte
```

### 3. Read-Only Filesystem

```yaml
services:
  sqlatte:
    read_only: true
    tmpfs:
      - /tmp
      - /app/logs
```

### 4. Network Isolation

```yaml
networks:
  sqlatte_network:
    driver: bridge
    internal: true  # No external access
```

---

## Updates & Upgrades

### Update SQLatte

```bash
# Pull latest code
git pull origin main

# Rebuild image
docker-compose build sqlatte

# Restart with new image
docker-compose up -d sqlatte
```

### Update Dependencies

```bash
# Update requirements.txt
pip install -r requirements.txt --upgrade
pip freeze > requirements.txt

# Rebuild
docker-compose build --no-cache
docker-compose up -d
```

---

## Production Checklist

Before deploying to production:

- [ ] Use environment variables for secrets
- [ ] Enable HTTPS with valid SSL certificate
- [ ] Set up PostgreSQL for analytics persistence
- [ ] Configure email/SMTP for scheduled reports
- [ ] Enable health checks
- [ ] Set resource limits (CPU, memory)
- [ ] Configure log rotation
- [ ] Set up monitoring (Prometheus + Grafana)
- [ ] Create backup strategy for PostgreSQL
- [ ] Use read-only config volume
- [ ] Restrict CORS to specific domains
- [ ] Enable rate limiting (nginx)
- [ ] Test disaster recovery

---

## Next Steps

- **Configure Database**: [Database Providers](../configuration/database-providers.md)
- **Configure LLM**: [LLM Providers](../configuration/llm-providers.md)
- **Enable Analytics**: [Analytics Setup](../configuration/analytics.md)
- **Setup Email**: [Email Settings](../configuration/email.md)

---

**Need help?** Check [GitHub Issues](https://github.com/osmanuygar/sqlatte/issues) or [Installation Guide](installation.md).