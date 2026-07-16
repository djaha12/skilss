---
name: translate-app-locales
description: Add or audit full application translations across base locale files and partial i18n namespaces. Use when the user asks to add a new language, fix incomplete translations, remove English fallbacks, verify locale coverage, or make a whole app section translatable — in React/TypeScript (or similar) apps that use JSON locale files, e.g. an i18next-style `locales/` directory plus per-namespace `partials/`.
metadata:
  adapted_from: "walterlow/freecut (.codex/skills/translate-app-locales), MIT — generalized: converted from an OpenAI Codex skill to a Claude Code skill, removed the project-specific locale map and a hardcoded author machine path. The bundled coverage script is upstream, unchanged."
---

# Translate App Locales

Use this skill for end-to-end language work — the whole locale tree, not a single JSON file.

## Workflow

1. **Locate the i18n wiring.** Find the entrypoint, the supported-language registry, the base locale files, and any partial/namespace files. See "Locating the structure" below.
2. **Register the language.** Add it to the supported-language registry (with direction, native name, English name if the registry tracks them) and import/load it wherever base locales are wired.
3. **Create the base locale file** for the new language.
4. **Translate every partial namespace** that has source-language keys, including any catch-all or "missing"/"extra" file — those often hold live UI.
5. **Preserve the source key tree exactly** for the target language — same keys, same nesting.
6. **Preserve non-prose tokens** (see Translation Rules): interpolation placeholders, ICU/plural suffixes, HTML tags, component placeholders, URLs, command names, code identifiers.
7. **Do not remove UI settings/keys** as part of translation — only when the user explicitly asks for a product-behavior change.
8. **Validate** JSON parsing, key coverage, and the app build or relevant tests.

## Locating the structure

Adapt to the repo; typical i18next-style layout:

- **Language registry** — a `languages.ts`/`.js` (or config) exporting the list of supported languages, direction, and display names.
- **Base locales** — `locales/<lang>.json` (or `<lang>/translation.json`): app shell, settings, common strings. Find where these are imported and aggregated.
- **Partial namespaces** — `locales/partials/*.json` or `locales/<lang>/<namespace>.json`: one file per feature area. Don't skip a `missing.json` / `extra.json` catch-all; it may contain active export/media/preview/timeline UI.
- **Loader** — how the app resolves a key to a string (i18next, a custom loader, or static imports). This tells you which files are actually reachable.

The rule regardless of layout: **the target language must cover every source-language leaf key that the app's loader can load.** Grep for the loader/import wiring rather than assuming a directory shape.

## Translation rules

- Translate user-facing prose naturally in the target language.
- Keep product/brand names unchanged unless the existing locales already translate them.
- Keep placeholders **byte-for-byte**: `{{count}}`, `{{name}}`, `%s`, `<code>`, `<strong>`, `<link>`, `<accent>`, and similar markers.
- Keep plural variants as separate keys: `_one`, `_other`, `_few`, etc.
- Keep technical acronyms when normal in the target language: FPS, GPU, URL, MP4, AAC.
- Prefer concise UI labels over literal long translations when button space is tight.
- For short language chips/toggles, use two-letter uppercase base codes (`EN`, `ZH`, `FR`, `TR`) unless the product has another standard.
- Preserve valid UTF-8. Non-ASCII is expected in translated locale files.

## Coverage validation

Use the bundled script after editing locale files. It verifies that JSON parses, that the target base locale has every source base leaf key, and that each partial's target language has every source leaf key.

```bash
node <skill>/scripts/check-locale-coverage.mjs \
  --locales <locales-dir> \
  --partials <partials-dir> \
  --source en \
  --target <lang>
```

- Omit `--partials` if the project has no partial directory.
- `--source`/`--target` are the locale codes (base filenames for base locales; the top-level language keys inside each partial JSON).
- Exit non-zero means missing keys — the script prints each missing dotted path.

## Verification

Run the repo's fastest meaningful checks:

- The coverage script for the target language.
- JSON parse validation if the project has custom locale loading.
- Focused tests for settings/language persistence when language controls changed.
- Build/typecheck when translations touch app wiring or TypeScript imports.

Report unrelated failures clearly — do not hide them as translation failures.
