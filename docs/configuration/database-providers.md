# Database Providers

SQLatte supports four database providers, selected via `database.provider`. Only the block for the selected provider needs to be filled in.

## Trino

Distributed SQL engine over Hive, Iceberg, Delta Lake, and other catalogs.

```yaml
database:
  provider: "trino"
  trino:
    host: "trino.example.com"
    port: 443
    user: "username"
    password: "password"
    catalog: "hive"
    schema: "default"
    http_scheme: "https"
```

## PostgreSQL

```yaml
database:
  provider: "postgresql"
  postgresql:
    host: "localhost"
    port: 5432
    database: "analytics"
    user: "postgres"
    password: "password"
    schema: "public"
    min_connections: 1
    max_connections: 10
```

`min_connections`/`max_connections` size the connection pool.

## MySQL

```yaml
database:
  provider: "mysql"
  mysql:
    host: "localhost"
    port: 3306
    database: "analytics"
    user: "root"
    password: "password"
    autocommit: true
```

## BigQuery

```yaml
database:
  provider: "bigquery"
  bigquery:
    project_id: "my-gcp-project"
    dataset: "analytics"
    location: "US"
    credentials_path: "/path/to/service-account.json"
    # or inline for containers: credentials_json: '{"type": "service_account", ...}'
    timeout: 300
    max_results: 10000
```

Requires a service account with `BigQuery Data Viewer` and `BigQuery Job User` roles at minimum. This is a separate credential path from `ops_agent.config.projects[].credentials_path`, which powers the [BigQuery Ops Console](../features/ops-console.md) — the two can point at different service accounts with different permission scopes if you want query access and ops/cost-analysis access separated.

## Multi-Tenant Overrides

If `plugins.auth` is enabled, each authenticated user connects with their own credentials passed at login time rather than the server's static `database` block — see [Security Overview](../security/overview.md#multi-tenant-auth-plugin). The top-level `database` config still applies to the default (unauthenticated) chat UI and widget.

| Database | Status | Required fields |
|---|---|---|
| Trino | Stable | host, port, catalog, schema |
| PostgreSQL | Stable | host, port, database, schema |
| MySQL | Stable | host, port, database |
| BigQuery | Stable | project_id, credentials |
