# Rebrand PHASE 1: theme system (start here)

## Goal

Keep the current theme as `legacy`, add a new theme as `rebrand`, and migrate screen-by-screen without breaking business logic.

## Rules

- Do not delete or rewrite the legacy `ThemeData`.
- New UI components should read values from theme/tokens.
- No hardcoded colors in new code (except token definitions).

## Recommended implementation

### 1) Introduce theme variants

- `legacy`: current `ThemeData`
- `rebrand`: new `ThemeData`

Switching should be possible without changing code:
- e.g. `--dart-define=USE_REBRAND_THEME=true`

### 2) Add design tokens for rebrand

Prefer `ThemeExtension` with:
- colors (background/surface/border/primary/text)
- typography (title/body/label)
- radii, spacing, shadows

### 3) Build Rebrand `ThemeData`

Compose from:
- `ColorScheme`
- `TextTheme`
- component themes (buttons, inputs, appbar)
- attach tokens via `ThemeData.extensions`

### 4) Make migration safe

During migration:
- old screens keep using legacy styles
- new screens/components use rebrand tokens

Two safe approaches:
- global toggle (rebrand on/off for whole app)
- per-screen toggle (wrap screen with `Theme(data: rebrandTheme, child: ...)`)

## Result of PHASE 1

- tokens module exists
- Rebrand `ThemeData` exists
- entrypoint can select legacy/rebrand
- business logic is unchanged

