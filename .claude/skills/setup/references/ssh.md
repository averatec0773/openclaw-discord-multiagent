# SSH — Server Access

## Config (~/.ssh/config on local machine)

```
Host openclaw
    HostName <VPS_IP>
    User root
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
```

## Common Commands

```bash
ssh openclaw                                    # shell on server
ssh -f -N -L 18789:127.0.0.1:18789 openclaw   # SSH tunnel (background)
```

## File Transfer

```bash
# Upload to server
scp ./Dockerfile.custom openclaw:~/openclaw/

# Download from server
scp openclaw:/root/openclaw/.env ./
```
