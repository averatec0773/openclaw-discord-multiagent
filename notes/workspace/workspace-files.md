# Workspace Files — Usage Rules

> Official refs:
> - https://docs.openclaw.ai/reference/templates/AGENTS
> - https://docs.openclaw.ai/reference/templates/SOUL
> - https://docs.openclaw.ai/reference/templates/TOOLS
> - https://docs.openclaw.ai/reference/templates/HEARTBEAT
>
> Workspace dir on server: `/root/.openclaw/workspace/`

Each file has a specific purpose. Mixing content between files degrades agent behavior — the LLM reads each file expecting a certain type of content.

---

## File-by-File Rules

### AGENTS.md — Behavioral Instructions

**What belongs here:**
- Session startup sequence (what to read, in what order)
- Behavioral rules ("always do X", "never do Y")
- Skill usage routing rules (which skill to use for what)
- Memory management conventions
- Platform-specific formatting rules
- Red lines / hard limits
- Group chat participation rules

**What does NOT belong here:**
- Infrastructure specifics (accounts, IDs) → TOOLS.md
- Personality / tone → SOUL.md
- User profile → USER.md

**Trimming guidance:** The official template is verbose and covers many scenarios (TTS voice, WhatsApp, Twitter). Remove sections that don't apply to your actual setup — unused instructions still cost tokens every session startup.

Removed from our AGENTS.md vs official template:
- BOOTSTRAP.md First Run section (archived)
- `sag` / ElevenLabs TTS (not installed)
- WhatsApp formatting rules (not configured)
- Twitter heartbeat checks (not configured)
- Heartbeat JSON state template (HEARTBEAT.md is empty)
- "Make It Yours" onboarding section (setup complete)

---

### TOOLS.md — Infrastructure Specifics

**What belongs here:**
- Specific accounts/emails in use
- Device identifiers, SSH hosts, camera names
- API defaults that are environment-specific
- IDs and handles unique to this deployment (Discord guild ID, owner user ID)
- Timezone for the owner

**What does NOT belong here:**
- Behavioral rules ("always use skill X") → AGENTS.md
- How a CLI tool works in general → the skill's SKILL.md
- Personality → SOUL.md

> Official definition: "Skills define *how* tools work. TOOLS.md holds *your* specifics —
> the environment-specific details unique to your setup."

---

### SOUL.md — Personality & Values

**What belongs here:**
- Agent personality traits and tone
- Core values and principles
- **Response style rules** (be specific — vague guidance like "concise when needed" doesn't work)

**What does NOT belong here:**
- Tool configs → TOOLS.md
- Behavioral rules → AGENTS.md
- User info → USER.md

**Note:** The official SOUL.md template is intentionally minimal — just Core Truths, Boundaries, and Vibe. It's a starting point to evolve. Add a concrete **Response Style** section with explicit rules (max length, forbidden phrases, when to go long) since LLMs default to verbose without hard constraints.

---

### USER.md — Human Profile

**What belongs here:**
- Name, preferred address, pronouns
- Timezone and language preferences
- GitHub, Discord handles
- Brief context about workflow preferences

**What does NOT belong here:**
- Behavioral rules for the agent → AGENTS.md
- Infrastructure config → TOOLS.md

---

### MEMORY.md — Long-Term Agent Memory

**What belongs here:**
- Distilled learnings from past sessions
- Decisions made, context behind them
- Patterns the agent should remember across restarts

**Load rules (critical):**
- Load ONLY in main session (direct chat with owner)
- Do NOT load in shared contexts (Discord channels, group chats)
- Reason: contains personal context that must not leak to others

**Maintenance:** During heartbeats, periodically review recent `memory/YYYY-MM-DD.md` files and distill key learnings into MEMORY.md. Remove outdated entries.

---

### HEARTBEAT.md — Periodic Task Checklist

**What belongs here:**
- Short checklist of periodic checks (email, calendar, etc.)
- One-off reminders

**Usage:**
- Keep minimal — every line costs tokens on every heartbeat poll
- Leave empty (or only comments) to skip heartbeat API calls entirely
- Heartbeat = batched, timing can drift; Cron = exact timing, isolated tasks

**Official template content:** Just a comment line saying to add tasks below when you want the agent to check something periodically. Nothing else.

---

### IDENTITY.md — Agent Self-Definition

**What belongs here:**
- Agent's chosen name, creature type, vibe, emoji, avatar path

**Note:** Auto-generated as a template by OpenClaw. The agent fills it in during first conversation.

---

## Current State (as of 2026-03-12)

| File | Status | Notes |
|------|--------|-------|
| `AGENTS.md` | Optimized | Trimmed from 168 → 93 lines; removed inapplicable sections |
| `TOOLS.md` | Good | Gmail account, Discord guild/user ID, timezone only |
| `SOUL.md` | Good | Official template + custom Response Style section |
| `USER.md` | Good | Scott's profile filled in |
| `MEMORY.md` | Active | Agent writes to this each session |
| `HEARTBEAT.md` | Empty | No periodic tasks configured |
| `IDENTITY.md` | Empty | Agent hasn't filled this in yet |

---

## Common Mistakes to Avoid

| Mistake | Problem | Fix |
|---------|---------|-----|
| Skill routing rules in TOOLS.md | LLM treats TOOLS.md as config, not instructions | Move to AGENTS.md |
| Backup instructions in TOOLS.md | Same issue | Move to AGENTS.md |
| Personal context in MEMORY.md loaded in group chats | Privacy leak | Gate MEMORY.md to main sessions in AGENTS.md |
| Long HEARTBEAT.md | Token burn on every poll | Keep minimal |
| Vague response style ("concise when needed") | LLM defaults to verbose | Add explicit rules in SOUL.md |
| Keeping unused template sections in AGENTS.md | Wasted tokens per session | Remove inapplicable sections (TTS, WhatsApp, Twitter) |
