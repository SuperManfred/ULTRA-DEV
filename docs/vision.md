# ULTRA-DEV Vision

## The Goal

Build infrastructure where you say "work on issues #1, #2, #3" and get back high-quality, fact-based implementations.

Each cluster:

- **Isolated** in its own worktree (can't corrupt others)
- **Runs as many iterations as needed** (not arbitrary limits - as many as it takes)
- **Produces work grounded in evidence** (file:line citations)
- **Reaches consensus through validating/invalidating assertions as facts**

## Reviewers as Quality Guardians

Reviewers (powered by Opus 4.5, GPT-5.2 high, GPT-5.2-codex) are **guardians of quality**, not busy bees disagreeing for the sake of it.

They are intelligent enough to:

- Raise ALL matters that concern them in their role
- Focus on matters with **significant impact** or that are **dependencies** of successfully resolving the issue
- Reach consensus **as efficiently as possible** while validating ALL significant concerns

This is not artificial friction. This is quality assurance by capable models doing their job.

## Anti-Sampling Principle

**NEVER sample. If a claim covers N items, verify ALL N items.**

This is foundational. Sampling creates false confidence:

| Sampling Failure                    | Correct Approach     |
| ----------------------------------- | -------------------- |
| "Tests pass" (ran 3 of 50)          | Run ALL 50 tests     |
| "All criteria met" (checked 2 of 7) | Check ALL 7 criteria |
| "Files look good" (read 1 of 5)     | Read ALL 5 files     |

**If full verification is impractical, state explicitly:**
"Verified X of Y items" - never imply completeness.

This applies to:

- **Orchestrator**: Verifying reviewer claims and evidence
- **Implementer**: Addressing acceptance criteria
- **Reviewers**: Checking implementation against requirements

Sampling is how slop gets approved. Full verification is how quality is assured.

## Why Tension Architecture

Single agents optimize for agreeableness over truth. They agree with wrong statements, build on flawed premises, and claim completion without verification.

Multiple reviewers with different training biases catch what a single agent misses. The goal is not disagreement - it's CONSENSUS found through TRUTH despite friction to IN/VALIDATE assertions to establish FACTS.

## Orchestrator Responsibilities

The orchestrator drives each cluster. It MUST:

1. Create worktree for the issue
2. Spawn implementer, capture output
3. Spawn reviewers (may be different number of turns with each)
4. Drive dialog until consensus on all significant concerns
5. Iterate implementation if needed
6. **Report back with:**
   - **Interaction diagram** - A line for each interaction showing the order of how cluster participants engaged (richer than a simple count)
   - What concerns were raised
   - How each was resolved (validated/invalidated with evidence)
   - Final outcome (success with commit, or escalation with unresolved items)

## The Roles

### Coordinator (the User or a special agent assigned by the User)

- Spawns orchestrators for each issue (parallel)
- Collects final results
- Reports to user
- Does NOT do the work itself

### Orchestrator (Separate process per issue)

- Owns one worktree
- Runs the implementation/review loop
- Drives dialog to consensus
- Reports detailed turn counts and resolutions

### Implementer

- Writes code
- Cites what was changed and why
- Receives iteration specs if changes needed

### Reviewers (3, cross-model)

- READ-ONLY guardians of quality
- Raise significant concerns with file:line evidence
- Reach consensus efficiently - not busywork
- Different models = different blind spots covered

## Evidence Requirements

Every assertion requires grounding:

- File:line citations for claims about code
- Specific examples for concerns
- Focus on what has significant impact or is a dependency

Opinions without evidence are dismissed. Evidence decides.

## What Success Means

**Success IS:**

- Issue resolved with high-quality, fact-based implementation
- All significant concerns resolved with evidence
- CONSENSUS found through TRUTH despite friction to IN/VALIDATE assertions to establish FACTS
- Orchestrator reports interaction diagram and resolution path
- Reusable knowledge captured from any failures-then-success patterns

**Acceptable escalation:**

- Unresolved disagreement on genuinely ambiguous requirements
- Better to escalate than produce slop

**Questions and Escalation:**

- Cluster agents can raise questions to the orchestrator
- Orchestrator answers or resolves with available context
- If orchestrator cannot resolve, escalate to user
- User escalation should not be common, but it's available when genuinely needed
- Don't guess - ask. Don't produce slop because a question wasn't asked.

## Limits (Safety Rails, Not Goals)

| Limit             | Value | Purpose                                      |
| ----------------- | ----- | -------------------------------------------- |
| MAX_ITERATIONS    | 3     | Escalate if stuck, don't loop forever        |
| MAX_DIALOG_ROUNDS | 5     | Escalate if deadlocked, don't debate forever |

These are safety rails for pathological cases. The goal is quality work in as many iterations as it takes, not fitting within bounds.

## Model Diversity

| Reviewer   | Model           | Why                                |
| ---------- | --------------- | ---------------------------------- |
| Reviewer-1 | Claude Opus 4.5 | Catches implementation gaps        |
| Reviewer-2 | GPT-5.2 high    | Different training, high reasoning |
| Reviewer-3 | GPT-5.2-codex   | Code-specialized perspective       |

These are capable models doing quality assurance, not artificial adversaries.

## Anti-Patterns

See `docs/anti-patterns.md`:

- One-shot without review
- Same-model review (no diversity)
- Disagreement for its own sake (busywork)
- Arbitrary turn limits as goals
- Not reporting interaction diagram and resolutions
- Not capturing reusable knowledge from failure patterns

## Self-Improvement Characteristic

**Every run produces TWO outputs:**

1. **The work itself** - Issue resolved, feature implemented, bug fixed
2. **Reusable knowledge** - Ultra-concise instructions that uplift all future agents

### Why This Matters

When an agent fails 3 times then succeeds on attempt 4, that pattern will be repeated by every future agent unless captured. Each repeated failure:
- Consumes context window
- Adds latency
- Wastes compute

Capturing the solution once → all future agents succeed on attempt 1 → **non-linear improvement**.

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

This is how the system improves itself. The cluster isn't just solving issues - it's **producing reusable knowledge that eliminates repeated failures**.

---

## Related

- `docs/orchestrator-flow.md` - Detailed flow diagrams
- `docs/spawn-cluster-spec.md` - Technical specification
- `docs/anti-patterns.md` - What not to do
- `docs/claims-evidence-limits.md` - What we've proven works
