# Evidence Rules

**Priority 6 (Weight: 5)** — file:line citations. Verify ALL N items, not a sample. Never imply completeness.

## Core Principles

1. **Every assertion requires grounding**
2. **Never sample — verify ALL N items**
3. **Opinions without evidence are dismissed**

---

## Evidence Requirements

### For Code Claims

Every claim about code needs a file:line citation:

| Claim | Required Evidence |
|-------|-------------------|
| "Missing null check" | Cite exact file:line where check is missing |
| "Doesn't handle edge case X" | Cite where handling should be |
| "Acceptance criterion Y not met" | Cite what's missing and where |
| "This will fail when..." | Cite exact file:line and explain why |

### For Concerns

Every concern needs:
- **Evidence**: file:line citation
- **Impact**: Why this matters
- **Severity**: HIGH/MEDIUM

### For Reviews

Every acceptance criterion needs:
- **Status**: MET or NOT MET
- **Evidence**: file:line showing it's met (or where it's missing)

---

## Anti-Sampling Principle

**NEVER sample. If a claim covers N items, verify ALL N items.**

### Examples

| Sampling Failure | Correct Approach |
|------------------|------------------|
| "Tests pass" (ran 3 of 50) | Run ALL 50 tests |
| "All criteria met" (checked 2 of 7) | Check ALL 7 criteria |
| "Files look good" (read 1 of 5) | Read ALL 5 files |
| "Citations verified" (spot-checked 1 of 10) | Verify ALL 10 citations |

### If Full Verification Is Impractical

State explicitly: "Verified X of Y items"

**Never imply completeness.** If you can't verify all N, say so.

---

## Proven Value

**Issue #9 (2026-01-13):**
- Claude approved with sampling ("looks good")
- GPT-5.2 did full verification and found the gap
- The difference was **sampling vs. full verification**, not model intelligence

Sampling creates false confidence. The gap that GPT-5.2 found would have caused real problems.

---

## Detection of Sampling

The orchestrator should be suspicious of:
- Quick approvals with few citations
- Generic statements like "looks good" without file:line evidence
- Reviews that don't mention specific acceptance criteria
- Brief reviews that should have taken longer

**Challenge:** "You claim all criteria met. Cite file:line evidence for EACH criterion."

---

## Evidence Quality Verification

A citation must be **verified**, not just present:

1. **Spot-check** 2-3 citations from each review
2. Does the file:line actually support the claim made?
3. If fabricated or cherry-picked, call it out

**Example challenge:** "Citation at file.ext:42 doesn't support your claim. Provide valid evidence."

Never accept evidence at face value. Verify.

---

## Role-Specific Requirements

### Implementer

- Cite every file modified and what was changed
- Reference specific requirements from the issue
- State "FILES_READ" and "FILES_CREATED" explicitly

### Reviewer

- For EACH acceptance criterion: MET/NOT MET with file:line
- For EACH concern: severity, evidence, impact
- Never approve without substantive file:line citations

### Orchestrator

- Verify reviewer claims by spot-checking citations
- Ensure all concerns are resolved with evidence
- Track that ALL criteria were verified, not just some

---

## What Counts as Evidence

| Evidence Type | Example |
|---------------|---------|
| File:line citation | `src/auth.ts:42` |
| Command output | `npm test` output showing pass/fail |
| Git diff | `git diff` showing actual changes |
| Error message | Stack trace or error output |

| NOT Evidence | Why |
|--------------|-----|
| "I checked" | No verifiable citation |
| "Looks good" | No specific reference |
| "It works" | No proof provided |
| "I believe" | Opinion, not fact |

---

## Related

- [../vision.md](../vision.md) — Priority 6 overview
- [anti-patterns.md](anti-patterns.md) — Sampling as an anti-pattern
- [../architecture/dialog-protocol.md](../architecture/dialog-protocol.md) — Evidence in dialog
