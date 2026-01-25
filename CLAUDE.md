# ai-sync-packages

Curated collection of ai-sync content packages for use across projects.

## Packages

| Package | Description |
|---------|-------------|
| `ai-sync-defaults/` | Core agents, rules, skills, and commands for general use |
| `ai-sync-advanced/` | Advanced commands, workflows, and specialized skills |
| `ai-sync-aiflow-tasks/` | Task planning and project management commands |
| `ai-sync-flutter/` | Flutter/Dart development content |
| `ai-sync-nodejs/` | Node.js development content |

## Usage

### Remote (from GitHub)

Load packages via git loader with subpaths:

```yaml
# .ai-sync/config.yaml
loaders:
  # Load entire defaults package
  - type: git
    source: github:TechupBusiness/ai-sync-packages/ai-sync-defaults
    
  # Load from advanced with full filtering options
  - type: git
    source: github:TechupBusiness/ai-sync-packages/ai-sync-advanced
    use:
      skills: [research]           # Only specific skills by name
      commands: [commit, self-review]  # Only specific commands
      rules: '*'                   # All rules
      agents: []                   # No agents from this loader
      workflows: '*'               # All workflows
```

### Local Development

For local development or testing changes before pushing:

```yaml
# .ai-sync/config.yaml
loaders:
  # Point to local clone of this repo
  - type: local
    source: ../ai-sync-packages/ai-sync-defaults
    
  # Load multiple packages with selective filtering
  - type: local
    source: ../ai-sync-packages/ai-sync-advanced
    use:
      skills: '*'                  # All skills
      commands: [commit]           # Just the commit command
      rules: []                    # No rules
      
  # Temporarily disable a package
  - type: local
    source: ../ai-sync-packages/ai-sync-flutter
    enabled: false
```

### Filter Syntax Reference

| Config | Meaning |
|--------|---------|
| `'*'` | Include all items of that type |
| `['name1', 'name2']` | Include only items with these exact names |
| `[]` | Exclude all items of that type |
| *(omitted)* | Defaults to include all (same as `'*'`) |

## Structure

Each package follows the standard ai-sync directory structure:

```
package-name/
├── agents/      # Agent definitions
├── commands/    # Slash commands
├── rules/       # Always-on or glob-matched rules
├── skills/      # On-demand capabilities
└── workflows/   # Multi-step workflows
```

## License

MIT
