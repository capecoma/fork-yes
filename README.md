# Fork Yes!

A `/fork` command for [Claude Code](https://claude.ai/code) that lets you branch conversations mid-session into a new terminal tab.

Like Gemini Studio's conversation branching, but for Claude Code.

## Platform Support

| Platform | Status |
|----------|--------|
| Windows  | Tested |
| macOS    | **Untested** - should work with Terminal.app, PRs welcome! |
| Linux    | Not yet supported |

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
/fork          # Opens in new tab (default)
/fork window   # Opens in new window
```

That's it. Claude executes two bash commands and opens a new terminal with your forked conversation.

**Why use this?**
- Branch off to explore a different approach without losing your current context
- Keep researching in one session while implementing in another
- Works reliably even with 10+ sessions open in the same project

**How it works:**
1. Generates a unique marker that gets logged to the session file
2. Searches session files to find the exact current session
3. Opens a new terminal tab with the forked session

**Requirements:**

*Windows:*
- Windows Terminal (`wt`)
- PowerShell
- Git Bash

*macOS:*
- Terminal.app (default) or iTerm2
- Bash/Zsh

## Performance Tips

- **Fork early, not late** - Fork when you hit an interesting branch point, not when context is almost full
- Long conversations = slower fork (full history is copied to new session)
- If context is near the limit, Claude compacts/summarizes on load, adding delay
- A short conversation forks in ~2 seconds; a near-full context can take 20+ seconds

## Contributing

Found a bug on Mac? PRs welcome! The macOS support is untested.

## License

MIT
