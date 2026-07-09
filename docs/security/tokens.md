# API Tokens & Admin Auth

## API Token Lifecycle

API tokens are the recommended way to authenticate MCP clients (and any other programmatic caller) without embedding raw database credentials. A token is a persisted, revocable credential bound to a user's existing auth-widget session — it carries that session's `db_config` (catalog/schema scope) forward.

| Endpoint | Method | Purpose |
|---|---|---|
| `/auth/token/generate` | POST | Create a token from an active session. Params: `ttl_hours` (default 24), `description`, optional `daily_query_limit` |
| `/auth/token/validate` | POST | Validate a token, return a fresh session ID — this is what the MCP server calls on connect |
| `/auth/token/revoke` | POST | Revoke a token (owner-only) |
| `/auth/tokens` | GET | List the caller's own active tokens |

**Generating a token (UI flow):** log in → open the chat widget → **🔑 API Tokens** → Generate. The raw token value is shown exactly once at creation time and is not retrievable afterward — copy it immediately into your MCP client config.

**Daily query limits:** an optional per-token cap. If the admin has set a platform-wide default (`token-policy`), the effective limit is the *minimum* of the token's own limit and the platform default — a token can only be more restrictive than the platform ceiling, never less.

**Revocation:** deleting a token server-side invalidates every session created from it immediately — no waiting for TTL expiry, no credential rotation elsewhere.

## Admin-Side Token Management

From the admin panel (`/admin` → **Tokens**):

| Endpoint | Method | Purpose |
|---|---|---|
| `/admin/tokens` | GET | List all issued tokens across all users |
| `/admin/token/revoke` | POST | Force-revoke any token |
| `/admin/token-policy` | GET / POST | Get/set the platform-wide default daily query limit applied to all tokens |
| `/admin/token/set-limit` | POST | Override the limit on a specific token |
| `/admin/mcp-mask-rules` | GET / POST / PUT / DELETE | Manage field-masking rules — see [Field Masking](../mcp/field-masking.md) |

## Admin Panel Authentication

The admin panel (`/admin`) is separate from user/MCP auth entirely — a single configured username/password pair in `config.yaml`:

```yaml
admin:
  enabled: true
  username: "${ADMIN_USERNAME:admin}"
  password: "${ADMIN_PASSWORD}"
```

- **Sessions** are in-memory, token-based (via an `sqlatte_admin` cookie), with a configurable TTL (`session_ttl_minutes`, default 480).
- **Credential comparison** uses `secrets.compare_digest` (constant-time) to avoid timing attacks.
- **Brute-force protection**: 10 failed attempts within a 5-minute window locks the source IP out for 10 minutes, mirroring the same pattern used for `/auth/login`.
- **Fail-closed by design**: if `admin.password` is empty or `admin.enabled` is `false`, authentication is refused outright rather than falling back to an open panel — there is no "auth disabled" bypass path.
- HTML page routes redirect unauthenticated browsers to `/admin/login`; JSON API routes return a `401` instead, so `fetch()` callers never receive an HTML login page they'd try to parse as JSON.

## Auto-Session (Embedded Widgets)

For the standard embeddable widget (no user login), `plugins.auth.auto_session` lets the server mint a short-lived session backed by its own configured credentials, so an unauthenticated widget can still reach governed endpoints:

```yaml
plugins:
  auth:
    auto_session:
      enabled: true
      ttl_hours: 1
      label: "auto-session"
```

This session is shorter-lived than a real user session (1h vs. the 480-minute default) and is clearly labeled in audit logs so widget traffic is distinguishable from authenticated user traffic.
