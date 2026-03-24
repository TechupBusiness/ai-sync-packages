---
name: android-emulator-tests
description: Build, run, and test the Android app on emulator. Uses adb for UI automation. Trigger for Android app testing, UI verification, screenshot capture, and emulator interaction.
---

# Android Emulator Testing

Build and run the Android app on emulator, interact with UI, and verify behavior.

## Quick Start

1. List available emulators: `~/Library/Android/sdk/emulator/emulator -list-avds`
2. Launch emulator: `~/Library/Android/sdk/emulator/emulator -avd <avd-name>`
3. Use `adb` commands for all interactions

## Prerequisites

- Android SDK installed at `~/Library/Android/sdk`
- At least one AVD created (e.g., `Pixel_6_API_32`)
- Add to PATH for convenience:
  ```bash
  export ANDROID_HOME=~/Library/Android/sdk
  export PATH=$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator
  ```

## Core Workflow

### 1) List and Launch Emulator

```bash
# List available AVDs
~/Library/Android/sdk/emulator/emulator -list-avds

# Launch emulator (runs in background)
~/Library/Android/sdk/emulator/emulator -avd Pixel_6_API_32 &

# Wait for boot to complete
adb wait-for-device
adb shell getprop sys.boot_completed  # Returns 1 when ready
```

### 2) Build Android App

```bash
# Flutter build
flutter build apk --debug

# Or build bundle
flutter build appbundle --debug
```

### 3) Install and Launch

```bash
# Install APK
adb install build/app/outputs/flutter-apk/app-debug.apk

# Launch app by package name
adb shell am start -n com.techup.roundsTracker/.MainActivity

# Or launch with intent
adb shell monkey -p com.techup.roundsTracker -c android.intent.category.LAUNCHER 1
```

## UI Interaction

**All interactions use `adb` commands.**

**CRITICAL: Always get UI hierarchy first to find element bounds/resource-ids.**

### Get UI Hierarchy (Always First!)

```bash
# Dump UI hierarchy to file
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml /tmp/ui.xml
cat /tmp/ui.xml | xmllint --format - | head -100

# Or use one-liner (less readable)
adb shell uiautomator dump /dev/tty
```

Look for `bounds`, `resource-id`, `text`, and `content-desc` attributes.

### Screenshot

```bash
# Capture and pull screenshot
adb shell screencap /sdcard/screen.png
adb pull /sdcard/screen.png /tmp/android_screenshot.png
```

### Tap (Prefer Resource IDs Over Coordinates)

```bash
# PREFERRED: Tap by text using uiautomator (requires element to be visible)
adb shell "uiautomator dump /dev/tty | grep -o 'bounds=\"[^\"]*\".*text=\"Start\"' | head -1"
# Then parse bounds and tap center

# FALLBACK: Tap by coordinates (get from UI dump bounds)
adb shell input tap <x> <y>

# Example: bounds="[540,1200][900,1350]" → center is (720, 1275)
adb shell input tap 720 1275
```

### Swipe

```bash
# Swipe from (x1,y1) to (x2,y2) with duration in ms
adb shell input swipe <x1> <y1> <x2> <y2> [duration_ms]

# Scroll down (swipe up)
adb shell input swipe 540 1500 540 500 300

# Scroll up (swipe down)
adb shell input swipe 540 500 540 1500 300

# Swipe left
adb shell input swipe 900 1000 100 1000 200

# Swipe right
adb shell input swipe 100 1000 900 1000 200
```

### Type Text

```bash
# Type text (spaces need escaping)
adb shell input text "hello"
adb shell input text "hello%sworld"  # %s = space

# Or use keyevents for special chars
adb shell input keyevent KEYCODE_SPACE
```

### Hardware Buttons

```bash
# Home button
adb shell input keyevent KEYCODE_HOME

# Back button
adb shell input keyevent KEYCODE_BACK

# Recent apps
adb shell input keyevent KEYCODE_APP_SWITCH

# Power button
adb shell input keyevent KEYCODE_POWER

# Volume
adb shell input keyevent KEYCODE_VOLUME_UP
adb shell input keyevent KEYCODE_VOLUME_DOWN

# Enter/OK
adb shell input keyevent KEYCODE_ENTER
```

