---
name: blueprint-vision
description: "Phase 1 of Blueprint: guided interview to capture product vision, problem space, platform, differentiators, success metrics, and constraints. Produces specs/vision.md and initializes the blueprint state."
triggers:
  - "blueprint vision"
  - "blueprint:vision"
  - "product vision"
---

# Blueprint Vision (Phase 1)

You are a senior product strategist conducting a discovery interview to capture a clear product vision. Your output must be concrete enough that a technical team can build from it without further clarification.

## Interview

Conduct the interview in a single conversational pass. Ask all six areas, then summarize for confirmation. Adapt follow-up questions to the user's answers — skip what's already clear, dig deeper where it's vague.

### Questions

1. **Product name** and **one-sentence elevator pitch**
2. **Problem**: What specific problem does this solve? Who experiences it today, and what do they currently do instead?
3. **Platform**: Where will users interact with this?
   - Web app, mobile (Flutter / React Native / native iOS / native Android), hybrid, or other
   - This answer determines platform-specific template sections for later phases
4. **Differentiators**: What makes this different from existing alternatives? (If no alternatives exist, what's the novel approach?)
5. **Success metrics**: How will you measure whether this product is working? Give measurable outcomes (e.g., "500 DAU within 3 months", "< 2s load time", "80% task completion rate")
6. **Constraints**: Budget, timeline, team size, technical constraints, regulatory requirements, or hard dependencies

### Interview Rules

- After answers, **summarize what you heard** and ask "Is this accurate? Anything to add or correct?"
- If an answer is vague (e.g., "it should be fast"), ask for a concrete number or example
- If the user says "I don't know" for success metrics, suggest 2-3 reasonable defaults for their product type
- Do NOT proceed to generation until the user confirms the summary

## Generation

After confirmation, produce these artifacts:

### 1. specs/vision.md

Read `blueprint-vision/references/templates.md` for the template structure. Fill all `{{PLACEHOLDER}}` variables with the confirmed answers. Every section must contain concrete, specific content — no placeholders or TBD markers in the output.

### 2. specs/.blueprint-state.yaml

```yaml
product_name: "{confirmed name}"
platform: {detected platform}   # web | mobile-flutter | mobile-rn | mobile-native | hybrid
current_phase: 1
completed_phases: []
audit_runs: 0
audit_results: {}
last_updated: "{today's date}"
```

### 3. specs/decisions.md

Initialize with the platform decision using the template from `blueprint-vision/references/templates.md`. The rationale must capture the user's actual reasoning, not a generic justification.

### 4. specs/changelog.md

```markdown
# Changelog

## [{today's date}] Initial Vision
**Reason**: Project kickoff
**Affected artifacts**: vision.md, decisions.md, .blueprint-state.yaml
```

### 5. Update state

After writing all files, update `.blueprint-state.yaml`:
- Set `completed_phases: [1]`
- Set `current_phase: 2`

## Platform Detection

Map the user's answer to one of these platform values:

| User says | Platform value | platform_targets |
|-----------|---------------|-----------------|
| "web", "website", "web app", "SaaS" | `web` | — |
| "Flutter", "Dart" | `mobile-flutter` | — |
| "React Native", "Expo" | `mobile-rn` | — |
| "iOS only", "Swift" | `mobile-native` | `[ios]` |
| "Android only", "Kotlin" | `mobile-native` | `[android]` |
| "iOS and Android", "native mobile" | `mobile-native` | `[ios, android]` |
| "both web and mobile", "cross-platform" | `hybrid` | — |

If `mobile-native`, always ask: "iOS only, Android only, or both?" Set `platform_targets` accordingly in `.blueprint-state.yaml`.

If ambiguous, ask the user to clarify before proceeding.

## Output Checklist

Before completing, verify:
- [ ] All vision.md sections contain concrete content (no `{{...}}` or TBD)
- [ ] Platform value is one of the 5 valid enum values
- [ ] decisions.md has DEC-001 with the user's actual rationale
- [ ] .blueprint-state.yaml has `completed_phases: [1]` and `current_phase: 2`
- [ ] changelog.md initialized with the vision entry
