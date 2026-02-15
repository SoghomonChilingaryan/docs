# Rebrand PHASE 1: система темы (начинаем здесь)

## Цель

Сохранить текущую тему как `legacy`, добавить новую как `rebrand`, и мигрировать экран за экраном без поломки бизнес-логики.

## Правила

- Не удалять и не переписывать legacy ThemeData.
- Новые UI-компоненты берут значения из темы/токенов.
- Никаких хардкодов цветов в новом коде (кроме определения токенов).

## Рекомендуемая реализация (целевая)

### 1) Ввести варианты темы

- `legacy`: текущая ThemeData
- `rebrand`: новая ThemeData

Переключение должно быть возможно без изменения кода:
- через `--dart-define=USE_REBRAND_THEME=true`

### 2) Добавить design tokens для rebrand

Создаем объект токенов (желательно `ThemeExtension`):
- colors (background/surface/border/primary/text)
- typography (title/body/label)
- radii, spacing, shadows

### 3) Собрать Rebrand ThemeData

Собираем ThemeData из:
- `ColorScheme`
- `TextTheme`
- component themes (buttons, inputs, appbar)
- attach the tokens via `ThemeData.extensions`

### 4) Сделать миграцию безопасной

Во время миграции:
- old screens keep using legacy styles
- new screens/components use rebrand tokens

Два безопасных подхода:
- global toggle (rebrand on/off for whole app)
- per-screen toggle (wrap screen with `Theme(data: rebrandTheme, child: ...)`)

## Результат PHASE 1

- есть модуль токенов
- есть Rebrand ThemeData
- entrypoint умеет выбирать legacy или rebrand
- бизнес-логика не меняется
