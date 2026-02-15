# Skeleton: rules and templates (EN)

This folder is the English version of the architecture standard.

## Reading order

1. `architecture.md`: layers, boundaries, DI, navigation, external integrations.
2. `feature_template.md`: how to add a new feature module safely.
3. `rebrand_theme_phase1.md`: strategy for legacy + rebrand themes.

## Goal

- Keep the codebase predictable across projects.
- Make changes localized: API issues -> packages/DI; UI issues -> screens/widgets; state issues -> bloc; design issues -> theme.

