# My notes from posit::conf(2026) virtual

- [Day 1: Conf Day 1](day1.qmd)
- [Day 2: Conf Day 2](day2.qmd)

## Browsing as a website

This project is also a Quarto website, for browsing the notes plus the derived knowledge content in one place.

```bash
quarto preview
```

This opens a local site with Home, Day 1, Day 2, and Knowledge (fetched reference articles) pages. `quarto render` builds it once to `_site/` instead of previewing live. See `AGENTS.md` for details.

The site is also published automatically to GitHub Pages on every push to `main`: **https://parmsam.github.io/posit-conf-2026-notes/** (see `.github/workflows/publish.yml`).
