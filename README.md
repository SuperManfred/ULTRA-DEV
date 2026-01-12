# ULTRA-DEV

Multi-agent development infrastructure. Groups of AI agents working on GitHub issues in isolated git worktrees, simultaneously, with enforced boundaries and self-verification.

## Goal

Multiple agent clusters running in parallel:
- Each cluster works on one GitHub issue
- Each cluster isolated in its own git worktree
- Each cluster verifies its own work before claiming done
- Agents cannot touch anything outside their worktree
- No human babysitting required for correct results

## Structure

```
ULTRA-DEV/
├── bin/                    # Scripts
│   ├── spawn-cluster       # Create worktree + start agent cluster
│   └── worktree-guard      # PreToolUse hook for isolation enforcement
├── hooks/                  # Claude hook scripts
│   └── enforce-worktree    # Block operations outside WORKTREE_ROOT
├── templates/              # CLAUDE.local.md templates per role
│   ├── implementer.md
│   ├── reviewer-1.md
│   └── reviewer-2.md
├── protocols/              # Coordination protocols
│   └── review-cycle.md     # How agents communicate within a cluster
└── docs/                   # Design docs and decisions
    └── architecture.md     # This system's design
```

## Key Mechanism: Environment-Based Isolation

Each worktree session is spawned with:
```bash
export WORKTREE_ROOT=/path/to/worktree
export WORKTREE_BRANCH=feature/issue-123
```

PreToolUse hook checks:
- Is `WORKTREE_ROOT` set? If no, normal session - approve all.
- Is `WORKTREE_ROOT` set? If yes, check target path.
- Target inside `WORKTREE_ROOT`? Approve.
- Target outside `WORKTREE_ROOT`? Block.

This means:
- Normal Claude sessions (no env var) are unrestricted
- Worktree sessions are restricted to their worktree only
- Multiple worktree sessions can run simultaneously without cross-contamination

## Status

- [ ] `spawn-cluster` script
- [ ] `enforce-worktree` hook
- [ ] Role templates
- [ ] Review cycle protocol
- [ ] Integration with existing ~/.claude/settings.json hooks
