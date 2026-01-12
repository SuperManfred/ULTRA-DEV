# Alignment Plan

This plan aligns all documentation and implementation with `docs/vision.md`.

## Files to Update

### 1. `docs/orchestrator-flow.md`

**Current state:** Describes flow but with wrong framing (arbitrary limits as goals, uniform turn counts)

**Changes needed:**
- [ ] Reframe MAX_ITERATIONS and MAX_DIALOG_ROUNDS as safety rails, not goals
- [ ] Add: Orchestrator reports turn count with EACH reviewer (not uniform)
- [ ] Add: Questions/escalation hierarchy (agent → orchestrator → user)
- [ ] Reframe: Reviewers are guardians of quality, not busy bees
- [ ] Add: Orchestrator final report must include resolution path for each concern
- [ ] Remove: Any implication that fewer turns is better

### 2. `docs/spawn-cluster-spec.md`

**Current state:** Technical spec but may have wrong assumptions

**Changes needed:**
- [ ] Review against vision.md
- [ ] Add: Turn count reporting requirement in outputs
- [ ] Add: Question escalation mechanism
- [ ] Ensure: No arbitrary limits framed as goals
- [ ] Align: Role descriptions with "guardians of quality" framing

### 3. `docs/anti-patterns.md`

**Current state:** Documents some failures but missing key ones

**Changes needed:**
- [ ] Add: "Disagreement for its own sake (busywork)" as anti-pattern
- [ ] Add: "Not reporting turn counts and resolutions" as anti-pattern
- [ ] Add: "Guessing instead of asking" as anti-pattern
- [ ] Add: "Treating limits as goals" as anti-pattern
- [ ] Update: Success definition section if it contradicts vision.md

### 4. `docs/claims-evidence-limits.md`

**Current state:** Documents what we've proven

**Changes needed:**
- [ ] Review for alignment with vision.md
- [ ] Update any claims that contradict new understanding
- [ ] Add: Claims about what the experiment revealed (what NOT to do)

### 5. `docs/experimentation-principles.md`

**Current state:** Unknown - need to read

**Changes needed:**
- [ ] Review for alignment
- [ ] Update if conflicts with vision.md

### 6. `README.md`

**Current state:** Project overview

**Changes needed:**
- [ ] Point to vision.md as the authoritative source
- [ ] Ensure overview matches vision

### 7. `do-not-read.md`

**Current state:** Stale content moved from AGENTS.md

**Changes needed:**
- [ ] Either delete or update to reflect current state
- [ ] Content is about tooling mechanics, not vision

### 8. `AGENTS.md`

**Current state:** Empty or minimal

**Changes needed:**
- [ ] Decide: Should it point to vision.md or contain agent-specific instructions?
- [ ] Should NOT duplicate vision.md content

## Implementation to Evaluate

### 9. Worktree implementations (`wt-1`, `wt-2`, `wt-3`)

Scripts created during failed experiment:
- `spawn-cluster` in wt-1
- `cluster-status` in wt-2
- `cluster-cleanup` in wt-3

**Changes needed:**
- [ ] Evaluate against vision.md
- [ ] These were created without review phase - are they correct?
- [ ] Should NOT be merged until validated by proper experiment

### 10. `prompts/` directory

**Changes needed:**
- [ ] Review orchestrator-experiment.md for alignment
- [ ] Update any prompts that contradict vision.md

### 11. `hooks/` directory

**Changes needed:**
- [ ] Review enforce-worktree hook
- [ ] Ensure still aligns with isolation requirements

### 12. `.claude/` directory

**Changes needed:**
- [ ] Review any orchestrator prompts
- [ ] Ensure alignment with vision.md

## Execution Order

1. **Update docs first** (orchestrator-flow.md, spawn-cluster-spec.md, anti-patterns.md)
2. **Review supporting docs** (claims-evidence-limits.md, experimentation-principles.md)
3. **Clean up stale content** (do-not-read.md, README.md)
4. **Evaluate implementations** (worktree scripts)
5. **Run alignment test** - Have orchestrator validate all docs against vision.md

## Validation

Before proceeding with next experiment:

1. Orchestrator reads vision.md
2. Orchestrator reads all other docs
3. Orchestrator identifies any contradictions
4. Contradictions resolved before experiment runs

This is the system validating itself.
