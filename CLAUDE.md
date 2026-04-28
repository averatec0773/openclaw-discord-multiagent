# openclaw-discord-multiagent

Reference implementation: self-hosted [OpenClaw](https://docs.openclaw.ai) gateway on a VPS with multiple Discord bots — each mapped to a separate agent with its own workspace, persona, and permissions.

## Prerequisites

- VPS with Docker (Hetzner or similar), SSH alias `openclaw` configured
- Discord applications with bot tokens (one per agent)
- API keys: LLM provider (OpenAI / Anthropic), optionally Tavily, fish-audio TTS

## Quick Start

Follow [.claude/skills/setup/SKILL.md](.claude/skills/setup/SKILL.md) for the full deployment SOP.

## Infrastructure

| Component | Value |
|---|---|
| VPS | Hetzner (Ubuntu), SSH alias `openclaw` |
| Gateway port | `18789` (loopback — access via SSH tunnel: `ssh -N -L 18789:127.0.0.1:18789 openclaw`) |
| Docker image | `openclaw:custom` (built from `Dockerfile.custom`) |
| Compose dir | `~/openclaw/` on server |
| Config | `/root/.openclaw/openclaw.json` on server |

## Agents

| Agent | Model | Workspace | Scope |
|---|---|---|---|
| `main` | primary model | `/root/.openclaw/workspace/` | Owner DMs + private channels, full tools |
| `public` | cost-efficient model | `/root/.openclaw/workspace-public/` | Guild messages, exec/bash/computer denied |
| `roleplay01` | primary model | `/root/.openclaw/workspace-roleplay01/` | Optional — persona agent, TTS enabled |

## Key Files

| File | Purpose |
|---|---|
| `config/openclaw.json` | Full config template (all secrets redacted as `<YOUR_...>`) |
| `Dockerfile.custom` | Extend base image with your tools (gh, gog, clawhub, ffmpeg) |
| `docker-compose.yml` | Container definition |
| `templates/` | Workspace files for each agent — push to server at setup |
| `.claude/skills/setup/` | VPS deployment SOP |
| `.claude/skills/sync/` | Config backup workflow (server → repo) |

## Container Management

```bash
ssh openclaw "cd ~/openclaw && docker compose up -d"             # start / recreate (.env changes)
ssh openclaw "cd ~/openclaw && docker compose restart"           # reload openclaw.json
ssh openclaw "cd ~/openclaw && docker compose build && docker compose up -d"  # rebuild image
ssh openclaw "cd ~/openclaw && docker compose logs -f openclaw-gateway"
```

## Workspace File Rules

| File | Purpose | Maintained by |
|---|---|---|
| `AGENTS.md` | Behavioral rules | Human (from `templates/`) |
| `SOUL.md` | Persona and tone | Human (from `templates/`) |
| `TOOLS.md` | Env-specific config — accounts, IDs | Human (from `templates/`) |
| `USER.md` | Owner profile | Human (from `templates/`) |
| `MEMORY.md` | Session memory | Agent — never overwrite |

## Adapting This Template

- Replace all `<YOUR_...>` placeholders in `config/openclaw.json` with real values
- Edit `templates/` files to match your agents' desired behavior
- Add tools to `Dockerfile.custom` as needed
- The `roleplay01` agent is optional — remove its entry from `openclaw.json` if unused
