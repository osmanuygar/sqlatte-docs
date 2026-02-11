# LLM Providers

SQLatte supports three LLM providers. Configure one in `config.yaml`:

```yaml
llm:
  provider: "anthropic"  # anthropic, gemini, vertex
```

---

## Anthropic Claude (Recommended)

**Best for:** SQL generation, complex reasoning, highest quality

```yaml
llm:
  provider: "anthropic"
  anthropic:
    api_key: "sk-ant-xxxxx"
    model: "claude-sonnet-4-20250514"    # Recommended
    max_tokens: 4096
    temperature: 0.0                      # Deterministic (best for SQL)
    timeout: 60
```

### Models

| Model | Best For | Speed | Cost |
|-------|----------|-------|------|
| `claude-sonnet-4-20250514` | Balanced (recommended) | Fast | $$ |
| `claude-opus-4-20250514` | Highest quality | Slow | $$$ |
| `claude-haiku-4-20250316` | Simple queries | Fastest | $ |

### Getting API Key

1. Visit [console.anthropic.com](https://console.anthropic.com)
2. Sign up / Log in
3. Go to **API Keys**
4. Click **Create Key**
5. Copy `sk-ant-xxxxx`
6. Add to config.yaml

**Pricing:** Pay-as-you-go, ~$3 per 1M input tokens (Sonnet)

---

## Google Gemini

**Best for:** Free tier, fast responses, Google ecosystem

```yaml
llm:
  provider: "gemini"
  gemini:
    api_key: "your-gemini-api-key"
    model: "gemini-2.0-flash-exp"         # Latest model
    max_tokens: 4096
    temperature: 0.0
```

### Getting API Key

1. Visit [makersuite.google.com/app/apikey](https://makersuite.google.com/app/apikey)
2. Sign in with Google account
3. Click **Create API Key**
4. Copy key
5. Add to config.yaml

**Free Tier:** 60 requests/minute, 1,500 requests/day

**Pricing:** Free tier available, then pay-as-you-go

---

## Google Vertex AI

**Best for:** Enterprise GCP deployments, production scale

```yaml
llm:
  provider: "vertex"
  vertex:
    project_id: "my-gcp-project"
    location: "us-central1"               # or europe-west1, asia-northeast1
    model: "gemini-2.0-flash-exp"
    credentials_path: "/path/to/service-account.json"
    max_tokens: 4096
    temperature: 0.0
```

### Service Account Setup

1. **Create Service Account:**
   ```bash
   # GCP Console → IAM & Admin → Service Accounts
   ```

2. **Grant Permissions:**
   - `Vertex AI User`
   - `Service Account Token Creator`

3. **Download JSON Key:**
   ```bash
   # Keys → Add Key → JSON
   # Save to /path/to/service-account.json
   ```

4. **Enable Vertex AI API:**
   ```bash
   gcloud services enable aiplatform.googleapis.com
   ```

**Pricing:** Enterprise pricing, similar to Gemini API

---

## Comparison

| Feature | Anthropic Claude | Google Gemini | Vertex AI |
|---------|-----------------|---------------|-----------|
| **Quality (SQL)** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Speed** | Fast | Very Fast | Fast |
| **Cost** | $$ | $ (free tier) | $$ |
| **Setup** | Easy | Easiest | Complex |
| **Free Tier** | No | Yes | No |
| **Best For** | Production | Testing/Free | Enterprise GCP |

---

## Model Parameters

### Temperature
```yaml
temperature: 0.0   # Deterministic (recommended for SQL)
temperature: 0.5   # Balanced
temperature: 1.0   # Creative (not recommended for SQL)
```

**Recommendation:** Always use `0.0` for SQL generation to ensure consistent queries.

### Max Tokens
```yaml
max_tokens: 4096   # Standard (recommended)
max_tokens: 8192   # For complex queries with large schemas
```

### Timeout
```yaml
timeout: 60        # Default (60 seconds)
timeout: 120       # For complex queries
```

---

## Testing LLM Connection

```bash
# Via Admin Panel
http://localhost:8000/admin
# → Providers tab → Test LLM button

# Via API
curl http://localhost:8000/health
# Check: "llm": "available"
```

---

## Switching Providers

Change LLM provider in real-time:

**Option 1: Config File**
```yaml
# Edit config.yaml
llm:
  provider: "gemini"  # Changed from anthropic
```

**Option 2: Admin Panel**
```
http://localhost:8000/admin
→ Providers tab
→ Select LLM provider
→ Save
→ Hot reload (no restart needed!)
```

---

## Cost Optimization

### Use Cheaper Models for Simple Queries
```yaml
# For simple queries, use Haiku (Anthropic) or Gemini free tier
```

### Adjust Token Limits
```yaml
max_tokens: 2048   # Reduce if queries are simple
```

### Cache Prompts (Future)
```python
# Cache common schema descriptions
# Reuse prompt templates
```

---

## Troubleshooting

### Invalid API Key
```bash
# Error: 401 Unauthorized
# - Verify API key is correct
# - Check key hasn't expired
# - Ensure no extra spaces in config.yaml
```

### Rate Limit Exceeded
```bash
# Error: 429 Too Many Requests
# - Reduce request frequency
# - Upgrade to paid tier
# - Use different model
```

### Model Not Found
```bash
# Error: 404 Model not found
# - Check model name spelling
# - Verify model is available in your region
# - Use latest model names
```

### Timeout
```bash
# Error: Request timeout
# - Increase timeout in config
# - Simplify query/schema
# - Use faster model (Haiku, Gemini Flash)
```

---

## Best Practices

### For Production
- ✅ Use **Claude Sonnet 4** for best SQL quality
- ✅ Set `temperature: 0.0` for deterministic results
- ✅ Monitor API costs with usage limits
- ✅ Use environment variables for API keys

### For Development
- ✅ Use **Gemini free tier** for testing
- ✅ Test with sample queries before production
- ✅ Validate generated SQL before execution

### For Enterprise
- ✅ Use **Vertex AI** for GCP integration
- ✅ Set up service account with minimal permissions
- ✅ Enable audit logging
- ✅ Use VPC for network isolation

---

## Environment Variables

Store API keys securely:

```bash
# .env file (don't commit!)
ANTHROPIC_API_KEY=sk-ant-xxxxx
GEMINI_API_KEY=your-gemini-key
```

```yaml
# config.yaml
llm:
  provider: "anthropic"
  anthropic:
    api_key: "${ANTHROPIC_API_KEY}"  # From environment
```

---

**Next:** [Analytics Setup](analytics.md) | [Full Config Reference](config-yaml.md)