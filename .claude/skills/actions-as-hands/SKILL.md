---
name: actions-as-hands
description: Dispatch guide for all GitHub Actions workflows (60+) in .github/workflows — the ONLY hands this agent has on prod, because the Claude sandbox has no internet; consult it BEFORE any attempt to curl prod, build the mobile app, probe mashina.kg, set a Vercel env var, or verify a deploy. Triggers — "проверь прод", "дёрни прод", "запусти воркфлоу", "запусти смоук", "собери APK", "собери AAB", "iOS сборка", "пропиши ключ в Vercel", "дозор mashina", "разведка mashina.kg", "проверь SMS", "прогони блюр", "check production", "run the smoke", "dispatch a workflow", "build the app", "set the env var", "probe prod", "нет сети", "no network in sandbox".
---

# Actions — руки агента

У песочницы Claude НЕТ исходящей сети: ни curl к проду, ни api.vercel.com, ни
mashina.kg отсюда не достать. Всё это делает раннер GitHub Actions. В
`.github/workflows/` лежат все воркфлоу (60+) — это готовый инструментарий; НЕ пиши
новый воркфлоу, пока не проверил, что подходящий уже есть.

Прод живёт на двух базах: `https://auto-superapp-kg.vercel.app` (дефолт в
воркфлоу) и `https://garaj.kg`.

## Как дёргать и как читать результат

Запуск — MCP-инструментом `mcp__github__actions_run_trigger` (repo
`djaha12/auto-superapp-kg`) или через API:

```
POST /repos/djaha12/auto-superapp-kg/actions/workflows/<file>.yml/dispatches
     {"ref":"main","inputs":{...}}
```

Чтение — `mcp__github__actions_list` (последние запуски),
`mcp__github__actions_get` (статус) и `mcp__github__get_job_logs` (лог шага —
именно там смоуки печатают строки с ✓/✗). CLI `gh` в песочнице НЕ установлен —
не рассчитывай на него, работай через MCP.

Dispatch не возвращает run id — после запуска опроси список запусков воркфлоу
и возьми свежий по `created_at`.

## КАПКАН №1: dispatch отвечает 404, пока файла нет на main

`workflow_dispatch` работает только для воркфлоу, существующего в default-ветке.
Новый воркфлоу из рабочей ветки задиспатчить нельзя — API отдаст 404 (капкан
задокументирован в комментарии .github/workflows/probe-import-parse.yml:
«dispatch 404 до появления в default»). Обход, принятый в репо: триггер
`on.push` на СВОЮ рабочую ветку с paths-фильтром на сам файл воркфлоу — пуш
файла и есть запуск:

```yaml
on:
  workflow_dispatch:        # заработает после мержа в main
  push:
    branches: ["claude/<твоя-ветка>"]
    paths: [".github/workflows/<имя>.yml"]
```

Поэтому у probe-mashina2…10, fetch-model-photos*, probe-carcheck, verify-prod
триггеры прибиты к давно мёртвым веткам `claude/beautiful-mayer-x2pq9n` и др. —
это архив разведок; чтобы перезапустить такой, повтори паттерн push-триггера
на своей ветке (мерж не нужен), либо добавь `workflow_dispatch` и смержи
(мерж — только по явному слову основателя, см. garaj-session-protocol).

## Настоящий CI (запускается сам)

