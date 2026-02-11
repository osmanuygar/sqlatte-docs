# Runtime Prompts

Customize SQLatte's AI behavior by editing prompts directly from the admin panel.

---

## Overview

SQLatte uses 4 customizable prompts to control AI behavior:

1. **Intent Detection** - Determine if question needs SQL or chat
2. **Barista Personality** - Chat response tone and style
3. **SQL Generation** - Natural language to SQL rules
4. **Insights Generation** - Data analysis patterns

**Edit anytime:** Changes apply immediately with hot reload (no restart needed).

---

## Quick Start

### Edit Prompts

1. Go to `http://localhost:8000/admin`
2. Click **Prompts** tab
3. Select prompt to edit
4. Modify text
5. Click **Save**
6. ✅ Changes applied instantly!

---

## The 4 Prompts

### 1. Intent Detection

**Purpose:** Decide if question requires SQL query or is general chat.

**Default behavior:**
- "Show me sales" → SQL
- "Hello!" → Chat
- "What tables do I have?" → SQL (metadata query)

**When to customize:**
- Add domain-specific keywords
- Handle special cases
- Adjust confidence thresholds

**Example customization:**
```
If question contains "forecast" or "predict" → intent: chat
(Because we don't have forecasting capability)
```

---

### 2. Barista Personality

**Purpose:** Define chat response tone and style.

**Default personality:**
- Friendly and helpful
- Uses coffee metaphors occasionally
- Concise but warm
- Professional yet approachable

**When to customize:**
- Match brand voice
- Industry-specific tone (e.g., medical, legal)
- Language/cultural adaptation
- Formality level

**Example customization:**
```yaml
# More formal
You are SQLatte - a professional data assistant.
Be concise and business-focused.
Avoid metaphors. Provide direct answers.

# More playful
You are SQLatte ☕ - the coolest data barista around!
Serve up insights with a side of fun.
Use coffee puns liberally. Keep it light!
```

---

### 3. SQL Generation

**Purpose:** Rules for converting natural language to SQL.

**Key sections:**
- Database-specific syntax
- JOIN strategies
- Performance optimizations
- Partition handling

**When to customize:**
- Add database-specific optimizations
- Enforce company SQL standards
- Handle special table structures
- Add partition column rules

**Example customization:**
```yaml
# Add partition optimization for your database
⚡ PERFORMANCE RULES:
- Table 'events' has partition column 'event_date'
- ALWAYS filter: WHERE event_date >= '2025-01-01'
- Table 'logs' has partition column 'dt' (format: YYYYMMDD)
- ALWAYS filter: WHERE dt >= '20250101'

# Add company standards
COMPANY STANDARDS:
- Use table aliases: SELECT e.* FROM events e
- Always add LIMIT (max 1000 rows)
- Prefer EXISTS over IN for subqueries
```

---

### 4. Insights Generation

**Purpose:** Rules for analyzing query results and generating insights.

**Insight types:**
- Trends (increasing/decreasing)
- Anomalies (outliers, unexpected values)
- Summaries (key takeaways)
- Recommendations (actions to take)

**When to customize:**
- Add domain-specific patterns
- Adjust insight priorities
- Change severity thresholds
- Focus on specific metrics

**Example customization:**
```yaml
# E-commerce focus
Look for patterns in:
- Conversion rate changes
- Cart abandonment spikes
- Revenue per customer trends
- Product performance shifts

# Security focus
Prioritize:
- Unusual login patterns
- Access anomalies
- Failed authentication spikes
- Suspicious IP addresses
```

---

## Template Variables

### Available Variables

All prompts support template variables:

| Variable | Description | Available In |
|----------|-------------|--------------|
| `{schema_info}` | Table schemas | Intent, SQL Generation |
| `{question}` | User's question | All prompts |
| `{user_question}` | Original question | Insights |
| `{sql_query}` | Generated SQL | Insights |
| `{data_preview}` | Result sample | Insights |

**Usage:**
```yaml
sql_generation: |
  Schema: {schema_info}
  Question: {question}
  
  Generate SQL based on the schema and question.
```

---

## Prompt Management

### Save & Restore

**Save prompt:**
- Edit in admin panel
- Click **Save**
- Stored in database (if analytics enabled)

**Reset to default:**
- Click **Reset to Default**
- Restores original prompt
- Previous custom prompt lost

**Export/Import:**
```bash
# Export current prompts
curl http://localhost:8000/admin/prompts/export > prompts.yaml

# Import prompts
curl -X POST http://localhost:8000/admin/prompts/import \
  -F "file=@prompts.yaml"
```

---

### Version Control

Keep prompts in version control:

