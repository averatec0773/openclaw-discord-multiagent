# Installation & Setup

> Reference: https://docs.openclaw.ai/install/hetzner

## What is OpenClaw?

OpenClaw is a self-hosted gateway that connects messaging apps (WhatsApp, Telegram, Discord, iMessage) to AI coding agents. Runs as a single process, bridging chat platforms with an always-available assistant while keeping data under your control.

---

## Hetzner VPS Setup

### Prerequisites

- Hetzner VPS (Ubuntu / Debian) with root SSH access
- SSH client on your laptop
- Model API key (OpenAI and/or Anthropic)
- Optional: Discord bot token, Gmail OAuth, Google Places API key
- ~20 minutes setup time

---

### Step 1 — Provision VPS

```bash
ssh root@YOUR_VPS_IP
```

---

### Step 2 — Install Docker

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

Verify:

```bash
docker --version
docker compose version
```

---

### Step 3 — Clone Repository

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

---

### Step 4 — Create Persistent Directories

```bash
# Single-agent setup
mkdir -p /root/.openclaw/workspace

# Multi-agent setup (add a workspace per additional agent)
mkdir -p /root/.openclaw/workspace
mkdir -p /root/.openclaw/workspace-public

chown -R 1000:1000 /root/.openclaw
```

> The container runs as uid 1000 (`node`). If you create directories as root, set ownership before starting the container or OpenClaw will fail to write workspace files.

---

### Step 5 — Configure Environment Variables

Create `.env` in the repository root:

```bash
# Image
OPENCLAW_IMAGE=openclaw:averatec-custom

# Gateway
OPENCLAW_GATEWAY_TOKEN=<generate: openssl rand -hex 32>
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

# Paths
OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace
XDG_CONFIG_HOME=/home/node/.openclaw

# ClawHub skill registry CLI (get token at clawhub.ai)
CLAWHUB_TOKEN=<your clawhub token>

# Google — Gmail/Calendar via OAuth (see notes/gog.md for headless setup)
GOG_KEYRING_PASSWORD=<any password — encrypts the token file>
GOG_ACCOUNT=<your gmail address>

# Google Places API key (separate from OAuth; get at console.cloud.google.com)
GOOGLE_PLACES_API_KEY=<your google places api key>
```

Generate secure token:

```bash
openssl rand -hex 32
```

> **Note:** All vars in `.env` are automatically injected into the container via `env_file: - .env` in `docker-compose.yml`. No need to duplicate them elsewhere.
>
> **Warning:** Never commit `.env` to version control.

---

### Step 6 — Docker Compose Configuration

Create `docker-compose.yml`:

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build:
      context: .
      dockerfile: Dockerfile.custom   # use custom Dockerfile (default is "Dockerfile")
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
        "--allow-unconfigured",
      ]
```

> Port is bound to loopback (`127.0.0.1`) by default. Remove the prefix to expose publicly — manage firewall and tokens accordingly.

---

### Step 7 — Bake Required Binaries (Critical)

Create `Dockerfile`:

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

> **Critical:** All external binaries required by skills must be baked at image build time. Installing at runtime will be lost on container restart.

#### Using Dockerfile.custom (recommended)

Instead of modifying the base `Dockerfile`, keep your additions separate in `Dockerfile.custom`:

```dockerfile
FROM openclaw:latest

USER root

