# REQ-024 Findings — Firefox iOS Home-Screen Icon

**Lane F (F.26.10.03.1) · 2026-10-03**
Test PWA: https://overcastbtc.github.io/ff-icon-test/

## Result: 0 / 8 strategies produce a real home-screen icon in Firefox iOS

Every strategy's **Add-to-Home-Screen dialog preview is an identical black tile
with a white letter monogram** — none of the icon declarations influence it.

| Strategy | `<head>` declaration | iOS **share-sheet** icon (what Firefox resolved) | **Add-to-Home-Screen** tile preview | Verdict |
|---|---|---|---|---|
| A | apple-touch-icon 180 opaque, no sizes | ✅ real icon (inset on white) | ⬛ monogram | FAIL |
| B | + `sizes="180x180"` | ✅ real icon | ⬛ monogram | FAIL |
| C | manifest icons **only** | 🧭 Safari-compass fallback | ⬛ monogram | FAIL |
| D | both apple-touch-icon + manifest (**BATTERY's current**) | ✅ real icon | ⬛ monogram | FAIL |
| E | apple-touch-icon-precomposed | ✅ real icon | ⬛ monogram | FAIL |
| F | apple-touch-icon 180 **transparent** | ✅ real icon (alpha→white) | ⬛ monogram | FAIL |
| G | full set 120/152/167/180 | ✅ real icon | ⬛ monogram | FAIL |
| H | apple-touch-icon **absolute url** | ✅ real icon | ⬛ monogram | FAIL |

## Root cause

**Firefox for iOS does not honor any web-declared icon for the Home-Screen
web-clip.** Its *Add to Home Screen* generates a **first-letter monogram of the
shortcut name** on a dark tile, regardless of apple-touch-icon (any rel/size/
alpha/path) or manifest icons.

Two independent code paths were observed:
- **Share sheet** — *does* resolve the apple-touch-icon (icons A/B/D/E/F/G/H all
  showed the correct art; C fell back to the Safari compass because it has no
  apple-touch-icon). This proves the icon files and declarations are valid and
  reachable — Firefox finds them.
- **Home-Screen tile** — ignores all of it and draws a name monogram.

This is an **iOS platform constraint**: Apple only grants the full custom
web-clip-icon pipeline to Safari. Third-party iOS browsers (all on WKWebView)
cannot set a custom home-screen icon, so Firefox substitutes a monogram.
**No change to BATTERY's `<head>` can fix the icon in Firefox iOS.**

## The one available lever: the monogram letter = the app name

The monogram is the first character of the Add-to-Home-Screen **name** (defaults
from `<title>` / `apple-mobile-web-app-title`). Confirmed with the `z/`
discriminator page (named "Zebra" → tile shows **"Z"**, not "F").
*(see confirmation below once Adam runs z/)*

BATTERY today: `<title>BATTERY</title>`, `apple-mobile-web-app-title="BATTERY"`,
manifest name/short_name `"BATTERY"` → Firefox-iOS monogram = **"B"** on a dark
tile. That "B" tile *is* the "fallback letter" reported in REQ-024.

## Recommendation (routes Q → Lane A/E)

1. **No icon markup change is needed or will help for Firefox iOS.** BATTERY's
   existing icon set is already correct for Safari (the supported path) and for
   the share sheet. **Leave the `<head>` icon block as-is.** Do not spend effort
   adding/altering apple-touch-icon or manifest icons to chase Firefox iOS — it
   is unwinnable at the markup layer.
2. **Make the unavoidable monogram look intentional.** Keep the name "BATTERY"
   (monogram "B"); optionally shorten the default Add name to **"Battery"** so the
   capital **B** reads cleanly. Nothing else about the tile is controllable.
3. **Guide iOS users to Safari for a real icon.** Add a small conditional hint in
   BATTERY (shown when the browser is Firefox iOS, or in install docs):
   *"On iPhone, add to Home Screen from **Safari** to get the Battery icon.
   Firefox on iOS shows a plain letter tile — an Apple limitation for non-Safari
   browsers."* Detection: UA contains `FxiOS`.
4. **Close REQ-024 as "works as designed in Safari; Firefox-iOS limitation,
   mitigated by guidance."** Not a BATTERY code defect.

## Reproduce / assets
- Pages: `a/`…`h/` (one strategy each), `z/` (name discriminator), `index.html` (hub).
- Icon generators: `scratchpad/gen_icons.py`, `scratchpad/gen_html.py`.
