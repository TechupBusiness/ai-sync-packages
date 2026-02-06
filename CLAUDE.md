# Claude Code Context - ai-sync-packages

## nested-repos

> Nested git repositories that require separate commits

# Nested Git Repository

## Important: Separate Commit Required

The `ai-sync-packages/` directory is a **single git repository** independent of the main project.

## Commit Workflow

When modifying files in this directory:

1. **Commit to the nested repo:**
   ```bash
   cd ai-sync-packages
   git add .
   git commit -m "feat(agents): update agent descriptions"
   git push
   ```

2. **Then update the reference in the main repo** (if using submodule references)

## Why This Matters

- Changes in this repo are **not tracked** by the parent git repository
- `git status` in the root will NOT show changes in this folder
- You must `cd` into `ai-sync-packages/` to see/commit changes

## Checking for Changes

```bash
# Check nested repo status
cd ai-sync-packages && git status

# Or from root
git -C ai-sync-packages status
```
