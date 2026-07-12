---
name: apply-skills
description: >-
  Explicit founder command to apply ALL relevant installed skills to the current
  task: inventory .claude/skills, select every skill matching the task (not just
  one), read and follow them, then report which were applied. ALWAYS invoke when
  the user says «примени скиллы», «применяй скиллы», «используй скиллы»,
  «подключи скиллы», «по скиллам», «со скиллами», «прогони по скиллам»,
  «apply skills», «use skills» — or asks whether skills are being used at all.
---

# Примени скиллы (диспетчер)

## Зачем это

Основатель не перечисляет нужные скиллы по именам — он говорит «примени
скиллы». Эта команда означает одно: работать на максимуме качества,
задействовав все подходящие установленные скиллы, а для крупных задач — и
субагентов. Лимиты не экономить (постоянное правило основателя от 11.07.2026,
зафиксировано в CLAUDE.md проекта). Команда работает на любой модели Claude —
скиллы это файлы репозитория, а не функция конкретной модели.

## Как исполнять

1. **Инвентаризация.** Список доступных скиллов уже подгружен в сессию из
   `.claude/skills/` (при сомнении — `ls .claude/skills`).
2. **Выбор.** Определи тип текущей задачи и возьми ВСЕ релевантные скиллы,
   а не один. Протокольные скиллы применяй независимо от типа задачи, если
   они установлены: `garaj-session-protocol`, `garaj-known-traps`.
3. **Применение.** Прочитай выбранные SKILL.md и работай строго по ним.
   Скилл — карта, а не территория: конкретные детали продукта (цвета,
   тексты кнопок, пути экранов) сверяй по живому коду репозитория, а не
   цитируй шпаргалку вслепую — она могла устареть (урок замера 12.07.2026).
4. **Масштаб.** Для больших задач подключай субагентов по скиллам
   `subagent-driven-development` / `dispatching-parallel-agents`, а проверку
   перед «готово» — по `verification-before-completion`.
5. **Отчёт.** В конце ответа одной строкой перечисли применённые скиллы:
   `Скиллы: …` — так основатель видит, что команда исполнена.

## Быстрая карта задач → скиллы

Строка применима, если скилл установлен в текущем репозитории; отсутствующие
пропускай молча.

| Задача | Скиллы |
|---|---|
| Любая фича или правка кода | garaj-session-protocol, garaj-known-traps, vercel-react-best-practices, verification-before-completion |
| SEO, видимость в Google | seo-audit, schema, programmatic-seo, ai-seo, site-architecture |
| App Store / Google Play / RuStore | app-store-lifecycle, capacitor-app-store, capacitor-apple-review-preflight, aso |
| Баг, «не работает», жалоба | diagnosing-bugs, garaj-known-traps, debugging-capacitor, ios-android-logs, sentry-debug-issue |
| Деньги, платежи, баланс | kv-data-patterns, tdd, differential-review, insecure-defaults |
| UI, дизайн, «сделай красиво» | frontend-design, web-design-guidelines, vercel-composition-patterns |
| Тексты интерфейса, переводы | garaj-i18n-parity |
| Инструкция для основателя | founder-runbook-writer, doc-coauthoring |
| Пуш-уведомления | capacitor-push-notifications |
| База данных, Redis, кеши | kv-data-patterns, upstash-redis-js, upstash-ratelimit-js |
| GitHub Actions, проверка прода | actions-as-hands, agentic-actions-auditor, webapp-testing |
| Цены, конкуренты, запуск | pricing, competitors, launch |
| Платная реклама, таргет, объявления Meta/TikTok | ads, ad-creative, image, copywriting, competitors |
| Контент, посты, SMM, «что постить» | garaj-content-week, content-strategy, social, video, copywriting, image, ai-seo |
| Совет маркетологов, «как продвигать» | marketing-council, marketing-ideas, marketing-plan, customer-research |
| Вебхуки (банк, платежи, интеграции) | webhook-handler-patterns, kv-data-patterns, differential-review |
| ИИ-фичи продукта (Anthropic API) | claude-api |
| Большая многошаговая работа | writing-plans, executing-plans, subagent-driven-development, using-git-worktrees, wayfinder |

Если задача не подходит ни под одну строку — всё равно просмотри полный список
установленных скиллов и возьми ближайшие по смыслу. «Ни один скилл не подошёл» —
допустимый ответ только после реальной проверки списка.
