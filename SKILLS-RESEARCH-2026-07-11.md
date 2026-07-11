# Deep Research: экосистема агентских скиллов Claude Code — 11 июля 2026

Исследование выполнено под профиль ежедневной работы над **GARAJ.KG** (`auto-superapp-kg`).
Методика: анализ репозитория 6 параллельными агентами → веб-исследование экосистемы →
пофайловый security-веттинг 34 кандидатов с адверсариальной верификацией →
deep-research с проверкой каждого факта 3 независимыми верификаторами
(107 агентов, 25 источников, 90 claims → 25 проверено, 20 подтверждено, 5 опровергнуто).

---

## 1. Итог: что установлено

Библиотека выросла с 29 до **56 скиллов**. Все сторонние прошли проверку безопасности: **34/34 чисто, 0 опасных**.

| Источник | Установлено | Зачем для GARAJ.KG |
|---|---|---|
| anthropics/skills (официальные) | 17/17 — сверены пофайлово, **идентичны upstream**, обновление не требовалось | frontend-design (UX-полировка — приоритет №1 фаундера), webapp-testing (Playwright-верификация перед мержем), claude-api (8 фич продукта на Anthropic SDK), doc-coauthoring (русские ранбуки), docx/pdf/pptx/xlsx (паки для сторов, досье) |
| mattpocock/skills → **v1.1.0** | синхронизация: `to-issues` → **`to-tickets`** (upstream объединил и удалил старое имя), обновлены grilling/handoff/implement/prototype/setup; **новые: `wayfinder`, `research`** | основной рабочий цикл grill → tickets → implement → handoff; wayfinder — для больших фич-волн, research — фоновое изучение по первоисточникам (Next.js 16 новее training data!) |
| obra/superpowers v6.1.1 | 9: writing-plans, executing-plans, subagent-driven-development, dispatching-parallel-agents, using-git-worktrees, requesting-/receiving-code-review, verification-before-completion, finishing-a-development-branch | фаундер запускает мультиагентные адверсариальные аудиты по 4–70 агентов — это готовый каркас именно такой работы; verification-before-completion = культура «В ПРОДЕ только после проверки» |
| vercel-labs/agent-skills | 3: vercel-react-best-practices (72 правила), web-design-guidelines (правила запинены локально), vercel-composition-patterns | Next.js 16 App Router + React 19 на Vercel — прямое попадание; web-design — «единый организм» дизайна |
| Cap-go/capgo-skills | 4: capacitor-app-store, capacitor-apple-review-preflight, capacitor-ci-cd, safe-area-handling | App Store одобрил 08.07 после 3 реджектов, v1.0.1 в полёте, Google Play/RuStore впереди; preflight-скилл покрывает ровно пережитые реджекты (5.1.2(i), 2.1.0, 2.1(b)); релизы полностью через GitHub Actions |
| trailofbits/skills | 3: differential-review, insecure-defaults, agentic-actions-auditor | Bakai-эквайринг: деньги = differential-review перед мержем; insecure-defaults — fail-open конфиги; agentic-actions-auditor — у GARAJ 65 workflow, часть запускает AI-агентов |
| coreyhaines31/marketingskills v2.6.0 | 6: seo-audit, schema, programmatic-seo, ai-seo, site-architecture, aso | еженедельный SEO-файрфайтинг против mashina.kg: JSON-LD Car/Product/Offer, программные /buy/[brand]/[model] лендинги, инцидент с перезаписью title Google'ом; aso — оптимизация листингов сторов |

Полный каталог с описаниями — в [README.md](README.md).

## 2. Профиль, под который делался отбор

Из анализа `auto-superapp-kg` (6 агентов, ~600k токенов):

