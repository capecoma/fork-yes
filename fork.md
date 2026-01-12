---
allowed-tools: Bash(grep:*,basename:*,pwd:*,sed:*,echo:*,wt:*,ls:*,osascript:*,uname:*)
description: Fork current conversation into new terminal tab
---

# Fork Conversation

Fork the current Claude Code session into a new terminal tab, preserving full conversation context. Works reliably even with multiple sessions open in the same project.

**Supports:** Windows (tested) | macOS (untested - contributions welcome!)

## How It Works

The session file is only written after each tool call completes, so we use a TWO-STEP approach:
1. **Step 1**: Generate a unique marker (gets logged to session file when this command finishes)
2. **Step 2**: Search for the marker to find the exact current session, then launch the fork

## Instructions

**IMPORTANT**: Execute these as TWO SEPARATE bash commands (not one combined command).

### Step 1 - Generate marker (all platforms)
```bash
FORK_MARKER="FORK_$(date +%s)_$$" && echo "FORK_MARKER:$FORK_MARKER"
```

### Step 2 - Find session and fork (run immediately after step 1)

**IMPORTANT**: Replace `MARKER_FROM_STEP1` below with the exact marker value from Step 1's output (e.g., `FORK_1768156595_340898`).

Detect OS and run the appropriate command:

```bash
if [[ "$OSTYPE" == "darwin"* ]]; then
  PROJECT_DIR=$(ls -d "$HOME/.claude/projects/"* 2>/dev/null | grep -i "$(pwd | sed 's|^/|--|' | sed 's|/|-|g')" | head -1) && SESSION_FILE=$(grep -l "MARKER_FROM_STEP1" "$PROJECT_DIR"/*.jsonl 2>/dev/null | grep -v agent- | head -1) && SESSION_ID=$(basename "$SESSION_FILE" .jsonl) && WORK_DIR=$(pwd) && echo "Forking session: $SESSION_ID" && osascript -e "tell application \"Terminal\" to do script \"cd '$WORK_DIR' && claude --resume $SESSION_ID --fork-session --dangerously-skip-permissions\""
else
  PROJECT_DIR=$(ls -d "$HOME/.claude/projects/"* 2>/dev/null | grep -i "$(pwd | sed 's|^/c/|C--|i' | sed 's|/|-|g')" | head -1) && SESSION_FILE=$(grep -l "MARKER_FROM_STEP1" "$PROJECT_DIR"/*.jsonl 2>/dev/null | grep -v agent- | head -1) && SESSION_ID=$(basename "$SESSION_FILE" .jsonl) && WORK_DIR=$(pwd | sed 's|^/c|C:|' | sed 's|/|\\\\|g') && echo "Forking session: $SESSION_ID" && wt new-tab -d "$WORK_DIR" powershell -NoExit -Command "claude --resume $SESSION_ID --fork-session --dangerously-skip-permissions"
fi
```

After successful execution, report: "Forked session $SESSION_ID opened in new terminal tab. Both sessions now have full context but will diverge independently."
