# Skills

Отобранная библиотека агентских скиллов для Claude Code. Источники (по состоянию на 11 июля 2026):

| Источник | Лицензия | Что взято |
|---|---|---|
| [anthropics/skills](https://github.com/anthropics/skills) — официальные скиллы Anthropic (160k★) | Anthropic | все 17, полный актуальный каталог |
| [mattpocock/skills](https://github.com/mattpocock/skills) v1.1.0 (165k★) | MIT | рабочий процесс: 13 скиллов |
| [obra/superpowers](https://github.com/obra/superpowers) v6.1.1, Джесси Винсент (252k★) | MIT | мультиагентная разработка: 9 скиллов |
| [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) (29k★) | MIT | React/Next.js/UI: 3 скилла |
| [Cap-go/capgo-skills](https://github.com/Cap-go/capgo-skills) — команда Capgo | MIT | Capacitor/App Store: 4 скилла |
| [trailofbits/skills](https://github.com/trailofbits/skills) — Trail of Bits | CC BY-SA 4.0 | security: 3 скилла |
| [coreyhaines31/marketingskills](https://github.com/coreyhaines31/marketingskills) v2.6.0 (37.6k★) | MIT | SEO/ASO: 6 скиллов |

Плюс `CLAUDE.md` в корне — поведенческие правила по мотивам наблюдений Андрея Карпатого ([forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills)): думай прежде чем кодить, простота прежде всего, хирургические правки, движение к цели. Действуют постоянно во всех сессиях в этом репозитории.

Скиллы лежат в `.claude/skills/` — Claude Code автоматически подхватывает их в любой сессии внутри этого репозитория. Все сторонние скиллы прошли пофайловую проверку безопасности перед установкой — методика и результаты в [SKILLS-RESEARCH-2026-07-11.md](SKILLS-RESEARCH-2026-07-11.md).

## Быстрый старт

Перед первым использованием `/to-tickets` и `/implement` запустите один раз:

```
/setup-matt-pocock-skills
```

Он настроит issue tracker (GitHub / GitLab / локальные markdown-файлы), словарь меток и место для доменной документации.

## Основной рабочий цикл

1. **`/grill-me`** (или `/grill-with-docs` для кода) — агент задаёт вам детальные вопросы, чтобы согласовать требования до начала работы.
2. **`/to-tickets`** — разбивает план на маленькие независимые задачи (vertical slices) с критериями приёмки. *Бывший `/to-issues`: в upstream v1.1.0 он объединён с `to-plan` в `to-tickets`, старое имя удалено.*
3. **`/implement`** — агент последовательно берёт задачи в работу.
4. **`/handoff`** — передача контекста в свежую сессию, когда текущая переполнилась.

Для больших работ, не помещающихся в одну сессию: **`/wayfinder`** (карта пути через «туман войны»), для параллельной работы нескольких агентов — связка скиллов superpowers ниже.

## Скиллы

### Рабочий процесс (Matt Pocock, v1.1.0)

| Скилл | Что делает |
|---|---|
| `/grill-me` | Согласование требований вопросами (не только для кода) |
| `/grill-with-docs` | То же + доменная документация (CONTEXT.md, ADR) |
| `/grilling` | Базовая техника «прожарки»; в v1.1.0 — стоп-гейт подтверждения, факты vs решения |
| `/to-tickets` | План → маленькие задачи в трекере; умеет wide refactors через expand–contract |
| `/implement` | Реализация задачи из трекера |
| `/wayfinder` | **Новое.** Планирование огромного куска работы — больше, чем влезает в одну сессию |
| `/research` | **Новое.** Фоновый агент исследует вопрос по первоисточникам, оставляет cited markdown |
| `/handoff` | Передача контекста в новую сессию |
| `/diagnosing-bugs` | Систематическая диагностика багов и регрессий |
| `/tdd` | Разработка через тесты (red-green-refactor) |
| `/prototype` | Быстрый одноразовый прототип; теперь model-invoked |
| `/domain-modeling` | Доменная модель, CONTEXT.md, ADR |
| `/setup-matt-pocock-skills` | Одноразовая настройка репозитория под остальные скиллы |

### Мультиагентная разработка (obra/superpowers)

Связный набор: планирование → исполнение сабагентами → ревью → финиш ветки. Скиллы ссылаются друг на друга.

| Скилл | Что делает |
|---|---|
| `writing-plans` | Спека → детальный план реализации с чекпойнтами |
| `executing-plans` | Исполнение готового плана в отдельной сессии с ревью-чекпойнтами |
| `subagent-driven-development` | Исполнение плана свежими сабагентами: implementer → spec-reviewer → code-reviewer на каждую задачу |
| `dispatching-parallel-agents` | 2+ независимые задачи → параллельные агенты без разделяемого состояния |
| `using-git-worktrees` | Изолированные рабочие копии для параллельной работы |
| `requesting-code-review` | Запрос ревью перед merge — проверка соответствия требованиям |
| `receiving-code-review` | Как принимать ревью: верификация вместо слепого согласия |
| `verification-before-completion` | «Сделано» только после запуска проверок: evidence before assertions |
| `finishing-a-development-branch` | Структурированное завершение: merge / PR / cleanup |

### Официальные скиллы Anthropic

Автовызываемые — Claude применяет их сам, когда задача подходит. Полный актуальный каталог upstream (сверен пофайлово 11.07.2026):

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

### React / Next.js / UI (Vercel Labs)

| Скилл | Что делает |
|---|---|
| `vercel-react-best-practices` | 72 правила производительности React/Next.js от Vercel Engineering (App Router, RSC, бандл, кеширование) |
| `web-design-guidelines` | Ревью UI по Web Interface Guidelines (доступность, UX, 100+ правил). Правила пинятся локально: `references/web-interface-guidelines-pinned-2026-07-11.md` — работает и офлайн |
| `vercel-composition-patterns` | Паттерны композиции React 19: compound components, render props, `use()` |

### Мобильное / App Store (Capgo)

| Скилл | Что делает |
|---|---|
| `capacitor-app-store` | Публикация Capacitor-приложений в App Store и Google Play: подготовка, скриншоты, метаданные, сабмит |
| `capacitor-apple-review-preflight` | Префлайт перед сабмитом/после реджекта Apple: чеклисты гайдлайнов, privacy manifests, типовые причины отказов |
| `capacitor-ci-cd` | CI/CD для Capacitor: GitHub Actions, подпись, автоматизация релизов |
| `safe-area-handling` | Safe area: notch, Dynamic Island, home indicator, Android cutouts |

### Security (Trail of Bits)

| Скилл | Что делает |
|---|---|
| `differential-review` | Security-ревью диффа (PR/коммит): blast radius, регрессии безопасности, git-контекст |
| `insecure-defaults` | Поиск fail-open дефолтов: захардкоженные секреты, слабая auth, permissive-конфиги |
| `agentic-actions-auditor` | Аудит GitHub Actions с AI-агентами: prompt injection в CI/CD, опасные конфигурации, аллоулисты |

### SEO / ASO / рост (Corey Haines, marketingskills v2.6.0)

| Скилл | Что делает |
|---|---|
| `seo-audit` | Технический SEO-аудит: метаданные, индексация, Core Web Vitals, «почему упал трафик» |
| `schema` | Структурированные данные JSON-LD: Product/Offer/FAQ/Breadcrumb, rich results |
| `programmatic-seo` | Шаблонные страницы под ключевые запросы в масштабе (каталоги, город+категория) |
| `ai-seo` | Видимость в AI-поиске (GEO/AEO): цитируемость в ChatGPT/Perplexity/AI Overviews, llms.txt |
| `site-architecture` | Иерархия страниц, URL-структура, навигация, внутренняя перелинковка |
| `aso` | App Store Optimization: аудит листинга App Store / Google Play, ключевые слова, конверсия |

### Мета

| Скилл | Что делает |
|---|---|
| `find-skills` | Ищет и устанавливает скиллы из открытой экосистемы по запросу («есть ли скилл для X?») |

## Что не вошло и почему

- **`vercel-optimize`** — мощный аудитор стоимости/производительности Vercel-проектов, но 75+ исполняемых JS-модулей и требует локально аутентифицированный `vercel` CLI. Ставьте по надобности: `npx skills add vercel-labs/agent-skills@vercel-optimize`.
- **`AgricIDaniel/claude-seo`** (11k★) — самый популярный SEO-скилл, но это тяжёлый Python-фреймворк (84 скрипта, свои install-хуки, MCP-расширения); в песочницах без сети не работает. Наш выбор — markdown-скиллы marketingskills. Для глубокого технического SEO локально: `git clone https://github.com/AgricIDaniel/claude-seo` + их установщик.
- **superpowers: `test-driven-development`, `systematic-debugging`, `brainstorming`, `writing-skills`** — дублируют установленные `tdd`, `diagnosing-bugs`, `grilling`, `skill-creator`; двойное срабатывание вреднее пользы.
- **capawesome-team/skills** — хорошие Capacitor-скиллы, но пересекаются с набором Capgo; выбран один источник.
- **Trail of Bits: остальные 37 плагинов** — для security-ресёрчеров (смарт-контракты, криптография, реверс, malware); не наш профиль.
- **`/code-review`** (mattpocock) — конфликтует по имени со встроенной командой Claude Code.
- **`/to-spec`, `/triage`, `/ask-matt`** и прочие процессные скиллы mattpocock — окупаются на проектах с потоком внешних issue.
- **steeef/claude-skill-github-actions** — 1★, заброшен; не прошёл порог качества.
- **everything-claude-code, awesome-агрегаторы** — сборники ссылок, не скиллы.

## Официальные плагины Anthropic (ставятся отдельно)

Не скиллы, а плагины — ставятся командой `/plugin marketplace add anthropics/claude-code`, затем `/plugin install <имя>`: `feature-dev` (7-фазная разработка фич), `pr-review-toolkit` (агенты ревью PR), `commit-commands`, `security-guidance` (hook-напоминания о безопасности), `hookify` (создание hooks), `plugin-dev`, `agent-sdk-dev`.

## Безопасность сторонних скиллов

Каждый сторонний скилл перед установкой прошёл проверку: чтение всех файлов, поиск prompt-injection, эксфильтрации, скрытых инструкций (zero-width символы, HTML-комментарии), внешних эндпоинтов, install-хуков и деструктивных команд + адверсариальная верификация флагов. Результат 11.07.2026: **34/34 чисто, 0 опасных**. Единственная найденная поверхность — `web-design-guidelines` тянул правила с изменяемого URL; закрыто локальным пином. Подробности и чеклист — в [SKILLS-RESEARCH-2026-07-11.md](SKILLS-RESEARCH-2026-07-11.md).

## Обновление

Скиллы скопированы из upstream-репозиториев 11 июля 2026. Чтобы обновить: склонируйте нужный источник из таблицы выше и пересинхронизируйте соответствующие каталоги в `.claude/skills/` (сверка: `diff -rq`). Новые сторонние версии перед установкой прогоняйте через ту же проверку безопасности.
