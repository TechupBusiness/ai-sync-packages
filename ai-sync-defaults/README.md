# ai-sync-defaults

Default agents, commands, rules, and skills for [ai-sync](https://github.com/TechupBusiness/ai-sync).

## Installation

```bash
ai-sync plugins add github:TechupBusiness/ai-sync-defaults
```

Or add to your `.ai-sync/config.yaml`:

```yaml
use:
  plugins:
    - name: ai-sync-defaults
      source: github:TechupBusiness/ai-sync-defaults
      enabled: true
```

## Contents

### Agents (11)

Specialized AI agent personas for different development tasks:

| Agent | Description |
|-------|-------------|
| `architect` | System design and architecture decisions |
| `implementer` | Code implementation and feature development |
| `coordinator` | Multi-agent task coordination |
| `security-hacker` | Security analysis and vulnerability assessment |
| `performance-optimizer` | Performance profiling and optimization |
| `data-specialist` | Data modeling and database design |
| `devops-specialist` | CI/CD, infrastructure, and deployment |
| `growth-hacker` | Product metrics and growth strategies |
| `test-zealot` | Testing strategies and quality assurance |
| `ux-psychologist` | User experience and interface design |
| `hyper-critic` | Critical code review and quality analysis |

### Commands (5)

Slash commands for common development tasks:

| Command | Description |
|---------|-------------|
| `/format` | Format code using project conventions |
| `/lint-fix` | Run linter and auto-fix issues |
| `/type-check` | Run TypeScript type checking |
| `/simplify` | Simplify and refactor complex code |
| `/reflect` | Reflect on implementation decisions |

### Rules (3)

Guidelines that apply based on file patterns:

| Rule | Description |
|------|-------------|
| `code-review` | Code review standards and checklist |
| `documentation` | Documentation conventions |
| `authoring-guide` | Guide for writing ai-sync content |

### Skills (1)

On-demand capabilities:

| Skill | Description |
|-------|-------------|
| `create-project` | Interactive project scaffolding |

## Selective Loading

Load only what you need:

```yaml
use:
  plugins:
    - name: ai-sync-defaults
      source: github:TechupBusiness/ai-sync-defaults
      include: [agents]  # Only load agents
      # or
      exclude: [commands]  # Load everything except commands
```

## License

MIT
