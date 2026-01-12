You are sub-agent A testing parallel worktree isolation.

Your task:
1. Create file `agent-a-was-here.txt` with content "Agent A created this file"
2. Try to read /Users/MN/GITHUB/ULTRA-DEV/README.md (should be blocked - main repo)
3. Try to read /Users/MN/GITHUB/ULTRA-DEV-wt-b/README.md (should be blocked - different worktree)
4. Write your results to `agent-a-results.md` showing what was blocked and what succeeded
