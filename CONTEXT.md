# OpenClaw Setup Context

> Docs: https://docs.openclaw.ai/start/getting-started | https://docs.openclaw.ai/install/hetzner

This file is the primary reference for understanding and managing this OpenClaw instance.
Read it first in any new session before making changes.

---

## What is OpenClaw

OpenClaw is a self-hosted AI gateway that connects messaging channels (Discord, WhatsApp, Telegram, iMessage)
to AI coding agents. It runs as a single Docker container on a VPS.

Official docs: https://docs.openclaw.ai

---

## Infrastructure

| Component | Value |
|---|---|
| VPS provider | Hetzner (Ubuntu) — or your preferred VPS |
| SSH alias | `openclaw` (configured in `~/.ssh/config`) |
| Working directory on server | `~/openclaw/` |
| Gateway port | `18789` (loopback only, access via SSH tunnel) |
| Control UI | `http://127.0.0.1:18789/` after tunnel |

---

## Docker Setup

| Image | Purpose |
|---|---|
| `openclaw:latest` | Upstream image from Docker Hub |
| `openclaw:<your-tag>-custom` | Custom image built on top via `Dockerfile.custom` |

`Dockerfile.custom` adds tools on top of the upstream image. The custom image name is controlled by `OPENCLAW_IMAGE=openclaw:<your-tag>-custom` in `~/openclaw/.env` on the server.
The `.env` is gitignored by the upstream repo — upstream `git pull` will never overwrite it.

### Container management

```bash
ssh openclaw
cd ~/openclaw

docker compose build          # rebuild custom image layer
docker compose up -d          # start / recreate (picks up .env changes)
docker compose down           # stop
docker compose restart        # soft restart (picks up openclaw.json changes only)
docker compose logs -f openclaw-gateway
```

### When to restart

| Change | Action |
|---|---|
| Edit `openclaw.json` | `docker compose restart` |
| Edit `.env` | `docker compose up -d` (recreates container) |
| Rebuild image (`Dockerfile.custom`) | `docker compose build && docker compose up -d` |

---

## Volume Mapping

| Container path | Host path | Persistent |
|---|---|---|
| `/home/node/.openclaw/` | `/root/.openclaw/` | Yes — config, credentials |
| `/home/node/.openclaw/skills/` | `/root/.openclaw/skills/` | Yes — global managed skills (shared across all agents) |
| `/home/node/.openclaw/workspace/` | `/root/.openclaw/workspace/` | Yes — main agent workspace files |
| `/app/skills/` | *(none)* | No — bundled skills, baked into image |

### Skills Scope

| Location | Scope | Managed by |
|---|---|---|
| `/app/skills/` | All agents (bundled, read-only) | Image / OpenClaw upstream |
| `/home/node/.openclaw/skills/` | All agents (global) | `clawhub install` |
| `<workspace>/skills/` | Agent-specific only | Manual / scp |

---

## OpenClaw Configuration

Config file on server: `~/.openclaw/openclaw.json`
Template (secrets redacted): [config/openclaw.json](config/openclaw.json)

Key settings pattern used in this deployment:
- `agents.defaults.model.primary` → your chosen primary model
- `skills.load.extraDirs` → `["/home/node/.openclaw/skills"]` (makes user skills auto-loaded)
- `channels.discord` → enabled, `groupPolicy: allowlist`, `streaming: partial` (per-account)
- `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback` → `true` (required for SSH tunnel access)

After editing `openclaw.json`: `docker compose restart`

---

## Authenticated Services

Customize based on what you integrate. Example services:

| Service | Storage | Notes |
|---|---|---|
| Google (gog) | `/home/node/.openclaw/gogcli/keyring/` | File keyring, set `GOG_KEYRING_PASSWORD` in `.env` |
| GitHub (gh) | `/home/node/.openclaw/gh/hosts.yml` | Token-based |
| ClawHub | `/home/node/.openclaw/clawhub/config.json` | Token in `.env` |
| Tavily Search | — | API key in `openclaw.json` env section |

