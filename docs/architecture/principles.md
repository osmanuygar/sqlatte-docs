# Design Principles

## Governed Access, No Exceptions

The core architectural constraint: there is no code path where an AI agent (via MCP) or a human (via chat/widget) can execute SQL without going through intent detection, validation, and audit logging. `bypass_intent` (used by MCP) only skips the "is this a data question" classification step — it never skips `is_select_only()` or the audit write. This is a deliberate constraint on where shortcuts are allowed, not an oversight.

## Optional Complexity

Start with two required config blocks (`llm`, `database`) and a working chat interface. Every other capability — analytics, scheduler, semantic layer, MCP, ops console, multi-tenant auth, config persistence — is off by default and additive. Enabling one doesn't require configuring the others, and most features degrade to an in-memory/SQLite fallback rather than hard-failing when their backing store isn't configured. See [Persistence Model](overview.md#persistence-model).

## Widget-First, Not App-First

The chat interface isn't a standalone product bolted onto a backend — it's one of several equally-weighted entry points (standard widget, auth widget, standalone dashboard, MCP) that all terminate in the same backend pipeline. This is why embedding SQLatte in an existing site is a `<script>` tag, not a separate deployment.

## Plugin Architecture

`src/plugins/base_plugin.py` defines a minimal extension contract: `initialize()`, `register_routes()`, `before_request()`/`after_request()` hooks, and `shutdown()`. The `PluginManager` registers only enabled plugins (config-gated) and calls their hooks around every request. The auth plugin (`auth_plugin.py`) — multi-tenant sessions, per-user DB credentials, catalog/schema restrictions — is the reference implementation; custom plugins (Slack notifications, alternate auth providers, custom middleware) follow the same contract without touching core request handling.

`BasePlugin._safe_config()` automatically masks any config key containing `password`, `secret`, `key`, or `token` before it's exposed via `get_info()` — plugin introspection endpoints can't accidentally leak credentials.

## Provider Abstraction

Database and LLM access go through a factory interface, not direct SDK calls scattered through business logic. Adding a fifth database provider means implementing the provider interface once, not touching every route that queries data.

## Server-Side Credential Custody

Raw database credentials and LLM API keys live in `config.yaml` (or `config_db`) and never leave the server. Every client-facing auth mechanism — session cookies, API tokens, MCP tokens — is a revocable, scoped indirection layer on top of those credentials, not a copy of them. See [Security Overview](../security/overview.md).

## Hot Reload Over Restart

Prompt edits, mask rules, token policy, and most config changes made through the admin panel apply on the next request, not the next deploy. This matters specifically because prompts and masking rules are the levers most likely to need adjusting in production based on real traffic — waiting on a restart cycle for a masking-rule fix is a real operational risk.
