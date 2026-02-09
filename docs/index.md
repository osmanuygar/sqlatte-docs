# SQLatte Documentation

Welcome to **SQLatte** - your friendly barista for data analytics! ☕

SQLatte transforms natural language questions into SQL queries, making database analytics accessible to everyone while maintaining enterprise-grade capabilities.

## What is SQLatte?

SQLatte is a natural language to SQL analytics platform that combines the power of Large Language Models with your databases. Think of it as having a knowledgeable data analyst who speaks your language and understands your data.

### Key Features

- **Natural Language Queries**: Ask questions in plain English, get SQL results
- **Multi-Database Support**: Works with Trino, PostgreSQL, MySQL, and BigQuery
- **Multiple LLM Providers**: Choose from Anthropic Claude, Google Gemini, or Vertex AI
- **Flexible Deployment**: Widget-embeddable or standalone dashboard
- **Optional Complexity**: Start simple, scale to enterprise features when needed
- **Scheduled Queries**: Automate reports with email delivery
- **Analytics Dashboard**: Track usage, insights, and query patterns
- **Runtime Prompts**: Customize AI behavior through admin panel

## Quick Example

```javascript
// Embed SQLatte widget in your app
<div id="sqlatte-widget"></div>
<script src="sqlatte-widget.js"></script>
<script>
  SQLatte.init({
    container: 'sqlatte-widget',
    apiUrl: 'http://your-sqlatte-instance:5001'
  });
</script>
```

Then ask questions like:
- "Show me top 10 customers by revenue this month"
- "What's the trend of daily active users over the last quarter?"
- "Compare sales between regions for Q4"

## Architecture Highlights

### The Barista Personality

SQLatte acts like a friendly barista who:
- Understands your requests naturally
- Remembers your preferences
- Suggests relevant insights
- Makes complex queries feel simple

### Optional Complexity Philosophy

SQLatte grows with your needs:

| Feature | Basic Mode | Enterprise Mode |
|---------|-----------|-----------------|
| Querying | ✓ Config-based | ✓ Session-based auth |
| Storage | ✓ In-memory | ✓ PostgreSQL persistence |
| Automation | - | ✓ Scheduled queries |
| Analytics | - | ✓ Full dashboard |
| Customization | - | ✓ Runtime prompts |

Features activate through configuration - no breaking changes, no forced complexity.

## Getting Started

<div class="grid cards" markdown>

-   :material-rocket-launch:{ .lg .middle } __Quick Start__

    ---

    Get SQLatte running in 5 minutes

    [:octicons-arrow-right-24: Installation Guide](getting-started/installation.md)

-   :material-cog:{ .lg .middle } __Configuration__

    ---

    Configure databases, LLMs, and features

    [:octicons-arrow-right-24: Configuration Guide](configuration/overview.md)

-   :material-widgets:{ .lg .middle } __Embed Widgets__

    ---

    Add SQLatte to your applications

    [:octicons-arrow-right-24: Widget Guide](widgets/overview.md)

-   :material-api:{ .lg .middle } __API Reference__

    ---

    REST endpoints and WebSocket API

    [:octicons-arrow-right-24: API Docs](api/endpoints.md)

</div>

## Supported Technologies

### Database Providers
- **Trino** - Distributed SQL query engine
- **PostgreSQL** - Advanced open-source database
- **MySQL** - Popular relational database
- **BigQuery** - Google Cloud data warehouse

### LLM Providers
- **Anthropic Claude** - Advanced reasoning and SQL generation
- **Google Gemini** - Fast and efficient
- **Vertex AI** - Enterprise Google Cloud AI

## Use Cases

**Business Analytics**
: Enable non-technical teams to query data without learning SQL

**Data Exploration**
: Quickly explore datasets and find insights interactively

**Automated Reporting**
: Schedule queries and email results to stakeholders

**Embedded Analytics**
: Add natural language querying to your SaaS applications

**Enterprise BI**
: Complement existing BI tools with conversational interface

## Community & Support

- **GitHub**: [github.com/osmanuygar/sqlatte](https://github.com/osmanuygar/sqlatte)
- **Issues**: Report bugs or request features
- **Discussions**: Ask questions and share use cases

## License

SQLatte is open source. Check the [main repository](https://github.com/osmanuygar/sqlatte) for license details.

---

Ready to brew some queries? Start with the [Installation Guide](getting-started/installation.md)! ☕
