<div align="center">
  <h1>OpenClaw Discord Multi-Agent Setup</h1>
  <p>A production reference for running a self-hosted <a href="https://openclaw.ai">OpenClaw</a> instance with two Discord bots on one gateway — a private assistant and a public-facing agent, each with its own workspace, persona, and permissions.</p>

  <a href="https://openclaw.ai"><img src="https://img.shields.io/badge/OpenClaw-CC2233?style=flat-square&logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAABmJLR0QA/wD/AP+gvaeTAAAA80lEQVRYhWNgGAUjHTASUmDFxfufEguOffuM1w4mSgynBsDpOpjPYT747+9AUkgwbjzAiM0cdDD4QoCQz7+t2c/AwMDAwBXiyECMOKGQGDwhAHPh0dwgiMCNh1g1cB68xsDAwMDw3V6LKHE40JBnYGBgYLCevI6BgQEREoMnBP6Xx6OmchwhQDaAhgDc4s6FgyMEWGAMWNzA0wCNAMweGBjwECC5HCDZgiFTDqCDEVMXEARWXLz/KW0T4AMDHgIEW0QwQGooEGoJwcDgDwF4LelqTJLB1rvPMjAwDKc2IQwQKg9g+Z+QOTAweEMAHQzbXDAKAN+acUaL3dFjAAAAAElFTkSuQmCC&logoColor=white" /></a>
  <a href="https://discord.com"><img src="https://img.shields.io/badge/Discord-5865F2?style=flat-square&logo=discord&logoColor=white" /></a>
  <a href="https://www.docker.com"><img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white" /></a>
  <a href="https://claude.ai/code"><img src="https://img.shields.io/badge/Claude_Code-CC785C?style=flat-square&logo=anthropic&logoColor=white" /></a>
  <a href="https://chatgpt.com"><img src="https://img.shields.io/badge/ChatGPT-74aa9c?style=flat-square&logo=openaigym&logoColor=white" /></a>
  <a href="https://www.hetzner.com"><img src="https://img.shields.io/badge/Hetzner_VPS-D50C2D?style=flat-square&logo=hetzner&logoColor=white" /></a>
</div>

---

## What this is

This repo is a **knowledge base and template** for a self-hosted OpenClaw deployment on a VPS. It documents the architecture decisions, config patterns, and operational workflows of a real production instance.

Use it as:
- A starting point for your own OpenClaw multi-agent Discord setup
- A reference for routing multiple Discord bots to separate agents on one gateway
- A pattern for managing your instance with **Claude Code** (AI-assisted server management)
- A guide for extending the official Docker image with your own tools

> This is not the OpenClaw source code. It is a configuration and knowledge base for running your own instance.

---

## Who this is for

- You want to self-host an AI assistant accessible via Discord
- You want **separate agents** for private use and public guild channels, on one server
- You want to extend the default OpenClaw image with tools like GitHub CLI, Gmail, or Google Places
- You want to manage your deployment with **Claude Code** — reading, editing, and syncing server files from your local machine

---

## Architecture

Two agents run on the same gateway, each bound to a dedicated Discord bot via `bindings` in `openclaw.json`:

| Agent | Purpose | Discord | Shell | Memory |
|---|---|---|---|---|
| `main` | Private assistant — owner only | Private bot | Full access | Persistent `MEMORY.md` |
| `public` | Public guild bot | Public bot | Restricted (no exec/bash) | Stateless |

Routing is done by Discord bot `accountId` — each bot token maps to one agent. This means two Discord apps, one gateway, zero overlap.

---

## What's included

### Multi-agent Discord routing
Config patterns for running `main` and `public` agents on one gateway, with per-agent permissions, workspaces, and personas. Includes guild allowlisting, `requireMention`, user-level access control for the private bot, and `dmPolicy` options.

### Custom Docker image
A `Dockerfile.custom` pattern for extending `openclaw:latest` with your own tools (GitHub CLI, Gmail OAuth CLI, Google Places, ClawHub). The custom image name is pinned via `.env` so upstream `git pull` never overwrites it.

### OpenClaw config template
A fully redacted `openclaw.json` template covering:
- Multi-agent `bindings`
- Discord account config (streaming, groupPolicy, dmPolicy)
- Model config with fallbacks
- Skill loading from managed paths
- Web search (Tavily) and Google Workspace integration

### Workspace file organization
Documented rules for what belongs in each workspace file (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `MEMORY.md`) — what to put where, what to trim, common mistakes. Templates for both private and public agents.

### Claude Code skill system
A set of `.claude/skills/` that turn Claude Code into an AI-assisted management tool for your OpenClaw instance:

| Skill | Purpose |
|---|---|
| `setup` | Step-by-step VPS deployment SOP — Docker, config, workspace, skills, TTS |
| `sync` | Config backup workflow — copy server-side changes back to repo |

### Operations runbooks
Step-by-step guides for Docker image management, SSH tunnel setup, updating OpenClaw, and skill installation — all documented from real operational experience.

---

## Repository structure

```
openclaw-discord-multiagent/
├── CLAUDE.md                # Agent entry point — quick start, key files, container commands
├── Dockerfile.custom        # Custom image definition (extend openclaw:latest)
├── docker-compose.yml       # Container configuration
├── Makefile                 # Common project commands
├── .env.example             # Environment variable reference
├── config/
│   └── openclaw.json        # Config template (all secrets redacted as <YOUR_...>)
├── templates/
│   ├── AGENTS.md            # Main agent behavioral rules template
│   ├── AGENTS.public.md     # Public agent behavioral rules template
│   ├── SOUL.md              # Private agent persona template
│   ├── SOUL.public.md       # Public agent persona template
│   ├── TOOLS.md             # Environment-specific config template (main)
│   ├── TOOLS.public.md      # Environment-specific config template (public)
│   └── USER.md              # User profile template
└── .claude/skills/          # Claude Code skills for AI-assisted management
    ├── setup/               # VPS deployment SOP with step-by-step instructions
    └── sync/                # Config backup workflow — server → repo
```

---

## Getting started

1. Read [CLAUDE.md](CLAUDE.md) for infrastructure overview and key files
2. Follow [.claude/skills/setup/SKILL.md](.claude/skills/setup/SKILL.md) for the full deployment SOP
3. Fill in your keys — replace all `<YOUR_...>` in `config/openclaw.json`
4. Push `templates/` files to your agent workspaces and customize them

---

## Related

- [OpenClaw Documentation](https://docs.openclaw.ai)
- [ClawHub](https://clawhub.ai) — skill registry for OpenClaw
- [OpenClaw on GitHub](https://github.com/openclaw/openclaw)
