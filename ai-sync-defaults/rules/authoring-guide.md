---
name: authoring-guide
description: Guide for drafting ai-sync rules, personas, commands, and hooks
version: 1.0.0
always_apply: false
globs:
  - ".ai-sync/**/*.md"
  - ".ai-sync/**/*.yaml"
  - ".ai-sync/config.yaml"
category: documentation
priority: medium
---

# AI Tool Sync Authoring Guide

Guide for drafting rules, personas, commands, and hooks using the generic ai-sync format. This skill provides a single source of truth for creating AI assistant configurations that work across multiple platforms.

## Quick Reference

| Kind | Location | Required Fields | Primary Use |
|------|----------|-----------------|-------------|
| rule | `.ai-sync/rules/` | `name` | Guidelines, skills, context |
| persona | `.ai-sync/personas/` | `name` | Agent personalities |
| command | `.ai-sync/commands/` | `name` | Slash commands |
| hook | `.ai-sync/hooks/` | `name`, `event` | Lifecycle automation |

## File Structure

All content files use Markdown with YAML frontmatter:

```markdown
---
name: my-content
description: Human-readable description
version: 1.0.0
# ... additional fields ...
---

# Markdown Body

Content goes here...
```

## Creating Rules

Rules provide guidelines, skills, and context that AI assistants reference when working on specific files or tasks.

### Rule Schema

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | string | Yes | - | Unique identifier (becomes slug) |
| `description` | string | No | - | Human-readable description |
| `version` | string | No | - | Semver version (e.g., `1.0.0`) |
| `globs` | string[] | No | - | File patterns to trigger this rule. **Omit entirely** to mean "apply always" — the rule inlines into root entry-points (`CLAUDE.md` / `AGENTS.md`). `globs: []` (explicit empty) is rejected as ambiguous. |
| `always_apply` | boolean | No | - | **DEPRECATED** (`AISYNC_DEP_001`, removed in ai-sync v2.0). Build emits a one-shot deprecation warning per process listing every affected rule. Migration: drop the line — with no `globs`, the rule already applies always. |
| `targets` | string[] | No | `[cursor, claude, factory]` | Platforms to generate for |
| `requires` | string[] | No | - | Dependent rules to load together |
| `category` | string | No | - | Organization category |
| `priority` | string | No | `medium` | Priority level (`low`, `medium`, `high`) |

> **Apply-always semantics** (since ai-sync fn-2): a rule that **omits both** `always_apply` and `globs` is treated as **always-apply**. This is the canonical shape going forward; the legacy `always_apply: true` form is soft-deprecated. The structurally-bad cases are now parse-time errors:
>
> - `globs: []` → omit `globs` (apply always) or list at least one pattern.
> - `always_apply: false` with no `globs` → remove the rule, or add `globs`.

### Rule Categories

- `core` - Fundamental project rules
- `infrastructure` - Deployment, CI/CD, environment
- `testing` - Test patterns and requirements
- `security` - Security guidelines
- `documentation` - Documentation standards
- `tooling` - Development tools and workflows
- `other` - Uncategorized

### Rule Example

```yaml
# Scoped rule: globs trigger attachment
---
name: database
description: Database schema and query patterns
version: 1.0.0
globs:
  - "**/*.sql"
  - "**/migrations/**"
  - "**/supabase/**"
targets:
  - cursor
  - claude
  - factory
category: infrastructure
priority: high
---

# Database Guidelines

[Rule content here...]
```

```yaml
# Apply-always rule: omit globs (the new canonical shape)
---
name: project
description: Always-on project context
version: 1.0.0
targets:
  - cursor
  - claude
  - factory
priority: high
---

# Project

[Always-on context that inlines into CLAUDE.md / AGENTS.md...]
```

## Creating Personas

Personas define agent personalities with specific capabilities, tools, and communication styles.

### Persona Schema

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | string | Yes | - | Unique identifier |
| `description` | string | No | - | Human-readable description |
| `version` | string | No | - | Semver version |
| `tools` | string[] | No | - | Available tools (see Tool Mappings) |
| `model` | string | No | `default` | Model to use (see Model Mappings) |
| `targets` | string[] | No | `[cursor, claude, factory]` | Target platforms |
| `traits` | object | No | - | Additional characteristics |

### Persona Example

```yaml
---
name: code-reviewer
description: Thorough code reviewer focused on quality
version: 1.0.0
tools:
  - read
  - search
  - glob
model: default
targets:
  - cursor
  - claude
  - factory
---

# Code Reviewer

You are a meticulous code reviewer...

## Review Focus Areas

- Correctness and logic errors
- Security vulnerabilities
- Performance implications
- Code maintainability
```

## Creating Commands

Commands define slash commands that users can invoke to trigger specific actions.

### Command Schema

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | string | Yes | - | Command name (invoked as `/name`) |
| `description` | string | No | - | Human-readable description |
| `version` | string | No | - | Semver version |
| `execute` | string | No | - | Script or command to run |
| `args` | object[] | No | - | Command arguments |
| `globs` | string[] | No | - | File patterns where relevant |
| `allowedTools` | string[] | No | - | Tool restrictions |
| `variables` | object[] | No | - | Variable placeholders |
| `targets` | string[] | No | `[cursor, claude, factory]` | Target platforms |

### Argument Schema

```yaml
args:
  - name: environment
    type: string          # string, number, boolean
    description: Target environment
    default: staging
    choices: [staging, production]
    required: false
```

### Variable Placeholders

Commands support these built-in variables:
- `$ARGUMENTS` - User-provided arguments after the command name

### Command Example

```yaml
---
name: deploy
description: Deploy to specified environment
version: 1.0.0
execute: scripts/deploy.sh
args:
  - name: environment
    type: string
    default: staging
    choices: [staging, production]
targets:
  - cursor
  - claude
  - factory
---

# Deploy Command

Deploy the application to the specified environment.

Usage: `/deploy [environment]`

The command will:
1. Run pre-deployment checks
2. Build the application
3. Deploy to $ARGUMENTS (default: staging)
4. Run smoke tests
```

