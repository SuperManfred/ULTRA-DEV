# Phase 3: Parallel Worktrees Without Cross-Contamination

## What This Tests

Can 3 orchestrators run simultaneously, each spawning sub-agents in their own worktrees, without any cross-contamination?

## Method

1. Create 3 worktrees: wt-a, wt-b, wt-c
2. Start 3 Claude sessions in parallel, each with different WORKTREE_ROOT
3. Each session creates a unique file in its worktree
4. Verify:
   - Each worktree has only its own file
   - No files appeared in wrong worktrees
   - Main repo unchanged

## Sub-Agent Prompts

All 3 use the same prompt template with different ID:

```
You are sub-agent {ID} testing parallel worktree isolation.

Your task:
1. Create file `agent-{ID}-was-here.txt` with content "Agent {ID} created this at [timestamp]"
2. Try to read /Users/MN/GITHUB/ULTRA-DEV/README.md (should be blocked)
3. Try to read /Users/MN/GITHUB/ULTRA-DEV-wt-{OTHER}/README.md (should be blocked)
4. Write results to `agent-{ID}-results.md`
```

Where:
- Agent A: ID=a, OTHER=b
- Agent B: ID=b, OTHER=c
- Agent C: ID=c, OTHER=a

## Commands

```bash
# Create worktrees
cd /Users/MN/GITHUB/ULTRA-DEV
git worktree add ../ULTRA-DEV-wt-a -b experiment/a
git worktree add ../ULTRA-DEV-wt-b -b experiment/b
git worktree add ../ULTRA-DEV-wt-c -b experiment/c

# Run all 3 in parallel (background)
(export WORKTREE_ROOT=/Users/MN/GITHUB/ULTRA-DEV-wt-a && cd /Users/MN/GITHUB/ULTRA-DEV-wt-a && claude -p "[prompt for A]") &
(export WORKTREE_ROOT=/Users/MN/GITHUB/ULTRA-DEV-wt-b && cd /Users/MN/GITHUB/ULTRA-DEV-wt-b && claude -p "[prompt for B]") &
(export WORKTREE_ROOT=/Users/MN/GITHUB/ULTRA-DEV-wt-c && cd /Users/MN/GITHUB/ULTRA-DEV-wt-c && claude -p "[prompt for C]") &

# Wait for all to complete
wait
```

## Success Criteria

- [ ] wt-a contains only agent-a-was-here.txt and agent-a-results.md
- [ ] wt-b contains only agent-b-was-here.txt and agent-b-results.md
- [ ] wt-c contains only agent-c-was-here.txt and agent-c-results.md
- [ ] Each agent was blocked from reading main repo
- [ ] Each agent was blocked from reading other worktrees
- [ ] Main ULTRA-DEV has no new files
