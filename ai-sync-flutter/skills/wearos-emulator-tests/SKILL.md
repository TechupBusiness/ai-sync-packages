---
name: wearos-emulator-tests
description: Build, run, and test the Wear OS app on emulator. Uses adb for UI automation. Trigger for Wear OS smartwatch app testing, UI verification, screenshot capture, and emulator interaction.
---

# Wear OS Emulator Testing

Build and run the Wear OS app on emulator, interact with UI, and verify behavior.

## Quick Start

1. List available emulators: `~/Library/Android/sdk/emulator/emulator -list-avds`
2. Launch Wear OS emulator: `~/Library/Android/sdk/emulator/emulator -avd WearOS5`
3. Use `adb` commands for all interactions

## Prerequisites

- Android SDK installed at `~/Library/Android/sdk`
- Wear OS AVD created (e.g., `WearOS5`)
- Add to PATH for convenience:
  ```bash
  export ANDROID_HOME=~/Library/Android/sdk
  export PATH=$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator
  ```

**Create Wear OS AVD if needed:**
```bash
sdkmanager "system-images;android-34;android-wear;arm64-v8a"
avdmanager create avd -n "WearOS5" -k "system-images;android-34;android-wear;arm64-v8a" -d "wear_round"
```

## Core Workflow

### 1) Launch Wear OS Emulator

```bash
# Launch emulator
~/Library/Android/sdk/emulator/emulator -avd WearOS5 &

# Wait for boot
adb wait-for-device
adb shell getprop sys.boot_completed  # Returns 1 when ready

# May need extra time for Wear OS
sleep 10
```

### 2) Build Wear OS App

```bash
# Flutter with Wear OS support
flutter build apk --debug

# For dedicated Wear OS module
cd wear && ./gradlew assembleDebug
```

### 3) Install and Launch

```bash
# Install APK
adb install build/app/outputs/flutter-apk/app-debug.apk

# Launch app
adb shell monkey -p com.techup.roundsTracker -c android.intent.category.LAUNCHER 1
```

## UI Interaction

**All interactions use `adb` commands.**

**CRITICAL: Always use accessibility labels/content-desc, not hardcoded coordinates.** Wear OS screens vary in size (round, square, different resolutions).

### Get UI Hierarchy (Always First!)

```bash
# Dump UI hierarchy
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml /tmp/ui.xml
cat /tmp/ui.xml | xmllint --format - | head -100
```

Look for `bounds`, `content-desc`, and `text` attributes.

### Screenshot

```bash
adb shell screencap /sdcard/screen.png
adb pull /sdcard/screen.png /tmp/wearos_screenshot.png
```

### Get Screen Size Dynamically

```bash
# Get actual screen dimensions
adb shell wm size
# Example output: Physical size: 454x454 (round watch)
```

### Tap (Calculate from UI Bounds)

```bash
# 1. Get UI hierarchy first
adb shell uiautomator dump /sdcard/ui.xml && adb pull /sdcard/ui.xml /tmp/ui.xml

# 2. Find element bounds from dump
# Example: bounds="[100,200][200,300]"
# Center: x=(100+200)/2=150, y=(200+300)/2=250

# 3. Tap
adb shell input tap 150 250
```

### Swipe (Calculate from Screen Size)

```bash
# Get screen size first
SIZE=$(adb shell wm size | grep -o '[0-9]*x[0-9]*')
WIDTH=$(echo $SIZE | cut -d'x' -f1)
HEIGHT=$(echo $SIZE | cut -d'x' -f2)

# Calculate positions
CENTER_X=$((WIDTH / 2))
TOP_Y=$((HEIGHT / 10))
BOTTOM_Y=$((HEIGHT * 9 / 10))

# Scroll down
adb shell input swipe $CENTER_X $BOTTOM_Y $CENTER_X $TOP_Y 300

# Scroll up
adb shell input swipe $CENTER_X $TOP_Y $CENTER_X $BOTTOM_Y 300
```

### Hardware Buttons

These are size-independent - **prefer over coordinate-based interactions:**

```bash
# Home (show watch face / app drawer)
adb shell input keyevent KEYCODE_HOME

# Back
adb shell input keyevent KEYCODE_BACK

# Power button
adb shell input keyevent KEYCODE_POWER

# Wake up
adb shell input keyevent KEYCODE_WAKEUP
```

### Rotary Input (Digital Crown)

