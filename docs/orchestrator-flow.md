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
│ │  Spawns orchestrators in parallel (one per issue):                      │ │
│ │                                                                         │ │
│ │  Task(prompt="Orchestrator for issue #1...")  ───┐                      │ │
│ │  Task(prompt="Orchestrator for issue #2...")  ───┼─── Run in parallel   │ │
│ │  Task(prompt="Orchestrator for issue #3...")  ───┘                      │ │
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
│  ORCHESTRATOR                                                               │
│       │                                                                     │
│       │ Task(prompt="Implement: [issue.md]")                                │
│       ▼                                                                     │
│  ┌─────────────┐                                                            │
│  │ IMPLEMENTER │                                                            │
│  │ (Claude)    │                                                            │
│  └──────┬──────┘                                                            │
│         │                                                                   │
│         │ Returns: brief message of what was done                           │
│         │                                                                   │
│         ▼                                                                   │
│  ORCHESTRATOR receives work summary                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ REVIEW PHASE (3 independent reviewers in parallel)                          │
│                                                                             │
│  ORCHESTRATOR spawns all 3 simultaneously:                                  │
│       │                                                                     │
│       ├──► REVIEWER-1 (Claude Opus `c`)                                     │
│       │         └── Returns: brief message + .claude/reviews/reviewer-1.md  │
│       │                                                                     │
│       ├──► REVIEWER-2 (GPT-5.2 xhigh via Codex)                             │
│       │         └── Returns: brief message + .claude/reviews/reviewer-2.md  │
│       │                                                                     │
│       └──► REVIEWER-3 (GPT-5.2-codex via Codex)                             │
│                 └── Returns: brief message + .claude/reviews/reviewer-3.md  │
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
│  IF all reviewers APPROVE (no unresolved concerns):                         │
│    → Commit changes                                                         │
│    → Report success to COORDINATOR                                          │
│    → Exit loop                                                              │
│                                                                             │
│  IF changes needed:                                                         │
│    → ORCHESTRATOR compiles spec from consensus                              │
│    → Precise requirements for next iteration                                │
│    → Resume IMPLEMENTER with spec                                           │
│    → Loop back to REVIEW PHASE                                              │
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
| ORCHESTRATOR | Claude Opus | Task tool | Sub-agent |
| IMPLEMENTER | Claude Opus | Task tool | Sub-agent, can write |
| REVIEWER-1 | Claude Opus | Task tool (`c`) | Read-only |
| REVIEWER-2 | GPT-5.2 | Codex CLI xhigh | Read-only |
| REVIEWER-3 | GPT-5.2-codex | Codex CLI | Read-only |

## File Structure Per Worktree

```
repo-wt-N/
├── .claude/
│   ├── issue.md                    # Fetched from GitHub
│   ├── reviews/
│   │   ├── reviewer-1.md           # Claude Opus review
│   │   ├── reviewer-2.md           # GPT-5.2 xhigh review
│   │   └── reviewer-3.md           # GPT-5.2-codex review
│   ├── dialog/
│   │   └── round-N.md              # Dialog protocol transcripts
│   └── iterations/
│       ├── iteration-1-spec.md     # What implementer was asked to do
│       └── iteration-2-spec.md     # Next iteration spec (if needed)
└── ... (repo files)
```

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