## Creating Hooks

Hooks automate actions based on lifecycle events during AI assistant sessions.

### Hook Schema

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `name` | string | Yes | - | Unique identifier |
| `description` | string | No | - | Human-readable description |
| `version` | string | No | - | Semver version |
| `event` | string | Yes | - | Trigger event (see Event Mappings) |
| `tool_match` | string | No | - | Pattern to match tools |
| `execute` | string | No | - | Script to execute |
| `targets` | string[] | No | `[cursor, claude, factory]` | Target platforms |

### Generic Events

| Event | Description | Can Block |
|-------|-------------|-----------|
| `PreToolUse` | Before any tool execution | Yes |
| `PostToolUse` | After tool execution | No |
| `PreMessage` | Before processing user message | Yes |
| `PostMessage` | After generating response | No |
| `PreCommit` | Before git commit | Yes |

### Tool Match Patterns

```yaml
# Match specific tool
tool_match: "Bash"

# Match tool with arguments
tool_match: "Bash(git commit*)"

# Match multiple tools
tool_match: "Write|Edit"
```

### Hook Example

```yaml
---
name: pre-commit-lint
description: Run linting before commits
version: 1.0.0
event: PreToolUse
tool_match: "Bash(git commit*)"
execute: npm run lint
targets:
  - cursor
  - claude
  - factory
---

# Pre-commit Lint Hook

Automatically runs linting before git commits to ensure code quality.
```

## Tool Mappings

Generic tool names map to platform-specific equivalents:

| Generic | Cursor | Claude | Factory | Description |
|---------|--------|--------|---------|-------------|
| `read` | Read | Read | Read | Read file contents |
| `write` | Create | Write | Write | Create new files |
| `edit` | Edit | Edit | Edit | Modify existing files |
| `execute` | Execute | Bash | Bash | Run shell commands |
| `search` | Grep | Search | Search | Search file contents |
| `glob` | Glob | Glob | Glob | Find files by pattern |
| `fetch` | FetchUrl | Fetch | Fetch | HTTP requests |
| `ls` | LS | LS | LS | List directory contents |

## Model Mappings

Generic model values for the `model` field:

| Value | Description |
|-------|-------------|
| `default` | Platform's default model |
| `fast` | Optimized for speed (lower latency) |
| `powerful` | Highest capability model |
| `inherit` | Use the current session's model |

## Platform Extensions

Override generic settings for specific platforms using extension objects:

```yaml
---
name: my-rule
description: Rule with platform overrides
version: 1.0.0
globs:
  - "**/*.ts"
targets:
  - cursor
  - claude
  - factory
# Platform-specific overrides
cursor:
  alwaysApply: true           # Cursor uses camelCase
  globs: ["src/**/*.ts"]      # Different globs for Cursor
claude:
  import_as_skill: true       # Claude-specific option
factory:
  reasoningEffort: high       # Factory-specific option
---
```

### Cursor Extensions

| Field | Type | Description |
|-------|------|-------------|
| `alwaysApply` | boolean | Cursor's native field (camelCase). Maps to the generic `always_apply` field. Setting either triggers the `AISYNC_DEP_001` deprecation warning — prefer omitting both and letting "missing globs = apply always" carry the semantics. |
| `globs` | string[] | Override glob patterns |
| `description` | string | Override description |
| `allowedTools` | string[] | Tool restrictions for commands |

### Claude Extensions

| Field | Type | Description |
|-------|------|-------------|
| `import_as_skill` | boolean | Import as a skill in CLAUDE.md |
| `tools` | string[] | Tool restrictions |
| `model` | string | Model override |

### Factory Extensions

| Field | Type | Description |
|-------|------|-------------|
| `allowed-tools` | string[] | Tool allowlist |
| `tools` | string[] | Tool restrictions |
| `model` | string | Model override |
| `reasoningEffort` | string | Reasoning effort (`low`, `medium`, `high`) |

## Naming Conventions

### Slugification Rules

Names are converted to slugs for filenames and identifiers:

- Lowercase all characters
- Replace spaces with hyphens
- Replace underscores with hyphens
- Remove special characters
- Collapse multiple hyphens

**Examples:**
- `Database Rules` → `database-rules`
- `API_Gateway` → `api-gateway`
- `My Cool Feature!` → `my-cool-feature`

### File Naming

- All content files use `.md` extension
- Filename should match the `name` field slug
- Use lowercase with hyphens (kebab-case)

```
.ai-sync/
├── rules/
│   ├── database-rules.md      # name: database-rules
│   └── api-guidelines.md      # name: api-guidelines
├── personas/
│   └── code-reviewer.md       # name: code-reviewer
├── commands/
│   └── deploy.md              # name: deploy
└── hooks/
    └── pre-commit-lint.md     # name: pre-commit-lint
```

## Validation Workflow

Before committing new content, follow this checklist:

### 1. Dry-run Preview

Preview what will be generated without writing files:

```bash
ai-sync create <kind> "<name>" --dry-run
```

### 2. Lint Validation (Rules Only)

For rules, verify linting passes:

```bash
ai-sync create rule "<name>" --dry-run --run-lint
```

### 3. Full Sync Test

Test the complete sync process:

```bash
ai-sync --dry-run
```

### 4. Validate Configuration

Check for schema and configuration errors:

```bash
ai-sync validate --verbose
```

### 5. Warning Checklist

Review output for warnings about:
- Unknown tool names (not in mapping)
- Unmapped model values
- Missing required fields
- Invalid glob patterns
- Schema validation errors

## Collision and Overwrite Guidance

### Same-name content collisions (rules / agents / commands / skills)

Files with matching names will collide during sync:
- Project files in `.ai-sync/` override defaults
- Use `overrides/` folder to customize default content

#### Avoiding Conflicts

1. Check existing files: `ai-sync status`
2. Use unique names for project-specific content
3. Override defaults explicitly in `overrides/` folder

#### Force Overwrite

When intentionally replacing files:

```bash
ai-sync --force
```

Or in configuration:

```yaml
output:
  overwrite: true
```

