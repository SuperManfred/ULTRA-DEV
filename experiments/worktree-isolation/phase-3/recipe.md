# Phase 3 Recipe: Parallel Worktrees Without Cross-Contamination

This is the exact recipe to reproduce Phase 3.

## What This Proves

Multiple Claude sessions can run in parallel, each restricted to its own worktree, with no cross-contamination.

## Prerequisites

- Phase 1 complete (hook installed)
- Phase 2 complete (sub-agent spawning works)

## Step 1: Create Multiple Worktrees

```bash
cd /Users/MN/GITHUB/ULTRA-DEV
git worktree add ../ULTRA-DEV-wt-a -b experiment/a
git worktree add ../ULTRA-DEV-wt-b -b experiment/b
git worktree add ../ULTRA-DEV-wt-c -b experiment/c
```

## Step 2: Run Sessions in Parallel

```bash
# Run all 3 in background, wait for completion
(export WORKTREE_ROOT=/Users/MN/GITHUB/ULTRA-DEV-wt-a && cd /Users/MN/GITHUB/ULTRA-DEV-wt-a && claude -p "[prompt-a]") &
(export WORKTREE_ROOT=/Users/MN/GITHUB/ULTRA-DEV-wt-b && cd /Users/MN/GITHUB/ULTRA-DEV-wt-b && claude -p "[prompt-b]") &
(export WORKTREE_ROOT=/Users/MN/GITHUB/ULTRA-DEV-wt-c && cd /Users/MN/GITHUB/ULTRA-DEV-wt-c && claude -p "[prompt-c]") &
wait
```

## Step 3: Verify No Cross-Contamination

```bash
# Check each worktree has only its own files
ls /Users/MN/GITHUB/ULTRA-DEV-wt-a/*agent*.txt
ls /Users/MN/GITHUB/ULTRA-DEV-wt-b/*agent*.txt
ls /Users/MN/GITHUB/ULTRA-DEV-wt-c/*agent*.txt

# Verify no cross-contamination
find /Users/MN/GITHUB/ULTRA-DEV-wt-b /Users/MN/GITHUB/ULTRA-DEV-wt-c -name "*agent-a*"
# Should return nothing

find /Users/MN/GITHUB/ULTRA-DEV -maxdepth 1 -name "*agent*"
# Should return nothing (main repo unchanged)
```

## Expected Results

| Worktree | Contains | Does NOT Contain |
|----------|----------|------------------|
| wt-a | agent-a-* files | agent-b-*, agent-c-* |
| wt-b | agent-b-* files | agent-a-*, agent-c-* |
| wt-c | agent-c-* files | agent-a-*, agent-b-* |
| main | no agent files | any agent-* files |

## Key Learnings

1. **Each session gets its own WORKTREE_ROOT**: The `export` before `claude -p` sets the boundary for that session only.

2. **Parallel execution is safe**: Multiple Claude sessions with different WORKTREE_ROOT values can run simultaneously without interference.

3. **Hook prevents cross-worktree access**: Each agent was blocked when trying to read from main repo or other worktrees.

4. **No special coordination needed**: The isolation is enforced by the hook, not by agent cooperation.

## Prompts Used

See:
- `prompt-a.md`
- `prompt-b.md`
- `prompt-c.md`
