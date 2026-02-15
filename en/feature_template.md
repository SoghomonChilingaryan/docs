# Feature template

Use this template to create `lib/src/features/<feature_name>` safely.

## When to create a feature module

Create a feature when:
- it has its own state and business rules
- it has its own data sources/repositories
- it can be developed/tested in isolation

## Structure

Add subfolders only when they make sense.

```text
lib/src/features/<feature_name>/
  domain/
    entities/             # optional (pure Dart)
    repositories/         # repository contracts (interfaces)
    usecases/             # optional orchestration
  data/
    repositories/         # implementations (RepositoryImpl)
    datasources/          # optional
    mappers/              # optional (DTO <-> domain mapping)
  bloc/
    <feature_name>_bloc.dart
    <feature_name>_event.dart
    <feature_name>_state.dart
  screens/
  widgets/
```

If the feature does not access the external world (no HTTP/storage/SDK/platform/remote config/analytics), you may start with:

- `bloc/ + screens/ + widgets/`

If it does access the external world, `domain/repositories` contract is mandatory and BLoC must depend only on the contract.

## Layer responsibilities

- `domain/entities`: pure Dart domain objects owned by this feature
- `domain/repositories`: contracts (no Flutter/SDK imports)
- `domain/usecases`: optional orchestration
- `data/repositories`: implementations and external calls + mapping into domain-friendly types
- `bloc`: presentation logic (Flutter-free)
- `screens`: route destinations
- `widgets`: reusable widgets inside the feature

## BLoC rules (strict)

- exactly 3 files in `bloc/`: bloc/event/state
- bloc/event/state are Flutter-free:
  - no `package:flutter/...` imports
  - allowed: `package:bloc/bloc.dart`, `package:equatable/equatable.dart`, etc.
- no `BuildContext`, `Navigator`, `Theme`, `MediaQuery` inside BLoC/state
- external access only via repository contract (`domain/repositories`)
- `state` is Flutter-free (no `Color`, `TextStyle`, `Widget`, etc.)

Event handling recommendation:

- usually keep a single `on<FeatureEvent>` and route events via `switch (event) { ... }`
- for special concurrency policies (restartable/droppable/etc.), use a dedicated `on<SpecificEvent>` intentionally

## Repository recommendations

- keep a minimal public API (only what BLoC needs)
- keep dependencies as private `final` fields
- map external errors to `AppFailure` (when your project uses it)

Small settings storage:

- for small flags/theme/locale, use `AppPreferencesStorage` from `lib/src/common/storage`
- concrete storage implementation is hidden behind DI
- BLoC must not import concrete storage

## UI rules

- `screens/` are pages (routes)
- `widgets/` are internal reusable UI parts of the feature
- prefer composition by named widgets over `_buildHeader()`-style functions

## Cross-feature imports

- feature A must not import internals of feature B (`data/`, `bloc/`, `screens/`, `widgets/`)
- move shared parts to `common/*` or depend on a contract and wire via DI

