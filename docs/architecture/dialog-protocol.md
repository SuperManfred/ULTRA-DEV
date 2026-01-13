# Dialog Protocol

**Priority 1: Quality through structured architecture** — Consensus building through truth, not artificial friction.

## Overview

Reviewers cannot see each other directly. The orchestrator is the communication hub, relaying exact quotes verbatim.

---

## Concern Classification

When orchestrator receives all 3 reviews:

| Classification | Condition | Action |
|----------------|-----------|--------|
| Common | 3/3 or 2/3 agree | Legit concern → add to next iteration spec |
| Idiosyncratic | 2 raise, 1 doesn't | 3rd reviewer validates/invalidates with evidence |
| Solo | 1 raises, 2 don't | Other 2 validate/invalidate with evidence |

Continue dialog until consensus with evidence or MAX_DIALOG_ROUNDS=5 (then escalate).

---

## Message Relay Format

**Orchestrator relays between reviewers:**
```
"Reviewer 1 says: [exact content verbatim]"
```

**Reviewers address each other via orchestrator:**
```
"Send to R2: [message with file:line evidence]"
```

**Rules:**
- Do NOT paraphrase — relay exact quotes
- Include all file:line citations verbatim
- Collate responses for efficiency, but preserve attribution

---

## Dialog Turn Example

```
ORCHESTRATOR: "R1 raised concern about missing error handling.
  Reviewer 1 says: 'file.ts:42 throws but caller at main.ts:15 has no try/catch.'
  R2 and R3, validate or invalidate with evidence."

R2: "Send to R1: Validated. main.ts:15 indeed has no error handling. The function
  at file.ts:42 can throw per line 44-46. This is a legitimate concern."

R3: "Send to R1 and R2: Partial agreement. file.ts:42 throws, but the throw
  is caught at file.ts:50 before it propagates. main.ts:15 is safe."

ORCHESTRATOR: "R3 found evidence at file.ts:50 that contradicts R1's concern.
  R1, validate R3's claim about line 50."
```

---

## Push-Harder-Before-Escalating Rule

Before escalating ANY disagreement to the user:

1. **Round 1**: Ask dissenting reviewer to provide file:line evidence
2. **Round 2**: If they maintain position, ask others to counter with evidence
3. **Minimum 2 rounds** of evidence-based back-and-forth required
4. Only escalate if genuinely irresolvable after pushback

**Bad:** "R1 and R2 disagree. Escalating to user."
**Good:** "R1 cites X, R2 counters with Y, R1 responds with Z, R2 concedes because Z proves the point."

Do NOT accept "I disagree" without evidence. Push for file:line citations.

### Minority-Correct Principle

If 2 reviewers disagree with 1, the 1 may still be correct.

Demand evidence from the 2, not just acceptance. Vote counts don't determine truth — **evidence decides**.

---

## Collusion Detection

Be suspicious of:
- All reviewers approving quickly without substantive concerns
- Reviewers agreeing without citing file:line evidence
- Reviews that look templated or minimal-effort

**If suspected, challenge:**
"Your review seems brief. Cite specific file:line for each acceptance criterion you verified."

Quick unanimous approval is a red flag, not a green light.

---

## Evidence Quality Verification

A citation must be VERIFIED, not just present:

1. **Spot-check** 2-3 citations from each review
2. Does the file:line actually support the claim made?
3. If fabricated or cherry-picked, call it out: "Citation at file.ext:42 doesn't support your claim. Provide valid evidence."

Never accept evidence at face value. Verify.

---

## Turn Count Tracking

Track dialog turns PER REVIEWER (may differ):
- R1: 3 turns
- R2: 5 turns
- R3: 2 turns

Different reviewers may need different amounts of dialog to reach consensus.

---

## Document Dialog

Each dialog round in `.claude/dialog/round-N.md`:
- What was disputed
- Arguments from each side
- Final decision and reasoning

---

## Related

- [cluster-structure.md](cluster-structure.md) — The roles and how they interact
- [../principles/evidence-rules.md](../principles/evidence-rules.md) — Evidence requirements
- [../vision.md](../vision.md) — The 6 priorities
