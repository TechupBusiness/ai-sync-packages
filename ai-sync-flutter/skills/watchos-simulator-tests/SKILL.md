---
name: watchos-simulator-tests
description: Build, run, and test the watchOS app on simulator. Uses AXe CLI for UI automation since XcodeBuildMCP doesn't support watchOS directly. Trigger for watchOS app testing, UI verification, screenshot capture, and simulator interaction.
---

# watchOS Simulator Testing

Build and run the watchOS app on simulator, interact with UI, and verify behavior.

## Quick Start

1. List booted simulators: `xcrun simctl list devices | grep -i watch | grep Booted`
2. Get watch UDID from output
3. Use AXe CLI commands for all interactions

## Prerequisites

- macOS 14.5+, Xcode 16.x+
- AXe: `brew install cameroncooke/axe/axe`
- Paired iPhone + Watch simulators booted

**Boot paired simulators:**
```bash
# Boot iPhone first (watch pairs with it)
xcrun simctl boot "iPhone 16 Pro"

# Boot watch simulator
xcrun simctl boot "Apple Watch Series 9 (45mm)"
```

## Core Workflow

### 1) Find Watch Simulator

```bash
# List all booted simulators
xcrun simctl list devices | grep Booted

# Or use AXe to list
axe list-simulators | grep -i watch | grep Booted
```

Save the watch UDID (e.g., `A284E44B-168C-47B9-8241-29A2E231D2A7`).

### 2) Build watchOS App

```bash
# Flutter with watchOS extension
cd ios && xcodebuild -workspace Runner.xcworkspace \
  -scheme "Watch App" \
  -configuration Debug \
  -destination 'platform=watchOS Simulator,name=Apple Watch Series 9 (45mm)' \
  build

# Or use flutter build with watch target if configured
flutter build ios --debug --no-codesign --simulator
```

### 3) Install and Launch

```bash
WATCH_ID="<your-watch-udid>"

# Install app on watch simulator
xcrun simctl install $WATCH_ID /path/to/WatchApp.app

# Launch by bundle ID
xcrun simctl launch $WATCH_ID com.techup.roundsTracker.watchkitapp
```

## UI Interaction

**All interactions use AXe CLI with `--udid` parameter.**

**CRITICAL: Always use accessibility labels/IDs, not coordinates.** Watch screens vary in size (38mm to 49mm Ultra), so coordinates are unreliable.

### Get UI Hierarchy (Always First!)

```bash
axe describe-ui --udid $WATCH_ID
```

This returns JSON with `AXLabel` and `AXUniqueId` for each element - use these for tapping.

### Screenshot

```bash
xcrun simctl io $WATCH_ID screenshot /tmp/watch_screenshot.png
```

### Tap (Prefer Labels Over Coordinates)

```bash
# PREFERRED: By accessibility label (works on any watch size)
axe tap --udid $WATCH_ID --label "Start"

# PREFERRED: By accessibility ID (works on any watch size)
axe tap --udid $WATCH_ID --id "start-button"

# FALLBACK ONLY: By coordinates (use describe-ui to get AXFrame)
# Only when no label/id available - check AXFrame in describe-ui output
axe tap --udid $WATCH_ID -x <x> -y <y>
```

### Swipe (Use Relative Positioning)

