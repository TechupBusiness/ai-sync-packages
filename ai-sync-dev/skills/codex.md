---
name: codex
description: Use OpenAI Codex CLI as a co-programmer to get a second AI perspective on your code. Harden ideas, review implementations, find difficult bugs, and validate architectural decisions by consulting Codex as an independent expert. Use when you want feedback, a code review, a fresh perspective on a bug, or to stress-test your approach.
allowed-tools: Bash(codex:*)
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

Omit `-m` to use the default (`gpt-5.3-codex`). For deep thinking, add `-c 'model_reasoning_effort="xhigh"'`.

## Reasoning Effort

Control how deeply Codex thinks with `-c 'model_reasoning_effort="LEVEL"'`:

| Level | When to Use |
|-------|-------------|
| (omit) | Default — good for most tasks |
| `high` | Complex reviews, multi-file analysis |
| `xhigh` | Hardest bugs, architecture validation, security audits |

## Core Patterns

### 1. Code Review

Review uncommitted changes:

```bash
codex review --uncommitted
```

Review against a branch with custom focus:

```bash
codex review --base main "Focus on security vulnerabilities and error handling gaps"
```

### 2. Get Feedback on Specific Code

Pass file content to Codex for a second opinion. Keep context focused — pass relevant snippets, not entire files:

```bash
codex exec -s read-only -o /tmp/codex-feedback.md "Review this code for bugs, edge cases, performance issues, and simpler alternatives. Be direct and critical.

$(sed -n '1,100p' path/to/file.ts)"
```

Then read `/tmp/codex-feedback.md` with the Read tool.

### 3. Bug Investigation

When stuck on a difficult bug, use xhigh reasoning:

```bash
codex exec -s read-only \
  -c 'model_reasoning_effort="xhigh"' \
  -o /tmp/codex-analysis.md \
  "I have a bug with these symptoms: [DESCRIBE SYMPTOMS]

Relevant code:
$(sed -n '10,80p' path/to/suspect.ts)

Find the root cause. Consider race conditions, off-by-one errors, state mutations, and edge cases."
```

### 4. Architecture Validation

Before implementing, validate your approach:

```bash
codex exec -s read-only \
  -c 'model_reasoning_effort="xhigh"' \
  -o /tmp/codex-critique.md \
  "I'm planning this approach: [DESCRIBE PLAN]

Existing code context:
$(sed -n '1,80p' path/to/relevant/module.ts)

Critique this plan. What could go wrong? What am I missing? Suggest a better approach if you see one."
```

### 5. Harden an Implementation

After writing code, stress-test it:

```bash
codex exec -s read-only \
  -c 'model_reasoning_effort="high"' \
  -o /tmp/codex-hardening.md \
  "Act as a hostile code reviewer for this implementation:

$(sed -n '1,150p' path/to/implementation.ts)

Find: bugs that escape unit tests, inputs that break it, concurrency issues, security vulnerabilities.
Output a numbered list: finding, severity (critical/high/medium/low), and fix."
```

### 6. Compare Approaches

```bash
codex exec -s read-only -o /tmp/codex-comparison.md \
  "Compare these two approaches for [PROBLEM]:

Approach A: [DESCRIBE OR PASTE CODE]
Approach B: [DESCRIBE]

Which is better? Consider maintainability, performance, and correctness."
```

## Execution Rules

- Always use `codex exec` or `codex review` — never bare `codex` (interactive)
- Default to `-s read-only` — you are the implementer, Codex is the reviewer
- Capture output with `-o /tmp/codex-*.md`, then read it back with the Read tool
- Keep context focused: use `sed -n 'START,ENDp'` to pass relevant line ranges, not whole files
- Be specific about what kind of feedback you want
- Treat Codex output as suggestions — you make the final call

## Error Handling

If `codex` is not found:
- Install: `npm install -g @openai/codex` or `brew install codex`
- Login: `codex login`

If model errors occur:
- `gpt-5.3-codex-spark` requires ChatGPT Pro
- Fall back to `gpt-5.3-codex` (works with all paid ChatGPT plans)
- If model names change, check available models with `codex --help`
