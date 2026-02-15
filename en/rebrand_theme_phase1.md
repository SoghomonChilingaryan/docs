# Rebrand Theme (Phase 1)

Phase 1 goal: start a theme rebrand safely without breaking business logic and without rewriting the whole UI at once.

## Principles

- Theme is owned centrally in `lib/src/common/theme/`.
- Feature logic must not depend on theme: only change presentation (screens/widgets) and shared UI components.
- Migrate screen by screen: mixed old/new visuals are acceptable, business logic changes are not.

## Baseline plan (screen-by-screen)

1. Add/update `AppTheme` (light/dark) and define core tokens (surface/outline/primary).
2. Wire theme selection during bootstrap via `SettingsBloc` (see `lib/src/common/core/app/app.dart`).
3. Pick 1 screen (e.g. Player) and apply the rebrand style:
   - do not change button count or actions
   - change only sizes/colors/spacing/shapes/typography
   - keep feature boundaries (do not import other feature internals)
4. Extract repeated UI patterns into `lib/src/common/widgets/` (buttons, cards, pills).

## Definition of done (Phase 1)

- A stable rebrand dark theme exists and is used on at least one screen.
- Settings can switch theme/accent if needed.
- Changes do not touch domain/data, UI only.

