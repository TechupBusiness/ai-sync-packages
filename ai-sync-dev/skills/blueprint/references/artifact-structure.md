# Blueprint Artifact Structure

## Output Directory

All artifacts live under `specs/` in the project root.

```
specs/
├── .blueprint-state.yaml            # Phase tracking + platform + audit results
├── vision.md                        # Phase 1: Product brief
├── prd.md                           # Phase 2: User stories + acceptance criteria
├── architecture.md                  # Phase 2: Tech stack + data models + system overview
├── decisions.md                     # ADR-style decision log
├── changelog.md                     # Change history (what/when/why)
├── screens/                         # Phase 3: Screen specs + designs
│   ├── _inventory.md                # Screen list, routes, navigation map
│   ├── {screen-name}.md             # Per-screen spec (states, components, data)
│   └── {screen-name}.pen            # Per-screen Pencil design
├── components/                      # Phase 3: Shared components
│   ├── _inventory.md                # Component catalog + design tokens
│   └── design-system.pen            # Design system (Pencil)
├── services/                        # Phase 2: Service contracts
│   ├── api.md                       # Endpoints, request/response schemas, errors
│   ├── auth.md                      # Auth flow, tokens, roles (if applicable)
│   ├── i18n.md                      # Locales, key format, pluralization (if applicable)
│   └── native.md                    # Native capabilities, deep links (if mobile/hybrid)
└── audit-report.md                  # Phase 4: Decomposed audit results
```

## State File Format

`specs/.blueprint-state.yaml` tracks progress across sessions:

```yaml
product_name: "MyApp"
platform: web                        # web | mobile-flutter | mobile-rn | mobile-native | hybrid
platform_targets: [ios, android]     # (mobile-native only) which targets
current_phase: 2
completed_phases: [1]
audit_runs: 0
audit_results: {}
pencil_available: true               # false if Pencil MCP was unavailable
auth_required: true                  # false if no auth stories exist
i18n_required: false                 # true if multi-locale product
last_updated: "2026-02-27"
```

| Field | Type | Description |
|-------|------|-------------|
| `product_name` | string | Product name from Phase 1 |
| `platform` | enum | Target platform, drives template additions |
| `platform_targets` | string[] | (optional) For `mobile-native`: `[ios]`, `[android]`, or `[ios, android]` |
| `current_phase` | int | 1-4, phase currently in progress |
| `completed_phases` | int[] | Phases fully complete |
| `audit_runs` | int | Number of times audit has been run |
| `audit_results` | map | Last audit verdicts: `{audit_name: "PASS\|WARN\|FAIL"}` |
| `pencil_available` | bool | (optional) `false` if Pencil MCP was unavailable during Phase 3 |
| `auth_required` | bool | (optional) `false` if product has no auth; skips auth.md generation |
| `i18n_required` | bool | (optional) `true` if multi-locale; triggers i18n.md generation |
| `last_updated` | date | ISO date of last state change |

## Naming Conventions

- Screen specs: `specs/screens/{kebab-case-name}.md`
- Screen designs: `specs/screens/{kebab-case-name}.pen`
- Service specs: `specs/services/{service-name}.md`
- Decision IDs: `DEC-{NNN}` (sequential, zero-padded to 3 digits)
