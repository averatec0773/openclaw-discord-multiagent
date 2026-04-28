---
name: sync
description: Sync changes from this private repo (averatec-openclaw) to the public reference repo (openclaw-discord-multiagent). Run after committing to private when any file in references/file-map.md changed. Direction is always private → public only.
metadata:
  version: 1.0.0
  category: workflow
  dangerous: false
---

# Sync — Private to Public Repo

Direction is always **private → public**. Never edit shared files directly in the public repo.

## When to use

After committing to this repo when any file listed in `references/file-map.md` (Sync Map section) changed. Check the last `pub-sync-*` tag to find what's new.

## Prerequisites

| Repo | Local path | Remote |
|---|---|---|
| Private (source) | `/Users/averatec/CODING/github/averatec-openclaw` | `averatec0773/averatec-openclaw` |
| Public (target) | `/Users/averatec/CODING/github/openclaw-discord-multiagent` | `averatec0773/openclaw-discord-multiagent` |

Both repos must be up to date (`git fetch` in each before starting).

## Steps

### 1 — Find what changed since last sync

```bash
git tag --sort=-creatordate | grep pub-sync | head -1       # e.g. pub-sync-20260401
git diff pub-sync-<date>..HEAD --name-only                  # files changed since then
```

Filter output against the Sync Map in `references/file-map.md`.

### 2 — Apply structural cleanup (if this is a major sync)

If the private repo underwent structural reorganization (new directories, deleted directories), first delete stale paths from the public repo listed in the Structural Cleanup table in `references/file-map.md`:

```bash
cd /Users/averatec/CODING/github/openclaw-discord-multiagent
rm -rf <stale path>
```

Skip this step for routine file-level changes.

### 3 — Copy changed files to public repo

```bash
PRIV="/Users/averatec/CODING/github/averatec-openclaw"
PUB="/Users/averatec/CODING/github/openclaw-discord-multiagent"
cp "$PRIV/<file>" "$PUB/<file>"
```

Copy only files that actually changed. Apply redaction rules from `references/file-map.md` before copying `config/openclaw.json`.

### 4 — Update public-specific files if needed

| If this changed in private... | Also review in public repo |
|---|---|
| File or directory structure | `README.md` paths, `CLAUDE.md` links |
| Skill added or modified | Public `CLAUDE.md` skill list |
| `config/openclaw.json` structure | Public `CLAUDE.md` config section |

Edit public-specific files (README, CLAUDE.md) directly in the public repo — they are intentionally different from private.

### 5 — Commit and push public repo

```bash
cd /Users/averatec/CODING/github/openclaw-discord-multiagent
git add <specific changed files>
git commit -m "<type>: <description>"
git push origin main
```

No `Co-Authored-By` in commits — public repo is single-authored.

### 5 — Tag sync point in private repo

```bash
cd /Users/averatec/CODING/github/averatec-openclaw
git tag pub-sync-$(date +%Y%m%d)
git push origin --tags
```

The tag marks exactly which private commit was last synced. Required for Step 1 of the next run.

## DO NOT

- Do not sync files in the Never Sync list in `references/file-map.md` — they contain personal content.
- Do not bulk-copy with `cp -r` — copy only files that changed.
- Do not push to public before tagging the sync point in private.

## Acceptance Criteria

- [ ] Public repo commit pushed and visible on GitHub
- [ ] `pub-sync-<date>` tag exists in private repo and pushed to remote
- [ ] `git diff pub-sync-<new-tag>..HEAD --name-only` returns only non-syncable files
