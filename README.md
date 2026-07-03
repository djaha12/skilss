# Skills

Отобранные агентские скиллы для Claude Code из трёх источников:

- [mattpocock/skills](https://github.com/mattpocock/skills) (MIT) — рабочий процесс: согласование требований → разбивка на задачи → реализация
- [anthropics/skills](https://github.com/anthropics/skills) — официальные скиллы Anthropic (документы, фронтенд, тестирование и др.)
- [vercel-labs/skills](https://github.com/vercel-labs/skills) — `find-skills`, мета-скилл для поиска новых скиллов

Плюс `CLAUDE.md` в корне — поведенческие правила по мотивам наблюдений Андрея Карпатого ([forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills)): думай прежде чем кодить, простота прежде всего, хирургические правки, движение к цели. Действуют постоянно во всех сессиях в этом репозитории.

Скиллы лежат в `.claude/skills/` — Claude Code автоматически подхватывает их в любой сессии внутри этого репозитория и делает доступными как слэш-команды. Работают с любой моделью: Opus, Sonnet, Fable.

## Быстрый старт

Перед первым использованием `/to-issues` и `/implement` запустите один раз:

```
/setup-matt-pocock-skills
```

Он настроит issue tracker (GitHub / GitLab / локальные markdown-файлы), словарь меток и место для доменной документации.

## Основной рабочий цикл

1. **`/grill-me`** (или `/grill-with-docs` для кода) — агент задаёт вам детальные вопросы, чтобы согласовать требования до начала работы.
2. **`/to-issues`** — разбивает большой план на маленькие независимые задачи (vertical slices) с критериями приёмки и метками для агента. *Бывший `/to-tickets`.*
3. **`/implement`** — агент последовательно берёт задачи в работу.
4. **`/handoff`** — передача контекста в свежую сессию, когда текущая переполнилась. *Бывший `/hand-off`.*

## Скиллы

### Рабочий процесс (Matt Pocock)

| Скилл | Что делает |
|---|---|
| `/grill-me` | Согласование требований вопросами (не только для кода) |
| `/grill-with-docs` | То же + доменная документация (CONTEXT.md, ADR) |
| `/grilling` | Базовая техника «прожарки» (используется двумя скиллами выше) |
| `/to-issues` | План → маленькие задачи в трекере с критериями приёмки |
| `/implement` | Реализация задачи из трекера |
| `/handoff` | Передача контекста в новую сессию |
| `/diagnosing-bugs` | Систематическая диагностика багов и регрессий |
| `/tdd` | Разработка через тесты (red-green-refactor) |
| `/prototype` | Быстрый одноразовый прототип для проверки идеи |
| `/domain-modeling` | Доменная модель, CONTEXT.md, ADR (нужен для grill-with-docs) |
| `/setup-matt-pocock-skills` | Одноразовая настройка репозитория под остальные скиллы |

### Официальные скиллы Anthropic

Автовызываемые — Claude применяет их сам, когда задача подходит:

| Скилл | Что делает |
|---|---|
| `frontend-design` | Отличительный, нешаблонный дизайн интерфейсов |
| `webapp-testing` | Тестирование локальных веб-приложений через Playwright |
| `docx` / `pdf` / `pptx` / `xlsx` | Создание и правка Word, PDF, PowerPoint, Excel |
| `doc-coauthoring` | Совместное написание доков, спек, proposals |
| `mcp-builder` | Создание MCP-серверов (Python/TypeScript) |
| `skill-creator` | Создание и оптимизация собственных скиллов |
| `claude-api` | Справочник по Claude API при разработке LLM-приложений |
| `canvas-design` / `algorithmic-art` / `theme-factory` / `slack-gif-creator` | Визуальный дизайн, генеративное искусство, темы, GIF |
| `web-artifacts-builder` | Сложные HTML-артефакты (React, Tailwind, shadcn/ui) |
| `brand-guidelines` / `internal-comms` | Фирменный стиль Anthropic, внутренние коммуникации |

### Мета

| Скилл | Что делает |
|---|---|
| `find-skills` | Ищет и устанавливает скиллы из открытой экосистемы по запросу («есть ли скилл для X?») |

## Что не вошло и почему

- **`/code-review`** — конфликтует по имени со встроенной командой Claude Code, которая делает то же самое.
- **`/triage`, `/to-prd`, `/research`** — процессные скиллы для проектов с потоком внешних issue; окупаются только там.
- **`/ask-matt`, `/teach`, `/writing-great-skills`, `/codebase-design`, `/improve-codebase-architecture`, `/resolving-merge-conflicts`, `/git-guardrails-claude-code`, `/setup-pre-commit`** — нишевые или дублируют встроенные возможности.
- **`/caveman` и `/zoom-out`** — автор сам удалил их из upstream-репозитория.
- Скиллы из `deprecated/` и `in-progress/`, личные скиллы автора.

Любой из них можно добавить позже из [mattpocock/skills](https://github.com/mattpocock/skills).

## Обновление

Скиллы скопированы из upstream-репозитория (июль 2026). Чтобы обновить, склонируйте [mattpocock/skills](https://github.com/mattpocock/skills) и пересинхронизируйте каталоги в `.claude/skills/`.
