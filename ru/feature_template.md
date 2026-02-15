# Шаблон фичи (копируй и используй)

## Когда создавать модуль фичи

Создавай `src/features/<feature_name>`, когда:
- у UI есть свое состояние и бизнес-правила
- у фичи есть свои источники данных или репозитории
- фичу можно разрабатывать/тестировать изолированно

## Структура

```
src/features/<feature_name>/
  domain/
    entities/
    repositories/
    usecases/
    models/
  data/
    datasources/
    repositories/
    mappers/
  bloc/
    <feature_name>_bloc.dart
    <feature_name>_event.dart
    <feature_name>_state.dart
  screens/
  widgets/
```

## Ответственность по слоям

- `domain/entities`: чистые модели, без JSON и без Flutter.
- `domain/repositories`: только интерфейсы.
- `domain/usecases`: оркестрация, вызывает репозитории, возвращает domain-результаты/ошибки.
- `data/datasources`: вызовы API/storage/SDK, DTO, JSON, platform services.
- `data/repositories`: реализует domain-интерфейсы, маппит DTO <-> domain.
- `bloc`: презентационная логика, использует usecases или репозитории (интерфейсы).
- `screens/widgets`: только Flutter UI, без доступа к данным.

## Чеклист регистрации в DI

В `src/common/di`:
- регистрируем datasources (data слой)
- регистрируем repository implementations (биндим на domain интерфейсы)
- регистрируем usecases
- регистрируем BLoC/factory (presentation)

## Запрет на cross-feature импорты

- Фича не импортирует `data/` другой фичи.
- Если фиче A нужны данные фичи B:
  - описываем интерфейс в `domain/repositories` фичи B
  - фича A зависит от этого интерфейса
  - связываем реализацию через DI
