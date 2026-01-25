---
name: start-implementation
description: Start implementation from task plan
---

Please start the implementation according to the implementation plan (.ai-flow/tasks/). Run tests & lint without sandbox mode if they fail.

Use interview style to clarify options with the user upfront when starting, if there any details to clarify!

## Change verification
!!! Do it - especially tests - only for changed files to make the runs more efficient.

pnpm format
pnpm lint
pnpm typecheck
pnpm test


Important context:
- .ai-flow/project.md & project-multi-platform.md - Project plan & Architecture
- .ai-flow/learnings.md - Past mistakes

Always update the implementation plan in tasks/*.md and mark the checkboxes when tasks are done!!

Always use `pnpm lint:fix && pnpm format:fix` to fix files before you try it yourself !!!

For excessive string replacements (for example renaming a specific term), use a command line replace command, if you think it will be much faster and token saving. Then check the changes afterwards, and fix eventual mistakes. Dont change everything with your normal tool in such cases. 

For file name changes: use `mv` command (dont deleting and recreate with the normal file edit tool).
