# System Overview

## High-Level Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│  Entry Points                                                     │
│  Chat UI  │  Embedded Widgets  │  Admin Panel  │  MCP Clients     │
│           │  (standard/auth)   │                │  (stdio / SSE)  │
└─────┬─────┴──────────┬─────────┴────────┬───────┴────────┬────────┘
      │                │                  │                │
┌─────▼────────────────▼──────────────────▼────────────────▼────────┐
│                    API Layer (FastAPI)                             │
│  /query  │  /auth/*  │  /admin/*  │  /api/*  │  /mcp  │  /ops-agent│
└─────┬────────────────────────────────────────────────────┬────────┘
      │                                                     │
┌─────▼─────────────────────────────────────────────────────▼────────┐
│                  Core Processing Layer                              │
│  Intent Detection → SQL Generation → Validation & Risk Scoring      │
│  → Execution → Field Masking (MCP) → Audit Log                      │
│  Conversation Memory  │  Insights Engine  │  Dashboard Generator    │
└─────┬─────────────────────────────────────────────────────┬────────┘
      │                                                     │
┌─────▼──────────────────────────┐   ┌─────────────────────▼────────┐
│  Database Provider Factory     │   │  Ops Agent (BigQuery)         │
│  Trino │ PostgreSQL │ MySQL │  │   │  Cost / Security / Perf ops   │
│  BigQuery                      │   │  Alarm Service (APScheduler)  │
└─────────────────────────────────┘  └───────────────────────────────┘
```

Every request — chat UI, embedded widget, or MCP client — passes through the same core pipeline: intent detection, SQL generation, validation, execution, audit logging. There is no separate, unaudited path for AI agents. See [Security Overview](../security/overview.md) for what validation actually checks.

## Component Map

| Layer | Responsibility |
|---|---|
| **Frontend** (`frontend/`) | Server-rendered HTML pages (chat, admin, dashboards, scheduler, ops console) + two embeddable widget scripts |
| **API** (`src/api/`) | FastAPI routers — one file per feature area (admin, dashboards, schedules, semantic, ops agent, alarms, audit, analytics, demo), plus `mcp_sse.py` for the network MCP transport |
| **Core** (`src/core/`) | Config loading/management, SQL validation, conversation memory, insights engine, scheduler, email, dashboard manager, config-DB persistence |
| **Providers** (`src/providers/`) | Pluggable database (`database/`), LLM (`llm/`), and ops-agent (`ops_agents/`) implementations behind a common factory interface |
| **Plugins** (`src/plugins/`) | Optional, config-gated extensions — the auth plugin (multi-tenant sessions) is the reference implementation; see [Plugin Architecture](principles.md#plugin-architecture) |
| **MCP** (`sqlatte_mcp_server.py`, `src/api/mcp_sse.py`) | Local (stdio) and network (SSE) MCP transports — both call into the exact same `/auth/*` endpoints the auth widget uses |

## Provider Factory Pattern

Database and LLM providers implement a common interface and are instantiated at startup based on `database.provider` / `llm.provider` in config — swapping Trino for BigQuery, or Anthropic for Vertex AI, is a config change, not a code change. The same pattern extends to `ops_agent.provider` for BigQuery Ops.

## Conversation Memory

Session-scoped, in-process by default. A follow-up like "what about last month?" is resolved against the prior turn's SQL and schema context rather than starting from scratch — see `src/core/conversation_manager.py`.

## Persistence Model

Most features degrade gracefully from PostgreSQL to in-memory/SQLite when their backing store isn't configured, so the platform is usable zero-config and scales up feature-by-feature:

| Feature | In-memory/SQLite (default) | PostgreSQL (opt-in) |
|---|---|---|
| Query history, favorites | Yes | via `analytics.enabled` |
| Dashboards | Yes | via `analytics.enabled` |
| Semantic layer | Yes | via `config_db.enabled` |
| Config overrides, API tokens, audit logs, mask rules | Process memory only | via `config_db.enabled` |
| Alarms | JSON file (`alarms.storage_path`) | — |

For anything beyond a single dev instance, enable `config_db` — see [Full Config Reference](../configuration/config-yaml.md#config_db).
