---
name: conf-notetaker
description: Turn one or more session transcripts (under subtitles/markdown/, produced by the m3u8-subtitles skill) into day1.qmd/day2.qmd conference notes following this repo's house style. For each session, dispatches a parallel subagent to draft directly from that session's transcript — keeping the raw transcript text out of the main conversation — then assembles the results into the target day file in schedule order. Use when the user says "convert the transcripts to notes", "turn this into day2.qmd notes", "digest the transcripts", "write up the notes for these sessions", or similar.
---

# Conf Notetaker

Converts session transcripts into structured `day1.qmd`/`day2.qmd` notes, using one subagent per session to keep large transcripts out of the main session's context.

$ARGUMENTS: optional list of session slugs/titles to process and/or the target day file (`day1`/`day2`). If omitted, work it out from context — pending transcripts under `subtitles/markdown/` not yet reflected in the day file — and confirm with the user if it's ambiguous which transcripts to process or what order they happened in.

## Step 1 — Identify inputs

- List candidate transcripts: `ls subtitles/markdown/<day>-*.md`. Diff against the target day file's existing `##` talk headings (`grep '^## ' day2.qmd`) to figure out which sessions are already covered — skip those unless the user explicitly asks to redo one.
- For each transcript, work out whether it's a **single-talk session** or a **multi-talk session** (one recording covering several back-to-back short talks, e.g. a themed block of four 20-minute talks). The user will often paste the schedule directly in chat (title/speaker/time per talk) right before or after asking for the m3u8 — if so, that's the lineup; capture it verbatim, don't paraphrase or reorder it.
- If a session has no given lineup (e.g. an impromptu lightning-talk session assembled at the last minute), that's fine — the drafting agent will identify individual talk/speaker segments itself from the transcript.
- Note the sessions' actual chronological order (by scheduled time, or by which m3u8 was fetched first if no times were given) — this is the order fragments get assembled in.

## Step 2 — Dispatch one subagent per session

Launch **one general-purpose agent per session** (not per talk), all in a single message when there's more than one pending, since each is independent. Never read the full transcript files yourself first "just to check" — that defeats the point of delegating; only the agent needs to load that transcript's ~50–70k characters.

Each agent's prompt must be self-contained (it starts with zero context) and include, verbatim:

1. **The transcript file path** under `subtitles/markdown/`.
2. **The known talk lineup**, if one was given — exact titles, speakers, and times, in order. Tell the agent to locate each talk's portion of the transcript by topic/content shifts, not by evenly splitting the runtime.
3. **If no lineup was given**: instruct the agent to identify individual talk/speaker segments itself (self-introductions, moderator hand-offs, "next up," topic shifts) and to describe an unclear speaker by role/topic instead of guessing a name it can't verify.
4. **The house style rules**, pasted in full (the agent hasn't read `AGENTS.md`, so restate the substance rather than pointing at the file):
   - One `## <Talk Title> - <Speaker Name>` heading per talk (or `## Keynote: <Title> - <Speaker>` for a keynote slot — including a lightning-talk session that *replaced* a keynote, which still gets the `Keynote:` prefix plus a bullet noting the swap).
   - Structure each talk with its own `###` subsections (setup/context, main content or case study, key takeaways/closing) instead of one flat bullet list.
   - Be concise: one tight bullet per point, merge closely related points, cut filler/repetition/audience banter/color commentary. Keep concrete specifics — names, numbers, quotes, package/tool names, URLs — since those are hard to reconstruct later.
   - Use Quarto markdown features sparingly, one or two per talk at most, only when something genuinely stands apart from a plain bullet:
     - `> ...` blockquote for a single verbatim quote, attributed on its own line (`>\n> — Speaker Name`).
     - `::: {.callout-important}` for the talk's single core thesis, with a short `## Title` as its first line.
     - `::: {.callout-tip}` for one actionable takeaway, same title convention.
     - `::: {.callout-warning}` / `.callout-caution` for a specific risk/failure mode called out.
     - `::: {.callout-note}` for a conceptual aside worth flagging distinctly (e.g. connects to another talk).
     - A definition list (`term\n:   definition`) when a talk formally defines several terms.
     - Not every talk needs a callout — most segments in a lightning-talk session will just be plain bullets.
   - Bare URLs for any tool/package/repo mentioned go on their own bullet line, no markdown link syntax needed (this site auto-links bare URLs and uses them to build a references index).
   - No top-level `#` heading, no YAML frontmatter, no repeating the "notes are AI-assisted" disclaimer — the target file already has those once, not per-talk. Output starts directly with the first `##` heading, no code fence around it.
5. **Auto-caption-error handling**: the transcript comes from auto-generated captions, not a human transcript — fix obviously-misheard names/jargon using judgment (well-known people, product/package names) but never invent facts, numbers, or URLs that aren't actually present. Flag anything left uncertain in the final report rather than silently guessing.
6. **The exact output path** to write the fragment to — a scratchpad path, one file per session, e.g. `<scratchpad>/<day>-fragment-<session-slug>.qmd`.
7. **Instruction to reply briefly** — one sentence per talk plus the fragment's line count, not the full fragment text. The coordinating session reads the fragment file directly; repeating it in the chat reply just burns context for nothing.

## Step 3 — Assemble once all agents finish

- Read each fragment file directly (much smaller than the raw transcripts — typically 80–170 lines vs. tens of thousands of characters).
- Concatenate them onto the target day file's existing header (title + `::: {.callout-note}` disclaimer — never duplicate this; it's per-file, not per-talk) in the sessions' real chronological order.
- Sanity-check the assembled file: `grep -n '^# '` should show only the file's own title line (no stray top-level headings leaked from a fragment), and `grep -c '^## '` should roughly match the expected talk count (callout titles are also `##`, so the count will run a bit higher than the talk count — that's expected, see AGENTS.md's callout convention).

## Step 4 — Verify and report

- `quarto render <day>.qmd` (or a full `quarto render`) to confirm no Pandoc/Quarto errors before calling it done.
- Report what was added: talk/session count, and anything a drafting agent flagged as uncertain (a garbled name, an unverifiable URL, an ambiguous talk boundary) so the user can spot-check against the source recording.

## Notes

- This skill assumes `m3u8-subtitles` has already produced the transcript(s) it's given — it doesn't fetch new manifests itself. Point the user at that skill first if a session's transcript doesn't exist yet.
- Per-talk boundaries inside a multi-talk fragment are always a judgment call by the drafting agent (transcripts don't carry timestamps once flattened to prose) — imperfect splits are expected and fine as long as content lands under the right talk heading.
- Fragments written to the scratchpad are throwaway once merged — no need to preserve them.
- For a single small edit to one already-drafted talk (not a whole new session), it's fine to skip subagent delegation and edit `day1.qmd`/`day2.qmd` directly — delegation exists specifically to keep a ~50–70k-character multi-talk transcript out of the main session's context, not as a blanket rule for every edit.
- Committing/pushing the updated day file is a separate, explicit ask — this skill's job ends at a rendered, verified `day*.qmd`.
