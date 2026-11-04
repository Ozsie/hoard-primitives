# @hoard-engine/primitives

Theme tokens and canvas drawing primitives for the HOARD engine: the
theme-token contract (`UiTheme`) with its default values and a deep-merge
resolver, plus text (measuring, ellipsized truncation, wrapping, font
composition), geometry (`Rect`, `Point`, hit testing) and rounded-rect
clipping. A dependency-free leaf of the HOARD engine — pure computation
and rendering contracts, no game or framework knowledge.

## Pluggable, interoperable, standalone

HOARD engine packages share the same design goals:

- **Pluggable.** Adopt a package on its own, or swap it out, without
  touching the rest of the engine — there is no central engine object
  to install and no package that owns the others.
- **Interoperable.** Packages compose with each other and with
  consumer code through data: minimal shapes, callback seams and plain
  values — never through shared state or a common framework.
- **As standalone as possible.** Each package is usable as-is in any
  project, HOARD-based or not; the leaf packages carry no dependencies
  and import no siblings.

This package meets all three by construction.

## Install

```bash
npm install @hoard-engine/primitives
```

Requires Node 18+ (ESM only).

## Usage

```ts
import {
  UiTheme,
  defaultTheme,
  resolveTheme,
  composeFont,
  measureTextWidth,
  truncateText,
  wrapText,
  drawRoundedRect,
  roundRectPath,
  isPointInRect,
  Rect,
  Point,
} from '@hoard-engine/primitives';

// The theme is data: merge overrides over the defaults (frozen result).
const theme = resolveTheme({ palette: { text: { body: '#e2e8f0' } } });

// Fonts compose from theme tokens; text measures through a context or
// falls back to a per-character approximation, so layout stays
// deterministic without a canvas.
const font = composeFont(theme, theme.typography.sizes.sm);
const width = measureTextWidth(null, 'Hello', font, 6.5);
const lines = wrapText(null, 'Hello world', 60, font, 6.5);
const fitted = truncateText(null, 'Hello world', 30, font, 6.5);

// Hit testing shares one geometry with layout.
const rect: Rect = { x: 0, y: 0, width: 60, height: 20 };
if (isPointInRect(10, 10, rect)) {
  /* ... */
}

// Rounded rects trace the current path (the caller fills or strokes);
// roundRectPath prefers the native ctx.roundRect when available.
drawRoundedRect(ctx, 0, 0, 60, 20, 8);
```

The package never asks for your world state or your framework — every
canvas arrives as a caller-owned `CanvasRenderingContext2D`, every color
and font through the tokens.

## API

| Export                                                     | Description                                                                                                 |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `UiTheme`                                                  | The theme-token contract: role-named color, typography, size and spacing tokens, consumed by every renderer |
| `DeepPartial<T>`                                           | Recursively optional variant of a theme, for overrides                                                      |
| `defaultTheme`                                             | The built-in default token values                                                                           |
| `resolveTheme(overrides?)`                                 | Deep-merges overrides over `defaultTheme`; the result is frozen                                             |
| `composeFont(theme, size, weight?, family?)`               | A canvas font string composed from theme tokens                                                             |
| `measureTextWidth(ctx, text, font, fallbackPerChar)`       | The width through a context, or the per-character approximation without one                                 |
| `truncateText(ctx, text, maxWidth, font, fallbackPerChar)` | The longest `'...'`-terminated prefix fitting `maxWidth`                                                    |
| `wrapText(ctx, text, maxWidth, font, fallbackPerChar)`     | Greedy word wrap into lines                                                                                 |
| `wrapTextMeasured(measure, text, maxWidth, font)`          | Canvas-free `wrapText`, measuring through an injected `TextMeasure`                                         |
| `PER_CHAR_WIDTH_FALLBACK`                                  | The per-character width approximation (6.5 px) used without a context                                       |
| `Point` / `Rect`                                           | Pixel-space geometry shapes: `{ x, y }` and `{ x, y, width, height }`                                       |
| `isPointInRect(x, y, rect)`                                | Hit test sharing the same geometry as layout                                                                |
| `drawRoundedRect(ctx, x, y, width, height, radius)`        | Traces a rounded rect as the current path (the caller fills or strokes), radius clamped to the box          |
| `roundRectPath(ctx, x, y, width, height, radius)`          | Traces through the native `ctx.roundRect` corners when available, plain-rect fallback                       |

## Design notes

- **The theme is data.** Tokens are role-named, not value-named: two
  roles may hold the same value today so a future theme can split them,
  and one role never changes meaning between components. `resolveTheme`
  merges overrides over the defaults; the merged result is frozen.
- **Deterministic layout.** Text measures through a context or falls
  back to a per-character approximation, so layout stays reproducible
  without a canvas and can be pinned and replayed.
- The same primitives serve UI chrome (`@hoard-engine/uikit`) and world
  rendering (the future `scene` package): both draw, so both consume
  this package.

## Development

```bash
npm test          # vitest
npm run check     # tsc --noEmit
npm run lint      # eslint over src
npm run prettier  # prettier --check (fix via prettier:format)
npm run build     # tsc declarations + esbuild ESM bundle to dist/
npm publish       # prepublishOnly runs check + build + test first
```
