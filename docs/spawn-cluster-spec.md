# spawn-cluster Specification

## Purpose

Automate the creation of a worktree and spawning of agent sessions for a development task. This is the "one command" that kicks off a cluster of agents working on a GitHub issue.

## Cluster Structure

Each cluster consists of:
- **1 Implementer** - Full write access, does the coding
- **3 Reviewers** - Read-only, **guardians of quality**

Reviewers are intelligent models (Opus 4.5, GPT-5.2 high, GPT-5.2-codex) doing quality assurance - not busy bees. They raise ALL significant concerns AND reach consensus as quickly as possible by validating/invalidating what matters.

| Role | CLI | Isolation Mechanism | Access |
|------|-----|---------------------|--------|
| Implementer | `claude -p` | `WORKTREE_ROOT` + PreToolUse hook | Read/Write |
| Reviewer-1 | `claude -p` | `WORKTREE_ROOT` + PreToolUse hook | Read-only (instruction-based) |
| Reviewer-2 | `codex exec` | `-s read-only` sandbox | Read-only (enforced) |
| Reviewer-3 | `codex exec` | `-s read-only` sandbox | Read-only (enforced) |

**Note:** Codex reviewers run sequentially (not parallel) to avoid auth conflicts.

## Inputs

```
spawn-cluster <repo-path> <issue-id> [options]
```

| Input | Required | Description |
|-------|----------|-------------|
| `repo-path` | Yes | Path to the main repository |
| `issue-id` | Yes | GitHub issue number (fetched via `gh issue view`) |
| `--prompt-file` | No | Override: use local file instead of fetching from GitHub |
| `--claude-only` | No | Use Claude for all 3 agents (Reviewer-2 uses same gatekeeper template) |

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

### Codex Agent (Reviewer-2)

Uses built-in sandbox mode - **cwd IS the boundary**:

```bash
cd /path/to/<repo>-wt-<issue-id>
codex review --base main -s read-only "$(cat .claude/CLAUDE.local.reviewer-2.md)"
```

The `-s read-only` flag restricts Codex to:
- No file writes
- Read access to working directory tree

No custom hook needed - isolation is architectural.

## What It Does

1. **Check worktree doesn't exist**
   ```bash
   if [ -d "../<repo>-wt-<issue-id>" ]; then
     echo "ERROR: Worktree already exists at ../<repo>-wt-<issue-id>"
     echo "Another cluster may be working on issue #<issue-id>"
     exit 1
   fi
   ```

2. **Create worktree**
   ```bash
   cd <repo-path>
   git worktree add ../<repo>-wt-<issue-id> -b feature/<issue-id>
   ```

3. **Create context directories**
   ```bash
   mkdir -p ../<repo>-wt-<issue-id>/.claude/outputs
   ```

4. **Fetch issue from GitHub** (or use --prompt-file override)
   ```bash
   # Default: fetch from GitHub
   gh issue view <issue-id> --json title,body --jq '"# " + .title + "\n\n" + .body' \
     > ../<repo>-wt-<issue-id>/.claude/issue.md

   # Override with --prompt-file
   cp <prompt-file> ../<repo>-wt-<issue-id>/.claude/issue.md
   ```

5. **Generate role files** (before execution)
   - `.claude/CLAUDE.local.md` for Implementer
   - `.claude/CLAUDE.local.reviewer-1.md` for Reviewer-1
   - `.claude/CLAUDE.local.reviewer-2.md` for Reviewer-2

6. **Spawn agent sessions**

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
   claude -p "$(cat .claude/CLAUDE.local.reviewer-1.md)" > .claude/outputs/reviewer-1.txt 2>&1
   ```

   **Reviewer-2 (Codex):**
   ```bash
   cd /path/to/<repo>-wt-<issue-id>
   codex review --base main -s read-only "$(cat .claude/CLAUDE.local.reviewer-2.md)" > .claude/outputs/reviewer-2.txt 2>&1
   ```

7. **Capture outputs**
   - All outputs go to `.claude/outputs/`

## Outputs

### Directory Structure Created

```
<repo>-wt-<issue-id>/
├── .claude/
│   ├── issue.md                    # Task/issue description
│   ├── CLAUDE.local.md             # Implementer role file
│   ├── CLAUDE.local.reviewer-1.md  # Reviewer-1 role file
│   ├── CLAUDE.local.reviewer-2.md  # Reviewer-2 role file
│   └── outputs/
│       ├── implementer.txt         # Captured AFTER execution
│       ├── reviewer-1.txt
│       └── reviewer-2.txt
├── ... (repo files)
```

### Exit Behavior

- Returns 0 if all agents complete (success or failure - both are valid outcomes)
- Returns non-zero if worktree creation fails or agents can't be spawned

## Role Templates

### Implementer (Claude)

```markdown
# Role: IMPLEMENTER
# Cluster: <issue-id>

## First Action
Read .claude/issue.md and confirm you understand the requirements.

## Your Contract
Before writing ANY code:
1. Summarize what you'll build
2. List files you expect to modify
3. State any assumptions you're making
4. Then proceed with implementation