RUN npm install -g clawhub
RUN apt-get update && apt-get install -y gh && rm -rf /var/lib/apt/lists/*

RUN GOPLACES_URL=$(curl -s https://api.github.com/repos/steipete/goplaces/releases/latest \
  | grep "browser_download_url.*linux_amd64.tar.gz" | cut -d '"' -f 4) \
  && curl -L "$GOPLACES_URL" | tar -xz -C /usr/local/bin \
  && chmod +x /usr/local/bin/goplaces

RUN GOGCLI_URL=$(curl -s https://api.github.com/repos/steipete/gogcli/releases/latest \
  | grep "browser_download_url.*linux_amd64.tar.gz" | cut -d '"' -f 4) \
  && curl -L "$GOGCLI_URL" | tar -xz --no-anchored -C /usr/local/bin gog \
  && chmod +x /usr/local/bin/gog

USER node
```

> The base image runs as the `node` user by default, which has no write access to `/usr/local/lib/node_modules/`.
> Switch to `root` before installing, then switch back to `node` afterward.
>
> The actual `Dockerfile.custom` is version-controlled in this repo root.

Then point `docker-compose.yml` to it:

```yaml
build:
  context: .
  dockerfile: Dockerfile.custom
```

This way, your customizations (extra tools, skills CLI) stay cleanly separated from the base image.

---

### Step 8 — Build and Launch

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Verify binaries:

```bash
docker compose exec openclaw-gateway which gh
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which clawhub
# Expected: /usr/bin/gh, /usr/local/bin/{gog,goplaces}, /usr/local/bin/clawhub
```

---

### Step 9 — Access the Gateway

Check logs:

```bash
docker compose logs -f openclaw-gateway
# Look for: "listening on ws://0.0.0.0:18789"
```

Set up your SSH alias first (recommended — see [notes/ssh.md](../notes/ssh.md)):

```
Host openclaw
    HostName <VPS_IP>
    User root
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
```

Then create the SSH tunnel:

```bash
ssh -N -L 18789:127.0.0.1:18789 openclaw
# or directly: ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Open: `http://127.0.0.1:18789/` (use your `OPENCLAW_GATEWAY_TOKEN` to authenticate)

---

### Step 10 — Post-launch: Authenticate Services

#### GitHub CLI (gh)

```bash
docker compose exec --user root openclaw-gateway gh auth login
```

#### ClawHub

```bash
docker compose exec --user root openclaw-gateway \
  clawhub login --token <YOUR_CLAWHUB_TOKEN>
```

#### Google (gog — Gmail/Calendar)

Headless OAuth requires an SSH tunnel. Full instructions: [notes/gog.md](../notes/gog.md)

Short version:
1. Run `gog auth` on the server host (not inside container)
2. Tunnel the callback port from local machine via SSH
3. Complete OAuth in local browser
4. Copy token files into the container
5. Add `GOG_KEYRING_PASSWORD` and `GOG_ACCOUNT` to `.env`, then `docker compose up -d`

---

### Step 11 — Install Skills (persistent)

```bash
# Install a skill into the persistent config volume
docker compose exec --user root openclaw-gateway \
  clawhub install <slug> --workdir /home/node/.openclaw --dir skills

# Verify
docker compose exec -w /home/node/.openclaw openclaw-gateway clawhub list
```

Skills are auto-loaded via `skills.load.extraDirs` in `openclaw.json`.

---

## Persistence Architecture

| Component | Host Path | Notes |
|-----------|-----------|-------|
| Gateway config | `/root/.openclaw/` | Tokens, configs |
| Auth profiles | `/root/.openclaw/` | OAuth / API keys |
| Global skills | `/root/.openclaw/skills/` | Managed by `clawhub`, shared across all agents |
| Main workspace | `/root/.openclaw/workspace/` | Agent artifacts, AGENTS.md, SOUL.md, etc. |
| Public workspace | `/root/.openclaw/workspace-public/` | Second agent workspace (multi-agent setup) |
| Gmail keyring | `/root/.openclaw/gogcli/keyring/` | Requires `GOG_KEYRING_PASSWORD` |
| External binaries | `/usr/local/bin/` | Baked into Docker image at build time |

---

## Infrastructure-as-Code (Optional)

Community Terraform configs for reproducible Hetzner deployments:

- [openclaw-terraform-hetzner](https://github.com/andreesg/openclaw-terraform-hetzner)
- [openclaw-docker-config](https://github.com/andreesg/openclaw-docker-config)
