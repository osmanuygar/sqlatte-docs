# MCP Server — AI Agent Gateway

SQLatte is a native [MCP](https://modelcontextprotocol.io/) (Model Context Protocol) server. Any MCP-compatible client — Claude Desktop, Claude Code, or a custom agent — gets controlled, audited access to your data warehouse without ever holding raw database credentials.

## Why It Exists

Giving an AI agent a database password is an all-or-nothing decision: full read access, no way to scope it to a schema, no revocation short of rotating the password, no record of what the agent actually queried. SQLatte's MCP server replaces the password with a short-lived, revocable token, and routes every generated query through the same validation and audit pipeline used by the chat UI.

- **Short-lived tokens** scoped to a specific catalog and schema
- **Every generated query validated** (`is_select_only` + risk scoring) before execution — see [Security Overview](../security/overview.md)
- **Every call logged** with token counts and source (`ui` / `widget` / `mcp`) — see [Audit Logs](../features/audit-logs.md)
- **Field-level masking** of sensitive columns before results reach the agent — see [Field Masking](field-masking.md)
- **One-click revocation** from the admin panel — no credential rotation needed

## Two Ways to Connect

| Mode | Transport | Requires local Python? | Use case |
|---|---|---|---|
| [**Local Setup**](local-setup.md) | stdio | Yes — runs `sqlatte_mcp_server.py` as a subprocess | Claude Desktop / Claude Code on your own machine |
| [**Network Setup**](network-setup.md) | SSE over HTTP | No — clients connect via URL | Multiple users sharing one SQLatte deployment, no local install |

Both modes expose the same three tools and go through the same validation, masking, and audit pipeline — they differ only in how the client reaches the server.

## Available Tools

| Tool | Description |
|---|---|
| `ask_database` | Natural language question → generated SQL → executed → results (skips intent classification since the caller already knows it wants data) |
| `list_tables` | List tables in the connected catalog/schema |
| `get_schema` | Column definitions for a specific table |

```bash
pip install mcp httpx
```

Next: pick [Local (stdio)](local-setup.md) or [Network (SSE)](network-setup.md) setup.
