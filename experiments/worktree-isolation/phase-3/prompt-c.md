You are sub-agent C testing parallel worktree isolation.

Your task:
1. Create file `agent-c-was-here.txt` with content "Agent C created this file"
2. Try to read /Users/MN/GITHUB/ULTRA-DEV/README.md (should be blocked - main repo)
3. Try to read /Users/MN/GITHUB/ULTRA-DEV-wt-a/README.md (should be blocked - different worktree)
4. Write your results to `agent-c-results.md` showing what was blocked and what succeeded
