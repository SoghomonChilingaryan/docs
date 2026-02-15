# Правила архитектуры (обязательно)

Эта папочная структура и правила описывают целевую модель. Папки добавляются по мере необходимости: если какого-то слоя/подпапки еще нет, это нормально, пока соблюдаются границы.

## Три зоны приложения (концепт)

Проект делится на 3 независимые зоны, чтобы проблему всегда можно было быстро локализовать:

1) `common/` — ядро приложения и всё общее для 2+ фич
- bootstrap/composition root: запуск, DI, конфиг, тема, подключение SDK
- shared infra: storage/http/platform/remote_config
- shared UI/widgets + утилиты

2) `navigation/` — навигация как отдельная область
- контракты (например, `AppNavigator`) и адаптеры под выбранный движок
- router setup/guards + навигационные виджеты (bottom navigation и т.п.)

3) `features/` — бизнес-модули
- каждая фича изолирована и развивается независимо

## Структура папок (каноничная)

```
packages/                 # (рекомендуемо) внешние интеграции/SDK wrappers (API, DB, analytics, etc.)
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
      functions/           # утилиты
      interceptors/        # сетевые перехватчики (infra)
      logging/             # ILogger/IAnalytics + реализации
      models/              # общие типы данных и AppFailure
      domain/              # (рекомендуемо) общие контракты + модели для внешних интеграций
        api/
        models/
      theme/               # legacy + rebrand theme system
      widgets/             # общие UI-компоненты
      http/                # (опционально) http helpers
      storage/             # key-value wrappers (например, AppPreferencesStorage)
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
          usecases/         # (опционально)
        data/
          repositories/
          datasources/      # (опционально)
          mappers/          # (опционально)
        bloc/
        screens/
        widgets/
```

## Порядок запуска (bootstrap)

Базовая идея: зависимости собираем в одном месте (composition root) и делаем это до того, как начинаем строить полноценный UI.

Рекомендуемый поток:

- `main.dart` -> `runThisApp(AppConfig)`
- `App` показывает Splash (минимальный UI) и асинхронно собирает DI (`diSetup`)
- после DI:
  - создаем глобальные блоки (например, `SettingsBloc`, который читает theme/locale из репозитория)
  - строим `MaterialApp` с корректными `themeMode/locale`

Важно: Splash может жить до `MaterialApp`, поэтому он должен быть максимально “самодостаточным” (например, иметь `Directionality`), не требуя темы/локализации.

## Направление зависимостей (строго)

- `presentation` (UI/BLoC) -> `domain` -> `data`
- `domain` не импортирует Flutter, SDK, Dio, storage, remote config.
- `data` может зависеть от внешних библиотек (Dio, Firebase, Hive и т.д.).

Изоляция фич:

- Фича не импортирует внутренности другой фичи (особенно `data/`, `bloc/`, `screens/`, `widgets/`).
- Если нужна зависимость между фичами: выносим общее в `lib/src/common/*` или делаем зависимость от контракта (абстракции), а связывание выполняем в DI.

## Правила UI / presentation (обязательно)

- `screens/` и `widgets/` это только presentation: строим UI, реагируем на state и диспатчим события.
- В виджетах не должно быть бизнес-логики и data-логики:
  - никаких вызовов репозиториев/файлов/SDK
  - никаких циклов/вычислений "подготовки данных", чтобы что-то заработало (prefetch, кэш, вычисление доменных значений)
  - никакой orchestration, которая по смыслу относится к BLoC или репозиториям
- Допустимо в UI:
  - навигация, `showDialog`/bottom sheet/snackbar
  - UI-only state (controllers, animations)
  - небольшое форматирование для отображения (например, `Duration -> mm:ss`)
- Если логика кнопки/действия не тривиальная: выносим в отдельный виджет и/или переносим orchestration в BLoC.

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

### Допущение (про маленькие данные)

- Репозитории в `data/` могут хранить небольшие настройки/флаги через минимальный key-value интерфейс `AppPreferencesStorage` (из `lib/src/common/storage`).
- Конкретная реализация (например, SharedPreferences) — деталь инфраструктуры и не должна “протекать” в BLoC/UI.

Важно (про дубли и нейминг):

