---
name: codex
description: Use OpenAI Codex CLI as a co-programmer to get a second AI perspective on your code. Harden ideas, review implementations, find difficult bugs, and validate architectural decisions by consulting Codex as an independent expert. Use when you want feedback, a code review, a fresh perspective on a bug, or to stress-test your approach.
allowed-tools:
  - Bash(codex:*)
  - Bash(mktemp:*)
  - Bash(sed:*)
  - Bash(cat:*)
triggers:
  - "codex"
  - "second opinion"
  - "code review with codex"
  - "ask codex"
  - "codex review"
  - "fresh perspective"
---

# Codex Co-Programmer

Use OpenAI's Codex CLI as an independent co-programmer to harden your work.

## Models

| Model | Use For |
|-------|---------|
| `gpt-5.3-codex` | Deep review, architecture, complex bugs (default, best) |
| `gpt-5.3-codex-spark` | Fast iteration, quick checks (Pro only, near-instant) |

Omit `-m` to use the default. For deep thinking, add `-c 'model_reasoning_effort="xhigh"'`.

## Reasoning Effort

Control how deeply Codex thinks with `-c 'model_reasoning_effort="LEVEL"'`:

| Level | When to Use |
|-------|-------------|
| (omit) | Default — good for most tasks |
| `high` | Complex reviews, multi-file analysis |
| `xhigh` | Hardest bugs, architecture validation, security audits |

## Two Commands: `codex review` vs `codex exec`

These are different subcommands with different flag support:

| Feature | `codex review` | `codex exec` |
|---------|----------------|--------------|
| `-m MODEL` | No (use `-c model="MODEL"`) | Yes |
| `-s SANDBOX` | No | Yes |
| `--ephemeral` | No | Yes |
| `-o FILE` | No | Yes |
| `--json` | No | Yes |
| stdin pipe (`-`) | No | Yes |

Use `codex review` for quick diff-based reviews. Use `codex exec` for everything else.

## Core Patterns

### 1. Quick Code Review (codex review)

Review uncommitted changes:

```bash
codex review --uncommitted
```

Review against a branch with custom focus:

```bash
codex review --base main "Focus on security vulnerabilities and error handling gaps"
```

### 2. Deep Code Review (codex exec)

For more control (model, reasoning, output capture), use `codex exec`:

```bash
out=$(mktemp /tmp/codex-review.XXXXXX.md)
{ echo "Review this diff for bugs, edge cases, and security issues. For each finding provide: severity, file:line, evidence, and fix suggestion."; echo; git diff HEAD; } | codex exec -s read-only --ephemeral -o "$out" -
```

Then read the output file path with the Read tool.

### 3. Get Feedback on Specific Code

Pipe focused snippets via stdin — include file path and line numbers for context:

```bash
out=$(mktemp /tmp/codex-feedback.XXXXXX.md)
{ echo "Review path/to/file.ts lines 1-100 for bugs, edge cases, and simpler alternatives. Be direct and critical."; echo; sed -n '1,100p' path/to/file.ts; } | codex exec -s read-only --ephemeral -o "$out" -
```

### 4. Bug Investigation

When stuck on a difficult bug, use xhigh reasoning. Include symptoms, relevant code, test failures, and runtime errors:

```bash
out=$(mktemp /tmp/codex-analysis.XXXXXX.md)
{ cat <<'PROMPT'
I have a bug with these symptoms: [DESCRIBE SYMPTOMS]
Test failure / error output: [PASTE]

Relevant code:
PROMPT
sed -n '10,80p' path/to/suspect.ts
echo
echo "Find the root cause. Consider race conditions, off-by-one errors, state mutations, and edge cases."
} | codex exec -s read-only -c 'model_reasoning_effort="xhigh"' --ephemeral -o "$out" -
```

### 5. Architecture Validation

Before implementing, validate your approach:

```bash
out=$(mktemp /tmp/codex-critique.XXXXXX.md)
{ cat <<'PROMPT'
I'm planning this approach: [DESCRIBE PLAN]

Existing code context:
PROMPT
sed -n '1,80p' path/to/relevant/module.ts
echo
echo "Critique this plan. What could go wrong? What am I missing? Suggest a better approach if you see one."
} | codex exec -s read-only -c 'model_reasoning_effort="xhigh"' --ephemeral -o "$out" -
```

### 6. Harden an Implementation

After writing code, stress-test it:

```bash
out=$(mktemp /tmp/codex-hardening.XXXXXX.md)
{ echo "Act as a hostile code reviewer for this implementation:"; echo; sed -n '1,150p' path/to/implementation.ts; echo; cat <<'PROMPT'
Find: bugs that escape unit tests, inputs that break it, concurrency issues, security vulnerabilities.
Output a numbered list: finding, severity (critical/high/medium/low), file:line, evidence, and fix.
PROMPT
} | codex exec -s read-only -c 'model_reasoning_effort="high"' --ephemeral -o "$out" -
```

### 7. Compare Approaches

```bash
out=$(mktemp /tmp/codex-comparison.XXXXXX.md)
codex exec -s read-only --ephemeral -o "$out" "Compare these two approaches for [PROBLEM]:

Approach A: [DESCRIBE OR PASTE CODE]
Approach B: [DESCRIBE]

Which is better? Consider maintainability, performance, and correctness."
```

## Execution Rules

- Always use `codex exec` or `codex review` — never bare `codex` (interactive)
- Default to `-s read-only` for `codex exec` — you are the implementer, Codex is the reviewer
- Use `--ephemeral` to avoid session persistence (note: code is still sent to OpenAI's API)
- Capture output with `-o "$out"` using `mktemp`, then read it back with the Read tool
- For large context, pipe via stdin (`| codex exec ... -`) instead of inline `$(...)` substitution
- Keep context focused: include file paths, line numbers, relevant tests, and error output
- Be specific about what kind of feedback you want — request structured output (severity, evidence, fix)
- Treat Codex output as suggestions — you make the final call
- If output file is empty, the request likely failed — check stderr and retry

## Error Handling

If `codex` is not found:
- Install: `npm install -g @openai/codex` or `brew install codex`
- Login: `codex login`

If model errors occur:
- `gpt-5.3-codex-spark` requires ChatGPT Pro
- Fall back to default model (omit `-m` entirely)
- Check login status: `codex login`
