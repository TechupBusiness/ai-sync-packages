---
name: ai-sync-maintenance
description: Keep ai-sync configuration synchronized with codebase changes
always_apply: true
---

# ai-sync Maintenance

When making changes to the codebase, consider if ai-sync files need updates.

## Update Triggers

When making code changes, check if corresponding ai-sync files need updates:

| Code Change Type | ai-sync Area to Check |
|------------------|----------------------|
| Database/data layer changes | Rules about data patterns, models |
| UI component patterns | Rules about UI conventions, styling |
| New branding/theming | Rules about design system, branding |
| Directory structure changes | Project overview rules, path references |
| Build/deploy changes | Commands, workflow definitions |
| New workflows or processes | Skills, commands |

**Tip:** Run `ls .ai-sync/` to see your project's content directories.

<!-- TODO: Once T411 is implemented, replace with:
{bash: ai-sync list --format=table}
-->

## Self-Check Prompts

After significant changes, ask:
- Does the project overview still accurately describe the architecture?
- Are coding conventions documented that I'm now using?
- Have build commands or workflows changed?

## Integration Guidelines

- Don't create separate rules for one-off patterns
- Integrate new learnings into existing relevant rules
- Keep rules focused - split if they exceed ~100 lines
- Update globs if file locations change

## Validation

After updating ai-sync files:
```bash
ai-sync lint .ai-sync/rules/changed-file.md
ai-sync validate
ai-sync build
```
