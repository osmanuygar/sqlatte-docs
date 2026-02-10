# System Overview

This document provides a detailed technical overview of SQLatte's system architecture, component interactions, and data flow patterns.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Interfaces                          │
├──────────────┬──────────────┬──────────────┬────────────────────┤
│ Standard     │ Auth Widget  │ Dashboard    │ Admin Panel        │
│ Widget       │ (Multi-user) │ (Unified)    │ (Config)           │
└──────┬───────┴──────┬───────┴──────┬───────┴────────┬───────────┘
       │              │              │                │
       └──────────────┴──────────────┴────────────────┘
                              ↓
       ┌──────────────────────────────────────────────────┐
       │              FastAPI Backend                      │
       │  ┌────────────────────────────────────────────┐  │
       │  │         API Routes & Endpoints             │  │
       │  ├────────────────────────────────────────────┤  │
       │  │  /api/query    /auth/login   /admin/*     │  │
       │  └────────────────────────────────────────────┘  │
       └──────────────────────────────────────────────────┘
                              ↓
       ┌──────────────────────────────────────────────────┐
       │            Core Business Logic                    │
       ├──────────────┬───────────────┬───────────────────┤
       │ Conversation │ Session       │ Query History     │
       │ Manager      │ Manager       │ Manager           │
       ├──────────────┼───────────────┼───────────────────┤
       │ Config       │ Scheduler     │ Email Service     │
       │ Manager      │ Manager       │                   │
       └──────────────┴───────────────┴───────────────────┘
                              ↓
       ┌──────────────────────────────────────────────────┐
       │         Provider Abstraction Layer                │
       ├──────────────────────┬───────────────────────────┤
       │  LLM Providers        │  Database Providers       │
       │  - Anthropic Claude   │  - Trino                  │
       │  - Google Gemini      │  - PostgreSQL             │
       │  - Vertex AI          │  - MySQL                  │
       │                       │  - BigQuery               │
       └───────────────────────┴───────────────────────────┘
                              ↓
       ┌──────────────────────────────────────────────────┐
       │           External Systems                        │
       ├──────────────────────┬───────────────────────────┤
       │  LLM APIs             │  User Databases           │
       │  - api.anthropic.com  │  - Production DBs         │
       │  - generativelanguage │  - Data Warehouses        │
       │  - aiplatform.google  │  - Analytics DBs          │
       └───────────────────────┴───────────────────────────┘
```

---

## Core Components

### 1. API Layer (FastAPI)

**Location:** `src/api/app.py`

**Responsibilities:**
- HTTP request handling
- Route registration
- CORS configuration
- Plugin initialization
- Error handling

**Key Endpoints:**

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/query` | POST | Execute natural language query (standard widget) |
| `/api/tables` | GET | List available database tables |
| `/api/schema` | POST | Get table schema information |
| `/auth/login` | POST | Authenticate user (auth widget) |
| `/auth/query` | POST | Execute query with user credentials |
| `/auth/tables` | GET | List tables for authenticated user |
| `/admin/*` | GET/POST | Admin panel endpoints |
| `/analytics/*` | GET | Analytics dashboard endpoints |
| `/health` | GET | Health check |

**Request Flow Example:**

```python
@app.post("/api/query")
async def query_endpoint(request: QueryRequest):
    # 1. Get/create conversation session
    session_id, session = conversation_manager.get_or_create_session(
        request.session_id
    )
    
    # 2. Add user message to session
    conversation_manager.add_message(
        session_id, 
        role="user", 
        content=request.question
    )
    
    # 3. Get conversation context
    context = conversation_manager.get_conversation_context(session_id)
    
    # 4. Determine intent (SQL vs chat)
    intent = llm_provider.determine_intent(
        request.question, 
        request.schema
    )
    
    # 5. Execute based on intent
    if intent == "sql":
        sql, explanation = llm_provider.generate_sql(...)
        columns, data = db_provider.execute_query(sql)
        insights = insights_engine.generate_insights(...)
        result = {"sql": sql, "data": data, "insights": insights}
    else:
        response = llm_provider.generate_chat_response(...)
        result = {"message": response}
    
    # 6. Add assistant response to session
    conversation_manager.add_message(
        session_id,
        role="assistant",
        content=result
    )
    
    # 7. Log to analytics (if enabled)
    query_history.add_query(...)
    
    return result
```

---

### 2. Conversation Manager

**Location:** `src/core/conversation_manager.py`

**Purpose:** Manages conversation sessions and message history for context-aware responses.

**Class Structure:**

```python
class ConversationMessage:
    role: str           # "user" or "assistant"
    content: str        # Message content
    metadata: dict      # Additional data
    timestamp: datetime # When message was created

class ConversationSession:
    session_id: str                      # Unique session ID
    messages: List[ConversationMessage]  # Message history
    created_at: datetime                 # Session creation time
    last_activity: datetime              # Last message timestamp
    
    def add_message(role, content, metadata)
    def get_llm_context(max_messages=10) → List[dict]
    def is_expired(timeout_minutes) → bool

class ConversationManager:
    sessions: Dict[str, ConversationSession]  # session_id → session
    session_timeout_minutes: int              # Default: 60
    max_context_messages: int                 # Default: 10
    
    def create_session() → str
    def get_or_create_session(session_id) → (session_id, session)
    def add_message(session_id, role, content, metadata)
    def get_conversation_context(session_id) → List[dict]
```

**Key Features:**
- ✅ **In-Memory Storage:** Fast access, no DB overhead
- ✅ **Session Timeout:** Auto-expire inactive sessions
- ✅ **Context Window:** Last N messages sent to LLM
- ✅ **Thread-Safe:** Supports concurrent sessions

**Conversation Flow:**

```
User: "Show me top customers"
    ↓
ConversationManager.add_message(role="user", content="Show me...")
    ↓
Session.messages = [
    {role: "user", content: "Show me top customers"}
]
    ↓
LLM generates SQL and executes
    ↓
ConversationManager.add_message(role="assistant", content={sql, data})
    ↓
Session.messages = [
    {role: "user", content: "Show me top customers"},
    {role: "assistant", content: {sql: "SELECT...", data: [...]}}
]
    ↓
User: "What about last month?"
    ↓
ConversationManager.get_conversation_context()
    ↓
Returns last 10 messages to LLM for context
    ↓
LLM understands "last month" refers to customers query
```

---

### 3. Session Manager (Auth Plugin)

**Location:** `src/plugins/session_manager.py`

**Purpose:** Manages authenticated user sessions for multi-tenant deployments.

**Class Structure:**

```python
class AuthSession:
    session_id: str          # Unique session ID
    username: str            # User identifier
    db_config: Dict          # User's database credentials
    created_at: datetime     # Session start time
    last_activity: datetime  # Last request time
    ttl_minutes: int         # Session timeout (default: 480 = 8 hours)
    conversation_id: str     # Linked conversation session
    
    def is_expired() → bool
    def touch()  # Update last_activity

class SessionManager:
    sessions: Dict[str, AuthSession]  # session_id → auth_session
    lock: threading.RLock             # Thread-safe access
    cleanup_task: Thread              # Background cleanup thread
    
    def create_session(username, db_config) → str
    def get_session(session_id) → AuthSession
    def delete_session(session_id)
    def cleanup_expired_sessions()  # Background task
```

**Thread Safety:**

```python
class SessionManager:
    def __init__(self):
        self.sessions = {}
        self.lock = threading.RLock()  # Reentrant lock
    
    def get_session(self, session_id):
        with self.lock:  # Thread-safe access
            session = self.sessions.get(session_id)
            if session:
                session.touch()  # Update activity
            return session
```

**Background Cleanup:**

```python
def cleanup_expired_sessions(self):
    """Background task runs every 5 minutes"""
    while self.running:
        time.sleep(300)  # 5 minutes
        
        with self.lock:
            expired = [
                sid for sid, session in self.sessions.items()
                if session.is_expired()
            ]
            
            for sid in expired:
                del self.sessions[sid]
                print(f"🗑️ Cleaned up expired session: {sid[:8]}...")
```

---

### 4. Provider Factory

**Location:** `src/core/provider_factory.py`

**Purpose:** Creates database and LLM provider instances based on configuration.

**Pattern:**

```python
class ProviderFactory:
    @staticmethod
    def create_db_provider(config):
        provider_type = config['database']['provider']
        
        if provider_type == 'trino':
            return TrinoProvider(config['database']['trino'])
        elif provider_type == 'postgresql':
            return PostgreSQLProvider(config['database']['postgresql'])
        elif provider_type == 'mysql':
            return MySQLProvider(config['database']['mysql'])
        elif provider_type == 'bigquery':
            return BigQueryProvider(config['database']['bigquery'])
        else:
            raise ValueError(f"Unknown provider: {provider_type}")
    
    @staticmethod
    def create_llm_provider(config):
        provider_type = config['llm']['provider']
        
        if provider_type == 'anthropic':
            return AnthropicProvider(config['llm']['anthropic'])
        elif provider_type == 'gemini':
            return GeminiProvider(config['llm']['gemini'])
        elif provider_type == 'vertex':
            return VertexAIProvider(config['llm']['vertex'])
        else:
            raise ValueError(f"Unknown LLM provider: {provider_type}")
```

**Benefits:**
- ✅ Single point of provider instantiation
- ✅ Easy to add new providers
- ✅ Configuration-driven selection

---

### 5. Database Providers

**Base Interface:**

```python
class DatabaseProvider(ABC):
    @abstractmethod
    def execute_query(self, sql: str) → Tuple[List[str], List[List]]:
        """Execute SQL and return (columns, rows)"""
        pass
    
    @abstractmethod
    def get_tables(self) → List[str]:
        """Get list of available tables"""
        pass
    
    @abstractmethod
    def get_table_schema(self, table_name: str) → str:
        """Get schema for a specific table"""
        pass
```

**Trino Provider:**

```python
class TrinoProvider(DatabaseProvider):
    def __init__(self, config):
        self.conn = trino.dbapi.connect(
            host=config['host'],
            port=config['port'],
            user=config['user'],
            catalog=config['catalog'],
            schema=config['schema'],
            http_scheme=config.get('http_scheme', 'https')
        )
    
    def execute_query(self, sql):
        cursor = self.conn.cursor()
        cursor.execute(sql)
        columns = [desc[0] for desc in cursor.description]
        data = cursor.fetchall()
        return columns, data
```

**PostgreSQL Provider:**

```python
class PostgreSQLProvider(DatabaseProvider):
    def __init__(self, config):
        self.pool = psycopg2.pool.SimpleConnectionPool(
            minconn=1,
            maxconn=config.get('pool_size', 5),
            host=config['host'],
            port=config['port'],
            database=config['database'],
            user=config['user'],
            password=config['password']
        )
    
    def execute_query(self, sql):
        conn = self.pool.getconn()
        try:
            cursor = conn.cursor()
            cursor.execute(sql)
            columns = [desc[0] for desc in cursor.description]
            data = cursor.fetchall()
            return columns, data
        finally:
            self.pool.putconn(conn)
```

---

### 6. LLM Providers

**Base Interface:**

```python
class LLMProvider(ABC):
    @abstractmethod
    def determine_intent(self, question: str, schema: str) → dict:
        """Determine if question requires SQL or is chat"""
        pass
    
    @abstractmethod
    def generate_sql(self, question: str, schema: str) → Tuple[str, str]:
        """Generate SQL query and explanation"""
        pass
    
    @abstractmethod
    def generate_chat_response(self, question: str, schema: str = "") → str:
        """Generate chat response for non-SQL questions"""
        pass
    
    @abstractmethod
    def generate_insights(self, data, question, sql) → List[dict]:
        """Analyze query results and generate insights"""
        pass
```

**Anthropic Provider:**

```python
class AnthropicProvider(LLMProvider):
    def __init__(self, config):
        self.client = anthropic.Anthropic(
            api_key=config['api_key']
        )
        self.model = config.get('model', 'claude-sonnet-4-20250514')
        self.temperature = config.get('temperature', 0.0)
        self.max_tokens = config.get('max_tokens', 4096)
    
    def generate_sql(self, question, schema):
        # Get custom prompt from config
        prompt_template = config_manager.get_prompt('sql_generation')
        
        # Inject variables
        prompt = prompt_template.format(
            schema_info=schema,
            question=question
        )
        
        # Call Claude API
        message = self.client.messages.create(
            model=self.model,
            max_tokens=self.max_tokens,
            temperature=self.temperature,
            messages=[{"role": "user", "content": prompt}]
        )
        
        # Parse response
        response = message.content[0].text
        sql = self._extract_sql(response)
        explanation = self._extract_explanation(response)
        
        return sql, explanation
```

---

### 7. Configuration Manager

**Location:** `src/core/config_manager_enhanced.py`

**Purpose:** Centralized configuration management with hot reload support.

```python
class ConfigManager:
    def __init__(self):
        self.config_path = "config/config.yaml"
        self.base_config = {}           # From YAML
        self.runtime_overrides = {}     # From admin panel
        self.db_enabled = False         # PostgreSQL persistence
        self.config_db = None           # Optional ConfigDB
    
    def load_config(self):
        """Load base config from YAML"""
        with open(self.config_path) as f:
            self.base_config = yaml.safe_load(f)
    
    def get_config(self):
        """Merge base + overrides"""
        merged = deep_merge(self.base_config, self.runtime_overrides)
        return merged
    
    def set_runtime_override(self, key, value):
        """Update config without restart"""
        self.runtime_overrides[key] = value
        if self.db_enabled:
            self.config_db.save_override(key, value)
    
    def reload_providers(self):
        """Hot reload LLM/DB providers"""
        config = self.get_config()
        global llm_provider, db_provider
        llm_provider = ProviderFactory.create_llm_provider(config)
        db_provider = ProviderFactory.create_db_provider(config)
```

**Configuration Hierarchy:**

```
1. config.yaml (Base configuration)
    ↓
2. Environment Variables (Override specific values)
    SQLATTE_DB_HOST=production.db.com
    SQLATTE_LLM_API_KEY=sk-ant-xxxxx
    ↓
3. Runtime Overrides (Admin panel changes)
    config_manager.set_runtime_override('llm.temperature', 0.5)
    ↓
4. Database Persistence (Optional)
    ConfigDB.save_override('prompts.sql_generation', new_prompt)
    ↓
5. Final Merged Configuration
```

---

### 8. Scheduler Manager

**Location:** `src/core/scheduler_manager.py`

**Purpose:** Background job scheduling for automated query execution.

```python
class ScheduleManager:
    def __init__(self):
        self.scheduler = BackgroundScheduler()  # APScheduler
        self.schedules_db = ScheduledQueriesDB()
        self.email_service = EmailService()
    
    def create_schedule(self, schedule_config):
        """Create a new scheduled query"""
        # Save to database
        schedule_id = self.schedules_db.create_schedule(schedule_config)
        
        # Register with APScheduler
        trigger = CronTrigger.from_crontab(schedule_config['cron'])
        self.scheduler.add_job(
            func=self.execute_schedule,
            trigger=trigger,
            args=[schedule_id],
            id=schedule_id
        )
        
        return schedule_id
    
    async def execute_schedule(self, schedule_id):
        """Execute a scheduled query"""
        schedule = self.schedules_db.get_schedule(schedule_id)
        
        # Execute query
        db_provider = ProviderFactory.create_db_provider(config)
        columns, data = db_provider.execute_query(schedule['sql'])
        
        # Format result
        formatted = format_result(
            columns, data, 
            format_type=schedule['format']  # csv, excel, html
        )
        
        # Send email
        await self.email_service.send_email(
            recipients=schedule['email_recipients'],
            subject=schedule['email_subject'],
            body=self.email_service.create_report_email(...),
            attachments=[formatted]
        )
        
        # Log execution
        self.schedules_db.log_execution(schedule_id, success=True)
```

**Cron Examples:**

```python
# Daily at 9 AM
cron = "0 9 * * *"

# Every Monday at 8 AM
cron = "0 8 * * 1"

# First day of month at 6 AM
cron = "0 6 1 * *"
```

---

### 9. Plugin System

**Base Plugin Class:**

```python
class BasePlugin(ABC):
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.enabled = config.get('enabled', False)
    
    @abstractmethod
    def initialize(self, app: FastAPI):
        """Called on application startup"""
        pass
    
    @abstractmethod
    def register_routes(self, app: FastAPI):
        """Register plugin-specific endpoints"""
        pass
    
    async def before_request(self, request):
        """Middleware: before request processing"""
        return None
    
    async def after_request(self, request, response):
        """Middleware: after request processing"""
        return response
    
    def shutdown(self):
        """Cleanup on application shutdown"""
        pass
```

**Plugin Manager:**

```python
class PluginManager:
    def __init__(self):
        self.plugins: List[BasePlugin] = []
    
    def register_plugin(self, plugin: BasePlugin):
        if plugin.enabled:
            self.plugins.append(plugin)
    
    def initialize_all(self, app: FastAPI):
        for plugin in self.plugins:
            plugin.initialize(app)
            plugin.register_routes(app)
```

---

## Data Flow Diagrams

### Standard Widget Query Flow

```
1. User enters question in widget
    ↓
2. Widget → POST /api/query
    {
        question: "Show top customers",
        schema: "Table: customers...",
        session_id: "abc123"
    }
    ↓
3. Backend: Get/create conversation session
    conversation_manager.get_or_create_session("abc123")
    ↓
4. Backend: Add user message to session
    conversation_manager.add_message(
        session_id="abc123",
        role="user",
        content="Show top customers"
    )
    ↓
5. Backend: Get conversation context
    context = conversation_manager.get_conversation_context("abc123")
    # Returns last 10 messages for LLM context
    ↓
6. Backend: Determine intent
    intent = llm_provider.determine_intent(
        question="Show top customers",
        schema="Table: customers..."
    )
    # Returns: {intent: "sql", confidence: 0.95}
    ↓
7. Backend: Generate SQL (if intent = sql)
    sql, explanation = llm_provider.generate_sql(
        question="Show top customers",
        schema="Table: customers..."
    )
    # Returns: "SELECT * FROM customers ORDER BY revenue DESC LIMIT 10"
    ↓
8. Backend: Execute SQL
    columns, data = db_provider.execute_query(sql)
    # Returns: ([columns], [[row1], [row2], ...])
    ↓
9. Backend: Generate insights (if enabled)
    insights = insights_engine.generate_insights(
        question="Show top customers",
        sql=sql,
        data=data
    )
    ↓
10. Backend: Add assistant response to session
    conversation_manager.add_message(
        session_id="abc123",
        role="assistant",
        content={sql: ..., data: ..., insights: ...}
    )
    ↓
11. Backend: Log to analytics (if enabled)
    query_history.add_query(
        session_id="abc123",
        question="Show top customers",
        sql=sql,
        execution_time_ms=150,
        success=True
    )
    ↓
12. Backend → Widget: Response
    {
        sql: "SELECT...",
        columns: ["id", "name", "revenue"],
        data: [[1, "Acme", 50000], ...],
        insights: [{type: "trend", message: "..."}],
        session_id: "abc123"
    }
    ↓
13. Widget: Display results
    - Show SQL with syntax highlighting
    - Render data table
    - Display insights
    - Enable chart/export buttons
```

### Auth Widget Query Flow

```
1. User logs in with credentials
    ↓
2. Widget → POST /auth/login
    {
        username: "user@company.com",
        password: "***",
        database_type: "trino",
        host: "trino.company.com",
        catalog: "hive"
    }
    ↓
3. Auth Plugin: Test database connection
    db_provider = ProviderFactory.create_db_provider(user_credentials)
    db_provider.execute_query("SELECT 1")  # Test query
    ↓
4. Auth Plugin: Create session
    session_id = session_manager.create_session(
        username="user@company.com",
        db_config={host: ..., user: ..., password: ...}
    )
    ↓
5. Auth Plugin → Widget: Session ID
    {
        session_id: "xyz789",
        username: "user@company.com"
    }
    ↓
6. Widget: Store session ID (localStorage)
    localStorage.setItem('sqlatte_session_id', 'xyz789')
    ↓
7. User enters question
    ↓
8. Widget → POST /auth/query
    Headers: {X-Session-ID: "xyz789"}
    Body: {
        question: "Show top orders",
        schema: "Table: orders..."
    }
    ↓
9. Auth Plugin: Validate session
    session = session_manager.get_session("xyz789")
    if not session or session.is_expired():
        return 401 Unauthorized
    ↓
10. Auth Plugin: Get/create conversation session
    if not session.conversation_id:
        conv_id = conversation_manager.create_session()
        session.conversation_id = conv_id
    ↓
11. Auth Plugin: Execute query with user's DB config
    # Uses ThreadPoolExecutor for isolation
    result = await executor.run(
        _execute_query_for_session,
        db_config=session.db_config,
        question="Show top orders",
        schema="Table: orders..."
    )
    ↓
12. Auth Plugin → Widget: Results
    {
        sql: "SELECT...",
        data: [...],
        insights: [...]
    }
    ↓
13. Widget: Display results
```

---

## Performance Considerations

### Connection Pooling

```python
# PostgreSQL example
class PostgreSQLProvider:
    def __init__(self, config):
        self.pool = psycopg2.pool.SimpleConnectionPool(
            minconn=config.get('min_connections', 1),
            maxconn=config.get('pool_size', 5),
            ...
        )
    
    def execute_query(self, sql):
        conn = self.pool.getconn()  # Get from pool
        try:
            cursor = conn.cursor()
            cursor.execute(sql)
            return cursor.fetchall()
        finally:
            self.pool.putconn(conn)  # Return to pool
```

### Async Execution

```python
# Auth Plugin uses ThreadPoolExecutor
class AuthPlugin:
    def __init__(self, config):
        self.executor = ThreadPoolExecutor(
            max_workers=config.get('max_workers', 40)
        )
    
    async def execute_query_async(self, session, question, schema):
        loop = asyncio.get_event_loop()
        result = await loop.run_in_executor(
            self.executor,
            self._execute_query_sync,
            session.db_config,
            question,
            schema
        )
        return result
```

### Caching Strategy (Future)

```python
# Future: Redis-backed query cache
class QueryCache:
    def __init__(self):
        self.redis = redis.Redis(...)
        self.ttl = 3600  # 1 hour
    
    def get_cached_result(self, sql_hash):
        key = f"query:{sql_hash}"
        cached = self.redis.get(key)
        if cached:
            return json.loads(cached)
        return None
    
    def cache_result(self, sql_hash, result):
        key = f"query:{sql_hash}"
        self.redis.setex(key, self.ttl, json.dumps(result))
```

---

## Deployment Architecture

### Single Instance (Current)

```
┌────────────────────────────────────────┐
│         SQLatte Container              │
│  ┌──────────────────────────────────┐  │
│  │  FastAPI App                     │  │
│  │  - API Routes                    │  │
│  │  - Conversation Manager (memory) │  │
│  │  - Session Manager (memory)      │  │
│  │  - APScheduler (background)      │  │
│  └──────────────────────────────────┘  │
│               ↓                         │
│  ┌──────────────────────────────────┐  │
│  │  Database Connections            │  │
│  │  - User DB (Trino/PostgreSQL)    │  │
│  │  - Analytics DB (optional)       │  │
│  └──────────────────────────────────┘  │
└────────────────────────────────────────┘
```

### Multi-Instance (Future)

```
┌─────────────────────┐
│   Load Balancer     │
│   (Nginx/HAProxy)   │
└──────────┬──────────┘
           │
    ┌──────┴──────┬──────────┬──────────┐
    ↓             ↓          ↓          ↓
┌────────┐   ┌────────┐  ┌────────┐  ┌────────┐
│SQLatte │   │SQLatte │  │SQLatte │  │SQLatte │
│Instance│   │Instance│  │Instance│  │Instance│
│   1    │   │   2    │  │   3    │  │   4    │
└────┬───┘   └────┬───┘  └────┬───┘  └────┬───┘
     │            │            │            │
     └────────────┴────────────┴────────────┘
                  ↓
     ┌────────────────────────────┐
     │   Redis (Session Store)    │  # Future
     │   - Conversation sessions  │
     │   - Auth sessions          │
     │   - Query cache            │
     └────────────┬───────────────┘
                  ↓
     ┌────────────────────────────┐
     │   PostgreSQL               │
     │   - Analytics              │
     │   - Configuration          │
     │   - Schedule metadata      │
     └────────────────────────────┘
```

**Current Limitation:** Conversation memory is in-memory (not shared across instances)

**Roadmap:**
- Redis-backed session store
- Distributed conversation manager
- Sticky sessions at load balancer

---

## Security Architecture

### Authentication Flow (Auth Widget)

```
1. User submits credentials
    ↓
2. Auth Plugin validates + tests DB connection
    ↓
3. Create session with encrypted credentials
    session = AuthSession(
        session_id=uuid4(),
        username=username,
        db_config={
            'user': username,
            'password': encrypt(password),  # Future enhancement
            'host': host,
            ...
        }
    )
    ↓
4. Return session ID to widget (stored in localStorage)
    ↓
5. All subsequent requests include: X-Session-ID header
    ↓
6. Session Manager validates session on each request
    - Check if session exists
    - Check if not expired
    - Update last_activity
    ↓
7. Execute query with session's db_config
```

### Data Isolation

```
User A:
    Session A → DB Config A → Connection Pool A → User A's Database
    
User B:
    Session B → DB Config B → Connection Pool B → User B's Database
    
ThreadPoolExecutor ensures:
    - Each query executes in isolated thread
    - No data leakage between users
    - Concurrent execution without interference
```

---

## Monitoring & Observability

### Logging

```python
# Structured logging
import logging

logger = logging.getLogger(__name__)

# Request logging
logger.info(f"📝 Query: {question[:50]}...")
logger.info(f"✅ Result: {len(data)} rows in {exec_time}ms")
logger.error(f"❌ Error: {error}")

# Performance logging
logger.debug(f"⏱️ LLM call: {llm_time}ms")
logger.debug(f"⏱️ DB query: {db_time}ms")
```

### Metrics (Future)

```python
# Prometheus metrics
from prometheus_client import Counter, Histogram

query_counter = Counter('sqlatte_queries_total', 'Total queries')
query_duration = Histogram('sqlatte_query_duration_seconds', 'Query duration')

@query_duration.time()
def execute_query(sql):
    query_counter.inc()
    # Execute...
```

---

## Summary

SQLatte's architecture is designed for:

1. **Modularity** - Each component has a single responsibility
2. **Extensibility** - Plugin system for custom features
3. **Flexibility** - Provider abstraction for databases and LLMs
4. **Scalability** - Thread-safe design, connection pooling
5. **Simplicity** - Optional features, graceful degradation

**Core Components:**
- FastAPI (API layer)
- Conversation Manager (context memory)
- Session Manager (multi-tenancy)
- Provider Factory (abstraction)
- Scheduler Manager (automation)
- Plugin System (extensibility)

**Next Steps:**
- Review [Design Principles](principles.md) for architectural decisions
- Check [Configuration Reference](../configuration/config-yaml.md) for setup details

---

**Questions?** Open an issue on [GitHub](https://github.com/osmanuygar/sqlatte/issues).
