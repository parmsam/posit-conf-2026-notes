# AGENTS.md

Notes for coding agents working in this repo.

## What this is

Personal notes from posit::conf(2026) virtual, kept as Quarto documents.

- `day1.qmd`, `day2.qmd` — the actual conference notes (one file per day). `day2.qmd` is still mostly empty as the conference progresses.
- `README.md` — links to the day files and how to browse the Quarto site.
- `knowledge/` — clean-markdown copies of URLs referenced in the day files, fetched locally, plus `knowledge/index.md` as the index. Populated by the `qmd-url-defuddle` skill; don't hand-edit its contents. Only the per-article subfolders (`knowledge/day1/`, `knowledge/day2/`, …) and `*.pdf` are gitignored — `knowledge/index.md` itself is tracked and committed, since `knowledge.qmd` includes it and the GitHub Pages build (see "Deployment" below) needs it present.
- `m3u8/` — gitignored. Session recording HLS manifests (`.m3u8`), saved manually from the browser (signed URLs expire).
- `subtitles/vtt/`, `subtitles/markdown/` — gitignored. WebVTT subtitle files and flowing-text transcripts derived from `m3u8/`, one pair per session, named `<day>-<kebab-case-session-title>`. Populated by the `m3u8-subtitles` skill; don't hand-edit. Not part of the Quarto site (see "Website" below) — kept as local files only.
- `_quarto.yml`, `index.qmd`, `knowledge.qmd` — the Quarto website that ties all of the above together for browsing (see "Website" below). `_site/` (render output) and `.quarto/` (cache) are gitignored.
- `study.qmd`, `study-flashcards.qmd`, `study-quiz.qmd` — the "Study Materials" tab: a landing page plus a flashcards deck and a quiz deck (both RevealJS, via the `_extensions/parmsam/flashcards` and `_extensions/parmsam/quiz` extensions) hand-written from `day1.qmd`. Refresh them when Day 1 notes gain new sections — see "Website" below for extension details.

## Conventions

