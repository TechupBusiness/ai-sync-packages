---
name: ios-simulator-tests
description: Build, run, and test the iOS app on simulator. Uses XcodeBuildMCP when available, falls back to CLI commands when MCP is unavailable. Trigger for iOS app testing, UI verification, screenshot capture, and simulator interaction. For ralph-tui overnight runs, supports graceful degradation when MCP fails.
---

# iOS Simulator Testing

Build and run the iOS app on simulator, interact with UI, and verify behavior.

## Quick Start

1. Try `mcp__XcodeBuildMCP__list_sims`
2. If works → use MCP workflow below
3. If unavailable → read `.ai-sync/skills/ios-simulator-tests/FALLBACK_CLI.md` for CLI commands

## Prerequisites

- macOS 14.5+, Xcode 16.x+, Node.js 18.x+
- AXe: `brew install cameroncooke/axe/axe`
- Booted iOS Simulator

**Install XcodeBuildMCP (optional):**
```bash
npx -y @smithery/cli@latest install cameroncooke/xcodebuildmcp --client claude-code
```

## Core Workflow

### 1) Discover simulator
Call `mcp__XcodeBuildMCP__list_sims`, select simulator with state `Booted`.

### 2) Set session defaults
Call `mcp__XcodeBuildMCP__session-set-defaults` with `projectPath`/`workspacePath`, `scheme`, `simulatorId`.

### 3) Build + run
Call `mcp__XcodeBuildMCP__build_run_sim`. For launch only: `mcp__XcodeBuildMCP__launch_app_sim`.

## UI Interaction

**CRITICAL: Always call `describe_ui` before any tap/swipe/type action.**

```
# Correct:
1. describe_ui → get element IDs/labels
2. tap(id: "element-id") or tap(label: "Button Text")

# Wrong (will warn):
1. screenshot → guess coordinates
2. tap(x: 100, y: 200)
```

### Tools

| Tool | Purpose |
|------|---------|
| `describe_ui` | Get accessibility hierarchy - **ALWAYS FIRST** |
| `tap` | Tap element (prefer `id` or `label` over coordinates) |
| `type_text` | Enter text after focusing field |
| `swipe` | Navigate (duration in SECONDS: 0.1-0.2 for fast swipe) |
| `screenshot` | Visual capture only, not for coordinates |

### Swipe Examples
```
swipe(x1: 350, y1: 500, x2: 90, y2: 500, duration: 0.15)  # Left
swipe(x1: 90, y1: 500, x2: 350, y2: 500, duration: 0.15)  # Right
swipe(x1: 220, y1: 600, x2: 220, y2: 300, duration: 0.2)  # Down
```

## Flutter Apps

If `describe_ui` returns few elements, the app needs `Semantics` widgets:
```dart
Semantics(label: 'Start session', child: Text('Start'))
```

## Logs

- Start: `mcp__XcodeBuildMCP__start_sim_log_cap` with bundle id
- Stop: `mcp__XcodeBuildMCP__stop_sim_log_cap`

## Troubleshooting

| Problem | Solution |
|---------|----------|
| MCP tools not found | Use `.ai-sync/skills/ios-simulator-tests/FALLBACK_CLI.md` or install XcodeBuildMCP |
| "describe_ui not called" | Call `describe_ui` before tapping |
| AXe command failed | `brew install cameroncooke/axe/axe` |
| Few elements in describe_ui | Flutter app needs Semantics widgets |

### MCP Server Fails to Start

**Quick fix:**
```bash
rm -rf ~/.npm/_npx/*/node_modules/@smithery/.cli-*
npm cache clean --force
```
Then restart AI client. See `.ai-sync/skills/ios-simulator-tests/references/mcp-recovery.md` for details.

### Ralph-TUI: MCP Failure During Overnight Runs

When MCP fails during autonomous operation:

1. **Try self-repair first:**
   ```bash
   rm -rf ~/.npm/_npx/*/node_modules/@smithery/.cli-*
   npm cache clean --force
   ```

2. **If MCP still unavailable, choose:**
   - **Task requires UI verification:** Exit WITHOUT `<promise>COMPLETE</promise>`. Ralph-TUI retries with fresh Claude instance.
   - **Task can skip UI verification:** Complete code, run CLI quality gates, signal `<promise>COMPLETE</promise>`.

See `.ai-sync/skills/ios-simulator-tests/references/mcp-recovery.md` for detailed recovery strategies.
