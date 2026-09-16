---
name: quarto-quiz
description: >
  Use when a user wants to add interactive multiple-choice quiz slides to a
  Quarto RevealJS presentation using the quarto-quiz extension.
---

## Overview

**quarto-quiz** adds interactive multiple-choice quiz slides to Quarto RevealJS presentations. Users get immediate visual feedback, optional per-answer explanations, and optional scoring.

Install with:
```bash
quarto add parmsam/quarto-quiz
```

Requires Quarto >= 1.4.0 and RevealJS output format.

## Document Setup

Add to the YAML header:

```yaml
format:
  revealjs:
    quiz:
      checkKey: 'c'
      resetKey: 'q'
      shuffleKey: 't'
      allowNumberKeys: true
      shuffleOptions: true
      includeScore: false
revealjs-plugins:
  - quiz
```

## Writing Quiz Slides

**Single-choice question** — one correct answer:
```markdown
## What is the capital of France? {.quiz-question}

- Berlin
- [Paris]{.correct}
- Madrid
- Rome
```

**Multiple-choice question** — select all that apply:
```markdown
## Which of these are programming languages? {.quiz-question .quiz-multiple}

- [Python]{.correct}
- [JavaScript]{.correct}
- [HTML]{data-explanation="HTML is a markup language, not a programming language."}
- CSS
```

**With explanations** — shown after checking the answer:
```markdown
## What is the area of a circle? {.quiz-question}

- [$2\pi r$]{data-explanation="That's the circumference, not the area."}
- [$[\pi r^2$]{.correct}
- [$\pi d$]{data-explanation="That's the circumference using diameter."}
```

## Key Rules

- Slide must have `{.quiz-question}` class to be treated as a quiz slide
- Wrap the correct answer(s) with `[text]{.correct}`
- Add `{.quiz-multiple}` for select-all-that-apply questions
- Add `{data-explanation="..."}` to any option to show a hint after checking
- Non-quiz slides in the same deck work normally

## Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `checkKey` | `'c'` | Keyboard shortcut to check answer |
| `resetKey` | `'q'` | Keyboard shortcut to reset question |
| `shuffleKey` | `'t'` | Keyboard shortcut to shuffle slide order |
| `allowNumberKeys` | `true` | Press 1–4 to select options |
| `disableOnCheck` | `false` | Lock options after checking |
| `disableReset` | `false` | Hide the reset button |
| `shuffleOptions` | `true` | Randomize option order on load |
| `defaultCorrect` | `""` | Custom correct feedback message |
| `defaultIncorrect` | `""` | Custom incorrect feedback message |
| `includeScore` | `false` | Show running score on slides |
