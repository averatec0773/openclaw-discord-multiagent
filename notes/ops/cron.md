# Cron — Scheduled Agent Tasks

OpenClaw includes a built-in cron scheduler. Jobs are defined per-agent and persist across container rebuilds.

---

## Storage

Jobs are stored in `/root/.openclaw/cron/jobs.json` on the host — inside the persistent config volume.
They survive `docker compose up -d`, image rebuilds, and container restarts.

---

## How it works

When a scheduled job fires, OpenClaw creates an agent session and injects the `payload.message` as a user turn.
The agent processes it exactly like a real message from a user, reads any referenced files, and delivers the result via the configured channel.

```
cron scheduler
  → trigger time reached
  → create session (isolated or resuming)
  → load workspace files (AGENTS.md, SOUL.md, MEMORY.md, etc.)
  → inject payload.message as user input
  → agent generates response
  → deliver via channel (Discord DM, channel post, etc.)
```

---

## Job fields

| Field | Description |
|---|---|
| `agentId` | Which agent runs the job (`main`, `public`, etc.) |
| `schedule.kind` | `"cron"` — standard cron expression |
| `schedule.expr` | Cron expression, e.g. `"0 9 * * *"` (9 AM daily) |
| `schedule.tz` | Timezone for the expression, e.g. `"America/New_York"` |
| `sessionTarget` | `"isolated"` — fresh session each run, no history; or `"resume"` — continues last session |
| `wakeMode` | `"now"` — fires immediately at trigger time |
| `payload.kind` | `"agentTurn"` — full LLM call; agent reads, reasons, and responds |
| `payload.message` | The instruction injected as the user message |
| `payload.timeoutSeconds` | Max time the agent has to complete (e.g. `120`) |
| `delivery.mode` | `"announce"` — delivers to a channel or user |
| `delivery.channel` | Channel type, e.g. `"discord"` |
| `delivery.to` | Target, e.g. `"user:<discord_user_id>"` |

---

## Token consumption

`payload.kind: "agentTurn"` is a real LLM call. Each run consumes tokens.

Typical breakdown per run:

| Component | Approximate tokens |
|---|---|
| System context (AGENTS.md + SOUL.md + MEMORY.md + USER.md) | 3000–5000 |
| Tool call results (e.g. file read) | 200–500 |
| Generated output | 200–400 |
| **Total per run** | **~3500–6000** |

With `sessionTarget: "isolated"`, context is reset each run — token usage is predictable and does not grow over time.

---

## Managing jobs

### Via the agent (recommended)

Tell the agent directly in your messaging channel:

```
Create a cron job that runs every day at 8 AM New York time and sends me a summary of X.
Disable the cron job named "Daily TODO summary".
Show me all active cron jobs.
```

The agent can create, update, enable, disable, and delete jobs via the `openclaw` CLI or internal tools.

### Via the dashboard

Open the Control UI at `http://127.0.0.1:18789/` (requires SSH tunnel) and manage jobs from the Cron section.

### Directly (editing jobs.json)

Not recommended — the scheduler reads this file at runtime. Edits while the container is running may be overwritten. Use the agent or dashboard instead.

---

## Example: daily morning summary

```json
{
  "name": "Daily TODO summary DM",
  "agentId": "main",
  "schedule": { "kind": "cron", "expr": "0 9 * * *", "tz": "America/New_York" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "Read TODO.md and send a brief daily summary via Discord DM.",
    "timeoutSeconds": 120
  },
  "delivery": { "mode": "announce", "channel": "discord", "to": "user:<YOUR_DISCORD_USER_ID>" }
}
```
