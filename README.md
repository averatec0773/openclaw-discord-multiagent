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
| `session` | Session startup: load context, read skills, establish language rules |
| `sync` | Sync server workspace files to repo; conflict detection before any write |
| `edit` | Safe modification protocol for server and repo files |
| `self-improve` | Record learnings; propose structural improvements to repo and server |

### Operations runbooks
Step-by-step guides for Docker image management, SSH tunnel setup, updating OpenClaw, and skill installation — all documented from real operational experience.

---

## Repository structure

```
openclaw-discord-multiagent/
├── CONTEXT.md               # Full environment reference — paths, config, current state
├── Dockerfile.custom        # Custom image definition (extend openclaw:latest)
├── docker-compose.yml       # Container configuration
├── config/
│   └── openclaw.json        # Config template (all secrets redacted)
├── installation/
│   └── setup.md             # Deployment guide — VPS, Docker, initial setup
├── notes/
│   ├── ops/
│   │   ├── docker.md        # Image management and container operations
│   │   ├── ssh.md           # SSH alias, tunnel, file transfer
│   │   ├── updating.md      # Update procedure for Docker-based setup
│   │   ├── cron.md          # Scheduled agent tasks — job fields, token cost, management
│   │   ├── compaction.md    # Context compaction — config, manual trigger, cost monitoring
│   │   └── usage.md         # Token usage analysis — reading JSONL, cost breakdown scripts
│   ├── workspace/
│   │   ├── skills.md        # Skills loading, directory structure, file format
│   │   ├── installed-skills.md  # Skills installed in this deployment
│   │   └── workspace-files.md   # Workspace file usage rules
│   ├── channels/
│   │   └── discord.md       # Discord channel config reference
│   ├── services/
│   │   └── models.md        # LLM provider and model reference
│   └── clawhub.md           # ClawHub CLI reference
├── templates/
│   ├── AGENTS.md            # Main agent behavioral rules template
│   ├── AGENTS.public.md     # Public agent behavioral rules template
│   ├── SOUL.md              # Private agent persona template
│   ├── SOUL.public.md       # Public agent persona template
│   ├── TOOLS.md             # Environment-specific tools template
│   └── USER.md              # User profile template
└── .claude/skills/          # Claude Code skills for managing this instance
    ├── session/             # Session startup and context loading
    ├── sync/                # Server ↔ repo sync with conflict detection
    ├── edit/                # Safe file modification protocol
    └── self-improve/        # Learning capture and structural improvement
```

---

## Getting started

1. Read [CONTEXT.md](CONTEXT.md) for the full environment reference
2. Follow [installation/setup.md](installation/setup.md) for deployment steps
3. Copy `config/openclaw.json` to your server and fill in your keys
4. Copy the `templates/` files to your agent workspace and customize them
5. If using Claude Code to manage your instance, load `.claude/skills/` into your project

---

## Related

- [OpenClaw Documentation](https://docs.openclaw.ai)
- [ClawHub](https://clawhub.ai) — skill registry for OpenClaw
- [OpenClaw on GitHub](https://github.com/openclaw/openclaw)
