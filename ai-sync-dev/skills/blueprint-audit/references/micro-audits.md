# Micro-Audit Definitions

Each audit is a focused, narrow-scope check that reads 1-3 files and produces a verdict: `PASS`, `WARN(reason)`, or `FAIL(reason)`. All 18 audits are independent and can run in parallel.

---

## Coverage Audits

### 1. story-coverage

**Check**: Every user story in prd.md maps to at least one screen in screens/_inventory.md.
**Reads**: `specs/prd.md`, `specs/screens/_inventory.md`
**PASS**: All P0/P1 stories have at least one screen in the "Parent Stories" column.
**WARN**: A P2 story has no screen (acceptable for backlog items).
**FAIL**: A P0 or P1 story has no corresponding screen.

### 2. screen-coverage

**Check**: Every screen in the inventory traces back to at least one user story.
**Reads**: `specs/screens/_inventory.md`, `specs/prd.md`
**PASS**: Every screen references valid story IDs that exist in prd.md.
**WARN**: A screen references a story ID not found in prd.md (orphan screen).
**FAIL**: Multiple screens have no story reference (untraceable screens).

### 3. api-coverage

**Check**: Every screen that displays data has a corresponding API endpoint.
**Reads**: `specs/screens/_inventory.md`, all `specs/screens/*.md`, `specs/services/api.md`
**PASS**: All data-showing screens reference endpoints that exist in api.md.
**WARN**: A screen's data dependency mentions an endpoint not in api.md.
**FAIL**: Multiple screens lack API endpoint coverage.

### 4. auth-coverage

**Check**: Auth-required screens in inventory match protected routes in auth.md.
**Reads**: `specs/screens/_inventory.md`, `specs/services/auth.md`
**PASS**: All screens marked "Auth Required: yes" have matching protected route patterns.
**WARN**: A protected route exists in auth.md with no matching screen.
**FAIL**: Auth-required screen has no protected route, or public screen is marked protected.
**SKIP**: If `specs/services/auth.md` does not exist (no auth in this product).

### 5. platform-coverage

**Check**: Architecture.md includes platform-specific sections for the detected platform.
**Reads**: `specs/architecture.md`, `specs/.blueprint-state.yaml`
**PASS**: Platform-specific sections present and filled (not placeholder).
**WARN**: Platform sections present but sparse (fewer than 3 fields defined).
**FAIL**: No platform-specific sections exist for the detected platform.

---

## Completeness Audits

### 6. state-completeness

**Check**: Every screen spec defines all 4 mandatory states.
**Reads**: `specs/screens/*.md` (all screen specs)
**PASS**: Every screen spec has loading, error, empty, and success sections with specific content.
**WARN**: A state section exists but is generic (e.g., "shows an error message" without specifics).
**FAIL**: A screen spec is missing one or more state sections.

### 7. acceptance-criteria

**Check**: Every P0 user story has Given/When/Then acceptance criteria with concrete predicates.
**Reads**: `specs/prd.md`
**PASS**: All P0 stories have Given/When/Then blocks with specific values (not "appropriate" or "valid").
**WARN**: A P0 story has Given/When/Then but with vague predicates.
**FAIL**: A P0 story is missing Given/When/Then entirely.

### 8. api-contract-completeness

**Check**: Every API endpoint has request schema, response schema, error table, and auth requirement.
**Reads**: `specs/services/api.md`
**PASS**: Every endpoint has all four sections with concrete content.
**WARN**: An endpoint has a section but it's incomplete (e.g., error table with only one entry).
**FAIL**: An endpoint is missing request schema, response schema, or error table.

---

## Consistency Audits

### 9. data-model-consistency

**Check**: API endpoints reference data models defined in architecture.md.
**Reads**: `specs/services/api.md`, `specs/architecture.md`
**PASS**: All model/type names in API responses correspond to models in architecture.md.
**WARN**: A field name difference between API response and model definition (possible mapping issue).
**FAIL**: An API endpoint references a model not defined in architecture.md.

