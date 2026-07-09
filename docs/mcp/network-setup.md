# Network Setup (SSE)

Network mode serves MCP over HTTP directly from the SQLatte FastAPI app — no local Python script, no subprocess. Any user with a URL and a token can connect. This is the right mode when several people share one SQLatte deployment.

## Enable It

In `config/config.yaml`:

```yaml
mcp:
  sse:
    enabled: true
```

This mounts `/mcp` on the running server:

| Route | Purpose |
|---|---|
| `GET /mcp/sse` | SSE connection — clients connect here |
| `POST /mcp/messages/` | MCP protocol message endpoint |

## Client Config

```json
{
  "mcpServers": {
    "sqlatte": {
      "url": "http://your-sqlatte-server:8000/mcp/sse",
      "headers": { "x-mcp-token": "<token>" }
    }
  }
}
```

The token can also be passed as a `?token=` query parameter if your client doesn't support custom headers. Generate a token the same way as for local setup — see [API Tokens & Admin Auth](../security/tokens.md).

## How Isolation Works

Each SSE connection validates its token against the config database and creates its own session — there is no shared mutable state between users. Per-connection identity (session ID, mask rules, SQL dialect for the connected database provider) is stored in `contextvars`, so concurrent connections from different users never see each other's session or masking rules. A missing or invalid token gets a `401` before any session is created.

```
GET /mcp/sse?token=<token>
  → validate token against config DB
  → create isolated session (username, db_config, api_token)
  → load that user's active mask rules
  → run MCP server loop scoped to this connection
```

## Notes

- DNS rebinding protection is intentionally disabled at the transport layer — SQLatte handles authentication itself via the token, so this doesn't weaken security, but it does mean the endpoint should still sit behind TLS in production (reverse proxy termination, same as the rest of the app).
- The SSE server reuses the exact same three tools (`ask_database`, `list_tables`, `get_schema`), the same masking logic, and the same audit logging as the local stdio server — the only difference is transport and how the session is established.
