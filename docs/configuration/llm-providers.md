# LLM Providers

SQLatte supports three LLM providers, selected via `llm.provider`.

## Anthropic Claude (Recommended)

```yaml
llm:
  provider: "anthropic"
  anthropic:
    api_key: "sk-ant-your-key-here"
    model: "claude-sonnet-4-20250514"
    max_tokens: 4096
```

Default and most capable — any Claude model (Opus, Sonnet, Haiku) is supported by name.

## Google Gemini

```yaml
llm:
  provider: "gemini"
  gemini:
    api_key: "your-gemini-key"
    model: "gemini-pro"
    max_tokens: 1000
```

Free tier available — good for evaluation.

## Google Vertex AI

```yaml
llm:
  provider: "vertexai"
  vertexai:
    project_id: "my-gcp-project"
    location: "europe-west1"
    model: "gemini-2.5-pro"
    credentials_path: "/path/to/service-account.json"
    # or inline for containers: credentials_json: "..."
```

Enterprise GCP path — uses service-account auth rather than an API key, and is the only provider with built-in per-task `model_routing` in the sample config (though `model_routing` works the same way regardless of which provider block it's nested under or set at top level).

## Task-Based Model Routing

Route each task to a different model to balance cost and accuracy — cheap/fast models for classification, capable models for actual SQL generation:

```yaml
model_routing:
  enabled: true
  tasks:
    intent_detection:
      provider: "anthropic"
      model: "claude-haiku-3-5-20241022"
      max_tokens: 500
    sql_generation:
      provider: "anthropic"
      model: "claude-sonnet-4-20250514"
      max_tokens: 4096
    insights:
      provider: "anthropic"
      model: "claude-sonnet-4-20250514"
      max_tokens: 2000
    chat:
      provider: "anthropic"
      model: "claude-haiku-3-5-20241022"
      max_tokens: 1000
```

Or, nested under `llm.vertexai.model_routing` for a single-provider shorthand:

```yaml
llm:
  vertexai:
    model_routing:
      intent_detection: "gemini-2.5-flash"
      chat: "gemini-2.5-flash"
      sql: "gemini-2.5-pro"
      insights: "gemini-2.5-flash"
      ops_insights: "gemini-2.5-flash"
```

Tasks not explicitly routed fall back to the top-level `llm.provider`/model.

| Provider | Models | Notes |
|---|---|---|
| Anthropic Claude | Opus, Sonnet, Haiku | Default; all models supported |
| Google Gemini | gemini-pro and newer | Free tier available |
| Google Vertex AI | gemini-pro and newer | Enterprise GCP, service-account auth |
