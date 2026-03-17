---
name: edit
description: Use before any operation that writes to server files or repo files. Governs modifications to both sides — requires reading current state, assessing cross-side impact, and planning changes explicitly before executing. Prevents blind overwrites in either direction.
user-invocable: true
---

# Edit — Modification Planning

This skill governs all write operations that touch server files or repo files.
It ensures changes are planned, cross-impact is assessed, and nothing is overwritten without understanding what currently exists.

---

## When to invoke

- Before modifying any server workspace file (AGENTS.md, SOUL.md, TOOLS.md, openclaw.json, etc.)
- Before modifying any repo file that has a server equivalent (templates/, config/)
- Before pushing local changes that may conflict with server state
- Before pulling server state that may conflict with local repo

---

## Core Rules

1. Never overwrite a file on either side without first reading its current content.
2. Before writing to server: read the server file, compare with repo, identify what would be lost.
3. Before writing to repo: check if the change should also be reflected on the server.
4. State the plan explicitly before executing any write.
5. If content would be lost in either direction, stop and ask the user.

---

## Workflow

### Step 1 — Read current state

Read the current version of the target file from whichever side you are about to modify.

For server files:
```bash
ssh openclaw "cat <file-path>"
```

For repo files: use the Read tool.

If a mapping exists (see sync skill's file-map.md), also read the counterpart on the other side.

### Step 2 — Assess cross-side impact

For each file being changed, answer:

| Question | If yes |
|---|---|
| Does this server file have a repo equivalent? | Note that repo will be out of date after the edit |
| Does this repo file have a server equivalent? | Note that server will be out of date after the edit |
| Does either side have content the other doesn't? | Identify what would be overwritten or lost |
| Would a sync be needed after this change? | Plan to run sync after |

### Step 3 — Plan explicitly

State the plan before writing:
- Which file(s) will be changed
- What will be added, modified, or removed
- What will be preserved
- Whether a sync will be needed afterward
- Any content at risk of being lost

If there is server-only content not in the repo (or vice versa) that the planned change would destroy, stop and present the conflict to the user.

### Step 4 — Execute with minimal footprint

**For server file edits** — make targeted in-place edits, not full overwrites:

```bash
# Text file: targeted replacement via python
ssh openclaw "python3 << 'EOF'
content = open('/path/to/file').read()
content = content.replace('old section', 'new section')
open('/path/to/file', 'w').write(content)
EOF"

# Append a section
ssh openclaw "python3 -c \"
open('/path/to/file', 'a').write('\n## New Section\n...\n')
\""

# JSON: modify specific keys only
ssh openclaw "python3 << 'EOF'
import json
with open('/root/.openclaw/openclaw.json') as f:
    c = json.load(f)
c['key'] = 'value'   # only the key that needs changing
with open('/root/.openclaw/openclaw.json', 'w') as f:
    json.dump(c, f, indent=2)
    f.write('\n')
EOF"
```

Do not use:
- `scp local -> server` for workspace files (full overwrite without reading)
- `cat > remote_file` style full rewrites unless content has been read and preserved

**For repo file edits** — use Edit tool for targeted changes, not full Write unless the file is being created fresh.

### Step 5 — Verify

Read the modified file back to confirm the result is correct.

### Step 6 — Sync if needed

If the change creates a divergence between server and repo, run the sync skill to resolve it.

---

## File mapping reference

See `.claude/skills/sync/file-map.md` for the complete list of files with server-to-repo mappings.
