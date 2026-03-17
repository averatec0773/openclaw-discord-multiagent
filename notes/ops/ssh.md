# SSH — OpenClaw Server Access

## SSH Alias

`~/.ssh/config` on local machine:

```
Host openclaw
    HostName <VPS_IP>
    User root
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60

Host tunnel-openclaw
    HostName <VPS_IP>
    User root
    LocalForward 18789 127.0.0.1:18789
    ServerAliveInterval 60
```

Usage:
```bash
ssh openclaw                  # open shell on server
ssh -N -L 18789:127.0.0.1:18789 openclaw  # tunnel only (access UI at http://127.0.0.1:18789/)
```

## File Transfer

```bash
# Download a file from server
scp openclaw:/root/openclaw/.env ./

# Upload a file to server
scp ./Dockerfile.custom openclaw:/root/openclaw/
```
