---
name: session
description: Invoke at the start of every new session. Loads context about this repo, OpenClaw, and all available skills. Establishes language and writing rules for the session.
user-invocable: true
---

# Session Startup

Run this skill at the beginning of every new session before doing any work.

---

## Agent Identity

This agent (Claude Code) works in partnership with you to manage a self-hosted OpenClaw instance.

**Agent responsibilities:**

| Responsibility | Description |
|---|---|
| Modify and improve the repo | Update documentation, skills, config templates, and notes |
| Modify and improve the server | Edit OpenClaw workspace files, config, and server-side content via SSH |
| Verify repo-server consistency | Detect and resolve divergence between repo files and their server equivalents |

**Repo purpose:**

| Purpose | Description |
|---|---|
| Operational guide | Instructs the agent on how to assist you in setting up, modifying, and optimizing OpenClaw |
| Knowledge base | Records customizations, decisions, and operational knowledge specific to your instance |

---

## Step 1 — Read all skills

Read every skill in `.claude/skills/` to understand what tools are available:

- `.claude/skills/session/SKILL.md` — this file (session startup)
- `.claude/skills/sync/SKILL.md` — server-to-repo sync with conflict detection
- `.claude/skills/sync/file-map.md` — server ↔ repo file mapping and redaction rules
- `.claude/skills/edit/SKILL.md` — governs all write operations to server or repo files
- `.claude/skills/self-improve/SKILL.md` — records learnings and proposes structural improvements

---

## Step 2 — Read repo context

Read these files to understand the environment and customizations:

**Required:**
- `CONTEXT.md` — infrastructure, Docker setup, multi-agent configuration, installed skills
- `README.md` — project overview

**As needed (read before working on related topics):**
- `notes/ops/docker.md` — Docker image management and container operations
- `notes/ops/ssh.md` — SSH alias and file transfer
- `notes/ops/updating.md` — OpenClaw update procedure
- `notes/channels/discord.md` — Discord channel config
- `notes/services/models.md` — LLM provider and model reference
- `notes/workspace/skills.md` — skills loading, directory structure, file format
- `notes/workspace/installed-skills.md` — currently installed skills
- `notes/workspace/workspace-files.md` — workspace file usage rules (AGENTS, SOUL, TOOLS, etc.)
- `notes/clawhub.md` — ClawHub CLI reference
- `config/openclaw.json` — config template (redacted)
- `templates/AGENTS.md` — main agent behavioral rules
- `templates/AGENTS.public.md` — public agent behavioral rules
- `templates/SOUL.md` — main agent persona
- `installation/setup.md` — initial setup guide

---

## Step 3 — Load OpenClaw knowledge

Official documentation: https://docs.openclaw.ai

Key sections relevant to this setup:
- https://docs.openclaw.ai/start/getting-started
- https://docs.openclaw.ai/install/hetzner

Fetch documentation pages when working on topics not covered by the repo files.

---

## Task Execution Rules

### When to create a todo list

Create a todo list for any task that is long, multi-step, or involves changes to multiple files or systems.

**Triggers:**
- Task requires 3 or more distinct operations
- Task spans both repo and server (two systems)
- Task involves unfamiliar territory or commands not recently used
- User provides a list of things to accomplish

### How to work through a todo list

1. **Break down first** — before executing anything, decompose the full task into discrete steps
2. **Think before each step** — read relevant files or docs, understand current state, then plan
3. **One step at a time** — mark in-progress before starting; mark complete only after verifying
4. **Do not guess** — if the correct approach is unclear, read docs or files first
5. **Stop and ask when blocked** — ask the user rather than proceeding with a best guess

### Operations that require extra care

| Operation | Risk | Required before executing |
|---|---|---|
| `scp` or file writes to server | Overwrites existing content | Read target file first; use `edit` skill |
| `docker compose up -d` or `restart` | Restarts live service | Confirm config is valid |
| `git push` | Permanent remote change | Verify staged files and commit message |
| Editing `openclaw.json` | May break gateway | Read current file first; modify only target keys |
| Deleting files or directories | Irreversible | Confirm the path and contents are not needed |
| SSH commands that modify system state | May affect running service | Understand command fully before running |

If you are not certain a command is safe and correct, do not run it. Read more context, check docs, or ask.

---

## Language Rules

| Scope | Language |
|---|---|
| All file content written or modified | English only |
| Conversation with user | Match the user's language |
| Git commit messages | English only |

---

## Summary

After completing steps 1-3, briefly confirm:
- Skills loaded
- Key context files read
- Ready to assist
