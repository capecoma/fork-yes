---
allowed-tools: Bash(grep:*,basename:*,pwd:*,sed:*,echo:*,wt:*,ls:*,osascript:*,uname:*,start:*)
description: Fork current conversation into new terminal tab
---

# Fork Conversation

Fork the current Claude Code session into a new terminal tab, preserving full conversation context. Works reliably even with multiple sessions open in the same project.

**Supports:** Windows (tested) | macOS (untested - PRs welcome!)

**Smart behavior:**
- Windows Terminal → opens new tab in same window
- Standalone PowerShell → opens new window (tabs not supported)
- macOS iTerm2 → opens new tab in same window
- macOS Terminal.app → opens new tab in same window (requires Accessibility permissions)

**macOS Note:** Terminal.app tab creation uses keyboard simulation which requires granting Accessibility permissions to Terminal in System Preferences → Privacy & Security → Accessibility.

## How It Works

The session file is only written after each tool call completes, so we use a TWO-STEP approach:
1. **Step 1**: Generate a unique marker (gets logged to session file when this command finishes)
2. **Step 2**: Search for the marker to find the exact current session, then launch the fork

## Instructions

**IMPORTANT**: Execute these as TWO SEPARATE bash commands (not one combined command).

### Step 1 - Generate marker (all platforms)
```bash
FORK_MARKER="FORK_$(date +%s)_$$" && echo "Fork yes! Tagging this session..." && echo "FORK_MARKER:$FORK_MARKER"
```

### Step 2 - Find session and fork (run immediately after step 1)

**IMPORTANT**: Replace `MARKER_FROM_STEP1` below with the exact marker value from Step 1's output (e.g., `FORK_1768156595_340898`).

Detect OS and terminal type, then run the appropriate command:

```bash
if [ "$TERM_PROGRAM" = "iTerm.app" ]; then
  # macOS iTerm2 - open new tab (native AppleScript support)
  PROJECT_DIR=$(ls -d "$HOME/.claude/projects/"* 2>/dev/null | grep -i "$(pwd | sed 's|/|-|g')" | head -1) && SESSION_FILE=$(grep -l "MARKER_FROM_STEP1" "$PROJECT_DIR"/*.jsonl 2>/dev/null | grep -v agent- | head -1) && SESSION_ID=$(basename "$SESSION_FILE" .jsonl) && WORK_DIR=$(pwd) && echo "Fork yes! Launching session $SESSION_ID in new tab..." && osascript -e 'tell application "iTerm"' -e 'tell current window' -e 'create tab with default profile' -e 'tell current session' -e "write text \"cd '$WORK_DIR' && claude --resume $SESSION_ID --fork-session --dangerously-skip-permissions\"" -e 'end tell' -e 'end tell' -e 'end tell'
elif [ "$TERM_PROGRAM" = "Apple_Terminal" ]; then
  # macOS Terminal.app - open new tab (keyboard simulation, requires Accessibility permissions)
  PROJECT_DIR=$(ls -d "$HOME/.claude/projects/"* 2>/dev/null | grep -i "$(pwd | sed 's|/|-|g')" | head -1) && SESSION_FILE=$(grep -l "MARKER_FROM_STEP1" "$PROJECT_DIR"/*.jsonl 2>/dev/null | grep -v agent- | head -1) && SESSION_ID=$(basename "$SESSION_FILE" .jsonl) && WORK_DIR=$(pwd) && echo "Fork yes! Launching session $SESSION_ID in new tab..." && osascript -e 'tell application "Terminal" to activate' -e 'tell application "System Events" to keystroke "t" using command down' -e 'delay 0.5' -e "tell application \"Terminal\" to do script \"cd '$WORK_DIR' && claude --resume $SESSION_ID --fork-session --dangerously-skip-permissions\" in front window"
elif [ -n "$WT_SESSION" ]; then
  # Windows Terminal - open new tab in same window
  PROJECT_DIR=$(ls -d "$HOME/.claude/projects/"* 2>/dev/null | grep -i "$(pwd | sed 's|^/c/|C--|i' | sed 's|/|-|g')" | head -1) && SESSION_FILE=$(grep -l "MARKER_FROM_STEP1" "$PROJECT_DIR"/*.jsonl 2>/dev/null | grep -v agent- | head -1) && SESSION_ID=$(basename "$SESSION_FILE" .jsonl) && WORK_DIR=$(pwd | sed 's|^/c|C:|' | sed 's|/|\\\\|g') && echo "Fork yes! Launching session $SESSION_ID in new tab..." && wt -w 0 new-tab -d "$WORK_DIR" powershell -NoExit -Command "claude --resume $SESSION_ID --fork-session --dangerously-skip-permissions"
else
  # Standalone PowerShell - open new window
  PROJECT_DIR=$(ls -d "$HOME/.claude/projects/"* 2>/dev/null | grep -i "$(pwd | sed 's|^/c/|C--|i' | sed 's|/|-|g')" | head -1) && SESSION_FILE=$(grep -l "MARKER_FROM_STEP1" "$PROJECT_DIR"/*.jsonl 2>/dev/null | grep -v agent- | head -1) && SESSION_ID=$(basename "$SESSION_FILE" .jsonl) && WORK_DIR=$(pwd | sed 's|^/c|C:|' | sed 's|/|\\\\|g') && echo "Fork yes! Launching session $SESSION_ID in new window..." && start powershell -NoExit -WorkingDirectory "$WORK_DIR" -Command "claude --resume $SESSION_ID --fork-session --dangerously-skip-permissions"
fi
```

After successful execution, report: "Fork yes! Your conversation has been forked. Both sessions have full context and will now diverge independently."
