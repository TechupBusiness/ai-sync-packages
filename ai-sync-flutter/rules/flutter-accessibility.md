---
name: flutter-accessibility
description: Flutter accessibility and semantic labels for testability
globs:
  - "**/*.dart"
always_apply: false
---

# Flutter Accessibility & Testing

Add semantic labels to interactive widgets for accessibility and automated UI testing (iOS Simulator MCP, integration tests).

## Why Semantics Matter

1. **Accessibility**: Screen readers use semantic labels
2. **Automated Testing**: XcodeBuildMCP `describe_ui` discovers elements via accessibility tree
3. **Integration Tests**: `find.bySemanticsLabel()` for reliable widget finding

## Adding Semantics to Widgets

### Buttons

```dart
// ✓ Good: Semantic label for testing
ElevatedButton(
  onPressed: onTap,
  child: Semantics(
    label: 'Start breathing session',
    child: const Text('Start'),
  ),
)

// ✓ Good: Using tooltip (often used as accessibility label)
IconButton(
  icon: const Icon(Icons.play_arrow),
  tooltip: 'Play session',  // Used by screen readers
  onPressed: onPlay,
)

// ✗ Bad: No semantic context
ElevatedButton(
  onPressed: onTap,
  child: const Text('Start'),
)
```

### Text Fields

```dart
// ✓ Good: Input decoration label serves as semantic label
FormBuilderTextField(
  name: 'title',
  decoration: const InputDecoration(
    labelText: 'Profile Title',  // Accessible
    hintText: 'Enter a name for your profile',
  ),
)
```

### Cards and Tappable Areas

```dart
// ✓ Good: Wrap with Semantics for complex tappables
Semantics(
  label: 'Box Breathing profile, tap to start',
  button: true,
  child: InkWell(
    onTap: () => navigateToPlayer(profile),
    child: ProfileCard(profile: profile),
  ),
)
```

### Lists

```dart
// ✓ Good: Each list item has semantic identity
ListView.builder(
  itemBuilder: (context, index) {
    final profile = profiles[index];
    return Semantics(
      label: '${profile.title}, ${profile.description}',
      child: ProfileListTile(profile: profile),
    );
  },
)
```

## Key Widgets That Need Semantics

| Widget Type | Recommended Approach |
|-------------|---------------------|
| `ElevatedButton`, `TextButton` | Wrap child with `Semantics(label:)` or text is auto-used |
| `IconButton` | Always add `tooltip` |
| `InkWell`, `GestureDetector` | Wrap with `Semantics(label:, button: true)` |
| `Card` (tappable) | Wrap with `Semantics` describing the action |
| `TextField` | Use `InputDecoration.labelText` |
| Custom widgets | Add `Semantics` at the interaction point |

## Semantic Properties

```dart
Semantics(
  label: 'Profile: Box Breathing',     // Primary description
  hint: 'Double tap to start',         // Action hint
  button: true,                        // Is a button
  enabled: isEnabled,                  // Disabled state
  selected: isSelected,                // Selection state
  child: widget,
)
```

## Testing with Semantics

```dart
// Integration test
await tester.tap(find.bySemanticsLabel('Start breathing session'));

// Find by semantic properties  
find.byWidgetPredicate((widget) => 
  widget is Semantics && widget.properties.label == 'Play');
```

## Excluding Decorative Elements

```dart
// Decorative images shouldn't be announced
Semantics(
  excludeSemantics: true,
  child: Image.asset('decorative_border.png'),
)
```

## Touch Target Size Standards

WCAG 2.5.5 (Level AAA) recommends touch targets be at least 44x44dp. For standalone interactive elements (especially TextButtons outside of dialogs), always set a minimum size.

### Setting Minimum Touch Target Size

```dart
// ✓ Good: Explicit minimum size for standalone TextButton
TextButton(
  onPressed: onSkip,
  style: TextButton.styleFrom(
    minimumSize: const Size(48, 48),  // 48dp for comfortable touch
  ),
  child: const Text('Skip'),
)

// ✓ Good: FilledButton with adequate padding (usually sufficient by default)
FilledButton(
  onPressed: onContinue,
  style: FilledButton.styleFrom(
    padding: const EdgeInsets.symmetric(vertical: 16),  // Creates adequate height
  ),
  child: const Text('Continue'),
)

// ✗ Bad: TextButton without minimum size (may be too small)
TextButton(
  onPressed: onSkip,
  child: const Text('Skip'),
)
```

### When to Set Minimum Size

| Widget Context | Needs Explicit Size? |
|----------------|---------------------|
| `TextButton` in `AlertDialog.actions` | No (Flutter provides adequate area) |
| `TextButton` standalone (e.g., Skip, See all) | **Yes** - add `minimumSize` |
| `IconButton` | No (Flutter ensures 48dp by default) |
| `FilledButton`, `ElevatedButton` | Usually no (padding typically sufficient) |
| `OutlinedButton` | Check padding provides 44dp+ height |

### Key Touch Target Sizes

- **Minimum**: 44x44dp (WCAG 2.5.5 Level AAA)
- **Recommended**: 48x48dp (Material Design guidelines)
- **Spacing**: Ensure at least 8dp between adjacent touch targets

## Priority Areas for This App

Focus on adding semantics to:

1. **Navigation**: Bottom nav items, tab bars
2. **Profile cards**: On home screen and profile list
3. **Player controls**: Play, pause, skip buttons
4. **Form fields**: Already good via InputDecoration
5. **Action buttons**: FABs, save/cancel buttons
6. **Empty states**: "Try again" buttons, CTAs
