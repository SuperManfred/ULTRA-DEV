# Phase 1 Recipe: Worktree Isolation via Hook

This is the exact recipe to reproduce Phase 1. Follow these steps precisely.

## What This Proves

A Claude session started with `WORKTREE_ROOT` set will be blocked from reading or writing files outside that path.

## Prerequisites

- Claude Code installed and working
- A git repository to create worktrees from

## Step 1: Create the Hook Script

Create file `/Users/MN/GITHUB/ULTRA-DEV/hooks/enforce-worktree` with this exact content:

```bash
#!/bin/bash
# enforce-worktree: PreToolUse hook for worktree isolation
#
# If WORKTREE_ROOT is set, blocks file operations outside that path.
# If WORKTREE_ROOT is not set, approves everything (normal session).

set -euo pipefail

input=$(cat)
tool=$(echo "$input" | jq -r '.tool_name')

# If no WORKTREE_ROOT set, this is a normal session - approve everything
if [[ -z "${WORKTREE_ROOT:-}" ]]; then
  echo '{"decision": "approve"}'
  exit 0
fi

# Tools that operate on file paths
case "$tool" in
  Edit|Write|Read)
    target_path=$(echo "$input" | jq -r '.tool_input.file_path // empty')
    ;;
  Bash)
    # For Bash, approve for now - git operations are handled separately
    echo '{"decision": "approve"}'
    exit 0
    ;;
  *)
    # Unknown tool - approve
    echo '{"decision": "approve"}'
    exit 0
    ;;
esac

# If we have a target path, check it
if [[ -n "$target_path" ]]; then
  # Resolve to absolute path
  if [[ "$target_path" != /* ]]; then
    cwd=$(echo "$input" | jq -r '.cwd // empty')
    if [[ -n "$cwd" ]]; then
      target_path="$cwd/$target_path"
    fi
  fi

  # Normalize path
  resolved_path=$(realpath -m "$target_path" 2>/dev/null || echo "$target_path")
  resolved_worktree=$(realpath -m "$WORKTREE_ROOT" 2>/dev/null || echo "$WORKTREE_ROOT")

  # Check if target is within worktree
  if [[ "$resolved_path" == "$resolved_worktree"* ]]; then
    echo '{"decision": "approve"}'
  else
    echo "{\"decision\": \"block\", \"reason\": \"Path $target_path is outside worktree boundary $WORKTREE_ROOT\"}"
  fi
else
  echo '{"decision": "approve"}'
fi
```

Make it executable:
```bash
chmod +x /Users/MN/GITHUB/ULTRA-DEV/hooks/enforce-worktree
```

## Step 2: Add Hook to Claude Settings

Edit `~/.claude/settings.json`. In the `PreToolUse` array, add:

```json
{
  "matcher": "Edit|Write|Read",
  "hooks": [
    {
      "type": "command",
      "command": "/Users/MN/GITHUB/ULTRA-DEV/hooks/enforce-worktree"
    }
  ]
}
```

## Step 3: Create a Worktree

```bash
cd /Users/MN/GITHUB/ULTRA-DEV
git worktree add ../ULTRA-DEV-wt-test -b experiment/test
```

## Step 4: Start Claude with WORKTREE_ROOT

```bash
export WORKTREE_ROOT=/Users/MN/GITHUB/ULTRA-DEV-wt-test
cd /Users/MN/GITHUB/ULTRA-DEV-wt-test
claude -p "Read README.md"
```

## Step 5: Verify

- Operations inside `/Users/MN/GITHUB/ULTRA-DEV-wt-test/` should succeed
- Operations outside that path should be blocked with error message

## Test Commands

```bash
# Should succeed (inside worktree)
claude -p "Read README.md"

# Should be blocked (outside worktree)
claude -p "Read /Users/MN/GITHUB/ULTRA-DEV/README.md"
```

## Result

If blocked operations return error "Path X is outside worktree boundary Y", the hook is working.
