# Launcher Operations

How to start orchestrators for GitHub issues.

---

## Quick Start

**Single issue:**
```bash
bin/orchestrate 18
```

**Multiple issues in parallel:**
```bash
bin/orchestrate-batch 9 10 11 12 13
```

---

## What `bin/orchestrate` Does

1. **Validates issue exists** via `gh issue view`
2. **Creates worktree** at `../ULTRA-DEV-wt-<issue-number>` with branch `feature/<issue-number>`
3. **Creates context directories:**
   ```
   .claude/
   ├── outputs/
   ├── reviews/
   ├── dialog/
   └── iterations/
   ```
4. **Fetches issue** from GitHub to `.claude/issue.md`
5. **Writes `CLAUDE.local.md`** with orchestrator context (see below)
6. **Opens new Terminal window** via osascript with:
   - `WORKTREE_ROOT` exported
   - Interactive `c` session started

---

## The CLAUDE.local.md Pattern

The launcher injects the orchestrator prompt into `CLAUDE.local.md` at the worktree root. Claude reads this automatically when the session starts.

**Contents:**
- Quick reference for the orchestrator
- The full issue text from `.claude/issue.md`
- The complete orchestrator prompt from `prompts/orchestrator.md`
- Reminder: **DO NOT COMMIT** - produce artifacts only

**Why this pattern:**
- `claude -p` has length limits
- CLAUDE.local.md is read automatically
- Keeps prompts in version control but injects at runtime
- Substitutes `ISSUE_NUMBER` dynamically

---

## Hook Installation

The `enforce-worktree` hook restricts file operations to `WORKTREE_ROOT`.

### Install

Add to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write|Read|Bash",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/ULTRA-DEV/hooks/enforce-worktree"
          }
        ]
      }
    ]
  }
}
```

Replace `/path/to/ULTRA-DEV` with actual path (e.g., `/Users/MN/GITHUB/ULTRA-DEV`).

### How It Works

| Condition | Action |
|-----------|--------|
| `WORKTREE_ROOT` not set | Approve all (normal session) |
| Target inside `WORKTREE_ROOT` | Approve |
| Target outside `WORKTREE_ROOT` | Block with error message |

### Scope

| Tool | Enforcement |
|------|-------------|
| Read, Write, Edit | Path checked against `WORKTREE_ROOT` |
| Bash | Approved (not a security boundary) |

**Note:** This prevents accidental cross-worktree contamination, not malicious escape. Bash is not restricted.

---

## Worktree Lifecycle

### Creation
```bash
bin/orchestrate 18
# Creates: ../ULTRA-DEV-wt-18
# Branch: feature/18
```

### During Work
- All state in `.claude/` directory
- Orchestrator coordinates implementer + reviewers
- Changes accumulate in worktree

### After Completion
```bash
# Review changes
cd ../ULTRA-DEV-wt-18
git status
git diff

# If approved, commit and merge
git add -A
git commit -m "feat: implement issue #18"
git checkout main
git merge feature/18

# Cleanup
git worktree remove ../ULTRA-DEV-wt-18
git branch -d feature/18
```

---

## Files

| File | Purpose |
|------|---------|
| `bin/orchestrate` | Launch single orchestrator |
| `bin/orchestrate-batch` | Launch multiple orchestrators |
| `hooks/enforce-worktree` | PreToolUse hook for isolation |
| `prompts/orchestrator.md` | Orchestrator instructions |

---

## Related

- [../architecture/cluster-structure.md](../architecture/cluster-structure.md) — Roles and isolation
- [../execution/parallel-processing.md](../execution/parallel-processing.md) — Worktree isolation details
- [../architecture/file-structure.md](../architecture/file-structure.md) — `.claude/` directory layout
