---
name: garaj-i18n-parity
description: >-
  MANDATORY procedure for adding or changing ANY user-facing string in GARAJ.KG —
  every string must land in all 4 language dictionaries (ru/ky/zh/en) and the
  parity counter must be bumped, or CI fails (exception — server notification
  texts live in a separate dictionary NOT covered by the parity test; the skill
  covers both cases). Use it EVERY time you add a button, label, error message,
  toast, notification text, or translation. Triggers — "add a string", "new
  label", "translate", "i18n", "parity test failed", "EXPECTED_DICT", "добавь
  текст", "добавь кнопку/надпись", "переведи", "перевод", "локализация", "новая
  строка", "текст на кыргызском/китайском/английском", "упал тест i18n", "не
  хватает ключа".
---

# GARAJ.KG: добавление пользовательской строки (i18n parity)

## Устройство (почему так)

- 4 языка: `ru`, `ky`, `zh`, `en` (тип `Lang` в web/lib/i18n.tsx).
- **RU — база и источник истины.** web/lib/i18n/ru.ts импортируется СИНХРОННО в web/lib/i18n.tsx (SSR, первый кадр, фолбэк). ky/zh/en грузятся лениво отдельными чанками — чтобы ~44 КБ gzip чужих переводов не летели в first-load. Поэтому забытый ключ в ky/zh/en НЕ падает в рантайме — там молча покажется русский текст. Единственная защита — parity-тест.
- В каждом словаре web/lib/i18n/{ru,ky,zh,en}.ts два экспорта:
  - `dict` — строки интерфейса, ключи вида `"nav.buy"`. Читаются через `t(key)`.
  - `values` — переводы ОТОБРАЖАЕМЫХ значений данных (топливо, кузов, город, цвет…), ключ = само русское слово (`"белый"`). Читаются через `tv(value)`. **`tv` меняет только показ** — в объявлениях, фильтрах и БД значение остаётся русским. Никогда не переводи данные при записи.

## Процедура добавления строки (чек-лист)

1. Добавь ключ + русский текст в web/lib/i18n/ru.ts (в `dict` или `values` — см. выше).
2. Добавь ТОТ ЖЕ ключ с переводом в web/lib/i18n/ky.ts, web/lib/i18n/zh.ts, web/lib/i18n/en.ts — в том же месте файла (файлы синхронны ключ-в-ключ). Пустая строка перевода = провал CI.
3. Подними счётчик в web/test/i18n-parity.test.ts:
   - `EXPECTED_DICT` — число ключей `dict`,
   - `EXPECTED_VALUES` — число ключей `values`.
   Актуальные числа НЕ бери из этого скилла — смотри их в самом web/test/i18n-parity.test.ts (для примера: на 11.07.2026 это 1467 и 66). Счётчик живёт В САМОМ тест-файле, не в словарях. Он нужен, чтобы ловить случайную потерю целого блока переводов. В комментарии к `EXPECTED_DICT` принято дописывать что добавлено и дату, по образцу существующего: `// 1465 + 2: appbar.iosTitle/iosBtn — ... (10.07.2026)`.
4. Запусти тест из web/:
   ```bash
   cd web && npx vitest run test/i18n-parity.test.ts
   ```
   (полный прогон: `cd web && npm test`). Тест называется `i18n parity`.
5. В компоненте используй `const { t, tv } = useLang()` из web/lib/i18n.tsx. Не хардкодь русский текст в JSX.

## Уведомления (СМС/пуш/WhatsApp/Telegram) — ДРУГОЙ словарь

Серверные тексты уведомлений живут в web/lib/server/notify-i18n.ts, НЕ в клиентских словарях. Там объект `N: Record<string, Entry>`, где `Entry = Record<Lang, string>` — все 4 языка в одной записи. Читается через `nt(key, lang, vars)`; плейсхолдеры `{x}` подставляются ПОСЛЕ выбора строки, фолбэк на `ru` встроен. Язык получателя берётся из web/lib/server/user-lang.ts по номеру. Parity-тест этот файл НЕ проверяет — заполняй все 4 языка вручную и внимательно.

## Частые провалы

- **Забыл язык** → `dict key set === ru dict key set` красный для этого языка. Diff набора ключей покажет, какой ключ где потерян.
- **Забыл поднять `EXPECTED_DICT`/`EXPECTED_VALUES`** → `ru has the expected key counts` красный. Не подгоняй число вслепую — сначала убедись, что разница = ровно твои новые ключи.
- **Пустой перевод** (`""`) → `has no unexpected empty-string dict values` красный. Осознанно пустые переводы (у слова нет эквивалента) белятся в `ALLOWED_EMPTY_DICT` в web/test/i18n-parity.test.ts (текущий состав списка смотри там же, в самом тест-файле). Не добавляй туда ключи, чтобы «заткнуть» тест — это только для реально отсутствующих эквивалентов.
- **Положил ключ в `dict` в одном языке и в `values` в другом** → падают оба key-set теста. `dict` и `values` — раздельные наборы.
- **Перевёл значение данных при сохранении** → фильтры и поиск ломаются: данные всегда хранятся по-русски, переводится только показ через `tv`.
