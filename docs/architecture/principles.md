# Design Principles

SQLatte is built on a foundation of **optional complexity** - start simple, add features when needed. This philosophy permeates every architectural decision, from the plugin system to the configuration management.

## Core Design Principles

### 1. Widget-First Architecture 

**Philosophy:** SQLatte is designed to be embeddable anywhere, not just a standalone application.

**Key Decisions:**
- **Two Widget Modes:**
  - **Standard Widget** - Config-based auth, shared database connection
  - **Auth Widget** - Per-user credentials, session isolation
- **Single-File Components** - All HTML/CSS/JS in one file for easy embedding
- **CORS-Ready** - Cross-domain support built-in
- **Zero Dependencies** - Pure JavaScript, no framework lock-in

**Benefits:**
```html
<!-- Embed in any site with 2 lines -->
<script src="https://your-server/static/js/sqlatte-badge.js"></script>
<script>SQLatteWidget.configure({apiBase: 'https://your-server'});</script>
```

**Design Trade-offs:**
- ✅ **Pro:** Extremely easy to integrate anywhere
- ✅ **Pro:** No build process for users
- ⚠️ **Con:** Larger initial JavaScript bundle
- ⚠️ **Con:** Limited to vanilla JS patterns

---

### 2. Optional Complexity 

**Philosophy:** Every feature beyond basic NL-to-SQL is optional and can be disabled.

**Progressive Feature Adoption:**
```yaml
# Phase 1: Basic (Day 1)
database: {...}
llm: {...}

# Phase 2: Add Analytics (Week 1)
analytics:
  enabled: true

# Phase 3: Add Sessions (Month 1)
plugins:
  auth:
    enabled: true

# Phase 4: Add Scheduling (Month 2)
scheduler:
  enabled: true
email:
  enabled: true
```

**Benefits:**
- ✅ Quick onboarding (5 minutes to first query)
- ✅ Scale complexity with needs
- ✅ Lower resource usage when features disabled
- ✅ Easier debugging (fewer moving parts)

**Implementation:**
- Features check `enabled` flag at runtime
- Graceful degradation (e.g., email defaults to mock mode)
- No startup errors when optional features disabled

---

### 3. Conversational Data Analytics 

**Philosophy:** Data exploration should feel like a conversation, not writing code.

**Conversation Memory System:**
```python
# Session-Based Memory
User: "Show me top customers"
  ↓ [Stores in session]
Assistant: [Returns SQL + Results]
  ↓ [Stores in session]
User: "What about last month?"
  ↓ [Uses context from session]
Assistant: [Understands "last month" refers to customers]
```

**Architecture:**
```
ConversationManager (In-Memory)
  ├── Session 1 (User A)
  │   ├── Message 1: "Show top products"
  │   ├── Message 2: [SQL Result]
  │   ├── Message 3: "What about revenue?"
  │   └── Message 4: [SQL Result with context]
  └── Session 2 (User B)
      └── Independent conversation
```

**Key Components:**
- `ConversationSession` - Single conversation thread
- `ConversationMessage` - Individual message (user/assistant)
- `ConversationManager` - Manages multiple sessions
- **Session Timeout:** Configurable (default 60 min)
- **Context Window:** Last 10 messages sent to LLM

**Benefits:**
- ✅ Natural follow-up questions
- ✅ Context-aware query refinement
- ✅ Reduced user effort (no repeated details)
- ⚠️ **Limitation:** In-memory only (resets on restart)

---

### 4. Plugin-Based Extensibility 

**Philosophy:** Core should be minimal, extensions should be modular.

**Plugin Architecture:**
```python
BasePlugin (Abstract)
    ├── initialize(app)       # Called on startup
    ├── register_routes(app)  # Add custom endpoints
    ├── before_request()      # Request middleware
    ├── after_request()       # Response middleware
    └── shutdown()            # Cleanup

AuthPlugin (Implementation)
    ├── /auth/login           # Custom endpoint
    ├── /auth/query           # Override default query
    ├── SessionManager        # Per-user sessions
    └── ThreadPoolExecutor    # Concurrent user isolation
```

**Plugin System Benefits:**
- ✅ **Isolation:** Plugins don't interfere with core
- ✅ **Hot-Swappable:** Enable/disable without code changes
- ✅ **Backward Compatible:** Standard widget works without plugins
- ✅ **Extensible:** Add Slack plugin, webhooks, custom auth

**Plugin Examples:**

**Auth Plugin:**
```yaml
plugins:
  auth:
    enabled: true
    session_ttl_minutes: 480
    max_workers: 40
```
**Adds:** Multi-tenant auth, per-user DB connections

**Custom Integration Plugin:**
```python
class SlackPlugin(BasePlugin):
    def register_routes(self, app):
        @app.post("/slack/query")
        async def slack_query(payload):
            # Handle Slack slash command
            return {"text": "Results..."}
```
**Adds:** Slack integration for team queries

---

### 5. Database Provider Abstraction

