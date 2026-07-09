# Installation

## Prerequisites

- Python 3.11+ (3.8+ supported, 3.11-slim is what the Docker image uses)
- An LLM API key ([Anthropic Claude](https://console.anthropic.com/) recommended, or Google Gemini / Vertex AI)
- A database to query: Trino, PostgreSQL, MySQL, or BigQuery

## 1. Clone and Install

```bash
git clone https://github.com/osmanuygar/sqlatte.git
cd sqlatte
pip install -r requirements.txt
```

Key dependencies: FastAPI, Uvicorn, the `mcp` SDK (for the MCP server), your chosen LLM SDK (`anthropic`, `google-generativeai`, or `google-cloud-aiplatform`), and a driver for your database (`trino`, `psycopg2-binary`, `mysql-connector-python`, or `google-cloud-bigquery`).

## 2. Configure

Everything lives in a single file: `config/config.yaml`. At minimum, set an LLM provider and a database:

```yaml
app:
  host: "0.0.0.0"
  port: 8000

llm:
  provider: "anthropic"
  anthropic:
    api_key: "sk-ant-your-key-here"
    model: "claude-sonnet-4-20250514"

database:
  provider: "trino"       # trino | postgresql | mysql | bigquery
  trino:
    host: "your-trino-host.com"
    port: 443
    user: "your-username"
    password: "your-password"
    catalog: "hive"
    schema: "default"
    http_scheme: "https"
```

Values also accept `${ENV_VAR:default}` interpolation, so secrets can be kept out of the file entirely and supplied via environment variables. See the [Full Config Reference](../configuration/config-yaml.md) for every section (analytics, scheduler, email, semantic layer, ops agent, alarms, admin auth, MCP, etc.) — all optional and disabled by default except database/LLM.

## 3. Run

```bash
python run.py
# or
python -m src.api.app
```

For production, run behind Gunicorn with Uvicorn workers:

```bash
gunicorn src.api.app:app -w 4 -k uvicorn.workers.UvicornWorker -b 0.0.0.0:8000
```

## 4. Access

| URL | Purpose |
|---|---|
| `http://localhost:8000/` | Chat interface |
| `http://localhost:8000/admin` | Admin panel (config, prompts, tokens, audit logs) |
| `http://localhost:8000/demo` | Embeddable widget demo |
| `http://localhost:8000/docs` | Auto-generated OpenAPI docs |
| `http://localhost:8000/ops-agent` | BigQuery Ops Console (if `ops_agent.enabled: true`) |

Next: [Quick Start](quickstart.md) for a first query, or [MCP Overview](../mcp/overview.md) to connect Claude Desktop / Claude Code directly to your data.