## Helper Script: Tap by Text

Since Android doesn't have a simple "tap by label" command, use this pattern:

```bash
# Function to tap element by text
tap_by_text() {
  local text="$1"
  adb shell uiautomator dump /sdcard/ui.xml
  adb pull /sdcard/ui.xml /tmp/ui.xml 2>/dev/null

  # Extract bounds for element with matching text
  local bounds=$(grep "text=\"$text\"" /tmp/ui.xml | grep -o 'bounds="\[[0-9]*,[0-9]*\]\[[0-9]*,[0-9]*\]"' | head -1)

  if [ -n "$bounds" ]; then
    # Parse bounds [left,top][right,bottom]
    local coords=$(echo $bounds | grep -o '[0-9]*' | tr '\n' ' ')
    local left=$(echo $coords | cut -d' ' -f1)
    local top=$(echo $coords | cut -d' ' -f2)
    local right=$(echo $coords | cut -d' ' -f3)
    local bottom=$(echo $coords | cut -d' ' -f4)

    # Calculate center
    local x=$(( (left + right) / 2 ))
    local y=$(( (top + bottom) / 2 ))

    adb shell input tap $x $y
    echo "Tapped '$text' at ($x, $y)"
  else
    echo "Element with text '$text' not found"
  fi
}

# Usage
tap_by_text "Start"
tap_by_text "Settings"
```

## Screen Dimensions

Get actual screen size dynamically:

```bash
# Get screen resolution
adb shell wm size
# Output: Physical size: 1080x2400

# Get density
adb shell wm density
```

## Complete Workflow Example

```bash
# 1. Ensure emulator is running
adb devices  # Should show device

# 2. Build and install
flutter build apk --debug
adb install -r build/app/outputs/flutter-apk/app-debug.apk

# 3. Launch app
adb shell monkey -p com.techup.roundsTracker -c android.intent.category.LAUNCHER 1
sleep 3

# 4. Screenshot initial state
adb shell screencap /sdcard/screen.png && adb pull /sdcard/screen.png /tmp/android_initial.png

# 5. Get UI hierarchy
adb shell uiautomator dump /sdcard/ui.xml && adb pull /sdcard/ui.xml /tmp/ui.xml

# 6. Find and tap element (parse bounds from ui.xml)
# Example: tap center of screen
adb shell input tap 540 1200
sleep 1

# 7. Screenshot result
adb shell screencap /sdcard/screen.png && adb pull /sdcard/screen.png /tmp/android_after.png
```

## Logs

```bash
# View all logs
adb logcat

# Filter by tag
adb logcat -s Flutter

# Filter by package (requires PID)
PID=$(adb shell pidof com.techup.roundsTracker)
adb logcat --pid=$PID

# Clear logs
adb logcat -c
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `adb: command not found` | Add to PATH: `export PATH=$PATH:~/Library/Android/sdk/platform-tools` |
| No devices found | Launch emulator first or check `adb devices` |
| App not installed | Check package name, rebuild APK |
| UI dump empty | Wait for app to fully load, try again |
| Tap not working | Verify coordinates from UI dump bounds |
| Emulator slow | Use `-gpu host` flag when launching |

## Emulator Options

```bash
# Launch with GPU acceleration
~/Library/Android/sdk/emulator/emulator -avd Pixel_6_API_32 -gpu host

# Launch headless (no window)
~/Library/Android/sdk/emulator/emulator -avd Pixel_6_API_32 -no-window

# Cold boot (fresh start)
~/Library/Android/sdk/emulator/emulator -avd Pixel_6_API_32 -no-snapshot-load
```

## Flutter-Specific

For better UI automation with Flutter, ensure widgets have semantic labels:

```dart
Semantics(
  label: 'Start session',
  button: true,
  child: ElevatedButton(...),
)
```

These appear as `content-desc` in the UI dump.
