# REQ-024 Findings — iOS Home-Screen Icon, cross-browser

**Lane F (F.26.10.03.1) · 2026-10-03**
Test PWA: https://overcastbtc.github.io/ff-icon-test/

## Headline

- **Firefox iOS** ignores *all* web-declared icons for the Home-Screen tile and
  draws a **name monogram** (first letter of the shortcut name). Confirmed:
  names "FF-*" → "F"; name "Zebra" → "Z". **Unfixable at the markup layer.**
- **DuckDuckGo iOS** *does* render the icon, but **requires the `sizes` attribute
  on `apple-touch-icon`.** Strategy A (no `sizes`) FAILED; B (identical but
  `sizes="180x180"`) SUCCEEDED. This is the one concrete, fixable markup finding.
- Safari / Chrome iOS: reported working (full confirmation pending).

## Cross-browser matrix

| Strategy | `<head>` declaration | DuckDuckGo iOS | Firefox iOS | Safari | Chrome iOS |
|---|---|---|---|---|---|
| A | apple-touch-icon 180 opaque, **no sizes** | ❌ FAIL | monogram | _pending_ | _pending_ |
| B | apple-touch-icon 180 **+ sizes** | ✅ pass | monogram | _pending_ | _pending_ |
| C | manifest icons only | 🟡 color only | monogram | _pending_ | _pending_ |
| D | both apple-touch-icon + manifest (**BATTERY's current**) | ✅ pass | monogram | _pending_ | _pending_ |
| E | apple-touch-icon-precomposed | ✅ pass | monogram | _pending_ | _pending_ |
| F | apple-touch-icon transparent | ✅ pass | monogram | _pending_ | _pending_ |
| G | full set 120/152/167/180 (all w/ sizes) | ✅ pass | monogram | _pending_ | _pending_ |
| H | apple-touch-icon absolute url | ✅ pass | monogram | _pending_ | _pending_ |
| I | **rel="icon" only** (no apple-touch-icon, no manifest) | _new, pending_ | _pending_ | _pending_ | _pending_ |
| Z | apple-touch-icon + name "Zebra" | ✅ pass | **"Z" monogram** | _pending_ | _pending_ |

Legend: ✅ real icon · ❌ no icon · 🟡 partial (color, no mark) · monogram = name letter.

## What each browser needs

- **DuckDuckGo iOS:** `apple-touch-icon` **must carry a `sizes` attribute.** Bare
  `apple-touch-icon` (no sizes) is ignored. precomposed / transparent / multi-size
  / absolute-url all fine. Manifest-only yields color-only.
- **Firefox iOS:** nothing works for the tile — it always uses a name monogram.
  The only lever is the **name** (first letter). No markup fix exists.
- **Safari / Chrome iOS:** honor apple-touch-icon normally (confirmation pending).

## BATTERY status vs these findings

BATTERY's live `<head>` already ships `apple-touch-icon` at 180/167/152/120
**each with `sizes`** + manifest icons + title/name "BATTERY".
- → Already satisfies **DuckDuckGo** (has sizes), and Safari/Chrome.
- → In **Firefox iOS** it renders as a **"B"** monogram tile — that *is* the
  "fallback letter" in REQ-024. No markup can change that.

## Recommendation (routes Q → Lane A/E)

1. **No required BATTERY icon-markup change** for DuckDuckGo/Safari/Chrome —
   BATTERY already uses `apple-touch-icon` **with `sizes`** (the thing DDG needs).
   Keep it. If A/E ever strip the `sizes` attribute, DuckDuckGo would break — so
   **treat `sizes` on apple-touch-icon as required, not optional.**
2. **Firefox iOS is a client limitation, not a BATTERY defect.** Add a small
   FxiOS-conditional hint (UA contains `FxiOS`): *"On iPhone, add to Home Screen
   from Safari, Chrome, or DuckDuckGo for the Battery icon — Firefox on iOS shows
   a plain letter tile."* Keep the name "BATTERY" so the monogram is a clean "B".
3. Close REQ-024 as: **icon works in Safari / Chrome / DuckDuckGo; Firefox iOS
   limited to a name monogram (mitigated by guidance).** Not a code defect.

_(Pending to finalize: Strategy I result, and Safari/Chrome columns.)_

## Assets
- Pages: `a/`…`i/` (one strategy each), `z/` (name discriminator), `index.html` (hub).
- Generators: `scratchpad/gen_icons.py`, `scratchpad/gen_html.py`.
