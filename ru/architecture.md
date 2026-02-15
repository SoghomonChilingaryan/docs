# Правила архитектуры (обязательно)

## Структура папок (каноничная)

```
lib/
  main.dart
  main_dev.dart            # optional
  main_prod.dart           # optional
  src/
    common/
      app/                 # entrypoints, app bootstrap
      config/              # AppConfig, env, remote config adapters
      di/                  # initDI / registrations (single entry)
      functions/           # утилиты
      interceptors/        # сетевые перехватчики
      logging/             # глобальный логгер / аналитика
      models/              # общие типы данных и AppFailure
      theme/               # legacy + rebrand theme system
      widgets/             # общие UI-компоненты
      http/                # (опционально) API client / http helpers
      storage/             # (опционально) SharedPrefs / Hive wrappers
      platform/            # (опционально) permissions / connectivity / system info
    navigation/
      helpers/             # observers, helpers
      navigator/           # wrappers/extensions/mixins
      router/              # routes, guards, AutoRoute setup
      widgets/             # BottomNavigation и навигационные виджеты
    features/
      <feature_name>/
        domain/
          entities/
          repositories/    # interfaces only
          usecases/
          models/
        data/
          datasources/
          repositories/    # implementations
          mappers/
        bloc/              # presentation logic (BLoC)
        screens/
        widgets/
```

## Направление зависимостей (строго)

- `presentation` (UI/BLoC) -> `domain` -> `data`
- `domain` не импортирует Flutter, SDK, Dio, storage, remote config.
- `data` может зависеть от внешних библиотек (Dio, Firebase, Hive и т.д.).

## Доступ к внешним данным (строго)

Любой доступ к внешним данным должен идти только через интерфейс репозитория в `domain`.

К внешним данным относятся:
- HTTP / API
- Local storage (SharedPrefs, Hive, Files, DB)
- Remote config
- Analytics / logging SDKs
- Push / notifications
- Platform services (permissions, connectivity, device info)

Запрещено:
- Вызывать API/storage/SDK напрямую из UI
- Хранить в BLoC ссылку на datasource

Правильный поток:
- UI -> BLoC -> UseCase (опционально) -> Domain Repository (interface) -> Data RepositoryImpl -> DataSource

## DI (целевой стандарт)

- DI централизован в `src/common/di`.
- Используем `get_it` (один контейнер).
- Никаких глобальных синглтонов вне DI.
- Регистрации группируются по слоям: common сначала, затем фичи.

Правило важнее конкретной DI-библиотеки: код зависит от абстракций и связывается через DI.

## Навигация (целевой стандарт)

- Используем `auto_route`.
- Все маршруты и guards живут в `src/navigation/router`.
- Экраны фич не создают свои роутеры.

## Обработка ошибок

- Всегда `await` для async.
- Внешние вызовы оборачиваем в `try/catch`.
- Ошибки маппим в `AppFailure` (domain-friendly).

## Нейминг

- папки/файлы: `snake_case`
- классы: `PascalCase`
- приватные члены: `_underscore`
- события/состояния: `<Feature>Event`, `<Feature>State`
- избегать сокращений в названиях
