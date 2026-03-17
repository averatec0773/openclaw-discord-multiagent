# ClawHub — Skill Registry for OpenClaw

> ClawHub is the official public skill registry for OpenClaw.
> The `clawhub` CLI is required to install, update, or publish skills.
>
> Registry: https://clawhub.ai — Reference: https://docs.openclaw.ai/tools/clawhub

---

## Install the CLI

```bash
npm i -g clawhub
# or
pnpm add -g clawhub
```

Already included in `openclaw:averatec-custom` via `Dockerfile.custom`.

---

## Authentication

```bash
clawhub login            # opens browser flow
clawhub login --token <token>   # token-based login
clawhub logout
clawhub whoami           # check current logged-in user
```

Token is stored in `CLAWHUB_TOKEN` in the server `.env` for reference.
Credentials are saved in the config volume — persist across container rebuilds.

---

## Usage Inside the OpenClaw Docker Container

Skills must be installed into `/home/node/.openclaw/skills/` — this is the persistent config volume and is registered in `openclaw.json` via `skills.load.extraDirs`.

```bash
# Install a skill (persistent + auto-loaded by openclaw)
docker compose exec --user root openclaw-gateway \
  clawhub install <slug> --workdir /home/node/.openclaw --dir skills

# List installed skills
docker compose exec -w /home/node/.openclaw openclaw-gateway clawhub list

# Update all skills
docker compose exec --user root openclaw-gateway \
  clawhub update --all --workdir /home/node/.openclaw --dir skills

# Login (run once — credentials persist in config volume)
docker compose exec --user root openclaw-gateway \
  clawhub login --token <YOUR_CLAWHUB_TOKEN>
```

> **Why `--user root`?** The container runs as `node` by default, which has no write access to the config directory.
> **Why not `/app/skills/`?** That path is inside the image layer — it disappears on container rebuild.

---

## Search & Install (generic)

```bash
clawhub search "postgres backups"
clawhub search "git" --limit 10

clawhub install <slug>
clawhub install <slug> --version 1.2.3
clawhub install <slug> --force     # overwrite existing
```

---

## Update

```bash
clawhub update <slug>
clawhub update --all
clawhub update --all --no-input    # non-interactive (CI-friendly)
```

---

## Publish Your Own Skill

```bash
clawhub publish ./my-skill \
  --slug my-skill \
  --name "My Skill" \
  --version 1.0.0 \
  --changelog "Initial release" \
  --tags "git,automation"

# Batch scan and upload a skills folder
clawhub sync
```

---

## Delete / Restore

```bash
clawhub delete <slug> --yes      # owner or admin only
clawhub undelete <slug> --yes
```

---

## Global Options

| Flag | Description |
|------|-------------|
| `--workdir <dir>` | Override working directory |
| `--dir <dir>` | Skills subdirectory (default: `skills`) |
| `--site <url>` | Override site base URL |
| `--registry <url>` | Override registry API endpoint |
| `--no-input` | Disable interactive prompts |

---

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `CLAWHUB_SITE` | Custom site URL |
| `CLAWHUB_REGISTRY` | Custom registry API endpoint |
| `CLAWHUB_CONFIG_PATH` | Custom config file path |
| `CLAWHUB_WORKDIR` | Default working directory |
| `CLAWHUB_DISABLE_TELEMETRY=1` | Opt out of telemetry |
