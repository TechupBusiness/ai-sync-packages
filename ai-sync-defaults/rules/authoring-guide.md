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
| `always_apply` | boolean | No | `false` | Always load regardless of context |
| `globs` | string[] | No | - | File patterns to trigger this rule |
| `targets` | string[] | No | `[cursor, claude, factory]` | Platforms to generate for |
| `requires` | string[] | No | - | Dependent rules to load together |
| `category` | string | No | - | Organization category |
| `priority` | string | No | `medium` | Priority level (`low`, `medium`, `high`) |

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
---
name: database
description: Database schema and query patterns
version: 1.0.0
always_apply: false
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
always_apply: false
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
| `alwaysApply` | boolean | Override always_apply (camelCase) |
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

### File Collisions

Files with matching names will collide during sync:
- Project files in `.ai-sync/` override defaults
- Use `overrides/` folder to customize default content

### Avoiding Conflicts

1. Check existing files: `ai-sync status`
2. Use unique names for project-specific content
3. Override defaults explicitly in `overrides/` folder

### Force Overwrite

When intentionally replacing files:

```bash
ai-sync --force
```

Or in configuration:

```yaml
output:
  overwrite: true
```

## Common Pitfalls

### Frontmatter Mistakes

| Mistake | Correct |
|---------|---------|
| `alwaysApply: true` | `always_apply: true` (snake_case in generic) |
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

Set any supported frontmatter field under `overrides.<type>.<name>`: `description`, `targets`, `priority`, `category`, `always_apply`, `globs`, etc. For rules and skills, several fields (`description`, `targets`, `globs`, `priority`, `category`, `compatibility`) also accept `{ file: ... }` / `{ bash: ... }`.

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

        # Section-level modifications (.md only) — reuses the same modification syntax
        "references/guide.md":
          modifications:
            - position: replace
              section: "## Overview"
              content: { file: "patches/overview.md" }
            - position: end
              section: "## New Section"
              create_if_missing: true
              content: "..."

        # Add a brand-new file not in the upstream bundle
        "scripts/post-process.sh":
          content: { file: "patches/post-process.sh" }
```

Rules:
- Each path uses **exactly one** of `delete` / `content` / `append` / `modifications`.
- `append` and `modifications` are `.md`-only and require the path to exist in the source bundle.
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
- **MCP servers**: no per-server override in loader config. Merge happens via `mcp/servers.yaml`; last loader wins.
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

## Further Reading

- `docs/CONFIGURATION.md` - Full configuration reference
- `docs/LOADERS.md` - External content loaders
- `docs/GENERATORS.md` - Target-specific output details
