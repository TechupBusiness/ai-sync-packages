---
name: flutter-quality-gates
description: Required quality gates and verification steps for all Flutter work
globs:
  - "**/*.dart"
  - "pubspec.yaml"
always_apply: true
---

# Flutter Quality Gates

All Flutter code changes MUST pass these verification steps before completion.

## Required Quality Commands

These commands must pass for EVERY task involving Dart/Flutter code:

```bash
# 1. Static analysis - must report 0 errors
flutter analyze

# 2. Run unit and widget tests - must pass
flutter test

# 3. Run E2E integration tests - must pass
flutter test integration_test/

# 4. Build verification - both must succeed
flutter build ios --debug --no-codesign
flutter build apk --debug
```

## Verification Order

Always run in this order:
1. `flutter analyze` - Catch type errors and lint issues first
2. `flutter test` - Ensure no unit/widget test regressions
3. `flutter test integration_test/` - Ensure E2E flows work correctly
4. Build commands - Verify compilation succeeds

## UI Story Verification

For stories that modify UI or user-facing behavior, ALSO verify visually:

1. Use the `ios-simulator-tests` skill to launch the app on simulator
2. Navigate to the affected screen(s)
3. Capture screenshots to verify the changes
4. Test user interactions (tap, swipe, etc.)
5. Check accessibility with `describe_ui` to verify semantic labels

### iOS Simulator Verification Steps

```
1. mcp__xcodebuildmcp__build_run_sim - Build and launch app
2. mcp__xcodebuildmcp__screenshot - Capture current state
3. mcp__xcodebuildmcp__describe_ui - Get accessibility hierarchy
4. mcp__xcodebuildmcp__tap/swipe/etc - Test interactions
5. Verify expected behavior matches acceptance criteria
```

## Player/Session Story Verification

For stories involving the Player screen, breathing visualizations, or session flows, run these specific E2E tests:

```bash
# Player flow tests (session start to completion)
flutter test integration_test/app_test.dart
flutter test integration_test/player_visual_polish_test.dart
flutter test integration_test/statistics_integration_test.dart
```

### Player-Specific Verification Checklist

- [ ] Session starts correctly with profile data
- [ ] Step transitions work smoothly
- [ ] Round counter increments correctly
- [ ] Timer/counter displays show correct values
- [ ] BreathingCircle animation renders
- [ ] ProgressRing updates with session progress
- [ ] Session completion screen displays results
- [ ] Results are saved to database (check via Statistics screen)

## When to Skip

- **Never skip** `flutter analyze`
- **Unit test skip**: Only if explicitly adding/fixing tests (run specific test file instead)
- **E2E test skip**: Only for pure model/utility changes with no UI impact
- **Build skip**: Only for pure test file changes
- **UI verification skip**: Only for non-UI changes (models, providers, utilities)

## Failure Handling

If any quality gate fails:
1. Fix the issue before proceeding
2. Re-run the full verification sequence
3. Do NOT mark task as complete until all gates pass
