---
title: "Documentation · Defuddle"
author: "kepano"
site: "kepano"
source: "https://defuddle.md/docs"
language: "en"
description: "Documentation for Defuddle, a library that extracts the main content from web pages and returns clean Markdown or HTML."
word_count: 1352
---

## Installation

```shell
npm install defuddle
```

For Node.js use, install a DOM implementation:

```shell
npm install defuddle linkedom
```

Or [JSDOM](https://github.com/jsdom/jsdom):

```shell
npm install defuddle jsdom
```

To use the CLI globally, install with `-g`, or use `npx` to run without installing globally:

```shell
# Install globally
npm install -g defuddle

# Or use npx
npx defuddle parse https://example.com/article
```

## Browser use

In the browser, create a Defuddle instance with a `Document` object and call `parse()`.

```ts
import Defuddle from 'defuddle';

const result = new Defuddle(document).parse();

console.log(result.content);  // cleaned HTML string
console.log(result.title);    // page title
```

You can also parse HTML strings using `DOMParser`:

```ts
const parser = new DOMParser();
const doc = parser.parseFromString(htmlString, 'text/html');
const result = new Defuddle(doc).parse();
```

Pass options as the second argument:

```ts
const result = new Defuddle(document, {
  url: 'https://example.com/article',
  debug: true
}).parse();
```

## Node.js use

The Node.js API accepts a DOM `Document` from any implementation (linkedom, JSDOM, happy-dom, etc.) and returns a promise.

```ts
import { parseHTML } from 'linkedom';
import { Defuddle } from 'defuddle/node';

const { document } = parseHTML(htmlString);
const result = await Defuddle(document, 'https://example.com/article', {
  markdown: true
});
```

Or with JSDOM:

```ts
import { JSDOM } from 'jsdom';
import { Defuddle } from 'defuddle/node';

const dom = new JSDOM(htmlString, { url: 'https://example.com/article' });
const result = await Defuddle(dom.window.document, 'https://example.com/article');
```

**Note:** For `defuddle/node` to import properly, your `package.json` must have `"type": "module"`.

## CLI use

Defuddle includes a CLI for parsing web pages from the terminal. You can run it with `npx` or install it globally with `npm install -g defuddle`.

```shell
# Parse a local HTML file
npx defuddle parse page.html

# Parse a URL
npx defuddle parse https://example.com/article

# Output as markdown
npx defuddle parse page.html --markdown

# Output as JSON with metadata
npx defuddle parse page.html --json

# Extract a specific property
npx defuddle parse page.html --property title

# Save output to a file
npx defuddle parse page.html --output result.html
```

### CLI options

| Option | Alias | Description |
| --- | --- | --- |
| `--output <file>` | `-o` | Write output to a file instead of stdout |
| `--markdown` | `-m` | Convert content to markdown |
| `--md` |  | Alias for `--markdown` |
| `--json` | `-j` | Output as JSON with metadata and content |
| `--property <name>` | `-p` | Extract a specific property |
| `--debug` |  | Enable debug mode |

### Using templates

Use [Knap](https://knap.md/cli#from-defuddle) to format extracted content and metadata with a Markdown template. Pipe Defuddle's JSON output into Knap:

```shell
npx defuddle parse https://example.com/article --markdown --json \
  | npx knap render template.md --data - --output note.md
```

Your `template.md` can reference Defuddle properties such as `{{ title }}` and `{{ content }}`. See the [Knap CLI docs](https://knap.md/cli#from-defuddle) for a complete example.

## API use

Use the hosted API to extract a web page without installing Defuddle. Append the page URL to `https://defuddle.md/`:

```shell
curl https://defuddle.md/stephango.com
```

The response is Markdown with YAML frontmatter containing page metadata. You can include the target URL's `https://` prefix and path, for example `https://defuddle.md/https://stephango.com/saw`.

To return clean HTML instead, add `/html/` before the target URL:

```shell
curl https://defuddle.md/html/stephango.com
```

For additional requests, buy a request block and pass your API key in the `Authorization` header. See [pricing](https://defuddle.md/pricing) for request blocks, usage checks, and top-ups.

```shell
curl -H "Authorization: Bearer YOUR_KEY" https://defuddle.md/stephango.com
```

## Options

Options can be passed when creating a Defuddle instance (browser) or as the third argument (Node.js).

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `url` | string |  | URL of the page being parsed |
| `markdown` | boolean | false | Convert `content` to Markdown |
| `separateMarkdown` | boolean | false | Keep `content` as HTML and return Markdown in `contentMarkdown` |
| `removeExactSelectors` | boolean | true | Remove elements matching exact selectors (ads, social buttons, etc.) |
| `removePartialSelectors` | boolean | true | Remove elements matching partial selectors |
| `removeHiddenElements` | boolean | true | Remove elements hidden via CSS (display:none, visibility:hidden, etc.) |
| `removeLowScoring` | boolean | true | Remove non-content blocks by scoring (navigation, link lists, etc.) |
| `removeSmallImages` | boolean | true | Remove small images (icons, tracking pixels, etc.) |
| `removeImages` | boolean | false | Remove images from the output |
| `useAsync` | boolean | true | Allow async extractors to fetch from third-party APIs when no local content is available. |
| `standardize` | boolean | true | Standardize HTML (footnotes, headings, code blocks, etc.) |
| `contentSelector` | string |  | CSS selector to use as the main content element, bypassing auto-detection |
| `language` | string |  | Preferred language (BCP 47 tag, e.g. `en`, `fr`). Sets `Accept-Language` header and selects transcript language. |
| `includeReplies` | boolean \| 'extractors' | 'extractors' | Include replies: `'extractors'` for site-specific extractors only, `true` for all, `false` for none |
| `debug` | boolean | false | Enable debug logging and return debug info in the response |

## Response

The `parse()` method returns an object with the following properties:

| Property | Type | Description |
| --- | --- | --- |
| `content` | string | Cleaned HTML string of the extracted content |
| `contentMarkdown` | string | Markdown version (when `separateMarkdown` is true) |
| `title` | string | Title of the article |
| `description` | string | Description or summary |
| `author` | string | Author of the article |
| `site` | string | Name of the website |
| `domain` | string | Domain name of the website |
| `favicon` | string | URL of the website's favicon |
| `image` | string | URL of the article's main image |
| `language` | string | Language of the page in BCP 47 format (e.g. `en`, `en-US`) |
| `published` | string | Publication date |
| `wordCount` | number | Number of words in the extracted content |
| `parseTime` | number | Time taken to parse in milliseconds |
| `metaTags` | object\[\] | Meta tags from the page |
| `schemaOrgData` | object | Schema.org data extracted from the page |
| `extractorType` | string | Type of site-specific extractor used, if any |
| `debug` | object | Debug info including content selector and removals (when `debug: true`) |

## Bundles

Defuddle is available in three bundles:

| Bundle | Import | Description |
| --- | --- | --- |
| Core | `defuddle` | Browser usage. No dependencies. Handles math content but without MathML/LaTeX conversion fallbacks. |
| Full | `defuddle/full` | Includes math equation parsing (MathML ↔ LaTeX) and Markdown conversion via Turndown. |
| Node.js | `defuddle/node` | For Node.js. Accepts any DOM Document (linkedom, JSDOM, happy-dom, etc.). Includes full capabilities for math and Markdown conversion. |

The core bundle is recommended for most use cases.

## HTML standardization

Defuddle standardizes HTML elements to provide a consistent input for downstream tools like Markdown converters.

### Headings

- The first H1 or H2 is removed if it matches the title.
- H1s are converted to H2s.
- Anchor links in headings are removed.

### Code blocks

Code blocks are standardized. Line numbers and syntax highlighting are removed, but the language is retained.

```html
<pre>
  <code data-lang="js" class="language-js">
    // code
  </code>
</pre>
```

Inline references and footnotes are converted to a standard format using `sup`, `a`, and an ordered list with `class="footnote"`.

### Math

Math elements, including MathJax and KaTeX, are converted to standard MathML with a `data-latex` attribute containing the original LaTeX source.

### Callouts

Callout and alert elements from various sources are standardized to the [Obsidian callout format](https://help.obsidian.md/Editing+and+formatting/Callouts). When converting to Markdown, these become Obsidian-style callouts.

Supported sources:

- GitHub markdown alerts (`div.markdown-alert`)
- Obsidian callouts (`div.callout[data-callout]`)
- Callout asides (`aside.callout-*`)
- Bootstrap alerts (`div.alert.alert-*`)
```html
<div data-callout="info" class="callout">
  <div class="callout-title">
    <div class="callout-title-inner">Info</div>
  </div>
  <div class="callout-content">
    <p>This is an informational callout.</p>
  </div>
</div>
```

## Debugging

### Debug mode

When debug mode is enabled:

- Returns a `debug` field in the response with detailed information about content extraction
- More verbose console logging about the parsing process
- Preserves HTML class and id attributes that are normally stripped
- Retains all `data-*` attributes
- Skips div flattening to preserve document structure
```ts
const result = new Defuddle(document, { debug: true }).parse();

// CSS selector path of chosen main content element
console.log(result.debug.contentSelector);

// Array of removed elements with step, reason, selector, and text preview
console.log(result.debug.removals);
```

The `debug` field contains:

| Property | Type | Description |
| --- | --- | --- |
| `contentSelector` | string | CSS selector path of the chosen main content element |
| `removals` | array | List of elements removed during processing |

Each removal entry contains:

| Property | Type | Description |
| --- | --- | --- |
| `step` | string | Pipeline step (e.g. `removeLowScoring`, `removeBySelector`, `removeHiddenElements`) |
| `selector` | string | CSS selector or pattern that matched |
| `reason` | string | Why the element was removed (e.g. `score: -20`, `display:none`) |
| `text` | string | First 200 characters of removed element's text content |

### Pipeline toggles

Disable individual pipeline steps to diagnose content extraction issues:

```ts
// Skip content scoring
const result = new Defuddle(document, { removeLowScoring: false }).parse();

// Skip hidden element removal
const result = new Defuddle(document, { removeHiddenElements: false }).parse();

// Skip small image removal
const result = new Defuddle(document, { removeSmallImages: false }).parse();

// Skip HTML standardization
const result = new Defuddle(document, { standardize: false }).parse();
```

### Content selector

Use `contentSelector` to bypass auto-detection and specify the main content element directly. Falls back to auto-detection if the selector doesn't match.

```ts
const result = new Defuddle(document, {
  contentSelector: 'article.post-content'
}).parse();
```