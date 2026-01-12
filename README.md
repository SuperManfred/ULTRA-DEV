# ULTRA-DEV

Multi-agent development infrastructure. Groups of AI agents working on GitHub issues in isolated git worktrees, simultaneously, with enforced boundaries and self-verification.

## Goal

Multiple agent clusters running in parallel:
- Each cluster works on one GitHub issue
- Each cluster isolated in its own git worktree
- Each cluster verifies its own work before claiming done
- Agents cannot touch anything outside their worktree
- No human babysitting required for correct results

## Current Structure

```
ULTRA-DEV/
├── hooks/
│   └── enforce-worktree        # PreToolUse hook - blocks operations outside WORKTREE_ROOT
├── docs/
│   └── experimentation-principles.md
├── experiments/
│   └── worktree-isolation/     # Phased testing of isolation mechanism
│       ├── phase-1/            # Hook enforcement - PASSED
│       ├── phase-2/            # Sub-agent spawning - PASSED
│       └── phase-3/            # Parallel execution - PASSED
└── prompts/                    # Saved prompts for traceability
```

## Key Mechanism: Environment-Based Isolation

Each worktree session is spawned with:
```bash
export WORKTREE_ROOT=/path/to/worktree
cd /path/to/worktree
claude -p "task prompt"
```

PreToolUse hook (integrated into ~/.claude/settings.json) checks:
- Is `WORKTREE_ROOT` set? If no, normal session - approve all.
- Is `WORKTREE_ROOT` set? If yes, check target path.
- Target inside `WORKTREE_ROOT`? Approve.
- Target outside `WORKTREE_ROOT`? Block.

## Proven (All Phases Complete)

| Phase | What It Tests | Result |
|-------|---------------|--------|
| 1 | Hook blocks operations outside WORKTREE_ROOT | PASSED (5/5 tests) |
| 2 | Orchestrator spawns restricted sub-agent via `claude -p` | PASSED |
| 3 | 3 parallel worktrees, no cross-contamination | PASSED |

Each phase has a documented recipe in `experiments/worktree-isolation/phase-N/recipe.md` that can be followed to reproduce the results.

## What This Enables

An orchestrator can:
1. Create a git worktree for a feature/issue
2. Export WORKTREE_ROOT and spawn sub-agents via `claude -p`
3. Those sub-agents are automatically restricted to the worktree
4. Multiple orchestrators can run in parallel without interference

## Not Yet Built

- spawn-cluster script (automate worktree creation + agent spawning)
- Role templates (implementer, reviewers)
- Coordination protocols (how agents communicate within a cluster)

These will be created when we have precise, documented plans for them.
