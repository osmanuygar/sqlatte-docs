# Barista Personality

SQLatte's conversational tone for anything that isn't a data question — greetings, help requests, small talk. The `intent_detection` prompt decides whether a message needs SQL or a conversational reply; when it's the latter, the `barista_personality` prompt shapes how SQLatte responds.

The shipped default: helpful, friendly, occasionally uses coffee/brewing metaphors, and steers lost users toward how to actually use the tool rather than just chatting indefinitely.

## Why It's a Separate Prompt

Keeping personality separate from SQL-generation rules means you can retune tone (more formal for an enterprise internal tool, more casual for a public-facing widget) without touching how queries are written — the two prompts are edited and reset independently. See [Runtime Prompts](runtime-prompts.md) for how to change it.

## Customizing

`/admin` → **Prompts** → `barista_personality`. There's nothing coffee-specific hardcoded outside the prompt text itself — replace the metaphors and tone guidance with whatever fits your product, and it takes effect immediately, no restart.
