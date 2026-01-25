---
name: rebase-worktree
description: Rebase worktree to base branch
---

## Prerequisites

Verify you are operating in a git worktree by checking the current project path. If not in a worktree, inform the user and stop.

## Rebase Process

1. **Check status**: Run `git status` to verify all changes are committed
   - If uncommitted changes exist, report what they are and wait for user decision

2. **Read session config**: Get the base branch from `.ai-sync-session` file (default: "main")

3. **Perform rebase**: Run `git rebase <base-branch>` (no fetch needed)
   - If rebase fails due to unclean status, analyze the situation and report
   - If merge conflicts occur, resolve them one by one
   - **Task-ID conflicts**: If conflicts occur in `.ai-flow/` specification files due to Task-IDs already used in the base branch, rename our Task-IDs to the next available ID. Never lose specification content—preserve all our specs with updated IDs.

4. **Inform user**: After successful rebase, tell the user that when they exit, they can transfer changes back to the base branch

## Important

Do NOT transfer changes to main directly. The external scripts handle this safely when the session ends.
