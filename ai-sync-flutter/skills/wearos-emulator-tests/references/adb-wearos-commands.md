# ADB Command Reference for Wear OS

Wear OS-specific adb commands and patterns. See also `android-emulator-tests/references/adb-commands.md` for general adb reference.

## Setup

```bash
export ANDROID_HOME=~/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/platform-tools:$ANDROID_HOME/emulator
```

## Emulator Management

```bash
# List AVDs
~/Library/Android/sdk/emulator/emulator -list-avds

# Launch Wear OS emulator
~/Library/Android/sdk/emulator/emulator -avd WearOS5 &

# Launch with options
~/Library/Android/sdk/emulator/emulator -avd WearOS5 \
  -gpu host \
  -no-snapshot-load &

# Wait for boot
adb wait-for-device
while [ "$(adb shell getprop sys.boot_completed 2>/dev/null)" != "1" ]; do
  sleep 2
done
echo "Boot complete"
```

## Screen Size Detection

**Always get screen size dynamically - never hardcode coordinates:**

```bash
# Get screen dimensions
adb shell wm size
# Output: Physical size: 454x454

# Parse into variables
SIZE=$(adb shell wm size | grep -o '[0-9]*x[0-9]*')
WIDTH=$(echo $SIZE | cut -d'x' -f1)
HEIGHT=$(echo $SIZE | cut -d'x' -f2)

echo "Width: $WIDTH, Height: $HEIGHT"
```

## Wear OS-Specific Key Events

### Hardware Buttons

```bash
# Home (watch face / app drawer)
adb shell input keyevent KEYCODE_HOME

# Back
adb shell input keyevent KEYCODE_BACK

# Power
adb shell input keyevent KEYCODE_POWER

# Wake up screen
adb shell input keyevent KEYCODE_WAKEUP

# Sleep
adb shell input keyevent KEYCODE_SLEEP
```

### Rotary Input (Digital Crown)

```bash
# Scroll up (crown rotate up)
adb shell input keyevent KEYCODE_NAVIGATE_PREVIOUS

# Scroll down (crown rotate down)
adb shell input keyevent KEYCODE_NAVIGATE_NEXT

# Alternative scroll
adb shell input keyevent KEYCODE_SCROLL_UP
adb shell input keyevent KEYCODE_SCROLL_DOWN

# Page navigation
adb shell input keyevent KEYCODE_PAGE_UP
adb shell input keyevent KEYCODE_PAGE_DOWN
```

### Button Gestures

```bash
# Long press power (power menu)
adb shell input keyevent --longpress KEYCODE_POWER
```

## Dynamic Tap Calculation

```bash
#!/bin/bash
# tap_element.sh - Tap element by text from UI dump

TEXT="$1"

# Dump UI
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml /tmp/ui.xml 2>/dev/null

# Find element with matching text
BOUNDS=$(grep "text=\"$TEXT\"" /tmp/ui.xml | grep -o 'bounds="\[[0-9]*,[0-9]*\]\[[0-9]*,[0-9]*\]"' | head -1)

if [ -z "$BOUNDS" ]; then
  # Try content-desc
  BOUNDS=$(grep "content-desc=\"$TEXT\"" /tmp/ui.xml | grep -o 'bounds="\[[0-9]*,[0-9]*\]\[[0-9]*,[0-9]*\]"' | head -1)
fi

if [ -n "$BOUNDS" ]; then
  # Parse bounds="[left,top][right,bottom]"
  COORDS=$(echo $BOUNDS | grep -o '[0-9]*')
  LEFT=$(echo $COORDS | cut -d' ' -f1)
  TOP=$(echo $COORDS | cut -d' ' -f2)
  RIGHT=$(echo $COORDS | cut -d' ' -f3)
  BOTTOM=$(echo $COORDS | cut -d' ' -f4)

  # Calculate center
  X=$(( (LEFT + RIGHT) / 2 ))
  Y=$(( (TOP + BOTTOM) / 2 ))

  echo "Tapping '$TEXT' at ($X, $Y)"
  adb shell input tap $X $Y
else
  echo "Element '$TEXT' not found"
  exit 1
fi
```

## Dynamic Swipe Calculation

