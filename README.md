# Claude Code Skills

Custom slash commands for [Claude Code](https://claude.ai/code) that extend its functionality.

## Installation

Copy the `.md` files from this repo to your Claude Code commands directory:

```bash
# Global (available in all projects)
cp *.md ~/.claude/commands/

# Or project-specific
cp *.md /path/to/your/project/.claude/commands/
```

Restart Claude Code to load the new commands.

## Available Skills

### `/fork` - Fork Current Conversation

Fork your current Claude Code session into a new terminal tab, preserving full conversation context. Both sessions diverge independently from that point.

**Why use this?**
- Branch off to explore a different approach without losing your current context
- Keep researching in one session while implementing in another
- Similar to Gemini Studio's conversation branching feature

**How it works:**
1. Generates a unique marker that gets logged to the session file
2. Searches session files to find the exact current session (works even with 10+ sessions open)
3. Opens a new Windows Terminal tab with the forked session

**Usage:**
```
/fork
```

Claude will execute two bash commands automatically - the fork opens in a new terminal tab.

**Requirements:**
- Windows Terminal (`wt`)
- PowerShell
- Git Bash (for the bash commands)

**Notes:**
- Forking a long conversation takes longer to load (the full history is copied)
- If context is near the limit, Claude may compact/summarize older messages on load
- The `--dangerously-skip-permissions` flag is included for seamless continuation

## Contributing

Found a useful Claude Code workflow? PRs welcome!

## License

MIT
