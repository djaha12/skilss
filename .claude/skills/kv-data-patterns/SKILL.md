---
name: kv-data-patterns
description: >-
  MUST-READ before touching any data storage, keys, balances, counters, locks, caches or
  migrations in GARAJ.KG — the entire database is Upstash Redis via web/lib/server/kv.ts
  with bespoke idioms (fencing locks, fail-closed money, sharded listings, dry-run
  migrations) that generic Redis knowledge gets wrong. Triggers — "add a field", "store",
  "save to database", "new counter", "migration", "race condition", "double charge",
  "Redis", "KV", "cache"; Russian — "хранилище", "сохранить", "база данных", "ключ",
  "счётчик", "баланс", "списание", "зачисление", "миграция", "гонка", "двойное списание",
  "кэш", "лок", "объявления пропали".
---

# KV-паттерны GARAJ.KG: Redis как единственная база

## 0. Декрет основателя: НИКАКОГО SQL

Единственная база данных проекта — **Upstash Redis** через `web/lib/server/kv.ts`.
НЕ предлагай Prisma, Postgres, Supabase, SQLite, ORM или «давайте нормальную БД».
Перенос денег/пользователей на Postgres числится в `ROADMAP.md` как стратегический
пункт «под рост» — это решение основателя на будущее, не твоё. Любая новая фича
хранит данные в kv() по паттернам ниже.

## 1. Три реализации Kv и синглтон kv()

`kv()` в `web/lib/server/kv.ts` выбирает стор по env (кэш в `globalThis.__garajKv`):

1. **UpstashKv** (прод) — если заданы URL+токен. Цепочка имён (все три пары понимаются):
   `FRAKVURL`/`FRAKVTOKEN` (Франкфурт, приоритет) → `UPSTASH_REDIS_REST_URL`/`_TOKEN` →
   `KV_REST_API_URL`/`_TOKEN` (Vercel-маркетплейс). Снял FRAKV* — автоматический откат
   на старую базу США, без правки кода.
2. **BlobKv** (фолбэк) — только если есть `BLOB_READ_WRITE_TOKEN` без Upstash.
   НЕ атомарен (read-then-write). На проде это авария: kv() кричит в console.error
   каждый холодный старт.
3. **MemoryKv** (dev/тесты) — без env вообще. Атомарность честно воспроизведена:
   incr/incrby/setnx/lockAcquire сделаны СИНХРОННО (peek/put без внутренних await),
   иначе два конкурентных вызова вклинивались бы между чтением и записью.

**Философия «ноль моков»:** все тесты (`web/test/*.test.ts`, vitest) гоняют
НАСТОЯЩИЙ серверный код in-process на MemoryKv — без сети, без vi.mock на хранилище
(единственное исключение — `kv-write-errors.test.ts`: стаб env и fetch, чтобы прогнать
ошибки записи через настоящий UpstashKv, всё равно in-process и без сети).
`web/test/setup.ts` перед каждым тестом сбрасывает `globalThis.__garajKv = undefined`,
и следующий kv() создаёт свежий MemoryKv. Новые тесты пиши так же: импортируй боевую
функцию, зови её, проверяй результат через тот же kv(). Мокать kv — запрещённый паттерн.

## 2. Схема ключей

Коллекции лежат ЦЕЛИКОМ под одним ключом (JSON-массив), без сканов по префиксу.
`scanKeys("bal:*")` существует только для обслуживания (обнуление балансов) — на
горячем пути SCAN запрещён; фиксированные бакеты читаются одним MGET
(пример: `extraPromoTotals` в `web/lib/server/billing.ts`).

Объявления — ШАРДИРОВАННАЯ схема (`web/lib/server/listings.ts`, шапка файла):

| Ключ | Что | Зачем |
|---|---|---|
| `listing:{id}` | JSON одного объявления | точечные чтение/запись: PATCH, блюр, AI-осмотр не перетирают весь список |
| `listing:slug:{slug}` | id | резолв страницы/чатов/блюра |
| `listings:ids` | массив id, createdAt DESC | индекс читается целиком, значения — постранично MGET |
| `listings:owner:{key}` | массив id владельца | кабинет и счётчик «Продал через GARAJ.KG» без скана |
| `listings:ver` | INCR-счётчик версий | дешёвая валидация module-кэша между инстансами |
| `listings:migrated` | флаг | новая схема активна |
| `listings:all` | ЛЕГАСИ, весь список | НЕ удалять — страховка для отката |

Деньги/CRM (`web/lib/server/billing.ts`): `bal:{userKey}` (число, сомы),
`bal:tx:{userKey}` (леджер, хвост 20), `crm:payments` (журнал, хвост 500),
`order:{orderId}` (TTL 90 дней), `pay:orders:pending` (индекс досверки кроном).

