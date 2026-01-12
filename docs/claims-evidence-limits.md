# Claims, Evidence, Limits

## Claims

### 1. WORKTREE_ROOT + hook prevents accidental cross-worktree file operations

**Evidence:**
- `hooks/enforce-worktree`
- `experiments/worktree-isolation/phase-1/results.md`

**Limits:** Only Read/Write/Edit tools. Bash is approved. Not a security boundary.

### 2. Orchestrator can spawn a restricted sub-agent via separate CLI session

**Evidence:**
- `experiments/worktree-isolation/phase-2/recipe.md`
- `experiments/worktree-isolation/phase-2/results.md`

**Limits:** Task tool cannot set per-subagent WORKTREE_ROOT. Requires separate CLI process.

### 3. Parallel CLI sessions with distinct WORKTREE_ROOT values do not cross-contaminate

**Evidence:**
- `experiments/worktree-isolation/phase-3/recipe.md`
- `experiments/worktree-isolation/phase-3/results-a.md`
- `experiments/worktree-isolation/phase-3/results-b.md`
- `experiments/worktree-isolation/phase-3/results-c.md`

**Limits:** File-tool isolation only. Not hardened against Bash or escape attempts.

---

## Canonical Recipe

To spawn a restricted agent:

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

## Assumptions / Non-Goals

- **Not a security sandbox.** Bash commands are approved. An agent could theoretically use Bash to access files outside the boundary.
- **Threat model is accidental contamination.** The hook prevents agents from accidentally reading/writing wrong files, not from intentionally escaping.
- **Absolute paths are acceptable.** Audience is internal (future agents in this system), not external users.

---

## What the Failed Experiment Revealed (2026-01-12)

### Claim: Task tool cannot test tension architecture

**Evidence:**
- `docs/anti-patterns.md` - Anti-pattern #1
- 3 implementers spawned via Task tool, each ONE turn, ZERO review, ZERO dialog

**What went wrong:**
- Task tool runs agents within the current session
- No way to spawn orchestrator that drives multi-turn dialog with reviewers
- Result: one-shot implementations that test NOTHING about the architecture

### Claim: One-shot without review is useless

**Evidence:**
- Worktrees `ULTRA-DEV-wt-1`, `wt-2`, `wt-3` contain scripts created without review
- These scripts exist but have NOT been validated by the tension architecture
- They should NOT be merged until validated by an actual experiment

### Claim: Turn count reporting is essential for verification

**Evidence:**
- Without turn counts, a one-shot looks identical to a thorough multi-turn dialog
- User cannot verify quality process occurred
- See `docs/vision.md` - Orchestrator MUST report turn count with EACH reviewer
