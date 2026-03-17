# Docker — OpenClaw Image & Container Management

> Docs: https://docs.openclaw.ai/install/docker

## Custom Image (averatec-custom)

### Image Layers

- `openclaw:latest` — upstream image from Docker Hub
- `openclaw:averatec-custom` — built on top via `Dockerfile.custom`, adds `clawhub`, `gh`, `gog`, `goplaces`

`Dockerfile.custom` is version-controlled in this repo.
Image name is set via `OPENCLAW_IMAGE=openclaw:averatec-custom` in `~/openclaw/.env` (gitignored, safe from upstream overwrites).

### Update Workflow

```bash
# Pull latest upstream, rebuild custom layer, restart
docker pull openclaw:latest
docker compose build
docker compose up -d
```

### Handling Upstream Updates

| File | Overwritten by `git pull` | Strategy |
|---|---|---|
| `.env` | No (gitignored) | Safe — image name lives here |
| `docker-compose.yml` | Yes | Don't modify — control via env vars |
| `Dockerfile.custom` | No (not in upstream) | Re-copy from this repo after merge |

---

## Skill Management (clawhub)

Skills must go into `/home/node/.openclaw/skills/` — persistent config volume, auto-loaded via `extraDirs`.

```bash
# Install a skill
docker compose exec --user root openclaw-gateway \
  clawhub install <slug> --workdir /home/node/.openclaw --dir skills

# List installed skills
docker compose exec -w /home/node/.openclaw openclaw-gateway clawhub list

# Update all skills
docker compose exec --user root openclaw-gateway \
  clawhub update --all --workdir /home/node/.openclaw --dir skills
```

> `--user root` required: default `node` user has no write access to config directory.
> Do **not** install to `/app/skills/` — that path is in the image layer and disappears on rebuild.

---

## Volume Mapping Reference

| Container path | Host path | Persistent |
|---|---|---|
| `/home/node/.openclaw/` | `/root/.openclaw/` | Yes — config, credentials |
| `/home/node/.openclaw/skills/` | `/root/.openclaw/skills/` | Yes — global skills (all agents) |
| `/home/node/.openclaw/workspace/` | `/root/.openclaw/workspace/` | Yes — main agent workspace |
| `/app/skills/` | *(none)* | No — bundled skills, image layer |

> `workspace-public/` (public agent) is not mapped in `docker-compose.yml` by default — it sits inside `/root/.openclaw/` which is already mounted as the top-level config volume.

### Skills loading order (highest → lowest priority)

1. `<workspace>/skills/` — agent-specific, overrides everything
2. `/home/node/.openclaw/skills/` — global managed skills (`clawhub install`)
3. `/app/skills/` — bundled with image (read-only)

---

## Common Commands

```bash
# Rebuild and restart
docker compose build && docker compose up -d

# View logs
docker compose logs -f openclaw-gateway

# Open shell in container
docker compose exec openclaw-gateway bash
docker compose exec --user root openclaw-gateway bash

# Check image list
docker images

# Verify baked binaries
docker compose exec openclaw-gateway which gh
docker compose exec openclaw-gateway which clawhub
```
