# Bug report — Firefox for iOS

**Status: ✅ SUBMITTED 2026-10-04** by OvercastBTC →
**https://github.com/mozilla-mobile/firefox-ios/issues/35900**
(filed on the mozilla-mobile/firefox-ios GitHub tracker; no exact duplicate found —
related issues #14095/#17116 are about the feature existing, not the wrong icon.)

The text below is what was filed (template-formatted version lives in the issue).

---

**Title:** Firefox iOS "Add to Home Screen" ignores `apple-touch-icon` / manifest
icons and installs a name-monogram instead of the site icon

**Component:** Firefox for iOS — Home Screen shortcuts / web-clips

**Severity:** minor (cosmetic, but makes installed PWAs look broken vs Safari/Chrome/DDG)

**Summary**
When adding a web page to the iOS Home Screen from Firefox, the resulting icon is a
generated letter-monogram (first character of the shortcut name on a dark tile)
rather than the icon the page declares. Safari, Chrome iOS, and DuckDuckGo on the
same device and the same pages render the real icon.

**Steps to reproduce**
1. On iOS, open any page that declares `<link rel="apple-touch-icon" sizes="180x180" href="…">`
   (e.g. https://overcastbtc.github.io/ff-icon-test/diag/ or any PWA) in Firefox iOS.
2. Share → Add to Home Screen → Add.
3. Observe the Home-Screen icon.

**Expected:** the page's declared `apple-touch-icon` (as Safari/Chrome/DuckDuckGo show).

**Actual:** a dark tile with a single letter (the first character of the shortcut name).

**Evidence / scope (empirical test matrix)**
A dedicated test PWA was built isolating every icon declaration strategy
(https://github.com/OvercastBTC/ff-icon-test). Across ALL of the following, the
Firefox iOS installed tile was a name-monogram, while Safari & DuckDuckGo rendered
the real icon from the same pages:
- `apple-touch-icon` with/without `sizes`, `-precomposed`, transparent vs opaque,
  absolute vs relative href, single and multi-size (120/152/167/180)
- `rel="icon"` PNG (single, multi-size, 512/1024), `rel="icon"` SVG
- base64 data-URI icons (these render nothing even in the Firefox share sheet)
- web manifest icons; declaration ordering (icon-first vs apple-touch-icon-first);
  "volume" (many declarations)
- with Firefox set as the **default browser** (shortcut becomes a real standalone
  web-clip — `navigator.standalone:true` — but the icon is still a monogram)

Notably, Firefox **does** render `rel="icon"` correctly as the browser **tab favicon**,
and renders the icon in the **share-sheet preview** — only the Home-Screen web-clip
path substitutes a monogram.

**Environment:** iOS (reproduced on current iOS/Firefox iOS, Oct 2026), multiple sizes,
Retina (dpr 3). Firefox iOS runs on WKWebView (no Gecko on iOS).

**Impact:** PWAs installed from Firefox iOS look broken/unbranded compared with every
other iOS browser, pushing users/developers to tell people "install from Safari."

**Suggested fix:** when creating the Home-Screen web-clip, pass the page's resolved
`apple-touch-icon` (falling back to manifest icons / `rel="icon"`) to the clip icon,
as Safari/Chrome/DuckDuckGo do, instead of generating a name-monogram.
