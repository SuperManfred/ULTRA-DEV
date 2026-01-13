# Parallel Multi-Issue Processing

**Priority 3 (Weight: 15)** — Work on issues #1, #2, #3 simultaneously. Each cluster isolated in its own worktree.

## Core Principle

Multiple clusters run in parallel. Each cluster owns one issue in one worktree. They cannot interfere with each other.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ USER                                                                        │
│   │                                                                         │
│   │ "Work on issues #1, #2, #3"                                             │
│   ▼                                                                         │
│ ┌─────────────────────────────────────────────────────────────────────────┐ │
│ │ COORDINATOR (Current Claude Session)                                    │ │
│ │                                                                         │ │
│ │  Spawns orchestrators in parallel (one per issue via Bash):             │ │
│ │                                                                         │ │
│ │  claude -p "Orchestrator for issue #1..." &  ───┐                       │ │
│ │  claude -p "Orchestrator for issue #2..." &  ───┼─── Run in parallel    │ │
│ │  claude -p "Orchestrator for issue #3..." &  ───┘                       │ │
│ │                                                                         │ │
│ └─────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
                              │
         ┌────────────────────┼────────────────────┐
         ▼                    ▼                    ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ ORCHESTRATOR #1 │  │ ORCHESTRATOR #2 │  │ ORCHESTRATOR #3 │
│ Issue #1        │  │ Issue #2        │  │ Issue #3        │
│ Worktree: wt-1  │  │ Worktree: wt-2  │  │ Worktree: wt-3  │
└────────┬────────┘  └────────┬────────┘  └────────┬────────┘
         │                    │                    │
         └────────────────────┴────────────────────┘
                              │
                    (Each runs independently)
```

---

## Worktree Isolation

Each cluster works in its own git worktree:

```bash
# Create worktree
git worktree add ../REPO-wt-N -b feature/N

# Set environment
export WORKTREE_ROOT=/path/to/REPO-wt-N

# Navigate
cd $WORKTREE_ROOT

# Spawn agent
claude -p "prompt"
```

### Enforcement Mechanism

The `enforce-worktree` hook (in `~/.claude/settings.json`) checks:
- Is `WORKTREE_ROOT` set? If no → normal session, approve all
- Is `WORKTREE_ROOT` set? If yes → check target path
- Target inside `WORKTREE_ROOT`? → Approve
- Target outside `WORKTREE_ROOT`? → Block

### What This Prevents

- Cluster #1 accidentally reading files from Cluster #2's worktree
- Cluster #3 accidentally writing to Cluster #1's worktree
- Cross-contamination of changes

### What This Does NOT Prevent

- Bash commands (not a security boundary)
- Intentional escape attempts

**Threat model:** Accidental contamination, not malicious escape.

---

## Proven Capability

All phases tested and passed:

| Phase | What It Tests | Result | Evidence |
|-------|---------------|--------|----------|
| 1 | Hook blocks operations outside WORKTREE_ROOT | PASSED | `experiments/worktree-isolation/phase-1/` |
| 2 | Orchestrator spawns restricted sub-agent via `claude -p` | PASSED | `experiments/worktree-isolation/phase-2/` |
| 3 | 3 parallel worktrees, no cross-contamination | PASSED | `experiments/worktree-isolation/phase-3/` |

Each phase has a documented recipe that can be reproduced.

---

## Coordinator Responsibilities

The coordinator:
1. Receives list of issues from user
2. Creates one worktree per issue
3. Spawns one orchestrator per worktree (parallel)
4. Waits for all orchestrators to complete
5. Collects final results
6. Reports summary to user

**The coordinator does NOT:**
- Do the work itself
- Interfere with orchestrators
- Manage individual agents within clusters

---

## Orchestrator Independence

Each orchestrator:
- Owns its worktree completely
- Runs its own implementation/review loop
- Manages its own dialog protocol
- Reports only when complete

Orchestrators do not communicate with each other.

---

## Worktree Lifecycle

### Creation

```bash
# From main repo
cd /path/to/REPO
git worktree add ../REPO-wt-N -b feature/N
```

### Work

All work happens inside the worktree:
```bash
export WORKTREE_ROOT=/path/to/REPO-wt-N
cd $WORKTREE_ROOT
# ... cluster does work ...
```

### Completion

On success:
```bash
cd /path/to/REPO-wt-N
git add -A
git commit -m "feat: implement issue #N"
```

### Cleanup (Manual)

After merging:
```bash
cd /path/to/REPO
git worktree remove ../REPO-wt-N
git branch -d feature/N
```

---

## Conflict Prevention

### Git Level

- Each worktree has its own branch
- Branches can be merged independently
- Merge conflicts only happen at merge time, not during work

### File Level

- Hook prevents cross-worktree file operations
- Each cluster can only touch its own files

### State Level

- Each worktree has its own `.claude/` directory
- No shared state between clusters

---

## Hard-Won Knowledge

### Task Tool Cannot Set Per-Subagent WORKTREE_ROOT

Task tool runs agents within the current session. No way to set different WORKTREE_ROOT for different subagents.

**Solution:** Use separate CLI processes (`claude -p`, `codex exec`).

### Codex Reviewers Must Run Sequentially

Running multiple `codex exec` commands in parallel causes auth conflicts.

**Solution:** Run codex reviewers one at a time:
```bash
codex exec ... # Reviewer 2 - wait for completion
codex exec ... # Reviewer 3 - wait for completion
```

Claude reviewer can run in parallel with its own codex reviewer (different CLI).

---

## Related

- [../vision.md](../vision.md) — Priority 3 overview
- [../architecture/cluster-structure.md](../architecture/cluster-structure.md) — Roles within each cluster
- [../learnings/proven-claims.md](../learnings/proven-claims.md) — Evidence for isolation
