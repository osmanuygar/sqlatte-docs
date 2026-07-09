# Field Masking

Sensitive columns can be masked before results ever reach an MCP client — the AI agent sees a masked value, never the raw one. Rules are managed entirely in the Admin Panel under **MCP Masking**; no code changes or restart required.

## Strategies

| Strategy | Example output | Notes |
|---|---|---|
| `hash` | `a3f8bc2d1e4f9a07` | SHA-256, truncated to 16 hex chars — same input always hashes the same, so it stays groupable/joinable in aggregate queries |
| `partial` | `j**@company.com` | Emails keep the first local-part character and the domain; other strings keep first/last character |
| `redact` | `[REDACTED]` | Full replacement, no information retained |

`NULL` and empty-string values are always passed through unmasked (masking an absence of data is meaningless).

## Rule Configuration

Each rule is a `field_pattern` + `strategy` + enabled toggle:

- **Pattern matching** uses shell-style wildcards (`fnmatch`), e.g. `*email*`, `*_phone`, `ssn`
- **Case-insensitive** — matched against the lowercased column name
- **Toggle on/off** individually without deleting the rule
- Rules are fetched fresh on every query (a local config-DB read, ~1ms) so changes apply to the very next call — no caching lag

## Alias-Aware Matching

A naive implementation would only check the *displayed* column name — which an LLM-generated query can trivially rename (`SELECT email AS contact_info`), letting sensitive data slip past a rule targeting `*email*`.

SQLatte parses the generated SQL with [`sqlglot`](https://github.com/tobymao/sqlglot) (dialect-aware — `bigquery`, `mysql`, `postgres`, or `trino` based on the connected provider) to resolve each output alias back to its real source column(s), including multi-column expressions like `CONCAT(first_name, last_name)`. Masking rules are then matched against the source columns, not whatever alias the query happened to give them. If parsing fails for any reason, matching falls back to the display name — the same behavior as before this existed, never a hard failure.

```
SELECT email AS contact_info FROM users
                ↓ sqlglot resolves alias → source column
        contact_info → [email]
                ↓ rule "*email*" → partial
        contact_info: "j**@company.com"
```

## Scope

Masking is applied identically in both [local (stdio)](local-setup.md) and [network (SSE)](network-setup.md) MCP modes, and only to MCP tool responses — it does not affect the chat UI or embedded widgets, which have their own access model.
