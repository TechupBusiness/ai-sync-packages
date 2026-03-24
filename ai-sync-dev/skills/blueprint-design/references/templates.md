# Design Templates

## specs/screens/_inventory.md

````markdown
# Screen Inventory

## Screens

| Screen | Route | Parent Stories | States | Auth Required |
|--------|-------|----------------|--------|---------------|
| {{screen-name}} | {{/route}} | {{US-NNN}} | loading, error, empty, success | {{yes/no}} |

## Navigation Map

```
{{screen}} --[action]--> {{screen}}
{{screen}} --[action]--> {{screen}}
  └── {{screen}} --[back]--> {{screen}}
```
````

## specs/screens/{name}.md

````markdown
# {{Screen Name}}

**Route**: `{{/route/path}}`
**Auth**: {{required | public}}
**Parent stories**: {{US-NNN, US-NNN}}

## Purpose

{{One sentence: what the user accomplishes on this screen}}

## States

### Loading
{{What the user sees while data loads — skeleton, spinner, specific placeholder}}

### Error
{{What the user sees on failure — error message, retry button, fallback content}}

### Empty
{{What the user sees when there's no data — illustration, CTA, onboarding prompt}}

### Success (Default)
{{The primary state with data present — describe layout and content}}

## Components

| Component | Type | Props/Config | Notes |
|-----------|------|--------------|-------|
| {{name}} | {{shared/local}} | {{key props}} | {{notes}} |

## Data Dependencies

| Data | Source | Shape |
|------|--------|-------|
| {{data}} | {{GET /endpoint}} | `{{TypeName}}` from architecture.md |

## User Interactions

| Trigger | Action | Result |
|---------|--------|--------|
| {{tap/click/submit}} {{element}} | {{what happens}} | {{outcome or navigation}} |
````

## specs/components/_inventory.md

````markdown
# Component Inventory

## Shared Components

| Component | Type | Props | Used In |
|-----------|------|-------|---------|
| {{name}} | {{button/card/input/modal/...}} | {{key props}} | {{screen1, screen2}} |

## Design Tokens

| Token | Value | Usage |
|-------|-------|-------|
| color-primary | {{#hex}} | Buttons, links, active states |
| color-background | {{#hex}} | Page backgrounds |
| color-surface | {{#hex}} | Cards, modals |
| color-text | {{#hex}} | Body text |
| color-text-secondary | {{#hex}} | Labels, placeholders |
| color-error | {{#hex}} | Error states, destructive actions |
| color-success | {{#hex}} | Success states, confirmations |
| spacing-xs | {{N}}px | Tight spacing |
| spacing-sm | {{N}}px | Compact spacing |
| spacing-md | {{N}}px | Default spacing |
| spacing-lg | {{N}}px | Section spacing |
| radius-sm | {{N}}px | Buttons, inputs |
| radius-md | {{N}}px | Cards |
| radius-lg | {{N}}px | Modals, panels |
| font-body | {{family}} | Body text |
| font-heading | {{family}} | Headings |
````
