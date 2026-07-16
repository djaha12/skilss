---
name: changelog
description: Curate git commits into a polished, user-facing changelog. Use when the user asks to add a changelog entry, draft release notes, turn recent commits into user-facing bullets, close out a release, or backfill historical entries. Produces Keep-a-Changelog style CHANGELOG.md updates (and keeps any machine-readable mirror in sync) by translating dev-speak commits into changes a user would actually notice.
metadata:
  adapted_from: "walterlow/freecut (.claude/skills/changelog), MIT — generalized: removed project-specific file paths, versioning scheme, UI schema, and internal incident references."
---

# Changelog

Turn a stream of commits into release notes a **user** would recognize. The commit log is written from inside the code; the changelog is read from outside by someone deciding whether "what's new" matters to them. Your job is the translation.

There is no auto-append CI assumed. You (or the operator, on request) curate commits into user-facing bullets and update the changelog file(s) together.

## Files to keep in sync

- **`CHANGELOG.md`** — human-facing, [Keep a Changelog](https://keepachangelog.com) style. The source of truth.
- **A machine-readable mirror** (e.g. `changelog.json`, a data module the UI imports) — *only if the project has one*. If it exists, always update it and `CHANGELOG.md` together, never one alone. If the project has no such file, skip this.

Detect the versioning scheme from the existing `CHANGELOG.md` rather than assuming one: **SemVer** (`1.4.0`), **CalVer** (`2026.04.13`, often the Monday of a release week), or plain dated sections. Match whatever is already in use.

## Preconditions — run on every invocation

The default mode is incremental *append*, which trusts the existing changelog to already hold everything prior. That trust breaks silently when a period is skipped: those commits never enter the changelog and nothing flags them. Close that gap by reconciling against git on **every** run, so infrequent triggering is lossless instead of lossy.

1. **Pick the right ref.** Read commits from the branch where the changes actually landed. Work often merges to an integration branch (`develop`/`staging`) before the default branch, so recent commits may not be on `main`/`master` yet. Confirm with `git branch --show-current` or `git log <branch> --no-merges` if unsure. If the user just mentioned a commit you don't see, you're on the wrong branch — re-query.
2. **Reconcile against the log.** Walk `git log <branch> --no-merges --since=<date-of-last-entry>` (or `<last-tag>..HEAD`) and confirm every user-facing commit (after the curation rules below) is represented in the in-progress section or an existing entry. Surface anything missing and fold it in — do not assume the current section is complete just because it has bullets. The reconciliation is against the *curated* log: dropped noise (chore/ci/test/refactor, same-period regressions, follow-ups) is expected to be absent and is not a gap.
3. **Keep headings honest.** If the changelog has a literal "current / unreleased" heading with a date or version, recompute it when you write — never leave it frozen at an old value.

## Modes

The user invokes one explicitly. Default to **draft mode** if unclear.

### Draft mode (most common)

"Draft changelog bullets for this release/week." Produce bullets from commits — do **not** write files yet, just show the curated list for approval.

1. `git log <branch> --no-merges --pretty=format:"%H|%ad|%s" --date=short <range>` — `--no-merges` drops PR-merge commits, which carry no useful subject. Range is usually `<last-tag-or-entry>..HEAD` or an explicit `--since`/`--until`.
2. Apply the **curation rules**. Drop noise, rewrite dev-speak into user language, dedupe revisits.
3. **Verify net state** (see below): drop any Added/Improved bullet whose feature was torn out again before HEAD.
4. Group into Added / Fixed / Improved.
5. Show the result. Wait for approval before touching files.

### Append mode

After the user approves drafted bullets, write them into the in-progress ("Unreleased" / "Current") section.

1. Merge with existing bullets — **dedupe by title**. If a new bullet refines or walks back something already there, edit the existing bullet instead of adding a duplicate.
2. Update the in-progress date/heading if the scheme uses one.
3. Insert new bullets at the top of each group. Mirror into the machine-readable file if one exists.

If any one group already has ~15+ bullets when you arrive, mention it — that's a signal a release cut is overdue, not a license to keep appending.

### Rollup / release mode

Close the in-progress section into a versioned release.

1. Determine the version per the project's scheme (next SemVer, this period's CalVer, or a date).
2. Move the in-progress bullets into a new released entry at the top, with the version and date.
3. Empty the in-progress section (or seed it with anything landed since the cut).
4. Print the commit/tag/push commands for the operator to run manually — don't create tags or push yourself unless asked.

### Backfill mode

Given a git range, produce historical entries. Used when seeding from scratch or filling a gap.

1. Walk merges chronologically: `git log --merges --first-parent <branch> --pretty=format:"%H|%ad|%s" --date=short <range>`.
2. Group by release period. Union all non-merge commits per period.
3. Apply curation, dedupe revisits.
4. Emit one entry per period with at least one user-visible change. Skip empty periods.

## The user-facing test

Before keeping any bullet, ask: **would a user notice this if they used the product today vs. yesterday, without diffing the code?** If no, drop it. This is the single most important filter — apply it to every candidate, not just borderline ones.

A bullet **survives** only if it changes something the user can see, do, or feel:
- A new control, panel, shortcut, format, endpoint, or capability
- A bug they hit (or could hit) is gone
- A workflow that visibly stalled, jittered, or felt sluggish is now smooth in a way they'd remark on

A bullet **fails** if it only describes:
- Internal mechanics (allocation, caching, dispatch, refs, props, type plumbing)
- Sub-pixel visual tweaks (centering, alignment, hover-color shifts, tiny margins)
- Hit-target/drop-zone/ghost-position adjustments — fold into the parent feature
- Same-period regression fixes for code shipped in the same period
- Anything the operator only knows happened because they read the diff

When in doubt, drop it. Fewer bullets beat a wall of text.

