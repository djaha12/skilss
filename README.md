# Skills

Отобранная библиотека агентских скиллов для Claude Code. Источники (по состоянию на 11 июля 2026):

| Источник | Лицензия | Что взято |
|---|---|---|
| [anthropics/skills](https://github.com/anthropics/skills) — официальные скиллы Anthropic (160k★) | Anthropic | все 17, полный актуальный каталог |
| [mattpocock/skills](https://github.com/mattpocock/skills) v1.1.0 (165k★) | MIT | рабочий процесс: 13 скиллов |
| [obra/superpowers](https://github.com/obra/superpowers) v6.1.1, Джесси Винсент (252k★) | MIT | мультиагентная разработка: 10 скиллов |
| [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) (29k★) | MIT | React/Next.js/UI: 3 скилла |
| [Cap-go/capgo-skills](https://github.com/Cap-go/capgo-skills) — команда Capgo | MIT | Capacitor/App Store: 7 скиллов |
| [trailofbits/skills](https://github.com/trailofbits/skills) — Trail of Bits | CC BY-SA 4.0 | security: 3 скилла |
| [coreyhaines31/marketingskills](https://github.com/coreyhaines31/marketingskills) v2.6.0 (37.6k★) | MIT | SEO/ASO/рост/контент: 20 скиллов |
| [upstash/skills](https://github.com/upstash/skills) — официальные скиллы Upstash | MIT | Redis/rate-limiting: 2 скилла |
| [getsentry/sentry-for-ai](https://github.com/getsentry/sentry-for-ai) — официальный плагин Sentry | MIT | мониторинг ошибок: 2 скилла |
| [hookdeck/webhook-skills](https://github.com/hookdeck/webhook-skills) — официальные скиллы Hookdeck | MIT | вебхуки: 1 скилл |
| [nick-vels/VelsVisual](https://github.com/nick-vels/VelsVisual) v0.2.1 — Nick Vels | MIT | генерация медиа через KIE API: 1 скилл |

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
| `brainstorming` | Дизайн до кода: вопросы по одному, 2–3 подхода, спека с гейтами. ⚠️ Локальный сервер-компаньон грузит логотип с primeradiant.com (маячок, отключается `SUPERPOWERS_DISABLE_TELEMETRY=1`) |

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
| `capacitor-push-notifications` | Push-уведомления: FCM/APNs настройка, токены, каналы, траблшутинг |
| `debugging-capacitor` | Отладка Capacitor-приложений: WebView-инспекторы, white screen, сеть. ⚠️ Совет про `NSAllowsArbitraryLoads` — только для разработки, в прод нельзя (реджект App Store) |
| `ios-android-logs` | Логи устройств: adb logcat, simctl, Console.app, крэш-логи |

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
| `competitors` | Страницы сравнения с конкурентами (/vs, /alternatives), честный анализ |
| `launch` | Плейбук запуска продукта/фичи: фазы, каналы, чек-лист |
| `pricing` | Ценообразование: тарифы, фреймворки (Van Westendorp и др.), freemium vs trial |
| `analytics` | Веб-аналитика: план событий, GA4/GTM, UTM-дисциплина, consent |

### Контент и маркетинг (Corey Haines)

| Скилл | Что делает |
|---|---|
| `content-strategy` | Контент-стратегия: столпы, кластеры тем, приоритизация 40/30/20/10 |
| `social` | Соцсети: Reels/карусели/хуки/календарь, соушл-листенинг (только черновики, без автопостинга) |
| `video` | AI-видео: сценарии, промпты (Veo/Sora/Runway/Kling/Seedance), Remotion-пайплайны |
| `copywriting` | Продающие тексты страниц: заголовки, CTA, структура |
| `copy-editing` | Редактура готовых текстов: сжать, усилить, освежить |
| `image` | Картинки для маркетинга: размеры площадок, OG-картинки (@vercel/og), оптимизация |
| `marketing-plan` | Полный маркетинг-план по AARRR (13 разделов) |
| `marketing-ideas` | 139 идей роста по категориям — когда «не знаю, что ещё попробовать» |
| `marketing-council` | «Совет маркетологов»: Годин, Огилви, Хормози и др. спорят о вашем вопросе |
| `customer-research` | Исследование клиентов: интервью, JTBD, майнинг отзывов |

### Генерация медиа (VelsVisual)

| Скилл | Что делает |
|---|---|
| `visual` | Генерация фото/видео/аудио через KIE API (kie.ai): живой реестр моделей и схем, оценка стоимости в кредитах и $ до запуска, обязательное подтверждение при трате >$1 или >10% баланса |

Требует CLI и ключ KIE: `npx -y velsvisual setup --local` (флаг `--local` берёт скилл из пакета вместо `npx skills add`), либо `export KIE_API_KEY=…`. Проверка — `velsvisual credits`. Дополняет `image` и `video` (те про промпты, размеры площадок и пайплайны; этот — про сам вызов моделей).

### База данных и надёжность (Upstash, Sentry — официальные вендорские)

| Скилл | Что делает |
|---|---|
| `upstash-redis-js` | Паттерны Upstash Redis: REST-семантика, пайплайны, Lua-атомарность, TTL/кеши/сессии. *Локальная правка: удалена upstream-инструкция агенту создавать временную базу (TTL 3 дня) — риск тихой потери данных в проде* |
| `upstash-ratelimit-js` | Rate limiting: алгоритмы (sliding window, token bucket), стоимость в командах Redis, edge/serverless-ловушки |
| `sentry-debug-issue` | Отладка продакшн-ошибок через Sentry; встроенная защита: данные Sentry = недоверенный ввод |
| `sentry-create-alert` | Настройка алертов Sentry: правила, метрики, интеграции |
| `webhook-handler-patterns` | Обработчики вебхуков (Hookdeck): подпись на raw body до parse, идемпотентность, ретраи, ловушки Next.js |

### Мета

| Скилл | Что делает |
|---|---|
| `find-skills` | Ищет и устанавливает скиллы из открытой экосистемы по запросу («есть ли скилл для X?») |
| `apply-skills` | Диспетчер по команде «примени скиллы»: берёт ВСЕ подходящие скиллы под текущую задачу и отчитывается, какие применил |

## Что не вошло и почему

- **`vercel-optimize`** — мощный аудитор стоимости/производительности Vercel-проектов, но 75+ исполняемых JS-модулей и требует локально аутентифицированный `vercel` CLI. Ставьте по надобности: `npx skills add vercel-labs/agent-skills@vercel-optimize`.
- **`AgricIDaniel/claude-seo`** (11k★) — самый популярный SEO-скилл, но это тяжёлый Python-фреймворк (84 скрипта, свои install-хуки, MCP-расширения); в песочницах без сети не работает. Наш выбор — markdown-скиллы marketingskills. Для глубокого технического SEO локально: `git clone https://github.com/AgricIDaniel/claude-seo` + их установщик.
- **superpowers: `test-driven-development`, `systematic-debugging`, `writing-skills`** — дублируют установленные `tdd`, `diagnosing-bugs`, `skill-creator`; двойное срабатывание вреднее пользы. (`brainstorming` добавлен в раунде 3 — генеративная идеация, дополняет «прожарку».)
- **capawesome-team/skills** — хорошие Capacitor-скиллы, но пересекаются с набором Capgo; выбран один источник.
- **Trail of Bits: остальные 37 плагинов** — для security-ресёрчеров (смарт-контракты, криптография, реверс, malware); не наш профиль.
- **`/code-review`** (mattpocock) — конфликтует по имени со встроенной командой Claude Code.
- **`/to-spec`, `/triage`, `/ask-matt`** и прочие процессные скиллы mattpocock — окупаются на проектах с потоком внешних issue.
- **steeef/claude-skill-github-actions** — 1★, заброшен; не прошёл порог качества.
- **everything-claude-code, awesome-агрегаторы** — сборники ссылок, не скиллы.
- **Партнёрские плагины Anthropic `telegram` и `playwright`** (anthropics/claude-plugins-official/external_plugins) — это MCP-плагины с серверным кодом, вендорить как скиллы нельзя; ставить целиком: `/plugin install telegram@claude-plugins-official`.
- **redis/agent-skills** (официальный Redis) — про «настоящий» Redis-протокол/кластеры; при Upstash REST лучше подходят скиллы Upstash.
- **Остальные скиллы Upstash (qstash, vector, search, workflow) и Sentry (10 из 12)** — под продукты, которые не используются; добавить по мере надобности из уже проверенных источников.
- **telegram-bot-builder (davila7 и др. community)** — средняя репутация, а рабочий Telegram-бот уже есть; официальный партнёрский плагин Anthropic закрывает нишу лучше.
- **Community-SMM-скиллы** (Social Media Content Engine, Social Media Manager и т.п. с маркетплейсов) — авторы без репутации; контент-пакет Кори Хейнса покрывает то же из проверенного источника.
- **`autopilot`** ([nick-vels/skills](https://github.com/nick-vels/skills), MIT) — автономный сборщик проектов «под ключ» за 9 фаз (манифест → спека → таски → субагенты → ревью → слепая приёмка) с живым дашбордом. Проверку безопасности прошёл чисто и местами сам её усиливает (гейт редактирования секретов, `.env` в `.gitignore` до первого коммита, необратимые действия остаются вопросом во всех режимах, сервер дашборда только на `127.0.0.1`). Не установлен по другой причине: дублирует уже установленный рабочий цикл — `to-tickets`/`implement`, `writing-plans`/`executing-plans`/`subagent-driven-development`, `tdd`, `requesting-code-review`. Срабатывает на «собери под ключ», «build it for me». Поставить при желании: `npx skills add nick-vels/skills`.

## Официальные плагины Anthropic (ставятся отдельно)

Не скиллы, а плагины. Официальный каталог — [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) (37 внутренних + 15 партнёрских, Apache-2.0): `/plugin marketplace add anthropics/claude-plugins-official`, затем `/plugin install <имя>`. Рекомендуемые: `feature-dev` (7-фазная разработка фич), `pr-review-toolkit` (агенты ревью PR), `commit-commands`, `security-guidance` (hook-напоминания о безопасности), `hookify` (создание hooks), `plugin-dev`, `agent-sdk-dev`.

## Безопасность сторонних скиллов

Каждый сторонний скилл перед установкой прошёл проверку: чтение всех файлов, поиск prompt-injection, эксфильтрации, скрытых инструкций (zero-width символы, HTML-комментарии), внешних эндпоинтов, install-хуков и деструктивных команд + адверсариальная верификация флагов. Результат 11–12.07.2026 (три раунда): **57/57 чисто, 0 опасных**. Закрытые поверхности: `web-design-guidelines` тянул правила с изменяемого URL — запинен локально; `upstash-redis-js` содержал инструкцию агенту создавать временную базу (TTL 3 дня) — удалена. Подробности и чеклист — в [SKILLS-RESEARCH-2026-07-11.md](SKILLS-RESEARCH-2026-07-11.md).

Раунд генерации медиа (17.08.2026), тот же чеклист по обоим репозиториям Nick Vels: **2/2 чисто, 0 опасных**. `visual` установлен, `autopilot` отклонён по дублированию (см. «Что не вошло»). У `VelsVisual` проверен и CLI, а не только скилл: ноль npm-зависимостей, 63/63 теста зелёные, никаких install-хуков и lifecycle-скриптов, весь исходящий трафик — только хосты kie.ai, ключ хранится в `~/.velsvisual/config.json` с `chmod 600` и уходит исключительно как Bearer. Отмечено к сведению: `velsvisual setup` без `--local` выполняет `npx -y skills add`. Загрузка локальных файлов (`upload`, `--image`) отправляет Bearer-ключ на `kieai.redpandaai.co` — отдельный от `kie.ai` домен; **проверено: это официальный эндпоинт KIE** «File Stream Upload» из docs.kie.ai, с теми же параметрами `uploadPath`/`fileName`, что и в коде. Загруженные входные файлы KIE удаляет через 3 дня.

### Провайдер: kie.ai (проверка 17.08.2026)

Агрегатор-перепродавец чужих моделей (Veo, Kling, Seedance, Runway, Suno, Nano Banana, плюс LLM) по одному API. Оператор — NEXUSAI SERVICES LLC и INNOLEAP AI LLC, домену ~3.8 года, ScamAdviser считает сайт легитимным. Цены обычно на ~30% ниже официальных API, на отдельных моделях 60–70%; `$5 ≈ 1000 кредитов`, то есть курс `1 кредит = $0.005` из `pricing.js` совпадает с реальным. Хранение: сгенерированные файлы 14 дней, логи 2 месяца, входные файлы 3 дня; из персональных данных — только email при регистрации.

**Риск здесь не в коде, а в надёжности и биллинге.** Trustpilot ~2.5–2.7/5, и жалобы повторяющиеся: списание кредитов за упавшие и зависшие генерации при заявленной политике «failed = $0», отказы в возврате, простои (особенно Sora 2 API), поддержка в основном по азиатскому времени. Отсюда правила эксплуатации: пополнять небольшими суммами (бонус +10% дают с $1250 — столько там держать не стоит), выставить на ключ суточный и общий кап расхода в дашборде KIE, при необходимости включить IP-allowlist, ключ при подозрении пересоздать. Со стороны скилла подстраховка уже есть: `--dry-run` не тратит кредиты, а перед реальным запуском обязательны оценка цены и подтверждение при трате >$1 или >10% баланса.

## Обновление

Скиллы скопированы из upstream-репозиториев 11 июля 2026. Чтобы обновить: склонируйте нужный источник из таблицы выше и пересинхронизируйте соответствующие каталоги в `.claude/skills/` (сверка: `diff -rq`). Новые сторонние версии перед установкой прогоняйте через ту же проверку безопасности.
