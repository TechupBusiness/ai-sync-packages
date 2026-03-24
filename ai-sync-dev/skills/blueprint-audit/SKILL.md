---
name: blueprint-audit
description: "Phase 4 of Blueprint: run 18 decomposed micro-audits to verify spec coverage, completeness, consistency, quality, and traceability. Produces specs/audit-report.md with pass/warn/fail verdicts."
triggers:
  - "blueprint audit"
  - "blueprint:audit"
  - "audit specs"
---

# Blueprint Audit (Phase 4)

You are a meticulous spec reviewer. Your job is to find every gap, inconsistency, and ambiguity in the blueprint specs before they reach implementation. Be thorough and precise — cite specific files, story IDs, and endpoints when reporting issues.

## Prerequisites

Read `specs/.blueprint-state.yaml`. Phases 1-3 must be complete. If not, instruct the user to complete missing phases first.

This skill can also be invoked independently at any time to re-audit existing specs (e.g., after `/blueprint:update`).

## Audit Execution

### 1. Load Audit Definitions

Read `blueprint-audit/references/micro-audits.md` for all 18 audit definitions.

### 2. Execute Audits

Execute each micro-audit as a focused check. Each audit:
- Reads only the files specified in its definition (1-2 files)
- Evaluates the specific check predicate
- Produces a verdict: `PASS`, `WARN(reason)`, or `FAIL(reason)`
- Produces findings: specific items that triggered WARN or FAIL

**Parallelism**: All 18 audits are independent. Maximize parallel execution by launching each audit as an independent sub-agent task via the Task tool. Send all 18 task calls in a single message.

Each sub-agent task prompt should be:
```
Read the following spec files: {file1}, {file2}
Execute this audit check: {check description from micro-audits.md}
Return a verdict (PASS, WARN, or FAIL) with specific findings.
Cite file paths, story IDs, screen names, or endpoint paths for each finding.
```

### 3. Codex Cross-Validation (Optional)

Check if `codex` CLI is available:
```bash
command -v codex >/dev/null 2>&1
```

If available, run critical audits (FAIL or WARN results) through Codex for a second perspective:
```bash
out=$(mktemp /tmp/blueprint-audit-XXXXXX.md)
{ echo "Audit these specs for: {specific check predicate}"; echo; cat specs/{file1}; echo "---"; cat specs/{file2}; } | codex exec -s read-only -c 'model_reasoning_effort="high"' --ephemeral -o "$out" -
```

Merge findings: deduplicate, note agreements, flag disagreements.

### 4. Compile Report

Generate `specs/audit-report.md`:

```markdown
# Blueprint Audit Report

**Run**: #{audit_runs + 1} | **Date**: {today} | **Product**: {product_name}
**Engines**: {Claude + Codex | Claude only}

## Summary

| Status | Count |
|--------|-------|
| PASS   | {N}   |
| WARN   | {N}   |
| FAIL   | {N}   |
| SKIP   | {N}   |

## Results

### Coverage Checks

#### story-coverage: {VERDICT}
{findings or "All P0/P1 stories have screen coverage."}

#### screen-coverage: {VERDICT}
{findings}

#### api-coverage: {VERDICT}
{findings}

#### auth-coverage: {VERDICT}
{findings}

#### platform-coverage: {VERDICT}
{findings}

### Completeness Checks

#### state-completeness: {VERDICT}
{findings}

#### acceptance-criteria: {VERDICT}
{findings}

#### api-contract-completeness: {VERDICT}
{findings}

### Consistency Checks

#### data-model-consistency: {VERDICT}
{findings}

#### component-usage: {VERDICT}
{findings}

#### route-integrity: {VERDICT}
{findings}

#### design-spec-alignment: {VERDICT}
{findings}

### Quality Checks

#### ambiguity-scan: {VERDICT}
{findings}

#### i18n-completeness: {VERDICT}
{findings}

### Traceability Checks

#### example-completeness: {VERDICT}
{findings}

#### decision-log-continuity: {VERDICT}
{findings}

#### changelog-continuity: {VERDICT}
{findings}

#### design-system-consistency: {VERDICT}
{findings}

## Recommendations

{For each FAIL, map to the originating phase and provide specific fix guidance:}
1. **FAIL: {audit_name}** — Revisit Phase {N}: {specific action to fix}
```

### 5. Exit Gate Logic

After compiling the report:

- **All PASS + WARN**: Specs are ready for task breakdown. Print: "Blueprint audit passed. Specs are ready for implementation task breakdown."
- **Any FAIL**: Print specific remediation steps. Each FAIL maps to a phase:

| Audit Category | Fix Phase |
|----------------|-----------|
| Coverage audits (1-5) | Phase 2 (specify) or Phase 3 (design) |
| Completeness audits (6-8) | Phase 2 (specify) or Phase 3 (design) |
| Consistency audits (9-12) | Phase 2 or Phase 3 (depends on which artifact) |
| Quality audits (13-14) | Any phase (run ambiguity scan on flagged files) |
| Traceability audits (15-18) | Phase 1-3 (depends on which artifact) |

### 6. Update Tracking

**specs/.blueprint-state.yaml** — update:
- Increment `audit_runs`
- Set `audit_results` to the verdict map: `{story-coverage: "PASS", ...}`
- If all PASS/WARN: add `4` to `completed_phases`
- Update `last_updated`

**specs/changelog.md** — append:
```markdown
## [{today's date}] Audit Run #{N}
**Reason**: Phase 4 audit execution
**Results**: {N} PASS, {N} WARN, {N} FAIL
**Affected artifacts**: audit-report.md
```

## Output Checklist

Before completing, verify:
- [ ] All 18 audits executed (or marked SKIP with reason)
- [ ] Every FAIL has specific findings citing files, IDs, or names
- [ ] Every FAIL has a remediation recommendation mapping to a specific phase
- [ ] Audit report written to specs/audit-report.md
- [ ] .blueprint-state.yaml updated with audit_runs and audit_results
- [ ] Changelog updated with audit run entry
