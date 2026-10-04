# ff-icon-test

Minimal isolation test PWA for **REQ-024** — diagnosing why the BATTERY PWA
(`https://overcastbtc.github.io/battery/`) shows a fallback icon instead of the
real icon when added to the iPhone Home Screen **from Firefox on iOS**.

Built by **Lane F (F.26.10.03.1)**. This repo is intentionally separate from the
BATTERY repo — it only produces findings + a recommended fix.

## Why Firefox iOS is the problem surface

Firefox for iOS runs on **WKWebView** (Apple mandates WebKit on iOS — no Gecko),
so its Add-to-Home-Screen icon behavior tracks WebKit but with Firefox's own
quirks. Empirically it tends to honor `<link rel="apple-touch-icon">` at a
180×180 opaque PNG and often **ignores** manifest `icons`, and is sensitive to
alpha/transparency, the `sizes` attribute, `precomposed`, and absolute vs
relative href. None of this is reliably documented — hence this empirical test.

## Strategies (one per page)

| Page | Strategy |
|------|----------|
| `a/` | apple-touch-icon, 180×180, opaque, **no** sizes attr |
| `b/` | apple-touch-icon, 180×180, opaque, **with** `sizes="180x180"` |
| `c/` | web manifest `icons` **only** (no apple-touch-icon link) |
| `d/` | **both** apple-touch-icon (180) + manifest icons (BATTERY's current approach) |
| `e/` | `apple-touch-icon-precomposed`, 180×180, opaque |
| `f/` | apple-touch-icon, 180×180, **transparent** (alpha) |
| `g/` | full multi-size set 120 / 152 / 167 / 180, opaque |
| `h/` | apple-touch-icon, 180×180, opaque, **absolute** URL href |

Each page displays its strategy label + expected icon preview on screen and sets
a distinct Home-Screen name (`FF-A` … `FF-H`) so results are unambiguous.

## Test protocol (for Adam, in Firefox iOS)

For each page: open in Firefox iOS → Share → **Add to Home Screen** → observe the
Home-Screen icon → record **PASS** (real coloured-letter icon) or **FAIL**
(white box / screenshot / globe). Report results back to Lane F → Q.

## Deploying

GitHub Pages (static). Live index:
`https://overcastbtc.github.io/ff-icon-test/`

## Regenerating assets

- `scratchpad/gen_icons.py` — generates the distinct per-strategy PNGs into `icons/`.
- `scratchpad/gen_html.py` — generates the strategy pages + manifests.