### Overlay destination collisions (loader vs loader)

When **two loaders ship overlays that resolve to the same destination path** — e.g. both ship `.husky/pre-commit` — `ai-sync build` **fails the build** with a deterministic error:

```
Overlay error: Collision: ".husky/pre-commit" deployed by:
  - ai-sync-defaults (.../ai-sync-packages/ai-sync-defaults/overlays/.husky/pre-commit)
  - my-flutter-pack (.../ai-sync-packages/my-flutter-pack/overlays/.husky/pre-commit)

Use `remap_to` (with `source_match` to scope) to send one of them
elsewhere. ai-sync does not silently pick a winner.
```

The build exits non-zero. Sources in the message are sorted by `(destPath, friendlyLoaderName)` so the output is byte-stable. **Identical content does not short-circuit the check** — even if both files are byte-identical, the build still fails. This is by design: silent overwrites hide intent.

#### Resolving with `source_match` + `remap_to`

The consumer adds an overlay rule scoped to a single loader (via `source_match`) that rewrites the destination (via `remap_to`):

```yaml
# Project config.yaml — both loaders ship .husky/pre-commit
loaders:
  - type: local
    source: ai-sync-packages/ai-sync-defaults
  - type: local
    source: ai-sync-packages/my-flutter-pack

overlays:
  rules:
    # Scope a remap to one loader; the other keeps the default destination.
    - pattern: ".husky/pre-commit"
      source_match: "**/my-flutter-pack/**"
      remap_to: ".husky/pre-commit-flutter"
      executable: true
```

After `ai-sync build`:

```
.husky/pre-commit            # from ai-sync-defaults
.husky/pre-commit-flutter    # from my-flutter-pack (executable)
```

You can also remap both side-by-side if neither loader's "default" location should win:

```yaml
overlays:
  rules:
    - pattern: ".husky/pre-commit"
      source_match: "*ai-sync-defaults*"
      remap_to: ".husky/pre-commit-defaults"
      executable: true
    - pattern: ".husky/pre-commit"
      source_match: "*my-flutter-pack*"
      remap_to: ".husky/pre-commit-flutter"
      executable: true
```

### Project overlay vs loader overlay (silent project win)

A project overlay in `.ai-sync/overlays/<path>` at the same destination as a loader overlay is **not** a collision — the project overlay wins, and the loader overlay is silently dropped (debug-logged). This is the intended override pattern: consumers shadow loader files by placing their own at the same path.

If you want a loud signal when a loader adds a new overlay you weren't aware of, use `source_match` + `remap_to` to send the loader overlay to a sibling path explicitly, instead of relying on the silent project-side win.

## Common Pitfalls

### Frontmatter Mistakes

| Mistake | Correct |
|---------|---------|
| Using `always_apply: true` (deprecated) | Omit both `always_apply` and `globs` — that's the new "apply always" shape (`AISYNC_DEP_001`, removed in ai-sync v2.0). |
| `globs: []` (explicit empty array) | Either omit `globs` (apply always) or list at least one pattern (scoped). Empty arrays are now a parse-time validation error. |
| `always_apply: false` with no `globs` | Remove the rule (it would never apply) or add `globs`. Now a parse-time validation error. |
| `alwaysApply: true` | If you really need the legacy form, write `always_apply: true` (snake_case) — but prefer the omit-`globs` shape. |
| `toolMatch: "..."` | `tool_match: "..."` (snake_case) |
| Missing `name` field | Always include `name` |
| Invalid `version` format | Use semver: `1.0.0` |

### Glob Pattern Issues

| Issue | Solution |
|-------|----------|
| Pattern doesn't match | Test with `ai-sync validate --verbose` |
| Too broad patterns | Be specific: `src/**/*.ts` not `**/*` |
| Missing quotes | Quote patterns with special chars |

### Platform-Specific Gotchas

- **Cursor**: Uses camelCase in frontmatter (`alwaysApply`, `allowedTools`)
- **Claude**: Skills are auto-invoked based on relevance, not globs
- **Factory**: Droids are invoked explicitly, not auto-triggered

## Output Locations

After running `ai-sync`, content is generated to:

| Kind | Cursor | Claude | Factory |
|------|--------|--------|---------|
| Rules | `.cursor/rules/*.mdc` | `.claude/skills/<name>/SKILL.md` | `.factory/skills/<name>/SKILL.md` |
| Personas | `.cursor/commands/roles/*.md` | `.claude/agents/<name>.md` | `.factory/droids/<name>.md` |
| Commands | `.cursor/commands/*.md` | `.claude/commands/*.md` | `.factory/commands/*.md` |
| Hooks | `.cursor/hooks.json` | `.claude/settings.json` | `~/.factory/settings.json` |

## Extending Loaded Content

When content is loaded from an external source (git, npm, local folder, url, claude-plugin), you can extend or modify it **without forking** — via the `overrides` block on the loader in `config.yaml`.

### Quick Example

```yaml
loaders:
  - type: git
    source: github:company/ai-pack
    variables:
      company_name: "Acme"
    agents:
      model: opus                       # Default model for all agents from this loader
      skills:
        add: [research]
      overrides:
        architect:
          model: sonnet                 # Per-agent model override
    overrides:
      agents:
        architect:
          targets: [claude]
          variables: { team: "Backend" }
          modifications:
            - position: end
              content: "## Team Notes\nYou work for {{company_name}}, {{team}}."
            - position: replace
              section: "## Instructions"
              content: { file: "custom-instructions.md" }
      rules:
        security:
          priority: high
          modifications:
            - position: end
              section: "## Checklist"
              content: "- Additional check"
```

### Modification Positions

Each entry in `modifications:` is one of three types, selected via the optional `type` field:

