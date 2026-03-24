# Pencil MCP Integration

Step-by-step workflow for generating .pen design files using the Pencil MCP tools. Follow this sequence exactly.

## Availability Check

Before starting, verify Pencil MCP tools are available by checking if `mcp__pencil__get_editor_state` is callable. If not available:
1. Skip all .pen file generation
2. Generate only .md screen specs
3. Note in `.blueprint-state.yaml`: `pencil_available: false`
4. The audit phase will flag missing .pen files as WARN (not FAIL)

## Step 1: Style Guide

```
1. Call get_style_guide_tags() to retrieve available tags
2. Select 5-10 tags that match the product's domain and aesthetic:
   - Include platform tag: "website", "mobile", or "webapp"
   - Include industry/domain tags from the vision
   - Include aesthetic tags if specified in vision constraints
3. Call get_style_guide(tags=[...]) with selected tags
4. Save the returned style guide — use it for all .pen files
```

## Step 2: Design System (.pen)

Create `specs/components/design-system.pen`:

```
1. Call get_guidelines(topic="design-system") for composition rules
2. Call open_document(filePathOrTemplate="specs/components/design-system.pen")
   — opening with a file path creates/opens the .pen at that path
3. Create reusable components matching the component inventory:
   - Use batch_design() to create component frames with reusable: true
   - Apply design tokens from components/_inventory.md
   - Apply style guide colors, typography, spacing
4. The file is auto-saved at the path specified in open_document()
5. Call get_screenshot() to verify each component visually
```

### Component Creation Pattern

For each shared component in the inventory:

```
frame=I(document, {
  type: "frame",
  name: "ComponentName",
  reusable: true,
  layout: "vertical",
  ...style properties from tokens
})
// Add children (text, icons, sub-frames)
child=I(frame, { type: "text", content: "Label", ...typography })
```

## Step 3: Per-Screen Designs (.pen)

For each screen in `specs/screens/_inventory.md`:

```
1. Read the screen's .md spec for layout, components, states, interactions
2. Determine guidelines topic:
   - Web → get_guidelines(topic="web-app")
   - Mobile → get_guidelines(topic="mobile-app")
   - Landing page → get_guidelines(topic="landing-page")
3. Call open_document(filePathOrTemplate="specs/screens/{screen-name}.pen")
4. Recreate shared components in this document:
   - Read design-system.pen with batch_get() to get component structures
   - Recreate needed components as reusable frames in this document
   - Each .pen file is self-contained (no cross-file refs)
5. Build the screen layout using batch_design():
   - Create the screen frame at the platform dimensions
   - Insert component instances (type: "ref") referencing local reusable frames
   - Add screen-specific elements
   - Design the "success" state as the primary frame
6. Create additional state frames:
   - Copy the success frame, modify for loading state
   - Copy again for error state
   - Copy again for empty state
7. File is auto-saved at the path specified in open_document()
8. Call get_screenshot() on each state frame to verify
```

### Screen Dimensions

| Platform | Width | Height |
|----------|-------|--------|
| Web (desktop) | 1440 | 900 |
| Web (tablet) | 768 | 1024 |
| Web (mobile) | 375 | 812 |
| Mobile (iOS) | 390 | 844 |
| Mobile (Android) | 360 | 800 |

Create the primary dimension for the detected platform. Add responsive variants if the platform is web or hybrid.

### Layout Strategy

```
1. Use find_empty_space_on_canvas() to position each screen frame
2. Use snapshot_layout() to verify no overlaps after placement
3. Use get_screenshot() on the final screen to visual-check:
   - Text is readable (not clipped, not overlapping)
   - Spacing is consistent with design tokens
   - Components match the design system
   - All 4 states are visually distinct
```

## Step 4: Verification

After all .pen files are created:

```
1. For each .pen file:
   a. Call get_screenshot() on the root frame
   b. Visually verify: layout, spacing, text readability, component consistency
   c. If issues found → fix with batch_design() updates, re-screenshot
2. Cross-check: every screen in _inventory.md has a corresponding .pen file
3. Cross-check: every shared component in the design system is used in at least one screen
```

## Parallelism

Individual screen .pen files are independent. The orchestrator can create multiple screen designs in parallel using sub-agents, as long as:
- The design system .pen is created first (shared dependency)
- Each sub-agent has access to the style guide and design tokens
- Each sub-agent creates its own document (no concurrent writes to same .pen)
