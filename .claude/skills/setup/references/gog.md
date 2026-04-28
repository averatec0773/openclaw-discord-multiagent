# gog — Google Workspace Re-auth

`gog` is the CLI for Gmail, Calendar, Drive. Installed in `openclaw:averatec-custom`.

Required entries in `~/openclaw/.env`:

```
GOG_KEYRING_PASSWORD=<password>      # encrypts the token file
GOG_ACCOUNT=ayetek0773@gmail.com     # default account
```

Both vars are picked up automatically — no `--account` flag needed.
Token file: `/home/node/.openclaw/gogcli/keyring/` (config volume, persistent).

## Re-auth (if token expires)

OAuth callback requires a browser. Use SSH tunnel for headless server:

```bash
# 1. On server: start auth flow
ssh openclaw
/tmp/gog auth keyring file   # only needed once
GOG_KEYRING_PASSWORD=<password> /tmp/gog auth add ayetek0773@gmail.com
# Note the callback port in output, e.g. 127.0.0.1:40055

# 2. On local machine: tunnel that port
ssh -f -N -L <PORT>:127.0.0.1:<PORT> openclaw

# 3. Complete OAuth in local browser, then run on server:
curl 'http://127.0.0.1:<PORT>/oauth2/callback?...'

# 4. Copy token files into container
docker cp /root/.config/gogcli/. openclaw:/home/node/.openclaw/gogcli/
docker compose exec --user root openclaw-gateway chown -R node:node /home/node/.openclaw/gogcli/

# 5. Verify
docker compose exec openclaw-gateway bash -c \
  'GOG_KEYRING_PASSWORD=<password> gog auth list'
```

> The `gog` binary on server host is at `/tmp/gog`. Permanent one is inside container at `/usr/local/bin/gog`.
