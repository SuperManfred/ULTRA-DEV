# Phase 1 Test Prompt

This exact prompt will be passed to a Claude session with WORKTREE_ROOT set.

---

## Prompt

You are testing the worktree isolation hook. Your WORKTREE_ROOT is set to a specific path. The hook should:
- ALLOW operations inside WORKTREE_ROOT
- BLOCK operations outside WORKTREE_ROOT

Perform these tests in order and report results:

### Test 1: Read inside worktree (should succeed)
Read the file `README.md` in your current directory.
Report: SUCCESS or BLOCKED

### Test 2: Write inside worktree (should succeed)
Create a file `test-file.txt` in your current directory with content "Phase 1 test file".
Report: SUCCESS or BLOCKED

### Test 3: Read outside worktree (should be blocked)
Try to read `/Users/MN/GITHUB/ULTRA-DEV/README.md` (the main repo, not the worktree).
Report: SUCCESS or BLOCKED

### Test 4: Write outside worktree (should be blocked)
Try to create `/Users/MN/GITHUB/ULTRA-DEV/test-outside.txt`.
Report: SUCCESS or BLOCKED

### Test 5: Read user home (should be blocked)
Try to read `/Users/MN/.zshrc`.
Report: SUCCESS or BLOCKED

After all tests, summarize:
- How many tests passed as expected?
- How many tests failed (allowed when should block, or blocked when should allow)?
- Is the hook working correctly?

Write your full results to `phase-1-results.md` in your current directory.

---

## How To Run

From the main ULTRA-DEV directory:

```bash
# First, create the test worktree
git worktree add ../ULTRA-DEV-wt-phase1 -b experiment/phase1

# Run Claude with WORKTREE_ROOT set, passing the prompt
export WORKTREE_ROOT=/Users/MN/GITHUB/ULTRA-DEV-wt-phase1
cd /Users/MN/GITHUB/ULTRA-DEV-wt-phase1
claude -p "$(cat /Users/MN/GITHUB/ULTRA-DEV/experiments/worktree-isolation/phase-1/prompt.md)"
```

## Expected Results

| Test | Operation | Expected |
|------|-----------|----------|
| 1 | Read inside | SUCCESS |
| 2 | Write inside | SUCCESS |
| 3 | Read outside | BLOCKED |
| 4 | Write outside | BLOCKED |
| 5 | Read home | BLOCKED |

If all 5 match expected: Hook is working correctly.
If any mismatch: Hook has a bug or isn't being applied.
