---
name: flutter-conventions
description: Flutter and Dart coding conventions for Rounds Tracker
globs:
  - "**/*.dart"
---

# Flutter/Dart Conventions

## Code Style

- Use `flutter_lints` rules (see analysis_options.yaml)
- Prefer `const` constructors where possible
- Use named parameters for constructors with 3+ parameters

## State Management

Use Provider pattern consistently:
```dart
// Access providers via context
final provider = Provider.of<ProfileProvider>(context, listen: false);

// Or use Consumer for reactive UI
Consumer<ProfileProvider>(
  builder: (context, provider, child) => ...
)
```

## Model Pattern

Models should:
- Have `toJson()` and `fromJson()` factory methods
- Include `clone()` method for copying
- Use nullable fields with defaults where appropriate

```dart
class Model {
  final int? id;
  final String name;
  
  Model({this.id, required this.name});
  
  Map<String, dynamic> toJson() => {'id': id, 'name': name};
  
  factory Model.fromJson(Map<String, dynamic> json) =>
    Model(id: json['id'], name: json['name']);
}
```

## Imports

Order imports:
1. Dart SDK (`dart:`)
2. Flutter (`package:flutter/`)
3. External packages (`package:`)
4. Project imports (relative)

## Async Patterns

- Use `async`/`await` over raw Futures (not `.then()`)
- Handle errors with try/catch in providers
- Initialize async resources in `initState` or dedicated init methods

### Anti-Patterns to Avoid

**Never call async methods from constructors:**
```dart
// ❌ BAD: Race condition - async not awaited
class BadManager {
  BadManager() {
    initialize();  // Async method called without await
  }
  Future<void> initialize() async { ... }
}

// Also bad: calling initialize() in constructor AND awaiting it elsewhere
final manager = BadManager();  // initialize() starts here
await manager.initialize();     // May run twice!

// ✅ GOOD: Explicit initialization only
class GoodManager {
  bool _initialized = false;

  Future<void> initialize() async {
    if (_initialized) return;  // Idempotent
    _initialized = true;
    // ... initialization logic
  }
}

// Usage:
final manager = GoodManager();
await manager.initialize();  // Only place it's called
```

**Use Completer for single-execution guarantees:**
```dart
class SingleInitManager {
  Completer<void>? _initCompleter;

  Future<void> initialize() {
    _initCompleter ??= Completer<void>()..complete(_doInit());
    return _initCompleter!.future;
  }

  Future<void> _doInit() async {
    // Actual initialization - only runs once
  }
}
```

**Consistent error handling:**
```dart
// ✅ GOOD: try/catch with async/await
Future<void> loadData() async {
  try {
    final data = await fetchData();
    setState(() => _data = data);
  } catch (e) {
    setState(() => _error = e.toString());
  }
}

// ❌ BAD: Mixed patterns make errors hard to trace
Future<void> loadData() {
  return fetchData()
    .then((data) => setState(() => _data = data))
    .catchError((e) => print(e));  // Silent failure
}
```

## Widget Organization

- Keep widgets focused and small
- Extract reusable widgets to `screens/widgets/`
- Use `const` for static widgets
- Prefer composition over inheritance

## Accessibility

Add semantic labels to interactive widgets for testing and accessibility.
See `flutter-accessibility` rule for detailed guidance.

Key points:
- Use `tooltip` on `IconButton`
- Wrap tappable areas with `Semantics(label:, button: true)`
- `InputDecoration.labelText` provides text field accessibility

## Flutter Migration Notes

### AssetManifest.bin (Flutter 3.7+)

**Breaking change:** Flutter 3.7+ uses `AssetManifest.bin` instead of `AssetManifest.json`.

```dart
// ❌ OLD (fails on Flutter 3.7+):
final manifest = await rootBundle.loadString('AssetManifest.json');
final Map<String, dynamic> assets = json.decode(manifest);

// ✅ NEW: Use AssetManifest API (correct approach)
import 'package:flutter/services.dart' show AssetManifest, rootBundle;

final AssetManifest assetManifest =
    await AssetManifest.loadFromAssetBundle(rootBundle);
final allAssets = assetManifest.listAssets();

// Filter by directory
final profileAssets = allAssets
    .where((key) => key.startsWith('assets/roundProfiles/') && key.endsWith('.json'))
    .toList();
```

**Symptoms of this issue:**
- Asset discovery returns empty list
- Bundled resources (profiles, configs) don't load
- App works in debug but not release builds
- Works on older Flutter versions but fails on newer

### Dispose Order for Animations

Always dispose child animations before parent controllers:

```dart
// ✅ CORRECT order
@override
void dispose() {
  _curvedAnimation.dispose();    // Child first
  _animationController.dispose(); // Parent second
  super.dispose();
}

// ❌ WRONG order - may cause errors
@override
void dispose() {
  _animationController.dispose(); // Parent disposed
  _curvedAnimation.dispose();     // Child references disposed parent!
  super.dispose();
}
```

### Timer Cleanup

Cancel timers before any other cleanup in dispose:

```dart
@override
void dispose() {
  _timer?.cancel();  // Cancel FIRST
  _timer = null;
  // ... other cleanup
  super.dispose();
}
```
