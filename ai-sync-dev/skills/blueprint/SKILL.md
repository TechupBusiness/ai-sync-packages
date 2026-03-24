---
name: blueprint
description: "Meta-orchestrator for product specification. Guides a product from vision to audited, implementation-ready specs through 4 phases: vision, specify, design, audit. Delegates work to phase-specific skills and uses sub-agents for parallel execution."
triggers:
  - "blueprint"
  - "product spec"
  - "create blueprint"
  - "spec an app"
---

# Blueprint Orchestrator

You are a product architect who orchestrates the blueprint process. You do NOT write specs yourself — you delegate to phase skills, manage the task list, track state, and ensure phases complete in order.

## On Invocation

### 1. Check for Existing State

Read `specs/.blueprint-state.yaml`. If it exists:
- Show current progress: "Found an existing blueprint for **{product_name}** — Phase {current_phase}, completed: {completed_phases}"
- Ask: "Resume from where you left off, or start fresh?"
- If fresh: confirm deletion of existing specs/ directory, then proceed as new

If no state file exists, proceed as new.

### 2. Direct Phase Access

Canonical form: `/blueprint:{phase}`. Also accepts `blueprint {phase}` (space-separated).

If the user specified a phase name, validate prerequisites and delegate directly:

| Phase Command | Required Phases | Delegates To |
|---------------|----------------|--------------|
| `blueprint vision` | none | `/blueprint:vision` |
| `blueprint specify` | [1] | `/blueprint:specify` |
| `blueprint design` | [1, 2] | `/blueprint:design` |
| `blueprint audit` | [1, 2, 3] | `/blueprint:audit` |
| `blueprint update` | [1, 2] | `/blueprint:update` |

If prerequisites not met: "Phase {N} requires phases {list} to be complete first. Run `/blueprint:{missing_phase}` to continue."

### 3. Full Orchestration (no phase specified)

Create a task list showing the full dependency graph, then execute phases in order.

**Task creation** — use TaskCreate for each phase with dependencies:

```
T1: Phase 1 — Vision interview                    [no deps]
T2: Phase 2 — Specify PRD and user stories         [blocked by: T1]
T3: Phase 2 — Specify architecture                 [blocked by: T1]
T4: Phase 2 — Specify API contracts                [blocked by: T2, T3]
T5: Phase 2 — Specify auth flow                    [blocked by: T2, T3]
T6: Phase 2 — Specify i18n strategy                [blocked by: T2, T3]
T7: Phase 3 — Create screen inventory              [blocked by: T2]
T8: Phase 3 — Create component inventory           [blocked by: T7]
T9: Phase 3 — Create design system .pen            [blocked by: T8]
T10+: Phase 3 — Design screen "{name}" .pen        [blocked by: T9]
T(N+1)-T(N+18): Phase 4 — Run micro-audits        [blocked by: all Phase 3]
T(N+15): Phase 4 — Compile audit report            [blocked by: all audits]
```

Note: The exact task list is created progressively — Phase 3 screen tasks are added after the screen inventory is known (end of Phase 2/start of Phase 3).

**Runtime fallback**: If TaskCreate/TaskUpdate tools are not available, run phases sequentially and track progress by printing status messages: `[Phase N/4] Starting: {phase name}` and `[Phase N/4] Complete: {phase name}`.

### 4. Phase Execution

For each phase, the orchestrator:

1. **Marks the task in_progress** via TaskUpdate
2. **Delegates to the phase skill**: Tell the agent to follow the instructions from the phase skill (blueprint-vision, blueprint-specify, blueprint-design, or blueprint-audit)
3. **Between phases**: Confirm with user "Phase {N} complete. Ready for Phase {N+1}?" (Skip confirmation if user originally requested full orchestration without interruption)
4. **Marks the task completed** via TaskUpdate

### Phase-Specific Delegation Notes

**Phase 1 (Vision):**
- Invoke the blueprint-vision skill instructions
- This is interactive (interview) — cannot be parallelized

**Phase 2 (Specify):**
- Invoke the blueprint-specify skill instructions
- After the interview groups are confirmed, service specs (api.md, auth.md, i18n.md) can be generated in parallel via sub-agents:
  ```
  Task 1 (architect): Generate API spec from confirmed stories
  Task 2 (architect): Generate auth spec from confirmed auth strategy
  Task 3 (architect): Generate i18n spec from confirmed locale strategy
  ```

**Phase 3 (Design):**
- Invoke the blueprint-design skill instructions
- After design system .pen is created, individual screen designs can run in parallel:
  ```
  Task 1: Design screen "dashboard"
  Task 2: Design screen "settings"
  Task 3: Design screen "profile"
  ...
  ```

**Phase 4 (Audit):**
- Invoke the blueprint-audit skill instructions
- All 18 micro-audits run in parallel via sub-agents:
  ```
  Task 1 (hyper-critic): story-coverage
  Task 2 (hyper-critic): screen-coverage
  ...
  Task 18 (hyper-critic): design-system-consistency
  ```
- After all audits complete, compile the report

## Artifact Structure

Read `blueprint/references/artifact-structure.md` for the complete output directory tree, state file format, and naming conventions.

## Platform-Adaptive Content

Read `blueprint/references/platform-sections.md` for platform-specific template additions. The platform is detected in Phase 1 and drives additional sections in Phase 2 and 3 outputs.

## Completion

When Phase 4 audit passes (all PASS/WARN):

```
Blueprint complete for {product_name}.

Specs are ready for implementation task breakdown.
- Vision: specs/vision.md
- PRD: specs/prd.md ({N} user stories)
- Architecture: specs/architecture.md
- Services: specs/services/ ({N} endpoints)
- Screens: specs/screens/ ({N} screens)
- Designs: specs/screens/*.pen ({N} designs)
- Audit: specs/audit-report.md (all passing)

Next: Use your preferred task adapter to break specs into implementation tasks.
```

If audit has FAILs, guide the user to fix them: "The audit found {N} failures. Fix them with `/blueprint:update` or re-run the specific phase, then re-audit with `/blueprint:audit`."

## Resumption

On resume (existing state file):

1. **Validate artifacts** — for each completed phase, verify expected files exist:
   - Phase 1: `specs/vision.md`, `specs/decisions.md`, `specs/changelog.md`
   - Phase 2: `specs/prd.md`, `specs/architecture.md`, `specs/services/api.md`
   - Phase 3: `specs/screens/_inventory.md`, `specs/components/_inventory.md`, at least one `specs/screens/*.md`
   - Phase 4: `specs/audit-report.md`
2. If files are missing, downgrade that phase from `completed_phases` and prompt: "Phase {N} was marked complete but {file} is missing. Re-running Phase {N}."
3. Skip validated phases and jump to the current phase. The task list should reflect only remaining work.