For swipes, get the screen dimensions from `describe-ui` (look at the root Application's `AXFrame`), then calculate relative positions:

```bash
# Get screen size first
axe describe-ui --udid $WATCH_ID | head -20
# Look for: "AXFrame" : "{{0, 0}, {WIDTH, HEIGHT}}"

# Then use percentages of screen size for swipes
# Example for 198x242 screen - adjust based on actual size:
# Swipe up: from 80% height to 20% height, centered horizontally
# Swipe down: from 20% height to 80% height, centered horizontally
```

### Hardware Buttons (Size-Independent!)

These work on ALL watch sizes - prefer buttons over swipe gestures when possible:

```bash
# Digital Crown (home) - ALWAYS WORKS
axe button home --udid $WATCH_ID

# Side button - ALWAYS WORKS
axe button side-button --udid $WATCH_ID

# Siri - ALWAYS WORKS
axe button siri --udid $WATCH_ID

# Lock - ALWAYS WORKS
axe button lock --udid $WATCH_ID
```

### Type Text

```bash
axe type --udid $WATCH_ID --text "Hello"
```

## Size-Independent Navigation

Use hardware buttons instead of swipe coordinates whenever possible:

```bash
WATCH_ID="<udid>"

# Go to app grid (from watch face) - WORKS ON ALL SIZES
axe button home --udid $WATCH_ID
sleep 0.5
axe button home --udid $WATCH_ID

# Return to watch face - WORKS ON ALL SIZES
axe button home --udid $WATCH_ID

# App switcher - WORKS ON ALL SIZES
axe button side-button --udid $WATCH_ID
```

## Dynamic Coordinate Calculation

When you MUST use coordinates (no accessibility labels available), calculate them dynamically:

```bash
# 1. Get screen dimensions from describe-ui
SCREEN_INFO=$(axe describe-ui --udid $WATCH_ID | head -1)
# Parse width/height from AXFrame

# 2. Calculate center
# CENTER_X = WIDTH / 2
# CENTER_Y = HEIGHT / 2

# 3. For swipes, use percentages:
# TOP = HEIGHT * 0.1
# BOTTOM = HEIGHT * 0.9
# LEFT = WIDTH * 0.1
# RIGHT = WIDTH * 0.9
```

## Complete Workflow Example

```bash
# 1. Get watch simulator UDID
WATCH_ID=$(xcrun simctl list devices | grep "Apple Watch" | grep Booted | grep -oE "[A-F0-9-]{36}")
echo "Watch UDID: $WATCH_ID"

# 2. Take initial screenshot
xcrun simctl io $WATCH_ID screenshot /tmp/watch_initial.png

# 3. Navigate to app grid (size-independent)
axe button home --udid $WATCH_ID
sleep 0.5
axe button home --udid $WATCH_ID
sleep 1

# 4. Capture app grid
xcrun simctl io $WATCH_ID screenshot /tmp/watch_apps.png

# 5. Get UI hierarchy to find app by label
axe describe-ui --udid $WATCH_ID

# 6. Tap on app by label (from describe-ui output)
axe tap --udid $WATCH_ID --label "Rounds Tracker"
sleep 2

# 7. Capture app screen
xcrun simctl io $WATCH_ID screenshot /tmp/watch_app.png
```

## Logs

```bash
# Stream watch app logs
xcrun simctl spawn $WATCH_ID log stream --predicate 'subsystem == "com.techup.roundsTracker.watchkitapp"'

# View recent logs
xcrun simctl spawn $WATCH_ID log show --last 5m
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Watch not booted | `xcrun simctl boot "Apple Watch Series 9 (45mm)"` |
| AXe not found | `brew install cameroncooke/axe/axe` |
| Watch not pairing | Boot iPhone first, then watch |
| describe-ui empty children | App may not have accessibility labels |
| No label for element | Check `AXFrame` in describe-ui and calculate tap point |
| Tap not registering | Ensure coordinates are within screen bounds |

## MCP Limitation

XcodeBuildMCP does not currently support watchOS simulators directly. All watchOS interactions must use:
- `xcrun simctl` for screenshots, install, launch
- `axe` CLI with `--udid` for UI automation

This is equivalent functionality to what MCP provides for iOS, just via direct CLI commands.

## Flutter watchOS Apps

If `describe-ui` returns few elements, add `Semantics` widgets in your watchOS Flutter code:

```dart
Semantics(
  label: 'Start breathing session',
  button: true,
  child: YourButton(),
)
```

## Watch Screen Sizes Reference

Only use this for calculating relative positions when coordinates are unavoidable:

| Device | Width | Height |
|--------|-------|--------|
| 38mm | 136 | 170 |
| 40mm | 162 | 197 |
| 41mm | 176 | 215 |
| 42mm | 156 | 195 |
| 44mm | 184 | 224 |
| 45mm | 198 | 242 |
| 49mm Ultra | 205 | 251 |

**Best practice:** Get actual dimensions from `describe-ui` AXFrame rather than assuming from this table.
