---
name: transfer-worktree
description: Transfer changes from worktree to main workspace
---

# transfer-worktree

Transfer changes from a git worktree back to the main workspace.

---

## Context

When running in a git worktree, you have two relevant paths:
- **Worktree path**: Where you currently are (e.g., `~/.ai-sync/worktrees/{project}/{worktree-id}`)
- **Workspace path**: The original/main workspace (e.g., `~/Sources/{project}`)

Use `git worktree list` to see both paths if needed.

---

## Phase 0: Preflight Checks

### 0.1 Verify worktree context

Check if we are in a worktree by reading the session file:

```bash
cat .ai-sync-session 2>/dev/null
```

If the file doesn't exist, abort with:

```
ERROR: Not in a worktree session

This command must be run from within a git worktree that was created by ai-sync.
The .ai-sync-session file is missing, which indicates this is not a valid worktree.

To create a new worktree session:
  ai-sync run <tool>

To list existing worktrees:
  git worktree list
```

Parse the session file to extract:
- `SESSIONID` - The session identifier
- `TOOL` - The AI tool being used (e.g., claude, cursor, factory)
- `WORKTREEPATH` - Full path to this worktree
- `BRANCHNAME` - Current branch name
- `BASEBRANCH` - Target branch for transfer (e.g., main)

### 0.2 Display session resume information

**⏸️ DISPLAY prominently:**

```
================================================================================
SESSION: {SESSIONID}
TOOL:    {TOOL}
================================================================================

If this session is interrupted, resume with:
  cd {WORKTREEPATH}
  {resume-command}

================================================================================
```

Build the resume command based on the tool:
- **Claude**: `claude --resume {SESSIONID}`
- **Factory**: `factory --continue {SESSIONID}`
- **Cursor**: `Open Cursor IDE in the worktree directory and continue in the active chat session`
- **Codex**: `Run codex in the worktree directory`
- **Aider**: `Run aider in the worktree directory`
- **Other**: `cd {WORKTREEPATH} && {tool}`

### 0.3 Check target branch state

Check if the target workspace has uncommitted changes:

```bash
# Get workspace path from git worktree list
git worktree list | grep -v "$(pwd)" | head -1 | awk '{print $1}'
```

Then check target workspace status WITHOUT changing directories:

```bash
git -C {workspace-path} status --porcelain
```

**If target is dirty (non-empty output):**

```
⚠️  TARGET BRANCH HAS UNCOMMITTED CHANGES
===================================

The main workspace has uncommitted changes:
{list files}

This could cause conflicts or data loss during transfer.

**Options:**
1. **Commit** - Commit changes in target first (recommended)
2. **Stash** - Stash changes in target (automatic)
3. **Proceed** - Continue anyway (higher conflict risk)
4. **Abort** - Cancel and review manually

**Go back to main menu?** [Y/N]
```

Wait for user decision before proceeding.

If user chooses **Stash**, run:
```bash
git -C {workspace-path} stash push -m "ai-sync: auto-stash before transfer"
```

---

## Phase 1: Discovery & Summary

### 1.1 Gather information

Run these commands to understand the current state:

```bash
git worktree list                    # shows worktree and main workspace paths
git status -sb                       # current worktree status
git log --oneline {BASEBRANCH}..HEAD # commits ahead of base branch
```

### 1.2 Present summary and wait for confirmation

**⏸️ STOP - Present this summary to the user:**

```
## Transfer Summary

**From:** {worktree-path}
**To:**   {workspace-path}
**Base:** {BASEBRANCH}

### Worktree state:
- Commits ahead of {BASEBRANCH}: {count}
- Uncommitted changes: {yes/no - list files if yes}
- Clean: {yes/no}

### Recommended approach:
- {If has commits}: **Rebase** - ensures linear history (mandatory)
- {If only uncommitted changes}: **Patch** - quick transfer of WIP
- {If nothing to transfer}: Nothing to do

**Options:**
1. **Rebase** - Move worktree commits onto {BASEBRANCH} (linear history)
2. **Patch** - Transfer uncommitted changes only (for WIP)
3. **Discard** - Delete worktree without transferring
4. **Abort** - Cancel

Which option?

**Go back?** [Y/N]
```

**Wait for user to choose before proceeding.**

