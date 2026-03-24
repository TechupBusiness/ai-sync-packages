# MCP Recovery Guide

Detailed troubleshooting for XcodeBuildMCP failures and ralph-tui autonomous recovery.

## Table of Contents

- [MCP Server Fails to Start](#mcp-server-fails-to-start)
- [Ralph-TUI Autonomous Recovery](#ralph-tui-autonomous-recovery)
- [Manual Intervention](#manual-intervention)

## MCP Server Fails to Start

**Symptoms:**
- MCP tools like `mcp__XcodeBuildMCP__list_sims` unavailable
- Claude shows "xcodebuildmcp · ✘ failed" in MCP server list
- npx commands fail with `ENOTEMPTY` errors

**Root Cause:** npm cache corruption from interrupted smithery CLI installations.

**Fix:**

```bash
# Remove corrupted cache and clean
rm -rf ~/.npm/_npx/*/node_modules/@smithery/.cli-*
npm cache clean --force

# Verify fix
npx -y @smithery/cli@latest --version

# Reinstall if needed (choose your client)
npx -y @smithery/cli@latest install cameroncooke/xcodebuildmcp --client claude-code
npx -y @smithery/cli@latest install cameroncooke/xcodebuildmcp --client cursor
```

**After fixing:** Restart AI client for MCP to reconnect.

## Ralph-TUI Autonomous Recovery

Ralph-TUI creates fresh Claude instances for each iteration. Use these strategies for overnight runs:

### Strategy 1: Exit Without Completion (Recommended)

If MCP fails mid-task and UI verification is required:

1. Attempt the npm cache fix (may work if corruption is the issue)
2. If MCP still unavailable, **exit without signaling `<promise>COMPLETE</promise>`**
3. Ralph-TUI will retry the task with a fresh Claude instance
4. The fresh instance loads MCP configuration anew

**When to use:** Task genuinely requires UI verification and can't proceed without it.

### Strategy 2: Skip UI Verification

If MCP fails but task can complete without UI verification:

1. Complete code implementation
2. Run CLI quality gates (`flutter analyze`, `flutter test`, `flutter build`)
3. Document: "UI verification skipped - MCP unavailable"
4. Signal `<promise>COMPLETE</promise>`

**When to use:** Task's core work is code changes; UI verification is nice-to-have.

### Strategy 3: Self-Repair Then Continue

Attempt to fix MCP within the current session:

```bash
# Fix npm cache corruption
rm -rf ~/.npm/_npx/*/node_modules/@smithery/.cli-*
npm cache clean --force
```

Then retry MCP tools. If they work, continue normally.

**When to use:** First attempt before deciding on Strategy 1 or 2.

## Manual Intervention

For human operators monitoring overnight runs:

```bash
# Stop ralph-tui
Ctrl+C

# Fix npm cache
rm -rf ~/.npm/_npx/*/node_modules/@smithery/.cli-*
npm cache clean --force

# Resume with fresh Claude instance
ralph-tui resume
```

## Pre-flight Check

Before starting long ralph-tui sessions:

```bash
# Verify MCP will work
ralph-tui run --verify

# Or test smithery directly
npx -y @smithery/cli@latest --version
```

## Config File Locations

| Client | Config Path |
|--------|-------------|
| Claude Code | `~/.claude.json` (mcpServers section) |
| Cursor | `~/.cursor/mcp.json` |
| Claude Desktop | `~/.claude/claude_desktop_config.json` |
