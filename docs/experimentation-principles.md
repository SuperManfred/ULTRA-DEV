# Experimentation Principles

When testing agent workflows, knowing something works is not enough. We need to know WHY it works - the exact cause and effect chain. Without this, we can't debug failures, replicate successes, create reliable documentation or build reliable systems.

## Core Principles

### 1. Save All Inputs Before Execution

Every prompt, instruction, or configuration passed to an agent MUST be saved to a file BEFORE execution. Not after. Not a summary. The exact verbatim content.

**Why**: If an agent fails or behaves unexpectedly, we need to see exactly what it was given. A summary or reconstruction from memory isn't sufficient - subtle wording differences matter.

**How**:

- Save orchestrator prompts to `prompts/orchestrator-{id}.md`
- Orchestrators must save sub-agent prompts to `prompts/subagent-{orchestrator-id}-{subagent-role}.md`
- Include timestamp and any variable substitutions that were made

### 2. Log Exact Outputs

Agent outputs must be captured verbatim, not summarized.

**How**:

- Agents write their outputs to files: `outputs/{agent-id}-output.md`
- Include: what actions were taken, what files were created/modified, what the final state was
- For multi-turn agents: log each turn if possible

### 3. Test One Thing At A Time

Each experiment should isolate a single capability or configuration. If testing multiple things at once, a failure doesn't tell you which thing failed.

**Examples**:

- Experiment A: Test that WORKTREE_ROOT blocks file operations outside worktree
- Experiment B: Test that orchestrator can spawn sub-agents via separate Claude sessions
- Experiment C: Test that sub-agents can complete work and signal back to orchestrator

Only after each is proven individually do we combine them.

### 4. Define Success Before Running

Write down what success looks like BEFORE running the experiment. Not "it should work" but specific, verifiable criteria.

**Example**:

```
SUCCESS CRITERIA:
- [ ] File /path/to/expected/file.md exists
- [ ] File contains specific content (check with grep)
- [ ] No files were modified outside /path/to/worktree/
- [ ] Git log shows commit on correct branch
- [ ] Output file contains "APPROVED" from reviewer
```

### 5. Trace Cause and Effect

For any outcome (success or failure), we must be able to trace:

1. What exact input was given
2. What the agent did with that input
3. What output resulted

If we can't trace this chain, we haven't learned anything - we've just observed a black box.

### 6. Document the Recipe

A working experiment produces a recipe: the exact configuration and inputs needed to reproduce the result. This recipe becomes the reference implementation.

**Recipe includes**:

- Environment variables required
- Files that must exist before execution
- Exact prompts (verbatim, in files)
- Commands to execute
- Expected outputs and how to verify them

### 7. Failures Are Data

When something fails, the failure is valuable IF we can analyze it. This requires:

- The exact input that caused the failure
- The exact output/error that resulted
- The state of the system at time of failure

Don't discard failed experiments - document what was tried and why it failed.

---

## File Structure for Experiments

```
ULTRA-DEV/
├── experiments/
│   └── {experiment-name}/
│       ├── README.md           # What this experiment tests, success criteria
│       ├── prompts/            # All prompts saved BEFORE execution
│       │   ├── orchestrator.md
│       │   └── subagent-*.md   # Populated by orchestrator during execution
│       ├── outputs/            # All agent outputs
│       │   ├── orchestrator-output.md
│       │   └── subagent-*-output.md
│       ├── results/            # Final artifacts created by agents
│       └── analysis.md         # Post-experiment: what worked, what didn't, why
```

---

## Applying to Current Experiment

For the worktree isolation test:

1. **Save orchestrator prompt** to `experiments/worktree-isolation/prompts/orchestrator-a.md` (and b, c)
2. **Orchestrators must save** the exact prompts they pass to sub-agents
3. **Define success criteria** before running:
   - Each worktree created at correct path
   - Each template file created in correct worktree
   - No cross-worktree contamination
   - Commits on correct branches
4. **After experiment**: Document what worked, what didn't, exact cause for any failures
