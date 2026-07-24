# Что доставить в рабочий репозиторий auto-superapp-kg

На 24.07.2026 в библиотеке 91 скилл, в рабочем репозитории — 56. Ниже — 35 скиллов, которых там нет,
разложенные по трём уровням. **Не нужно копировать все 35**: библиотека и так на грани, где описания
скиллов начинают конкурировать между собой и агент выбирает хуже. Уровень 3 копировать не надо вовсе.

Скиллы берутся из этого репозитория (`djaha12/skilss`), каталог `.claude/skills/`.

---

## Уровень 1 — обязательно (12)

Эти либо чинят уже сломанное, либо закрывают боль, которая в проекте повторялась.

**Сломанные ссылки — проверено по файлам в `auto-superapp-kg/.claude/skills`:**

- `subagent-driven-development/SKILL.md:65,80,81,270` ссылается на файл
  `../requesting-code-review/code-reviewer.md` — **такого пути в проекте нет**, скилла нет.
  Та же схема требует `superpowers:finishing-a-development-branch` — тоже нет.
- `executing-plans/SKILL.md:35-36` — «**REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch». Нет.
- `wayfinder/SKILL.md:25,78,79,111,123` вызывает `/setup-matt-pocock-skills`, `/prototype`, `/grilling`,
  `/domain-modeling` — ни одного из четырёх в проекте нет.
- твой собственный `apply-skills/SKILL.md:54,61` в таблице маршрутизации называет `doc-coauthoring`,
  `marketing-ideas`, `marketing-plan`, `customer-research` — их там нет, диспетчер указывает в пустоту.

| Скилл | Зачем именно в GARAJ |
|---|---|
| `requesting-code-review` | Битая ссылка в `subagent-driven-development` — мультиагентные аудиты доходят до финального ревью и упираются в отсутствующий файл |
| `finishing-a-development-branch` | Объявлен обязательным под-скиллом в `executing-plans`. Плюс 92 удалённых ветки, 38 не влиты, мержи «Already up to date» |
| `receiving-code-review` | Вторая половина цикла ревью: как принимать замечания через проверку, а не слепое согласие |
| `setup-matt-pocock-skills` | Разовая настройка трекера. Без неё `to-tickets`, `implement` и `wayfinder` не работают как задумано |
| `grilling` | Согласование до кода. За 12 дней 21 откат — сделали, показали, откатили байт-в-байт |
| `domain-modeling` | Формат ADR. Нужен под нерешённое: «мигрируем на кэш-модель Next 16 или фиксируем, что не мигрируем». Плюс ~40 префиксов ключей KV нигде не описаны |
| `prototype` | Показать вариант дёшево, вместо ветки + CI + деплоя (10–15 мин на каждый эксперимент с флагом) |
| `to-tickets` | Аудит 29.06 дал 31 находку: 11 починено, 20 висят в MD-файле без статуса. Среди них тикающие (id объявления 8 hex, ~50% коллизий при 77k лотов) |
| `implement` | Замыкает цикл: задача из трекера → работа |
| `handoff` | Передача контекста в свежую сессию. За 5 дней 4 раза теряли работу: пустая ветка, старый снапшот, осиротевший фикс биллинга |
| `research` | Изучение по первоисточникам. `web/AGENTS.md` прямо предупреждает «This is NOT the Next.js you know» — 16.2.7 новее, чем знает модель |
| `skill-creator` | Чтобы чинить свои 9 GARAJ-скиллов (в них протухшие факты) и писать те 5, которых в экосистеме нет вообще |

