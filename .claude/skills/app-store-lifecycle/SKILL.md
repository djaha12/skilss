---
name: app-store-lifecycle
description: GARAJ.KG app-store lifecycle playbook (Apple App Store / Google Play / RuStore / Samsung) — MUST be loaded for ANY work on store submissions, rejections, Resolution Center replies, iOS/Android release builds, version bumps, or store listings; it encodes the 3 real Apple rejection precedents and the anti-steering guardrail that got the app approved. Triggers — "App Store", "Apple rejected", "rejection", "Resolution Center", "TestFlight", "App Store Connect", "Google Play", "RuStore", "Samsung", "закрытое тестирование", "отказ Apple", "эпл отклонил", "отклонили приложение", "ответ ревьюеру", "ревью приложения", "собери iOS", "собери айос", "новая версия приложения", "билд в стор", "выложить приложение", "обновление приложения", "апстор", "гугл плей", "рустор".
---

# App Store / Google Play / RuStore / Samsung — жизненный цикл приложения GARAJ.KG

Приложение — Capacitor-оболочка живого сайта garaj.kg (`mobile/capacitor.config.ts`,
appId `kg.garaj.app`). Апп = прод-сайт: контент меняется с каждым деплоем веба,
номер билда на контент не влияет. Это значит: ревьюер Apple видит ТЕКУЩИЙ прод
в момент проверки — любая платная поверхность без гейта проваливает ревью,
даже если билд «старый».

## Текущее состояние (сверено 11.07.2026)

⚠️ Этот раздел — снимок на дату сверки и устаревает первым. При расхождении
с большими доками — верить докам и свежим записям CLAUDE.md, они первоисточник.

- **App Store — ОПУБЛИКОВАНО.** Версия 1.0 (билд 408) одобрена 08.07.2026 после
  трёх кругов ревью: https://apps.apple.com/app/garaj-kg/id6787182273.
  Обновление 1.0.1 (иконка+заставка) собрано и отправляется.
- **Google Play — НЕ опубликовано.** Аккаунт оплачен ($25), впереди обязательный
  закрытый тест 12 тестеров × 14 дней (личный аккаунт). План — `GOOGLE-PLAY-KG-PLAN.md`.
- **Промежуточные Android-каналы:** прямой подписанный APK на garaj.kg/app
  (`build-apk-release.yml`, нужен секрет `BLOB_READ_WRITE_TOKEN`; страница
  `web/app/app/page.tsx` сама покажет Android-блок, когда garaj.apk ляжет в Blob),
  RuStore (`RUSTORE-LISTING.md`) и Samsung Galaxy Store (`SAMSUNG-LISTING.md`).

## Три реальных отказа Apple — прецеденты (НЕ повторять ошибки)

**1) Билд 399 → Guideline 5.1.2(i)** — ревьюер увидел куки-плашку и потребовал
App Tracking Transparency, хотя рекламного трекинга нет (only first-party).
Что починило: `CookieNotice` скрыт в нативе гейтом `useIsNative` (веб не тронут).
ATT не внедряли — не нужен. ⛔ НЕ возвращать куки-плашку в натив.

**2) Билд 407 → сначала прочитан как 2.1.0 App Completeness** («не работает»).
Гипотеза «белый экран / ревьюер не смог войти» породила фиксы (безвредные,
оставлены в проде): `launchShowDuration` 750→2200 и `errorPath: "index.html"`
в `mobile/capacitor.config.ts`, ссылка «Скачать приложение» скрыта в нативе.
Урок: **не гадать по коду отказа — нести в чат ДОСЛОВНЫЙ текст Apple.**

**3) Тот же отказ расшифрован → Guideline 2.1(b) Information Needed** —
бизнес-модель: «похоже, приложение содержит платный цифровой контент — объясните».
5 вопросов про платные фичи/IAP/платный ли аккаунт. Это НЕ баг: ревью на паузе,
нужен ТОЛЬКО ответ в Resolution Center, новый билд не требуется. Что сработало
(дословный текст — `APP-STORE-2.1b-REPLY.md`): «приложение бесплатное, IAP нет,
аккаунт бесплатный (телефон+код), платная реклама — только на сайте garaj.kg
в браузере, оплата через банк». После этого ответа — одобрение 08.07.2026.

