# Quick Start

This walks through a first natural-language query, five minutes after cloning the repo.

## 1. Install and Configure

```bash
git clone https://github.com/osmanuygar/sqlatte.git
cd sqlatte
pip install -r requirements.txt
```

Edit `config/config.yaml` with an LLM key and a database connection — see [Installation](installation.md) for the minimal example. Everything else (analytics, scheduler, semantic layer, ops agent, MCP) stays off until you turn it on.

## 2. Start the Server

```bash
python run.py
```

```
============================================================
☕ SQLatte - Natural Language to SQL
============================================================
Starting server on http://0.0.0.0:8000
============================================================
```

## 3. Ask a Question

Open `http://localhost:8000`, pick a table from the schema panel, and ask something like:

```
Show me the top 10 rows ordered by created_at
```

SQLatte runs intent detection, generates SQL against the table's schema, validates it, executes it, and returns results with an AI-generated summary. Ask a follow-up ("what about last week?") and it uses conversation memory to modify the previous query instead of starting over.

## 4. Explore From Here

| Want to... | Go to |
|---|---|
| Connect Claude Desktop / Claude Code to your data | [MCP Overview](../mcp/overview.md) |
| Embed the chat widget in your own site | [UI Guide](../ui-guide.md) |
| Improve SQL accuracy with business-friendly table names | [Semantic Layer](../features/semantic-layer.md) |
| Schedule a recurring report | [Scheduled Queries](../features/schedules.md) |
| Run BigQuery cost/security/performance audits | [BigQuery Ops Console](../features/ops-console.md) |
| See every config option | [Full Config Reference](../configuration/config-yaml.md) |
| Deploy with Docker | [Docker Deployment](docker.md) |
