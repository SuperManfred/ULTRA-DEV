# Proven Claims

This document records what we've validated with evidence, including the limitations of each claim.

---

## Claim 1: WORKTREE_ROOT + hook prevents accidental cross-worktree file operations

**Evidence:**
- `hooks/enforce-worktree`
- `experiments/worktree-isolation/phase-1/results.md`

**Limits:**
- Only Read/Write/Edit tools are blocked
- Bash is approved (not a security boundary)
- Threat model is accidental contamination, not intentional escape

---

## Claim 2: Orchestrator can spawn a restricted sub-agent via separate CLI session

**Evidence:**
- `experiments/worktree-isolation/phase-2/recipe.md`
- `experiments/worktree-isolation/phase-2/results.md`

**Limits:**
- Task tool cannot set per-subagent WORKTREE_ROOT
- Requires separate CLI process (`claude -p`)

---

## Claim 3: Parallel CLI sessions with distinct WORKTREE_ROOT values do not cross-contaminate

**Evidence:**
- `experiments/worktree-isolation/phase-3/recipe.md`
- `experiments/worktree-isolation/phase-3/results-a.md`
- `experiments/worktree-isolation/phase-3/results-b.md`
- `experiments/worktree-isolation/phase-3/results-c.md`

**Limits:**
- File-tool isolation only
- Not hardened against Bash or escape attempts

---

## Canonical Recipe for Spawning Restricted Agent

```bash
git worktree add ../REPO-wt-NAME -b feature/NAME
export WORKTREE_ROOT=/path/to/REPO-wt-NAME
cd /path/to/REPO-wt-NAME
claude -p "prompt"
```

The agent will be blocked from Read/Write/Edit operations outside WORKTREE_ROOT.

For full reproduction steps, see:
- `experiments/worktree-isolation/phase-1/recipe.md`
- `experiments/worktree-isolation/phase-2/recipe.md`
- `experiments/worktree-isolation/phase-3/recipe.md`

---

## Claim 4: Task tool cannot test tension architecture

**Evidence:**
- `docs/principles/anti-patterns.md` — Anti-pattern #11
- 2026-01-12 experiment: 3 implementers spawned via Task tool, each ONE turn, ZERO review, ZERO dialog

**What went wrong:**
- Task tool runs agents within the current session
- No way to spawn orchestrator that drives multi-turn dialog with reviewers
- Result: one-shot implementations that test NOTHING about the architecture

---

## Claim 5: One-shot without review is useless

**Evidence:**
- Worktrees `ULTRA-DEV-wt-1`, `wt-2`, `wt-3` contain scripts created without review
- These scripts exist but have NOT been validated by the tension architecture
- They should NOT be merged until validated by a proper experiment

---

## Claim 6: Turn count reporting is essential for verification

**Evidence:**
- Without turn counts, a one-shot looks identical to a thorough multi-turn dialog
- User cannot verify quality process occurred
- See `docs/vision.md` — Orchestrator MUST report turn count with EACH reviewer

---

## Claim 7: Cross-model diversity works (Empirically Proven)

**Evidence (Issue #9, 2026-01-13):**
- GPT-5.2 high caught issues that Claude Opus 4.5 missed
- The issue was substantive and would have caused downstream problems
- Single-model review would have approved the flawed implementation

**What This Proves:**
- Cross-model review is not just theoretical value — it's empirically validated
- Different training produces genuinely different blind spots
- The tension architecture's model diversity is a real quality mechanism

**Limits:**
- Sample size of 1 run (more data needed)
- We know it works, but we don't know the rate of divergence
- Does not prove every run will find cross-model catches

---

## Claim 8: Sampling creates false confidence

**Evidence (Issue #9, 2026-01-13):**
- Claude approved with sampling ("looks good")
- GPT-5.2 did full verification and found the gap
- The difference was sampling vs. full verification, not model intelligence

**What This Proves:**
- The anti-sampling rule is critical, not just good practice
- Even capable models miss things when they sample
- Full verification catches what sampling misses

---

## Assumptions / Non-Goals

- **Not a security sandbox.** Bash commands are approved. An agent could theoretically use Bash to access files outside the boundary.
- **Threat model is accidental contamination.** The hook prevents agents from accidentally reading/writing wrong files, not from intentionally escaping.
- **Absolute paths are acceptable.** Audience is internal (future agents in this system), not external users.

---

## Related

- [../principles/experimentation.md](../principles/experimentation.md) — How to run experiments
- [../vision.md](../vision.md) — The 6 priorities
- [../execution/parallel-processing.md](../execution/parallel-processing.md) — Uses these proven claims
