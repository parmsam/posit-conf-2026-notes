---
name: m3u8-frames
description: Extract still frames at specific timestamps from a session recording's m3u8 HLS manifest (Mux) in this project's m3u8/ folder, saving them as jpgs under images/<slug>/ for embedding in day1.qmd/day2.qmd notes. Use when the user asks to "grab a frame at this timestamp", "pull a screenshot from this talk", "extract slide images", "get frames for the important moments in a talk", or similar.
allowed-tools: Bash
---

# m3u8 → Frames

Given an HLS master playlist (`.m3u8`) saved under this project's `m3u8/` folder and a short list of timestamps, extract still frames with `ffmpeg` and save them as jpgs under `images/<slug>/`, ready to embed in the day notes with `![...](images/<slug>/<file>.jpg)`.

$ARGUMENTS: optional path to a specific `.m3u8` file, and/or the session slug/title, and/or the timestamps to grab.

## Steps

1. **Resolve the manifest file.** If `$ARGUMENTS` names one, use it. Otherwise pick the most recently modified file in `m3u8/`:

   ```bash
   ls -t m3u8/*.m3u8 | head -1
   ```

2. **Get the slug.** Same convention as `m3u8-subtitles`: `<day>-<kebab-case-title>` (lowercase, spaces/punctuation → hyphens). If this talk already has a transcript, reuse the exact slug from `subtitles/vtt/` / `subtitles/markdown/` so the images line up with it — check those folders rather than re-deriving the slug from scratch. If no transcript exists yet, ask for day + title and build the slug the same way.

3. **Get the timestamps.** Ask the user for the moments to capture (accept `MM:SS` or `HH:MM:SS`, with an optional short label per timestamp, e.g. `12:34 architecture-diagram`).

   **Always derive candidate timestamps from a fresh subtitle pull off the exact manifest you're about to extract frames from — never from an existing `subtitles/vtt/<slug>.vtt`/`subtitles/markdown/<slug>.md` file, even one that already exists for this same slug.** A re-saved `.m3u8` is not guaranteed to share a timeline with an earlier one for "the same" talk: pre-show banter/intro length varies recording to recording, and a session can be a materially different edit (a live product demo present in one capture and absent in another has been observed in practice). Trusting an old transcript's timestamps against a newly saved manifest silently produced frames of the *wrong slide* — several minutes off and drifting further as the video went on — with no error to catch it.

   Pull the new manifest's own English subtitle track the same way `m3u8-subtitles` does (`grep -o 'NAME="English".*URI="[^"]*"' "<manifest>" | ...` then `ffmpeg -y -i "<subtitle-uri>" -c copy /tmp/<slug>-frames-check.vtt`) — a scratch copy is fine, this doesn't need to land in `subtitles/`. Search that fresh VTT for the keywords/quotes tied to each moment you want (the notes bullets or the existing `subtitles/markdown/<slug>.md` prose are a good source of search terms even though their *timestamps* aren't trustworthy) to get this manifest's actual timing, then let the user confirm before extracting.

   **Keep the count modest** — a handful of key-moment frames per talk (title slide, a key diagram, a key stat) is usually plenty; each one is a separate seek-and-decode and every embedded image adds page weight. If the user gives a long list without a stated limit, ask how many to keep or pick the most load-bearing subset yourself and say which you dropped.

4. **Pick the highest-resolution video rendition** from the master playlist. Mux manifests list one `#EXT-X-STREAM-INF` line per rendition, each followed by its URI on the next line:

   ```bash
   grep -A1 '^#EXT-X-STREAM-INF' "<manifest>" | grep -B1 'RESOLUTION' | paste -d' ' - -
   ```

   or more simply, since renditions are listed highest-bandwidth-first in Mux manifests, take the first `#EXT-X-STREAM-INF` block's URI (the line immediately after it that doesn't start with `#`).

5. **Extract one frame per timestamp** with `ffmpeg` (requires `ffmpeg` on PATH), seeking on the input for speed:

   ```bash
   mkdir -p "images/<slug>"
   ffmpeg -y -ss "<timestamp>" -i "<rendition-uri>" -frames:v 1 -q:v 2 "images/<slug>/<mm-ss>.jpg"
   ```

   Use `<mm-ss>` (e.g. `12-34`) as the filename, or the user's label if they gave one (`<mm-ss>-<label>.jpg`). The signed rendition URL is time-limited — if a grab fails with a 403/expired error, the `.m3u8` is stale and needs to be re-saved from the browser (same failure mode as `m3u8-subtitles`).

6. **Report** the saved paths and hand back a ready-to-paste markdown snippet for each frame, e.g.:

   ```markdown
   ![architecture diagram](images/day1-example-talk/12-34.jpg)
   ```

   so the user can drop it straight into the relevant `###` subsection of `day1.qmd`/`day2.qmd`. If dropping these in directly yourself rather than handing them back, always leave a blank line after the image line before whatever follows (another bullet, a heading, prose) — an image glued directly to the next line with no blank line between them is a subtle Quarto/pandoc markdown bug that's easy to miss on a quick read.

## Notes

- `images/` is **not** gitignored — unlike `m3u8/`/`subtitles/`, these files are meant to be committed and published as part of the site once referenced from the day notes.
- `-ss` placed before `-i` does fast input-side seeking (nearest keyframe), which is fine for illustrative screenshots; if a frame lands a beat off from a scene/slide change, nudge the timestamp by a second and re-run rather than switching to slow output-side seeking.
- Don't delete the source `.m3u8` after use; other skills (`m3u8-subtitles`) may still need it, and multiple session manifests accumulate in that folder over the conference.
- If a requested timestamp is past the video's duration, `ffmpeg` will fail or emit a black/blank frame — sanity-check the extracted jpg (file size, or open it) rather than assuming success from a zero exit code alone.
- Always visually check each extracted frame (read the jpg back) against what it's supposed to show before handing off — a wrong-but-successfully-decoded frame (see the timestamp-drift note in step 3) looks identical to a correct one from the exit code and file size alone.
