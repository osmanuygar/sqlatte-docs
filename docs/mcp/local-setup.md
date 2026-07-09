# Local Setup (stdio)

The default MCP mode runs `sqlatte_mcp_server.py` as a local subprocess that talks to your SQLatte server over HTTP. This is what Claude Desktop and Claude Code use by default when you add an MCP server via a local command.

```bash
pip install mcp httpx
```

## Option A — API Token (Recommended)

Keeps real database credentials out of the MCP config entirely. A token is generated from the SQLatte UI and stores the connection details server-side; the MCP process only ever holds the token.

**1. Generate a token**

Log in to SQLatte → open the chat widget → **🔑 API Tokens** → Generate. The token is shown once — copy it immediately. Token TTL is configurable (24h default) via the admin panel's token policy. Each user generates their own token, and it carries that user's catalog/schema context. See [API Tokens & Admin Auth](../security/tokens.md) for the full lifecycle.

**2. Add to your MCP client config** (e.g. `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "sqlatte": {
      "command": "python3",
      "args": ["/path/to/sqlatte/sqlatte_mcp_server.py"],
      "env": {
        "SQLATTE_URL": "http://localhost:8000",
        "SQLATTE_TOKEN": "<token>"
      }
    }
  }
}
```

## Option B — Username / Password (Legacy)

Bypasses token generation but puts raw Trino credentials directly in the MCP config file — only use this if API tokens aren't an option.

```json
{
  "mcpServers": {
    "sqlatte": {
      "command": "python3",
      "args": ["/path/to/sqlatte/sqlatte_mcp_server.py"],
      "env": {
        "SQLATTE_URL": "http://localhost:8000",
        "TRINO_HOST": "your-trino-host",
        "TRINO_PORT": "443",
        "TRINO_USER": "your-username",
        "TRINO_PASSWORD": "your-password",
        "TRINO_CATALOG": "hive",
        "TRINO_SCHEMA": "default",
        "TRINO_HTTP_SCHEME": "https"
      }
    }
  }
}
```

## How It Works

`sqlatte_mcp_server.py` logs in once (via token validation or username/password) to obtain a session ID, then reuses it for every tool call, re-authenticating automatically on a `401`. Every `ask_database` call fetches the table's schema first if not supplied, then calls the same `/auth/query` endpoint the auth widget uses — with `bypass_intent: true` since an MCP tool call has already established intent. Mask rules are fetched once per session and applied to results before they're returned to the client.

Restart your MCP client after editing its config. Once connected, ask your agent something like *"list the tables in the connected catalog"* to confirm the tools are live.
