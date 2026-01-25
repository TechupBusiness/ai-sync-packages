---
name: create-task
description: Create implementation specification via interview
---

## Task: Create Implementation Specification

Use interview style to clarify all questions interactively with the user! Also involve appropriate agents if they exist.

### Context Files (Read First)

- `docs/*.md` - Project overview, architecture
- `.ai-flow/plan.md` - Tasks, completed work
- `.ai-flow/learnings.md` - Critical lessons from past implementations
- `.ai-flow/tasks/Txxx-*.md` - Related task specs for pattern reference (optional, if relevant)

### Your Job

Create a detailed implementation spec that enables a less-skilled AI to produce senior-level code without additional guidance.

### Output

**File**: `.ai-flow/tasks/Txxx-short-description.md`

- `Txxx` = Task number from plan.md (e.g., T050, T100, T336)
- `short-description` = Kebab-case summary (e.g., `user-auth`, `rest-api`, `rbac`)
- Examples: `T050-user-auth.md`, `T100-rbac.md`, `T336-rename-simulation.md`

---

## Task Spec Template

```md
# Txxx: [Task Title]

> **Depends on**: Txxx, Txxx (or "None")

## 1. Objective

One paragraph: What this task achieves and why it matters.

## 2. Prior Art (IMPORTANT)

Reference existing implementations to copy patterns from:

- `path/to/similar/file` - Copy X pattern from here
- `path/to/another/file` - Use same error handling approach

## 3. File Structure

Show what files will be created/modified:

\`\`\`
src/                          # Or your project's source folder
├── feature/
│   ├── index.ext             # Public exports (if applicable)
│   ├── feature.ext           # Main implementation
│   ├── feature.types.ext     # Types/interfaces (if applicable)
│   └── feature.test.ext      # Tests
\`\`\`

## 4. Interfaces & Types

Define ALL public interfaces/contracts. Adapt to your language:

\`\`\`
// Only show non-obvious types, reference existing for common patterns
// Examples: interfaces (TS), structs (Go/Rust), classes (Python), protocols (Swift)

FeatureOptions {
  field1: type  // Document each field's purpose
  field2: type
}
\`\`\`

## 5. Implementation Requirements

### 5.1 Must Follow (Project Patterns)

- [ ] Follow project's error handling pattern (see `path/to/error-handling`)
- [ ] Use project's logging/observability approach
- [ ] Follow module export conventions
- [ ] [Add project-specific patterns from learnings.md]

### 5.2 Critical Logic (Pitfall Prevention)

Only document non-obvious logic or where past mistakes occurred:

\`\`\`
// ❌ WRONG - This caused bug in Txxx
doSomething(input)

// ✅ CORRECT - Always validate/wrap/handle X
validated = validate(input)
doSomething(validated)
\`\`\`

### 5.3 Edge Cases to Handle

- [ ] Empty input → Return appropriate empty/default value
- [ ] Invalid format → Return error with clear message
- [ ] [List all edge cases explicitly]

## 6. Test Requirements

### 6.1 Test File Structure

\`\`\`
// Adapt to your testing framework (Jest, pytest, go test, etc.)
describe/test_group('FeatureName') {
  test('happy path') { /* ... */ }
  test('error handling') { /* ... */ }
  test('edge cases') { /* ... */ }
}
\`\`\`

### 6.2 Required Test Cases

- [ ] Basic functionality with valid input
- [ ] Each error condition from 5.3
- [ ] Integration with dependent modules
- [ ] [Reference similar test file for patterns]

## 7. Integration Points

- **Imports from**: `path/to/dependency` (use `functionName`)
- **Exports to**: Will be used by Txxx
- **Config**: Add to config schema if needed

## 8. Acceptance Criteria

- [ ] Code formatting passes (project's formatter)
- [ ] All tests pass
- [ ] No lint errors
- [ ] Types are strict (no `any`/`unknown`/untyped where applicable)
- [ ] Exports added to public API
- [ ] `plan.md` is updated (mark task complete)
- [ ] `README.md` updated if necessary
- [ ] `docs/*.md` updated if applicable

## 9. Out of Scope

- Feature X (handled in Txxx)
- Optimization Y (future task)
```

---

## Guidelines for Writing Specs

### DO Include

- Exact interface definitions (the contract)
- References to existing code to copy patterns from
- Specific pitfalls from `learnings.md` relevant to this task
- Non-obvious logic with code snippets
- All edge cases as a checklist

### DO NOT Include

- Obvious boilerplate code
- Full function implementations (unless tricky)
- Explanations of basic language/framework patterns
- Code that follows established patterns without deviation

### Quality Check Before Saving

- [ ] Could a junior dev implement this without asking questions?
- [ ] Are all project patterns explicitly referenced?
- [ ] Are relevant lessons from `learnings.md` incorporated?
- [ ] Is every edge case listed?
- [ ] Are test cases specific enough to verify correctness?

---

## Example: TypeScript/Node.js Task Spec

<details>
<summary>Click to expand TypeScript-specific example</summary>

```md
# T050: User Authentication

> **Depends on**: T040, T045

## 1. Objective

Implement JWT-based user authentication with login/logout endpoints.

## 2. Prior Art (IMPORTANT)

- `src/middleware/validation.ts` - Copy validation middleware pattern
- `src/utils/result.ts` - Use Result<T, E> for error handling

## 3. File Structure

\`\`\`
src/
├── auth/
│   ├── index.ts          # Public exports only
│   ├── auth.service.ts   # Main implementation
│   ├── auth.types.ts     # Types/interfaces
│   └── auth.test.ts      # Tests
\`\`\`

## 4. Interfaces & Types

\`\`\`typescript
interface LoginRequest {
  email: string;
  password: string;
}

interface AuthResult {
  token: string;
  expiresAt: Date;
}
\`\`\`

## 5. Implementation Requirements

### 5.1 Must Follow (Project Patterns)

- [ ] Use `Result<T, E>` for errors (see `src/utils/result.ts`)
- [ ] Use `AuthError` class extending `BaseError`
- [ ] Export only from `index.ts`

### 5.2 Critical Logic

\`\`\`typescript
// ❌ WRONG - Token in query string (logged in URLs)
app.get('/api/data?token=xxx')

// ✅ CORRECT - Token in Authorization header
headers: { Authorization: 'Bearer xxx' }
\`\`\`

### 5.3 Edge Cases

- [ ] Empty credentials → Return `Result.err(ValidationError)`
- [ ] Invalid password → Return `Result.err(AuthError)` (no details)
- [ ] Expired token → Return `Result.err(TokenExpiredError)`

## 8. Acceptance Criteria

- [ ] `npm run format:check` passes
- [ ] `npm test -- auth.test.ts` passes
- [ ] `npm run lint` passes
- [ ] No `any` types
```

</details>