- Если в проекте уже есть общий key-value storage (например, `PreferencesStorage`, `KeyValueStorage`, `AppStorage`), не создаем второй. Используем существующий.
- Название не принципиально: важны роль (тонкая KV-абстракция) и границы (подключается через DI, не протекает в UI/BLoC).

## DI (текущий стандарт)

- DI централизован в `lib/src/common/core/di`.
- Контейнер: `DiContainer` (кастомный, без `get_it`).
- Сборка графа: `diSetup(AppConfig)` возвращает контейнер.
- Поддерживаются scopes через GDID (контейнер может иметь parent).

Рекомендации по структуре регистраций:

- `di_setup.dart`: общие зависимости приложения (config, logger, storage, repositories).
- `api_setups.dart`: отдельный файл/модуль, где собирается серверная часть (http client + interceptors + API clients).

## Навигация (текущий стандарт)

- UI зависит от контракта `AppNavigator` (интерфейс).
- Конкретная реализация навигации (Navigator 1/2, `go_router`, `auto_route`) должна жить в bootstrap/app слое и не "протекать" в фичи.

## Обработка ошибок

- Всегда `await` для async.
- Внешние вызовы оборачиваем в `try/catch`.
- Ошибки маппим в `AppFailure` (domain-friendly).

## Логирование и аналитика (рекомендуемо)

- логирование: `lib/src/common/logging/logger.dart` (`ILogger`)
- аналитика: `lib/src/common/logging/analytics.dart` (`IAnalytics`)
- реализация по умолчанию: `lib/src/common/logging/app_logger.dart` (`AppLogger`)
- BLoC observer: `lib/src/common/logging/app_bloc_observer.dart` (`AppBlocObserver`) — подключается во время сборки DI (`diSetup`)

SDK-интеграции (Crashlytics/Sentry/Firebase/etc.) должны быть скрыты за DI и могут жить в `packages/*`.

### Правило ответственности (обязательно)

- Логирует **только** тот класс/модуль, который **непосредственно делает внешний вызов** (синхронный или асинхронный).
  - Обычно это `data`-слой: `RepositoryImpl` / `DataSource`.
  - Внешние источники: HTTP/API, storage/files/DB, platform services (permissions/device info), SDKs.
- `BLoC` логирует только то, что относится к его ответственности:
  - BLoC events/errors (через `AppBlocObserver`).
  - Редко: внутренние async-операции, которые не являются внешними вызовами (например, orchestration внутри BLoC, если она есть).
- Не дублируем одно и то же логирование в BLoC и репозитории: если ошибка пришла из внешнего вызова, ее логирует репозиторий.

Минимальный стандарт для репозитория:

- внешний вызов оборачиваем в `try/catch`
- в `catch (e, st)` делаем `logger.e(e, st, message: 'Class.method')`

## Внешние интеграции и большие данные (рекомендуемый подход)

Если в проекте есть серверный API с большим количеством запросов/DTO/эндпоинтов или локальное хранилище для больших данных (DB/кэш/файлы/сложные объекты), рекомендуется:

- вынести реализацию во внешние пакеты `packages/*`
- держать в приложении только контракты в `lib/src/common/domain` (например, `domain/api` + `domain/models`)
- связывать реализацию с контрактами через DI (`common/core/di`)

### Структура пакета для внешних данных (шаблон)

Рекомендуемый минимальный шаблон для пакета в `packages/<package_name>/lib`:

- `api/` — клиент(ы) внешнего мира (HTTP/SDK). Здесь же может быть разбиение по областям/эндпоинтам.
- `dto/` — DTO модели внешнего мира (request/response). Допустимы подпапки по доменам/областям.
- `<package_name>.dart` — barrel export: экспортирует публичные классы пакета одним импортом.

Важно:

- Этот пакет не должен импортироваться напрямую из UI/BLoC/domain.
- Приложение видит только контракты в `lib/src/common/domain/api` и `lib/src/common/domain/models`, а реализация подключается через DI.

## Нейминг

- папки/файлы: `snake_case`
- классы: `PascalCase`
- приватные члены: `_underscore`
- события/состояния: `<Feature>Event`, `<Feature>State`
- избегать сокращений в названиях
