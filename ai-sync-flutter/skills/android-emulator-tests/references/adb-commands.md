# ADB Command Reference for Android Emulator

Complete reference for adb commands used with Android emulators.

## Setup

```bash
# Add to PATH (add to ~/.zshrc for persistence)
export ANDROID_HOME=~/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator
```

## Device Management

```bash
# List connected devices/emulators
adb devices

# Connect to specific device (if multiple)
adb -s emulator-5554 <command>

# Wait for device to be ready
adb wait-for-device

# Check boot status
adb shell getprop sys.boot_completed  # Returns 1 when ready

# Reboot device
adb reboot
```

## App Management

```bash
# Install APK
adb install path/to/app.apk

# Install and replace existing
adb install -r path/to/app.apk

# Uninstall by package name
adb uninstall com.techup.roundsTracker

# List installed packages
adb shell pm list packages | grep techup

# Clear app data
adb shell pm clear com.techup.roundsTracker

# Force stop app
adb shell am force-stop com.techup.roundsTracker
```

## Launch App

```bash
# Using monkey (simplest)
adb shell monkey -p com.techup.roundsTracker -c android.intent.category.LAUNCHER 1

# Using am start with activity
adb shell am start -n com.techup.roundsTracker/.MainActivity

# Launch with intent
adb shell am start -a android.intent.action.MAIN -n com.techup.roundsTracker/.MainActivity
```

## UI Hierarchy

```bash
# Dump to device and pull
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml /tmp/ui.xml

# Format and view
cat /tmp/ui.xml | xmllint --format - | less

# One-liner dump to stdout (less readable)
adb shell uiautomator dump /dev/tty

# Key attributes to look for:
# - bounds="[left,top][right,bottom]" - element position
# - resource-id="com.app:id/button" - unique identifier
# - text="Button Text" - visible text
# - content-desc="Accessibility label" - accessibility label
# - clickable="true" - can be tapped
```

## Screenshots

```bash
# Capture screenshot
adb shell screencap /sdcard/screen.png
adb pull /sdcard/screen.png /tmp/screenshot.png

# One-liner
adb shell screencap -p > /tmp/screenshot.png

# With timestamp
adb shell screencap /sdcard/screen.png && adb pull /sdcard/screen.png "/tmp/screenshot_$(date +%H%M%S).png"
```

## Screen Recording

```bash
# Record screen (max 180 seconds)
adb shell screenrecord /sdcard/recording.mp4

# Stop with Ctrl+C, then pull
adb pull /sdcard/recording.mp4 /tmp/recording.mp4

# With time limit
adb shell screenrecord --time-limit 30 /sdcard/recording.mp4
```

## Input Commands

### Tap

```bash
# Tap at coordinates
adb shell input tap <x> <y>

# Example
adb shell input tap 540 1200
```

### Swipe

```bash
# Swipe from point to point
adb shell input swipe <x1> <y1> <x2> <y2> [duration_ms]

# Scroll down (content moves up)
adb shell input swipe 540 1500 540 500 300

# Scroll up (content moves down)
adb shell input swipe 540 500 540 1500 300

# Swipe left (next page)
adb shell input swipe 900 1000 100 1000 200

# Swipe right (previous page)
adb shell input swipe 100 1000 900 1000 200

# Long press (swipe to same point with duration)
adb shell input swipe 540 1200 540 1200 1000
```

### Text Input

```bash
# Type text (no spaces)
adb shell input text "hello"

# Text with spaces (use %s)
adb shell input text "hello%sworld"

# Special characters need escaping
adb shell input text "test\@email.com"
```

### Key Events

```bash
# Navigation
adb shell input keyevent KEYCODE_HOME
adb shell input keyevent KEYCODE_BACK
adb shell input keyevent KEYCODE_APP_SWITCH
adb shell input keyevent KEYCODE_MENU

# Text editing
adb shell input keyevent KEYCODE_ENTER
adb shell input keyevent KEYCODE_DEL          # Backspace
adb shell input keyevent KEYCODE_TAB
adb shell input keyevent KEYCODE_SPACE

# Media
adb shell input keyevent KEYCODE_VOLUME_UP
adb shell input keyevent KEYCODE_VOLUME_DOWN
adb shell input keyevent KEYCODE_VOLUME_MUTE
adb shell input keyevent KEYCODE_MEDIA_PLAY_PAUSE

# Power
adb shell input keyevent KEYCODE_POWER
adb shell input keyevent KEYCODE_WAKEUP
adb shell input keyevent KEYCODE_SLEEP

# Dpad (for navigation)
adb shell input keyevent KEYCODE_DPAD_UP
adb shell input keyevent KEYCODE_DPAD_DOWN
adb shell input keyevent KEYCODE_DPAD_LEFT
adb shell input keyevent KEYCODE_DPAD_RIGHT
adb shell input keyevent KEYCODE_DPAD_CENTER
```

## Screen Info

```bash
# Get screen resolution
adb shell wm size
# Output: Physical size: 1080x2400

# Get screen density
adb shell wm density

# Get display info
adb shell dumpsys display | grep -E "mDisplayWidth|mDisplayHeight"
```

## Logs

```bash
# View all logs
adb logcat

# Clear logs first
adb logcat -c && adb logcat

# Filter by tag
adb logcat -s Flutter
adb logcat -s ActivityManager

# Filter by priority (V=Verbose, D=Debug, I=Info, W=Warn, E=Error)
adb logcat *:E  # Errors only

# Filter by app PID
PID=$(adb shell pidof com.techup.roundsTracker)
adb logcat --pid=$PID

# Save to file
adb logcat > /tmp/logs.txt
```

## File Operations

```bash
# Push file to device
adb push /local/file.txt /sdcard/file.txt

# Pull file from device
adb pull /sdcard/file.txt /local/file.txt

# List files
adb shell ls /sdcard/

# Remove file
adb shell rm /sdcard/file.txt
```

## Shell Access

```bash
# Interactive shell
adb shell

# Run single command
adb shell <command>

# Run as root (if available)
adb root
adb shell
```

## Emulator Control

```bash
# List AVDs
~/Library/Android/sdk/emulator/emulator -list-avds

# Launch emulator
~/Library/Android/sdk/emulator/emulator -avd <avd-name>

# Launch options
~/Library/Android/sdk/emulator/emulator -avd <avd-name> \
  -gpu host \           # GPU acceleration
  -no-snapshot-load \   # Cold boot
  -no-window \          # Headless
  -no-audio             # No audio

# Kill emulator
adb emu kill
```

## Calculating Tap Coordinates

From UI dump bounds `bounds="[left,top][right,bottom]"`:

```bash
# Example: bounds="[100,200][300,400]"
# Center X = (100 + 300) / 2 = 200
# Center Y = (200 + 400) / 2 = 300
# Command: adb shell input tap 200 300
```

## Accessibility-Based Tapping

Parse content-desc or text from UI dump:

```bash
# Find element and extract bounds
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml /tmp/ui.xml

# Search for element
grep -o 'content-desc="[^"]*"[^>]*bounds="[^"]*"' /tmp/ui.xml
grep -o 'text="Start"[^>]*bounds="[^"]*"' /tmp/ui.xml
```

## Best Practices

1. **Always dump UI hierarchy first** to find element positions
2. **Use content-desc/text** to locate elements, then tap their bounds
3. **Get screen size dynamically** - don't hardcode coordinates
4. **Wait after actions** - UI may need time to update
5. **Clear logs before testing** for cleaner output
