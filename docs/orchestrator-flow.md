# Orchestrator Flow

## Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ USER                                                                        │
│   │                                                                         │
│   │ "Work on issues #1, #2, #3"                                             │
│   ▼                                                                         │
│ ┌─────────────────────────────────────────────────────────────────────────┐ │
│ │ COORDINATOR (Current Claude Session)                                    │ │
│ │                                                                         │ │
│ │  Spawns orchestrators in parallel (one per issue via Bash):             │ │
│ │                                                                         │ │
│ │  claude -p "Orchestrator for issue #1..." &  ───┐                       │ │
│ │  claude -p "Orchestrator for issue #2..." &  ───┼─── Run in parallel    │ │
│ │  claude -p "Orchestrator for issue #3..." &  ───┘                       │ │
│ │                                                                         │ │
│ └─────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
                              │
         ┌────────────────────┼────────────────────┐
         ▼                    ▼                    ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ ORCHESTRATOR #1 │  │ ORCHESTRATOR #2 │  │ ORCHESTRATOR #3 │
│ Issue #1        │  │ Issue #2        │  │ Issue #3        │
│ Worktree: wt-1  │  │ Worktree: wt-2  │  │ Worktree: wt-3  │
└────────┬────────┘  └────────┬────────┘  └────────┬────────┘
         │                    │                    │
         └────────────────────┴────────────────────┘
                              │
                    (Each runs independently)
                              │
                              ▼
