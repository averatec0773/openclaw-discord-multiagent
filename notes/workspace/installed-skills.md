# Installed Skills — Current State

> For loading mechanism, directory structure, and skill file format, see [skills.md](skills.md).

This file tracks the skills installed in this OpenClaw deployment.
Update it whenever you install, update, or remove a skill.

---

## Currently Installed Skills

Replace the example entries below with your actual installed skills.

| Skill | Version | Source | Location |
|---|---|---|---|
| `self-improving-agent` | latest | clawhub | `/home/node/.openclaw/skills/` |
| `discord` | latest | clawhub (steipete) | `/home/node/.openclaw/skills/` |
| (your custom skills) | — | custom | `/home/node/.openclaw/skills/` |

---

## Installing a Skill from ClawHub

```bash
# Install (persistent to config volume, auto-loaded)
docker compose exec --user root openclaw-gateway \
  clawhub install <slug> --workdir /home/node/.openclaw --dir skills

# List installed
docker compose exec openclaw-gateway clawhub list --workdir /home/node/.openclaw --dir skills

# Update all
docker compose exec --user root openclaw-gateway \
  clawhub update --all --workdir /home/node/.openclaw --dir skills
```

---

## Deploying a Custom Skill

```bash
# Create skill directory on server
ssh openclaw "mkdir -p /root/.openclaw/skills/<skill-name>"

# Upload SKILL.md
scp path/to/SKILL.md openclaw:/root/.openclaw/skills/<skill-name>/SKILL.md

# OpenClaw hot-reloads — no restart needed
```

---

## Discord Skill Division

If you install both `averatec-discord` (or your own proactive Discord skill) and `discord` (steipete), use them as follows:

| Skill | Use for |
|---|---|
| Proactive outbound skill | DMs, push notifications, initiating contact when no conversation exists |
| `discord` (steipete) | In-conversation — reactions, polls, threads, pins, search, moderation |
