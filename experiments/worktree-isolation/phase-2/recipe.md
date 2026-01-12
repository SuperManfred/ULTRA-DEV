# Phase 2 Recipe: Orchestrator Spawns Restricted Sub-Agent

This is the exact recipe to reproduce Phase 2.

## What This Proves

An orchestrator can spawn a sub-agent via `claude -p` with WORKTREE_ROOT exported, and that sub-agent will be restricted by the enforce-worktree hook.

## Prerequisites

- Phase 1 complete (hook installed and working)
- Git repository with at least one commit

## Step 1: Create Worktree

```bash
cd /Users/MN/GITHUB/ULTRA-DEV
git worktree add ../ULTRA-DEV-wt-phase2 -b experiment/phase2
```

## Step 2: Spawn Sub-Agent with WORKTREE_ROOT

```bash
export WORKTREE_ROOT=/Users/MN/GITHUB/ULTRA-DEV-wt-phase2
cd /Users/MN/GITHUB/ULTRA-DEV-wt-phase2
claude -p 'You are a sub-agent testing worktree isolation.

Your WORKTREE_ROOT should be set to restrict you to a specific path.

Perform these tasks:

1. Report your current working directory
2. Create a file called `subagent-was-here.txt` in your current directory with content "Sub-agent created this file"
3. Try to read /Users/MN/GITHUB/ULTRA-DEV/README.md (should be blocked)
4. Report whether you were blocked or allowed

Write your results to `subagent-results.md` in your current directory.'
```

## Step 3: Verify Results

```bash
# Check file was created
cat /Users/MN/GITHUB/ULTRA-DEV-wt-phase2/subagent-was-here.txt

# Check results
cat /Users/MN/GITHUB/ULTRA-DEV-wt-phase2/subagent-results.md
```

## Expected Results

1. `subagent-was-here.txt` exists with content "Sub-agent created this file"
2. `subagent-results.md` shows:
   - Operations inside worktree: SUCCESS
   - Operations outside worktree: BLOCKED

## Key Learning

When you run `claude -p` after exporting WORKTREE_ROOT, the new Claude session inherits that environment variable. The enforce-worktree hook reads it and enforces the boundary.

This means orchestrators can spawn restricted sub-agents by:
1. Exporting WORKTREE_ROOT before the `claude -p` command
2. The sub-agent is automatically restricted - no special instructions needed
