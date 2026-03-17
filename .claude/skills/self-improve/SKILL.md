---
name: self-improve
description: Proactive self-improvement skill. Invoked after significant operations to record learnings and update skills. Also invoked periodically or on request to analyze repo and server structure and propose improvements to the user.
user-invocable: true
---

# Self-Improve

This skill has two modes: **learning** (record knowledge from what just happened) and **analysis** (review structure and suggest improvements).

---

## Mode 1 — Record Learning

Invoke after completing a significant operation, making a mistake, or discovering something non-obvious.

### When to trigger

- After a mistake was made and corrected
- After discovering an undocumented behavior or edge case
- After a workflow that revealed a gap in the current skills
- After completing a complex multi-step operation for the first time

### What to record

Write to your Claude Code project memory directory for this repo. Create topic files as needed:

| Type | File | Examples |
|---|---|---|
| General patterns | `MEMORY.md` (concise, top 200 lines) | "Always read server file before SCP" |
| Topic-specific | `<topic>.md` | `server-ops.md`, `git-workflow.md` |

**Skill improvement** — if a skill has a gap that caused an issue, propose updating it:
1. Identify which skill was missing or unclear
2. Draft the addition
3. Present to user: "Suggest updating `<skill>` to add: `<change>`. Proceed?"
4. Only execute after user confirms

### Format for memory entries

```
## <topic>
<what was learned, in one or two sentences>
Source: <what operation revealed this>
```

---

## Mode 2 — Structural Analysis

Invoke on request (`/self-improve`) or after a session with significant changes.

### Step 1 — Read current state

Read all relevant files before forming any opinion:
- All files in `.claude/skills/`
- `CONTEXT.md`, `README.md`
- All files in `notes/`, `templates/`, `config/`, `installation/`

### Step 2 — Evaluate repo structure

| Question | Red flag |
|---|---|
| Does each directory have a clear, single purpose? | Mixed content in one directory |
| Is there content duplication between files? | Same info in multiple places |
| Are file locations consistent with their content type? | Docs in wrong dirs |
| Is any single file doing too many jobs? | File too long or covers unrelated topics |
| Are references in CONTEXT.md and README.md up to date? | Stale paths or descriptions |

### Step 3 — Evaluate server structure (optional)

```bash
ssh openclaw "ls /root/.openclaw/workspace/ && ls /root/.openclaw/skills/"
```

### Step 4 — Present findings

```
IMPROVEMENT SUGGESTIONS — <date>

HIGH (causes confusion or errors):
  1. <finding> → <proposed change>

MEDIUM (worth doing, no urgency):
  2. <finding> → <proposed change>

LOW (polish):
  3. <finding> → <proposed change>

For each item, choose:
  Y  Execute now
  N  Skip
  D  Defer (add to notes/todo.md or equivalent)
```

Present ALL findings before executing ANY changes.

### Step 5 — Execute approved changes

For each approved item:
1. Invoke the `edit` skill to plan the change
2. Execute with minimal footprint
3. Run `sync` skill if the change affects files with server equivalents

---

## Mode 3 — Skill Self-Update

When an existing skill is missing coverage or has a rule that caused an issue:

1. Read the current skill file
2. Draft the proposed addition or change
3. Present: "Suggest updating `.claude/skills/<name>/SKILL.md`: `<what and why>`. Approve?"
4. On approval: edit the skill file, commit
5. Also update `session/SKILL.md` skill list if a new skill was added

---

## Notes

- Never record secrets, API keys, or personal data in memory files
- Keep `MEMORY.md` under 200 lines — move detailed notes to topic files
- Suggestions must be based on actual observations, not speculation
- Always ask before executing structural changes
