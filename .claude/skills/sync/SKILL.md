---
name: sync
description: Use when updating the averatec-openclaw repo after server changes, or syncing server workspace files back to repo. Also invoke proactively before any operation that will write to repo files OR server workspace files. Triggered by "update repo", "sync docs", "同步", "更新 repo".
user-invocable: true
---

# averatec-openclaw Repo Sync

Consult [file-map.md](file-map.md) for the complete server-to-repo mapping table and redaction rules.

---

## Proactive Gate Rule

Run this skill before any operation that will write to repo files or server workspace files.
The conflict check must complete before any write proceeds.

---

## Step 0 — Conflict Detection

For each file pair relevant to the current operation, read both sides and compare:

```bash
ssh openclaw "cat <server-path>"   # server version
# compare with local repo file
```

| Result | Action |
|---|---|
| Only server changed | Proceed: server -> repo |
| Only repo changed | Describe changes, ask user whether to push to server |
| Both sides changed | STOP — show conflict summary and ask user to choose |
| Both identical | Skip |

When both sides have changed, present the following and wait for user input before proceeding:

```
CONFLICT: <filename>

Server: <what changed>
Repo:   <what changed>

Choose:
  A  Update repo from server (server -> repo)
  B  Update server from repo (confirm no server content will be lost first)
  C  Skip this file
  D  Show full diff
```

Do not overwrite either side without explicit user confirmation.

---

## Step 1 — Identify What Changed

Categorize affected files based on Step 0:

| Changed | Action |
|---|---|
| `workspace/AGENTS.md` | Update `templates/AGENTS.md` |
| `workspace/SOUL.md` | Update `templates/SOUL.md` |
| `workspace-public/AGENTS.md` | Update `templates/AGENTS.public.md` |
| `openclaw.json` | Update `config/openclaw.json` with redaction applied |
| New skill in `skills/` | Update `notes/workspace/installed-skills.md` and `CONTEXT.md` |
| Discord config changed | Update `notes/channels/discord.md` and `CONTEXT.md` Active Channels |
| Local doc-only edit | Skip server step, go straight to commit |

If unsure which files changed, ask — do not update everything blindly.

---

## Step 2 — Update Repo Files

Update only the files identified in Step 1.

For `config/openclaw.json`: apply redaction rules from [file-map.md](file-map.md) before writing.

For skill tables (`notes/workspace/installed-skills.md`, `CONTEXT.md`): both files contain the same skill table and must stay in sync.

---

## Step 3 — Update Cross-References Before Committing

Before staging any commit, check whether the changes require updates to other files.

**For every file being changed, verify:**

| If you changed... | Also check and update if needed |
|---|---|
| Any file path or directory structure | `README.md` directory tree, `CONTEXT.md` links, `session/SKILL.md` file list |
| A file listed in `CONTEXT.md` | The specific section that references it |
| `notes/workspace/installed-skills.md` skill list | `CONTEXT.md` Installed Skills table |
| `notes/channels/discord.md` | `CONTEXT.md` Active Channels section |
| `config/openclaw.json` key settings | `CONTEXT.md` OpenClaw Configuration section |
| A `.claude/skills/*/SKILL.md` | `session/SKILL.md` skill list |
| `templates/AGENTS.md` or `SOUL.md` | No cross-reference needed (templates are standalone) |

Update all affected cross-references **before** staging the commit so everything lands in a single coherent commit.

---

## Step 4 — Commit and Push

Stage only the files actually changed (including cross-reference updates from Step 3):

```bash
git add <specific files>
git commit -m "<type>: <clear description of what changed and why>"
git push origin main
```

**Commit message rules:**
- Use conventional commit prefixes: `docs:`, `feat:`, `fix:`, `refactor:`
- Describe **what changed**, not just which files were touched
- If multiple files were updated for the same reason, name the reason — e.g., `docs: restructure notes/ into ops/workspace/channels/services subdirectories`
- If cross-reference files were updated alongside a main change, include both in the description — e.g., `docs: update AGENTS.md startup sequence; sync README and CONTEXT references`

---

## What Never Goes into the Repo

See [file-map.md](file-map.md) — Never sync section.