### 10. component-usage

**Check**: Screen specs reference components that exist in the component inventory.
**Reads**: `specs/components/_inventory.md`, all `specs/screens/*.md`
**PASS**: All referenced components exist in the inventory.
**WARN**: A screen references a component with a slightly different name (typo candidate).
**FAIL**: A screen references a component not in the inventory.

### 11. route-integrity

**Check**: Navigation targets in screen specs reference existing screens.
**Reads**: `specs/screens/_inventory.md`, all `specs/screens/*.md`
**PASS**: All navigation targets (routes) in User Interactions tables point to screens in the inventory.
**WARN**: A navigation target uses a different route format than the inventory.
**FAIL**: A navigation target points to a screen/route not in the inventory.

### 12. design-spec-alignment

**Check**: Every screen .md spec has a corresponding .pen design file.
**Reads**: `specs/screens/` directory listing, `specs/.blueprint-state.yaml`
**PASS**: Every `{name}.md` has a matching `{name}.pen`.
**WARN**: .pen file missing but `pencil_available: false` is set in state (expected).
**FAIL**: .pen file missing and Pencil was available.

---

## Quality Audits

### 13. ambiguity-scan

**Check**: Spec files don't contain ambiguous language that would cause AI implementation errors.
**Reads**: All `specs/*.md` and `specs/**/*.md` files
**Scans for these ambiguous terms**: "appropriate", "etc.", "as needed", "similar", "relevant", "properly", "correctly", "should be", "as necessary", "and so on", "like this"
**PASS**: No ambiguous terms found.
**WARN**: 1-3 instances found (list each with file and line context).
**FAIL**: 4+ instances found (ambiguity is pervasive).

### 14. i18n-completeness

**Check**: If i18n is specified, screen specs should reference translation keys (not hardcoded strings).
**Reads**: `specs/services/i18n.md`, all `specs/screens/*.md`
**PASS**: Screen specs mention translation keys or i18n-aware patterns.
**WARN**: i18n spec exists but screens don't explicitly mention translation handling.
**FAIL**: i18n spec defines locales but screens contain hardcoded user-facing strings.
**SKIP**: If `specs/services/i18n.md` does not exist (single-locale product).

---

## Traceability Audits

### 15. example-completeness

**Check**: Every P0 user story has at least one concrete input/output example.
**Reads**: `specs/prd.md`
**PASS**: All P0 stories have an "Example" section with specific input and output values.
**WARN**: A P0 story has an example but with vague values (e.g., "some data").
**FAIL**: A P0 story is missing the concrete example entirely.

### 16. decision-log-continuity

**Check**: decisions.md has entries for key choices (platform, tech stack, auth strategy).
**Reads**: `specs/decisions.md`, `specs/architecture.md`
**PASS**: Every tech stack choice in architecture.md has a corresponding DEC-NNN entry.
**WARN**: Minor choices (e.g., lint config) lack decision entries (acceptable).
**FAIL**: Major architectural choices (framework, database, auth strategy) have no decision entry.

### 17. changelog-continuity

**Check**: changelog.md has entries for each completed phase.
**Reads**: `specs/changelog.md`, `specs/.blueprint-state.yaml`
**PASS**: One changelog entry per completed phase, each listing affected artifacts.
**WARN**: A changelog entry exists but is missing affected artifacts list.
**FAIL**: A completed phase has no changelog entry.

### 18. design-system-consistency

**Check**: Design system .pen exists and contains reusable components matching the component inventory.
**Reads**: `specs/components/_inventory.md`, `specs/components/design-system.pen` (directory listing)
**PASS**: design-system.pen exists and contains reusable components.
**WARN**: design-system.pen exists but has fewer components than the inventory lists.
**FAIL**: design-system.pen missing and Pencil was available.
**SKIP**: If `pencil_available: false` in state.
