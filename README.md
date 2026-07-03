# Skills

Коллекция агентских скиллов для Claude Code, основанная на [mattpocock/skills](https://github.com/mattpocock/skills) (MIT License).

Скиллы лежат в `.claude/skills/` — Claude Code автоматически подхватывает их в любой сессии внутри этого репозитория и делает доступными как слэш-команды (`/grill-me`, `/to-issues` и т.д.). Работают с любой моделью: Opus, Sonnet, Fable.

## Быстрый старт

Перед первым использованием engineering-скиллов запустите один раз:

```
/setup-matt-pocock-skills
```

Он настроит issue tracker (GitHub / GitLab / локальные markdown-файлы), словарь triage-меток и место для доменной документации.

## Основной рабочий цикл

1. **`/grill-me`** — агент задаёт вам детальные вопросы, чтобы согласовать требования до начала работы (для кода лучше `/grill-with-docs`).
2. **`/to-issues`** — разбивает большой план на маленькие независимые задачи (vertical slices / tracer bullets) с критериями приёмки и метками для агента. *Раньше назывался `/to-tickets` — автор переименовал его.*
3. **`/implement`** — агент последовательно берёт задачи в работу.
4. **`/code-review`** — ревью результата.
5. **`/handoff`** — передача контекста в новую сессию, когда текущая переполнилась. *Это тот самый `/hand-off`.*

## Все скиллы

### Engineering

| Скилл | Что делает |
|---|---|
| `/setup-matt-pocock-skills` | Одноразовая настройка репозитория под остальные скиллы |
| `/grill-with-docs` | Согласование требований + работа с доменной документацией |
| `/to-issues` | План → маленькие задачи в трекере (бывший `/to-tickets`) |
| `/to-prd` | Идея → PRD |
| `/implement` | Реализация задачи из трекера |
| `/triage` | Триаж входящих issue через state machine меток |
| `/code-review` | Код-ревью |
| `/tdd` | Разработка через тесты |
| `/diagnosing-bugs` | Систематическая диагностика багов |
| `/research` | Исследование кодовой базы / вопроса |
| `/prototype` | Быстрый прототип (UI или логика) |
| `/codebase-design` | Проектирование ("design it twice") |
| `/domain-modeling` | Доменная модель, CONTEXT.md, ADR |
| `/improve-codebase-architecture` | Аудит и улучшение архитектуры |
| `/resolving-merge-conflicts` | Разрешение merge-конфликтов |
| `/ask-matt` | Совет в стиле философии Matt Pocock |

### Productivity

| Скилл | Что делает |
|---|---|
| `/grill-me` | Согласование требований (не только для кода) |
| `/grilling` | Базовая техника «прожарки» вопросами |
| `/handoff` | Передача контекста в свежую сессию |
| `/teach` | Обучение с трекингом прогресса |
| `/writing-great-skills` | Как писать собственные скиллы |

### Misc

| Скилл | Что делает |
|---|---|
| `/git-guardrails-claude-code` | Хук, блокирующий опасные git-команды |
| `/setup-pre-commit` | Настройка pre-commit хуков |

## Что не вошло

- **`/caveman` и `/zoom-out`** — автор удалил их из репозитория (`caveman` был случайным дубликатом, `zoom-out` не прижился на практике).
- Скиллы из `deprecated/` и `in-progress/` — устаревшие и ещё не готовые.
- Личные скиллы автора (`obsidian-vault`, `edit-article`) и его курсовая утилита (`scaffold-exercises`, `migrate-to-shoehorn`).

## Обновление

Скиллы скопированы из upstream-репозитория (июль 2026). Чтобы обновить, склонируйте [mattpocock/skills](https://github.com/mattpocock/skills) и пересинхронизируйте каталоги в `.claude/skills/`, либо используйте официальный установщик: `npx skills@latest add mattpocock/skills`.