**Philosophy:** Support multiple databases through a unified interface.

**Provider Pattern:**
```python
DatabaseProvider (Abstract)
    ├── execute_query(sql) → (columns, data)
    ├── get_tables() → List[str]
    └── get_table_schema(table) → str

TrinoProvider (Implementation)
    └── Uses trino-python-client

PostgreSQLProvider (Implementation)
    └── Uses psycopg2

MySQLProvider (Implementation)
    └── Uses mysql-connector-python

BigQueryProvider (Implementation)
    └── Uses google-cloud-bigquery
```

**Factory Pattern:**
```python
ProviderFactory.create_db_provider(config)
  ↓
  Reads config['database']['provider']
  ↓
  Returns: TrinoProvider | PostgreSQLProvider | MySQLProvider | BigQueryProvider
```

**Benefits:**
- ✅ **Uniform API:** Same interface for all databases
- ✅ **Easy Migration:** Switch providers via config
- ✅ **Database-Specific Optimizations:** Each provider can optimize queries
- ✅ **Extensible:** Add new providers without touching core

**Design Considerations:**
- SQL syntax differences handled by prompts (database-specific rules)
- Connection pooling per provider
- Graceful error handling (DB-specific error messages)

---

### 6. LLM Provider Abstraction 

**Philosophy:** Not locked to a single AI provider.

**Provider Pattern:**
```python
LLMProvider (Abstract)
    ├── determine_intent(question, schema) → dict
    ├── generate_sql(question, schema) → (sql, explanation)
    ├── generate_chat_response(question) → str
    └── generate_insights(results) → List[Insight]

AnthropicProvider (Implementation)
    └── Uses anthropic SDK

GeminiProvider (Implementation)
    └── Uses google-generativeai

VertexAIProvider (Implementation)
    └── Uses google-cloud-aiplatform
```

**Benefits:**
- ✅ **Flexibility:** Choose best LLM for your use case
- ✅ **Cost Optimization:** Switch to cheaper provider for simple queries
- ✅ **A/B Testing:** Test prompt quality across providers
- ✅ **Fallback:** If one provider has issues, switch to another

---

### 7. Configuration Management 

**Philosophy:** Single source of truth, multiple override mechanisms.

**Configuration Hierarchy:**
```
1. config.yaml (Base configuration)
    ↓
2. Environment Variables (Override specific values)
    ↓
3. Runtime Overrides (Admin panel changes)
    ↓
4. Database Persistence (Saved overrides)
```

**Hot Reload Architecture:**
```python
ConfigManager
    ├── config.yaml (loaded on startup)
    ├── runtime_overrides (in-memory dict)
    ├── ConfigDB (optional persistent storage)
    └── reload() → Merge config + overrides
```

**Benefits:**
- ✅ **Environment-Specific:** Dev/staging/prod via env vars
- ✅ **Hot Reload:** Change LLM params without restart
- ✅ **Audit Trail:** Track all config changes
- ✅ **Rollback:** Snapshots for config recovery

**Key Features:**
- **Prompts Management:** Edit AI prompts via admin UI
- **Provider Switching:** Change LLM/DB without restart (where possible)
- **Secrets Management:** Support for `${ENV_VAR}` syntax
- **Validation:** Schema validation on load

---

### 8. Scheduled Execution System 

**Philosophy:** Automation should be simple, not enterprise-complex.

**Architecture:**
```
APScheduler (Background Jobs)
    ├── Cron Expressions (flexible scheduling)
    ├── Job Queue (in-memory or Redis)
    └── Execution Logs (PostgreSQL or in-memory)

Email Service
    ├── SMTP Client (sendgrid, gmail, etc.)
    ├── Template Engine (HTML reports)
    ├── Attachment Builder (CSV, Excel, HTML)
    └── Mock Mode (when SMTP disabled)

ScheduleManager
    ├── create_schedule()
    ├── execute_schedule()
    └── get_execution_history()
```

**Graceful Degradation:**
```python
if email.enabled:
    send_actual_email()
else:
    # Mock mode - logs but doesn't send
    logger.info(f" Mock: Would send to {recipients}")
```

**Benefits:**
- ✅ **Flexible Scheduling:** Cron syntax (daily, weekly, monthly)
- ✅ **Email Delivery:** Automated report distribution
- ✅ **Execution History:** Track success/failure
- ✅ **Works Without Email:** Scheduler runs, logs results

---

### 9. Analytics & Insights

**Philosophy:** Make data analysis accessible, not just query execution.

**Two-Tier Analytics:**

**Tier 1: Query Analytics (Optional)**
```
PostgreSQL Storage (or In-Memory)
    ├── query_history table
    ├── execution_logs table
    └── insights_cache table
```

**Tier 2: Insights Engine**
```
Hybrid Mode (Recommended)
    ├── LLM Insights (pattern detection, recommendations)
    └── Statistical Analysis (trends, anomalies)
```

