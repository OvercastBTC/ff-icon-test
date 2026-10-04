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

## FINAL recommended BATTERY `<head>` icon block (for Lane A/E)

BATTERY's **manifest already** declares icon-192 (any), icon-512 (any) and icon-512
(maskable) — so the two `<link rel="icon" type="image/png" 192/512>` tags in the
`<head>` are **redundant**, and they reproduce Strategy **O** (apple-touch-icon
followed by rel=icon PNGs) which made **DuckDuckGo render "color only."** Remove
them; keep favicon + SVG for the tab favicon, apple-touch-icon for iOS, manifest
for Android.

```html
<!-- Desktop / tab favicon (Firefox tab bar renders this reliably) -->
<link rel="icon" href="favicon.ico" sizes="any">
<link rel="icon" type="image/svg+xml" href="data:image/svg+xml;base64,…keep…">
<!-- iOS / iPadOS Home-Screen webclip — Safari, Chrome iOS, DuckDuckGo.
     KEEP the sizes attribute: DuckDuckGo ignores apple-touch-icon without it. -->
<link rel="apple-touch-icon" sizes="180x180" href="icon-180.png">
<link rel="apple-touch-icon" sizes="167x167" href="apple-touch-icon-167x167.png">
<link rel="apple-touch-icon" sizes="152x152" href="apple-touch-icon-152x152.png">
<link rel="apple-touch-icon" sizes="120x120" href="apple-touch-icon-120x120.png">
<!-- Android / Chrome PWA install (already covers 192/512 any+maskable) -->
<link rel="manifest" href="manifest.webmanifest">
```

**Change = DELETE these two lines** (manifest already provides them):
```html
<link rel="icon" type="image/png" sizes="192x192" href="icon-192.png">
<link rel="icon" type="image/png" sizes="512x512" href="icon-512.png">
```

Rationale from tests: apple-touch-icon+`sizes` is the proven installed-tile winner
(Strategy B) in DuckDuckGo/Safari/Chrome; extra rel=icon PNGs only undercut it
(Strategy O → "color only") and are redundant with the manifest. `data:` URIs are
NOT usable for the touch/home icon (Strategy J/K rendered nothing), so the icon
must stay a real file — BATTERY already does this. Verify the DuckDuckGo installed
tile before/after; Safari/Chrome unaffected.

**Firefox iOS:** still a "B" monogram after this change — unfixable at the markup
layer (bug: FF iOS hands iOS a name-monogram for webclips, ignoring all declared
icons; it only honors rel=icon for the tab favicon). Mitigate with an FxiOS hint
("add from Safari/Chrome/DuckDuckGo for the icon") and optionally file a Mozilla bug.

## Assets
- Pages: `a/`…`i/`, `j/k/m` (rel=/base64/combo), `n/o/p` (order/SVG), `v/s/t`
  (volume/size), `bi/` (real BATTERY art, I-method), `diag/` (on-device inspector),
  `z/` (name discriminator), `index.html` (hub).
- Generators: `scratchpad/gen_icons.py`, `gen_html.py`, `gen_variants{,2,3}.py`.