**Правило durable-счётчиков:** обрезаемый журнал — НЕ источник итогов. Лайфтайм-цифры
считай отдельными вечными ключами и инкременти их В МОМЕНТ события: `pay:count`,
`pay:sum:{kind}`, `promo:count:{tier}`/`promo:sum:{tier}`, `sales:count`/`sales:sum`,
подневные `stat:{день}:pay_*`/`promo_*`/`sales_*` (с TTL `STAT_TTL_SEC`). Суммы —
ТОЛЬКО атомарным `incrby` (read-modify-write терял слагаемые при гонке двух платежей
и занижал выручку — реальный инцидент). NaN-защита обязательна ДО incrby: NaN заразен
и портит ключ навсегда (см. `recordSale`).

## 3. Локи: SET NX EX + fencing-токен

`withLock(lockKey, fn, opts)` из `web/lib/server/kv.ts` — единственный способ
сериализовать read-modify-write между serverless-лямбдами. Устройство:

- захват: `lockAcquire` = `SET lock:{key} {token} NX EX {ttl}`; TTL по умолчанию 30с,
  до 30 попыток с шагом 70мс, иначе `LockBusyError`;
- **токен-владелец (fencing)**: случайный token на каждый вызов. `lockRelease` снимает
  лок атомарным EVAL compare-and-delete — ТОЛЬКО если value ещё == token. ПОЧЕМУ: если
  наш TTL истёк посреди работы и лок перезахватил другой процесс, слепой DEL удалил бы
  ЧУЖОЙ лок и третий процесс вошёл бы в ту же критическую секцию каскадом;
- release — best-effort в finally: с 10.07.2026 записи Upstash бросают при !res.ok, и
  исключение из finally ЗАТЁРЛО бы успешный результат fn() (деньги списаны, а вызывающий
  увидел ошибку). Неснятый лок безвреден — истечёт по TTL.

Использование в деньгах (`billing.ts`): `topUp`, `charge`, `setBalanceExact` делают
чтение→запись баланса под `withLock("bal:" + userKey(phone))`. Иначе два параллельных
списания оба «проходят» по одному остатку (двойная покупка за полцены). Лок НЕ
реентрантный: не вызывай setBalanceExact изнутри topUp/charge — дедлок до LockBusyError.