| Воркфлоу | Триггер | Что делает |
|---|---|---|
| ci.yml | push main/claude/** (paths web/**), dispatch | tsc, npm test, check-car-specs.mjs, prod-build |
| lint-workflows.yml | push .github/workflows/**, dispatch | actionlint на все воркфлоу (shellcheck выключен) |
| post-deploy-check.yml | deployment_status от Vercel | после КАЖДОГО прод-деплоя ждёт 30с (edge-распространение, иначе смоук бьёт в старую функцию) и гоняет probe-prod + probe-sell через workflow_call |

КАПКАН №2: при красном смоуке post-deploy-check создаёт issue
«🚨 Прод-смоук упал после деплоя …» — БЕЗ дедупа, каждый красный деплой = новый
issue. Серия деплоев со сломанным смоуком плодит дубли — проверяй и закрывай
устаревшие, прежде чем реагировать на «поломку».

## Прод-пробы (когда что дёргать)

| Воркфлоу | Inputs | Когда |
|---|---|---|
| prod-curl.yml | `path` (обязателен), `jq`, `base` | УНИВЕРСАЛЬНЫЙ GET к проду — первый выбор для любого «посмотреть, что отвечает прод» |
| probe-prod.yml | — | общий смоук-регресс (главная, скрытые страницы 404, демо-код, дилеры); после любого деплоя с сомнениями |
| probe-sell.yml | — | сквозной флоу продажи: вход → публикация → каталог → УДАЛЕНИЕ тестового объявления (DELETE /api/listings; шапка-комментарий файла про «скрытие» устарела — тело удаляет). ОБЯЗАТЕЛЕН после любого изменения sell/каталога/auth |
| probe-auth.yml | `base` | диагностика «вход слетает»: Set-Cookie, Max-Age, стабильность сессии между лямбдами |
| probe-admin.yml | `phone` | доступ к CRM /admin под номером основателя |
| probe-sms.yml | `phone`, `base` | боевой SMS-шлюз Nikita.kg: via:"nikita" = работает, via:"demo" = env NIKITA_* не подхватились |
| probe-demo-login.yml | — | демо-вход ревьюера Apple (DEMO_OTP_PHONES / DEMOOTPPHONES задан?) |
| probe-photos.yml | — | 12 AI-фото комтранса на проде (поллинг до 12 мин) |
| probe-plates.yml | `pr_token` | аудит читаемых госномеров на фото прода через Plate Recognizer (первые 40 фото не-демо объявлений) |
| kick-blur.yml | — | принудительный ретрай блюра номеров по реальным объявлениям (медленный: sleep 25с на слаг) |
| probe-import.yml | `ad_url` | парсер импорта на живых объявлениях mashina |
| probe-import-parse.yml | — | детерминированный разбор карточки mashina НАСТОЯЩИМ кодом web/lib/server/import-parse.ts через tsx |
| probe-bakai-swagger.yml | — | сверка контракта Bakai Open Banking (swagger с раннера) |
| probe-calk.yml | — | разведка calk.kg (растаможка) |
| domain-check.yml | `domains` | DNS + whois доменов |
| vercel-diag.yml | `vercel_token` | почему Vercel отвечает 402 на деплой |

probe-prod и probe-sell имеют `workflow_call` — их переиспользует
post-deploy-check с `secrets: inherit`. Оба требуют gh-секрет
`DEMO_OTP_SECRET` (== `DEMOOTPSECRET` в Vercel): probe-sell шлёт заголовок
`x-demo-otp-secret`, чтобы прод НЕ считал смоук-запросы в CRM-счётчики
(сервер сверяет в web/lib/server/smoke.ts) — иначе дайджест основателю врёт.

## Разведка конкурента mashina.kg

Живые (есть workflow_dispatch): probe-mashina-detail.yml (JSON-LD/og карточки,
телефон продавца), probe-mashina-feed.yml (JSON-лента свежих для дозора),
probe-mashina-prices.yml и probe-mashina-revenue.yml (тарифы/выручка),
probe-mashina-watch-test.yml (точная копия регэкспа scanMashina из
web/lib/server/mashina-watch.ts на живой выдаче), probe-phone.yml (где телефон
в HTML).

Архив (push-триггер на мёртвую ветку, dispatch нет): probe-mashina2…10 —
история вскрытия открытого API `api.mashina.kg/api/mbank-proxy/v1` (марки =
атрибут 16, модели = 17, кузова = 18). Не перезапускай — читай их код как
документацию по API конкурента.

mashina-watch-now.yml — ручной прогон «дозора» на проде
(`garaj.kg/api/cron/mashina-watch?key=CRON_SECRET&force=1`). Дозор СПИТ: крона
в vercel.json нет; gh-секрет CRON_SECRET исторически оказывался ПУСТЫМ — тогда
прод ответит 404/503, а кнопка «Проверить сейчас» в CRM работает всегда.

## Фабрика мобильных релизов

| Воркфлоу | Продукт | Подпись |
|---|---|---|
| build-android.yml | app-debug.apk (артефакт) — поставить на телефон | debug, секреты не нужны |
| build-android-release.yml | подписанный AAB для Google Play | требует 4 секрета: GOOGLE_KEYSTORE_BASE64, KEYSTORE_PASSWORD, KEY_ALIAS, KEY_PASSWORD |
| build-android-aab.yml | AAB + СВЕЖИЙ keystore за один запуск (два артефакта, keystore живёт 1 день) | генерирует ключ сам — для случая «секрет keystore обрезан/битый» |
| build-apk-release.yml | подписанный APK → Vercel Blob → стабильная ссылка garaj.kg/app/download | генерирует ключ сам; требует секрет BLOB_READ_WRITE_TOKEN |
| build-ios.yml | по умолчанию симулятор-сборка (только компилируемость); .ipa лишь при секретах APPLE_* | ⚠ macOS-раннер = ×10 минут на приватном репо — гонять экономно |
| make-keystore.yml | одноразовая генерация upload-keystore (приватный артефакт, 1 день) | — |

Канонический путь для Play после того как 4 keystore-секрета прописаны —
build-android-release.yml; build-android-aab.yml — только бутстрап/восстановление
ключа (так же описывает Android-раздел app-store-lifecycle).

⚠ Keystore = личность приложения: после build-apk-release/build-android-aab
СКАЧАТЬ артефакт `garaj-keystore-SECRET`, вписать 4 секрета, УДАЛИТЬ артефакт
со страницы запуска. FCM-шаги включаются только при секрете
GOOGLE_SERVICES_JSON — иначе тихий no-op (но Play-сборка без него поедет БЕЗ
пушей).

## Ops/секреты: одноразовый Vercel-токен

set-anthropic-key.yml (ANTHROPIC_API_KEY — автоблюр), set-admin-phones.yml
(ADMIN_PHONES), set-channel.yml (TELEGRAM_CHANNEL), set-platerecognizer.yml
(PLATE_RECOGNIZER_TOKEN), setup-blob.yml (BLOB_READ_WRITE_TOKEN),
setup-vercel.yml (Root Directory + AUTH_SECRET), setup-telegram.yml /
setup-telegram-full.yml (вебхук бота; токен бота — из gh-секрета
TELEGRAM_BOT_TOKEN, НЕ input). Все, КРОМЕ setup-telegram.yml, берут
`vercel_token` как input, маскируют через `::add-mask::`, апсертят env в
production+preview и передеплоивают main. setup-telegram.yml — исключение:
единственный input — `webhook_secret`, в api.vercel.com не ходит, env не
апсертит и не передеплоивает; он лишь ставит вебхук Telegram и напоминает
вручную задать в Vercel TELEGRAM_BOT_TOKEN / TELEGRAM_WEBHOOK_SECRET /
NEXT_PUBLIC_TELEGRAM_BOT + Redeploy.

ОБЯЗАТЕЛЬНЫЙ ритуал (причина — в CLAUDE.md репо: «env: в воркфлоу светит
инпуты в заголовке шага»; маскировка прячет значение в тексте лога, но не в
метаданных шага):

1. Запустить set-*/setup-* с одноразовым токеном.
2. Проверить результат (сами воркфлоу валидируют ключ живым запросом:
   set-anthropic-key бьёт в api.anthropic.com, set-platerecognizer — в
   api.platerecognizer.com; плюс prod-curl/probe-sell после передеплоя —
   probe-sell печатает anthropicKeySet/plateRecognizerSet/blobSet из
   /api/admin/check).
