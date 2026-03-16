# 🧠 Semantic Layer

> **Status:** Beta (v0.5.0+)

The Semantic Layer adds a **business intelligence metadata layer** on top of your raw database tables. It teaches the AI the meaning behind your data — business-friendly names, how tables relate to each other, and how key metrics should be calculated — so that everyone gets consistent, correct SQL.

---

## Why Use It?

Without the semantic layer, the LLM only sees raw table and column names:

```
cust_tbl_v2, ord_fact, amt, cust_rgn_cd
```

With the semantic layer, the LLM sees:

```
📊 Customer Master (hive.default.cust_tbl_v2)
   Dimensions: region, customer_type, segment

📊 Sales Transactions (hive.default.ord_fact)
   Metrics: amount, quantity

🔗 customer_orders: orders.customer_id → customers.id (LEFT JOIN)

📈 Total Revenue: SUM(orders.amount)  [currency]
```

**Result:** "Show me revenue by region" generates correct, JOIN-aware SQL every time — using the business-approved formula.

---

## Core Concepts

| Concept | What It Does |
|---|---|
| **Entity** | Maps a raw table to a business-friendly name + description. Marks which columns are dimensions (group by) and which are metrics (aggregate). |
| **Relationship** | Defines how two entities join. Once defined, the AI uses the exact ON clause — no guessing. |
| **Metric** | A named, centralized SQL expression (e.g. `SUM(orders.amount)` = "Total Revenue"). Everyone gets the same number. |

---

## Prerequisites

Semantic Layer requires `config_db` to be enabled (PostgreSQL). It uses the same database as the Config DB.

```yaml
# config/config.yaml
config_db:
  enabled: true
  postgresql:
    host: "localhost"
    port: 5432
    database: "sqlatte_config"
    user: "postgres"
    password: "your_password"
```

> If `config_db.enabled` is `false`, the Semantic Layer initializes in **in-memory SQLite** mode — metadata is lost on restart.

---

## Admin UI

Navigate to `http://localhost:8000/admin` → **🧠 Semantic** tab.

The tab has five sub-sections:

| Sub-tab | Purpose |
|---|---|
| 📊 **Entities** | Create / edit / delete entity definitions |
| 🔗 **Relationships** | Define JOIN paths between entities |
| 📈 **Metrics** | Create calculated metrics with SQL expressions |
| 🔍 **Auto-Discover** | Scan your database and get instant entity suggestions |
| ℹ️ **How to Use** | Built-in integration guide |

---

## Step-by-Step Setup

### 1. Create Entities

An entity is a table (or view) with business context attached.

**Via Admin UI:**
1. Go to **Semantic → Entities → + Add Entity**
2. Fill in:
   - **Catalog** — e.g. `hive` (leave empty for PostgreSQL/MySQL)
   - **Schema** — e.g. `default`, `public`
   - **Table Name** — exact table name as in the database
   - **Display Name** — business-friendly name, e.g. `Customer Master`
   - **Description** — what this table represents
   - **Primary Key** — e.g. `id`
3. After creating, add **columns** and tag each as:
   - **Dimension** — fields used in `GROUP BY` (region, category, date, status…)
   - **Metric** — numeric fields used in aggregations (amount, quantity, count…)

**Via API:**
```http
POST /api/semantic/entities
Content-Type: application/json

{
  "catalog": "hive",
  "schema_name": "default",
  "table_name": "orders",
  "display_name": "Sales Transactions",
  "description": "All order records",
  "primary_key": "order_id",
  "entity_type": "table"
}
```

### 2. Define Relationships

A relationship tells the AI exactly how to JOIN two entities.

**Via Admin UI:**
1. Go to **Semantic → Relationships → + Add Relationship**
2. Fill in:
   - **Name** — e.g. `customer_orders`
   - **From Entity / Column** — e.g. `orders.customer_id`
   - **To Entity / Column** — e.g. `customers.id`
   - **Type** — `many_to_one`, `one_to_many`, `one_to_one`
   - **Join Type** — `LEFT`, `INNER`, `RIGHT`

**Via API:**
```http
POST /api/semantic/relationships
Content-Type: application/json

{
  "name": "customer_orders",
  "from_entity_id": 2,
  "from_column": "customer_id",
  "to_entity_id": 1,
  "to_column": "id",
  "relationship_type": "many_to_one",
  "join_type": "LEFT",
  "description": "Each order belongs to one customer"
}
```

### 3. Create Metrics

A metric is a named SQL expression representing a business calculation.

**Via Admin UI:**
1. Go to **Semantic → Metrics → + Add Metric**
2. Fill in:
   - **Name** — e.g. `total_revenue`
   - **Display Name** — e.g. `Total Revenue`
   - **SQL Expression** — e.g. `SUM(orders.amount)`
   - **Format** — `currency`, `percentage`, or blank
   - **Category** — optional grouping label (e.g. `Finance`, `Operations`)