- **Стек:** Next.js 16.2.7 App Router (новее training data — читать bundled docs!), React 19, TS 5, Tailwind 4 CSS-first, Vercel (fra1, crons, Blob), Upstash Redis (собственный KV-слой, **никакого SQL/ORM**), Vitest (~227 тестов), Playwright, Capacitor 6 — тонкая оболочка над живым сайтом, @anthropic-ai/sdk в 8 фичах, 65 GitHub Actions как «руки» агента (в песочнице нет сети).
- **Топ повторяющихся работ:** фич-волны за флагами → пре-мерж верификация → диагностика багов по жалобам → мультиагентные аудиты → ops через Actions → App Store lifecycle → i18n×4 → SEO → платежи Bakai → UI-полировка.
- **Анти-потребности (сознательно НЕ ставилось):** SQL/Prisma/миграции, Docker/K8s, нативная iOS/Android разработка, монорепо-тулинг, OpenAI/LangChain/RAG, скаффолдеры, generic-SMM. Любой такой скилл тянул бы проект в сторону, запрещённую архитектурой.

## 3. Ландшафт экосистемы (июль 2026)

**Официальное от Anthropic — два источника:**
1. `anthropics/skills` — 17 скиллов (160k★). Каталог стабилен, наши копии байт-в-байт совпали.
2. **`anthropics/claude-plugins-official`** *(проверено 11.07.2026, голоса 3-0)* — официальный каталог плагинов: создан 20.11.2025, ~32k★, Apache-2.0, **37 внутренних плагинов** (feature-dev, pr-review-toolkit, commit-commands, security-guidance, hookify, plugin-dev…) **+ 15 внешних партнёрских**, всего 255 зарегистрированных. Ставится через `/plugin`.

**Крупнейшие комьюнити-коллекции** (звёзды — по страницам GitHub на 11.07.2026; deep-research не смог независимо подтвердить цифры, считать ориентиром):
obra/superpowers ≈252k★ (v6.1.1, принят в маркетплейс Anthropic), mattpocock/skills ≈165k★ (v1.1.0), coreyhaines31/marketingskills ≈37.6k★ (v2.6.0), vercel-labs/agent-skills ≈29k★, trailofbits/skills (40 плагинов, CC BY-SA), AgricIDaniel/claude-seo ≈11k★, Cap-go/capgo-skills (49 скиллов), capawesome-team/skills (26 скиллов).

**Маркетплейсы:** skills.sh (`npx skills`, лидерборд по установкам; по данным поисковых снапшотов лидеры — find-skills ~2M установок, frontend-design ~532k; точные цифры верифицировать не удалось — домен закрыт сетевой политикой песочницы), ClawHub (OpenClaw), skills.rest, skillsmp.com. Масштаб — сотни тысяч опубликованных скиллов, хвост в основном мусорный.

## 4. Что отклонено и почему

| Кандидат | Причина |
|---|---|
| `AgricIDaniel/claude-seo` (11k★) | Топ SEO-скилл, но: тяжёлый Python-фреймворк — 84 скрипта, 18 sh, 11 ps1, install-хуки, MCP-расширения. Исследование (arXiv:2601.10338, 3-0) подтверждает: скиллы со скриптами в **2,12× чаще уязвимы**. В облачной песочнице GARAJ без сети его фетчеры бесполезны. Выбраны markdown-скиллы marketingskills. Локально при желании: `git clone https://github.com/AgricIDaniel/claude-seo` |
| `vercel-optimize` | Прошёл веттинг (чисто), но 75+ исполняемых JS-модулей и требует локальный аутентифицированный `vercel` CLI. Ставить по требованию: `npx skills add vercel-labs/agent-skills@vercel-optimize` |
| superpowers: tdd, systematic-debugging, brainstorming, writing-skills | Дублируют установленные `tdd`, `diagnosing-bugs`, `grilling`, `skill-creator` — двойное срабатывание описаний вредит |
| capawesome-team/skills | Достойный, но пересекается с набором Capgo; один источник на нишу |
| Trail of Bits: остальные 37 плагинов | Для security-ресёрчеров: смарт-контракты, криптография, YARA, реверс — не профиль |
| steeef/claude-skill-github-actions | 1★, заброшен — ниже порога качества |
| everything-claude-code (агрегатор), awesome-списки | Сборники ссылок, не устанавливаемые скиллы |
| mattpocock: code-review, to-spec, triage, ask-matt | code-review конфликтует имением со встроенной командой; остальные окупаются при потоке внешних issue |

