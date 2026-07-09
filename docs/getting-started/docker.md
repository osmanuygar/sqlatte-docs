# Docker Deployment

## Using Docker Compose (Recommended)

```bash
# 1. Edit config/config.yaml with your credentials
vi config/config.yaml

# 2. Start
docker-compose up -d

# 3. Open
open http://localhost:8002
```

The shipped `docker-compose.yml`:

```yaml
services:
  sqlatte:
    build: .
    ports:
      - "8002:8000"
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY:-}
    restart: unless-stopped
    volumes:
      - ./config:/app/config:ro
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
```

Notes:

- `config/` is **mounted read-only**, not baked into the image — the image itself never contains credentials.
- Prefer editing `config/config.yaml` over passing secrets as environment variables; the file supports `${VAR:default}` interpolation if you still want them sourced from the environment.
- The container drops all Linux capabilities and blocks privilege escalation by default.
- Adjust the host-side port mapping (`"8002:8000"`) to whatever you want exposed; the app always listens on the port set in `config.yaml` (`app.port`) inside the container.

## Using the Dockerfile Directly

```bash
docker build -t sqlatte .

docker run -d -p 8000:8000 \
  -v "$(pwd)/config:/app/config:ro" \
  --name sqlatte \
  sqlatte
```

The image (`python:3.11-slim`) runs as a non-root `sqlatte` user, copies only `src/` and `frontend/`, and creates an empty `config/` placeholder — the real `config.yaml` must be supplied via volume mount at runtime.

## Production Checklist

- Run behind a reverse proxy (nginx, Caddy) with TLS termination
- Point `config_db` at a managed PostgreSQL instance for config persistence and durable audit logs — see [Full Config Reference](../configuration/config-yaml.md)
- Set a strong `admin.password` (or `ADMIN_PASSWORD` env var) — the admin panel fails closed if it's unset
- If exposing MCP over the network, enable `mcp.sse.enabled: true` and require tokens — see [MCP Network Setup](../mcp/network-setup.md)
