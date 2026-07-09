# Semantic Layer

A business-intelligence metadata layer on top of raw tables: it teaches the AI the *meaning* behind your schema — business-friendly names, how tables relate, and how key metrics should be calculated — so SQL generation is more accurate and consistent across users.

## What It Solves

Without it, the LLM only sees raw column/table names (`cust_tbl_v2`, `ltv`, `dt`). With it, "show me customers with high lifetime value" resolves to the right table, the right column, and the right JOIN — every time, for every user, because the mapping is defined once centrally instead of relied upon per-prompt.

## Concepts

| Concept | What it defines |
|---|---|
| **Entity** | A business-friendly name + description for a table, plus per-column display names |
| **Relationship** | A JOIN path between two entities (e.g. `customer.id → orders.customer_id`), used automatically without the user specifying it |
| **Metric** | A named, centrally-defined calculation (e.g. `total_revenue = SUM(orders.amount)`) so "revenue" means the same thing in every query |

## Auto-Discovery

Scans the connected database and suggests entity definitions automatically — a starting point rather than hand-writing every entity from scratch. Available via `POST /api/semantic/discover` or the **Auto-Discover** tab in the admin panel.

## Admin UI

`/admin` → **Semantic Layer**, five sub-tabs: Entities, Relationships, Metrics, Auto-Discover, How to Use.

## REST API

| Endpoint | Purpose |
|---|---|
| `/api/semantic/entities` | CRUD for entity definitions |
| `/api/semantic/entities/{id}/columns`, `/api/semantic/columns` | Column-level display names |
| `/api/semantic/relationships` | CRUD for JOIN relationships |
| `/api/semantic/metrics` | CRUD for calculated metrics |
| `/api/semantic/context` | Get the assembled semantic context as sent to the LLM |
| `/api/semantic/discover` | Auto-discovery scan |

## How It Reaches the LLM

Business names, dimension/metric definitions, and JOIN instructions are injected into the SQL-generation prompt automatically (`semantic_prompt_enhancer.py`) — no manual prompt editing required, though the base [SQL generation prompt](runtime-prompts.md) can still be customized on top of it.

## Storage

PostgreSQL when [`config_db`](../configuration/config-yaml.md#config_db) is enabled, with an in-memory SQLite fallback otherwise — same pattern as most other persisted features.

!!! note "Beta"
    The semantic layer shipped as beta in v0.5.0. The UI and API surface may still evolve; existing raw-schema queries are unaffected either way — this is additive.
