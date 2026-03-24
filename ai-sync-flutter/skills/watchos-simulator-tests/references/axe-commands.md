# AXe Command Reference for watchOS

Complete reference for AXe CLI commands used with watchOS simulators.

## Core Syntax

All commands require `--udid <watch-simulator-udid>`:

```bash
WATCH_ID="<your-watch-udid>"
axe <command> --udid $WATCH_ID [options]
```

## Commands

### list-simulators

List all available simulators with their state.

```bash
axe list-simulators

# Output format:
# UDID | Device Name | State | Device Type | OS Version

# Filter for booted watches
axe list-simulators | grep -i watch | grep Booted
```

### describe-ui

Get the accessibility hierarchy of the current screen. **Always call this first!**

```bash
axe describe-ui --udid $WATCH_ID
```

Returns JSON with:
- `AXFrame`: Element position and size `"{{x, y}, {width, height}}"`
- `AXLabel`: Accessibility label (use with `--label`)
- `AXUniqueId`: Accessibility identifier (use with `--id`)
- `role`: Element type (Application, Button, StaticText, etc.)
- `children`: Nested elements

**Use this to:**
1. Find element labels/IDs for tapping
2. Get screen dimensions from root Application's AXFrame
3. Verify UI state after interactions

### tap

Tap on an element. **Prefer labels/IDs over coordinates for size-independence.**

```bash
# PREFERRED: By accessibility label (works on any watch size)
axe tap --udid $WATCH_ID --label "Start"

# PREFERRED: By accessibility ID (works on any watch size)
axe tap --udid $WATCH_ID --id "start-button"

# FALLBACK: By coordinates (only when no label/id available)
axe tap --udid $WATCH_ID -x <x> -y <y>

# With delays
axe tap --udid $WATCH_ID --label "OK" --pre-delay 0.5 --post-delay 1.0
```

### swipe

Perform swipe gesture. Calculate coordinates dynamically from screen size.

```bash
axe swipe --udid $WATCH_ID \
  --start-x <x1> --start-y <y1> \
  --end-x <x2> --end-y <y2> \
  [--duration <seconds>] \
  [--delta <pixels>]
```

Options:
- `--duration`: Swipe duration in seconds (default: 0.3)
- `--delta`: Pixel increment for gesture (default: 10)
- `--pre-delay`, `--post-delay`: Delays before/after

**Dynamic swipe example:**
```bash
# 1. Get screen size from describe-ui
# Assume AXFrame shows width=198, height=242

# 2. Calculate relative positions
# Center X = 198 / 2 = 99
# Top Y = 242 * 0.1 = 24
# Bottom Y = 242 * 0.9 = 218

# 3. Swipe up (scroll down)
axe swipe --udid $WATCH_ID --start-x 99 --start-y 218 --end-x 99 --end-y 24 --duration 0.2
```

### button

Press hardware buttons. **These are size-independent - prefer over swipes!**

```bash
axe button <button-type> --udid $WATCH_ID [--duration <seconds>]
```

Available buttons:
| Button | Action |
|--------|--------|
| `home` | Digital Crown press - go to watch face or app grid |
| `side-button` | Side button - app switcher |
| `siri` | Activate Siri |
| `lock` | Lock device |
| `apple-pay` | Apple Pay (double-click side button) |

Long press example:
```bash
axe button lock --udid $WATCH_ID --duration 2.0
```

### type

Type text character by character.

```bash
axe type --udid $WATCH_ID --text "Hello World"
```

### key

Press a specific key by keycode.

```bash
axe key --udid $WATCH_ID --keycode <code>
```

## Size-Independent Patterns

### Navigation (No Coordinates Needed)

```bash
# Go to app grid (double-press home)
axe button home --udid $WATCH_ID
sleep 0.3
axe button home --udid $WATCH_ID

# Return to watch face
axe button home --udid $WATCH_ID

# Open app switcher
axe button side-button --udid $WATCH_ID

# Activate Siri
axe button siri --udid $WATCH_ID
```

### Tap by Label (No Coordinates Needed)

```bash
# Tap any button/element by its accessibility label
axe tap --udid $WATCH_ID --label "Settings"
axe tap --udid $WATCH_ID --label "Start"
axe tap --udid $WATCH_ID --label "Cancel"
```

### Dynamic Swipes (When Necessary)

```bash
# Get dimensions first
WIDTH=$(axe describe-ui --udid $WATCH_ID | grep -o '"width" : [0-9]*' | head -1 | grep -o '[0-9]*')
HEIGHT=$(axe describe-ui --udid $WATCH_ID | grep -o '"height" : [0-9]*' | head -1 | grep -o '[0-9]*')

# Calculate positions
CENTER_X=$((WIDTH / 2))
TOP_Y=$((HEIGHT / 10))
BOTTOM_Y=$((HEIGHT * 9 / 10))

# Swipe up
axe swipe --udid $WATCH_ID --start-x $CENTER_X --start-y $BOTTOM_Y --end-x $CENTER_X --end-y $TOP_Y
```

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| `Missing expected argument '--udid'` | Forgot UDID | Add `--udid $WATCH_ID` |
| `Missing expected argument '--start-x'` | Wrong swipe syntax | Use `--start-x/y --end-x/y` |
| `Either provide both -x/-y, or use --id/--label` | Mixed tap syntax | Use EITHER coordinates OR label/id |
| Empty describe-ui children | No accessibility elements | App needs Semantics widgets |
| Tap has no effect | Wrong label or coordinates | Use `describe-ui` to verify element exists |

## Best Practices

1. **Always call `describe-ui` first** to understand what elements are available
2. **Use labels/IDs** for taps - they work on all watch sizes
3. **Use hardware buttons** for navigation - they're size-independent
4. **Calculate coordinates dynamically** when labels aren't available
5. **Never hardcode coordinates** - they break on different watch sizes