## 5. Безопасность: почему проверка обязательна (проверенные факты)

Все факты ниже подтверждены 3 независимыми адверсариальными верификаторами по первоисточникам (голосование 3-0):

- **ClawHavoc (январь 2026)** — крупнейший инцидент: **1 184 вредоносных скилла** от 12 аккаунтов (один залил 677), общий C2 `91.92.242[.]30`, доставляли Atomic Stealer (AMOS) — крипто-кошельки, SSH-ключи, пароли браузеров macOS; ~247 693 установки; 5 из 7 самых скачиваемых скиллов ClawHub на пике были малварью. Площадка — ClawHub (OpenClaw), не дистрибуция Anthropic, но формат SKILL.md тот же. Источники: OWASP AST01, Koi Security, The Hacker News, Antiy CERT.
- **Snyk ToxicSkills (05.02.2026)**: из 3 984 скиллов ClawHub+skills.sh — **36,82% с минимум одной уязвимостью**, 13,4% с критикой, **76 подтверждённых вредоносных пейлоадов** (поправка: заголовок Snyk «1467 malicious payloads» вводит в заблуждение — 1 467 это скиллы с флагами, вредоносных 76). Демо-репо `snyk-labs/toxicskills-goof`: фейковый «Vercel»-скилл эксфильтрует данные на pastebin, скрытые инструкции через ASCII-smuggling.
- **USENIX Security 2026** (Liu et al., arXiv:2602.06547): 98 380 скиллов → 157 вредоносных, 632 уязвимости. Ключевое: **84,2% уязвимостей живут в прозе SKILL.md, а не в коде** — статический анализ кода пропускает главный вектор атаки. Именно поэтому наша проверка читала каждый markdown-файл.
- **Trail of Bits «The sorry state of skill distribution» (03.06.2026)**: продемонстрировали обход всех трёх сканеров skills.sh, ClawHub и Cisco (например, префикс из 100 000 символов). Их вывод — сканер не заменяет построчное человеческое ревью; их `trailofbits/skills-curated` принимает скиллы только через него.
- **Контекст маркетплейсов:** skills.sh с 17.02.2026 прогоняет автоматические аудиты трёх вендоров (Gen, Socket, Snyk) по 60 000+ скиллов, скрывает вредоносные из лидерборда и с CLI v1.4.0 показывает риск-оценку перед установкой (важно: это НЕ install-time блокировка — опровергнуто 0-3). **NVIDIA SkillSpector** (Apache-2.0, ~12.9k★) — открытый пре-инсталл сканер: 68 паттернов, 17 категорий.

### Как проверялся каждый скилл в этом репозитории (методика соответствует OWASP AST01)

На каждый из 34 кандидатов — отдельный агент, читающий **все** файлы:
1. prompt-injection в прозе SKILL.md (главный вектор — 84,2%): инструкции читать `~/.ssh`, env, слать данные наружу;
2. скрытые инструкции: zero-width/bidi-символы на уровне байтов, HTML-комментарии, base64, обфускация;
3. скрипты: curl/fetch/POST на внешние хосты, eval, pipe-to-shell, install-хуки;
4. деструктивные/привилегированные команды;
5. инвентаризация всех внешних эндпоинтов (docs-ссылки — ок; отправка данных — флаг).
Любой флаг → 3 независимых адверсариальных верификатора (нужно ≥2 голосов «опасно»).

**Результат:** 34/34 чисто. Единственная закрытая поверхность: `web-design-guidelines` тянул правила с изменяемого URL (main-ветка) — снапшот запинен в `references/web-interface-guidelines-pinned-2026-07-11.md`, SKILL.md дополнен офлайн-фоллбеком.

### Чеклист для будущих установок

