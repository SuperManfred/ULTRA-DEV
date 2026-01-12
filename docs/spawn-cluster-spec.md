# spawn-cluster Specification

## Purpose

Automate the creation of a worktree and spawning of agent sessions for a development task. This is the "one command" that kicks off a cluster of agents working on a GitHub issue.

## Inputs

```
spawn-cluster <repo-path> <issue-id> [options]
```

| Input | Required | Description |
|-------|----------|-------------|
| `repo-path` | Yes | Path to the main repository |
| `issue-id` | Yes | GitHub issue number or identifier |
| `--agents` | No | Number of agents to spawn (default: 1) |
| `--prompt-file` | No | Path to prompt file for agents |

## What It Does

1. **Create worktree**
   ```bash
   cd <repo-path>
   git worktree add ../<repo>-wt-<issue-id> -b feature/<issue-id>
   ```

2. **Create prompt directory**
   ```bash
   mkdir -p ../<repo>-wt-<issue-id>/.cluster/prompts
   ```

3. **Save agent prompts** (before execution)
   - Copy or generate prompt to `.cluster/prompts/agent-1.md`
   - If multiple agents, `.cluster/prompts/agent-2.md`, etc.

4. **Spawn agent sessions**
   ```bash
   export WORKTREE_ROOT=/path/to/<repo>-wt-<issue-id>
   cd /path/to/<repo>-wt-<issue-id>
   claude -p "$(cat .cluster/prompts/agent-1.md)" > .cluster/outputs/agent-1.txt 2>&1
   ```

5. **Capture outputs**
   - Agent outputs go to `.cluster/outputs/agent-N.txt`

## Outputs

### Directory Structure Created

```
<repo>-wt-<issue-id>/
├── .cluster/
│   ├── prompts/
│   │   └── agent-1.md      # Saved BEFORE execution
│   └── outputs/
│       └── agent-1.txt     # Captured AFTER execution
├── ... (repo files)
```

### Exit Behavior

- Returns 0 if all agents complete (success or failure - both are valid outcomes)
- Returns non-zero if worktree creation fails or agents can't be spawned

## What "Done" Means

The script is done when:
1. Worktree exists at expected path
2. Branch exists with expected name
3. Prompts are saved in `.cluster/prompts/`
4. Agent sessions have completed (not necessarily succeeded)
5. Outputs are captured in `.cluster/outputs/`

The script does NOT:
- Verify agents completed their task correctly
- Merge anything to main
- Clean up the worktree

Those are separate operations.

## Example Usage

```bash
# Single agent working on issue #123
spawn-cluster /Users/MN/GITHUB/myrepo 123

# Three agents with custom prompt
spawn-cluster /Users/MN/GITHUB/myrepo 456 --agents 3 --prompt-file ./task.md
```

## Success Criteria for Implementation

- [ ] Creates worktree at correct path
- [ ] Creates branch with correct name
- [ ] Saves prompts before agent execution
- [ ] Exports WORKTREE_ROOT correctly
- [ ] Spawns agent with `claude -p`
- [ ] Captures agent output to file
- [ ] Works for single agent
- [ ] Works for multiple agents in parallel

## Dependencies

- `git` (for worktree operations)
- `claude` CLI (for agent sessions)
- enforce-worktree hook installed in `~/.claude/settings.json`

## Not In Scope (Future)

- GitHub API integration (fetching issue details)
- Role-based agents (implementer/reviewer differentiation)
- Coordination protocols between agents
- Automatic PR creation
- Worktree cleanup