⚠️ Три скилла из superpowers (`requesting-code-review`, `receiving-code-review`,
`finishing-a-development-branch`) у нас лежат срезом v6.1.1 от 02.07, а 23.07 вышла v6.2.0, где изменились
все 10 скиллов набора. Если будешь ставить — логичнее взять сразу из upstream
[obra/superpowers](https://github.com/obra/superpowers) и обновить обе копии разом.

---

## Уровень 2 — по делу (15)

Не горит, но работа под них в репозитории реально идёт.

| Скилл | Зачем |
|---|---|
| `doc-coauthoring` | Русские ранбуки и спеки; на него ссылается твой `apply-skills` |
| `customer-research` | Исследование клиентов, JTBD, майнинг отзывов; тоже в таблице `apply-skills` |
| `marketing-plan`, `marketing-ideas` | В таблице `apply-skills`. Оговорка: `marketing-plan` сам ссылается на 5 скиллов, которых нет ни в одной копии (`emails`, `onboarding`, `product-marketing`, `referrals`, `signup`) |
| `copy-editing` | На него ссылаются установленные `copywriting`, `ad-creative`, `marketing-council` |
| `launch` | Плейбук запуска; впереди Google Play, RuStore, Samsung |
| `docx`, `pdf`, `pptx`, `xlsx` | Паки для сторов, досье, отчётность основателю. ⚠️ Наши копии `docx`/`pptx`/`xlsx` устарели: upstream переписал их 17.07 и удалил файлы, на которые наши копии ссылаются — брать сразу из [anthropics/skills](https://github.com/anthropics/skills) |
| `grill-me`, `grill-with-docs` | Команды поверх `grilling`; если ставишь `grilling`, эти два дополняют его |
| `brainstorming` | Генеративная идеация до кода. ⚠️ У локального сервера-компаньона маячок на primeradiant.com, отключается `SUPERPOWERS_DISABLE_TELEMETRY=1` |
| `find-skills` | Поиск и установка скиллов из экосистемы по запросу |
| `analytics` | ⚠️ Польза ограничена: скилл про GA4/GTM/Mixpanel/Segment, а у тебя своя аналитика на KV и ноль пикселей в коде. Брать только если решишь ставить внешнюю аналитику |

---

## Уровень 3 — не нужно (8)

Копировать не стоит: работы под них в проекте нет, а место в каталоге они займут и будут конкурировать
описаниями с нужными скиллами.

| Скилл | Почему нет |
|---|---|
| `brand-guidelines` | Это фирменный стиль **Anthropic** — цвета и типографика чужого бренда. Для GARAJ вредно |
| `internal-comms` | Внутрикорпоративные коммуникации, статус-репорты команде. Ты работаешь один |
| `mcp-builder` | Создание MCP-серверов. Пока не строишь свой — не нужен |
| `web-artifacts-builder` | Сложные HTML-артефакты под claude.ai, а не под прод-сайт на Next.js |
| `canvas-design`, `algorithmic-art`, `theme-factory`, `slack-gif-creator` | Визуальный дизайн, генеративное искусство, темы, GIF для Slack — не профиль проекта |

---

## Как доставить (когда дойдут руки)

Из рабочего репозитория `auto-superapp-kg`:

```bash
git clone --depth 1 https://github.com/djaha12/skilss /tmp/skilss

# Уровень 1 — обязательный минимум
for s in requesting-code-review receiving-code-review finishing-a-development-branch \
         setup-matt-pocock-skills grilling domain-modeling prototype \
         to-tickets implement handoff research skill-creator; do
  cp -r "/tmp/skilss/.claude/skills/$s" .claude/skills/
done

# Уровень 2 — по желанию, добавляй точечно
for s in doc-coauthoring customer-research marketing-plan marketing-ideas copy-editing \
         launch docx pdf pptx xlsx grill-me grill-with-docs brainstorming find-skills; do
  cp -r "/tmp/skilss/.claude/skills/$s" .claude/skills/
done
```

После копирования — разовая настройка трекера, иначе `to-tickets`/`implement`/`wayfinder` работают вслепую:

```
/setup-matt-pocock-skills
```

Проверка, что доехало и ничего не сломалось:

```bash
ls .claude/skills | grep -v '^LICENSE' | wc -l     # было 56, станет 68 (ур.1) или 82 (ур.1+2)
diff -rq /tmp/skilss/.claude/skills/handoff .claude/skills/handoff   # выборочная сверка
```

---

## Обратное направление — уже сделано

9 кастомных GARAJ-скиллов (`actions-as-hands`, `garaj-known-traps`, `kv-data-patterns`,
`app-store-lifecycle`, `submit-appstore`, `garaj-session-protocol`, `garaj-i18n-parity`,
`founder-runbook-writer`, `garaj-content-week`) перенесены в библиотеку 24.07.2026 — теперь у них
есть вторая копия. В них найдены протухшие факты, разбор — в [SKILLS-RESEARCH-2026-07-24.md](SKILLS-RESEARCH-2026-07-24.md).
