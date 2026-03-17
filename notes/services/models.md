# LLM Models — OpenClaw Provider Reference

> Docs: https://docs.openclaw.ai/providers/models

Model IDs follow the format `provider/model-id` in `openclaw.json`.

---

## Current Setup

Primary model configured in `/root/.openclaw/openclaw.json`:

```json
"agents": {
  "defaults": {
    "model": {
      "primary": "openai/gpt-5.4"
    }
  }
}
```

To switch models, update `primary` in `openclaw.json` — **hot reloads automatically**, no restart needed.

---

## Anthropic (Claude)

**Env var:** `ANTHROPIC_API_KEY` in `openclaw.json` → `env` section

| Model ID | Notes |
|---|---|
| `anthropic/claude-sonnet-4-6` | Latest, recommended — best balance |
| `anthropic/claude-opus-4-6` | Most capable, higher cost |
| `anthropic/claude-haiku-4-5` | Fastest, cheapest |

**Special features:**
- Extended thinking: `/think` command in Discord
- 1M context window: add `params.context1m: true` to model config
- Prompt caching: reduces cost on repeated context (5-min and 1-hour durations)
- Adaptive thinking enabled by default on Claude 4.6

**Example config:**

```json
"agents": {
  "defaults": {
    "model": {
      "primary": "anthropic/claude-sonnet-4-6",
      "fallbacks": ["openai/gpt-5-mini"]
    }
  }
}
```

---

## OpenAI

**Env var:** `OPENAI_API_KEY` in `openclaw.json` → `env` section

| Model ID | Notes |
|---|---|
| `openai/gpt-5-mini` | Current default — fast, cheap |
| `openai/gpt-5.4` | More capable |
| `openai/gpt-5.4-pro` | Most capable OpenAI model |

**Special features:**
- WebSocket transport with warm-up (lower latency)
- Server-side compaction at 70% context threshold
- Priority processing via `service_tier`

---

## Other Supported Providers

OpenClaw supports 15+ providers total. Notable ones:

| Provider | Env Var | Example Model ID |
|---|---|---|
| Mistral | `MISTRAL_API_KEY` | `mistral/mistral-large-latest` |
| Google Gemini | `GEMINI_API_KEY` | `google/gemini-2.0-flash` |
| Groq | `GROQ_API_KEY` | `groq/llama-3.3-70b` |
| Ollama (local) | none | `ollama/llama3.2` |

---

## Switching Models at Runtime

Change in `openclaw.json` — hot reloads automatically (no restart needed for model changes):

```bash
# Edit on server
ssh openclaw
python3 -c "
import json
with open('/root/.openclaw/openclaw.json') as f:
    c = json.load(f)
c['agents']['defaults']['model']['primary'] = 'openai/gpt-5.4'
with open('/root/.openclaw/openclaw.json', 'w') as f:
    json.dump(c, f, indent=2)
"
# openclaw hot-reloads — no restart needed
```

Or ask the bot directly in Discord:
```
切换到 claude-sonnet-4-6 模型
```

---

## Multi-model (Fallback) Config

```json
"agents": {
  "defaults": {
    "model": {
      "primary": "anthropic/claude-sonnet-4-6",
      "fallbacks": ["openai/gpt-5-mini"]
    }
  }
}
```

If the primary model fails or rate-limits, automatically falls back to the next.