1. Репутация: источник с историей (компания/известный инженер), 1k+ установок или 10k+★; <100★ у неизвестного автора — стоп.
2. `git clone` во временную папку — **не** устанавливать сразу.
3. Прочитать каждый файл; скиллы «только markdown» предпочтительнее скиллов со скриптами (риск ×2,12).
4. Байтовая проверка скрытых символов: `grep -rP '[\x{200b}-\x{200f}\x{2060}\x{FEFF}]' <dir>`.
5. Инвентаризация эндпоинтов: любая ОТПРАВКА данных наружу — красный флаг.
6. Прогнать NVIDIA SkillSpector (быстрый первый фильтр; помнить про обходимость сканеров — ToB).
7. Пиновать удалённые инструкции локально; фиксировать версию/коммит источника.
8. Периодически пересинхронизировать с upstream через `diff -rq` и повторять проверку на новые версии.

## 6. Рекомендации — следующие шаги

1. **7 кастомных скиллов для репозитория GARAJ.KG** (анализ показал: категорий в экосистеме нет, а ценность максимальная): `garaj-session-protocol` (ветки claude/*, русские коммиты, мерж только по «мержи», журнал в CLAUDE.md), `garaj-known-traps` (~15 капканов: dual env-имена, next/image кеш, `next dev` 404 на /api в песочнице…), `app-store-lifecycle` (декодинг реджектов, шаблоны ответов в Resolution Center), `actions-as-hands` (какой из 65 workflow для чего), `garaj-i18n-parity`, `kv-data-patterns` (SETNX-локи с fencing, fail-closed money-writes), `founder-runbook-writer`. Это превратит 2 196 строк CLAUDE.md-дневника в детерминированное поведение.
2. **Официальные плагины Anthropic** — в сессиях Claude Code: `/plugin marketplace add anthropics/claude-plugins-official`, затем `feature-dev`, `pr-review-toolkit`, `security-guidance`, `commit-commands`.
3. **SkillSpector в пайплайн** обновления этой библиотеки.
4. По мере стабилизации эквайринга — прогонять `differential-review` на каждый мерж, трогающий деньги.

## 7. Ограничения данных

- Звёзды GitHub сняты с публичных страниц 11.07.2026 (первичный источник), но deep-research-верификаторы не смогли независимо подтвердить цифры части коллекций — считать порядком величины, не точным числом.
- Лидерборд skills.sh недоступен из песочницы (сетевая политика); цифры установок (find-skills ~2M и т.д.) — из поисковых снапшотов, не верифицированы.
- Инциденты ClawHavoc/ToxicSkills касались в первую очередь ClawHub; это не значит, что дистрибуция через GitHub-репозитории безопасна — формат и вектор (проза SKILL.md) общие.

## Источники (проверенные)

- https://owasp.org/www-project-agentic-skills-top-10/ast01 — OWASP Agentic Skills Top 10, AST01
- https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/ + https://github.com/snyk-labs/toxicskills-goof — Snyk ToxicSkills
- https://arxiv.org/html/2602.06547v1 + https://www.usenix.org/conference/usenixsecurity26/presentation/liu-yi — USENIX Security 2026
- https://arxiv.org/abs/2601.10338 — Agent Skills in the Wild (SkillScan)
- https://github.com/nvidia/skillspector — NVIDIA SkillSpector
- https://github.com/trailofbits/skills-curated + blog.trailofbits.com «The sorry state of skill distribution» (03.06.2026)
- https://vercel.com/changelog/automated-security-audits-now-available-for-skills-sh + vercel-labs/skills v1.4.0 (PR #383)
- https://github.com/anthropics/claude-plugins-official — официальный каталог плагинов Anthropic
- https://github.com/anthropics/skills, https://github.com/obra/superpowers, https://github.com/mattpocock/skills, https://github.com/vercel-labs/agent-skills, https://github.com/Cap-go/capgo-skills, https://github.com/trailofbits/skills, https://github.com/coreyhaines31/marketingskills, https://github.com/AgricIDaniel/claude-seo — источники скиллов (клонированы и проверены локально 11.07.2026)