---

## Phase 2: Transfer Methods

### 2.1 Branch Selection

Before proceeding with transfer, confirm the target branch:

```bash
# List local branches
git -C {workspace-path} branch --list
```

**⏸️ STOP - Confirm target branch:**

```
## Target Branch Selection

Current target: {BASEBRANCH}

Available branches:
{list branches}

Use {BASEBRANCH} as target? [Y/N]
If N, which branch should receive the changes?

**Go back?** [Y/N]
```

### Option 1: Rebase (Mandatory for Linear History)

Use this method to move your worktree commits to the tip of the target branch.
**Do NOT create merge commits with duplicate messages.**

If a merge commit is absolutely necessary (e.g. for long-lived feature branches), ensure the merge commit message clearly describes the merge itself (e.g. "Merge branch 'feat/new-parser'"). **Never copy the message of the feature commit.**

#### Step 1: Commit uncommitted changes (if any)

```bash
git add -A
git status -sb
```

**⏸️ STOP - If there are staged changes, ask user for commit message before committing.**

```bash
git commit -m "{user's message}"
```

#### Step 2: Update Target Reference

Ensure your target base is up to date (locally).

```bash
git fetch origin {BASEBRANCH}
```

#### Step 3: Rebase Worktree on Target

Replay your worktree commits on top of the latest target branch.

```bash
git rebase origin/{BASEBRANCH}
```

**⚠️ CRITICAL: Conflict Handling**

If `git rebase` exits with conflicts:

1. **Detect conflict:**
   ```bash
   git status | grep "both modified"
   ```

2. **Abort the rebase (do NOT attempt resolution here):**
   ```bash
   git rebase --abort
   ```

3. **Display AI tool resume instructions:**
   ```
   ================================================================================
   REBASE HAS CONFLICTS
   ================================================================================

   The rebase encountered conflicts that require resolution.
   The rebase has been safely aborted (no data lost).

   NEXT STEPS
   ----------
   1. Return to the worktree session to resolve conflicts:

      cd {WORKTREEPATH}
      {resume-command}

   2. In the resumed session, run: /rebase-worktree
   3. Resolve conflicts with AI assistance
   4. Then run: /transfer-worktree again

   CONFLICT FILES
   --------------
   {list conflicted files from git status}

   SESSION INFO
   ------------
   Session ID: {SESSIONID}
   Tool: {TOOL}

   ================================================================================

   **Go back to main menu?** [Y/N]
   ```

4. **Wait for user decision** - Do NOT proceed with transfer.

#### Step 4: Fast-forward Target Workspace

Now that your history is linear, update the target workspace (without changing directories):

```bash
git -C {workspace-path} merge {worktree-branch} --ff-only
```
*Note: `--ff-only` ensures no merge commit is created. If this fails, you didn't rebase correctly.*

**⏸️ STOP - Confirm success:**

```
## Rebase Complete

Changes have been transferred to {workspace-path}.
The target branch has been fast-forwarded.

**Go back?** [Y/N]
```

#### 🛑 STOP! NO PUSHING.

**Do NOT run `git push`.**
Your job is to update the local workspace only. The user will handle the push.

---

### Option 2: Patch (for uncommitted WIP)

Use when transferring work-in-progress without creating commits in the worktree.

#### Step 1: Create patch

```bash
git add -A
git diff --cached --binary -- . ':!node_modules' > /tmp/{worktree-id}-transfer.patch
```

#### Step 2: Preview in target

```bash
git -C {workspace-path} apply --stat /tmp/{worktree-id}-transfer.patch
```

**⏸️ STOP - Show preview, ask user to confirm apply.**

```
## Patch Preview

The following changes will be applied:
{patch stats}

Apply patch? [Y/N]

**Go back?** [Y/N]
```

#### Step 3: Apply patch

```bash
git -C {workspace-path} apply --3way /tmp/{worktree-id}-transfer.patch
```

If conflicts occur, display:

```
⚠️  PATCH HAS CONFLICTS
======================

The patch could not be applied cleanly.

NEXT STEPS
----------
1. Return to the worktree and commit your changes first
2. Then use the Rebase option for a cleaner transfer

**Go back to main menu?** [Y/N]
```

