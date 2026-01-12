# ULTRA-DEV Agent Guidelines

This repo builds infrastructure for multi-agent development workflows.

## Context

This is tooling that OTHER agents will use. When building here:

- Think about how the tools will be used by agents in worktrees
- Test isolation mechanisms thoroughly - a bug here breaks all worktree sessions
- Hooks must fail safe (approve if uncertain, never block legitimate work)

## Key Files

- `bin/spawn-cluster` - Creates worktree, sets env vars, starts agents
- `hooks/enforce-worktree` - PreToolUse hook for path isolation
- `templates/` - Role-specific CLAUDE.local.md files

## Testing

Before considering any hook complete:

1. Test with WORKTREE_ROOT unset - must approve everything
2. Test with WORKTREE_ROOT set - must block outside, approve inside
3. Test edge cases: relative paths, symlinks, git commands

## Integration

This repo's hooks integrate with ~/.claude/settings.json. Changes here affect the global Claude configuration. Be careful.
