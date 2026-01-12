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
│       ├── phase-1/            # Hook enforcement (COMPLETE - 5/5 passed)
│       ├── phase-2/            # Orchestrator spawns sub-agent
│       └── phase-3/            # Full parallel test
└── prompts/                    # Saved prompts for traceability
```

## Key Mechanism: Environment-Based Isolation

Each worktree session is spawned with:
```bash
export WORKTREE_ROOT=/path/to/worktree
```

PreToolUse hook (integrated into ~/.claude/settings.json) checks:
- Is `WORKTREE_ROOT` set? If no, normal session - approve all.
- Is `WORKTREE_ROOT` set? If yes, check target path.
- Target inside `WORKTREE_ROOT`? Approve.
- Target outside `WORKTREE_ROOT`? Block.

## Proven

- [x] enforce-worktree hook blocks operations outside WORKTREE_ROOT (Phase 1: 5/5 tests passed)

## In Progress

- [ ] Phase 2: Orchestrator spawns sub-agent with WORKTREE_ROOT
- [ ] Phase 3: Parallel worktrees without cross-contamination

## Not Yet Built

Future components will be created when we have precise, documented plans for them.