- Keep `day1.qmd` / `day2.qmd` as plain running notes — bullet points with bare URLs are fine, they don't need to be markdown links (the URL-extraction tooling handles both). Bare URLs still render clickable on the website because `_quarto.yml` sets `format.html.from: markdown+autolink_bare_uris` — don't remove that setting, or every bare URL in the day notes goes dead as plain text.
- **Structure each talk's notes with `###` subsections** (e.g. setup/context, the main demo or study, key takeaways, closing) instead of one long flat bullet list under the talk's `##` heading — makes a talk's notes skimmable at a glance. A talk that's just a placeholder link (not yet attended/transcribed) doesn't need subsections yet.
- **Be concise.** Compress transcript content into the substance, not a retelling — one tight bullet per point, merge closely related points, cut filler/repetition/color commentary. Keep concrete specifics (names, numbers, quotes, links) since those are what's hard to reconstruct later; cut everything else.
- **Use Quarto markdown features sparingly to highlight structure, not to decorate every bullet.** One or two per talk is plenty — reach for the plain bullet first, and only pull something out when it stands apart from the surrounding list:
    - `> ...` blockquote — a verbatim quote worth reading as a quote, not folded into a bullet. Attribute on its own line: `>\n> — Speaker Name`.
    - `::: {.callout-important}` — the talk's single core thesis or guiding principle.
    - `::: {.callout-tip}` — an actionable takeaway (a checklist, a rule of thumb) worth reading as a standalone box.
    - `::: {.callout-warning}` / `.callout-caution` — a specific risk or failure mode the speaker called out.
    - `::: {.callout-note}` — a conceptual aside worth flagging distinctly, e.g. one that connects to another talk.
    - Give every callout a `## Title` as its first line (short, a few words) — an untitled callout is harder to scan in a long page.
    - A definition list (`term\n:   definition`) beats a bullet list when a talk formally defines several terms (see Timothy Keyes's harness/tool/agent in `day1.qmd` for the pattern).
- Don't commit anything under `knowledge/day1/`, `knowledge/day2/`, `subtitles/`, or `m3u8/` — they're gitignored on purpose (fetched/derived content, not source notes). `knowledge/index.md` is the one exception under `knowledge/` and is committed (see above).
- `day1.qmd` and `day2.qmd` each open with a `::: {.callout-note}` disclaimer that notes are partly drafted from auto-generated captions and may contain mistakes — keep this in place (don't strip it when editing the file's top), and don't repeat it per-talk.
- This is not a software project: no build, lint, or test commands apply here.

## Tooling

Project-local skills live in `.claude/skills/`:

- `qmd-url-defuddle` — scans `day1.qmd`/`day2.qmd` (or any `.qmd` file) for URLs, fetches each as clean markdown via the local Defuddle CLI, and saves results under `knowledge/<day>/`, updating `knowledge/index.md`.
- `defuddle-cli` — the underlying single-URL fetch-as-markdown helper.
- `m3u8-subtitles` — extracts the English subtitle track from a session's `.m3u8` manifest (via `ffmpeg`) into `subtitles/vtt/<slug>.vtt`, then converts it into a flowing-text transcript at `subtitles/markdown/<slug>.md`.
- `quarto-flashcards` / `quarto-quiz` — reference for the RevealJS extension syntax (`.flashcard-front`/`.flashcard-back` divs; `{.quiz-question}` slides with `[answer]{.correct}`) used by `study-flashcards.qmd`/`study-quiz.qmd`. The extensions themselves live in `_extensions/parmsam/flashcards` and `_extensions/parmsam/quiz`.

Use `qmd-url-defuddle` when asked to pull references out of the notes rather than fetching URLs ad hoc. Use `m3u8-subtitles` when asked to get subs/transcript from a session recording manifest.

When asked to turn a `subtitles/markdown/` transcript into `day1.qmd`/`day2.qmd` notes: the transcript comes from auto-generated captions (not a human transcript), so it can contain misheard words, mangled names/jargon, and garbled sentences. Cross-check anything that looks off (speaker names, package/product names, numbers) against the talk's agenda listing or linked resources before writing it into the notes, rather than transcribing the transcript's mistakes verbatim.

## Website

`quarto preview` (live) or `quarto render` (one-shot, outputs to `_site/`) builds a small site with these pages: Home (`index.qmd`), Day 1, Day 2, Knowledge, and Study Materials (`study.qmd`, linking out to `study-flashcards.qmd` and `study-quiz.qmd`). Transcripts (`subtitles/markdown/`) are deliberately excluded from the site — kept as local files, not published.

- **Study Materials** (`study-flashcards.qmd`, `study-quiz.qmd`) render with `format: revealjs` (set in each file's own YAML frontmatter, overriding the project's default `html` format) via the `_extensions/parmsam/flashcards` and `_extensions/parmsam/quiz` extensions — installed with `quarto add parmsam/quarto-flashcards` / `quarto add parmsam/quarto-quiz` and checked into `_extensions/` per those extensions' own convention. See the `quarto-flashcards`/`quarto-quiz` skills or each extension's README for the slide markup.

- **Knowledge** (`knowledge.qmd`) pulls in `knowledge/index.md`'s table via `{{< include knowledge/index.md >}}` — the individual per-article files under `knowledge/day*/` are deliberately *not* rendered as their own site pages (just the index table, nothing more). This means the table's `File` column links won't resolve inside the site (Quarto warns "Unable to resolve link target" at render time — harmless, ignore it); those links are only meaningful when browsing `knowledge/index.md` directly (e.g. on disk or GitHub).

Since the per-article files under `knowledge/day*/` are gitignored (derived content, regenerated locally), those `File` column links only resolve when browsing on a machine that has actually run the `qmd-url-defuddle` skill — expected, since the individual articles aren't part of the published site anyway (see above). `knowledge/index.md` itself is committed, so the Knowledge page's table renders correctly both locally and in the GitHub Pages build.

Defuddle's fetched frontmatter includes a `language:` field, which collides with Quarto's own reserved `language:` metadata key (used for custom UI-string files) and breaks the render with a "Specified 'language' file does not exist" error *if* one of these files is ever rendered directly as a Quarto document. `qmd-url-defuddle`'s steps already rename this to `doc-language:` after each fetch as a defensive habit — not currently load-bearing (the per-article files aren't in the site's render path, see above), but keep doing it in case that changes, or if you add another URL-fetching path into `knowledge/`.

## Deployment

`.github/workflows/publish.yml` renders the Quarto site and deploys it to GitHub Pages via the official Actions-based flow (`actions/upload-pages-artifact` + `actions/deploy-pages`, no `gh-pages` branch). It runs on every push to `main` and can also be triggered manually (`workflow_dispatch`). No R/Python execution engine is configured or needed — the `.qmd` files here are plain markdown with no code chunks, so the workflow only needs `quarto render`, not a language runtime.

The repo's GitHub Pages source must be set to "GitHub Actions" (Settings → Pages → Build and deployment → Source) for this to work — a one-time manual step (or `gh api -X POST repos/<owner>/<repo>/pages -f build_type=workflow`) if it isn't already configured.
