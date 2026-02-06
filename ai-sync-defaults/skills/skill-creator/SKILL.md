---
name: skill-creator
description: Guide for creating effective skills. This skill should be used when users want to create a new skill (or update an existing skill) that extends Claude's capabilities with specialized knowledge, workflows, or tool integrations.
---

# Skill Creator

This skill provides guidance for creating effective skills.

## About Skills

Skills are modular, self-contained packages that extend Claude's capabilities by providing
specialized knowledge, workflows, and tools. They transform Claude from a general-purpose agent into a specialized agent equipped with procedural knowledge that no model can fully possess.

### What Skills Provide

1. Specialized workflows - Multi-step procedures for specific domains
2. Tool integrations - Instructions for working with specific file formats or APIs
3. Domain expertise - Company-specific knowledge, schemas, business logic
4. Bundled resources - Scripts, references, and assets for complex and repetitive tasks

## Core Principles

### Concise is Key

The context window is a public good. Skills share it with system prompt, conversation history, other Skills' metadata, and the user request.

**Default assumption: Claude is already very smart.** Only add context Claude doesn't already have. Challenge each piece of information: "Does Claude really need this?" and "Does this justify its token cost?"

Prefer concise examples over verbose explanations.

### Set Appropriate Degrees of Freedom

Match specificity to the task's fragility and variability:

- **High freedom (text instructions)**: Multiple approaches valid, decisions depend on context
- **Medium freedom (pseudocode/scripts with parameters)**: Preferred pattern exists, some variation acceptable
- **Low freedom (specific scripts, few parameters)**: Operations fragile, consistency critical, specific sequence required

### Anatomy of a Skill

Every skill consists of a required SKILL.md file and optional bundled resources:

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name + description, required)
│   └── Markdown instructions (required)
└── Bundled Resources (optional)
    ├── scripts/          - Executable code (Python/Bash/etc.)
    ├── references/       - Documentation loaded into context as needed
    └── assets/           - Files used in output (templates, icons, fonts, etc.)
```

#### SKILL.md (required)

- **Frontmatter** (YAML): `name` and `description` fields. These determine when the skill gets used, so be clear and comprehensive.
- **Body** (Markdown): Instructions and guidance. Only loaded AFTER the skill triggers.

#### Bundled Resources (optional)

**scripts/**: Executable code for tasks that require deterministic reliability or are repeatedly rewritten.

**references/**: Documentation loaded as needed into context. Keeps SKILL.md lean. For files >10k words, include grep search patterns in SKILL.md. Avoid duplicating info between SKILL.md and references.

**assets/**: Files used in output (templates, images, fonts, boilerplate). Not loaded into context.

#### What NOT to Include

Do NOT create extraneous files like README.md, CHANGELOG.md, INSTALLATION_GUIDE.md, etc. Only include files that directly support the skill's functionality.

### Progressive Disclosure

Skills use a three-level loading system:

1. **Metadata (name + description)** - Always in context (~100 words)
2. **SKILL.md body** - When skill triggers (<5k words)
3. **Bundled resources** - As needed (unlimited)

Keep SKILL.md under 500 lines. Split content into references/ when approaching this limit. Reference files from SKILL.md with clear descriptions of when to read them.

**Key principle:** When a skill supports multiple variations, keep only the core workflow in SKILL.md. Move variant-specific details into separate reference files.

## Skill Creation Process

1. Understand the skill with concrete examples
2. Plan reusable skill contents (scripts, references, assets)
3. Initialize the skill (run init_skill.py)
4. Edit the skill (implement resources and write SKILL.md)
5. Iterate based on real usage

### Step 1: Understanding with Concrete Examples

Skip only when usage patterns are already clearly understood.

Ask targeted questions:
- "What functionality should the skill support?"
- "Can you give examples of how this skill would be used?"
- "What would a user say that should trigger this skill?"

Avoid asking too many questions at once. Conclude when there's a clear sense of required functionality.

### Step 2: Planning Reusable Contents

For each concrete example, analyze:
1. How to execute it from scratch
2. What scripts, references, and assets would help when executing repeatedly

Examples:
- Rotating PDFs repeatedly → `scripts/rotate_pdf.py`
- Building webapps with same boilerplate → `assets/hello-world/`
- Querying BigQuery repeatedly → `references/schema.md`

### Step 3: Initializing the Skill

Skip if the skill already exists and only needs iteration.

Run the init script to generate a template:

```bash
scripts/init_skill.py <skill-name> --path <output-directory>
```

The script creates a SKILL.md template with TODO placeholders and example resource directories. Customize or remove generated files as needed.

### Step 4: Edit the Skill

Remember: the skill is for another Claude instance to use. Include non-obvious procedural knowledge and domain-specific details.

#### Design Patterns

- **Multi-step processes**: See references/workflows.md
- **Specific output formats**: See references/output-patterns.md

#### Start with Reusable Contents

Implement the resources identified in Step 2. This may require user input (e.g., brand assets, documentation). Test added scripts by running them. Delete any unneeded example files.

#### Write SKILL.md

**Frontmatter:**
- `name`: The skill name
- `description`: Primary triggering mechanism. Include what the skill does AND when to use it. All "when to use" info belongs here, not in the body.

**Body:** Instructions for using the skill and its bundled resources. Use imperative/infinitive form.

### Step 5: Iterate

1. Use the skill on real tasks
2. Notice struggles or inefficiencies
3. Update SKILL.md or bundled resources
4. Test again