**Via API:**
```http
POST /api/semantic/metrics
Content-Type: application/json

{
  "name": "total_revenue",
  "display_name": "Total Revenue",
  "sql_expression": "SUM(orders.amount)",
  "format_type": "currency",
  "category": "Finance",
  "description": "Sum of all completed order amounts"
}
```

### 4. Auto-Discover (Optional)

Instead of creating entities manually, let SQLatte scan the database and suggest definitions:

1. Go to **Semantic → Auto-Discover**
2. Enter the catalog and schema to scan
3. Optionally specify table names to limit the scan
4. Click **Discover** — the system suggests entities with column analysis
5. Review suggestions and import the ones you want

**Via API:**
```http
POST /api/semantic/discover
Content-Type: application/json

{
  "catalog": "hive",
  "schema": "default",
  "tables": ["customers", "orders", "products"]
}
```

---

## How It Affects SQL Generation

Once entities, relationships, and metrics are defined, the Semantic Layer automatically injects the following context into every LLM prompt:

```
=== BUSINESS CONTEXT (Semantic Layer) ===

📊 Available Entities (Tables with Business Names):
  • hive.default.customers
    Business Name: Customer Master
    Description: Central repository of customer information
    Dimensions: region, customer_type, segment
    Primary Key: id

  • hive.default.orders
    Business Name: Sales Transactions
    Dimensions: order_date, status
    Metrics: amount, quantity

🔗 Available Relationships (Automatic JOINs):
  • customer_orders: orders.customer_id → customers.id (many_to_one)

📈 Calculated Metrics (Business Logic):
  • Total Revenue
    Description: Sum of all order amounts
    SQL: SUM(orders.amount)
    Format: currency

=== SEMANTIC LAYER INSTRUCTIONS ===
When joining tables:
1. Check if a relationship exists in "Available Relationships"
2. Use the exact join condition specified
3. Prefer LEFT JOIN unless specified otherwise

When calculating metrics:
1. Check if a calculated metric exists in "Calculated Metrics"
2. Use the exact SQL expression provided — do not recalculate
```

The LLM follows these instructions automatically. No prompt editing required.

---

## API Reference

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/semantic/entities` | List entities (filter by `catalog`, `schema_name`) |
| `POST` | `/api/semantic/entities` | Create entity |
| `DELETE` | `/api/semantic/entities/{id}` | Delete entity |
| `POST` | `/api/semantic/entities/{id}/columns` | Add column to entity |
| `GET` | `/api/semantic/relationships` | List relationships |
| `POST` | `/api/semantic/relationships` | Create relationship |
| `DELETE` | `/api/semantic/relationships/{id}` | Delete relationship |
| `GET` | `/api/semantic/metrics` | List metrics (`active_only=true` by default) |
| `POST` | `/api/semantic/metrics` | Create metric |
| `DELETE` | `/api/semantic/metrics/{id}` | Delete metric |
| `GET` | `/api/semantic/context` | Get full semantic context (as injected into LLM) |
| `POST` | `/api/semantic/discover` | Auto-discover entities from database |

---

## Storage

Semantic Layer metadata is stored in **4 PostgreSQL tables** inside the `sqlatte_config` database:

| Table | Content |
|---|---|
| `semantic_entities` | Entity definitions (table mappings + display names) |
| `semantic_entity_columns` | Column definitions per entity (dimensions / metrics) |
| `semantic_relationships` | JOIN definitions between entities |
| `semantic_metrics` | Calculated metric expressions |

Tables are created automatically on first startup if they do not exist.

---

## Backward Compatibility

The Semantic Layer is **100% additive**. Existing queries work unchanged when:

- `config_db.enabled: false` — the enhancer returns the original schema unmodified
- `config_db.enabled: true` but no entities defined — context block is skipped entirely

No changes to existing `config.yaml` prompts or query behavior are required.

---

## Troubleshooting

**Semantic tab not appearing in Admin Panel**
- The tab is always present. If it shows an error, check that `config_db` is reachable and the server started without errors.

**Auto-Discover returns no tables**
- Verify the catalog/schema values match exactly what your database provider uses.
- For Trino, the catalog must match the Trino catalog name (e.g. `hive`).
- For PostgreSQL, leave catalog empty and set schema to `public`.

**LLM is not using semantic context**
- Confirm at least one active entity exists via `GET /api/semantic/context`.
- If the context response shows empty arrays, add entities first.
- Check server logs for `⚠️ Semantic layer not available` messages at startup.

**Metrics produce wrong results**
- The SQL expression in a metric must be a valid aggregate expression for your target database.
- Test the expression directly in the query interface before saving.

**Semantic layer initialized in-memory (data lost on restart)**
- Enable `config_db` with a PostgreSQL connection. In-memory mode is only suitable for local development and testing.