---
allowed-tools: Bash(grep:*,basename:*,pwd:*,sed:*,echo:*,wt:*,ls:*,osascript:*,uname:*,start:*)
description: Fork current conversation into new terminal tab or window
argument-hint: "[window]"
---

# Fork Conversation

Fork the current Claude Code session into a new terminal, preserving full conversation context. Works reliably even with multiple sessions open in the same project.

**Usage:**
- `/fork` - Opens in new tab (default)
- `/fork window` - Opens in new window

**Supports:** Windows (tested) | macOS (untested - contributions welcome!)

## How It Works

The session file is only written after each tool call completes, so we use a TWO-STEP approach:
1. **Step 1**: Generate a unique marker (gets logged to session file when this command finishes)
2. **Step 2**: Search for the marker to find the exact current session, then launch the fork

## Instructions

**IMPORTANT**: Execute these as TWO SEPARATE bash commands (not one combined command).

Check if user passed "window" argument: $ARGUMENTS

### Step 1 - Generate marker (all platforms)
```bash
FORK_MARKER="FORK_$(date +%s)_$$" && echo "FORK_MARKER:$FORK_MARKER"
```

### Step 2 - Find session and fork (run immediately after step 1)

Detect OS and check for "window" argument. If argument is "window", open new window; otherwise open new tab.

**If argument is "window":**

```bash
if [[ "$OSTYPE" == "darwin"* ]]; then
  PROJECT_DIR=$(ls -d "$HOME/.claude/projects/"* 2>/dev/null | grep -i "$(pwd | sed 's|^/|--|' | sed 's|/|-|g')" | head -1) && SESSION_FILE=$(grep -l "FORK_MARKER:FORK_" "$PROJECT_DIR"/*.jsonl 2>/dev/null | grep -v agent- | head -1) && SESSION_ID=$(basename "$SESSION_FILE" .jsonl) && WORK_DIR=$(pwd) && echo "Forking session: $SESSION_ID (new window)" && osascript -e "tell application \"Terminal\" to do script \"cd '$WORK_DIR' && claude --resume $SESSION_ID --fork-session --dangerously-skip-permissions\""
else
  PROJECT_DIR=$(ls -d "$HOME/.claude/projects/"* 2>/dev/null | grep -i "$(pwd | sed 's|^/c/|C--|i' | sed 's|/|-|g')" | head -1) && SESSION_FILE=$(grep -l "FORK_MARKER:FORK_" "$PROJECT_DIR"/*.jsonl 2>/dev/null | grep -v agent- | head -1) && SESSION_ID=$(basename "$SESSION_FILE" .jsonl) && WORK_DIR=$(pwd | sed 's|^/c|C:|' | sed 's|/|\\\\|g') && echo "Forking session: $SESSION_ID (new window)" && wt new-window -d "$WORK_DIR" powershell -NoExit -Command "claude --resume $SESSION_ID --fork-session --dangerously-skip-permissions"
fi
```

**If no argument (default - new tab):**

```bash
if [[ "$OSTYPE" == "darwin"* ]]; then
  PROJECT_DIR=$(ls -d "$HOME/.claude/projects/"* 2>/dev/null | grep -i "$(pwd | sed 's|^/|--|' | sed 's|/|-|g')" | head -1) && SESSION_FILE=$(grep -l "FORK_MARKER:FORK_" "$PROJECT_DIR"/*.jsonl 2>/dev/null | grep -v agent- | head -1) && SESSION_ID=$(basename "$SESSION_FILE" .jsonl) && WORK_DIR=$(pwd) && echo "Forking session: $SESSION_ID (new tab)" && osascript -e "tell application \"Terminal\" to activate" -e "tell application \"System Events\" to keystroke \"t\" using command down" -e "delay 0.5" -e "tell application \"Terminal\" to do script \"cd '$WORK_DIR' && claude --resume $SESSION_ID --fork-session --dangerously-skip-permissions\" in front window"
else
  PROJECT_DIR=$(ls -d "$HOME/.claude/projects/"* 2>/dev/null | grep -i "$(pwd | sed 's|^/c/|C--|i' | sed 's|/|-|g')" | head -1) && SESSION_FILE=$(grep -l "FORK_MARKER:FORK_" "$PROJECT_DIR"/*.jsonl 2>/dev/null | grep -v agent- | head -1) && SESSION_ID=$(basename "$SESSION_FILE" .jsonl) && WORK_DIR=$(pwd | sed 's|^/c|C:|' | sed 's|/|\\\\|g') && echo "Forking session: $SESSION_ID (new tab)" && wt new-tab -d "$WORK_DIR" powershell -NoExit -Command "claude --resume $SESSION_ID --fork-session --dangerously-skip-permissions"
fi
```

After successful execution, report: "Forked session $SESSION_ID opened. Both sessions now have full context but will diverge independently."
