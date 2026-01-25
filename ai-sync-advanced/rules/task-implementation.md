---
name: task-implementation
description: Guidelines for implementing tasks from .ai-flow/
globs:
  - ".ai-flow/**/*.md"
---

# Task Implementation Guidelines

## Task Spec Structure

Task specifications live in `.ai-flow/tasks/`. Before implementing:

1. **Read the task spec thoroughly** - especially Prior Art and Critical Logic sections
2. **Read `.ai-flow/learnings.md`** - avoid repeating past mistakes
3. **Check `.ai-flow/plan.md`** - understand dependencies

## Before Writing Code

### 1. Search for Existing Implementations

```bash
# Check if similar functionality exists
grep -r "functionName" src/
grep -r "ClassName" src/

# Check for existing types
grep -r "interface.*MyType" src/parsers/types.ts
grep -r "type.*MyType" src/capabilities/
```

### 2. Review Central Definitions

| What | Where | Action |
|------|-------|--------|
| Content types | `src/capabilities/matrix.types.ts` | Import `ContentType` |
| Target types | `src/parsers/types.ts` | Import `TargetType`, `ALL_TARGETS` |
| Platform extensions | `src/parsers/types.ts` | Check existing before creating |
| Discovery patterns | `src/cli/commands/migrate.ts` | Use `DISCOVERY_PATTERNS` |
| Target config | `targets/*.yaml` | Check terminology, output dirs |
| Capability matrix | `src/capabilities/matrix.ts` | Check `FIELD_CAPABILITIES` |

### 3. Check Prior Art from Task Spec

Task specs include "Prior Art" sections pointing to files to copy patterns from:

```markdown
## 2. Prior Art (IMPORTANT)
- `src/cli/commands/migrate.ts` - Copy discovery pattern from here
- `src/parsers/rule.ts` - Copy parser structure from here
```

**ALWAYS read these files before implementing.**

## Implementation Checklist

From task spec section 5.1 "Must Follow":

- [ ] Use `Result<T, E>` for all fallible operations
- [ ] Use `@/` path aliases (never relative paths)
- [ ] Use conditional spread for optional properties
- [ ] Export only from `index.ts`
- [ ] Run `npm run lint` before completing
- [ ] Run `npm run typecheck` before completing

## Common Mistakes to Avoid

From `.ai-flow/learnings.md`:

### Exact Optional Properties
```typescript
// ❌ WRONG
const opts: Options = { field: maybeValue ?? undefined };

// ✅ CORRECT
const opts: Options = {
  ...(maybeValue !== undefined ? { field: maybeValue } : {}),
};
```

### Duplicate Type Definitions
```typescript
// ❌ WRONG - Type already exists
type ContentType = 'rule' | 'agent';

// ✅ CORRECT - Import existing
import type { ContentType } from '@/capabilities/matrix.types.js';
```

### Hardcoded Platform-Specific Values
```typescript
// ❌ WRONG - Hardcoding platform paths
const droidsDir = '.factory/droids';

// ✅ CORRECT - Use target mapping
const mapping = await loadTargetMapping('factory');
const agentsDir = mapping.output.agents_dir;
```

## After Implementation

1. **Run quality checks:**
   ```bash
   pnpm lint
   pnpm typecheck
   pnpm test
   ```

2. **Update exports** in relevant `index.ts`

3. **Update plan** - Mark task as complete in `.ai-flow/plan.md`

4. **Add learnings** - If you discovered new patterns or pitfalls, add to `.ai-flow/learnings.md`

## Task Completion Criteria

From task spec section 8:

- [ ] Format check passes
- [ ] All tests pass
- [ ] No lint errors
- [ ] Types are strict (no `any`)
- [ ] Exports added to barrel files
- [ ] Plan updated
