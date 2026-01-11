# Fork Yes!

A `/fork` command for [Claude Code](https://claude.ai/code) that lets you branch conversations mid-session into a new terminal tab.

Like Gemini Studio's conversation branching, but for Claude Code.

## Installation

```bash
# Copy to your global Claude commands directory
curl -o ~/.claude/commands/fork.md https://raw.githubusercontent.com/capecoma/fork-yes/master/fork.md
```

Or clone and copy manually:
```bash
git clone https://github.com/capecoma/fork-yes.git
cp fork-yes/fork.md ~/.claude/commands/
```

Restart Claude Code to load the command.

## Usage

```
/fork
```

That's it. Claude executes two bash commands and opens a new terminal tab with your forked conversation.

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

**Performance Tips:**
- **Fork early, not late** - Fork when you hit an interesting branch point, not when context is almost full
- Long conversations = slower fork (full history is copied to new session)
- If context is near the limit, Claude compacts/summarizes on load, adding delay
- A short conversation forks in ~2 seconds; a near-full context can take 20+ seconds
- The `--dangerously-skip-permissions` flag is included for seamless continuation

## Contributing

Found a useful Claude Code workflow? PRs welcome!

## License

MIT
