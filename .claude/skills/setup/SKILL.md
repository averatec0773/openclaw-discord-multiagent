---
name: setup
description: VPS reinstall SOP for this OpenClaw deployment. Use for fresh installs or full rebuilds after data loss. For routine container updates only, stop here — use the update skill instead.
metadata:
  version: 1.0.0
  category: operations
  dangerous: true
---

# Setup — OpenClaw VPS Reinstall

## When to use

Fresh install on a new Hetzner VPS, or full rebuild after catastrophic failure. Check `memory/state.md` for current deployment state before starting.

## Prerequisites

- SSH alias `openclaw` configured — see `references/ssh.md`
- Backups pulled to local machine: `_local/`, `identity/`, `gogcli/`, Discord credentials
- API keys on hand: OpenAI, Tavily, Discord bot tokens (default / public / roleplay)

## Steps

### 1 — Install Docker on VPS

```bash
ssh root@<VPS_IP>
apt-get update && apt-get install -y curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

### 2 — Create persistent directories

```bash
mkdir -p /root/.openclaw/workspace /root/.openclaw/workspace-public /root/.openclaw/workspace-roleplay01
chown -R 1000:1000 /root/.openclaw
```

Ownership must be `1000` (node) — OpenClaw silently fails to write if owned by root.

### 3 — Upload project files

```bash
scp Dockerfile.custom docker-compose.yml openclaw:~/openclaw/
scp _local/.env openclaw:~/openclaw/.env
scp config/openclaw.json openclaw:/root/.openclaw/openclaw.json
```

Fill real secrets in `/root/.openclaw/openclaw.json` — replace all `<YOUR_...>` placeholders. Add the `roleplay01` agent entry (not in the redacted template — see `memory/state.md` for the full agent list).

### 4 — Build and launch

```bash
ssh openclaw "cd ~/openclaw && docker compose build && docker compose up -d"
ssh openclaw "docker compose logs openclaw-gateway --tail 20"
# Expect: "listening on ws://0.0.0.0:18789"
```

### 5 — Authenticate services

```bash
docker compose exec --user root openclaw-gateway gh auth login
docker compose exec --user root openclaw-gateway clawhub login --token <CLAWHUB_TOKEN>
# Google (gog): requires SSH tunnel — see references/gog.md
```

### 6 — Push workspace templates

```bash
for f in AGENTS SOUL TOOLS USER; do
  scp templates/$f.md openclaw:/root/.openclaw/workspace/$f.md
done
scp templates/AGENTS.public.md  openclaw:/root/.openclaw/workspace-public/AGENTS.md
scp templates/SOUL.public.md    openclaw:/root/.openclaw/workspace-public/SOUL.md
scp templates/TOOLS.public.md   openclaw:/root/.openclaw/workspace-public/TOOLS.md
ssh openclaw "chown -R 1000:1000 /root/.openclaw/workspace*"
```

### 7 — Install global skills

```bash
docker compose exec --user root openclaw-gateway \
  clawhub install <slug> --workdir /home/node/.openclaw --dir skills
```

Install each skill listed in `memory/state.md` → Installed Skills table.

### 8 — Install fish-audio TTS plugin

```bash
docker compose exec openclaw-gateway openclaw plugins install @conan-scott/openclaw-fish-audio
ssh openclaw "cd ~/openclaw && docker compose restart"
```

Add `messages.tts` config to `openclaw.json` — see `references/plugins.md`.

### 9 — Re-pair device

Approve in control UI at `http://127.0.0.1:18789/` after SSH tunnel, or:

```bash
docker compose exec openclaw-gateway openclaw devices approve <requestId>
```

## DO NOT

- Do not skip the `chown -R 1000:1000` step — workspace writes will fail silently.
- Do not restore `identity/` from a backup to a different machine without understanding re-pairing consequences.

## Acceptance Criteria

```bash
ssh openclaw "docker ps | grep openclaw-gateway"
ssh openclaw "head -3 /root/.openclaw/workspace/AGENTS.md"
docker compose exec openclaw-gateway clawhub list
# All three Discord bots online in guild
```
