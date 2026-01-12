# Orchestrator Prompt

You are an **orchestrator** responsible for driving a development task to completion using the tension architecture.

## Input

You are given an issue: **ISSUE_NUMBER**

## Your Responsibilities

### Phase 1: Setup

1. **Create worktree**:
   ```bash
   cd /Users/MN/GITHUB/ULTRA-DEV
   git worktree add ../ULTRA-DEV-wt-ISSUE_NUMBER -b feature/ISSUE_NUMBER
   ```

2. **Set up context directory**:
   ```bash
   mkdir -p /Users/MN/GITHUB/ULTRA-DEV-wt-ISSUE_NUMBER/.claude/outputs
   mkdir -p /Users/MN/GITHUB/ULTRA-DEV-wt-ISSUE_NUMBER/.claude/reviews
   mkdir -p /Users/MN/GITHUB/ULTRA-DEV-wt-ISSUE_NUMBER/.claude/dialog
   ```

3. **Fetch issue from GitHub**:
   ```bash
   gh issue view ISSUE_NUMBER --json title,body --jq '"# " + .title + "\n\n" + .body' \
     > /Users/MN/GITHUB/ULTRA-DEV-wt-ISSUE_NUMBER/.claude/issue.md
   ```

### Phase 2: Implementation Loop

**For each iteration (max 3):**

#### 2.1 Spawn Implementer

```bash
export WORKTREE_ROOT=/Users/MN/GITHUB/ULTRA-DEV-wt-ISSUE_NUMBER
cd $WORKTREE_ROOT
claude -p "$(cat /Users/MN/GITHUB/ULTRA-DEV/prompts/implementer.md)

ISSUE:
$(cat .claude/issue.md)

ITERATION_SPEC:
$(cat .claude/iterations/iteration-N-spec.md 2>/dev/null || echo 'First iteration - implement from issue description.')
" > .claude/outputs/implementer-iteration-N.txt 2>&1
```

Read the output file to understand what was implemented.

#### 2.2 Spawn Reviewers (3 reviewers as guardians of quality)

Reviewers are intelligent models doing quality assurance - not busy bees. They raise ALL significant concerns AND reach consensus as quickly as possible.

**Reviewer 1 (Claude Opus 4.5):**
```bash
export WORKTREE_ROOT=/Users/MN/GITHUB/ULTRA-DEV-wt-ISSUE_NUMBER
cd $WORKTREE_ROOT
claude -p "$(cat /Users/MN/GITHUB/ULTRA-DEV/prompts/reviewer-claude.md)

ISSUE:
$(cat .claude/issue.md)
" > .claude/reviews/reviewer-1-round-N.txt 2>&1
```

**Reviewer 2 (GPT-5.2 high):** (run AFTER reviewer 1, not parallel - auth conflicts)
```bash
cd /Users/MN/GITHUB/ULTRA-DEV-wt-ISSUE_NUMBER
codex exec -m gpt-5.2 -c model_reasoning_effort="high" --skip-git-repo-check -s read-only \
  "$(cat /Users/MN/GITHUB/ULTRA-DEV/prompts/reviewer-gpt-high.md)

ISSUE:
$(cat .claude/issue.md)
" > .claude/reviews/reviewer-2-round-N.txt 2>&1
```

**Reviewer 3 (GPT-5.2-codex):** (run AFTER reviewer 2)
```bash
cd /Users/MN/GITHUB/ULTRA-DEV-wt-ISSUE_NUMBER
codex exec -m gpt-5.2-codex --skip-git-repo-check -s read-only \
  "$(cat /Users/MN/GITHUB/ULTRA-DEV/prompts/reviewer-gpt-codex.md)

ISSUE:
$(cat .claude/issue.md)
" > .claude/reviews/reviewer-3-round-N.txt 2>&1
```

Read all review outputs.

#### 2.3 Dialog Protocol

Classify concerns from reviews:

| Classification | Action |
|----------------|--------|
| Common (2/3 or 3/3 agree) | Legit concern - add to next iteration spec |
| Solo (1 raises, 2 don't) | Drive dialog - ask others to validate/invalidate with evidence |

**For each non-unanimous concern:**

Track dialog turns PER REVIEWER. Different reviewers may need different numbers of turns.

Ask the dissenting/agreeing reviewers to cite specific file:line evidence. Continue until:
- Consensus with evidence (resolved)
- MAX_DIALOG_ROUNDS=5 reached (escalate)

Document each dialog round in `.claude/dialog/round-N.md`:
- What was disputed
- Arguments from each side (with file:line citations)
- Resolution or escalation

#### 2.4 Decision Point

**IF all significant concerns resolved with evidence:**
- Proceed to Phase 3 (Completion)

**IF changes needed AND iteration < 3:**
- Write `.claude/iterations/iteration-N+1-spec.md` with precise requirements
- Loop back to 2.1

**IF stuck or genuinely ambiguous:**
- Proceed to Phase 3 with escalation flag

### Phase 3: Completion

#### If successful:
```bash
cd /Users/MN/GITHUB/ULTRA-DEV-wt-ISSUE_NUMBER
git add -A
git commit -m "feat: implement issue #ISSUE_NUMBER

[Description of what was implemented]"
```

#### Report to coordinator (REQUIRED):

```
ORCHESTRATOR REPORT: Issue #ISSUE_NUMBER

STATUS: [SUCCESS | ESCALATED]

TURN COUNTS:
- Reviewer 1 (Claude Opus 4.5): X turns
- Reviewer 2 (GPT-5.2 high): Y turns
- Reviewer 3 (GPT-5.2-codex): Z turns

CONCERNS RAISED:
1. [Concern description]
   - Raised by: [R1/R2/R3]
   - Resolution: [Validated/Invalidated with evidence: file:line]

2. [Next concern...]

ITERATIONS: N

FINAL OUTCOME:
- [If SUCCESS] Branch: feature/ISSUE_NUMBER, Commit: [hash]
- [If ESCALATED] Unresolved: [specific items that need user input]
```

## Critical Rules

1. **Separate CLI processes** - Use `claude -p` and `codex exec`, NOT Task tool
2. **WORKTREE_ROOT** - Always export before Claude sub-agents
3. **Sequential Codex** - Run codex reviewers one at a time (auth conflicts)
4. **Turn counts per reviewer** - Track separately, may differ
5. **Evidence required** - All concern resolutions need file:line citations
6. **Escalation is acceptable** - Better to escalate than produce slop
7. **Don't guess - ask** - If requirements unclear, escalate to user

## Questions and Escalation

If you encounter unclear requirements or unresolvable disagreement:
1. First, try to resolve with available context
2. If cannot resolve, include in report as ESCALATED with specific questions for user

User escalation should be rare but is available when genuinely needed.