3. УДАЛИТЬ логи запуска (Delete run со страницы запуска или
   `DELETE /repos/djaha12/auto-superapp-kg/actions/runs/{id}`).
4. Отозвать одноразовый токен: revoke-token.yml (input `vercel_token`) — сам
   находит и удаляет себя через api.vercel.com. Не полагайся на «потом» —
   отзыв в том же заходе.

## Опасные ручки (необратимые операции на проде)

- reset-balances.yml — обнуление ВСЕХ балансов продвижения. Двойной
  предохранитель: сухой прогон всегда, реальное выполнение только при input
  `execute=yes` (`promos=yes` дополнительно снимает активные тарифы). НИКОГДА
  не передавай execute=yes без явного слова основателя.
- purge-test-listings.yml — удаление тестовых «Смоук Тест» объявлений
  (наследие старого probe-sell): dry-run, затем `?run=1` в одном запуске.

Оба ходят в /api/admin/* с `Authorization: Bearer CRON_SECRET` — при пустом
gh-секрете CRON_SECRET молча получишь 401/404.

## Cron-подобные ручные запуски

- digest-now.yml — дёргает /api/cron/digest (повтор в тот же день →
  `{"ok":true,"skipped":true}`, дедуп-метка живёт сутки).
- journal-now.yml — бот Автожурнала /api/cron/journal (до минуты; кладёт
  ЧЕРНОВИКИ, публикация — в админке).
- mashina-watch-now.yml — см. выше.

Все требуют gh-секрет CRON_SECRET = env CRON_SECRET в Vercel; его статус в
GitHub исторически «не подтверждён» — при 401/404/503 первым делом подозревай
пустой секрет, а не сломанный код.

## Fetch-воркфлоу (мост «CDN → репозиторий»)

Раннер качает то, что песочнице недоступно, и коммитит в репо
(`permissions: contents: write`): fetch-brand-logos.yml (dispatch; логотипы
марок из car-logos-dataset → коммит в main → Vercel), fetch-assets.yml,
fetch-commercial-photos.yml, fetch-model-photos.yml…photos4.yml (AI-фото машин
с CDN генератора; push-триггеры на старые ветки — архив), save-logos.yml
(dispatch; драфты логотипа в design/logo-drafts, коммит в старую ветку).
Новую партию картинок доставляй тем же паттерном: новый fetch-воркфлоу с
push-триггером на свою ветку.

## Вторая поверхность автоматики: Vercel Crons

web/vercel.json — 6 кронов (регион fra1):

| Путь | Расписание (UTC) |
|---|---|
| /api/cron/check-searches | 0 9 * * * |
| /api/cron/digest | 0 3 * * * (= 09:00 Бишкека) |
| /api/cron/cleanup-stale | 0 2 * * * |
| /api/cron/auto-bump | 0 5 * * * |
| /api/cron/fx | 0 4 * * * |
| /api/cron/reconcile-pay | */10 * * * * |

Авторизация ручного вызова: `Authorization: Bearer CRON_SECRET` или
`?key=CRON_SECRET`. Заголовку `x-vercel-cron` НЕ доверяем — он форгабелен
(комментарий в digest-now.yml). CRON_SECRET в Vercel задан и кроны живые
(проверено 30.06.2026); сомнение всегда про gh-секрет, не про Vercel.
Дозор mashina-watch в этот список НЕ входит — он спит намеренно.