```yaml
# config.yaml
prompts:
  intent_detection: |
    [Your custom prompt]
  
  sql_generation: |
    [Your custom prompt]
```

**Commit to git:**
```bash
git add config.yaml
git commit -m "Update SQL generation prompt for partition optimization"
```

---

## Best Practices

### Editing Prompts
- ✅ Test changes with sample queries
- ✅ Make small, incremental changes
- ✅ Document why you made changes
- ✅ Keep backup of original prompts
- ⚠️ Don't remove template variable placeholders
- ⚠️ Don't make prompts too long (LLM token limits)

### SQL Generation
- ✅ Add database-specific optimizations
- ✅ Include partition column rules
- ✅ Specify LIMIT clause requirements
- ✅ Add JOIN strategy guidance
- ⚠️ Don't over-constrain (reduces flexibility)

### Barista Personality
- ✅ Match your brand voice
- ✅ Consider your audience
- ✅ Be consistent with tone
- ⚠️ Don't be overly casual in formal contexts

### Insights Generation
- ✅ Focus on actionable insights
- ✅ Prioritize business-relevant patterns
- ✅ Set appropriate severity levels
- ⚠️ Don't generate too many insights (max 3)

---

## Examples

### E-Commerce Optimization

**SQL Generation prompt:**
```yaml
sql_generation: |
  E-COMMERCE RULES:
  - Table 'orders' has partition 'order_date' (DATE)
  - ALWAYS filter: WHERE order_date >= CURRENT_DATE - 30
  - Calculate conversion rate: orders / sessions
  - Include customer_id for attribution
  
  JOINS:
  - orders ⟷ customers ON customer_id
  - orders ⟷ products ON product_id
  - orders ⟷ sessions ON session_id
```

---

### Security Monitoring

**Insights Generation prompt:**
```yaml
insights_generation: |
  SECURITY FOCUS:
  
  High Priority Patterns:
  - Failed login > 10 attempts → CRITICAL anomaly
  - New IP for existing user → WARNING
  - Access outside business hours → INFO
  
  Generate insights as:
  TYPE: security_alert
  SEVERITY: critical/warning/info
  MESSAGE: [actionable description]
```

---

### Healthcare Analytics

**Barista Personality prompt:**
```yaml
barista_personality: |
  You are SQLatte - a HIPAA-compliant data assistant for healthcare.
  
  Guidelines:
  - Professional and empathetic tone
  - Never reference patient data in responses
  - Use medical terminology appropriately
  - Privacy is paramount
  
  When users ask non-data questions:
  - Respond helpfully but concisely
  - Redirect to data queries when appropriate
```

---

## Troubleshooting

### Changes Not Applied

```bash
# Check analytics is enabled (required for persistence)
analytics:
  enabled: true

# Verify save was successful (check logs)
docker-compose logs -f sqlatte | grep prompt

# Try hard refresh in browser
Ctrl+Shift+R
```

### Prompt Too Long

```bash
# Error: Token limit exceeded
# Solution: Shorten prompt or increase max_tokens

llm:
  anthropic:
    max_tokens: 8192  # Increase from 4096
```

### Variables Not Working

```bash
# Ensure correct syntax: {variable_name}
# Not: {{variable_name}} or $variable_name

# Check variable is available for that prompt type
# Example: {sql_query} only available in insights prompt
```

---

## Advanced Usage

### Multi-Tenant Prompts

Different prompts per customer (future feature):

```yaml
prompts:
  customer_a:
    sql_generation: |
      [Customer A specific rules]
  
  customer_b:
    sql_generation: |
      [Customer B specific rules]
```

---

### A/B Testing Prompts

Test different prompts to find best performance:

```python
# Route 50% to prompt_v1, 50% to prompt_v2
if random.random() < 0.5:
    use_prompt("sql_generation_v1")
else:
    use_prompt("sql_generation_v2")

# Track which performs better
```

---

### Dynamic Prompt Injection

Add context at runtime:

```python
# Inject current user's access level
prompt = base_prompt + f"\nUser access level: {user.access_level}"

# Inject current date context
prompt = base_prompt + f"\nCurrent date: {datetime.now()}"
```

---

## Migration Guide

### From Config File to Database

**Before:**
```yaml
# config.yaml
prompts:
  sql_generation: |
    [Custom prompt]
```

**After:**
1. Enable analytics
2. Go to Admin → Prompts
3. Paste prompt from config.yaml
4. Save
5. Remove from config.yaml

**Benefit:** Edit without code deployment

---

**Next:** [Barista Personality](barista.md) | [Full Config Reference](../configuration/config-yaml.md)