Заготовки на СЛЕДУЮЩИЕ отказы (3.1.1, «нашли платное», 2.1, 4.2 «обёртка сайта»,
5.1.x) — `APP-STORE-REJECTION-PLAYBOOK.md`. Разбор 2.1.0 + готовые Review Notes
и текст видео входа — `APP-STORE-RESUBMIT-408.md`.

## Как писать ответы в Resolution Center (структура реальных прошедших)

1. **Английский, коротко, по-человечески, без споров и юристов.** Начать с
   благодарности («Thank you for the review / the questions»), затем факты.
2. На нумерованные вопросы Apple — нумерованные короткие ответы (см. эталон
   в `APP-STORE-2.1b-REPLY.md`: абзац позиции + «Short answers: 1..5»).
3. **Позицию не менять никогда:** «в приложении ничего платного нет» — это
   и правда, и условие прохождения (всё платное за `useIsNative`).
4. «Information Needed» = пауза, нужен только ответ. «Rejected» = как правило
   ресабмит. ⚠️ Капкан App Store Connect: кнопка ресабмита СЕРАЯ, пока не
   изменён объект (новый билд или правка полей версии) — ответ в треде сам
   её не активирует.
5. При отказе про вход/работоспособность — прикладывать видео входа 20–30 с
   с iPhone (сценарий — `APP-STORE-RESUBMIT-408.md` п.5).
6. Бейджи ТОП/VIP на карточках = метки «Sponsored» для покупателей, в аппе
   их купить нельзя — готовый текст в `APP-STORE-REJECTION-PLAYBOOK.md` сцен. B.

## Anti-steering guardrail (главное правило кода)

Детект оболочки — `useIsNative()` в `web/lib/native-bridge.ts`: в браузере
false (сайт как был), в Capacitor true → платное прячется. Уже за гейтом:
`web/components/SellPromoOffer.tsx`, `web/components/cabinet/PromoTab.tsx`
(бесплатное «Поднять» оставлено), `web/components/cabinet/TopupPanel.tsx`,
`web/components/CookieNotice.tsx`, страница /offer через
`web/components/OfferNativeGate.tsx`, ссылки подвала на /promo и /offer.

- ⚠️ **КАЖДАЯ новая платная поверхность (цены, тарифы, «Пополнить», оферта,
  ссылки на них) ОБЯЗАНА иметь гейт `useIsNative`** — иначе следующий же
  ревьюер апдейта её увидит на живом проде и завернёт ревью.
- «Иди на сайт покупать» из аппа — тоже НЕЛЬЗЯ (anti-steering: Apple запрещает
  даже упоминание покупки на сайте). Нейтрально: «Продвижение недоступно
  в приложении».
- Эффект покупки (бейджи ТОП/VIP, позиция в топе) в аппе виден — это легальная
  модель «куплено в другом месте». Прятать нужно ПОКУПКУ, не эффект.
- Гейты не снимать без явного решения основателя; после новых фич — grep по
  ценовым/тарифным компонентам без `useIsNative`.

## Версии и сборка iOS (`.github/workflows/build-ios.yml`)

- **Версия для стора** = env `IOS_VERSION_NAME`, захардкожена в build-ios.yml
  (сейчас `"1.0.1"`) → `mobile/scripts/inject-ios-version.mjs` пишет её
  в MARKETING_VERSION. **После каждого выпуска в стор новое обновление требует
  НОВОЙ версии** — поднять `IOS_VERSION_NAME` в воркфлоу перед сборкой апдейта.
- **Номер билда** = `IOS_BUILD_NUMBER` = `github.run_number` (уникален на прогон)
  → CURRENT_PROJECT_VERSION. ASC отклоняет аплоад с уже существующим номером —
  поэтому номер никогда не задавать руками.
- Запуск: Actions → build-ios → Run workflow (только workflow_dispatch, ветка
  main). Раннер macos-15 — ASC с 2026 принимает только билды на iOS 26 SDK.
- ⚠️ Бамп `IOS_VERSION_NAME` действует только ПОСЛЕ попадания в main (dispatch
  запускается с main), а мерж в main — ТОЛЬКО по явному слову основателя
  (garaj-session-protocol §2). НЕ пушить в main ради одного бампа версии.
- iOS-проект генерируется в CI (`npx cap add ios`) и в репо НЕ лежит — все
  правки Info.plist/версий только через `mobile/scripts/inject-*.mjs`.

## Pre-flight перед сборкой/подачей (экономит деньги и круги ревью)

