---
name: garaj-known-traps
description: Catalog of permanent rules and known traps (капканы) of the GARAJ.KG codebase that have ALREADY burned production or the founder — consult it BEFORE touching build/stand commands, images/logos, env vars, native iOS/Android surfaces, React hooks, caching, salon logic, prices, or the mashina import. Triggers — "капкан", "почему не обновилось", "картинка старая", "лого", "заставка", "иконка", "не работает импорт", "код не приходит", "404 на api", "битая сборка", "next dev", "env не читается", "переменная в Vercel", "салон", "бейдж Автосалон", "тарифы", "цены", "натив", "мерцание", "назад", "свайп", "Показать ещё", "Недавно просмотренные", "вибрация", "blur", "модалка", "шторка", "скрин пустой", "Playwright", "force-dynamic", "ISR", "кэш", "new build", "native build", "hooks lint error", "stale image", "screenshot empty", "image not updating after deploy", "API returns 404 in dev", "build has no CSS".
---

# GARAJ.KG — известные капканы и вечные правила

Каждый пункт: **когда** → **правило** → почему → где закреплено.
Эти правила уже стоили прод-инцидентов или паники основателя. Нарушение = повторение уже прожитой боли.

## Стенд и сборка

**1. Проверяешь /api/* на стенде** → поднимай прод-сборку `next build && next start` на :3100, НЕ `next dev`.
`next dev` в песочнице отдаёт 404 на ВСЕ /api/* (включая /api/catalog) — это квирк стенда, не баг кода; на dev-сервере ты будешь «чинить» несуществующую поломку. Источник: CLAUDE.md (раздел «Госномера», стенд-квирк).

**2. Запускаешь `npm run build`** → сначала убей живой next-server: `pkill -f next-server` (именно next-server, не "next start"); при подозрении `rm -rf .next`.
Сборка при живом сервере на том же `.next` даёт битую сборку без CSS — выглядит как сломанная вёрстка, хотя код цел. Источник: CLAUDE.md (капканы тайлов 10.07).

**3. Пишешь код под Next.js** → Next.js 16 НОВЕЕ трейнинг-даты: перед кодом читай гайды в `web/node_modules/next/dist/docs/` (после `npm install`).
API отличаются от того, что «помнишь»: например, `priority` у next/image устарел → `preload` (см. живое использование в `web/components/Header.tsx`). Источник: web/AGENTS.md, CLAUDE.md («Как работать»).

**4. Снимаешь скрины Playwright для выводов о вёрстке** → fullPage-скрины ловят from-кадр анимаций `.anim-rise`/`.reveal` (opacity:0): «пустой hero»/«пустая карточка» на них — ФАНТОМ.
Для выводов обязателен живой вьюпорт-скрин, иначе «починишь» то, что не сломано. Источник: CLAUDE.md (капкан скрин-рига, 11.07).

## Кэши и картинки

**5. Меняешь PNG/лого/арт** → новый ассет кладётся под НОВЫМ именем файла (история: logo2.png → logo2-v4.png, splash3-*).
Оптимизатор next/image кэширует варианты в `.next/cache/images` + PWA-кэш браузера: замена файла с тем же именем показывает СТАРУЮ картинку и на скринах, и у пользователей. Источник: CLAUDE.md (капканы 10.07), живой пример `web/public/logo2-v4.png` в `web/components/Header.tsx`.

**6. Перевырезаешь лого из макетов** → выравнивай по bbox СИНЕГО арта (b>120, b−r>60, b−g>40) и по ЦЕНТРУ НАДПИСИ, а не по краям белой плитки.
Кромка белой плитки на белом фоне порогами неразличима — выравнивание по ней уже давало сдвиг надписи на 23.5px между темами. Источник: CLAUDE.md (приём выравнивания артов, 11.07 и 08.07). PWA-заставка (#app-splash в layout.tsx) собрана на splash3-hero(.dark).png / splash3-text(.dark).png; старые `logo.png`/`logo-dark.png` живых ссылок в коде не имеют (упоминание их в комменте Header.tsx устарело — написано до пересборки заставки на splash3-*).

**7. Меняешь ЛЮБУЮ логику разбора импорта по ссылке** → бампни версию кэша парсера: ключ `listimport:cache:vN:` (сейчас v3) в `web/app/api/listings/import/route.ts` (константа `cacheKey`).
Иначе повторный импорт той же ссылки 10 минут после деплоя отдаёт СТАРЫЙ результат — основатель уже принимал это за «не работает». Источник: CLAUDE.md (⚠️ ПРАВИЛО: версия кэша парсера).

**8. Трогаешь `/cars/[slug]` или программатик-лендинги `/buy/*`** → они ОБЯЗАНЫ оставаться `force-dynamic` (`web/app/cars/[slug]/page.tsx:32`).
Пустой generateStaticParams + ISR уже ронял прод 500 на всех карточках. НЕ возвращать ISR на [slug]-маршруты. Источник: комментарий в самом page.tsx + CLAUDE.md.

## Env-переменные

**9. Заводишь/читаешь env, заданную основателем в Vercel** → закладывай ДУБЛЬ-чтение имени без подчёркиваний: `X ?? XNOUNDERSCORE`.
Панель Vercel у основателя «съедает» `_` — так уже ломались демо-код входа и SMS. Живые образцы: `DEMO_OTP_SECRET ?? DEMOOTPSECRET` (web/lib/server/smoke.ts:13), `NIKITA_LOGIN ?? NIKITALOGIN` (web/lib/server/senders.ts:37), `MASHINA_WATCH`/`MASHINAWATCH` (web/lib/server/mashina-watch.ts).

**10. Работаешь с Redis-кредами** → kv.ts читает цепочку `FRAKVURL/FRAKVTOKEN` (Франкфурт, первыми) → `UPSTASH_REDIS_REST_*` → `KV_REST_API_*` (web/lib/server/kv.ts, `upstashUrl`/`upstashToken`). Снять FRAKV* = авто-откат на базу США.
Регионалку Upstash НЕЛЬЗЯ конвертить в global на месте — только новая база + DUMP/RESTORE-перенос. Токен из панели копировать только через 📋 (цепочки «AAAAAA» руками → WRONGPASS); после смены env обязателен свежий деплой. Источник: CLAUDE.md (переезд во Франкфурт).

## Натив (iOS / Android / WKWebView)

**11. Меняешь иконку, заставку, capacitor.config, нативные плагины** → эффект наступит ТОЛЬКО в НОВОЙ нативной сборке (build-ios.yml / build-android.yml); у основателя на телефоне до пересборки старое — это норма, предупреждай заранее.
Апп = Capacitor-оболочка на живом garaj.kg: веб-контент обновляется с деплоем, нативные ассеты — нет. Источник: CLAUDE.md (многократно: иконка 11.07, haptics 05.07, launchShowDuration).

**12. Добавляешь blur/backdrop-blur в стили** → НЕ добавляй. Только transform/opacity.
На части iOS/WKWebView композитные блюр-слои рисуются мимо скругления и ломают отрисовку — блюры уже дважды вычищали из прода (баннер аренды, CarCard/Gallery/JournalList). Источник: web/app/globals.css:487 («НИКАКОГО blur (WKWebView)»), web/components/HomeContent.tsx:282.

**13. Добавляешь внешнюю ссылку** → в нативе `target="_blank"` и `window.open` ТИХО НЕ РАБОТАЮТ — используй openExternal из `web/lib/native-bridge.ts` (tel:/mailto: WKWebView понимает сам).
Глобальный перехват _blank в CapacitorNative.tsx подстрахует забытую ссылку, но новым — мост явно. Источник: web/lib/native-bridge.ts:99.

**14. Ловишь нажатия в нативе (вибро и т.п.)** → через touch-события (touchstart/touchend на document, capture), НЕ click/pointer.
click/pointer до document в iOS-WKWebView доходят ненадёжно — две сборки (404, 405) ушли в TestFlight с неработающей вибрацией. Источник: web/components/CapacitorNative.tsx (глобальные touchstart/touchend-слушатели), CLAUDE.md (⚠️ ПРАВИЛО, сборка 406).

**15. Строишь платную поверхность (цены, тарифы, пополнение, оферта)** → обязателен гейт `useIsNative()` (web/lib/native-bridge.ts:175): в нативе НЕ показываем ни цен, ни ссылок на них, ни «иди на сайт покупать» (anti-steering Apple).
Покупки — только на сайте через банк; эта модель уже прошла ревью Apple (2.1(b)), новая незакрытая поверхность = отказ стора. Источник: CLAUDE.md (⚠️ ПРАВИЛО 06.07 + решение 08.07). Полный guardrail — скилл app-store-lifecycle (важный нюанс там: прятать ПОКУПКУ, не эффект — бейджи ТОП/VIP в аппе видны легально).

**Совет «NSAllowsArbitraryLoads=true» из отладочных гайдов** → только для локальной разработки, в прод НЕ коммитить: выключает App Transport Security целиком и ловится ревью App Store (реджект). Android-аналог `server.cleartext: true` — так же. Источник: security-ревью скилла debugging-capacitor, 11.07.2026.

**15a. Трогаешь iOS Back/edge-swipe/recent/«Показать ещё»** → сначала читать
`IOS-BACK-FLICKER-HANDOFF-2026-07-18.md` и PR #535; контрольная версия — TestFlight
`1.0.2 (426)`, подтверждена основателем на физическом iPhone.
Системный `allowsBackForwardNavigationGestures` ОБЯЗАН оставаться `false`;
обычный свайп реализует custom `UIScreenEdgePanGestureRecognizer` в
`GarajHomeBackSnapshotBridge`. Не возвращать executable `useScrollLock` в
`DetailOverlay`, не переставлять существующий recent slug, не убирать
`garajHomeReturnStateV1`/manual scroll restoration/восстановление раскрытых порций
и не уменьшать compositor settle 120мс без нового покадрового теста на iPhone.
`build-ios.yml` только `workflow_dispatch`; App Store review/release — только по
новому явному разрешению основателя. Источник: build 426, PR #535, handoff 18.07.2026.

## Серверные модули и деньги

**16. Тянешь sharp или другой тяжёлый нативный модуль в route** → только ленивый `await import("sharp")` ВНУТРИ after()/обработчика (образцы: web/lib/server/rehost-photos.ts:85, web/app/api/listings/import/route.ts:1238) + `export const maxDuration = 60` у роутов, где after() делает vision/sharp/Blob.
Статический импорт ронял публикацию 500; без maxDuration платформа молча убивала блюр номеров. Источник: CLAUDE.md («Качество: правила против регрессий»).

**17. Меняешь цены/тарифы** → единственный источник цен — `web/lib/promo-prices.ts` (сервер+клиент), хардкод запрещён; линейку/цены НЕ менять без явного «да» (закреплено: не менять до подключения банка).
Пять тарифных поверхностей сверены с этим модулем QA-прогоном (61 проверка); расхождение = враньё покупателю. Источник: web/lib/promo-prices.ts, CLAUDE.md (QA-прогон 11.07).

**18. Пишешь новый async-путь, который конвертирует Listing→Car или считает салонность** → сначала `await ensureDealerKeys()` (web/lib/server/listings.ts).
Салонность = строго `l.source==="dealer" && isDealerOwner(l.ownerPhone)` (И, не ИЛИ — см. `listingToCar` в listings.ts); холодный serverless-инстанс без ensureDealerKeys видел пустой набор дилеров → «прыгающие салоны» и «202 машины стали автосалоном». Источник: комментарии у `ensureDealerKeys` в listings.ts, CLAUDE.md (корень надёжности салонов).

**19. Трогаешь связку «Дозор mashina» ↔ «Импорт по ссылке»** → инварианты закреплены навсегда и стережёт их `web/test/dozor-contract.test.ts` (CI красный при поломке): LinkImport первым в CRM-«Действиях», имя события только из константы `web/lib/linkimport-event.ts`, scrollIntoView обязателен, сетевой fetch — СНАРУЖИ лока `mashwatch:scan` (web/lib/server/mashina-watch.ts, ⚠️-коммент у лока).
fetch под локом (до 15с) ронял кнопки CRM через LockBusy; менять инварианты = только явное решение основателя + обновить тест. Источник: CLAUDE.md (⭐ ЗАКРЕПЛЕНО НАВСЕГДА).

## React / хуки / UI

**20. Ловишь ошибку react-hooks линтера** → в репо react-hooks плагин v7 (7.1.1 по web/package-lock.json; eslint-config-next 16.2.7 → ^7.0.0): ЗАПРЕЩЕНЫ setState синхронно в теле эффекта и запись ref в рендере; impure-вызовы (Date.now()) — только внутри колбэков setState.
Паттерны-обходы: guarded setState в рендере (сравнение с prev), «latest ref» через useEffect без deps (образец: web/lib/use-closing.ts), опорное «сейчас» через `useState(() => Date.now())`, `setNow(() => Date.now())` в updater-колбэке. Источник: CLAUDE.md (раздел «⚠️ react-hooks v6» — версия в заголовке устарела, сами правила верны), комментарии use-closing.ts.

**21. Делаешь новую шторку/модалку с useClosing** → useScrollLock/useBackToClose ОБЯЗАНЫ вестись от `open && !closing`, не от `open` (образцы: web/components/BrandModelPicker.tsx, OfferBlock.tsx, SaveSearchModal.tsx).
Отложенный на 220мс history.back() проигрывал гонку pushState → сиротские записи истории и лишний «Назад» на Android. Источник: CLAUDE.md (⚠️ ПРАВИЛО, волна 2 плавности).

**22. Добавляешь пункт/ссылку в бургер-меню** → обязателен паттерн `closeMenuForNav` + `<Link replace>` (web/components/Header.tsx, коммент у пунктов меню), НЕ requestCloseMenu.
На iOS history.back() уборки маркера побеждал pushState навигации — «нажимаешь пункт, не переходит». Источник: комментарий в Header.tsx, CLAUDE.md (фикс бургера 08.07).

**23. Трогаешь ленту категорий в SellWizard** → БЕЗ scroll-snap, snap-x НЕ возвращать (комментарий в web/components/SellWizard.tsx:818).
iOS Safari пере-снапывает snap-ленту при любом изменении раскладки (фокус → клавиатура) — строка «скакала» при вводе. Лентам главной/каталога snap оставлен (там жалоб нет). Источник: SellWizard.tsx, CLAUDE.md (⚠️ ПРАВИЛО 10.07).

**24. Добавляешь i18n-ключи** → новая строка = ×4 языка (web/lib/i18n/{ru,ky,zh,en}.ts) + бамп счётчиков паритета в web/test/i18n-parity.test.ts, иначе CI красный.
Процедура и актуальные счётчики (`EXPECTED_DICT`/`EXPECTED_VALUES`) — скилл garaj-i18n-parity.

## Публичные поверхности и решения основателя

**25. Пишешь текст для публичной страницы** → НЕ упоминать Vercel/Upstash/Anthropic/Claude и «AI» как бренд на видимых пользователю страницах (⛔-коммент в web/app/privacy/page.tsx:19).
Решение основателя 11.07.2026 («не хочу чтоб люди узнали что это ии»); возвращать бренды — только с явного «да». Источник: privacy/page.tsx, CLAUDE.md.

**26. Правишь страницу /app** → iPhone/App Store-блок держать ПЕРВЫМ, пока Android-версия не станет реальной (комментарий в web/app/app/page.tsx).
Google переписывает title по контенту: когда первым стоял Android-блок, выдача показывала «Скачать приложение для Android», которого нет. Метадата динамическая (generateMetadata + Blob-проверка APK) — руками title не править. Источник: page.tsx, CLAUDE.md (11.07).

**27. Хочешь смержить в main / изменить видимое поведение / тронуть боевые данные** → только с ЯВНОГО «да»/«мержи» основателя; перед мержем `git fetch` и разбор чужих коммитов (на main часто параллельные сессии).
Это верхнее правило всего репо; параллельная сессия уже делала конфликтующий вариант той же фичи 11.07. Источник: CLAUDE.md (⛔ раздел «Стиль работы с основателем»). Какие именно фразы основателя считаются словом на мерж — скилл garaj-session-protocol.
