# Resilient Execution

**Priority 2 (Weight: 27)** — Maximum efficiency and effectiveness, capable of hours/overnight when task complexity demands.

## Core Principle

Agents do NOT die from poor context management. The cluster prevents this.

---

## How Agents Die

Agents fail when:
1. **Context exhaustion** — Too much context, can't fit new work
2. **State loss** — Agent crashes, no way to resume
3. **Infinite loops** — No termination condition, runs forever
4. **Unrecoverable errors** — Error with no recovery path

The cluster architecture prevents ALL of these.

---

## Context Management

### Calibrate Work to Fit Context

Before spawning an agent, estimate context needs:
- How many files need to be read?
- How large are those files?
- How complex is the task?

If too large for single agent → break into smaller tasks.

### Fresh Implementer is the Default

**This is a requirement, not an optimization.**

| Scenario | Action |
|----------|--------|
| Heavy context consumption | Spawn fresh |
| Light context, unrelated next phase | Spawn fresh |
| Light context, same context needed | May reuse |
| Uncertain | Spawn fresh |

**Logic:**
- Fresh implementer: 100% context available + focused prompt = MAXIMUM capacity
- Reused implementer: Consumed context + coordination overhead = LESS capacity

After each implementer completes, read its output for:
- `FILES_READ`: What files it consumed
- `FILES_CREATED`: What it produced

Use this to decide whether to reuse or spawn fresh.

---

## State Externalization

All state lives in `.claude/`:

```
.claude/
├── issue.md                    # Task description
├── outputs/
│   └── implementer-iteration-N.txt   # What was done
├── reviews/
│   └── reviewer-{1,2,3}-round-N.txt  # Review feedback
├── dialog/
│   └── round-N.md                    # Consensus building
├── iterations/
│   └── iteration-N-spec.md           # Requirements per iteration
└── skills/
    └── *.md                          # Captured learnings
```

Nothing is in-memory only. Everything is persisted.

---

## Checkpointing

Each phase writes before proceeding:

1. **Implementer completes** → writes to `outputs/implementer-iteration-N.txt`
2. **Reviewer completes** → writes to `reviews/reviewer-X-round-N.txt`
3. **Dialog round completes** → writes to `dialog/round-N.md`
4. **Iteration spec created** → writes to `iterations/iteration-N-spec.md`

If any agent fails mid-work:
- Previous checkpoints are intact
- Can resume from last completed phase

---

## Cluster Self-Correction

When an agent fails unexpectedly:

### Detection
- Agent spawn times out
- Agent returns error
- Agent output is empty or malformed

### Recovery

1. **Retry once** with same parameters
2. **If second failure**, try alternative:
   - Different model flag
   - Shorter prompt
   - Simpler task breakdown
3. **If third failure**, continue with remaining agents and note the gap
4. **NEVER** accept failure silently or skip without trying

Document all retry attempts in `.claude/dialog/`.

---

## Intelligent Retry

When a reviewer spawn times out or fails:

```
Attempt 1: Same parameters
  ↓ Failed
Attempt 2: Same parameters (transient issue?)
  ↓ Failed
Attempt 3: Alternative approach (different model, shorter prompt)
  ↓ Failed
Continue: Note gap in report, proceed with remaining reviewers
```

Three attempts before giving up. Never silent failure.

---

## Termination Conditions

Safety rails prevent infinite execution:

| Limit | Value | Action When Hit |
|-------|-------|-----------------|
| MAX_ITERATIONS | 3 | Escalate to user |
| MAX_DIALOG_ROUNDS | 5 | Escalate to user |

These are NOT goals. Quality is the goal. But they prevent pathological loops.

---

## Human Involvement

Human is involved for:
- **Genuine judgment calls** — Ambiguous requirements, design decisions
- **Unresolvable disagreements** — After push-harder-before-escalating exhausted

Human is NOT involved for:
- **Technical failures** — Cluster handles these
- **Transient errors** — Retry logic handles these
- **Context issues** — Fresh agent spawning handles these

---

## Overnight Execution

For complex tasks requiring hours of work:

1. **Break into phases** — Each phase fits in agent context
2. **Checkpoint after each phase** — All state externalized
3. **Monitor for stuck conditions** — Safety rails catch infinite loops
4. **Resume from checkpoint if needed** — No progress lost

The cluster can run unattended because:
- State is externalized
- Failures are handled
- Termination is guaranteed

---

## Related

- [../vision.md](../vision.md) — Priority 2 overview
- [../architecture/file-structure.md](../architecture/file-structure.md) — Where state lives
- [../architecture/cluster-structure.md](../architecture/cluster-structure.md) — Fresh implementer principle