If requirements are unclear, state your interpretation and proceed. Do not wait for human input.

Prefix all responses with: "The Implementer says: "
```

### Reviewer-1 (Claude)

```markdown
# Role: PR-REVIEWER-1 (Claude)
# Cluster: <issue-id>

## First Action
Read .claude/issue.md and prepare your review criteria.

## Your Review
1. Identify files that were modified
2. Check each acceptance criterion from the issue
3. List potential edge cases or issues found
4. Provide file:line evidence for any concerns

You are READ-ONLY. Do not modify files.

FINAL VERDICT (required): APPROVED or NEEDS_CHANGES with specific reasons.

Prefix all responses with: "PR Reviewer 1 says: "
```

### Reviewer-2 (Codex)

```markdown
# Role: PR-REVIEWER-2 (Codex/GPT-5)
# Cluster: <issue-id>

## First Action
Read .claude/issue.md and prepare your review criteria.

## Your Preparation
1. Identify files you expect to see modified
2. Note acceptance criteria you'll verify
3. List potential edge cases to check
4. Be ready for forensic-level review

YOU ARE A BLOCKING GATEKEEPER. YOUR DEFAULT ANSWER IS REJECT.

REVIEW CHECKLIST:
1. For EACH acceptance criterion in the issue, find file:line evidence
2. List ALL criteria with status: ✅ FOUND (with evidence) or ❌ MISSING
3. Check for security vulnerabilities, missing error handling, missing tests
4. FINAL VERDICT: APPROVE only if ALL criteria have evidence. Otherwise REJECT.

Prefix all responses with: "PR Reviewer 2 says: "
```

## Orchestrator Reporting Requirements

When the orchestrator completes, it MUST report:

1. **Turn count with EACH reviewer** (may differ - e.g., R1: 3 turns, R2: 5 turns, R3: 2 turns)
2. **What concerns were raised**
3. **How each concern was resolved** (validated/invalidated with evidence)
4. **Final outcome**: success with commit, or escalation with unresolved items

Escalation is an acceptable outcome. Better to escalate than produce slop.

## Questions and Escalation

Agents can ask questions when requirements are unclear or concerns can't be resolved:

1. **Agent** raises question to **Orchestrator**
2. **Orchestrator** resolves with available context if possible
3. If **Orchestrator** cannot resolve → escalate to **User**

User escalation should be rare but is available. Don't guess - ask.

## What "Done" Means

The script is done when:
1. Worktree exists at expected path
2. Branch exists with expected name
3. Issue file copied to `.claude/issue.md`
4. Role files generated in `.claude/`
5. Agent sessions have completed (not necessarily succeeded)
6. Outputs are captured in `.claude/outputs/`
7. **Orchestrator has reported turn counts and resolution paths**

The script does NOT:
- Verify agents completed their task correctly
- Merge anything to main
- Clean up the worktree

Those are separate operations.

## Anti-Patterns

| Anti-Pattern | Why It's Bad |
|--------------|--------------|
| Two implementers in same worktree | Write conflicts, no clear ownership |
| Parallel Codex invocations | Auth conflicts, unpredictable ordering |
| Reviewer with write access | Defeats the purpose of independent review |
| Skipping issue.md | Agents have no shared understanding of task |
| Infinite agent loops | Always include termination conditions |
| **One-shot without review** | Tests nothing about tension architecture |
| **Disagreement for its own sake** | Reviewers are guardians, not busy bees |
| **Not reporting turn counts** | User can't verify quality process occurred |
| **Guessing instead of asking** | Produces slop when question could resolve |
| **Treating limits as goals** | Limits are safety rails, not targets |

## Example Usage

```bash
# Standard cluster (1 implementer + 2 reviewers) for issue #123
spawn-cluster /Users/MN/GITHUB/myrepo 123 --prompt-file ./issue-123.md

# Claude-only cluster (no Codex)
spawn-cluster /Users/MN/GITHUB/myrepo 456 --prompt-file ./task.md --claude-only
```

## Success Criteria for Implementation

- [ ] Errors clearly if worktree already exists
- [ ] Creates worktree at correct path
- [ ] Creates branch with correct name
- [ ] Fetches issue from GitHub (or uses --prompt-file override)
- [ ] Creates `.claude/outputs/` directory
- [ ] Generates 3 role files from templates
- [ ] Exports WORKTREE_ROOT for Claude agents
- [ ] Spawns Claude implementer with `claude -p`
- [ ] Spawns Claude reviewer-1 with `claude -p`
- [ ] Spawns Codex reviewer-2 with `codex review -s read-only` (or Claude if --claude-only)
- [ ] Captures all agent outputs to files
- [ ] Implementer runs first, reviewers run after (sequential)

## Dependencies

- `git` (for worktree operations)
- `gh` CLI (for fetching GitHub issues)
- `claude` CLI (for Claude agent sessions)
- `codex` CLI (for Codex reviewer, unless `--claude-only`)
- `enforce-worktree` hook installed in `~/.claude/settings.json`

## Not In Scope (Future)

- Coordination protocols between agents (iteration loops)
- Automatic PR creation
- Worktree cleanup
- Agent resumption via agentId
