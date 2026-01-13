# Anti-Patterns: What NOT to Do

This document consolidates all anti-patterns discovered through experimentation and failure analysis.

---

## Anti-Pattern #1: One-Shot Implementation Without Review

### What Happens

```
COORDINATOR spawns implementers
Each implementer runs ONCE and returns
0 review rounds
0 dialog protocol
0 evidence validation
0 cross-model tension
```

### Why It's Useless

Single agents optimize for agreeableness over truth. They:
- Do what they're told without questioning assumptions
- Build on flawed premises
- Miss edge cases
- Produce code that "works" but doesn't meet actual requirements

### Evidence

2026-01-12 experiment: 3 "implementers" via Task tool, each ONE turn, ZERO review, ZERO dialog. The scripts created exist but have NOT been validated by the tension architecture.

### The Fix

Multiple iterations + 3 reviewers + dialog protocol + evidence validation.

---

## Anti-Pattern #2: Same-Model Review

### What Happens

All reviewers are the same model (e.g., all Claude).

### Why It's Useless

Same training = same blind spots = same biases = no real review.

Claude reviewing Claude's work means:
- Same tendency to agree with confident-sounding assertions
- Same gaps in knowledge
- No diversity in perspective

### Evidence

Issue #9 (2026-01-13): GPT-5.2 caught what Claude missed. The issue was substantive. Single-model review would have approved flawed implementation.

### The Fix

Cross-model reviewers: Claude Opus 4.5 + GPT-5.2 high + GPT-5.2-codex.

---

## Anti-Pattern #3: No Evidence Requirements

### What Happens

Implementers "implement" and "return summary" without:
- File:line citations
- Evidence of meeting acceptance criteria
- Justification for design decisions

### Why It's Useless

Without evidence requirements, agents:
- Claim completion without verification
- Build on assumptions instead of facts
- Miss acceptance criteria

### The Fix

Require file:line citations for ALL claims. Dismiss opinions without evidence.

---

## Anti-Pattern #4: No Default-Reject Stance

### What Happens

No reviewer is primed to be skeptical. Everyone assumes the work is fine.

### Why It's Useless

Without a gatekeeper:
- Incomplete work gets accepted
- Edge cases get ignored
- "Good enough" replaces "correct"

### The Fix

At least one reviewer (Reviewer-2 GPT-5.2) should have default stance: REJECT unless proven complete.

---

## Anti-Pattern #5: Disagreement for Its Own Sake

### What Happens

Treating reviewers as artificial adversaries who should disagree for the sake of friction.

### Why It's Wrong

Reviewers are **guardians of quality**, not busy bees. They are intelligent models capable of:
- Raising ALL matters that concern them
- Focusing on matters with **significant impact**
- Reaching consensus **as quickly as possible**

The goal is not disagreement — it's consensus found through truth.

### The Fix

Trust reviewers to be efficient. Don't incentivize artificial friction.

---

## Anti-Pattern #6: Not Reporting Turn Counts and Resolutions

### What Happens

Orchestrator reports "Issue #N complete" without:
- How many turns with EACH reviewer
- What concerns were raised
- How each was resolved

### Why It's Wrong

Without this information, user cannot verify quality assurance actually occurred. A one-shot with zero review looks the same as a thorough multi-turn dialog.

### The Fix

Report:
- Turn count per reviewer (R1: 3 turns, R2: 5 turns, R3: 2 turns)
- Concerns raised with evidence
- Resolution for each concern

---

## Anti-Pattern #7: Guessing Instead of Asking

### What Happens

Requirements are unclear. Agent makes an assumption and proceeds. Result is slop.

### Why It's Wrong

Agents can ask questions. The escalation hierarchy exists:
1. Agent → Orchestrator
2. Orchestrator → User (rare but available)

### The Fix

Don't guess — ask. Don't produce slop because a question wasn't asked.

---

