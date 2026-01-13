# Deterministic Validation First

**Priority 5 (Weight: 10)** — Catch mechanical errors (lint, typecheck, build, test) before AI reviewers engage.

## Core Principle

Every reviewer turn is high-value judgment. Don't waste it on errors a machine could catch.

---

## What to Validate Before Review

| Validation | Tool | Catches |
|------------|------|---------|
| Syntax | Linter (ESLint, Ruff, etc.) | Syntax errors, formatting issues |
| Types | Type checker (tsc, mypy, etc.) | Type mismatches, missing annotations |
| Build | Build tool (npm build, cargo build, etc.) | Compilation errors, missing dependencies |
| Tests | Test runner (jest, pytest, etc.) | Regression failures, broken functionality |

---

## Validation Sequence

Run BEFORE spawning reviewers:

```bash
# JavaScript/TypeScript
npm run lint && npm run typecheck && npm run build && npm run test

# Python
ruff check . && mypy . && pytest

# Rust
cargo clippy && cargo build && cargo test
```

**If any fail → implementer fixes first, then review.**

---

## Why This Matters

### Reviewer Cycles Are Expensive

Each reviewer turn involves:
- Model inference (compute cost)
- Context window consumption
- Latency

Don't spend these on:
- "Line 42 has a syntax error"
- "Type 'string' is not assignable to type 'number'"
- "Test 'should handle edge case' failed"

A linter catches these in milliseconds. A type checker catches these deterministically.

### Quality Is Higher

Reviewers can focus on:
- Design decisions
- Edge cases the tests don't cover
- Security implications
- Architecture concerns

Not mechanical errors that would fail CI anyway.

---

## Integration with Orchestrator

The orchestrator should:

1. After implementer completes → run deterministic validation
2. If validation fails → create new iteration spec with specific errors
3. Spawn fresh implementer to fix
4. Only proceed to reviewers when validation passes

```bash
# Pseudocode
npm run lint && npm run typecheck && npm run build && npm run test
if [ $? -ne 0 ]; then
  echo "VALIDATION FAILED - fix before review" >> .claude/iterations/iteration-N-spec.md
  # Spawn implementer to fix
else
  # Proceed to reviewers
fi
```

---

## What This Doesn't Replace

Deterministic validation catches mechanical errors. It does NOT catch:
- Logic errors
- Missing requirements
- Security vulnerabilities
- Design problems
- Edge cases not covered by tests

These still require AI reviewers.

---

## Hard-Won Knowledge

### Tests Must Exist First

Deterministic validation only works if:
- Tests exist for the code being changed
- Tests are comprehensive enough to catch regressions

If tests are missing, the implementer should write them as part of the task.

### CI Configuration Matters

If the project has CI:
- Run the same checks locally that CI would run
- Don't let reviewers waste time on something CI would catch

---

## Related

- [../vision.md](../vision.md) — Priority 5 overview
- [../architecture/cluster-structure.md](../architecture/cluster-structure.md) — Orchestrator responsibilities
- [resilient-execution.md](resilient-execution.md) — How to handle validation failures