**Insights Generation Flow:**
```
Query Results
    ↓
[Statistical Analysis]
    ├── Detect trends (growth/decline)
    ├── Find anomalies (outliers)
    └── Calculate metrics (avg, median, etc.)
    ↓
[LLM Analysis]
    ├── Context: User question + SQL + Results
    ├── Generate: Actionable insights
    └── Format: TYPE | SEVERITY | MESSAGE
    ↓
[Merge & Return]
    └── Best insights from both approaches
```

**Benefits:**
- ✅ **Hybrid Approach:** LLM creativity + statistical accuracy
- ✅ **Context-Aware:** Considers temporal patterns
- ✅ **Actionable:** Recommendations, not just observations
- ✅ **Optional:** Can disable entirely for lightweight deployment

---

### 10. Security & Multi-Tenancy 

**Philosophy:** Secure by design, with layers of isolation.

**Standard Widget (Single-Tenant):**
```
User → Backend Config → Shared DB Connection
    ├── No user credentials stored
    ├── Backend controls all access
    └── Suitable for internal tools
```

**Auth Widget (Multi-Tenant):**
```
User A → Login → Session A → DB Config A → Isolated Connection
User B → Login → Session B → DB Config B → Isolated Connection

SessionManager (Thread-Safe)
    ├── Per-user sessions
    ├── Credential encryption
    ├── Session timeouts
    └── Automatic cleanup
```

**Security Layers:**

1. **Session Isolation:**
```python
   class SessionManager:
       sessions: Dict[str, AuthSession]  # session_id → session
       lock = threading.RLock()  # Thread-safe access
```

2. **Credential Isolation:**
```python
   # User credentials never stored in logs
   db_config = {
       'user': session.username,
       'password': '***REDACTED***'  # Masked in logs
   }
```

3. **Connection Pooling:**
```python
   ThreadPoolExecutor(max_workers=40)
       └── Isolated query execution per user
```

**Benefits:**
- ✅ **Session-Based Auth:** JWT-free, simple
- ✅ **Per-User Connections:** Complete data isolation
- ✅ **Thread-Safe:** Concurrent users don't interfere
- ✅ **Automatic Cleanup:** Expired sessions removed

---

## Design Trade-offs

### 1. In-Memory vs Persistent Conversations

**Current:** In-memory (simple, fast)
```python
ConversationManager.sessions = {}  # RAM
```

**Trade-off:**
- ✅ **Pro:** Fast, no DB overhead
- ⚠️ **Con:** Lost on restart
- ⚠️ **Con:** Not shared across instances

**Future:** Optional PostgreSQL backend

### 2. APScheduler vs Celery

**Current:** APScheduler (simpler)

**Trade-off:**
- ✅ **Pro:** Zero dependencies, built-in
- ✅ **Pro:** Works without Redis
- ⚠️ **Con:** Limited to single instance
- ⚠️ **Con:** No distributed task queue

### 3. Vanilla JS vs React

**Current:** Vanilla JavaScript

**Trade-off:**
- ✅ **Pro:** No build step for users
- ✅ **Pro:** Smaller bundle (no framework)
- ⚠️ **Con:** More verbose code
- ⚠️ **Con:** No component state management

---

## Extension Points

### 1. Custom Plugins
```python
from src.plugins.base_plugin import BasePlugin

class CustomPlugin(BasePlugin):
    def register_routes(self, app):
        @app.get("/custom")
        async def custom_route():
            return {"message": "Custom feature"}
```

### 2. Custom Database Providers
```python
from src.core.db_provider import DatabaseProvider

class CustomDBProvider(DatabaseProvider):
    def execute_query(self, sql):
        # Custom DB logic
        pass
```

### 3. Custom LLM Providers
```python
from src.core.llm_provider import LLMProvider

class CustomLLMProvider(LLMProvider):
    def generate_sql(self, question, schema):
        # Custom LLM logic
        pass
```

---

## Future Architecture Enhancements

### Planned Features

1. **Distributed Sessions**

   - Redis-backed conversation storage
   - Shared sessions across instances

2. **Advanced Caching**

   - Query result caching (Redis)
   - Semantic similarity matching
   - Deduplicate identical queries

3. **Webhooks & Integrations**

   - Slack plugin
   - MS Teams plugin
   - Custom webhook endpoints

4. **Advanced Auth**

   - OAuth2 support
   - SAML integration
   - RBAC (Role-Based Access Control)

5. **Query Optimization**

   - Query plan analysis
   - Automatic index suggestions
   - Partition recommendation engine

---

## Summary

SQLatte's architecture prioritizes:

1. ✅ **Simplicity** - Get started in 5 minutes
2. ✅ **Flexibility** - Choose your database, LLM, features
3. ✅ **Extensibility** - Plugin system for custom needs
4. ✅ **Conversation-First** - Memory and context awareness
5. ✅ **Optional Complexity** - Add features when needed

**Core Philosophy:** Make data analytics conversational, embeddable, and progressively complex.

---

**Next:** [System Overview](overview.md) for component-level architecture details.