1. **Секреты до macOS-минут:** macOS-раннеры на приватном репо ×10 к Linux.
   В build-ios.yml уже есть job `secrets-check` на ubuntu — валидирует формат
   p12/profile/p8 ДО macOS. 7 секретов Apple перечислены в шапке build-ios.yml
   и в `APP-STORE-RELEASE-CHECKLIST.md` §3. Если подписи нет — воркфлоу молча
   делает только симулятор-сборку (IPA не будет!): проверять, что
   `APPLE_DISTRIBUTION_CERT_P12_BASE64` задан.
2. **Демо-вход жив:** Actions → `probe-demo-login.yml`, ждать `via:"demo"`.
   Демо-номер ревьюера 996700000001 (env `DEMO_OTP_PHONES ?? DEMOOTPPHONES` —
   дубль-имя из-за панели Vercel, см. garaj-known-traps №9; код показывается
   на экране оранжевой плашкой (стиль warn, web/components/auth/LoginFlow.tsx), SMS не шлётся) — НЕ отключать: нужен для ревью
   каждого обновления. Review Notes с шагами входа — `APP-STORE-RESUBMIT-408.md` п.3.
3. **Гейты целы:** CookieNotice / вкладка «Продвижение» / тарифы / оферта /
   баланс скрыты в нативе (чек-лист QA — `APP-STORE-RELEASE-CHECKLIST.md` §13).
4. **Позиционирование карточки — чистый классифайд:** НЕ обещать «безопасную
   сделку / проверку / гарантию / историю авто» (фичи за выключенными флагами,
   `web/lib/flags.ts`) и НЕ упоминать платное продвижение в iOS-текстах.
5. После аплоада основателю в ASC: дождаться билда в TestFlight → создать/выбрать
   версию → прикрепить билд → Review Notes с демо-входом → Add for Review.

## Android-сборки

- `build-android.yml` — debug APK (артефакт `garaj-android-apk`), для ручной установки.
- `build-android-release.yml` — ОСНОВНОЙ воркфлоу подписанного AAB для Google
  Play/RuStore/Samsung: подписывает постоянным ключом из 4 секретов
  (`GOOGLE_KEYSTORE_BASE64`, `KEYSTORE_PASSWORD`, `KEY_ALIAS`, `KEY_PASSWORD`).
- `build-android-aab.yml` — бутстрап/запаска (секретов ещё нет или keystore-секрет
  битый): генерит СВЕЖИЙ keystore на каждом прогоне. ⚠️ После первой загрузки
  в Play им больше НЕ собирать — Play привязывает подпись навсегда и отвергнет
  AAB с новым ключом. Ритуал после прогона (как в actions-as-hands): скачать
  артефакт `garaj-keystore-SECRET`, вписать из него 4 секрета,
  УДАЛИТЬ артефакт со страницы запуска — дальше собирать build-android-release.
- `build-apk-release.yml` — подписанный APK → Vercel Blob → garaj.kg/app
  (требует секрет `BLOB_READ_WRITE_TOKEN`); тоже генерит свой ключ —
  тот же keystore-ритуал.
- Версионирование Android — versionCode в `mobile/scripts/inject-signing.mjs`
  (та же логика «номер = run_number», что и iOS).

## Где лежат листинг-паки (не дублировать — читать при работе с конкретным стором)

| Стор | Документы |
|---|---|
| Apple App Store | `APP-STORE-SUBMISSION-PACK.md` (copy-paste полей ASC), `APP-STORE-RELEASE-CHECKLIST.md` (полный гид §0–14: секреты, App Privacy, скриншоты, QA), `APP-STORE-LAUNCH-DOSSIER.md` (аудит рисков) |
| Отказы Apple | `APP-STORE-REJECTION-PLAYBOOK.md`, `APP-STORE-RESUBMIT-408.md`, `APP-STORE-2.1b-REPLY.md` |
| Google Play | `GOOGLE-PLAY-KG-PLAN.md` (путь из КР: личный аккаунт, тест 12×14), `PLAY-LISTING.md` (тексты + Data safety), `PLAY-RELEASE-CHECKLIST.md` |
| RuStore | `RUSTORE-LISTING.md` (регистрация иностранца-физлица, AAB подписывать самим) |
| Samsung Galaxy Store | `SAMSUNG-LISTING.md` (Commercial Seller, тот же AAB) |

Все пути — от корня репо. При расхождении этого файла с большими доками —
верить докам и свежим записям CLAUDE.md, они первоисточник (особенно для
раздела «Текущее состояние» выше).
