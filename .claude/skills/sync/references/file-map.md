# File Map — Private to Public Sync

## Sync Map

Files to copy from private → public when changed.

| Private path | Public path | Notes |
|---|---|---|
| `Dockerfile.custom` | `Dockerfile.custom` | Direct copy |
| `docker-compose.yml` | `docker-compose.yml` | Direct copy |
| `Makefile` | `Makefile` | Direct copy |
| `.env.example` | `.env.example` | Direct copy |
| `config/openclaw.json` | `config/openclaw.json` | Redact before copying (private has real keys) |
| `templates/AGENTS.md` | `templates/AGENTS.md` | Direct copy |
| `templates/AGENTS.public.md` | `templates/AGENTS.public.md` | Direct copy |
| `templates/SOUL.md` | `templates/SOUL.md` | Direct copy |
| `templates/SOUL.public.md` | `templates/SOUL.public.md` | Direct copy |
| `templates/TOOLS.md` | `templates/TOOLS.md` | Direct copy |
| `templates/TOOLS.public.md` | `templates/TOOLS.public.md` | Direct copy |
| `templates/USER.md` | `templates/USER.md` | Direct copy |
| `.claude/skills/setup/SKILL.md` | `.claude/skills/setup/SKILL.md` | Direct copy |
| `.claude/skills/setup/references/` (all) | `.claude/skills/setup/references/` | Direct copy |
| `.claude/skills/sync/SKILL.md` | `.claude/skills/sync/SKILL.md` | Direct copy |
| `.claude/skills/sync/references/file-map.md` | `.claude/skills/sync/references/file-map.md` | Direct copy |

---

## Public Repo Exclusive

These files exist only in the public repo and are maintained there directly. When private repo structural changes affect them, edit in public repo — do not sync from private.

| File | Maintained in public because |
|---|---|
| `CLAUDE.md` | Audience differs — deployer-focused, not ops harness |
| `README.md` | Public-facing, GitHub audience |
| `conventions/` | Minimal template for public deployers |

---

## Never Sync

These files contain personal content or private workflow details.

| Path | Reason |
|---|---|
| `memory/` (all) | Personal rules, state, notes |
| `_local/` | Real secrets and running config |
| `conventions/` | Private workflow conventions — public has its own |
| `.claude/skills/git/SKILL.md` | Private workflow details |
| `.claude/skills/update/` | Private ops — references internal paths |
| `.claude/skills/changelog/` | Private ops |
| `.claude/skills/memory/` | Private ops |
| `.claude/skills/skill-creator/` | Private ops |

---

## Structural Cleanup

When the private repo undergoes major restructuring, delete these stale paths from the public repo **before** copying new files.

| Stale path in public repo | Replaced by |
|---|---|
| `notes/` | `.claude/skills/` pattern |
| `installation/` | `.claude/skills/setup/` |
| `CONTEXT.md` | `CLAUDE.md` |
| `.claude/skills/edit/` | Private-only — remove entirely |
| `.claude/skills/self-improve/` | Private-only — remove entirely |
| `.claude/skills/session/` | CLAUDE.md Session Protocol |

---

## Redaction Rules — config/openclaw.json

The private repo `config/openclaw.json` contains **real keys**. Replace all before copying to public.

| Key | Placeholder |
|---|---|
| `OPENAI_API_KEY` | `<YOUR_OPENAI_API_KEY>` |
| `ANTHROPIC_API_KEY` | `<YOUR_ANTHROPIC_API_KEY>` |
| `TAVILY_API_KEY` | `<YOUR_TAVILY_API_KEY>` |
| `FISH_AUDIO_API_KEY` | `<YOUR_FISH_AUDIO_API_KEY>` |
| `DISCORD_PRIVATE_BOT_TOKEN` | `<YOUR_PRIVATE_BOT_TOKEN>` |
| `DISCORD_PUBLIC_BOT_TOKEN` | `<YOUR_PUBLIC_BOT_TOKEN>` |
| `DISCORD_ROLEPLAY_BOT_TOKEN` | `<YOUR_ROLEPLAY_BOT_TOKEN>` |
| `GITHUB_TOKEN` | `<YOUR_GITHUB_PAT>` |
| Guild IDs (numeric strings) | `<YOUR_GUILD_ID>` |
| User IDs in `users` arrays | `<YOUR_DISCORD_USER_ID>` |
| Any `"token"` inside `channels.discord` | `<YOUR_BOT_TOKEN>` |
