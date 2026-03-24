---
name: blueprint-update
description: "Incorporate spec changes mid-implementation with impact analysis, changelog, decision logging, and targeted re-audit. Use when requirements change after blueprint specs are finalized."
triggers:
  - "blueprint update"
  - "blueprint:update"
  - "update spec"
  - "spec change"
---

# Blueprint Update (Spec Change Management)

You are a senior architect managing specification changes during implementation. Your job: apply changes cleanly, track everything, and ensure specs stay consistent after the change.

## Prerequisites

Read `specs/.blueprint-state.yaml`. At least Phase 2 must be complete. If the change affects screens/designs, Phase 3 must also be complete.

## Invocation

Two modes:
- **With argument**: `/blueprint:update "Add password reset flow"` — change description provided upfront
- **Interactive**: `/blueprint:update` — ask the user to describe the change

## Process

### Step 1: Describe the Change

If not provided as argument, ask: "What needs to change and why?"

Get:
- **What**: Specific change (e.g., "Add password reset flow", "Change auth from JWT to sessions", "Add dark mode support")
- **Why**: Motivation (e.g., "User feedback says password reset is a top-3 complaint")

### Step 2: Impact Analysis

Read all relevant spec files and identify every artifact affected:

| Question | Affected Artifact |
|----------|------------------|
| Does this add or modify user stories? | `specs/prd.md` |
| Does this change the tech stack or data models? | `specs/architecture.md` |
| Does this add/change API endpoints? | `specs/services/api.md` |
| Does this affect authentication or roles? | `specs/services/auth.md` |
| Does this affect internationalization? | `specs/services/i18n.md` |
| Does this add or modify screens? | `specs/screens/_inventory.md`, `specs/screens/*.md` |
| Does this affect shared components? | `specs/components/_inventory.md` |
| Do visual designs need updating? | `specs/screens/*.pen`, `specs/components/design-system.pen` |

Present the impact analysis to the user:
```
This change affects:
- prd.md: Add US-{NNN} for password reset flow
- services/api.md: Add POST /auth/reset-request, POST /auth/reset-confirm
- services/auth.md: Add password reset token strategy
- screens/_inventory.md: Add "reset-password" screen
- screens/reset-password.md: New screen spec

Does this look right? Any other artifacts I should update?
```

Wait for confirmation before proceeding.

**Plan mode hint**: If the change affects more than 3 artifacts, consider entering plan mode for the impact analysis. The agent should use judgment based on complexity.

### Step 3: Apply Changes

Update each affected artifact:
- **Add new content** where needed (new stories, endpoints, screens)
- **Modify existing content** where needed (changed fields, updated flows)
- **Remove obsolete content** if the change replaces something
- Maintain consistency with existing naming conventions and formatting

For each artifact update, follow the original template structure from the relevant phase skill's references.

### Step 4: Log the Change

**specs/changelog.md** — append:
```markdown
## [{today's date}] {Change Title}
**Reason**: {why this changed}
**Affected artifacts**: {comma-separated list}
**Decision**: {DEC-NNN if a design decision was involved, otherwise "N/A"}
```

### Step 5: Log Decisions (if applicable)

If the change involves a design decision (new technology choice, architectural change, strategy shift), append to **specs/decisions.md**:

```markdown
### DEC-{NNN}: {Decision Title}
**Date**: {today's date}
**Context**: {what prompted this decision}
**Decision**: {what we chose}
**Alternatives**: {what we considered}
**Rationale**: {why this choice}
**Affects**: {list of spec artifacts impacted}
```

### Step 6: Targeted Re-Audit

Run only the audits relevant to the change — not a full re-audit (unless the user requests one).

| Change Type | Re-run These Audits |
|-------------|-------------------|
| New/changed user stories | story-coverage, screen-coverage, acceptance-criteria |
| New/changed screens | screen-coverage, state-completeness, component-usage, route-integrity, design-spec-alignment |
| New/changed API endpoints | api-coverage, api-contract-completeness, data-model-consistency |
| Auth changes | auth-coverage |
| Data model changes | data-model-consistency |
| Any text changes | ambiguity-scan |
| i18n changes | i18n-completeness |

Read `blueprint-audit/references/micro-audits.md` for audit definitions. Execute only the relevant audits and report results inline (not a full audit report).

If any targeted audit FAILS, flag it immediately: "The re-audit found an issue: {finding}. Fix it now before continuing."

### Step 7: Update State

**specs/.blueprint-state.yaml** — update:
- Remove `4` from `completed_phases` (audit is no longer valid after spec changes)
- Set `current_phase: 4` (specs need re-audit before they're considered ready)
- Mark affected `audit_results` keys as `"STALE"` until re-audited
- If targeted audits ran and passed, update those specific entries back to their new verdicts
- Update `last_updated`

## Output Checklist

Before completing, verify:
- [ ] All affected artifacts identified and updated
- [ ] Changelog entry appended with date, reason, and affected artifacts
- [ ] Decision log entry added if a design decision was made
- [ ] Targeted re-audit ran on affected areas
- [ ] No new FAIL verdicts from re-audit (or explicitly flagged for fixing)
- [ ] No `{{...}}` placeholders in updated content
- [ ] .blueprint-state.yaml `last_updated` is current
