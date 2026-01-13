# ULTRA-DEV Vision

## The Goal

Build infrastructure where you say "work on issues #1, #2, #3" and get back high-quality, fact-based implementations.

## The 6 Weighted Priorities

Everything in this system derives from these priorities. Total weight: 100.

| Weight | Priority | Summary |
|--------|----------|---------|
| **28** | [Quality through structured architecture](#priority-1-quality-through-structured-architecture) | Quality emerges from cluster structure, not individual agents |
| **27** | [Hyper-efficient resilient execution](#priority-2-hyper-efficient-resilient-execution) | Maximum efficiency, agents don't die, cluster self-corrects |
| **15** | [Parallel multi-issue processing](#priority-3-parallel-multi-issue-processing) | Work on #1, #2, #3 simultaneously in isolated worktrees |
| **15** | [Self-improvement](#priority-4-self-improvement) | Every run produces work AND reusable knowledge |
| **10** | [Deterministic validation first](#priority-5-deterministic-validation-first) | Catch mechanical errors before AI reviewers engage |
| **5** | [Evidence-based, no sampling](#priority-6-evidence-based-no-sampling) | file:line citations, verify ALL N items |

---

## Priority 1: Quality through structured architecture

**Weight: 28** — Quality is a property of the cluster structure, not individual agents.

### Why Structure Matters

Single agents optimize for agreeableness over truth. They agree with wrong statements, build on flawed premises, and claim completion without verification.

Quality emerges from:
- **Clear roles**: Orchestrator coordinates, implementer implements, reviewers review
- **Diverse models**: Different training = different blind spots covered
- **Structured interaction**: Dialog protocol, evidence requirements, consensus through truth

### The Roles

| Role | Responsibility | Access |
|------|----------------|--------|
| **Coordinator** | Spawns orchestrators, collects results, reports to user | — |
| **Orchestrator** | Owns one worktree, runs implementation/review loop, drives dialog to consensus | Read/Write |
| **Implementer** | Writes code, cites changes, receives iteration specs | Read/Write |
| **Reviewers** (3) | Guardians of quality, raise concerns with evidence, reach consensus | Read-only |

### Model Diversity

| Reviewer | Model | Purpose |
|----------|-------|---------|
| Reviewer-1 | Claude Opus 4.5 | Catches implementation gaps |
| Reviewer-2 | GPT-5.2 high | Different training, high reasoning |
| Reviewer-3 | GPT-5.2-codex | Code-specialized perspective |

### Reviewers as Quality Guardians

Reviewers are **guardians of quality**, not busy bees disagreeing for the sake of it. They are intelligent enough to:
- Raise ALL matters that concern them
- Focus on matters with **significant impact** or that are **dependencies**
- Reach consensus **as quickly as possible** while validating ALL significant concerns

The goal is not disagreement — it's **consensus found through truth**.

**See:** [architecture/cluster-structure.md](architecture/cluster-structure.md), [architecture/dialog-protocol.md](architecture/dialog-protocol.md)

---

## Priority 2: Hyper-efficient resilient execution

**Weight: 27** — Maximum efficiency and effectiveness, capable of hours/overnight when task complexity demands.

### Agents Do NOT Die

The cluster prevents agent death by:
- **Calibrating work to fit context** — Break tasks to fit available context window
- **Externalizing state to files** — `.claude/` directory holds all state
- **Checkpointing progress** — Each iteration's work is saved before proceeding

### Cluster Self-Correction

If an agent fails unexpectedly:
1. Cluster detects the failure
2. Re-initializes with refined, better-calibrated objective
3. Continues from last checkpoint

Human involved only for genuine judgment calls, never technical failures.

### Fresh Implementer Principle

**Fresh implementer is the DEFAULT — not an optimization, a requirement.**

| Scenario | Action |
|----------|--------|
| Heavy context consumption | Spawn fresh |
| Light context, unrelated next phase | Spawn fresh |
| Light context, same context needed | May reuse |
| Uncertain | Spawn fresh |

Logic: Fresh implementer has 100% context + focused prompt = MAXIMUM capacity.
Reused implementer has consumed context + coordination overhead = LESS capacity.

**See:** [execution/resilient-execution.md](execution/resilient-execution.md)

---

## Priority 3: Parallel multi-issue processing

**Weight: 15** — Work on issues #1, #2, #3 simultaneously. Each cluster isolated in its own worktree.

### Worktree Isolation

```bash
git worktree add ../REPO-wt-N -b feature/N
export WORKTREE_ROOT=/path/to/REPO-wt-N
cd $WORKTREE_ROOT
claude -p "prompt"
```

The `enforce-worktree` hook checks:
- Target inside `WORKTREE_ROOT`? → Approve
- Target outside `WORKTREE_ROOT`? → Block

**Note:** This is not a security boundary — Bash is not restricted. It prevents accidental cross-worktree contamination.

### Proven Capability

| Phase | What It Tests | Result |
|-------|---------------|--------|
| 1 | Hook blocks operations outside WORKTREE_ROOT | PASSED |
| 2 | Orchestrator spawns restricted sub-agent via `claude -p` | PASSED |
| 3 | 3 parallel worktrees, no cross-contamination | PASSED |

**See:** [execution/parallel-processing.md](execution/parallel-processing.md), [learnings/proven-claims.md](learnings/proven-claims.md)

---

## Priority 4: Self-improvement

**Weight: 15** — Every run produces work AND reusable knowledge.

### Why This Matters

When an agent fails 3 times then succeeds on attempt 4, that pattern will be repeated by every future agent unless captured.

Each repeated failure:
- Consumes context window
- Adds latency
- Wastes compute

**Capture the solution once → all future agents succeed on attempt 1 → non-linear improvement.**

### What Captured Knowledge Looks Like

```markdown
# skill-name

Brief description of when this applies.

## Key Points
- Concise instruction 1
- Concise instruction 2

## Example
```bash
correct invocation here
```

## Common Errors
| Error | Cause | Fix |
|-------|-------|-----|
| error message | root cause | solution |
```

### When to Capture

- Agent fails then succeeds → capture the fix
- Unexpected behavior discovered → document it
- CLI invocation has non-obvious flags → document them
- Any pattern that would save future agents from wasted attempts

Skills are stored in `.claude/skills/` per worktree and can be promoted to the main repo.

---

## Priority 5: Deterministic validation first

**Weight: 10** — Catch mechanical errors (lint, typecheck, build, test) before AI reviewers engage.

### Every Reviewer Turn is High-Value Judgment

Don't waste reviewer cycles on:
- Syntax errors that a linter would catch
- Type errors that a compiler would catch
- Test failures that a test runner would catch

Run deterministic validation BEFORE spawning reviewers:

```bash
# Run before review phase
npm run lint && npm run typecheck && npm run test
```

If any fail → fix first, then review.

**See:** [execution/deterministic-validation.md](execution/deterministic-validation.md)

---

## Priority 6: Evidence-based, no sampling

**Weight: 5** — file:line citations. Verify ALL N items, not a sample. Never imply completeness.

### Anti-Sampling Principle

**NEVER sample. If a claim covers N items, verify ALL N items.**

| Sampling Failure | Correct Approach |
|------------------|------------------|
| "Tests pass" (ran 3 of 50) | Run ALL 50 tests |
| "All criteria met" (checked 2 of 7) | Check ALL 7 criteria |
| "Files look good" (read 1 of 5) | Read ALL 5 files |

**If full verification is impractical, state explicitly:**
"Verified X of Y items" — never imply completeness.

### Evidence Requirements

Every assertion requires grounding:
- File:line citations for claims about code
- Specific examples for concerns
- Focus on what has significant impact or is a dependency

Opinions without evidence are dismissed. Evidence decides.

### Proven Value

From Issue #9 (2026-01-13):
- Claude approved with sampling ("looks good")
- GPT-5.2 did full verification and found the gap
- The difference was sampling vs. full verification, not model intelligence

**See:** [principles/evidence-rules.md](principles/evidence-rules.md)

---

## Limits (Safety Rails, Not Goals)

| Limit | Value | Purpose |
|-------|-------|---------|
| MAX_ITERATIONS | 3 | Escalate if stuck, don't loop forever |
| MAX_DIALOG_ROUNDS | 5 | Escalate if deadlocked, don't debate forever |

These are safety rails for pathological cases. The goal is quality work in as many iterations as it takes, not fitting within bounds.

Escalation is an acceptable outcome — better to escalate than produce slop.

---

## Success Definition

**Success IS:**
- Issue resolved with high-quality, fact-based implementation
- All significant concerns resolved with evidence
- Consensus found through truth
- Orchestrator reports interaction diagram and resolution path
- Reusable knowledge captured from failure-then-success patterns

**Acceptable escalation:**
- Unresolved disagreement on genuinely ambiguous requirements
- Requirements that need user clarification
- Better to escalate than produce slop

**Questions and Escalation:**
- Cluster agents raise questions to orchestrator
- Orchestrator resolves with available context or escalates to user
- Don't guess — ask. Don't produce slop because a question wasn't asked.

---

## Document Map

| Document | Purpose |
|----------|---------|
| **Architecture** | |
| [architecture/cluster-structure.md](architecture/cluster-structure.md) | Roles, isolation, worktree mechanics |
| [architecture/dialog-protocol.md](architecture/dialog-protocol.md) | Consensus building, message relay |
| [architecture/file-structure.md](architecture/file-structure.md) | `.claude/` layout, what goes where |
| **Execution** | |
| [execution/deterministic-validation.md](execution/deterministic-validation.md) | Lint/test/build before reviewers |
| [execution/resilient-execution.md](execution/resilient-execution.md) | Context management, checkpointing, recovery |
| [execution/parallel-processing.md](execution/parallel-processing.md) | Worktree isolation, coordinator flow |
| **Principles** | |
| [principles/evidence-rules.md](principles/evidence-rules.md) | file:line citations, NO SAMPLING |
| [principles/anti-patterns.md](principles/anti-patterns.md) | What NOT to do |
| [principles/experimentation.md](principles/experimentation.md) | How to test new mechanics |
| **Learnings** | |
| [learnings/proven-claims.md](learnings/proven-claims.md) | What we've validated with evidence |
| **Operations** | |
| [operations/launcher.md](operations/launcher.md) | How to start orchestrators, hook installation |
