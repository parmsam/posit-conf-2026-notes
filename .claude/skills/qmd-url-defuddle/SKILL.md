---
name: qmd-url-defuddle
description: Extract every URL referenced in this project's day1.qmd / day2.qmd conference notes (or any .qmd file), fetch each one as clean markdown using the local defuddle-cli skill, and save the results into knowledge/day1/ or knowledge/day2/ (matching the source .qmd file's name), keeping knowledge/index.md in sync. Use when the user wants to "pull the references from day1.qmd", "fetch markdown copies of the links in my notes", "grab the URLs in day2 as markdown", or similar.
---

# QMD URL → Defuddle Markdown

Scan one or more `.qmd` files for URLs and fetch each as clean markdown via the local Defuddle CLI (same command as the `defuddle-cli` skill), depositing the results under the top-level `knowledge/` folder, in a subfolder named after the source file, and keeping `knowledge/index.md` up to date as the table of everything fetched.

$ARGUMENTS: optional list of `.qmd` files to process. If omitted, process every `*.qmd` file in the project root (e.g. `day1.qmd`, `day2.qmd`).

## Steps

1. **Resolve target files.** If `$ARGUMENTS` is empty, glob `*.qmd` in the project root. Otherwise use exactly the files given.

2. **For each `.qmd` file:**

   a. **Extract URLs** with a plain regex over the raw text (don't rely on markdown-link syntax — these notes contain bare URLs in bullet text, not just `[text](url)` links):

      ```bash
      grep -Eo 'https?://[^][()<>"'"'"'[:space:]]+' day1.qmd | sed -E 's/[.,;:]+$//' | sort -u
      ```

   b. **Determine the output folder**: `knowledge/<name>/`, where `<name>` is the `.qmd` filename without its extension (`day1.qmd` → `knowledge/day1/`). Create it with `mkdir -p` if missing.

   c. **For each URL**, derive an output filename the same way the `defuddle-cli` skill does: the last meaningful path segment (strip trailing slash, query string, and fragment), sanitized to a safe filename, plus `.md`. If the URL has no path segment (e.g. `https://example.com/`), fall back to the domain name. If two URLs in the same file would collide on the same filename, disambiguate by prefixing the parent path segment (e.g. `commons-articles.md` vs `commons-r-articles.md`).

   d. **Skip URLs that already have a saved file** in the destination folder (idempotent — don't re-fetch on repeat runs) unless the user explicitly asks to refresh/refetch.

   e. **Fetch with the local Defuddle CLI**, mirroring the `defuddle-cli` skill's command exactly:

      ```bash
      npx --yes defuddle parse "<url>" --markdown --frontmatter -o "<folder>/<slug>.md"
      ```

   f. **On failure** (403, timeout, empty/JS-rendered output), don't abort the whole run — note the URL and reason, skip it, and continue to the next one.

   g. **Rename the `language:` frontmatter key to `doc-language:`** in the saved file (e.g. `sed -i '' 's/^language:/doc-language:/' <file>`). Defuddle's `language` field collides with Quarto's own reserved `language:` metadata key (used for custom UI-string files) — the project's Quarto site renders everything under `knowledge/` directly, so an unrenamed `language:` value breaks the build with a "Specified 'language' file does not exist" error.

3. **Update `knowledge/index.md`**, the top-level index. It always starts with `# Knowledge Docs Index` followed by one Markdown table with columns `Day | Title | URL | File`:

   - `Day` — a human label derived from the `.qmd` filename (`day1.qmd` → `Day 1`).
   - `Title` — the `title:` field from the fetched file's YAML frontmatter.
   - `URL` — the source URL (also the `source:` frontmatter field), as a clickable Markdown link, e.g. `[https://example.com](https://example.com)` — not a bare URL.
   - `File` — a relative Markdown link to the saved file, e.g. `[day1/commons.md](day1/commons.md)`.

   Add one row per newly fetched file (skip rows for URLs that failed). If the file already exists from a prior run, leave its existing row as-is rather than duplicating it. Create `knowledge/index.md` if it doesn't exist yet.

4. **Report a summary** per `.qmd` file: which URLs were fetched (with destination path), which were skipped because a file already existed, and which failed (with reason).

## Notes

- This only fetches `http(s)` URLs — relative links to other project files (e.g. `(day1.qmd)` in `README.md`) are naturally excluded by the regex and are not the target of this skill anyway.
- Duplicate URLs within the same file are deduped before fetching.
- Does not work for PDFs (same limitation as `defuddle-cli`).
- Uses the local CLI only — no dependency on or fallback to the hosted `defuddle.md` proxy.
- All fetched markdown lives under `knowledge/` (never at the project root) — `knowledge/day1/`, `knowledge/day2/`, etc. — with `knowledge/index.md` as the single top-level entry point.
