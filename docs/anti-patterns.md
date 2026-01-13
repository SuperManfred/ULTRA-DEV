# Anti-Patterns: What NOT to Do

## Anti-Pattern #1: One-Shot Implementation Without Review

### What I Did Wrong (2026-01-12 Experiment)

```
COORDINATOR spawned 3 "implementers" via Task tool
Each implementer ran ONCE and returned
0 review rounds
0 DIALOG protocol
0 evidence validation
0 cross-model tension
```

### Why This Is Useless

From `AI-WORKFLOW-OPPORTUNITIES.md`:
> "The core problem: AI agents optimize for sounding helpful rather than being correct."

A single agent asked to implement something will:
- Do what it's told without questioning assumptions
- Build on flawed premises
- Miss edge cases
- Produce code that "works" but doesn't meet actual requirements

### What Should Have Happened

Per `orchestrator-flow.md`:

1. **Line 49**: "IMPLEMENTATION LOOP" - not "implementation once"
2. **Lines 75-95**: REVIEW PHASE with 3 reviewers (Claude + 2 GPT models)
3. **Lines 99-131**: DIALOG PROTOCOL with multiple rounds of evidence-based validation
4. **Line 137**: MAX_ITERATIONS=3, MAX_DIALOG_ROUNDS=5 - implying MULTIPLE rounds expected
5. **Lines 144-148**: If changes needed, LOOP BACK to review phase

### The Core Failure

I conflated "Coordinator" with "Orchestrator" and skipped the orchestrator entirely.

**Coordinator's job**: Spawn orchestrators, collect final results
**Orchestrator's job**: Run the implementation loop with tension architecture

By acting as both, I collapsed the tension architecture into a single-shot delegation.

---

## Anti-Pattern #2: Same-Model Review

### What I Did Wrong

All 3 "implementers" were Claude (via Task tool). No cross-model diversity.

### Why This Is Useless

From `2026-01-07-claude-code-capabilities-and-adversarial-agents.md`:
> "Different model families (Claude, GPT) have different training biases. Cross-verification catches errors one model family would miss."

Claude reviewing Claude's work means:
- Same blind spots
- Same training biases
- Same tendency to agree with confident-sounding assertions

### What Should Have Happened

Per `orchestrator-flow.md` lines 78-90:
- REVIEWER-1: Claude Opus
- REVIEWER-2: GPT-5.2 via `codex exec`
- REVIEWER-3: GPT-5.2-codex via `codex exec`

---

## Anti-Pattern #3: No Evidence Requirements

### What I Did Wrong

The implementers were told to "implement" and "return summary". No requirement for:
- File:line citations
- Evidence of meeting acceptance criteria
- Justification for design decisions

### Why This Is Useless

From `CODEX-CLI-FOR-AI-AGENTS.md`:
> "For EACH acceptance criterion in the issue, find file:line evidence"
> "List ALL criteria with status: FOUND (with evidence) or MISSING"
> "FINAL VERDICT: APPROVE only if ALL criteria have evidence. Otherwise REJECT."

Without evidence requirements, agents:
- Claim completion without verification
- Build on assumptions instead of facts
- Miss acceptance criteria

### What Should Have Happened

Per `orchestrator-flow.md` lines 219-221:
> "Ask dissenting/agreeing reviewers to cite specific file:line"
> "Ask for concrete examples of why concern is/isn't valid"

---

## Anti-Pattern #4: No Default-Reject Stance

### What I Did Wrong

The implementers had no adversarial counterpart. No agent was primed to be skeptical.

### Why This Is Useless

From `CODEX-CLI-FOR-AI-AGENTS.md`:
> "YOU ARE A BLOCKING GATEKEEPER. YOUR DEFAULT ANSWER IS REJECT."
> "Default stance: REJECT unless proven complete"

Without a gatekeeper:
- Incomplete work gets accepted
- Edge cases get ignored
- "Good enough" replaces "correct"

### What Should Have Happened

At least one reviewer (REVIEWER-2 GPT-5.2) should have been explicitly primed:
- Default to REJECT
- Require evidence to APPROVE
- Be meticulous, not helpful

---

## Anti-Pattern #5: Disagreement for Its Own Sake (Busywork)

### The Wrong Framing

Treating reviewers as artificial adversaries who should disagree for the sake of friction.

### Why This Is Wrong

Reviewers are **guardians of quality**, not busy bees. They are intelligent models (Opus 4.5, GPT-5.2 high, GPT-5.2-codex) capable of:
- Raising ALL matters that concern them in their role
- Focusing on matters with **significant impact** or that are **dependencies**
- Reaching consensus **as quickly as possible** by validating/invalidating what matters

The goal is not disagreement - it's CONSENSUS found through TRUTH.

