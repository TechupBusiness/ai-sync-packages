---
name: tdd
description: Test-driven development workflow with red-green-refactor cycle and commit checkpoints
---

## TDD Workflow

Implement a feature or fix using strict test-driven development. Follow the red-green-refactor cycle with commit gates between phases. **Do not skip phases or combine them.**

### Arguments

The user provides a description of **what** to build or fix. You determine the test cases.

---

### Phase 1: RED — Write Failing Tests

**Goal:** Define the expected behavior before writing any implementation.

1. **Analyze the requirement** — break it into discrete, testable behaviors
2. **Write test cases** for each behavior:
   - Happy path first
   - Edge cases and error conditions
   - One assertion per test where practical
3. **Run the tests** — verify they **fail** for the right reason (missing function, wrong return value — not syntax errors or import failures)

```bash
pnpm test <test-file> 2>&1 | tail -n 30
```

**Checkpoint:** All tests fail with meaningful failure messages. The failures describe the behavior gap, not broken test code.

4. **Commit the failing tests:**

```bash
git add <test-files>
git commit -m "test: add tests for <feature> (red phase)"
```

**Do NOT proceed until tests are committed.**

---

### Phase 2: GREEN — Minimum Implementation

**Goal:** Make every test pass with the simplest possible code.

1. **Write the minimum code** to make tests pass — nothing more
   - Hardcoded returns are acceptable if they pass the test
   - No optimization, no cleanup, no "while I'm here" changes
   - If you feel the urge to write more, stop — that's refactor phase
2. **Run the tests** — verify they **all pass**

```bash
pnpm test <test-file> 2>&1 | tail -n 30
```

3. **Run lint and typecheck** to catch obvious issues:

```bash
pnpm lint 2>&1 | head -n 30
pnpm typecheck 2>&1 | tail -n 30
```

**Checkpoint:** All tests pass. Code may be ugly — that's fine.

4. **Commit the passing implementation:**

```bash
git add <impl-files>
git commit -m "feat: implement <feature> (green phase)"
```

**Do NOT proceed until implementation is committed.**

---

### Phase 3: REFACTOR — Clean Up Under Protection

**Goal:** Improve code quality without changing behavior. Tests are your safety net.

1. **Review the implementation** for:
   - Duplication that can be extracted
   - Names that could be clearer
   - Structure that could be simpler
   - Patterns that don't match the rest of the codebase
2. **Refactor in small steps** — run tests after each change
3. **If tests break, undo the last change** — don't debug the refactor, revert it

```bash
pnpm test <test-file> 2>&1 | tail -n 30
```

**Checkpoint:** All tests still pass. Code is clean.

4. **Commit if anything changed:**

```bash
git add <changed-files>
git commit -m "refactor: clean up <feature>"
```

---

### Phase 4: ASSESS — Next Cycle or Done?

Review what's been built:

- **More behaviors to implement?** → Return to Phase 1 with the next test case
- **All behaviors covered?** → Run the full test suite to check for regressions:

```bash
pnpm test 2>&1 | tail -n 40
```

---

### Rules of Engagement

| Rule | Why |
|------|-----|
| Never write implementation before a failing test | The test defines the requirement |
| Never write more than the minimum to pass | Resist the urge to "finish" the feature in green phase |
| Commit between every phase | Each commit is a safe rollback point |
| One behavior per cycle | Small cycles catch bugs early |
| If green phase feels hard, the test is too big | Split the test into smaller behaviors |

### Report

After completing all cycles:

```markdown
**Feature:** [what was built]
**Cycles:** [number of red-green-refactor cycles]
**Tests:** [number of test cases]
**Commits:** [list of commit hashes with phase labels]
**Regressions:** [any existing tests that broke, and how resolved]
```
