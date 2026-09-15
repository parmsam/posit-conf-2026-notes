---
name: defuddle-cli
description: Fetch a URL as clean markdown using the local Defuddle CLI (npx defuddle), run entirely on-device without the defuddle.md hosted proxy. Use as a fallback when defuddle.md (the hosted service) is unreachable, rate-limited, or returns an error, or when the user asks to fetch a page "with the defuddle CLI" / "without the hosted proxy" / "locally".
allowed-tools: Bash
---

# Defuddle CLI — Fetch URL as Markdown (local, no hosted proxy)

Fetch $ARGUMENTS as clean markdown using the local `defuddle` npm CLI via `npx`. This is the same underlying extraction engine as the `defuddle.md` hosted service, but runs locally — use it when `defuddle.md` itself is down, rate-limited, or erroring, since it has no dependency on that remote proxy.

## How it works

`npx defuddle parse <url> --markdown` fetches the page itself, extracts the main article content (stripping nav/ads/boilerplate), and converts it to markdown.

## Command

```bash
npx --yes defuddle parse "https://example.com/some/page/" --markdown --frontmatter
```

- `--markdown` (`-m`): convert content to markdown
- `--frontmatter` (`-f`): prepend YAML frontmatter (title, author, source, etc.) — include this for parity with the hosted defuddle.md output
- `-o <file>`: write to a file instead of stdout
- `-u <string>`: pass a custom User-Agent if the request comes back 403/FORBIDDEN
- `-l <code>`: preferred language (BCP 47) if extraction picks the wrong locale

## Steps

1. Build the command with the target URL, `--markdown`, and `--frontmatter`.
2. Run it via Bash. First run may take a few extra seconds while `npx` fetches the `defuddle` package.
3. If it errors (403, timeout, JS-rendered page with no server-side HTML), report the failure — this CLI does not run a real browser, so heavily client-rendered pages may come back empty. In that case fall back further (e.g. WebFetch).
4. If saving to a file, mirror the same location convention as the `defuddle` skill: save to `ref-sources/` if that directory exists in the project, otherwise the current directory, using the last meaningful URL path segment + `.md` as the filename.

## Notes

- Requires `node`/`npx` on PATH.
- Does not work for PDFs.
- Unlike WebFetch, this does not summarize through an AI model — output is the verbatim extracted markdown.
- Unlike the `defuddle` skill (hosted `curl https://defuddle.md/...`), this makes no network call to defuddle.md at all — the fetch and extraction both happen locally via the npm package.
