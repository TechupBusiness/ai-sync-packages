---
name: flutter-ux-design
description: UX design principles for Flutter - responsive layouts, design tokens, adaptive spacing
globs:
  - "lib/screens/**/*.dart"
  - "lib/widgets/**/*.dart"
always_apply: false
---

# Flutter UX Design Principles

Design guidelines for creating harmonious, responsive, and user-friendly interfaces across phones, tablets, and foldables.

## Responsive Layouts

### Always Scrollable Content

Screens with dynamic content must be scrollable to work on all device sizes:

```dart
// ✓ Good: Content scrolls when needed
Scaffold(
  body: SafeArea(
    child: SingleChildScrollView(
      padding: const EdgeInsets.all(16),
      child: Column(children: [...]),
    ),
  ),
)

// ✗ Bad: Fixed layout that can overflow
Scaffold(
  body: SafeArea(
    child: Column(children: [...]),  // May overflow on small screens
  ),
)
```

### Avoid Spacer in Scrollable Layouts

`Spacer` widgets require bounded constraints and don't work in scrollable containers:

```dart
// ✗ Bad: Spacer causes errors in ScrollView
SingleChildScrollView(
  child: Column(
    children: [
      Header(),
      Spacer(),  // Error: unbounded height
      Footer(),
    ],
  ),
)

// ✓ Good: Use fixed spacing with design tokens
SingleChildScrollView(
  child: Column(
    children: [
      Header(),
      const SizedBox(height: AppSpacing.xl),
      Footer(),
    ],
  ),
)
```

### Platform-Specific Content

Use `dart:io` Platform checks for platform-specific text/features:

```dart
import 'dart:io' show Platform;

String get storeName {
  if (Platform.isIOS || Platform.isMacOS) return 'App Store';
  if (Platform.isAndroid) return 'Google Play';
  return 'your app store';
}
```

## Modern Spacing System (Design Tokens)

Use a **4-8dp baseline grid** with semantic design tokens. This approach is recommended by Apple HIG and Material Design 3 for adaptive layouts across phones, tablets, and foldables.

### Spacing Scale

| Token | Value | Use Case |
|-------|-------|----------|
| `xxs` | 4dp | Micro spacing, icon padding |
| `xs` | 8dp | Tight spacing, inline elements |
| `sm` | 12dp | Between related items |
| `md` | 16dp | Standard gaps, card padding |
| `lg` | 24dp | Section gaps |
| `xl` | 32dp | Major section breaks |
| `xxl` | 48dp | Hero spacing, screen margins |

### AppSpacing Constants

```dart
/// Design tokens for consistent spacing across the app
abstract class AppSpacing {
  static const double xxs = 4;
  static const double xs = 8;
  static const double sm = 12;
  static const double md = 16;
  static const double lg = 24;
  static const double xl = 32;
  static const double xxl = 48;
}

// Usage in layouts
Column(
  children: [
    const SizedBox(height: AppSpacing.xs),   // 8: After icon
    Text('Headline'),
    const SizedBox(height: AppSpacing.xxs),  // 4: Tight: headline to subhead
    Text('Subheadline'),
    const SizedBox(height: AppSpacing.md),   // 16: Section gap
    FeaturesList(),
    const SizedBox(height: AppSpacing.lg),   // 24: Major break before CTA
    ActionButton(),
  ],
)
```

## Responsive Breakpoints

### Breakpoint Constants

Define breakpoints centrally to avoid magic numbers:

```dart
/// Breakpoints for responsive layouts
abstract class Breakpoints {
  static const double mobile = 600;    // Phones
  static const double tablet = 900;    // Tablets, foldables unfolded
  static const double desktop = 1200;  // Large tablets, desktop

  static bool isMobile(BuildContext context) =>
    MediaQuery.sizeOf(context).width < mobile;

  static bool isTablet(BuildContext context) {
    final width = MediaQuery.sizeOf(context).width;
    return width >= mobile && width < desktop;
  }

  static bool isDesktop(BuildContext context) =>
    MediaQuery.sizeOf(context).width >= desktop;
}
```

