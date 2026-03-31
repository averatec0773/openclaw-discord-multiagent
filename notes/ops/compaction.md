# Compaction — Context Management

OpenClaw compacts conversation history when sessions grow large, summarizing older turns so the model sees a manageable context on each request.

Reference: https://docs.openclaw.ai/concepts/compaction

---

## Context window and the 272k threshold

The context window is the maximum tokens a model can process in a single request. For GPT-5.4 this is **272k tokens**. All session content — system context, conversation history, and the new message — must fit within this limit.

OpenClaw shows the current usage after compaction:
```
⚙️ Compacted (190k → 22k) • Context 22k/272k (8%)
```

This means: history was compressed from 190k to 22k, now using 8% of the 272k window. Compaction triggers automatically as the session approaches the limit, or can be run manually with `/compact`.

---

## Why it matters

Each conversation turn includes the full session history as input tokens. Without compaction, context grows unboundedly — a session that starts at 12K tokens/turn can reach 190K+ tokens/turn after weeks of daily use, with cost per turn increasing ~85x.

With compaction active, older turns are summarized and the effective context stays small.

---

## How it works

1. Session approaches the model's context limit (or `/compact` is issued manually)
2. Agent is prompted to save important information to memory files first
3. Older turns are condensed into compact summary entries
4. Recent messages remain unchanged
5. Full history stays on disk — compaction only affects what the model sees next turn

Compaction is distinct from **pruning**, which trims tool results in-memory per request without summarization.

---

## Configuration

Compaction model is set in `config/openclaw.json` under `agents.defaults.compaction`:

```json
"agents": {
  "defaults": {
    "compaction": {
      "model": "openai/gpt-5.4"
    }
  }
}
```

**Why a powerful model:** Compaction requires genuine understanding and judgment — it must decide what information is important to preserve, distill complex multi-turn reasoning into summaries, and avoid losing context that affects future behavior. A weaker model risks poor summaries that degrade agent quality.

Both `main` and `public` agents inherit this default. After editing `openclaw.json`: `docker compose restart`.

---

## Manual compaction

Trigger from your messaging channel:

```
/compact
/compact Focus on the key decisions made about X
```

Use `/compact` when:
- Cost per turn has climbed significantly (sign of large accumulated context)
- Starting a new topic where old history is no longer relevant
- Before a long task session where context budget matters

---

## Monitoring context growth

Check session JSONL files for per-turn input token counts:

```bash
ssh openclaw "python3 << 'EOF'
import json, glob
f = sorted(glob.glob('/root/.openclaw/agents/main/sessions/*.jsonl'))[-1]
with open(f) as fh:
    for line in fh:
        obj = json.loads(line)
        usage = obj.get('message',{}).get('usage',{})
        if usage.get('input',0) > 1000:
            print('input=%d  cacheRead=%d  cost=\$%.4f' % (
                usage['input'], usage.get('cacheRead',0), usage.get('cost',{}).get('total',0)))
EOF"
```

A healthy session shows small `input` and large `cacheRead` on most turns. Rising `input` with no `cacheRead` means the cache is cold (first turn of the day) or the context has grown past the cache boundary.