## Frame the story: launch vs. increment

Commit subjects are written from the inside, so a series of "increments" ("add Turkish locale", "localize timeline labels", "fix tts strings") can actually be the **launch of a whole system** that didn't exist last release. From outside, that's one headline, not five footnotes.

Before writing bullets for a feature area:

1. Pick the directory/module the commits touch most.
2. `git log --diff-filter=A --since=<period-start> -- <path>` to see when files there first appeared.
3. If most files were **born this period**, it's a launch — write one top-line bullet for the new capability and treat the individual "add X" commits as facets, not separate bullets.
4. If the area has older history, the commits are real increments — describe each that passes the user-facing test.

Lead with the capability ("Translated UI in 9 languages", "Subtitle editing on the timeline"). List supporting facets only if not obviously implied. Resist itemizing every commit just because they share a group.

## Verify the net state, not the commit stream

Commits are a *stream of deltas*; the changelog describes the *net difference* between the last release and HEAD. A feature added Tuesday and ripped out Thursday is, to the user, as if it never shipped — nothing to announce. But a naive draft still emits an "Added" bullet, because the `feat(...)` commit is sitting right there and nothing in the *text* cancels it. The removal happened in the code.

Before finalizing any **Added** or **Improved** bullet, confirm the thing still exists at HEAD:

1. **Find teardown commits in the window.** Scan subjects for `/\b(remove|delete|drop|revert|roll ?back|back out|undo|kill|rip out|disable)\b/i`, and list deletions with `git log <branch> --no-merges --diff-filter=D <range> --name-only`.
2. **Pair each candidate Added bullet against them by feature area, not literal path** — a feature can be dismantled by deleting a route, menu entry, flag, or registry line without deleting the file that introduced it.
3. **Added and removed within the same window → drop the bullet entirely.** Not Added, not Fixed, not Removed. It nets to zero.
4. **When unsure, verify against HEAD directly** — `git cat-file -e HEAD:<path>` for a file, or grep HEAD for the symbol / route / UI label the feature exposes. If it's gone from HEAD, it's gone from the changelog.

**The one exception — removing something that already shipped.** If the removed feature was in a *prior release*, the user has been using it, so its removal is itself a user-facing change worth announcing (plus any migration note). If your format has no "Removed" group, surface it to the user for wording rather than silently dropping it. The added-then-removed-same-window case is the opposite: the user never had it, so nothing is announced.

## Curation rules

### Drop

- Merge commits (`Merge pull request`, `Merge branch`)
- `chore(...)` (including `chore(release)`), `ci(...)`, internal `refactor(...)` (stores, types, utils, deps adapters, chunk splits)
- `test(...)` unless it documents notable test-infra changes
- `deps` / `deps-dev` bumps unless a major bump with user-visible impact
- **Follow-up fixes** — messages matching `/address.*(review|PR|findings|feedback)|code review|follow-?up|fix.*(lint|typecheck|build|pre-existing)/i` that land in the same period as a parent feature. Roll into the parent bullet silently.
- **Same-period regression fixes** — fixes for bugs introduced by other commits in the same period. The user never saw the regression; only the final state matters.
- **Visual polish micro-tweaks** — recentering, hover-color changes, padding nudges no user notices discretely. If a feature has 3+ such tweaks, the parent bullet absorbs them.
- **Internal perf on this period's new work** — the user experiences the feature once, smoothly. No separate "Improved" bullet.
- Reverts paired with a subsequent re-fix in the same period — keep only the final correct implementation.
- Revisits: the same feature improved several times in one period → one bullet describing the final state.
- Duplicates worded differently → one bullet.

### Keep and rewrite

- `feat(...)` — when user-perceivable (a new control, mode, format, shortcut, panel). Skip internal `feat(...)` that only changes plumbing.
- `fix(...)` — only if the bug was user-observable (rendering glitch, playback hitch, data loss, crash, wrong output) **and** shipped to users before the fix. Skip fixes for code never released, or shipped-and-corrected in the same period.
- `perf(...)` — only if the user would describe it in their own words ("playback is smoother"). Skip perf the user can't perceive without instruments.

### Rewrite style

Commit messages are dev-speak. Rewrite every bullet.

| Commit subject | Changelog bullet |
|---|---|
| `feat(timeline): add Alt+C as alternate split-at-playhead shortcut` | Split clips at playhead with Alt+C |
| `perf(filmstrip): fill zoom gaps with cover frame background` | Smoother filmstrip rendering when zooming |
| `fix(preview): retry with fresh blob URL on stale-blob load errors` | Preview no longer fails when media blobs expire |
| `feat(storage): migrate to workspace folder via File System Access API` | Projects now live in a folder you choose, not hidden browser storage |

Rules of thumb:
- Lead with the verb of the user experience, not the code change.
- Drop internal names (stores, modules, workers) unless the user knows them.
- If a bullet is only meaningful to developers, drop it.
- Aim for ≤12 words per bullet; prefer thematic phrasing over commit-level phrasing.
- **Sentence case** — start every bullet with a capital letter; ship the fixed string, don't rely on a UI fallback.

### Grouping

- **Added** — new user-facing features
- **Fixed** — bugs the user actually hit in shipped builds
- **Improved** — performance or polish wins the user would describe in their own words

Skip any empty group. **It is normal and good for "Improved" to be empty** — do not pad it with internal perf to look productive.

## When in doubt

- Fewer bullets > more bullets. A wall of text is worse than missing a tiny fix.
- 100 commits that all revisit the same 3 features → 3 bullets.
- Unsure whether a commit scope is user-visible? Check the files it touches: UI components and public APIs = user-visible; stores/utils/tests = usually not.
- "Update the changelog" with no other context → default to **draft mode**: show curated bullets, write nothing until approved.