#### Step 4: Cleanup

```bash
rm /tmp/{worktree-id}-transfer.patch
```

---

## Phase 3: Verification

After transfer, run verification in the main workspace:

```bash
git -C {workspace-path} status -sb
```

If the project uses npm/pnpm:
```bash
cd {workspace-path} && pnpm typecheck
cd {workspace-path} && pnpm lint
cd {workspace-path} && pnpm test
```

**⏸️ STOP - Report results:**

```
## Verification Results

**Git Status:** {clean/dirty}
**Type Check:** {pass/fail}
**Lint:** {pass/fail}
**Tests:** {pass/fail}

{If any failures, list details}

**Go back?** [Y/N]
```

Report results to user.

---

## Phase 4: Cleanup

**⏸️ STOP - Ask user:**

```
## Transfer Complete

Cleanup options:

1. **Delete worktree** - Remove entirely (recommended after rebase)
2. **Keep worktree** - Leave for further work
3. **Force delete** - Remove even if dirty (discards remaining changes)

Which option?

**Go back?** [Y/N]
```

### Delete commands:

```bash
# Clean worktree (after rebase):
git worktree remove {worktree-path}

# Dirty worktree or don't care about remaining changes:
git worktree remove --force {worktree-path}

# Also delete the branch (optional):
git branch -d {worktree-branch}   # safe - only if merged
git branch -D {worktree-branch}   # force delete
```

After cleanup, display final summary:

```
================================================================================
TRANSFER COMPLETE
================================================================================

Changes have been transferred from:
  {worktree-path}

To:
  {workspace-path}

Worktree: {deleted/kept}

To continue working in the main workspace:
  cd {workspace-path}

================================================================================
```

---

## Quick Reference

| State | Meaning | Can delete without --force? |
|-------|---------|----------------------------|
| **Clean** | All changes committed | Yes |
| **Dirty** | Has uncommitted changes | No (use --force) |

| Action | Command |
|--------|---------|
| List worktrees | `git worktree list` |
| Remove worktree | `git worktree remove {path}` |
| Force remove | `git worktree remove --force {path}` |
| Prune stale | `git worktree prune` |

---

## Edge Cases

### Session file missing
If `.ai-sync-session` doesn't exist, the command aborts with instructions to create a worktree first.

### Unknown AI tool in session
If the TOOL field contains an unknown tool, fall back to generic instructions:
```
cd {worktree-path} && {tool}
```

### Git lock file exists
If git reports "another git operation in progress":
```
ERROR: Git operation in progress

Another git operation is running. Wait for it to complete or remove:
  rm {workspace-path}/.git/index.lock

**Go back to main menu?** [Y/N]
```

### Target branch deleted
If the target branch no longer exists:
```
ERROR: Target branch missing

The target branch '{BASEBRANCH}' no longer exists.

**Options:**
1. Select a different target branch
2. Abort

**Go back?** [Y/N]
```

### Network timeout during fetch
If fetch fails due to network issues:
```
⚠️  FETCH FAILED
===============

Could not fetch from origin. This might be a network issue.

**Options:**
1. **Retry** - Try fetch again
2. **Skip** - Continue without updating (may cause conflicts)
3. **Abort** - Cancel transfer

**Go back?** [Y/N]
```

### Worktree and main diverged significantly
If the worktree and main have diverged by many commits:
```
⚠️  BRANCHES HAVE DIVERGED
==========================

Your worktree branch and {BASEBRANCH} have diverged:
- Worktree ahead by: {X} commits
- {BASEBRANCH} ahead by: {Y} commits

This is normal but may result in more conflicts during rebase.

**Continue with rebase?** [Y/N]

**Go back?** [Y/N]
```

---

## Notes

- **Rebase is preferred** over merge to maintain a linear history.
- **Patch is for WIP** - when you don't want commits in the worktree.
- **`--force` is safe** if you've already transferred changes or don't need them.
- `':!node_modules'` in the diff command excludes node_modules.
- `--3way` enables conflict markers instead of silent failure.
- Always use `git -C {path}` instead of `cd` to avoid confusion about current directory.
- Session info comes from `.ai-sync-session` file in worktree root.
- Resume instructions are built from platform definitions (see `src/platforms/*/definition.ts`).