---

## Web Search & Networking

Three ways the bot can access the internet:

| Method | Tool | Requires | Notes |
|---|---|---|---|
| `web_search` | Tavily/Brave Search API | `TAVILY_API_KEY` in `openclaw.json` | Best for open-ended queries |
| `web_fetch` | Built-in HTTP fetch | Nothing | Requires known URL; may fail on JS-heavy pages |
| `browser` | Chromium (headless) | Chromium in image or remote browser service | Not configured by default |

**After adding `TAVILY_API_KEY` to `openclaw.json`:** run `docker compose down && docker compose up -d` (full recreate required).

Tavily free tier: 1000 queries/month, no credit card required. Sign up at tavily.com.

---

## Active Channels

| Channel | Status | Notes |
|---|---|---|
| Discord | Active | `requireMention: true` (both bots) |

---

## Multi-Agent Setup

Two agents running on the same gateway instance, each with its own Discord bot and workspace:

| Agent ID | Model | Workspace | Scope |
|---|---|---|---|
| `main` | your primary model | `workspace/` | Owner DMs and private channels |
| `public` | your cost-efficient model | `workspace-public/` | Guild messages |

**Routing:** `bindings` array in `openclaw.json`, matched by `accountId` — each Discord bot account routes to its agent.

```json
{ "agentId": "main",   "match": { "channel": "discord", "accountId": "default" } },
{ "agentId": "public", "match": { "channel": "discord", "accountId": "public" } }
```

**`main` agent restrictions (guild channels):**
- `requireMention: true` — only responds when @mentioned
- `users: [<YOUR_DISCORD_USER_ID>]` — only processes messages from the owner

**`public` agent restrictions:**
- `requireMention: true` — only responds when @mentioned
- `tools.deny: ["exec", "bash", "computer"]` — no shell access
- No USER.md, no MEMORY.md in workspace — no personal context loaded

**Directory structure:**
```
/root/.openclaw/
├── workspace/            ← main agent workspace
├── workspace-public/     ← public agent workspace
│   ├── SOUL.md           ← public persona
│   └── AGENTS.md         ← simplified rules, no MEMORY.md loading
└── agents/
    ├── main/sessions/
    └── public/sessions/
```

---

## Installed Skills

See [notes/workspace/installed-skills.md](notes/workspace/installed-skills.md) for the full list of skills installed in this deployment.

Built-in skills (image layer, `/app/skills/`): 1password, apple-notes, canvas, clawhub, coding-agent,
discord, gemini, gh-issues, github, healthcheck, notion, obsidian, openai-image-gen, openai-whisper,
slack, spotify-player, summarize, tmux, trello, weather, and more.

### Install a new skill (persistent)

```bash
docker compose exec --user root openclaw-gateway \
  clawhub install <slug> --workdir /home/node/.openclaw --dir skills
```

---

## Access

### SSH Alias Setup

Add the following to `~/.ssh/config` on your local machine (replace `<VPS_IP>` with your server's IP):

```
Host openclaw
    HostName <VPS_IP>
    User root
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60

Host tunnel-openclaw
    HostName <VPS_IP>
    User root
    LocalForward 18789 127.0.0.1:18789
    ServerAliveInterval 60
```

After saving, `ssh openclaw` works from anywhere without specifying IP or key.

See [notes/ops/ssh.md](notes/ops/ssh.md) for file transfer (`scp`) examples.

### Shell Access

```bash
ssh openclaw                    # open shell on server
```

### Dashboard (Control UI)

Option A — one-off tunnel:
```bash
ssh -N -L 18789:127.0.0.1:18789 openclaw
```

Option B — use the `tunnel-openclaw` alias (stays connected):
```bash
ssh tunnel-openclaw
```

Then open: `http://127.0.0.1:18789/`
