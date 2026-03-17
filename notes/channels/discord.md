# OpenClaw Discord Channel

Docs: https://docs.openclaw.ai/channels/discord

## Current Configuration

Config file: `~/.openclaw/openclaw.json` — `channels.discord` field

Two Discord bots are configured under `channels.discord.accounts`, each routing to a different agent via `bindings`.

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "accounts": {
        "default": {
          "token": "$DISCORD_PRIVATE_BOT_TOKEN",
          "groupPolicy": "allowlist",
          "streaming": "partial",
          "guilds": {
            "<GUILD_ID>": {
              "requireMention": true,
              "users": ["<OWNER_DISCORD_USER_ID>"]
            }
          }
        },
        "public": {
          "token": "$DISCORD_PUBLIC_BOT_TOKEN",
          "groupPolicy": "allowlist",
          "streaming": "partial",
          "guilds": {
            "<GUILD_ID>": { "requireMention": true }
          }
        }
      }
    }
  },
  "bindings": [
    { "agentId": "main",   "match": { "channel": "discord", "accountId": "default" } },
    { "agentId": "public", "match": { "channel": "discord", "accountId": "public" } }
  ]
}
```

- `groupPolicy: "allowlist"` — only guilds listed in the `guilds` field are allowed
- `streaming: "partial"` — edits a single preview message as tokens arrive
- `requireMention: true` — both bots only respond when @mentioned in guild channels
- `users` (default account only) — restricts guild responses to the owner's Discord user ID; other users who @mention the private bot will be ignored
- `dmPolicy` not set, defaults to `"pairing"` — DM users must be approved via pairing code

DM allowlist: `~/.openclaw/credentials/discord-default-allowFrom.json`

> **Single-bot setup** (simpler alternative): Replace `accounts` with a single `token` field directly in `channels.discord`. See [Core Config Options](#core-config-options) below.

---

## Core Config Options

### groupPolicy

| Value | Description |
|---|---|
| `"allowlist"` | Only guilds listed in `guilds` field |
| `"open"` | All servers allowed |
| `"disabled"` | Guild channels disabled |

### dmPolicy

| Value | Description |
|---|---|
| `"pairing"` | Default — requires pairing code approval (1-hour expiry) |
| `"allowlist"` | Pre-approved users only |
| `"open"` | Unrestricted (requires `allowFrom: "*"`) |
| `"disabled"` | DMs blocked |

### streaming

| Value | Description |
|---|---|
| `"off"` | Wait for full response, then send |
| `"partial"` | Edit a single message as tokens stream in (current) |
| `"block"` | Send chunks with configurable break points |
| `"progress"` | Alias for `partial` (cross-channel consistency) |

### Guild Configuration

```json
{
  "guilds": {
    "GUILD_ID": {
      "requireMention": false,
      "users": ["USER_ID"],
      "roles": ["ROLE_ID"],
      "channels": {
        "general": { "allow": true },
        "help": { "allow": true, "requireMention": true }
      }
    }
  }
}
```

---

## Common Adjustments

### Edit config

```bash
# Edit on the server (restart container after)
docker compose exec openclaw bash
nano ~/.openclaw/openclaw.json

# Or set token via CLI
openclaw config set channels.discord.token "YOUR_TOKEN"
```

### Approve a DM pairing request (when dmPolicy is "pairing")

After the user sends a pairing request via DM:
```bash
docker compose exec openclaw openclaw pair --approve
```

### Block streaming config (when streaming: "block")

```json
{
  "channels": {
    "discord": {
      "draftChunk": {
        "minChars": 200,
        "maxChars": 800,
        "breakPreference": "paragraph"
      }
    }
  }
}
```

### Timeout config (for long-running tasks)

```json
{
  "channels": {
    "discord": {
      "eventQueue": { "listenerTimeout": 120000 },
      "inboundWorker": { "runTimeoutMs": 1800000 }
    }
  }
}
```

---

## Bot Permissions Required

- View Channels
- Send Messages
- Read Message History
- Embed Links
- Attach Files
- Add Reactions (optional)

Required Intents:
- Message Content Intent
- Server Members Intent
