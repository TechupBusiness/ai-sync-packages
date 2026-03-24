---
name: blueprint-design
description: "Phase 3 of Blueprint: generate screen inventory, per-screen specs, component inventory, and Pencil (.pen) design files. Produces specs/screens/*, specs/components/*, and visual designs."
triggers:
  - "blueprint design"
  - "blueprint:design"
  - "design screens"
---

# Blueprint Design (Phase 3)

You are a senior product designer creating comprehensive screen specifications and visual designs. Every screen must be fully specified — an AI agent implementing this app should never need to guess layout, states, or component behavior.

## Prerequisites

Read `specs/.blueprint-state.yaml`. Phases 1-2 must be complete (`[1, 2]` in `completed_phases`). If not, instruct the user to complete missing phases first.

Read these files to inform design decisions:
- `specs/vision.md` — product context, platform
- `specs/prd.md` — user stories (screens must cover all P0/P1 stories)
- `specs/architecture.md` — data models (screen data deps reference these)
- `specs/services/api.md` — endpoints (screen data sources)
- `specs/services/auth.md` — protected routes (if exists)

Read `blueprint-design/references/templates.md` for template structures.

## Process

### Step 1: Screen Inventory

1. Map every P0 and P1 user story to one or more screens
2. Identify additional screens needed: navigation shells, settings, error pages, onboarding
3. Generate `specs/screens/_inventory.md` using the template
4. Present the inventory to the user: "Here are the {{N}} screens I've identified. Should I add, remove, or modify any?"
5. Wait for confirmation before proceeding

### Step 2: Component Inventory

1. Scan the screen inventory for recurring UI patterns (buttons, cards, inputs, modals, nav bars, list items)
2. Define shared components with their props and which screens use them
3. Select design tokens (colors, spacing, typography, border-radius) that match the product's domain
4. Generate `specs/components/_inventory.md` using the template
5. Briefly confirm with user: "I've identified {{N}} shared components and the design tokens. Any adjustments?"

### Step 3: Per-Screen Specs

For each screen in the inventory, generate `specs/screens/{screen-name}.md`:

1. **Purpose**: One sentence describing what the user accomplishes
2. **States**: All 4 mandatory states (loading, error, empty, success) with specific descriptions — not generic. Example:
   - Loading: "Three skeleton cards with pulsing animation, header text visible"
   - Error: "Centered error illustration, message 'Could not load projects', retry button"
   - Empty: "Centered illustration of empty folder, text 'No projects yet', CTA 'Create your first project'"
   - Success: "Grid of project cards (max 3 columns), each showing title, status badge, last updated date"
3. **Components**: Table listing every component used on this screen
4. **Data dependencies**: Which API endpoints supply data, with response type references
5. **User interactions**: Every clickable/tappable element, what it does, where it navigates

### Step 3b: Reconcile Component Inventory

After all screen specs are written, scan them and update `specs/components/_inventory.md`:
- Add any new components discovered during screen spec writing
- Update "Used In" column to reflect actual usage across all screens
- Remove components that no screen references
- This ensures the component inventory is accurate before .pen generation

### Step 4: Visual Designs (.pen files)

Read `blueprint-design/references/pencil-integration.md` for the complete Pencil MCP workflow.

**Execution order:**
1. Get style guide (once, shared across all .pen files)
2. Create design system .pen (shared components as reusable frames)
3. Create per-screen .pen files (reference design system components)
4. Verify each design with screenshots

**Parallelism hint**: After the design system .pen is created, individual screen .pen files are independent. Use sub-agents to design multiple screens in parallel.

**Graceful degradation**: If Pencil MCP tools are not available:
- Skip all .pen file generation
- Add `pencil_available: false` to `.blueprint-state.yaml`
- The audit phase will flag missing .pen files as WARN, not FAIL
- Screen .md specs are still fully valuable without visual designs

### Step 5: Update Decision Log

If any design decisions were made (e.g., color palette choice, navigation pattern, responsive breakpoints), append entries to `specs/decisions.md`.

### Step 6: Update Tracking Files

**specs/changelog.md** — append:
```markdown
## [{today's date}] Design Phase Complete
**Reason**: Phase 3 completed — screen specs, component inventory, and visual designs
**Affected artifacts**: screens/_inventory.md, screens/*.md, screens/*.pen, components/_inventory.md, components/design-system.pen
```

**specs/.blueprint-state.yaml** — update:
- Add `3` to `completed_phases`
- Set `current_phase: 4`
- Update `last_updated`

## Output Checklist

Before completing, verify:
- [ ] Every P0 and P1 user story maps to at least one screen
- [ ] Every screen spec defines all 4 states (loading, error, empty, success) with specific descriptions
- [ ] Every screen spec lists data dependencies with API endpoint references
- [ ] Every screen spec lists all user interactions with outcomes
- [ ] Component inventory includes design tokens (colors, spacing, typography)
- [ ] Navigation map covers all screen-to-screen transitions
- [ ] .pen files exist for each screen (or pencil_available: false is set)
- [ ] Each .pen file has been verified with get_screenshot()
- [ ] No `{{...}}` placeholders remain in any generated file
- [ ] .blueprint-state.yaml shows `completed_phases: [1, 2, 3]`, `current_phase: 4`
