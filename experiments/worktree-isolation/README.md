# Experiment: Worktree Isolation

## Objective

Prove that agents can be restricted to a specific git worktree using environment variables and hooks, enabling parallel development clusters that cannot interfere with each other.

## Big Picture Fit

This is foundational for the eventual workflow:
1. Trigger (manual or automated) initiates work on a GitHub issue
2. Orchestrator spawns in a worktree for that issue
3. Orchestrator spawns sub-agents (implementer, reviewers) via separate Claude sessions
4. Sub-agents do the work, restricted to their worktree
5. Result: PR-ready changes on a feature branch

## Why Separate Claude Sessions

The orchestrator spawns sub-agents using `claude -p "prompt"` with WORKTREE_ROOT exported, NOT using the Task tool. Reasons:

1. **Environment control**: Each session starts with WORKTREE_ROOT set
2. **Hook enforcement**: The enforce-worktree hook sees WORKTREE_ROOT and enforces boundaries
3. **Process isolation**: If one agent crashes, others continue
4. **Deterministic**: Not relying on instructions, relying on enforcement

The Task tool spawns subagents within the same process. They can't have different WORKTREE_ROOT values. Separate sessions can.

---

## Phases

### Phase 1: Hook Enforcement Test

**What we're testing**: Does the enforce-worktree hook actually block operations outside WORKTREE_ROOT?

**Method**:
1. Create a test worktree manually
2. Start a Claude session with WORKTREE_ROOT set
3. Instruct it to read/write files inside the worktree (should succeed)
4. Instruct it to read/write files outside the worktree (should be blocked)

**Success Criteria**:
- [x] Operations inside worktree: APPROVED by hook
- [x] Operations outside worktree: BLOCKED by hook with clear error message
- [x] The blocking is deterministic (not instruction-dependent)

### Phase 2: Orchestrator Spawns Sub-Agent

**What we're testing**: Can an orchestrator spawn a Claude session with WORKTREE_ROOT, and does that session get enforced?

**Method**:
1. Manually start an orchestrator (this session, or a fresh one)
2. Orchestrator creates a worktree
3. Orchestrator runs: `export WORKTREE_ROOT=/path && claude -p "prompt" > output.txt`
4. Check if sub-agent was restricted

**Success Criteria**:
- [x] Sub-agent session starts successfully
- [x] Sub-agent can operate inside worktree
- [x] Sub-agent cannot operate outside worktree (hook blocks)
- [x] Output captured to file

### Phase 3: Full Parallel Test

**What we're testing**: Can 3 orchestrators run simultaneously, each spawning sub-agents in their own worktrees, without cross-contamination?

**Method**:
1. Start 3 orchestrators (via background processes or separate terminals)
2. Each creates its own worktree and spawns sub-agents
3. Sub-agents create files, make commits
4. Verify isolation

**Success Criteria**:
- [x] 3 worktrees created: ULTRA-DEV-wt-a, ULTRA-DEV-wt-b, ULTRA-DEV-wt-c
- [x] Each worktree has expected file(s) created
- [x] No files modified in wrong worktrees
- [x] No files modified in main ULTRA-DEV
- [ ] Commits on correct branches (experiment/a, experiment/b, experiment/c) - not tested, agents created files only

---

## Current Status

- [x] Phase 1: PASSED (5/5 tests)
- [x] Phase 2: PASSED
- [x] Phase 3: PASSED (no cross-contamination)

---

## Files

```
experiments/worktree-isolation/
├── README.md                 # This file
├── phase-1/
│   ├── prompt.md             # Exact prompt for phase 1 test
│   └── results.md            # What happened
├── phase-2/
│   ├── orchestrator-prompt.md
│   ├── subagent-prompt.md    # Saved by orchestrator BEFORE execution
│   └── results.md
└── phase-3/
    ├── orchestrator-a-prompt.md
    ├── orchestrator-b-prompt.md
    ├── orchestrator-c-prompt.md
    ├── subagent-prompts/     # Saved by orchestrators during execution
    └── results.md
```
