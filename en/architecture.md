# Architecture rules (mandatory)

These rules describe the target structure. Create folders only when needed, but keep boundaries strict.

## Three zones (concept)

We split the app into three independent zones so issues can be located fast:

1) `common/` — app core + everything shared by 2+ features
- bootstrap/composition root: startup, DI, config, theme, SDK initialization
- shared infra: storage/http/platform/remote_config
- shared widgets + utilities

2) `navigation/` — navigation as a separate area
- contracts (e.g. `AppNavigator`) + adapters for chosen routing engine
- router setup/guards + navigation widgets (bottom navigation, etc.)

3) `features/` — business modules
- each feature is isolated and evolves independently

## Canonical folder structure

```text
packages/                 # (recommended) external integrations/SDK wrappers (API, DB, analytics, etc.)
  <package_name>/

lib/
  main.dart
  main_dev.dart            # optional
  main_prod.dart           # optional
  src/
    common/
      core/
        app/               # entrypoints, app bootstrap (composition root)
        config/            # AppConfig, env, remote config adapters
        di/                # DI container + registrations
      functions/           # utilities
      interceptors/        # network interceptors (infra)
      logging/             # ILogger/IAnalytics + implementations
      models/              # shared types and AppFailure
      domain/              # (recommended) shared contracts + models for external integrations
        api/
        models/
      theme/               # legacy + rebrand theme system
      widgets/             # shared UI components
      http/                # (optional) http helpers
      storage/             # key-value wrappers (e.g. AppPreferencesStorage)
      platform/            # permissions/connectivity/system info
    navigation/
      helpers/
      navigator/
      router/
      widgets/
    features/
      <feature_name>/
        domain/
          entities/
          repositories/
          usecases/         # optional
        data/
          repositories/
          datasources/      # optional
          mappers/          # optional
        bloc/
        screens/
        widgets/
```

## Bootstrap order

Core idea: build dependencies once (composition root) before building the full UI.

Recommended flow:

- `main.dart` -> `runThisApp(AppConfig)`
- `App` shows Splash (minimal UI) and builds DI asynchronously (`diSetup`)
- after DI:
  - create global blocs (e.g. `SettingsBloc` reads theme/locale from repository)
  - build `MaterialApp` with correct `themeMode/locale`

Important: Splash may live before `MaterialApp`, so it must be self-sufficient (e.g. provide `Directionality`).

## Dependency direction (strict)

- `presentation` (UI/BLoC) -> `domain` -> `data`
- `domain` must not import Flutter/SDK/Dio/storage/remote config/etc.
- `data` may depend on external libraries (Dio/Firebase/Hive/etc.)

Feature isolation:

- a feature must not import internals of another feature (especially `data/`, `bloc/`, `screens/`, `widgets/`)
- if feature A needs something from feature B: move it to `common/*` or depend on a contract and wire via DI

## External data access (strict)

Any external data access must go through a repository interface in `domain`.

External data includes:
- HTTP / API
- Local storage (SharedPrefs/Hive/Files/DB)
- Remote config
- Analytics / logging SDKs
- Push / notifications
- Platform services (permissions/connectivity/device info)

Forbidden:
- calling API/storage/SDK directly from UI
- keeping a datasource reference inside BLoC

Correct flow:
- UI -> BLoC -> UseCase (optional) -> Domain Repository (interface) -> Data RepositoryImpl -> DataSource

### Allowance (small data)

- Repositories in `data/` may store small flags/settings using a minimal key-value abstraction like `AppPreferencesStorage` (from `lib/src/common/storage`).
- Concrete implementation (SharedPreferences/etc.) is an infrastructure detail and must not leak into UI/BLoC.

Important (duplicates/naming):

- If the project already has a shared key-value storage (`PreferencesStorage`, `KeyValueStorage`, etc.), do not create another one.
- Naming is not important; boundaries are: thin KV abstraction, DI-wired, not referenced from UI/BLoC directly.

## DI (current standard)

- DI is centralized in `lib/src/common/core/di`
- container: `DiContainer` (custom)
- composition: `diSetup(AppConfig)` returns a container
- GDID scopes are supported (container may have a parent)

Recommended registration layout:

- `di_setup.dart`: shared app dependencies (config, logger, storage, repositories)
- `api_setups.dart`: dedicated module for server-side setup (http client + interceptors + API clients)

## Navigation (current standard)

- UI depends on `AppNavigator` contract (interface)
- concrete navigation engine (Navigator/go_router/auto_route) must not leak into features

## Error handling

- always `await` async calls
- wrap external calls with `try/catch`
- map errors into `AppFailure` (domain-friendly)

## Logging and analytics (recommended)

- logging: `lib/src/common/logging/logger.dart` (`ILogger`)
- analytics: `lib/src/common/logging/analytics.dart` (`IAnalytics`)
- default implementation: `lib/src/common/logging/app_logger.dart` (`AppLogger`)
- BLoC observer: `lib/src/common/logging/app_bloc_observer.dart` (`AppBlocObserver`) — wired during DI setup (`diSetup`)

SDK integrations (Crashlytics/Sentry/Firebase/etc.) should be hidden behind DI and may live in `packages/*`.

## External integrations and big data (recommended)

If the project has a large server API (many endpoints/DTOs) or big local storage (DB/cache/files/complex objects), recommend:

- implement the integration in `packages/*`
- keep only contracts in `lib/src/common/domain` (e.g. `domain/api` + `domain/models`)
- wire implementations via DI (`common/core/di`)

### External data package structure (template)

Minimal recommended template for `packages/<package_name>/lib`:

- `api/` — external clients (HTTP/SDK), may be split by domains/endpoints
- `dto/` — external DTOs (request/response), may be grouped into subfolders
- `<package_name>.dart` — barrel export for a single import

Important:

- UI/BLoC/domain should not import these packages directly
- the app should see only contracts in `common/domain/*`, implementations are wired via DI

## Naming

- folders/files: `snake_case`
- classes: `PascalCase`
- private members: `_underscore`
- events/states: `<Feature>Event`, `<Feature>State`
- avoid abbreviations