```bash
#!/bin/bash
# Swipe based on screen size

# Get dimensions
SIZE=$(adb shell wm size | grep -o '[0-9]*x[0-9]*')
WIDTH=$(echo $SIZE | cut -d'x' -f1)
HEIGHT=$(echo $SIZE | cut -d'x' -f2)

CENTER_X=$((WIDTH / 2))
TOP_Y=$((HEIGHT * 15 / 100))      # 15% from top
BOTTOM_Y=$((HEIGHT * 85 / 100))   # 85% from top

case "$1" in
  up)
    adb shell input swipe $CENTER_X $TOP_Y $CENTER_X $BOTTOM_Y 300
    ;;
  down)
    adb shell input swipe $CENTER_X $BOTTOM_Y $CENTER_X $TOP_Y 300
    ;;
  left)
    adb shell input swipe $((WIDTH * 85 / 100)) $CENTER_Y $((WIDTH * 15 / 100)) $CENTER_Y 200
    ;;
  right)
    adb shell input swipe $((WIDTH * 15 / 100)) $CENTER_Y $((WIDTH * 85 / 100)) $CENTER_Y 200
    ;;
  *)
    echo "Usage: $0 {up|down|left|right}"
    ;;
esac
```

## Screenshots

```bash
# Basic screenshot
adb shell screencap /sdcard/screen.png
adb pull /sdcard/screen.png /tmp/wearos_screen.png

# With timestamp
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
adb shell screencap /sdcard/screen.png
adb pull /sdcard/screen.png "/tmp/wearos_${TIMESTAMP}.png"
```

## UI Hierarchy

```bash
# Dump and pull
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml /tmp/ui.xml

# View formatted
cat /tmp/ui.xml | xmllint --format - | less

# Search for elements
grep -o 'text="[^"]*"' /tmp/ui.xml | sort -u
grep -o 'content-desc="[^"]*"' /tmp/ui.xml | sort -u
grep -o 'resource-id="[^"]*"' /tmp/ui.xml | sort -u
```

## Round Screen Awareness

On round screens, corners are outside the visible area:

```bash
# Safe tap zone calculation for round screen
SIZE=$(adb shell wm size | grep -o '[0-9]*x[0-9]*')
WIDTH=$(echo $SIZE | cut -d'x' -f1)
HEIGHT=$(echo $SIZE | cut -d'x' -f2)

# For round screens, visible area is inscribed circle
# Safe zone is roughly 70% of diameter centered
SAFE_MARGIN=$((WIDTH * 15 / 100))  # 15% margin from edge

SAFE_LEFT=$SAFE_MARGIN
SAFE_TOP=$SAFE_MARGIN
SAFE_RIGHT=$((WIDTH - SAFE_MARGIN))
SAFE_BOTTOM=$((HEIGHT - SAFE_MARGIN))

echo "Safe tap zone: [$SAFE_LEFT,$SAFE_TOP] to [$SAFE_RIGHT,$SAFE_BOTTOM]"
```

## Wear OS Tiles

```bash
# Swipe right from watch face to access tiles
SIZE=$(adb shell wm size | grep -o '[0-9]*x[0-9]*')
WIDTH=$(echo $SIZE | cut -d'x' -f1)
HEIGHT=$(echo $SIZE | cut -d'x' -f2)
CENTER_Y=$((HEIGHT / 2))

# Swipe right for tiles
adb shell input swipe $((WIDTH * 15 / 100)) $CENTER_Y $((WIDTH * 85 / 100)) $CENTER_Y 200
```

## Quick Glances

```bash
# Swipe up from watch face for quick glances
SIZE=$(adb shell wm size | grep -o '[0-9]*x[0-9]*')
WIDTH=$(echo $SIZE | cut -d'x' -f1)
HEIGHT=$(echo $SIZE | cut -d'x' -f2)
CENTER_X=$((WIDTH / 2))

adb shell input swipe $CENTER_X $((HEIGHT * 85 / 100)) $CENTER_X $((HEIGHT * 15 / 100)) 300
```

## Logs

```bash
# All logs
adb logcat

# Wear-specific
adb logcat -s WearableService
adb logcat -s WearableListenerService

# App logs
PID=$(adb shell pidof com.techup.roundsTracker)
adb logcat --pid=$PID
```

## Pairing with Phone Emulator

```bash
# List all devices
adb devices

# If both phone and watch emulators running:
# Phone: emulator-5554
# Watch: emulator-5556

# Target specific device
adb -s emulator-5556 shell input tap 200 200  # Watch
adb -s emulator-5554 shell input tap 540 1200  # Phone
```

## Best Practices

1. **Always get screen size dynamically** - `adb shell wm size`
2. **Use hardware buttons** when possible - they're size-independent
3. **Parse UI dump for coordinates** - don't guess
4. **Account for round screens** - corners are not tappable
5. **Use rotary input** for scrolling - more reliable than swipes
6. **Wake screen first** - `KEYCODE_WAKEUP` before interactions