| `type` | Purpose |
|--------|---------|
| `section` (default, may be omitted) | Heading-anchored edits on markdown — the `position` + `section` pair below |
| `regex` | Pattern-based insert/replace/delete — see [Regex Modifications](#regex-modifications) |
| `script` | Pipe content through a user command — see [Script Modifications](#script-modifications) |

The `position`-based shape below is what you get when `type` is omitted (or `type: section`):

| Position | Without `section` | With `section` |
|----------|-------------------|----------------|
| `replace` | Replaces entire body (keeps frontmatter) | Replaces content between heading and next same-level heading |
| `end` | Appends at EOF | Appends before next same-level heading (or EOF) |
| `beginning` | Inserts right after frontmatter | Inserts right after section heading |
| `delete` | Removes the entire item from the load result | Removes the section (heading + body) |

Section targeting matches the **first occurrence** of the heading (e.g. `"## Instructions"`). If the section is missing, the modification is skipped with a warning — unless `create_if_missing: true` is set (see below).

### Deleting Content

Remove a section:

```yaml
overrides:
  agents:
    architect:
      modifications:
        - position: delete
          section: "## Deprecated Notes"
```

Remove an entire item (drops it from the load result before generation):

```yaml
overrides:
  skills:
    unwanted-skill:
      modifications:
        - position: delete     # no section → whole-item delete
```

Notes:
- Whole-item delete short-circuits any other modifications on the same item.
- Delete ignores `content:` if present (warns).
- A whole-body replace (`position: replace` with no `section:`) wipes the current content and substitutes its own. Modifications listed **before** it are redundant and skipped (with a warning); modifications listed **after** it run on the replaced content — useful for "replace the whole file, then patch a few lines via regex".
- To hide items without loading them in the first place, prefer the loader's `use:` filter.

### Creating Missing Sections

Opt in to section creation with `create_if_missing`:

```yaml
modifications:
  - position: end
    section: "## Team Notes"
    create_if_missing: true
    content: |
      Apply the Acme template.
```

- Valid only with `position: end` or `position: beginning`.
- Heading level is derived from leading `#` chars in `section` (default `##` if none).
- Makes the new section a first-class target for later layers to extend.

### Regex Modifications

Use `type: regex` to edit content by pattern instead of by markdown heading. This covers non-markdown files (JSON/YAML/scripts/dotfiles) and markdown edits anchored on non-heading lines.

```yaml
modifications:
  - type: regex
    pattern: '"version"\s*:\s*"[^"]*"'
    replace: '"version": "{{company_version}}"'
    all: false           # default — replace first match only
    required: true       # default — warn/error if 0 matches
  - type: regex
    pattern: '^## Checklist$'
    flags: m
    insert_after: |

      - New team-specific check
    all: false
```

Supported operations (pick exactly one):

| Field | Effect |
|-------|--------|
| `replace: "..."` | Replace each match with the string. Supports JS backrefs (`$1`, `$2`, `$&`). |
| `insert_before: ...` | Insert content immediately before each match. Accepts `string` \| `{ file: ... }` \| `{ bash: ... }`. |
| `insert_after: ...` | Insert content immediately after each match. Same source forms as above. |
| `delete: true` | Remove each match (equivalent to `replace: ""`). |

Other fields:

| Field | Type | Default | Notes |
|-------|------|---------|-------|
| `pattern` | string | required | JS regex source (no surrounding `/.../`). |
| `flags` | string | `""` | Subset of `gmisuy`. `g` is managed automatically via `all`. |
| `all` | boolean | `false` | `true` → replace every match; `false` → first match only. |
| `required` | boolean | `true` | If `true`, zero matches warns (or aborts under `strict`). |
| `strict` | boolean | `false` | Escalate failures to build errors — see [Strict Mode](#strict-mode). |

Notes:
- Backrefs use JavaScript syntax (`$1`, `$2`, `$&`), **not** sed's `\1`.
- Invalid regex patterns or flag strings produce a warning (or fatal under strict).
- `insert_before` / `insert_after` values support the same `{ file: ... }` / `{ bash: ... }` forms as `content:` (see [Content Sources](#content-sources)).

### Script Modifications

Use `type: script` to pipe the current file content through an external command. The command receives the file on stdin; its stdout becomes the new content.

```yaml
modifications:
  - type: script
    command: "jq '.devDependencies |= del(.\"@types/removed\")'"
    timeout: 10000           # ms, default 10000
    max_output_bytes: 10485760  # default 10 MB
```

| Field | Type | Default | Notes |
|-------|------|---------|-------|
| `command` | string | required | Any shell command. Stdin = current content, stdout = new content. |
| `timeout` | number | `10000` | Milliseconds. On timeout, the process group is killed. |
| `max_output_bytes` | number | `10485760` (10 MB) | Stdout cap; exceeding it fails the modification. |
| `strict` | boolean | `false` | Escalate failures to build errors — see [Strict Mode](#strict-mode). |

Environment variables exposed to the script:

| Variable | Value |
|----------|-------|
| `AI_SYNC_TARGET_ID` | Stable logical id — `${contentType}/${itemName}` (e.g. `skills/pdf`) for loader-level modifications, or `skill-aux:${skillName}/${path}` (e.g. `skill-aux:pdf/scripts/extract.py`) for aux-file modifications. **Not** a filesystem path. |
| `AI_SYNC_CONFIG_DIR` | Absolute path to the config directory (the one holding `config.yaml`). Use it to resolve patch files. |

Trust-boundary warning:
- `type: script` runs arbitrary shell from your YAML — the same risk surface as a `{ bash: "..." }` content source. Review loader sources (and wrapper packages) before enabling.
- Non-zero exit code always fails the modification (regardless of `strict`). The `strict` flag controls whether a failure warns or aborts the whole build.
- stderr is surfaced as a warning, prefixed with the modification's logical id.
- `ai-sync build --dry-run` (user-level) skips script execution; regular builds always execute it.

### Strict Mode

Every modification (all three types) accepts an optional `strict` flag:

```yaml
modifications:
  - type: regex
    pattern: '"version"\s*:\s*"[^"]*"'
    replace: '"version": "1.2.3"'
    strict: true        # abort the build if this mod cannot apply
```

Behavior:

| `strict` | Failure handling |
|----------|------------------|
| `false` (default) | Log a warning and skip the modification. The build continues and other items still generate. |
| `true` | Collect the failure as a **fatal** error. The build runs to completion, then `ai-sync build` exits non-zero. |

What counts as a failure:
- `type: section`: section not found (without `create_if_missing`), attempting to create with an unsupported position, etc.
- `type: regex`: invalid pattern/flags, or zero matches when `required: true`.
- `type: script`: non-zero exit, timeout, stdout exceeds `max_output_bytes`.

Use `strict: true` for CI builds where silent skips would hide drift (e.g. a version-bump regex must match exactly once; a `jq` patch must succeed). Leave the default `false` when the modification is a "best-effort" tweak.

### Content Sources

Any `content` field accepts three forms:

```yaml
content: "Inline string"           # Direct
content: { file: "addon.md" }      # Relative to config dir
content: { bash: "cat extra.txt", timeout: 5000 }  # Command output
```

### Variables

Template variables substitute `{{placeholder}}` in the body. Precedence (highest wins):

1. Per-item `variables` under `overrides.<type>.<name>.variables`
2. Loader-level `variables`
3. Frontmatter `optional_variables` defaults

Variable values themselves also accept `{ file: ... }` and `{ bash: ... }`.

### Frontmatter Overrides

Set any supported frontmatter field under `overrides.<type>.<name>`: `description`, `targets`, `priority`, `category`, `globs`, etc. For rules and skills, several fields (`description`, `targets`, `globs`, `priority`, `category`, `compatibility`) also accept `{ file: ... }` / `{ bash: ... }`.

> **`always_apply` overrides are deprecated** (`AISYNC_DEP_001`, removed in ai-sync v2.0). Setting `always_apply` from a loader override still works but emits the deprecation warning naming the affected rule. Migration: drop the override line. If you need the upstream rule to apply always, the canonical fix is at the source (the rule should be authored without `globs`). There is no field-deletion override that lets a consumer drop an inherited `globs` value purely from `config.yaml` — if the upstream rule is scoped and you need it to apply always in your project, fork it into `.ai-sync/rules/`. For scoped attachment, just set `globs:` directly.

### Agent-Specific Overrides

Agents have extra loader-level fields **outside** the generic `overrides` block:

```yaml
loaders:
  - type: git
    source: github:company/pack
    agents:
      model: opus                 # Default for all agents
      skills:
        add: [research]
        remove: []
      overrides:
        <agent-name>:
          model: sonnet           # Per-agent
          skills:
            add: [analysis]
            remove: [research]
```

Precedence: agent frontmatter → loader-level `agents` → per-agent `agents.overrides`.

### Skill-Bundle File Overrides

Skills loaded as directory bundles have auxiliary files alongside `SKILL.md` (e.g. `scripts/`, `references/`, `assets/`). Use `overrides.skills.<name>.files.<path>` to replace, append, delete, or add those files. Each key is a path relative to the skill's source dir.

```yaml
overrides:
  skills:
    pdf:
      files:
        # Replace an existing aux file (any file type)
        "scripts/extract.py":
          content: { file: "patches/my-extract.py" }

        # Delete an aux file
        "references/old.md":
          delete: true

        # Append at EOF (.md only)
        "references/guide.md":
          append: |
            ## Team Notes
            Apply the Acme template.

        # Modifications on an aux file. Section mods (`type: section` /
        # omitted) require a `.md` file because they anchor on ATX headings.
        # Regex and script mods work on any text file (JSON/YAML/scripts/
        # dotfiles), so they're accepted on non-`.md` paths too.
        "references/guide.md":
          modifications:
            - position: replace
              section: "## Overview"
              content: { file: "patches/overview.md" }
            - position: end
              section: "## New Section"
              create_if_missing: true
              content: "..."
            # Regex / script modifications on `.md` aux files
            - type: regex
              pattern: '\bupstream-name\b'
              replace: "Acme"
              all: true
              strict: true
            - type: script
              command: "sed 's/TODO/DONE/g'"
              timeout: 5000

        # Non-markdown aux file: regex / script mods work, section mods do not.
        "scripts/extract.py":
          modifications:
            - type: regex
              pattern: '^DEBUG\s*=\s*True$'
              flags: m
              replace: 'DEBUG = False'
            - type: script
              command: "black -q -"
              timeout: 10000

        # Add a brand-new file not in the upstream bundle
        "scripts/post-process.sh":
          content: { file: "patches/post-process.sh" }
```

Rules:
- Each path uses **exactly one** of `delete` / `content` / `append` / `modifications`.
- `append` is `.md`-only (line-level append); it requires the path to exist in the source bundle.
- `modifications` mixes by type: section modifications (`type: section` or omitted) are `.md`-only because they anchor on ATX headings; `type: regex` and `type: script` modifications work on **any text file** (JSON/YAML/scripts/dotfiles) and are the recommended way to patch non-markdown aux files. On a non-`.md` path, section entries in the list are skipped with a warning and regex/script entries pass through.
- `content` can replace an existing file or add a new one (any file type).
- The special path `SKILL.md` is rejected — use top-level `modifications:` instead.
- Keys must be relative paths inside the skill dir; `..` and absolute paths are rejected.
- Glob patterns are not supported — list each file explicitly.

### Wrapper Packages (Transitive Loaders)

A loader package can declare its own dependencies via a `loaders.yaml` file at its root. This lets you build **wrapper packages** that pull in third-party content and apply curated overrides before it reaches the consumer.

Example wrapper layout:

```
ai-sync-packages/my-anthropic-skills/
├── loaders.yaml
└── patches/
    ├── pdf-intro.md
    └── post-process.sh
```

`loaders.yaml`:

```yaml
version: 1.0.0
loaders:
  - type: git
    source: github:anthropics/skills#main
    use:
      skills: [pdf, docx, xlsx]
    overrides:
      skills:
        pdf:
          targets: [claude]
          modifications:
            - position: replace
              section: "## Overview"
              content: { file: "patches/pdf-intro.md" }
          files:
            "scripts/post-process.sh":
              content: { file: "patches/post-process.sh" }
        docx:
          modifications:
            - position: delete
              section: "## Deprecated"
```

Consumer project `config.yaml` just lists the wrapper:

```yaml
loaders:
  - type: local
    source: ai-sync-packages/my-anthropic-skills
    use:
      skills: '*'
```

Behavior:
- Local paths in transitive `loaders.yaml` resolve relative to the package dir (not the consumer's project root).
- File-based overrides (`{ file: "patches/..." }`) resolve relative to the `loaders.yaml` that declared them.
- Depth is limited (default 5); cycles are detected and skipped.
- The consumer can **further** override content on top of the wrapper's overrides by listing the wrapper in their own `overrides.skills.<name>` block.
- URL loaders have no transitive support.

### Capability Matrix by Content Type

| Capability | Rules | Agents | Commands | Skills (SKILL.md) | Skill aux `.md` | Skill aux (other) | Workflows | MCP | Overlays |
|---|---|---|---|---|---|---|---|---|---|
| Full-body replace | ✅ | ✅ | ✅ | ✅ | ✅ (via `files.content`) | ✅ (via `files.content`) | ❌ | ❌ | ❌ |
| Append / prepend (no section) | ✅ | ✅ | ✅ | ✅ | ✅ (via `files.append`) | ❌ | ❌ | ❌ | ❌ |
| Section replace / append / beginning | ✅ | ✅ | ✅ | ✅ | ✅ (via `files.modifications`) | ❌ | ❌ | ❌ | ❌ |
| Section delete | ✅ | ✅ | ✅ | ✅ | ✅ (via `files.modifications`) | ❌ | ❌ | ❌ | ❌ |
| Regex modification (`type: regex`) | ✅ | ✅ | ✅ | ✅ | ✅ (via `files.modifications`) | ❌ | ❌ | ❌ | ❌ |
| Script modification (`type: script`) | ✅ | ✅ | ✅ | ✅ | ✅ (via `files.modifications`) | ❌ | ❌ | ❌ | ❌ |
| `strict: true` (abort on failure) | ✅ | ✅ | ✅ | ✅ | ✅ (via `files.modifications`) | ❌ | ❌ | ❌ | ❌ |
| Whole-item / file delete | ✅ | ✅ | ✅ | ✅ | ✅ (via `files.delete`) | ✅ (via `files.delete`) | ❌ | ❌ | ❌ |
| Add new file | ❌ | ❌ | ❌ | ❌ | ✅ (via `files.content`) | ✅ (via `files.content`) | ❌ | ❌ | ❌ |
| `create_if_missing` for sections | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Frontmatter overrides | ✅ | ✅ | ✅ | ✅ | — | — | ❌ | ❌ | ❌ |
| Variables (`{{var}}`) | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Type-specific overrides | — | `model`, `skills` | — | — | — | — | — | — | `strategy` only |
| Resolvable fields (`file`/`bash`) | ✅ | — | — | ✅ | — | — | — | — | — |

### Known Limitations

- **Variables inside skill aux `.md` files**: `{{var}}` substitution currently runs only against the primary content (`SKILL.md`, rule, agent, command body), not against aux-file content. Embed substitutions in the override's `content` / `append` payload instead.
- **Workflows**: no override support. The loader passes workflows through unchanged.
- **MCP servers**: no per-server override in loader config. Loaders can supply MCP via `mcp/servers.yaml` (unified format) or a plugin bundle's `.mcp.json` (Claude plugin format) — both flow through `result.mcpServers`. When two sources declare a server with the same name, **project MCP** (from your `.ai-sync/mcp.yaml` / `mcp/*.json`) wins over loader-supplied MCP. Within the loader layer itself, last-loaded wins. There's no deep-merge yet (e.g. a project can't add `env:` attrs to a loader-declared server without replacing it whole — tracked as future `mcp_overrides:` feature).
- **Overlays**: not a parsed content type — only deployment strategy (`copy` / `merge_overlay_wins` / `merge_generator_wins` / `symlink` / `symlink_merge` / `keep`) is configurable.
- **Cross-loader dedup**: if two loaders produce content with the same name, the later loader wins. ai-sync now logs a warning naming both sources; use `namespace:` on the loader (see "Loading Claude Plugins") to prevent collisions.
- **Aux file glob patterns**: `files:` keys must be literal relative paths — no `scripts/**/*.py` mass operations.

### When to Extend vs Fork

- **Extend (overrides)**: small tweaks, env-specific variables, team-specific sections, model/skills adjustments, patching a handful of aux files.
- **Fork (copy into `.ai-sync/`)**: major rewrites, restructuring a skill bundle, customizing workflows, or large-scale aux-file churn.

## Loading Claude Plugins

Claude Code plugins (like `flow-next`, `anthropics/skills`, or community plugins from a marketplace) can be loaded through ai-sync instead of installed globally into `~/.claude/`. This keeps your `~/.claude/` clean, pins exact plugin versions per project, lets you apply ai-sync overrides on top of upstream plugin content, and makes setup reproducible for your team.

### The three ways to load a plugin

Set `source_format: claude-plugin` on any `local`, `git`, or `npm` loader. ai-sync's plugin adapter understands the Claude plugin bundle layout (`skills/`, `agents/`, `commands/`, `hooks/hooks.json`, `.claude-plugin/plugin.json`, plus the Codex equivalents).

```yaml
# (1) Community plugin from GitHub, directly at the repo root
loaders:
  - type: git
    source: github:gmickel/flow-next
    source_format: claude-plugin

# (2) Plugin inside a marketplace-style repo (subdir)
  - type: git
    source: github:example/claude-plugins-marketplace
    subdir: plugins/my-plugin
    source_format: claude-plugin

# (3) Already-cached local plugin (e.g. after Claude's own install)
  - type: local
    source: ~/.claude/plugins/cache/flow-next/flow-next/0.29.2
    source_format: claude-plugin
```

The `subdir:` field also works for `type: npm`. For `type: local`, just point `source:` at the subdirectory directly.

### What gets loaded

- **Skills** — bundle-aware. `skills/<name>/SKILL.md` and any aux files alongside (`references/`, `scripts/`, etc.) travel together. Aux files are copied into the generated skill directory.
- **Agents** — every `.md` file under `agents/`.
- **Commands** — every `.md` file under `commands/`, including nested dirs.
- **Hooks** — `hooks/hooks.json` and `codex/hooks.json` are deployed as overlays merged into `.claude/settings.json` and `.codex/settings.json` respectively. The `hooks-wrapper` transform handles both shapes: already-wrapped (`{hooks: {...}}`) and bare event keys at the root. No configuration required.
- **MCP servers** — a bundle's `.mcp.json` (or `mcp.json`, or a manifest-declared path via `plugin.json#mcpServers`) is ingested into the structured MCP pipeline. `${CLAUDE_PLUGIN_ROOT}` substitution and environment variable interpolation are applied. Servers flow to each enabled target's generator and are written to that target's native path (e.g. `.mcp.json` at the project root for Claude Code).
- **Plugin metadata** — the `name`, `version`, and `description` from `.claude-plugin/plugin.json` are recorded on the load result for logs and conflict warnings.

Non-content directories like `docs/`, `scripts/`, and the plugin's own same-name dir (e.g. `flow-next/`) are ignored.

### Namespacing

Claude Code gives plugin content an automatic `plugin-name:skill-name` namespace, but only at the plugin-runtime level — project-level `.claude/skills/` and `.claude/commands/` don't support that syntax (names must be lowercase letters, numbers, and hyphens only).

ai-sync resolves this by rewriting content names to use a **hyphen-separated prefix**:

- For `source_format: claude-plugin`, the prefix is auto-derived from `.claude-plugin/plugin.json#name`. If the manifest says `name: flow-next`, the skill `work` becomes `flow-next-work`, written to `.claude/skills/flow-next-work/SKILL.md` and invoked as `/flow-next-work`.
- Set `namespace:` explicitly on any loader to override the auto-derivation (or to add prefixing to non-plugin loaders):
  ```yaml
  loaders:
    - type: git
      source: github:anthropics/skills
      namespace: anthropic   # → skill "pdf" becomes "anthropic-pdf"
  ```
- Names already beginning with `<namespace>-` (case-insensitive) are **not** re-prefixed. So if a plugin authored its skills with the prefix already (`name: flow-next-work`), the output stays `flow-next-work`.
- Command frontmatter that self-namespaces with a colon (`name: flow-next:plan`) is preserved verbatim — the `:` is a signal to leave the name alone.

When you load multiple plugins without namespaces, ai-sync logs a warning at build time naming the two conflicting sources. Use `namespace:` to silence it properly.

### Overriding plugin content

All the override mechanisms from "Extending Loaded Content" work against plugin-loaded content. Key thing to remember: override keys must match the **namespaced** name.

```yaml
loaders:
  - type: local
    source: ~/.claude/plugins/cache/flow-next/flow-next/0.29.2
    source_format: claude-plugin
    overrides:
      skills:
        "flow-next-work":          # namespaced key
          files:
            "phases.md":
              content: { file: "patches/flow-next-phases.md" }
          modifications:
            - position: end
              section: "## Team Additions"
              create_if_missing: true
              content: "Always ping @team before kicking off a ralph loop."
```

### Caveats

- The **whole plugin** becomes content in your project's output. If you only want a subset, use `use:` to filter by content name — remember to use the namespaced names.
- Plugin-sourced hooks from multiple plugins may overlap on the same event key (`PreToolUse`, etc.). Later loaders win; ai-sync logs the overlay conflict.
- `anthropics/skills` and similar Claude-skills collections that are **not** packaged as plugins (no `.claude-plugin/plugin.json`) should use `source_format: unified` or `source_format: claude` instead of `claude-plugin`. Use `namespace:` if you still want prefixing.
- Plugin commands that resolve external paths via `${CLAUDE_PLUGIN_ROOT}` need those paths re-evaluated in the project. Override the relevant command body or aux files if the default paths don't exist.

## Loader-Shipped Overlays

Loaders aren't limited to shipping parsed content (rules, agents, commands, skills). A loader package can also ship **overlay files** — arbitrary files that get deployed to the consumer's project tree at build time. Hook scripts (`.husky/pre-commit`), editor configs (`.editorconfig`, `.prettierrc`), CI workflows (`.github/workflows/*.yml`), and any other project-root artifacts are all candidates.

This makes a loader package a single source of truth for *all* repository state, not just AI-tool config.

### How a loader ships an overlay

Place files under the package's `overlays/` directory. The path inside `overlays/` mirrors the consumer project root:

```
my-pack/
├── loaders.yaml       (optional, for transitive deps)
└── overlays/
    └── .husky/
        └── pre-commit         # → consumer's .husky/pre-commit
```

When a consumer loads `my-pack`, ai-sync discovers the overlay during `applyOverlays`. By default it's copied to the mirrored destination using the consumer's `overlays.default_strategy` (default: `copy`).

### Consumer-side controls (the four new `OverlayRule` fields)

The consumer scopes loader overlays from their own `config.yaml` using the four fields documented in `docs/configuration.md` → "Overlays System". Worked examples for each:

#### `executable: true` — `chmod 0o755` after deploy

Use for hook scripts and helper binaries. The bit is set after byte-writing strategies (`copy`, `merge_overlay_wins`, `merge_generator_wins`); a no-op (debug log) for `symlink` / `symlink_merge` / `keep` and in `--dry-run`. Cross-platform safe (`fs.chmod` on Windows is a documented harmless no-op).

```yaml
# Consumer config.yaml
overlays:
  rules:
    - pattern: ".husky/pre-commit"
      executable: true
    - pattern: ".claude/hooks/*.sh"
      executable: true
```

#### `requires: { bash: ... }` — guard on a dependency check

Skip the overlay entirely (no remap, no collision check, no write) when a bash command exits non-zero, times out, or fails to spawn. cwd = `projectRoot`, env includes `AI_SYNC_PROJECT_ROOT`. Default timeout 5000ms. Skips surface as warnings in the build summary; `ai-sync build` has no `--strict` flag, so a skip never fails the build (the per-modification `strict: true` flag elsewhere in the system is unrelated).

```yaml
overlays:
  rules:
    # Only deploy the husky hook when husky is installed
    - pattern: ".husky/pre-commit"
      executable: true
      requires:
        bash: "command -v husky >/dev/null 2>&1"
        timeout: 2000
```

In `--dry-run`, `requires:` still executes (so the predicted file list reflects the real skip behavior); only `chmod` and writes are deferred.

#### `remap_to: <literal-path>` — rename the destination

Rewrite the deploy destination to a literal project-relative path. Useful when you want to keep a loader-shipped file but at a non-default location.

```yaml
overlays:
  rules:
    - pattern: ".husky/pre-commit"
      remap_to: ".husky/pre-commit-flutter"
      executable: true
```

Validation rules (enforced at config-load):

- `pattern` must also be literal when `remap_to` is set — no `*?[` characters in either field.
- Globs in `remap_to` itself are rejected.
- Any `..` segment is rejected (the validator splits on `/` and rejects if any segment equals `..` — not just paths that escape the root).
- Absolute paths and empty strings are rejected.

Symmetric: `remap_to` works on consumer overlays (`.ai-sync/overlays/`) too, not just loader-shipped ones.

#### `source_match: <substring or glob>` — scope a rule to one specific loader

Match a substring or glob against the overlay's `sourceId` (the loader directory it came from). Lets a single rule apply to one specific loader without affecting others that ship the same `relativePath`.

```yaml
overlays:
  rules:
    # Plain string → substring match against sourceId
    - pattern: ".husky/pre-commit"
      source_match: "ai-sync-flutter"
      remap_to: ".husky/pre-commit-flutter"
```

Without `source_match`, an overlay rule applies to every overlay whose `relativePath` matches `pattern`. With `source_match`, the rule does NOT match consumer overlays in `.ai-sync/overlays/` (which carry no `sourceId`).

**Match semantics for `source_match`** (different from `pattern`'s minimatch — `sourceId` is a free-form identifier, not a structured path):

- **No glob characters (`*`, `?`, `[`)** → plain **substring** match. `source_match: "ai-sync-flutter"` matches any `sourceId` that *contains* `"ai-sync-flutter"`. Dots, slashes, and other punctuation are matched literally.
- **Glob characters present** → anchored glob. `*` matches any chars (including `/`); `?` matches one char (including `/`); `[abc]` is a character class. So `source_match: "ai-sync-*"` matches a sourceId starting with `ai-sync-`, and `source_match: "*ai-sync-flutter*"` is equivalent to the plain-string substring form above.
- No braces, extglobs, or other minimatch sugar; malformed globs safely fall through to "no match."

### Two-loader walkthrough: scoped collision resolution

The motivating case for `source_match` is **two loaders shipping the same file**, where the consumer wants to keep both.

**Setup.** Two loaders both ship `.husky/pre-commit`:

```
ai-sync-packages/
├── ai-sync-defaults/
│   └── overlays/.husky/pre-commit       # generic husky hook
└── ai-sync-flutter/
    └── overlays/.husky/pre-commit       # Flutter-specific extra steps
```

**Default behavior.** Without overlay rules, `ai-sync build` fails:

```
Overlay error: Collision: ".husky/pre-commit" deployed by:
  - ai-sync-defaults (.../ai-sync-packages/ai-sync-defaults/overlays/.husky/pre-commit)
  - ai-sync-flutter  (.../ai-sync-packages/ai-sync-flutter/overlays/.husky/pre-commit)

Use `remap_to` (with `source_match` to scope) to send one of them
elsewhere. ai-sync does not silently pick a winner.
```

**Resolution.** The consumer scopes a remap to `ai-sync-flutter`'s overlay only, leaving `ai-sync-defaults` at the default destination:

```yaml
# Consumer config.yaml
loaders:
  - type: local
    source: ai-sync-packages/ai-sync-defaults
  - type: local
    source: ai-sync-packages/ai-sync-flutter

overlays:
  rules:
    # Loader A (ai-sync-defaults) implicitly keeps `.husky/pre-commit` —
    # no rule needed (the default strategy `copy` applies).
    #
    # Loader B (ai-sync-flutter): remap to a sibling path, scoped via
    # source_match so this rule does NOT also match loader A.
    - pattern: ".husky/pre-commit"
      source_match: "*ai-sync-flutter*"
      remap_to: ".husky/pre-commit-flutter"
      executable: true
```

After `ai-sync build`:

```
.husky/pre-commit            # from ai-sync-defaults (executable bit per its own rule)
.husky/pre-commit-flutter    # from ai-sync-flutter (executable: true above)
```

If you want the Flutter overlay to win at the default location instead, swap which loader you remap. If you want both at sibling paths and neither at the default name, write two rules — one per `source_match`. See the worked variant in [Collision and Overwrite Guidance](#overlay-destination-collisions-loader-vs-loader) above.

### Project-vs-loader precedence

A project overlay (`.ai-sync/overlays/.husky/pre-commit`) at the same destination as a loader overlay is **not** a collision — the project overlay wins, and the loader overlay is silently dropped (debug-logged). This is the intended override pattern: consumers shadow loader files by placing their own at the same path.

If you want a loud signal when a loader adds a new overlay you weren't expecting (e.g. for compliance reasons), use `source_match` + `remap_to` to send the loader overlay somewhere explicit, instead of relying on the silent project win.

### The husky pattern as canonical reference

ai-sync itself uses this exact pattern for its own pre-commit hook:

- `ai-sync-packages/ai-sync-defaults/overlays/.husky/pre-commit` — the source of truth (POSIX `sh`).
- The `ai-sync` repo's own `config.yaml` carries an overlay rule for the husky path with `executable: true` so the deployed file is mode `100755`.
- The same pattern works for `.claude/hooks/*.sh` and any other script you want runnable.

When in doubt, mirror that recipe.

## Further Reading

- `docs/CONFIGURATION.md` - Full configuration reference
- `docs/LOADERS.md` - External content loaders
- `docs/GENERATORS.md` - Target-specific output details
