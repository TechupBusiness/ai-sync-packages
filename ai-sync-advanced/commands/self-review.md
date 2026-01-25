---
name: self-review
description: Run verification and self-review checklist
---

## Self-Review

### 1. Run Verification

```bash
pnpm format:check
pnpm typecheck 2>&1 | tail -n 40
pnpm lint 2>&1 | head -n 50
pnpm test 2>&1 | tail -n 40
```

Run each command separately. Limit output with `head`/`tail` to reduce tokens. Fix all errors before proceeding.

### 2. Check Against Project Standards - Worklist

IMPORTANT!!! Review your work against:
- .ai-flow/learnings.md - Past mistakes
- .ai-flow/tasks/Txxx.md - Current Task requirements

**Code Quality:**
- [ ] File structure matches spec
- [ ] Copied patterns from referenced "Prior Art" files
- [ ] Uses `Result<T, E>` for errors (no throwing)
- [ ] No `any` types, no `@ts-ignore`

**Completeness:**
- [ ] All edge cases from task spec handled
- [ ] Error messages are actionable (what + how to fix)
- [ ] Exports added to barrel `index.ts`
- [ ] README updated (if public API changed)

**Tests:**
- [ ] Each error path has a test
- [ ] Tests verify root cause, not symptoms
- [ ] No `test.skip` or `test.todo` left behind

**Documentation:**
- [ ] !!! IMPORTANT !!! Update `.ai-flow/plan.md` if task status changed or new tasks identified
- [ ] Let user review before executon: Update remaining tasks in `.ai-flow/plan.md` and `.ai-flow/tasks/Txxx.md` to fit to our changes
- [ ] Update `.docs/*.md` if architecture, Content Types or their behavior, or platforms changed
- [ ] Update `README.md` where needed

### 3. Report

```markdown
**Implemented:** [1-2 sentences]
**Deviations from spec:** [List or "None"]
**Uncertain about:** [List or "None"]
**Commands:** typecheck ✅/❌ | lint ✅/❌ | format ✅/❌ | test ✅/❌
**Docs updated:** .ai-flow/plan.md ✅/❌ | .ai-flow/tasks/Txxx.md ✅/❌ | docs/*.md ✅/❌ | README.md ✅/❌
```

### 4. Action

- **All pass** → "Ready for review"
- **Issues found** → Fix if quick (<5min), otherwise flag and ask
- Use `pnpm format` and `pnpm lint --fix` to auto-fix issues before manual fixes

---

**🚫 Common Mistakes to Catch:**
| Mistake | How to Detect |
|---------|---------------|
| Symptom fix | Test passes even without your core fix |
| Missing edge case | Spec lists it, but no test exists |
| New pattern | Doesn't match existing similar files |
| Over-engineering | Added abstraction not in spec |
| Stale docs | plan.md/README.md and doc/*.md don't reflect current state |
| format or lint errors occur| Didn't run `pnpm format` and `pnpm lint --fix` first |