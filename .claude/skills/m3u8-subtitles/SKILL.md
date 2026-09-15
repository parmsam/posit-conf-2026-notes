---
name: m3u8-subtitles
description: Extract the English subtitle track from a session recording's m3u8 HLS manifest (Mux) in this project's m3u8/ folder, save it as WebVTT under subtitles/vtt/, and produce a flowing-text markdown transcript under subtitles/markdown/. Use when the user asks to "get subs from the latest m3u8", "pull subtitles for this session", "make a transcript from the m3u8", or similar.
allowed-tools: Bash
---

# m3u8 → Subtitles → Transcript

Given an HLS master playlist (`.m3u8`) saved under this project's `m3u8/` folder, extract its English WebVTT subtitle track and produce both a raw `.vtt` file and a clean flowing-text markdown transcript, matching the naming convention already used under `subtitles/`.

$ARGUMENTS: optional path to a specific `.m3u8` file, and/or the session title. If no file is given, use the most recently modified `*.m3u8` file in `m3u8/`. If no title is given, ask the user for the talk/session title — it's needed to build the output filename slug and can't be reliably inferred from the manifest.

## Steps

1. **Resolve the manifest file.** If `$ARGUMENTS` names one, use it. Otherwise pick the most recently modified file in `m3u8/`:

   ```bash
   ls -t m3u8/*.m3u8 | head -1
   ```

2. **Get the session title and day**, if not already given. Ask the user which day (`day1`, `day2`, ...) and the talk/session title. Build a slug: `<day>-<kebab-case-title>` (lowercase, spaces/punctuation → hyphens), e.g. `day1-being-an-informed-and-savvy-ai-user`. This must match the slug style already used in `subtitles/vtt/` and `subtitles/markdown/` — check those folders for the existing pattern if unsure.

3. **Extract the English subtitle sub-playlist URI** from the manifest. Mux manifests list one `#EXT-X-MEDIA:TYPE=SUBTITLES,...` line per language:

   ```bash
   grep -o 'NAME="English".*URI="[^"]*"' "<manifest>" | grep -o 'URI="[^"]*"' | sed 's/URI="//;s/"$//'
   ```

   If there's no `NAME="English"` track, list the available `NAME=` values and ask the user which language to use.

4. **Download and mux the subtitle track to a single VTT file** with `ffmpeg` (requires `ffmpeg` on PATH):

   ```bash
   ffmpeg -y -i "<subtitle-uri>" -c copy "subtitles/vtt/<slug>.vtt"
   ```

   This works because the subtitle sub-playlist is itself segmented WebVTT; ffmpeg concatenates the segments into one file. The signed URLs in Mux manifests are time-limited — if this fails with an expired/403 error, the `.m3u8` file is stale and needs to be re-saved from the browser.

5. **Convert the VTT into a flowing-text markdown transcript.** Strip cue numbers, timestamp lines, the `WEBVTT` header, and `X-TIMESTAMP-MAP=...` metadata lines (these leak in from ffmpeg concatenating per-segment VTT chunks and are not dialogue). Collapse consecutive duplicate cue lines (rolling/live captions repeat the same line across multiple cues). Join what's left with single spaces into one paragraph:

   ```bash
   python3 -c "
   import re
   with open('subtitles/vtt/<slug>.vtt') as f:
       lines = f.read().splitlines()
   texts = []
   for line in lines:
       line = line.strip()
       if not line or line == 'WEBVTT':
           continue
       if re.match(r'^\d+\$', line):
           continue
       if '-->' in line:
           continue
       if line.startswith('X-TIMESTAMP-MAP'):
           continue
       texts.append(line)
   deduped = []
   for t in texts:
       if not deduped or deduped[-1] != t:
           deduped.append(t)
   body = ' '.join(deduped)
   with open('subtitles/markdown/<slug>.md', 'w') as f:
       f.write('# <Day N>: <Title> Transcript\n\n')
       f.write(body + '\n')
   "
   ```

   Use a title-cased version of the session title for the markdown H1 (e.g. `# Day 1: Being an Informed and Savvy AI User Transcript`), matching the existing files' style.

6. **Report** the two output paths and the transcript's approximate length (word/char count), plus the source manifest's duration if visible in the `ffmpeg` output.

## Notes

- `subtitles/` and `m3u8/` are both gitignored — these are local working files, not committed notes content.
- Don't delete the source `.m3u8` file; multiple session manifests can accumulate there over the conference.
- If a `.vtt`/`.md` pair already exists for the resolved slug, ask before overwriting rather than silently re-running.
- This does not fetch video/audio, only the subtitle track — much smaller and faster than downloading the full rendition.