```bash
# Scroll via rotary input (simulates crown rotation)
adb shell input keyevent KEYCODE_NAVIGATE_PREVIOUS  # Scroll up
adb shell input keyevent KEYCODE_NAVIGATE_NEXT      # Scroll down

# Or use generic scroll
adb shell input keyevent KEYCODE_SCROLL_UP
adb shell input keyevent KEYCODE_SCROLL_DOWN
```

## Size-Independent Navigation

Use hardware buttons whenever possible:

```bash
# Return to watch face
adb shell input keyevent KEYCODE_HOME

# Go back
adb shell input keyevent KEYCODE_BACK

# Wake screen
adb shell input keyevent KEYCODE_WAKEUP
```

## Dynamic Coordinate Calculation

When you must use coordinates:

```bash
#!/bin/bash
# Get screen dimensions
SIZE=$(adb shell wm size | grep -o '[0-9]*x[0-9]*')
WIDTH=$(echo $SIZE | cut -d'x' -f1)
HEIGHT=$(echo $SIZE | cut -d'x' -f2)

echo "Screen: ${WIDTH}x${HEIGHT}"

# Calculate common positions
CENTER_X=$((WIDTH / 2))
CENTER_Y=$((HEIGHT / 2))
echo "Center: ($CENTER_X, $CENTER_Y)"

# Tap center
adb shell input tap $CENTER_X $CENTER_Y
```

## Complete Workflow Example

```bash
# 1. Ensure emulator is running
adb devices

# 2. Get screen size
SIZE=$(adb shell wm size | grep -o '[0-9]*x[0-9]*')
echo "Screen size: $SIZE"

# 3. Build and install
flutter build apk --debug
adb install -r build/app/outputs/flutter-apk/app-debug.apk

# 4. Launch app
adb shell monkey -p com.techup.roundsTracker -c android.intent.category.LAUNCHER 1
sleep 3

# 5. Screenshot initial state
adb shell screencap /sdcard/screen.png && adb pull /sdcard/screen.png /tmp/wearos_initial.png

# 6. Get UI hierarchy
adb shell uiautomator dump /sdcard/ui.xml && adb pull /sdcard/ui.xml /tmp/ui.xml

# 7. Find element in UI dump and tap (example)
# grep for text or content-desc, extract bounds, calculate center, tap

# 8. Screenshot result
adb shell screencap /sdcard/screen.png && adb pull /sdcard/screen.png /tmp/wearos_after.png
```

## Logs

```bash
# View logs
adb logcat

# Filter by app
PID=$(adb shell pidof com.techup.roundsTracker)
adb logcat --pid=$PID

# Filter for Wear-specific
adb logcat -s WearableService
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Emulator slow to boot | Wear OS takes longer; wait 30-60 seconds |
| Screen stays black | Press power: `adb shell input keyevent KEYCODE_WAKEUP` |
| UI dump empty | Wait for app to load, wake screen first |
| Tap not working | Verify coordinates from UI dump bounds |
| Round screen clipping | Elements near edge may be outside tap area |

## Wear OS Screen Sizes Reference

Different Wear OS devices have different resolutions. **Always get actual size dynamically.**

| Device Type | Typical Resolution |
|-------------|-------------------|
| Small round | 384x384 |
| Medium round | 400x400 |
| Large round | 454x454 |
| Square | 280x280 to 320x320 |

**Best practice:** Always use `adb shell wm size` to get actual dimensions.

## Round Screen Considerations

On round screens, corners are not visible/tappable:

```
    ┌───────────┐
   /             \
  │   Visible    │
  │    Area      │
   \             /
    └───────────┘
```

- Elements in corners may be clipped
- Prefer tapping center or near-center areas
- Use UI dump to verify element is actually visible

## Flutter Wear OS Apps

Ensure widgets have semantic labels for automation:

```dart
Semantics(
  label: 'Start workout',
  button: true,
  child: ElevatedButton(...),
)
```

These appear as `content-desc` in the UI dump.

## Differences from watchOS

| Feature | Wear OS (adb) | watchOS (axe) |
|---------|--------------|---------------|
| Screenshot | `adb shell screencap` | `xcrun simctl io screenshot` |
| UI hierarchy | `adb shell uiautomator dump` | `axe describe-ui` |
| Tap by label | Parse UI dump + tap coords | `axe tap --label` |
| Hardware buttons | `adb shell input keyevent` | `axe button` |
| Rotary input | `KEYCODE_NAVIGATE_*` | Digital Crown via `button home` |