### MediaQuery vs LayoutBuilder

Use **MediaQuery** for global layout mode decisions, **LayoutBuilder** for widget-level adaptation:

```dart
// ✓ Good: Use MediaQuery.sizeOf() for performance (only rebuilds on size change)
final width = MediaQuery.sizeOf(context).width;

// ✗ Avoid: MediaQuery.of() rebuilds on ANY MediaQuery change
final width = MediaQuery.of(context).size.width;

// ✓ Good: LayoutBuilder for component-level responsiveness
LayoutBuilder(
  builder: (context, constraints) {
    final columns = constraints.maxWidth > 800 ? 4 :
                    constraints.maxWidth > 600 ? 3 : 2;
    return GridView.count(crossAxisCount: columns, children: items);
  },
)
```

### Adaptive Grid Pattern

```dart
class AdaptiveGrid extends StatelessWidget {
  final List<Widget> items;

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        int columns;
        double spacing;

        if (constraints.maxWidth >= 1200) {
          columns = 4;
          spacing = AppSpacing.lg;
        } else if (constraints.maxWidth >= 600) {
          columns = 3;
          spacing = AppSpacing.md;
        } else {
          columns = 2;
          spacing = AppSpacing.sm;
        }

        return GridView.builder(
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: columns,
            mainAxisSpacing: spacing,
            crossAxisSpacing: spacing,
          ),
          itemCount: items.length,
          itemBuilder: (context, index) => items[index],
        );
      },
    );
  }
}
```

### Master-Detail Pattern (Tablets/Foldables)

```dart
class MasterDetailLayout extends StatelessWidget {
  final Widget master;
  final Widget detail;

  @override
  Widget build(BuildContext context) {
    final isWide = MediaQuery.sizeOf(context).width > 720;

    if (isWide) {
      return Row(
        children: [
          Expanded(flex: 1, child: master),
          const VerticalDivider(width: 1),
          Expanded(flex: 2, child: detail),
        ],
      );
    }
    return master; // Navigate to detail on tap
  }
}
```

## Visual Hierarchy Guidelines

### Spacing Relationships

- **Related items**: Use `xs` (8dp) - e.g., icon and label
- **Grouped content**: Use `sm` (12dp) - e.g., list items
- **Sections**: Use `md` (16dp) - e.g., after a heading
- **Major sections**: Use `lg` (24dp) - e.g., between features and CTA
- **Screen edges**: Use `md` (16dp) for horizontal padding

### Example: Paywall Layout

```dart
Column(
  children: [
    const SizedBox(height: AppSpacing.xs),   // 8: After safe area
    PremiumIcon(),
    const SizedBox(height: AppSpacing.lg),   // 24: Icon to headline
    Headline(),
    const SizedBox(height: AppSpacing.xs),   // 8: Headline to subhead
    Subheadline(),
    const SizedBox(height: AppSpacing.md),   // 16: Intro to features
    FeaturesList(),
    const SizedBox(height: AppSpacing.lg),   // 24: Features to pricing
    PricingOptions(),
    const SizedBox(height: AppSpacing.sm),   // 12: Pricing to CTA
    PurchaseButton(),
    const SizedBox(height: AppSpacing.sm),   // 12: CTA to secondary
    RestoreButton(),
    const SizedBox(height: AppSpacing.xs),   // 8: Secondary to legal
    LegalText(),
    const SizedBox(height: AppSpacing.sm),   // 12: Bottom padding
  ],
)
```

## Overflow Prevention Checklist

Before completing any screen UI:

1. **Test on smallest supported device** (iPhone SE / small Android)
2. **Check landscape orientation** if supported
3. **Verify with large text** (accessibility settings)
4. **Look for yellow/black overflow indicators**
5. **Ensure legal/footer text is fully visible**
6. **Test on tablet/foldable if supporting those form factors**

## Touch Target Sizes

See [flutter-accessibility.md](flutter-accessibility.md) for touch target guidelines.

Key points:
- Minimum: 44x44dp (WCAG)
- Recommended: 48x48dp (Material)
- Spacing between targets: 8dp minimum
