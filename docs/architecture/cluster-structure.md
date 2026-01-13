# Cluster Structure

**Priority 1: Quality through structured architecture** — Quality is a property of the cluster structure, not individual agents.

## Cluster Composition

Each cluster consists of:
- **1 Implementer** — Full write access, does the coding
- **3 Reviewers** — Read-only, guardians of quality

| Role | CLI | Isolation Mechanism | Access |
|------|-----|---------------------|--------|
| Implementer | `claude -p` | `WORKTREE_ROOT` + PreToolUse hook | Read/Write |
| Reviewer-1 | `claude -p` | `WORKTREE_ROOT` + PreToolUse hook | Read-only (instruction-based) |
| Reviewer-2 | `codex exec` | `-s read-only` sandbox | Read-only (enforced) |
| Reviewer-3 | `codex exec` | `-s read-only` sandbox | Read-only (enforced) |

**Note:** Codex reviewers run sequentially (not parallel) to avoid auth conflicts.

---

## The Roles

### Coordinator

The User or a special agent assigned by the User.

- Spawns orchestrators for each issue (parallel)
- Collects final results
- Reports to user
- **Does NOT do the work itself**

### Orchestrator

Separate process per issue. The orchestrator MUST:

1. Create worktree for the issue
2. Spawn implementer, capture output
3. Spawn reviewers (may be different number of turns with each)
4. Drive dialog until consensus on all significant concerns
5. Iterate implementation if needed
6. **Report back with:**
   - Interaction diagram showing order of engagements
   - What concerns were raised
   - How each was resolved (validated/invalidated with evidence)
   - Final outcome (success with commit, or escalation with unresolved items)

### Implementer

- Writes code
- Cites what was changed and why
- Receives iteration specs if changes needed

### Reviewers (3, cross-model)

- READ-ONLY guardians of quality
- Raise significant concerns with file:line evidence
- Reach consensus efficiently — not busywork
- Different models = different blind spots covered

---

## Model Diversity

| Reviewer | Model | Purpose |
|----------|-------|---------|
| Reviewer-1 | Claude Opus 4.5 | Catches implementation gaps |
| Reviewer-2 | GPT-5.2 high | Different training, high reasoning |
| Reviewer-3 | GPT-5.2-codex | Code-specialized perspective |

### Why Diversity Matters

**Proven empirically (Issue #9, 2026-01-13):**
- GPT-5.2 caught what Claude missed
- The issue was substantive and would have caused downstream problems
- Single-model review would have approved the flawed implementation

Different training produces genuinely different blind spots.

---

## Isolation Mechanisms

### Claude Agents (Implementer, Reviewer-1)

Uses environment variable + PreToolUse hook:

```bash
export WORKTREE_ROOT=/path/to/<repo>-wt-<issue-id>
cd /path/to/<repo>-wt-<issue-id>
claude -p "$(cat .claude/CLAUDE.local.md)"
```

The `enforce-worktree` hook (in `~/.claude/settings.json`) checks:
- Is target path inside `WORKTREE_ROOT`? → Approve
- Is target path outside `WORKTREE_ROOT`? → Block

### Codex Agents (Reviewer-2, Reviewer-3)

Uses built-in sandbox mode — **cwd IS the boundary**:

```bash
cd /path/to/<repo>-wt-<issue-id>
codex exec -m gpt-5.2 -c model_reasoning_effort="high" --skip-git-repo-check -s read-only "prompt"
```

The `-s read-only` flag restricts Codex to:
- No file writes
- Read access to working directory tree

No custom hook needed — isolation is architectural.

---

## Hard-Won Knowledge

### Task tool cannot test tension architecture

**Evidence:** Anti-patterns from 2026-01-12 experiment

Task tool runs agents within the current session — no way to set per-subagent WORKTREE_ROOT. Requires separate CLI processes (`claude -p`, `codex exec`).

### Conflating Coordinator with Orchestrator collapses architecture

- **Coordinator's job:** Spawn orchestrators, collect final results
- **Orchestrator's job:** Run the implementation loop with tension architecture

Mixing them = single-shot delegation = no review = useless.

### NEVER IMPLEMENT DIRECTLY

There is NO scenario where the orchestrator doing implementation produces a better outcome than a fresh implementer.

| Agent | Context | Capacity |
|-------|---------|----------|
| Fresh implementer | 100% available + focused prompt | MAXIMUM |
| Orchestrator | Already consumed + coordination overhead | LESS |

This is logic, not a rule. If implementer fails, the problem is the prompt or the task — fix THAT, retry with a fresh implementer, or escalate.

---

## Spawning Commands

**Implementer (Claude):**
```bash
export WORKTREE_ROOT=/path/to/<repo>-wt-<issue-id>
cd /path/to/<repo>-wt-<issue-id>
claude -p "$(cat .claude/CLAUDE.local.md)" > .claude/outputs/implementer.txt 2>&1
```

**Reviewer-1 (Claude):**
```bash
export WORKTREE_ROOT=/path/to/<repo>-wt-<issue-id>
cd /path/to/<repo>-wt-<issue-id>
claude -p "$(cat .claude/CLAUDE.local.reviewer-1.md)" > .claude/reviews/reviewer-1.txt 2>&1
```

**Reviewer-2 (GPT-5.2 high):** (run AFTER reviewer-1)
```bash
cd /path/to/<repo>-wt-<issue-id>
codex exec -m gpt-5.2 -c model_reasoning_effort="high" --skip-git-repo-check -s read-only "prompt" > .claude/reviews/reviewer-2.txt 2>&1
```

**Reviewer-3 (GPT-5.2-codex):** (run AFTER reviewer-2)
```bash
cd /path/to/<repo>-wt-<issue-id>
codex exec -m gpt-5.2-codex --skip-git-repo-check -s read-only "prompt" > .claude/reviews/reviewer-3.txt 2>&1
```

---

## Related

- [dialog-protocol.md](dialog-protocol.md) — How reviewers reach consensus
- [file-structure.md](file-structure.md) — `.claude/` directory layout
- [../vision.md](../vision.md) — The 6 priorities
