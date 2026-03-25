---
name: dream
description: Memory consolidation command — a reflective pass over your memory files. Merges duplicates, removes stale entries, fixes metadata, and keeps MEMORY.md index tight. Inspired by Anthropic's unreleased Auto-dream feature. Use when user says "dream", "consolidate memory", "clean memory", "optimize memory".
---

# Dream — Memory Consolidation

You are performing a **dream** — a reflective pass over your memory files.

**Your mission**: ensure a fresh session with zero prior context can orient itself within the first 200 lines of the index file.

**Rules:**
- **Merge what's fragmented** — combine duplicate entries about the same topic into one canonical file
- **Fix what's drifted** — convert relative dates to absolute, correct outdated facts against current state
- **Prune what's stale** — mark superseded entries, don't delete them
- **Verify before claiming** — read files before writing counts or references about them
- **Log what you changed** — end with a changelog so the human can review in 10 seconds

Be conservative: when uncertain whether two entries overlap, keep both. **Information loss is worse than slight redundancy.**

Never modify code, configs, or non-memory files.

## Memory Locations

This skill works with Claude Code's built-in auto-memory system:

| Location | Purpose |
|----------|---------|
| `MEMORY.md` | Auto-memory index (keep ≤ 200 lines) |
| Memory topic files | Detailed memories linked from MEMORY.md |

> **Adapt these paths** to your own memory system. If you use custom memory directories (e.g., `memory/learnings/`, `memory/logs/`), update the paths in Phase 1 below.

## Safety Rules

- **Read-only** on code files — NEVER modify source code, skills, or configs
- **Write-only** on memory files — only touch MEMORY.md and memory topic files
- **Don't delete** — mark old entries as superseded, don't remove files
- **Ask before bulk changes** — if consolidation affects 10+ files, list them and ask for approval first
- **Preserve original dates** — never change `created` dates in frontmatter

## Phase 1 — Orient

1. Find your memory files:
   - Read `MEMORY.md` (the auto-memory index)
   - List any memory topic files linked from it
   - Check for additional memory directories if they exist

2. Understand current state:
   - How many memory files exist?
   - When was MEMORY.md last updated?
   - Any obvious problems? (stale entries, broken links)

**Output**: Brief inventory report — file counts, last updated dates, issues spotted

## Phase 2 — Gather Signal

Look for problems to fix. Priority order:

### 2a. Find Duplicates
Search for memories about the same topic saved multiple times:
- Same concept explained in different files
- Similar titles with slightly different wording
- Entries from different sessions about the same discovery

### 2b. Find Stale Entries
- Memories with relative dates ("yesterday", "next session", "TODO")
- Entries about tools/configs that have changed since creation
- TODOs that are now done

### 2c. Find Missing Metadata
- Files without proper frontmatter
- Entries missing tags or dates
- Truncated or vague titles

### 2d. Check for Gaps
- Important patterns that appear repeatedly but aren't captured
- Recurring lessons that should become permanent principles

**Output**: Problem list with specific file names and recommended actions

## Phase 3 — Consolidate

For each issue found, apply the appropriate fix:

### 3a. Merge Duplicates
1. Identify the **canonical version** (most complete, best formatted)
2. Copy any unique information from other versions into canonical
3. Add `superseded_by: [[canonical_filename]]` to the duplicate's frontmatter
4. Add `status: superseded` to the duplicate's frontmatter
5. Do NOT delete the duplicate file

### 3b. Fix Metadata
Add/fix YAML frontmatter:
```yaml
---
title: Clear, Searchable Title (< 80 chars)
created: YYYY-MM-DD
tags: [topic1, topic2]
source: session | manual | dream-consolidation
---
```

### 3c. Convert Relative Dates
- "yesterday" → actual date based on file creation date
- "next session" → check if done, update or mark as stale
- "TODO" → check if completed, update status

### 3d. Resolve Contradictions
If two memories contradict each other:
1. Determine which is correct (check current state)
2. Update the wrong one with correction note
3. Link related files together

**Output**: List of changes made with before/after for each

## Phase 4 — Prune and Index

Update `MEMORY.md` to stay under 200 lines:

1. **Remove stale pointers** — links to superseded or outdated memories
2. **Add new pointers** — link to important new/updated memories
3. **Keep it an INDEX** — one-line descriptions only, details in topic files
4. **Add timestamp**: `# Last consolidated: YYYY-MM-DD`
5. **Organize by topic** — group related memories under headers

### MEMORY.md Structure
```markdown
# Auto Memory
# Last consolidated: YYYY-MM-DD

## [Topic Group]
- [memory-file.md](path) — one-line description

## [Another Topic]
- ...
```

**Output**: Updated MEMORY.md content

## Final Report

After all phases, provide a summary:

```
## Dream Complete

### Stats
- Files scanned: X
- Duplicates merged: X
- Metadata fixed: X
- Stale entries updated: X
- MEMORY.md lines: X/200

### Key Changes
1. [change 1]
2. [change 2]

### Next Dream Recommendations
- [what to watch for next time]
```

## Modes

### `/dream` (default) — Full consolidation
Run all 4 phases. Best for periodic maintenance.

### `/dream --scan` — Read-only audit
Run Phase 1 + 2 only. Report problems without fixing them.
Good for checking if a dream is needed.

### `/dream --quick` — Light pass
Skip Phase 2b/2c/2d. Only merge obvious duplicates and update MEMORY.md.
Good for quick maintenance between sessions.

## When to Run

- Every 5+ sessions
- Before major project milestones
- After long debug sessions (lots of learnings generated)
- When memory search returns confusing/duplicate results
- Anytime you say "dream", "consolidate memory", or "clean memory"

## Optional: Nightly Cron

To auto-run every night, create a script:

```bash
#!/bin/bash
# nightly-dream.sh
LOGDIR="/tmp/claude-dream"
mkdir -p "$LOGDIR"
LOGFILE="$LOGDIR/dream-$(date +%Y-%m-%d).log"

cd /path/to/your/project
claude --print --model sonnet --max-turns 30 \
  "Run /dream --quick to consolidate memory files." \
  >> "$LOGFILE" 2>&1

# Keep only last 7 days of logs
find "$LOGDIR" -name "dream-*.log" -mtime +7 -delete 2>/dev/null
```

Then add to crontab: `0 0 * * * /path/to/nightly-dream.sh`
