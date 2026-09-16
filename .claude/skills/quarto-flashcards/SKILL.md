---
name: quarto-flashcards
description: >
  Use when a user wants to create interactive flip-card study presentations
  in Quarto RevealJS using the quarto-flashcards extension.
---

## Overview

**quarto-flashcards** turns Quarto RevealJS slides into flippable flashcards. Each slide can have a front and back; pressing a key or clicking a button flips between them. Useful for study decks, vocabulary drills, and Q&A presentations.

Install with:
```bash
quarto add parmsam/quarto-flashcards
```

Requires Quarto >= 1.3.0 and RevealJS output format.

## Document Setup

Add to the YAML header:

```yaml
format:
  revealjs:
    flashcards:
      flipKey: 'q'
      shuffleKey: 't'
      showFlipButton: true
      resetOnSlideChange: false
      flipButtonColorFront: "#2a76dd"
      flipButtonColorBack: "#28a745"
revealjs-plugins:
  - flashcards
```

## Writing Flashcard Slides

```markdown
## Optional slide title

::: {.flashcard-front}
Front content — question, term, or prompt
:::

::: {.flashcard-back}
Back content — answer, definition, or explanation
:::
```

Slides without `.flashcard-front` / `.flashcard-back` divs are treated as normal RevealJS slides and are unaffected.

## Supported Content

Flashcard fronts and backs support any Quarto/RevealJS content:

- Text and markdown formatting
- Images: `![alt text](image.png){width=50%}`
- R or Python code chunks
- LaTeX math: `$$E = mc^2$$`
- RevealJS slide attributes (e.g., `background-color="#ffcccc"`)

## Configuration Options

| Option | Default | Description |
|--------|---------|-------------|
| `flipKey` | `'q'` | Keyboard shortcut to flip the card |
| `shuffleKey` | `'t'` | Keyboard shortcut to shuffle and jump to first slide |
| `showFlipButton` | `true` | Show/hide the flip button in the top-right corner |
| `resetOnSlideChange` | `false` | Auto-reset cards to front when navigating slides |
| `flipButtonColorFront` | `"#2a76dd"` | Button color when showing the front |
| `flipButtonColorBack` | `"#28a745"` | Button color when showing the back |

## Usage Tips

- Press `t` to shuffle the deck and jump to the first slide for randomized study sessions
- Set `resetOnSlideChange: true` if you want cards to always start face-up when you navigate to them
- Mix flashcard slides with regular slides freely in the same deck