## Anti-Pattern #8: Treating Limits as Goals

### What Happens

Treating MAX_ITERATIONS=3 and MAX_DIALOG_ROUNDS=5 as targets to stay within.

### Why It's Wrong

These are **safety rails for pathological cases**, not goals.

- If quality achieved in 1 iteration: great
- If quality requires 3 iterations: fine
- If stuck after limits: escalate (acceptable)

### The Fix

Focus on quality, not fitting within bounds. Escalation is acceptable.

---

## Anti-Pattern #9: Sampling Instead of Full Verification

### What Happens

Agent claims "all X verified" but actually only checked a subset.

Examples:
- "All tests pass" (ran 3 of 50)
- "All criteria met" (checked 2 of 7)
- "All files reviewed" (read 1 of 5)

### Why It's Wrong

Sampling creates false confidence. The unverified items are where bugs hide.

### Evidence

Issue #9: Claude approved with sampling. GPT-5.2 did full verification and found the gap.

### The Fix

**NEVER sample.** If a claim covers N items, verify ALL N items.

If impractical, state: "Verified X of Y items" — never imply completeness.

---

## Anti-Pattern #10: Conflating Coordinator with Orchestrator

### What Happens

The coordinator tries to do the orchestrator's job, collapsing the architecture into single-shot delegation.

### Why It's Wrong

- **Coordinator's job**: Spawn orchestrators, collect final results
- **Orchestrator's job**: Run the implementation loop with tension architecture

Mixing them = no review loop = useless.

### The Fix

Keep roles separate. Coordinator spawns and waits. Orchestrator runs the loop.

---

## Anti-Pattern #11: Using Task Tool for Sub-Agent Spawning

### What Happens

Orchestrator uses Task tool to spawn implementers and reviewers.

### Why It's Wrong

Task tool runs agents within the current session:
- Cannot set per-subagent WORKTREE_ROOT
- Cannot enforce isolation
- Cannot capture output separately

### Evidence

2026-01-12: Task tool cannot test tension architecture.

### The Fix

Use separate CLI processes: `claude -p` and `codex exec`.

---

## Anti-Pattern #12: Orchestrator Implementing Directly

### What Happens

Orchestrator decides to "just do it myself" instead of spawning a fresh implementer.

### Why It's Wrong

This is **logically impossible** to produce a better outcome:

| Agent | Context | Capacity |
|-------|---------|----------|
| Fresh implementer | 100% available + focused prompt | MAXIMUM |
| Orchestrator | Already-consumed + coordination overhead | LESS |

### The Fix

If implementer fails, the problem is the prompt or the task. Fix THAT, retry with fresh implementer, or escalate. Never "I'll do it myself."

---

## Summary Table

| Anti-Pattern | Root Cause | Fix |
|--------------|------------|-----|
| One-shot without review | No tension architecture | Multiple iterations + reviewers |
| Same-model review | No diversity | Cross-model reviewers |
| No evidence requirements | No accountability | Require file:line citations |
| No default-reject | No skepticism | Gatekeeper reviewer |
| Disagreement for its own sake | Misunderstanding purpose | Trust efficient consensus |
| No turn count reporting | No visibility | Report per-reviewer details |
| Guessing instead of asking | Pride/speed over quality | Escalation hierarchy |
| Limits as goals | Wrong framing | Quality first, limits as rails |
| Sampling | Efficiency over thoroughness | Verify ALL N items |
| Conflating coordinator/orchestrator | Role confusion | Separate responsibilities |
| Task tool for spawning | Wrong tool | Separate CLI processes |
| Orchestrator implementing | Context exhaustion | Always spawn fresh |

---

## Related

- [../vision.md](../vision.md) — The 6 priorities that prevent anti-patterns
- [evidence-rules.md](evidence-rules.md) — How to avoid evidence anti-patterns
- [../architecture/cluster-structure.md](../architecture/cluster-structure.md) — Correct role separation