```

## Inside Each Orchestrator

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ ORCHESTRATOR (for issue #N)                                                 │
│                                                                             │
│  1. CREATE WORKTREE                                                         │
│     ├── git worktree add ../repo-wt-N -b feature/N                          │
│     ├── mkdir -p .claude/outputs                                            │
│     └── gh issue view N → .claude/issue.md                                  │
│                                                                             │
│  2. IMPLEMENTATION LOOP                                                     │
│     │                                                                       │
│     ▼                                                                       │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ IMPLEMENTATION PHASE                                                        │
│                                                                             │
│  ORCHESTRATOR runs via Bash:                                                │
│       │                                                                     │
│       │ claude -p "Implement: [issue.md]" > .claude/outputs/implementer.txt │
│       ▼                                                                     │
│  ┌─────────────┐                                                            │
│  │ IMPLEMENTER │                                                            │
│  │ (Claude)    │                                                            │
│  └──────┬──────┘                                                            │
│         │                                                                   │
│         │ Returns: brief message of what was done (captured to file)        │
│         │                                                                   │
│         ▼                                                                   │
│  ORCHESTRATOR reads output file for work summary                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ REVIEW PHASE (3 reviewers: Claude parallel-safe, Codex sequential)          │
│                                                                             │
│  ORCHESTRATOR runs reviewers:                                               │
│       │                                                                     │
│       │ # Claude reviewer (can run while Codex runs)                        │
│       ├──► claude -p "Review..." > .claude/reviews/reviewer-1.txt           │
│       │         └── REVIEWER-1 (Claude Opus)                                │
│       │                                                                     │
│       │ # Codex reviewers SEQUENTIAL (not parallel - auth conflicts)        │
│       ├──► codex exec -m gpt-5.2 -c model_reasoning_effort="high" \         │
│       │      --skip-git-repo-check -s read-only "Review..."                 │
│       │         └── REVIEWER-2 (GPT-5.2 xhigh) → output to reviewer-2.txt   │
│       │                                                                     │
│       └──► codex exec -m gpt-5.2-codex --skip-git-repo-check \              │
│              -s read-only "Review..."                                       │
│                 └── REVIEWER-3 (GPT-5.2-codex) → output to reviewer-3.txt   │
│                                                                             │
│  ORCHESTRATOR writes outputs to .claude/reviews/*.txt                       │
│  (Reviewers are read-only; orchestrator captures their text output)         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ DIALOG PROTOCOL (Consensus Building)                                        │
│                                                                             │
│  ORCHESTRATOR evaluates all 3 reviews:                                      │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │ CONCERN CLASSIFICATION                                                 │ │
│  │                                                                        │ │
│  │  Common (3/3 or 2/3 agree):                                            │ │
│  │    → Legit concern, add to next iteration spec                         │ │
│  │                                                                        │ │
│  │  Idiosyncratic (2 raise, 1 doesn't):                                   │ │
│  │    → 3rd reviewer validates/invalidates with evidence                  │ │
│  │    → Dialog until consensus                                            │ │
│  │                                                                        │ │
│  │  Solo (1 raises, 2 don't):                                             │ │
│  │    → Other 2 validate/invalidate with evidence                         │ │
│  │    → Dialog until consensus                                            │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  DIALOG TURNS (as needed):                                                  │
│                                                                             │
│    ORCHESTRATOR: "R1 raised X, R2/R3 didn't. Validate with evidence."       │
│         │                                                                   │
│         ├──► R2: "I agree because [evidence] / I disagree because [evidence]"
│         └──► R3: "I agree because [evidence] / I disagree because [evidence]"
│         │                                                                   │
│         ▼                                                                   │
│    ORCHESTRATOR: Evaluates responses, may trigger more dialog               │
│         │                                                                   │
│         ▼                                                                   │
│    Continue until consensus on all concerns                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ DECISION POINT                                                              │
│                                                                             │
│  LIMITS: MAX_ITERATIONS=3, MAX_DIALOG_ROUNDS=5                              │
│                                                                             │
│  IF all reviewers APPROVE (no unresolved concerns):                         │
│    → Commit changes                                                         │
│    → Report success to COORDINATOR                                          │
│    → Exit loop                                                              │
│                                                                             │
│  IF changes needed AND iteration < MAX_ITERATIONS:                          │
│    → ORCHESTRATOR compiles spec from consensus                              │
│    → Precise requirements for next iteration                                │
│    → Resume IMPLEMENTER with spec (new claude -p call)                      │
│    → Loop back to REVIEW PHASE                                              │
│                                                                             │
│  IF iteration >= MAX_ITERATIONS:                                            │
│    → Report failure to COORDINATOR with unresolved concerns                 │
│    → Exit loop (escalate to user)                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ COMPLETION                                                                  │
│                                                                             │
│  ORCHESTRATOR reports to COORDINATOR:                                       │
│    - "Issue #N complete. Branch: feature/N. Commit: [hash]"                 │
│    - OR "Issue #N failed after N iterations. Unresolved: [details]"         │
│                                                                             │
│  COORDINATOR reports to USER:                                               │
│    - Summary of all clusters                                                │
│    - Which issues resolved, which need attention                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Agent Specifications

| Role | Model | CLI | Mode |
|------|-------|-----|------|
| COORDINATOR | Claude Opus | Current session | - |
| ORCHESTRATOR | Claude Opus | `claude -p` via Bash | Separate process |
| IMPLEMENTER | Claude Opus | `claude -p` via Bash | Can write |
| REVIEWER-1 | Claude Opus | `claude -p` via Bash | Read-only (instruction) |
| REVIEWER-2 | GPT-5.2 | `codex exec -s read-only` | Read-only (enforced) |
| REVIEWER-3 | GPT-5.2-codex | `codex exec -s read-only` | Read-only (enforced) |

**Why Bash instead of Task tool:** Task tool cannot set per-subagent WORKTREE_ROOT (see claims-evidence-limits.md). Separate CLI processes inherit environment variables.

## File Structure Per Worktree

```
repo-wt-N/
├── .claude/
│   ├── issue.md                    # Fetched from GitHub
│   ├── outputs/
│   │   └── implementer.txt         # Implementer output (each iteration)
│   ├── reviews/
│   │   ├── reviewer-1.txt          # Claude Opus review (orchestrator writes)
│   │   ├── reviewer-2.txt          # GPT-5.2 xhigh review (orchestrator writes)
│   │   └── reviewer-3.txt          # GPT-5.2-codex review (orchestrator writes)
│   ├── dialog/
│   │   └── round-N.txt             # Dialog protocol transcripts
│   └── iterations/
│       ├── iteration-1-spec.md     # What implementer was asked to do
│       └── iteration-2-spec.md     # Next iteration spec (if needed)
└── ... (repo files)
```

## Limits

| Limit | Value | Purpose |
|-------|-------|---------|
| MAX_ITERATIONS | 3 | Prevent infinite implementation loops |
| MAX_DIALOG_ROUNDS | 5 | Prevent infinite consensus debates |

## DIALOG Protocol Detail

When concerns are not unanimous:

1. **Identify disagreement type**
   - 2v1: Two reviewers agree, one doesn't
   - 1v2: One reviewer raises concern, two don't

2. **Request evidence-based validation**
   - Ask dissenting/agreeing reviewers to cite specific file:line
   - Ask for concrete examples of why concern is/isn't valid

3. **Evaluate arguments**
   - Weight evidence over opinion
   - Look for factual errors in either position

4. **Reach consensus**
   - If evidence supports concern → add to spec
   - If evidence refutes concern → dismiss with documented reasoning
   - If genuinely ambiguous → escalate to user (rare)

5. **Document in dialog/round-N.md**
   - What was disputed
   - Arguments from each side
   - Final decision and reasoning