Новая критическая секция = withLock по ключу изменяемой сущности + секция обязана
уложиться в TTL (30с покрывает несколько round-trip'ов Upstash).

## 4. Деньги — fail-closed, журналы — best-effort

Два жёстких правила из `kv.ts` (инциденты 28.06 и аудиты 06–10.07.2026):

1. **Записи НЕ ретраятся.** Исход упавшего INCR/SET неизвестен — слепой повтор мог бы
   задвоить списание. HTTP-ошибка записи БРОСАЕТ (тихий undefined = потерянный баланс,
   хуже честного 500). Чтения из `RETRYABLE_READ_OPS` — до 3 попыток с бэкоффом и
   таймаутом 8с, при исчерпании тоже БРОСАЮТ, а не маскируются под null: «Upstash
   отбросил GET» ≠ «данных нет» (каталог схлопывался в 0).

2. **Денежные эндпоинты гейтятся `isAtomicKv()`.** Только UpstashKv даёт настоящий
   SET NX EX; на BlobKv/MemoryKv setnx/lock — read-then-write, идемпотентность
   зачисления ломается. Образцы (копируй при новом денежном приёмнике):
   - `web/app/api/billing/webhook/route.ts` — на проде при !isAtomicKv() отвечает
     503 `store_not_atomic` (банк повторит);
   - `web/app/api/billing/invoice/route.ts` — отвечает `result: 1, "temporary error,
     retry"` (протокол лицевых счетов; там же потолок `MAX_SINGLE_TOPUP` = отказ
     кодом 242 ДО зачисления);
   - `web/app/api/cron/reconcile-pay/route.ts` — выходит с `reason: "non_atomic_kv"`.

3. **Сверка по НАШЕМУ заказу.** Вебхук зачисляет `order.phone` сумму `order.amount`
   из ранее созданного `order:{orderId}` — бизнес-данные коллбэка банка игнорируются.
   Сумму сравнивай через `callbackAmountMatches` (целые минорные единицы, float-хвост
   не даёт ложный mismatch).

При этом ЖУРНАЛЫ вокруг денег — best-effort: `recordPayment`, `recordPromoSale`,
`recordSale` обёрнуты в try/catch целиком — сбой CRM-статистики не должен ронять сам
платёж. Различай: изменение баланса — под локом и бросает; учёт для дашборда — глотает.

## 5. Module-scope кэши: короткий TTL + версия + single-flight

Serverless-инстансы не делят память, поэтому кэш валидируется дёшево через KV:

- **Каталог** (`activeCarsCached` в `listings.ts`): кэш на 90с (`ACTIVE_CACHE_TTL_MS`),
  но КАЖДЫЙ хит сверяет один GET `listings:ver` — писатели INCR'ят версию, публикация
  видна сразу. Single-flight через inflight-Promise. Stale-if-error: сбой чтения при
  живом кэше отдаёт прежний снимок.
- **Защита от усечённого чтения** (`isTruncatedRead`): если MGET принёс <80% значений
  от индекса или индекс внезапно пуст при непустом прошлом кэше — это сбой/вытеснение
  KV, а НЕ «всё удалили». Усечённый снимок НЕ кэшировать (число объявлений прыгало
  300→8→0), отдавать last-good или бросать.
- **Салоны** (`kvDealerOwnerKeys`, TTL `DEALERS_TTL_MS` = 30с): синхронный горячий путь
  читает тёплый снапшот, протухание запускает ФОНОВОЕ обновление. Ловушка холодного
  инстанса: снапшот пуст → бейджи «Автосалон» пропадали и замораживались кэшем
  («прыгающие салоны», 04.07). Лечение: async-читатели, кормящие listingToCar, ОБЯЗАНЫ
  сначала `await ensureDealerKeys()`.

Новый кэш делай по этому шаблону: module-scope переменная + TTL 30–90с + сверка
версии-ключа при записях + single-flight + stale-if-error. НЕ изобретай Redis-кэш
поверх Redis.

## 6. Админ-миграции: сухой прогон, идемпотентность, без удалений

Образцы: `web/app/api/admin/migrate-listings/route.ts` и
`web/app/api/admin/migrate-redis/route.ts`. Канон:

- гейт `requireAdmin()` → при провале 404 (не 401 — не палим существование роута);
  migrate-redis дополнительно пускает по `?key=CRON_SECRET`;
- **GET без параметров = СУХОЙ ПРОГОН**: статус, счётчики, проверка связи, ничего не
  меняет. `?run=1` — выполнить. Хелперы принимают `{ dryRun }` (см. `stripAllPromos`);
- **идемпотентно и БЕЗ удалений**: существующие `listing:{id}` не перезаписываются
  (в шарде может быть более свежая версия), легаси-ключ остаётся для отката. Повторный
  запуск добирает пропущенное — рекомендация «прогнать дважды и сверить
  `legacyNotSharded == 0`»;
- массовые переносы — батчами: Promise.all по 25 сетов, MGET чанками, на Upstash —
  `/pipeline` (одна команда на HTTP-вызов превращала перенос в ~90мс × N ключей);
- `export const dynamic = "force-dynamic"` + `maxDuration = 60`.

## 7. Blob-мост (не трогать бездумно)

На время переезда Blob→Redis долгоживущие ключи из `BLOB_BRIDGE_PREFIXES` (`kv.ts`:
чаты `msg:*`, счётчики `views:`/`calls:`, `tglink:`, `plateblur:*`, …) при промахе в
Redis дочитываются из старого Vercel Blob и лениво мигрируют (write-through SET NX —
параллельный свежий писатель побеждает мост). Ошибка Blob ПРОБРАСЫВАЕТСЯ: «ключа нет»
≠ «не смогли прочитать», иначе сбой CDN зафиксировал бы пустоту навсегда. Старый
Blob-стор не чистить, пока мост жив (декрет в корневом CLAUDE.md). Эфемерным ключам
(otp, лимиты, локи, кэши) мост не нужен — новые эфемерные префиксы туда НЕ добавляй.

## 8. Чек-лист для новой фичи с данными

1. Ключ по конвенции `домен:{id}` / `домен:под:{id}`; коллекция целиком под одним
   ключом, если <~сотен КБ; иначе шардируй по образцу listings + индекс id.
2. Счётчики/суммы — `incr`/`incrby` (атомарно), никогда get→set.
3. Read-modify-write разделяемого значения — только под `withLock`.
4. Деньги — гейт `isAtomicKv()` + идемпотентность по setnx-метке + fail-closed ответ
   «повтори позже».
5. Горячее чтение — module-кэш 30–90с с версией и single-flight.
6. TTL всему эфемерному (`set(key, v, ttlSec)`); вечные ключи — осознанно.
7. Тест в `web/test/` на MemoryKv, без моков; сброс стора уже делает setup.ts.
