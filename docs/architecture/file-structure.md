# File Structure

**Priority 2: Hyper-efficient resilient execution** — Externalizing state to files enables checkpointing, recovery, and context management.

## Per-Worktree Structure

```
repo-wt-N/
├── .claude/
│   ├── issue.md                    # Fetched from GitHub (task description)
│   ├── outputs/
│   │   └── implementer-iteration-N.txt   # Implementer output per iteration
│   ├── reviews/
│   │   ├── reviewer-1-round-N.txt        # Claude Opus review per round
│   │   ├── reviewer-2-round-N.txt        # GPT-5.2 high review per round
│   │   └── reviewer-3-round-N.txt        # GPT-5.2-codex review per round
│   ├── dialog/
│   │   └── round-N.md                    # Dialog protocol transcripts
│   ├── iterations/
│   │   ├── iteration-1-spec.md           # Initial spec (from issue)
│   │   └── iteration-N-spec.md           # Refined spec from review feedback
│   └── skills/
│       └── {descriptive-name}.md         # Captured learnings
└── ... (repo files)
```

---

## File Purposes

### issue.md

The task description, fetched from GitHub:

```bash
gh issue view <issue-id> --json title,body --jq '"# " + .title + "\n\n" + .body' > .claude/issue.md
```

Or copied from a local file:
```bash
cp <prompt-file> .claude/issue.md
```

### outputs/

Captured output from each implementer iteration. Named by iteration number:
- `implementer-iteration-1.txt`
- `implementer-iteration-2.txt`
- etc.

### reviews/

Captured output from each reviewer per round. Named by reviewer and round:
- `reviewer-1-round-1.txt` (Claude Opus, round 1)
- `reviewer-2-round-1.txt` (GPT-5.2 high, round 1)
- `reviewer-3-round-1.txt` (GPT-5.2-codex, round 1)
- `reviewer-1-round-2.txt` (Claude Opus, round 2 if dialog needed)
- etc.

### dialog/

Dialog protocol transcripts. Documents:
- What was disputed
- Arguments from each side (verbatim quotes)
- Final decision and reasoning

### iterations/

Iteration specs that drive each implementation round:
- `iteration-1-spec.md` — Usually just references issue.md
- `iteration-2-spec.md` — Refined requirements from review feedback
- etc.

### skills/

Captured learnings from failure-then-success patterns. Each skill is its own file:
- `{descriptive-name}.md`

---

## Why This Structure

### Checkpointing

Each iteration's work is saved before proceeding. If an agent fails:
1. Cluster detects the failure
2. Can resume from last checkpoint (latest output files)
3. No progress lost

### Context Management

Fresh implementers can read only what they need:
- Issue: `.claude/issue.md`
- Current spec: `.claude/iterations/iteration-N-spec.md`
- Previous output (if relevant): `.claude/outputs/implementer-iteration-N-1.txt`

### Auditability

Complete trace of what happened:
- Every implementer output
- Every review
- Every dialog turn
- Every iteration spec

### Self-Improvement

Skills directory enables:
- Capturing learnings per-worktree
- Promoting valuable skills to main repo
- Future agents benefiting from past failures

---

## Creation Sequence

The launcher (`bin/orchestrate`) creates:

```bash
mkdir -p .claude/{outputs,reviews,dialog,iterations,skills}
gh issue view <issue-id> --json title,body --jq '...' > .claude/issue.md
```

Then the orchestrator:
1. Reads `.claude/issue.md`
2. Spawns implementer → writes to `.claude/outputs/implementer-iteration-1.txt`
3. Spawns reviewers → write to `.claude/reviews/reviewer-{1,2,3}-round-1.txt`
4. If dialog needed → writes to `.claude/dialog/round-1.md`
5. If iteration needed → writes `.claude/iterations/iteration-2-spec.md`
6. Repeat until consensus
7. Capture skills → `.claude/skills/*.md`

---

## Related

- [cluster-structure.md](cluster-structure.md) — The roles that create these files
- [dialog-protocol.md](dialog-protocol.md) — What goes in dialog/ files
- [../execution/resilient-execution.md](../execution/resilient-execution.md) — Context management
