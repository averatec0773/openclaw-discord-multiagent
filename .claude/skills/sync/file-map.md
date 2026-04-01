# File Map — Server ↔ Repo

## Full Mapping Table

| Server path | Repo path | Direction | Notes |
|---|---|---|---|
| `/root/.openclaw/workspace/AGENTS.md` | `templates/AGENTS.md` | server → repo | Behavioral rules for main agent |
| `/root/.openclaw/workspace/SOUL.md` | `templates/SOUL.md` | server → repo | Persona/identity |
| `/root/.openclaw/workspace-public/AGENTS.md` | `templates/AGENTS.public.md` | server → repo | Public agent rules |
| `/root/.openclaw/openclaw.json` | `config/openclaw.json` | server → repo | Redact all secrets before writing |
| `/root/.openclaw/skills/*/SKILL.md` | `notes/workspace/installed-skills.md` + `CONTEXT.md` | server → repo | Update skill tables, not raw file copy |

## Doc-only files (repo-maintained, no server equivalent)

These files are edited locally and committed — no server read needed:

- `notes/channels/discord.md`
- `notes/ops/cron.md`
- `notes/ops/docker.md`
- `notes/ops/ssh.md`
- `notes/ops/updating.md`
- `notes/services/gog.md`
- `notes/services/models.md`
- `notes/workspace/skills.md`
- `notes/workspace/installed-skills.md`
- `notes/workspace/workspace-files.md`
- `notes/clawhub.md`
- `notes/todo.md`
- `CONTEXT.md`
- `installation/setup.md`
- `README.md`

---

## Redaction Rules (config/openclaw.json)

When copying `openclaw.json` to repo, replace all real values under these keys with placeholders:

| Key | Placeholder |
|---|---|
| `OPENAI_API_KEY` | `<YOUR_OPENAI_API_KEY>` |
| `ANTHROPIC_API_KEY` | `<YOUR_ANTHROPIC_API_KEY>` |
| `TAVILY_API_KEY` | `<YOUR_TAVILY_API_KEY>` |
| `DISCORD_PRIVATE_BOT_TOKEN` | `<YOUR_PRIVATE_BOT_TOKEN>` |
| `DISCORD_PUBLIC_BOT_TOKEN` | `<YOUR_PUBLIC_BOT_TOKEN>` |
| `GITHUB_TOKEN` | `<YOUR_GITHUB_PAT>` |
| Any `"token": "<real value>"` inside `channels.discord` | `<YOUR_..._BOT_TOKEN>` |
| Guild IDs (numeric strings) | `<YOUR_GUILD_ID>` |
| User IDs (numeric strings) in `users` arrays | `<YOUR_DISCORD_USER_ID>` |

Python snippet to redact:

```python
import json, re

with open('/root/.openclaw/openclaw.json') as f:
    c = json.load(f)

REDACT = {
    "OPENAI_API_KEY": "<YOUR_OPENAI_API_KEY>",
    "ANTHROPIC_API_KEY": "<YOUR_ANTHROPIC_API_KEY>",
    "TAVILY_API_KEY": "<YOUR_TAVILY_API_KEY>",
    "DISCORD_PRIVATE_BOT_TOKEN": "<YOUR_PRIVATE_BOT_TOKEN>",
    "DISCORD_PUBLIC_BOT_TOKEN": "<YOUR_PUBLIC_BOT_TOKEN>",
    "GITHUB_TOKEN": "<YOUR_GITHUB_PAT>",
}
for k, v in REDACT.items():
    if k in c.get("env", {}):
        c["env"][k] = v

# Redact guild IDs and user IDs in discord accounts
def redact_discord(obj):
    if isinstance(obj, dict):
        result = {}
        for k, v in obj.items():
            if k == "guilds" and isinstance(v, dict):
                result[k] = {"<YOUR_GUILD_ID>": list(v.values())[0] if v else {}}
                # Redact user IDs inside guild
                for guild_cfg in result[k].values():
                    if isinstance(guild_cfg, dict) and "users" in guild_cfg:
                        guild_cfg["users"] = ["<YOUR_DISCORD_USER_ID>"]
            elif k == "token" and isinstance(v, str) and not v.startswith("<"):
                result[k] = "<YOUR_BOT_TOKEN>"
            else:
                result[k] = redact_discord(v)
        return result
    return obj

if "channels" in c and "discord" in c["channels"]:
    c["channels"]["discord"] = redact_discord(c["channels"]["discord"])

print(json.dumps(c, indent=2))
```

---

## Never sync to repo

These files contain real secrets or ephemeral state — never commit:

- `/root/.openclaw/openclaw.json` (real version)
- `/root/openclaw/.env`
- `/root/.openclaw/credentials/` (all files)
- `/root/.openclaw/identity/` (device keypair)
- `/root/.openclaw/workspace/memory/` (daily notes)
- `/root/.openclaw/workspace/MEMORY.md` (long-term memory)
- `/root/.openclaw/gh/hosts.yml`
- `/root/.openclaw/gogcli/`
- `/root/.openclaw/clawhub/config.json`
