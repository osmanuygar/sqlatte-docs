# WebSocket API

Real-time communication with SQLatte for streaming query results and live updates.

---

## Overview

WebSocket API enables:

- Real-time query execution
- Streaming large result sets
- Live schedule notifications
- Progress updates for long queries

**Status:** 🚧 **Coming Soon** - Currently in development

---

## Connection

### Endpoint

```
ws://localhost:8000/ws
```

### JavaScript Example

```javascript
const socket = new WebSocket('ws://localhost:8000/ws');

socket.onopen = () => {
  console.log('Connected to SQLatte');
};

socket.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Received:', data);
};

socket.onerror = (error) => {
  console.error('WebSocket error:', error);
};

socket.onclose = () => {
  console.log('Disconnected');
};
```

---

## Message Format

### Client → Server

```json
{
  "type": "query",
  "data": {
    "question": "Show me all orders",
    "schema": "Table: orders...",
    "session_id": "abc123"
  }
}
```

---

### Server → Client

```json
{
  "type": "query_result",
  "data": {
    "sql": "SELECT * FROM orders",
    "columns": ["id", "amount"],
    "rows": [[1, 100], [2, 200]],
    "total_rows": 2,
    "execution_time_ms": 150
  }
}
```

---

## Message Types

### Query Execution

**Client sends:**
```json
{
  "type": "query",
  "data": {
    "question": "Show sales today",
    "schema": "Table: orders..."
  }
}
```

**Server responds:**
```json
{
  "type": "query_start",
  "data": {
    "query_id": "q_123"
  }
}
```

```json
{
  "type": "query_progress",
  "data": {
    "query_id": "q_123",
    "progress": 50,
    "message": "Processing 500 rows..."
  }
}
```

```json
{
  "type": "query_result",
  "data": {
    "query_id": "q_123",
    "sql": "SELECT...",
    "columns": [...],
    "rows": [...]
  }
}
```

---

### Streaming Results

For large datasets (>1000 rows):

**Server sends chunks:**
```json
{
  "type": "result_chunk",
  "data": {
    "query_id": "q_123",
    "chunk": 1,
    "rows": [[1, 100], [2, 200], ...]
  }
}
```

```json
{
  "type": "result_complete",
  "data": {
    "query_id": "q_123",
    "total_rows": 5000,
    "total_chunks": 5
  }
}
```

---

### Live Notifications

**Schedule execution:**
```json
{
  "type": "schedule_executed",
  "data": {
    "schedule_id": "schedule_1",
    "schedule_name": "Daily Sales",
    "success": true,
    "rows_returned": 150
  }
}
```

**Error notification:**
```json
{
  "type": "error",
  "data": {
    "message": "Query execution failed",
    "error": "Table not found: orders"
  }
}
```

---

## Authentication

### With Session ID

```javascript
const socket = new WebSocket('ws://localhost:8000/ws');

socket.onopen = () => {
  // Send authentication
  socket.send(JSON.stringify({
    type: 'auth',
    data: {
      session_id: 'abc123...'
    }
  }));
};
```

---

## Examples

### Simple Query

```javascript
// Connect
const socket = new WebSocket('ws://localhost:8000/ws');

// Send query
socket.onopen = () => {
  socket.send(JSON.stringify({
    type: 'query',
    data: {
      question: 'Show top 10 customers',
      schema: 'Table: customers\nColumns: id, name, revenue'
    }
  }));
};

// Receive results
socket.onmessage = (event) => {
  const message = JSON.parse(event.data);
  
  if (message.type === 'query_result') {
    console.log('SQL:', message.data.sql);
    console.log('Rows:', message.data.rows);
  }
};
```

---

### Streaming Large Dataset

```javascript
let allRows = [];

socket.onmessage = (event) => {
  const message = JSON.parse(event.data);
  
  switch (message.type) {
    case 'query_start':
      console.log('Query started:', message.data.query_id);
      break;
      
    case 'query_progress':
      console.log('Progress:', message.data.progress + '%');
      break;
      
    case 'result_chunk':
      allRows.push(...message.data.rows);
      console.log(`Received chunk ${message.data.chunk}`);
      break;
      
    case 'result_complete':
      console.log('Complete! Total rows:', allRows.length);
      renderTable(allRows);
      break;
  }
};
```

---

### Live Dashboard

Subscribe to real-time updates:

```javascript
socket.onopen = () => {
  // Subscribe to schedule events
  socket.send(JSON.stringify({
    type: 'subscribe',
    data: {
      events: ['schedule_executed', 'query_completed']
    }
  }));
};

socket.onmessage = (event) => {
  const message = JSON.parse(event.data);
  
  if (message.type === 'schedule_executed') {
    updateDashboard(message.data);
  }
};
```

---

## Error Handling

### Connection Errors

```javascript
socket.onerror = (error) => {
  console.error('Connection error:', error);
};

socket.onclose = (event) => {
  if (!event.wasClean) {
    console.error('Connection lost:', event.code, event.reason);
    // Attempt reconnection
    setTimeout(reconnect, 5000);
  }
};
```

---

### Query Errors

```javascript
socket.onmessage = (event) => {
  const message = JSON.parse(event.data);
  
  if (message.type === 'error') {
    console.error('Query error:', message.data.message);
    showErrorToUser(message.data.message);
  }
};
```

---

## Reconnection

```javascript
class SQLatteWebSocket {
  constructor(url) {
    this.url = url;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
    this.connect();
  }
  
  connect() {
    this.socket = new WebSocket(this.url);
    
    this.socket.onopen = () => {
      console.log('Connected');
      this.reconnectAttempts = 0;
    };
    
    this.socket.onclose = () => {
      if (this.reconnectAttempts < this.maxReconnectAttempts) {
        this.reconnectAttempts++;
        setTimeout(() => this.connect(), 2000);
      }
    };
  }
}
```

---

## Performance Tips

### Batch Messages

```javascript
// Don't send messages one by one
messages.forEach(msg => socket.send(msg)); // ❌

// Batch them
socket.send(JSON.stringify({
  type: 'batch',
  data: {
    messages: messages
  }
})); // ✅
```

---

### Throttle Updates

```javascript
let buffer = [];
let timeout;

socket.onmessage = (event) => {
  buffer.push(JSON.parse(event.data));
  
  clearTimeout(timeout);
  timeout = setTimeout(() => {
    processMessages(buffer);
    buffer = [];
  }, 100);
};
```

---

## Limitations

- ⚠️ **Not yet implemented** - Coming in future version
- 🚧 **Planned features:**

  - Real-time query execution
  - Streaming results for large datasets
  - Live schedule notifications
  - Progress updates
  - Multi-user collaboration

---

## Roadmap

**Phase 1:** Basic WebSocket connection

- Query execution
- Result streaming

**Phase 2:** Live updates

- Schedule notifications
- Real-time dashboard

**Phase 3:** Collaboration

- Multi-user queries
- Shared sessions
- Live cursors

---

## Alternative: Server-Sent Events (SSE)

For one-way updates, consider SSE:

```javascript
const eventSource = new EventSource('http://localhost:8000/events');

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Update:', data);
};
```

**Use cases:**

- Live dashboard updates
- Schedule execution notifications
- System status monitoring

---

**Next:** [REST Endpoints](endpoints.md) 

---

**Note:** WebSocket API is currently in development. Use REST API for production deployments.