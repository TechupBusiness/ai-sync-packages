---
name: create-plan
description: Generate a development task plan
---

# Project Plan Generation Prompt

## Execution Approach

Perform an **extensive analysis** of this project before creating the plan. If your environment has specialist agents (e.g., architect, security, UX, performance), delegate analysis to them and synthesize their input.

> **Tip**: If an orchestrator/coordinator agent exists, use it to delegate analysis across relevant specialists before consolidating findings into the plan.

### Analysis Goals

- **Stability**: Identify fragile areas, missing tests, error handling gaps
- **Maintainability**: Find code smells, unclear architecture, missing documentation
- **Currency**: Outdated dependencies, deprecated patterns, tech debt
- **UI/UX Quality**: User experience issues, accessibility gaps, design inconsistencies (if applicable)

### Context Files (Read First)

- Project documentation (`docs/*.md`, `README.md`)
- Existing plan if any (`.ai-flow/plan.md`)
- Past learnings (`.ai-flow/learnings.md`)
- Architecture docs (`.ai-flow/project.md`)

---

## Plan Parameters

Fill in these values for your project:

**Project Name:** [Your project name]
**Project plan template:** [Path to template if exists, e.g., `.ai-flow/template/plan-template.md`]
**Project plan target document:** [Output path, e.g., `.ai-flow/plan.md`]
**Architecture Document:** [Path to architecture doc, e.g., `.ai-flow/project.md`]
**Tasks Directory:** [Path to task files, e.g., `.ai-flow/tasks/`]
**Learnings Document:** [Path to learnings, e.g., `.ai-flow/learnings.md`]

**Project Description:**
[Brief description of what the project does]

**Completed Work:**
[List completed phases/features, or say "None yet" for new projects]

**Remaining Features to Implement:**
[List the features/components that need to be built]

---

## Format Requirements

1. **Task ID Format:** Use `T{number}` format
   - Sequential numbering: T001, T002, T003...
   - Use 3+ digits for consistency (T001, not T1)
   - Numbers should be globally unique across the project
   - Example: T001, T002, T050, T100, T336

2. **Priority Indicators:** Use colored emoji balls
   - 🟢 P0 (MVP/Critical)
   - 🟠 P1 (Important)
   - 🔴 P2 (Nice to have)

3. **Status Markers:**
   - `[ ]` Todo
   - `[~]` In Progress
   - `[x]` Done

4. **Wave Organization:**
   - Wave 1: No dependencies on other remaining tasks (can start immediately)
   - Wave 2+: Tasks that depend on previous waves
   - Tasks within the same track should be sequential
   - Different tracks can be worked in parallel

5. **Track Organization:**
   - Group related tasks into tracks (A, B, C, D...)
   - Each track should have a descriptive name
   - Show track priority with emoji in track header

6. **Task Structure:**
   - Task ID in bold: `**T001**`
   - Clear task title after the ID
   - Link to task spec if exists: `(📋 [Spec](tasks/T001-short-description.md))`
   - Task spec filenames use format: `T{number}-short-description.md` (e.g., `T050-user-auth.md`)
   - Bullet points for implementation details
   - Dependencies listed with `**Deps: T001, T002**` format

7. **Include Sections:**
   - Legend explaining symbols
   - Completed Work Summary (table format with categories)
   - Remaining Work organized by Waves and Tracks
   - Future Feature Ideas (backlog)
   - Notes for development guidelines

---

Replace all `{{PLACEHOLDER}}` values with actual content. Do not leave any placeholders in the final document.


## Example Template:

```markdown
# {{Project Name}} - Development Tasks

Based on the architecture defined in `.ai-flow/project.md`, this document tracks remaining tasks and summarizes completed work.
Completed task details can be found in `.ai-flow/tasks/`.
Lessons learned from these tasks can be found in `.ai-flow/learnings.md`.

---

## Legend

- **Priority**: 🟢 P0 (MVP), 🟠 P1 (Important), 🔴 P2 (Nice to have)
- **Status**: `[ ]` Todo, `[~]` In Progress, `[x]` Done
- **Wave**: Execution order (Wave 1 first, then Wave 2, etc.)
- **Task ID Format**: `T{number}` (e.g., T001, T050)

---

## Completed Work Summary

| Category | Tasks | Description |
|----------|-------|-------------|
| **Phase 1: Foundation** | T001-T006 | Repository setup, build tooling |
| **Phase 2: Core Features** | T010-T025 | Main functionality |

---

## Remaining Work - Execution Plan

### Wave 1: Foundation

> Tasks with no dependencies that can start immediately.

#### Track A: Core Features (🟢 P0)

- [ ] **T050** - Implement user authentication (📋 [Spec](tasks/T050-user-auth.md))
  - Add login/logout endpoints
  - JWT token generation
  - Session management
  - **Deps**: None

- [ ] **T051** - Add user profile management (📋 [Spec](tasks/T051-user-profile.md))
  - CRUD operations for profiles
  - Avatar upload support
  - **Deps**: T050

#### Track B: API Layer (🟠 P1)

- [ ] **T060** - Create REST API structure (📋 [Spec](tasks/T060-rest-api.md))
  - Define route handlers
  - Add request validation
  - **Deps**: None

---

### Wave 2: Integration (Depends on Wave 1)

> Tasks that depend on Wave 1 completion.

#### Track A: Security (🟢 P0)

- [ ] **T100** - Add role-based permissions (📋 [Spec](tasks/T100-rbac.md))
  - Define permission levels
  - Middleware for access control
  - **Deps: T050, T060**

#### Track B: Testing (🟠 P1)

- [ ] **T110** - Integration test suite (📋 [Spec](tasks/T110-integration-tests.md))
  - API endpoint tests
  - Authentication flow tests
  - **Deps: T050, T060**

---

## Future Feature Ideas (Backlog)

### Advanced Features (🔴 P2)

- [ ] **T200** - Web dashboard
  - Admin interface
  - Analytics visualization

- [ ] **T201** - Plugin system
  - Extension API
  - Marketplace integration

---

## Notes

1. Breaking changes are acceptable during pre-release
2. Test coverage target: >80% for new features
3. Each task should have graceful error handling
```