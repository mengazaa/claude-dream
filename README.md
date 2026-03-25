# /dream — Memory Consolidation for Claude Code

A Claude Code skill that consolidates your project memory files — like how your brain consolidates memories during REM sleep.

Inspired by Anthropic's unreleased [Auto-dream feature](https://github.com/Piebald-AI/claude-code-system-prompts/blob/main/system-prompts/agent-prompt-dream-memory-consolidation.md), adapted for any Claude Code project.

## What it does

Claude Code remembers things about your project in `MEMORY.md` and memory topic files. Over time, these accumulate duplicates, stale entries, and messy metadata. `/dream` cleans it all up in 4 phases:

1. **Orient** — Scan existing memory files, understand current state
2. **Gather Signal** — Find duplicates, stale entries, missing metadata
3. **Consolidate** — Merge duplicates, fix metadata, resolve contradictions
4. **Prune & Index** — Keep MEMORY.md under 200 lines, organized by topic

## Install

### Option 1: Copy files manually

```bash
# Create skill directory
mkdir -p ~/.claude/skills/dream

# Copy the skill file
cp SKILL.md ~/.claude/skills/dream/SKILL.md

# Copy the slash command
cp dream.md ~/.claude/commands/dream.md
```

### Option 2: One-liner

```bash
mkdir -p ~/.claude/skills/dream ~/.claude/commands && curl -sL https://raw.githubusercontent.com/mengazaa/claude-dream/main/SKILL.md -o ~/.claude/skills/dream/SKILL.md && curl -sL https://raw.githubusercontent.com/mengazaa/claude-dream/main/dream.md -o ~/.claude/commands/dream.md
```

## Usage

In Claude Code, type:

```
/dream          # Full 4-phase consolidation
/dream --scan   # Read-only audit (see problems without fixing)
/dream --quick  # Light pass (just merge duplicates + update index)
```

## Optional: Nightly Auto-Dream

Set up a cron job to run `/dream --quick` every night:

```bash
# Create the script
cat > ~/.claude/scripts/nightly-dream.sh << 'EOF'
#!/bin/bash
LOGDIR="/tmp/claude-dream"
mkdir -p "$LOGDIR"
LOGFILE="$LOGDIR/dream-$(date +%Y-%m-%d).log"

cd /path/to/your/project  # <-- Change this!
claude --print --model sonnet --max-turns 30 \
  "Run /dream --quick to consolidate memory files." \
  >> "$LOGFILE" 2>&1

find "$LOGDIR" -name "dream-*.log" -mtime +7 -delete 2>/dev/null
EOF

chmod +x ~/.claude/scripts/nightly-dream.sh

# Add to crontab (runs at midnight)
(crontab -l 2>/dev/null; echo "0 0 * * * ~/.claude/scripts/nightly-dream.sh") | crontab -
```

## How it works

The core prompt tells Claude:

> "You are performing a dream — a reflective pass over your memory files. Synthesize what you've learned recently into durable, well-organized memories so that future sessions can orient quickly."

This maps directly to how human REM sleep consolidates short-term memories into long-term storage:

| Human Sleep | /dream |
|------------|--------|
| Scan recent memories | Phase 1: Orient |
| Identify important signals | Phase 2: Gather Signal |
| Consolidate into long-term | Phase 3: Consolidate |
| Prune unnecessary details | Phase 4: Prune & Index |

## Safety

- **Read-only** on your code — never modifies source files
- **Write-only** on memory files — only touches MEMORY.md and memory topic files
- **Never deletes** — marks duplicates as "superseded" instead
- **Asks first** — if consolidation affects 10+ files, asks for approval

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- A project with Claude Code memory enabled (MEMORY.md exists)

## Credits

- Inspired by [Anthropic's Auto-dream system prompt](https://github.com/Piebald-AI/claude-code-system-prompts/blob/main/system-prompts/agent-prompt-dream-memory-consolidation.md)
- Built by [@mengazaa](https://github.com/mengazaa)

## License

MIT
