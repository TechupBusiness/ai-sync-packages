# iOS Simulator Testing - CLI Fallback

Use these commands when XcodeBuildMCP MCP tools are unavailable.

## When to Use This Fallback

Use CLI commands instead of MCP tools when:
- `mcp__XcodeBuildMCP__*` tools are not available
- XcodeBuildMCP server failed to start
- Running in environments without MCP support

## Prerequisites

```bash
# Install AXe for UI automation
brew install cameroncooke/axe/axe

# Verify installation
which axe
axe --version
```

## Core Commands

### 1. List and Manage Simulators

```bash
# List all simulators
xcrun simctl list devices

# List booted simulators only
xcrun simctl list devices booted

# Boot a simulator
xcrun simctl boot "iPhone 15 Pro"

# Shut down a simulator
xcrun simctl shutdown <UDID>
```

### 2. Build the App

```bash
# Build for iOS simulator (Flutter)
flutter build ios --debug --no-codesign --simulator

# Build for iOS simulator (native Xcode project)
xcodebuild -workspace ios/Runner.xcworkspace \
  -scheme Runner \
  -configuration Debug \
  -destination 'platform=iOS Simulator,name=iPhone 15 Pro' \
  build
```

### 3. Install and Launch App

```bash
# Get simulator UDID (use booted one)
SIMULATOR_ID=$(xcrun simctl list devices booted -j | jq -r '.devices[][] | select(.state=="Booted") | .udid' | head -1)

# Install app on simulator
xcrun simctl install $SIMULATOR_ID /path/to/Runner.app

# For Flutter, the path is typically:
xcrun simctl install $SIMULATOR_ID build/ios/iphonesimulator/Runner.app

# Launch app by bundle ID
xcrun simctl launch $SIMULATOR_ID com.techup.roundsTracker

# Launch and wait for debugger (optional)
xcrun simctl launch --wait-for-debugger $SIMULATOR_ID com.techup.roundsTracker
```

### 4. Screenshots

```bash
# Capture screenshot to file
xcrun simctl io $SIMULATOR_ID screenshot /tmp/screenshot.png

# Capture with timestamp
xcrun simctl io $SIMULATOR_ID screenshot "/tmp/screenshot_$(date +%Y%m%d_%H%M%S).png"
```

### 5. UI Interaction with AXe

```bash
# Describe UI accessibility hierarchy
axe describe-ui

# Tap by coordinates
axe tap --point 200,400

# Tap by accessibility label (if available)
axe tap --label "Start"

# Type text
axe type "Hello World"

# Swipe gestures
axe swipe --from 300,500 --to 100,500 --duration 0.2  # Swipe left
axe swipe --from 100,500 --to 300,500 --duration 0.2  # Swipe right
axe swipe --from 200,600 --to 200,300 --duration 0.2  # Scroll down

# Press hardware buttons
axe button home
axe button lock
```

### 6. App Logs

```bash
# Stream logs from simulator
xcrun simctl spawn $SIMULATOR_ID log stream --predicate 'subsystem == "com.techup.roundsTracker"'

# Stream all logs (noisy)
xcrun simctl spawn $SIMULATOR_ID log stream

# View recent logs
xcrun simctl spawn $SIMULATOR_ID log show --last 5m --predicate 'subsystem == "com.techup.roundsTracker"'
```

## Complete Workflow Example

```bash
# 1. Get booted simulator
SIMULATOR_ID=$(xcrun simctl list devices booted -j | jq -r '.devices[][] | select(.state=="Booted") | .udid' | head -1)
echo "Using simulator: $SIMULATOR_ID"

# 2. Build the app
flutter build ios --debug --no-codesign --simulator

# 3. Install and launch
xcrun simctl install $SIMULATOR_ID build/ios/iphonesimulator/Runner.app
xcrun simctl launch $SIMULATOR_ID com.techup.roundsTracker

# 4. Wait for app to load
sleep 3

# 5. Capture initial state
xcrun simctl io $SIMULATOR_ID screenshot /tmp/app_launched.png

# 6. Describe UI for interaction targets
axe describe-ui

# 7. Interact with UI
axe tap --label "Profiles"
sleep 1
xcrun simctl io $SIMULATOR_ID screenshot /tmp/profiles_screen.png

# 8. Scroll down
axe swipe --from 200,600 --to 200,200 --duration 0.2
sleep 0.5
xcrun simctl io $SIMULATOR_ID screenshot /tmp/scrolled.png
```

## AXe Command Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `describe-ui` | Get accessibility tree | `axe describe-ui` |
| `tap` | Tap element | `axe tap --point 200,400` or `axe tap --label "OK"` |
| `type` | Enter text | `axe type "email@example.com"` |
| `swipe` | Swipe gesture | `axe swipe --from 300,500 --to 100,500 --duration 0.15` |
| `button` | Hardware button | `axe button home` |
| `screenshot` | Capture screen | `axe screenshot /tmp/shot.png` |
| `list-simulators` | List simulators | `axe list-simulators` |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `axe` not found | `brew install cameroncooke/axe/axe` |
| No booted simulator | Boot one: `xcrun simctl boot "iPhone 15 Pro"` |
| App not installing | Check build path exists, rebuild if needed |
| UI elements not found | Run `axe describe-ui` to see available elements |
| Swipe not working | Try shorter duration (0.1-0.2 seconds) |
| Screenshots blank | Wait longer after launch (`sleep 3`) |

## Integration with Quality Gates

When MCP is unavailable, the CLI workflow still satisfies UI verification:

1. Build: `flutter build ios --debug --no-codesign --simulator`
2. Install: `xcrun simctl install ...`
3. Launch: `xcrun simctl launch ...`
4. Screenshot: `xcrun simctl io ... screenshot ...`
5. Read screenshot with Claude's Read tool to verify visually
