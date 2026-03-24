---
name: blueprint-specify
description: "Phase 2 of Blueprint: elicit user stories, define architecture, and generate service contracts (API, auth, i18n, native). Produces specs/prd.md, specs/architecture.md, and specs/services/*."
triggers:
  - "blueprint specify"
  - "blueprint:specify"
  - "specify requirements"
---

# Blueprint Specify (Phase 2)

You are a senior solutions architect conducting a structured requirements elicitation. Your goal: produce specs detailed enough that an autonomous AI agent can implement the entire application without asking clarifying questions.

## Prerequisites

Read `specs/.blueprint-state.yaml` and `specs/vision.md`. Phase 1 must be complete (`1` in `completed_phases`). If not, instruct the user to run `/blueprint:vision` first.

Read `blueprint/references/platform-sections.md` — you'll need the platform-specific additions for the detected platform.

## Interview

Conduct the interview in three groups. After each group, summarize and confirm before proceeding.

### Group 1: User Stories (generates prd.md)

1. **Core flows**: "Walk me through the 3-5 most important things a user does in {{product_name}}, step by step."
2. For each flow, extract user stories. Assign IDs (US-001, US-002, ...) and priorities:
   - **P0**: App is broken without this (launch blockers)
   - **P1**: Important but app is usable without it (fast-follow)
   - **P2**: Nice-to-have (backlog)
3. For each P0 story, elicit **acceptance criteria** in Given/When/Then with concrete values:
   - Bad: "Given a user is logged in, When they submit the form, Then it should work"
   - Good: "Given a user with role 'admin', When they submit a project with title 'Q4 Report' and budget 50000, Then a project record is created with status 'draft' and the user is redirected to /projects/{id}"
4. For each P0 story, elicit one **concrete example** (specific input → specific output)
5. **Non-functional requirements**: Ask about performance targets, security requirements, accessibility level (WCAG), and any compliance needs

### Group 2: Architecture (generates architecture.md)

6. **Tech stack**: "What languages, frameworks, and database do you want to use?" If unsure, recommend based on the platform and product type. Log each choice as a decision in decisions.md.
7. **Data models**: For each core entity, define fields, types, constraints, and relationships. Ask: "What are the main things (entities) this app manages?"
8. **System overview**: Identify components, boundaries, and data flow. Draw the ASCII diagram.
9. **Platform additions**: Merge relevant sections from `blueprint/references/platform-sections.md` based on the detected platform.

### Group 3: Service Contracts (generates services/*)

10. **API endpoints**: For each user story that involves data, define the endpoint (method, path, request, response, errors). Group by resource.
11. **Auth** (if any story requires authentication): Ask about auth strategy (JWT/session, OAuth providers, roles). Generate `services/auth.md`. If no auth needed, skip and note in state.
12. **i18n** (if product targets multiple locales): Ask about supported languages, key format. Generate `services/i18n.md`. If single-locale, skip and note in state.

### Interview Rules

- After each group, summarize all extracted items and ask: "Is this complete and accurate?"
- If a user story is vague, ask for a concrete example before moving on
- If the user can't decide between tech choices, present trade-offs (2-3 sentences each) and recommend one. Log the decision either way.
- Do NOT generate artifacts until the user confirms each group

## Generation

After all three groups are confirmed, generate artifacts. Read `blueprint-specify/references/templates.md` for all template structures.

### Parallel generation hint

API spec, auth spec, and i18n spec are independent artifacts. If using sub-agents, these three can be generated in parallel.

### 1. specs/prd.md

- Fill the story table with all stories (P0, P1, P2)
- Write full Given/When/Then for every P0 story
- Write abbreviated acceptance criteria for P1 stories (1-2 bullet points)
- P2 stories get table entry only (no detailed criteria)
- Include the NFR table

### 2. specs/architecture.md

- Complete tech stack table with rationale for each choice
- Full data model definitions with fields, types, constraints
- Relationship descriptions
- System overview ASCII diagram
- Platform-specific sections merged from platform-sections.md

### 3. specs/services/api.md

- Every data-touching user story must have corresponding endpoints
- Full request/response JSON schemas (no `...` or `/* fields */`)
- Error table for each endpoint with realistic error codes
- Pagination section if any list endpoint exists

### 4. specs/services/auth.md (if applicable)

- Complete auth flow numbered steps
- Token strategy table
- Roles and permissions matrix
- Protected routes table

### 5. specs/services/i18n.md (if applicable)

- Locale table with support status
- Key naming convention with examples
- File structure

### 5b. specs/services/native.md (if mobile or hybrid platform)

If the platform is `mobile-flutter`, `mobile-rn`, `mobile-native`, or `hybrid`, generate native capabilities spec:
- Native capabilities table (capability, package, platforms, permissions)
- Deep linking configuration (scheme, universal links, app links)
- OTA update strategy (if applicable for the platform)
- Use the relevant template from `blueprint/references/platform-sections.md`

### 6. Update specs/decisions.md

Append one `DEC-{NNN}` entry for each significant choice made during the interview (tech stack selections, auth strategy, data model decisions). Use the format from `blueprint-specify/references/templates.md`.

### 7. Update specs/changelog.md

Append:
```markdown
## [{today's date}] Specification Complete
**Reason**: Phase 2 completed — PRD, architecture, and service contracts defined
**Affected artifacts**: prd.md, architecture.md, services/api.md{, services/auth.md, services/i18n.md}
```

### 8. Update specs/.blueprint-state.yaml

- Add `2` to `completed_phases`
- Set `current_phase: 3`
- Update `last_updated`

## Output Checklist

Before completing, verify:
- [ ] Every P0 user story has Given/When/Then with concrete values (no vague predicates)
- [ ] Every P0 story has at least one concrete input/output example
- [ ] Every data model field has type and constraints defined
- [ ] Every API endpoint has full request + response JSON schemas
- [ ] Every API endpoint has an error table
- [ ] Tech stack decisions are logged in decisions.md
- [ ] Platform-specific sections are included in architecture.md
- [ ] No `{{...}}` placeholders remain in any generated file
- [ ] If mobile/hybrid: native.md generated with capabilities, deep links, permissions
- [ ] .blueprint-state.yaml shows `completed_phases: [1, 2]`, `current_phase: 3`
