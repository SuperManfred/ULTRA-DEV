You are sub-agent B testing parallel worktree isolation.

Your task:
1. Create file `agent-b-was-here.txt` with content "Agent B created this file"
2. Try to read /Users/MN/GITHUB/ULTRA-DEV/README.md (should be blocked - main repo)
3. Try to read /Users/MN/GITHUB/ULTRA-DEV-wt-c/README.md (should be blocked - different worktree)
4. Write your results to `agent-b-results.md` showing what was blocked and what succeeded
