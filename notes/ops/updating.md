# Updating OpenClaw

Reference: https://docs.openclaw.ai/install/docker

## Our setup

We run `openclaw:averatec-custom` built on top of `ghcr.io/openclaw/openclaw:latest` (official registry image). The official update paths (npm global, source install) do not apply.

### Update flow

```bash
ssh openclaw "cd ~/openclaw && \
  docker pull ghcr.io/openclaw/openclaw:latest && \
  docker build -f Dockerfile.custom -t openclaw:averatec-custom . && \
  docker compose up -d"
```

1. `docker pull` — fetch latest official image from GitHub Container Registry
2. `docker build -f Dockerfile.custom` — rebuild custom layer (clawhub, gh, goplaces, gog)
3. `docker compose up -d` — recreate container with new image

> No more `git pull + docker build` for the base image. The source repo is not needed for routine updates.

After restart: re-approve device pairing in dashboard (`openclaw devices approve <requestId>`).

### Pre-update checklist

- Check release notes at https://github.com/openclaw/openclaw/releases
- Note any breaking changes (especially cron/delivery behavior)
- Workspace and config are on the host volume — safe across rebuilds

### Post-update verify

```bash
# Check container is running
ssh openclaw "docker ps"

# Check logs for errors
ssh openclaw "docker logs openclaw --tail 50"

# Re-establish SSH tunnel if needed
ssh -f -N -L 18789:127.0.0.1:18789 openclaw
```

### Rollback

Pin a specific official tag in `Dockerfile.custom`:

```dockerfile
FROM ghcr.io/openclaw/openclaw:2026.3.13-1  # pin to known-good version
```

Then rebuild:

```bash
ssh openclaw "cd ~/openclaw && \
  docker build -f Dockerfile.custom -t openclaw:averatec-custom . && \
  docker compose up -d"
```

---

## Official update paths (not used, for reference)

**Global npm install:**
```bash
npm i -g openclaw@latest
```

**Channel switching:**
```bash
openclaw update --channel beta
openclaw update --channel stable
```

**Doctor (safe update + migrate):**
```bash
openclaw doctor
```

**Useful CLI:**
```bash
openclaw gateway status
openclaw gateway restart
openclaw logs --follow
openclaw health
```