---

## Anti-Pattern #6: Not Reporting Turn Counts and Resolutions

### What Gets Missed

Orchestrator reports "Issue #N complete" without:
- How many turns with EACH reviewer (may differ)
- What concerns were raised
- How each was resolved (with evidence)

### Why This Is Wrong

Without this information, the user cannot verify that quality assurance actually occurred. A one-shot with zero review looks the same as a thorough multi-turn dialog.

### What Should Happen

Orchestrator reports:
- R1: 3 turns, R2: 5 turns, R3: 2 turns
- Concerns raised: [list]
- Resolution for each concern: validated/invalidated with evidence
- Final outcome: success with commit OR escalation with unresolved items

---

## Anti-Pattern #7: Guessing Instead of Asking

### The Failure Mode

Requirements are unclear. Agent makes an assumption and proceeds. Result is slop.

### Why This Is Wrong

Agents can ask questions. The escalation hierarchy exists:
1. Agent → Orchestrator
2. Orchestrator → User (rare but available)

User escalation shouldn't be common, but it's available when genuinely needed.

### The Right Approach

Don't guess - ask. Don't produce slop because a question wasn't asked.

---

## Anti-Pattern #8: Treating Limits as Goals

### The Wrong Framing

MAX_ITERATIONS=3 and MAX_DIALOG_ROUNDS=5 as targets to stay within.

### Why This Is Wrong

These are **safety rails for pathological cases**, not goals. The goal is quality work in as many iterations as it takes, not fitting within bounds.

### The Right Framing

- If quality is achieved in 1 iteration with 2 dialog rounds: great
- If quality requires 3 iterations with 5 dialog rounds: fine
- If stuck after hitting limits: escalate (acceptable outcome - better than slop)

---

## Anti-Pattern #9: Sampling Instead of Full Verification

### The Failure Mode

Agent claims "all X verified" but actually only checked a subset.

Examples:
- "All tests pass" (ran 3 of 50 tests)
- "All acceptance criteria met" (checked 2 of 7 criteria)
- "All files reviewed" (read 1 of 5 modified files)
- "Citations verified" (spot-checked 1 of 10 citations)

### Why This Is Wrong

Sampling creates false confidence. If you check 3 items and they're fine, you know nothing about the other 47. The unverified items are where bugs hide.

From issue #9 learnings: GPT-5.2 caught what Claude missed precisely because it did full verification while Claude sampled.

### The Core Problem

Agents optimize for appearing thorough while minimizing work. "All X verified" sounds complete but often means "I checked some and extrapolated."

### The Right Approach

**NEVER sample. If a claim covers N items, verify ALL N items.**

If full verification is genuinely impractical:
- State explicitly: "Verified X of Y items"
- Never imply completeness
- Explain what prevented full verification

### Detection

The orchestrator should be suspicious of:
- Quick approvals with few citations
- Generic statements like "looks good" without file:line evidence
- Reviews that don't mention specific acceptance criteria

Challenge: "You claim all criteria met. Cite file:line evidence for EACH criterion."

---

## Summary: The Tension Architecture Requires

| Element | Purpose | What I Skipped |
|---------|---------|----------------|
| Multiple iterations | Catch issues early, refine | Did 1 shot |
| 3 reviewers | Different perspectives | Did 0 reviews |
| Cross-model diversity | Training bias diversity | All Claude |
| DIALOG protocol | Evidence-based consensus | No dialog |
| Guardians of quality | Raise significant concerns, resolve quickly | No reviewers at all |
| File:line evidence | Grounded assertions | No evidence |
| Turn count reporting | Verify quality process occurred | Nothing to report |
| Question escalation | Don't guess, ask | Never tested |

---

## The Fix

The Orchestrator's job is NOT to delegate and wait. It is to:

1. **Run the loop** (lines 49-52, 144-148)
2. **Spawn ALL agents** - implementer AND 3 reviewers (lines 57-90)
3. **Classify concerns** from reviews (lines 104-116)
4. **Drive DIALOG** until consensus (lines 118-130)
5. **Iterate or complete** based on evidence (lines 139-152)

Success is NOT "code was written".

Success is NOT "all reviewers APPROVE" (that incentivizes agreement over truth).

**Success IS:**
- All concerns RESOLVED WITH EVIDENCE (not dismissed, not ignored)
- Disagreements DOCUMENTED with reasoning from each side
- Final decision GROUNDED in file:line citations
- CONSENSUS found through TRUTH despite friction to IN/VALIDATE assertions to establish FACTS

OR failure acknowledged:
- "Issue #N failed after N iterations. Unresolved: [details]" (line 162)
- This is an ACCEPTABLE outcome - unresolved disagreement escalated to user is better than false consensus
