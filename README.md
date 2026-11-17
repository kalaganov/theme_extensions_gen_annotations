# theme_extensions_gen_annotations

This package defines annotations used with `theme_extensions_gen` to generate strongly-typed `ThemeExtension` classes and tooling for Flutter theming.

---

## Purpose

Use these annotations to eliminate boilerplate in theme creation. Combined with `build_runner` and `theme_extensions_gen`, this package enables:

- Declarative theme structure via `@ThemeExtensionTemplate`
- Centralized and typed theme values via `@ThemeExtensionImpl`
- Optional build-time generation of `BuildContext` extensions

---

## Annotations

### `@ThemeExtensionTemplate()`

Marks an abstract interface as a template for a `ThemeExtension`. The generator creates:

- `copyWith`, `lerp`, `==`, `hashCode` and the `type` getter
- `debugFillProperties` if the interface uses `Diagnosticable`
- A concrete class with a factory constructor and mixin
- A `BuildContext` extension with getters for all extensions

**Example:**

```dart
import 'package:flutter/foundation.dart';
import 'package:material_ui/material_ui.dart';
import 'package:theme_extensions_gen_annotations/theme_extensions_gen_annotations.dart';

part 'themed_button_theme.g.dart';

@ThemeExtensionTemplate()
abstract interface class ThemedButtonTheme
    extends ThemeExtension<ThemedButtonTheme>
    with _$ThemedButtonThemeMixin, Diagnosticable {
  const factory({
    required Decoration decoration,
    required TextStyle textStyle,
    required EdgeInsets padding,
  }) = _$ThemedButtonTheme;
}
```

The classic `const factory ThemedButtonTheme({...})` works as well; the concise
`const factory({...})` needs Dart 3.13 and theme_extensions_gen 1.3.0 or later.

By default, the generator creates `lib/generated/theme_extensions/context_extensions.dart`.  
This can be customized or disabled via `build.yaml`:

```yaml
targets:
  $default:
    builders:
      theme_extensions_gen:contextExtensionsGenerator:
        enabled: false
```

Or with a custom path, relative to `lib/`:

```yaml
targets:
  $default:
    builders:
      theme_extensions_gen:contextExtensionsGenerator:
        options:
          output_path: "custom/context_extensions.dart"
```

---

### `@ThemeExtensionImpl({ String? group })`

Marks a top-level function, getter or variable that returns a `ThemeExtension`
or a list of them, to be combined into a generated list.  
Used like this:

```dart
@ThemeExtensionImpl()
List<ThemeExtension> get someFeatureThemeExtensions => const [
  ThemedCardTheme(/*...*/),
  ThemedButtonTheme(/*...*/),
];

@ThemeExtensionImpl(group: 'dark')
List<ThemeExtension<dynamic>> someFeatureDarkThemeExtensions() => const [
  ThemedCardTheme(/*...*/),
  ThemedButtonTheme(/*...*/),
];
```

The generator then combines all matching lists into:

- A default file with a `List<ThemeExtension> get themeExtensions` that holds
  `...someFeatureThemeExtensions` and the lists of other features.

- A group-specific file (e.g. for `"dark"`) with
  `List<ThemeExtension> get themeExtensionsDark` that holds
  `...someFeatureDarkThemeExtensions()`.

Configuration in `build.yaml`:

```yaml
targets:
  $default:
    builders:
      theme_extensions_gen:themeExtensionsImplCombiner:
        options:
          default_output:
            path: "generated/theme_extensions/theme_extensions.dart"
            list_name: "themeExtensions"
          output_groups:
            dark:
              path: "generated/theme_extensions/theme_extensions_dark.dart"
              list_name: "themeExtensionsDark"
```

Paths are relative to `lib/`. See the
[theme_extensions_gen README](https://pub.dev/packages/theme_extensions_gen) for all options.
