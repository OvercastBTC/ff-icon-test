# BATTERY bottom-nav / safe-area — Lane F advisory (2026-10-04)

Follow-up requested by Adam after REQ-024: make BATTERY's bottom menubar render
faithfully and **stay** stable across browsers. Advisory only — routes to Q → A/E.

## Current implementation (from the live shell + iframes)

```css
#stage  { position:absolute; top:0; left:0; right:0; bottom:50px; }   /* iframe host */
#tabbar { position:fixed; bottom:0; left:0; right:0; height:50px;
          padding-bottom:0; }                                        /* HOME/FUEL */
```
- Shell uses `env(safe-area-inset-top)` ×9 (top handled well) but **`safe-area-inset-bottom`
  nowhere** in the shell.
- **fuel.html uses `env(safe-area-inset-bottom)` ×4**, on bottom-sheet modals
  (`border-radius:18px 18px 0 0; padding:18px 16px calc(18px + env(safe-area-inset-bottom))`).
- **arm.html uses it 0 times.**

## The problem (inconsistent bottom safe-area ownership)

The iframes (`#f-arm`, `#f-fuel`) live **inside `#stage`, whose bottom edge is 50px above
the device bottom** (at the top of `#tabbar`). But `env(safe-area-inset-bottom)` inside a
WKWebView iframe still resolves to the **device** inset (~34px on home-indicator iPhones),
not the iframe's real distance from the device edge. So:

1. **fuel.html double-counts** — its bottom-sheets add ~34px of phantom bottom padding for
   a home indicator that is *below the tabbar*, not below the iframe. Content floats up / a
   dead gap appears at the bottom of Fuel sheets.
2. **arm.html and fuel.html disagree** — Fuel pads the bottom, Arm doesn't → the two tabs
   feel different at the bottom.
3. **`#tabbar` — the element actually at the device bottom — has no safe-area padding.** In
   a standalone/installed PWA (no browser chrome) on a home-indicator iPhone, the lower
   ~34px of the 50px buttons sit in the gesture zone.

Why Firefox looks most "faithful / stays put" today: Firefox iOS keeps a **persistent**
bottom toolbar, so `#tabbar` is always anchored above it and never shifts. Safari's toolbar
**auto-hides on scroll**, so the same `bottom:0` bar moves/relayouts as the toolbar
collapses — that's the instability you've felt. Installed/standalone removes the chrome
entirely, exposing issue #3.

## Recommended fix (shell OWNS the bottom safe area; iframes never touch it)

```css
/* shell */
#stage  { bottom: calc(50px + env(safe-area-inset-bottom)); }   /* reserve full bar height */
#tabbar { padding-bottom: env(safe-area-inset-bottom); }        /* lift buttons above indicator */
```
```css
/* fuel.html — REMOVE the device inset (it's inside the stage, above the tabbar) */
/* was: padding:18px 16px calc(18px + env(safe-area-inset-bottom)); */
         padding:18px 16px 18px;        /* ×3-4 bottom-sheet rules */
```
- arm.html: no change (already correct).
- Net rule: **exactly one layer — the shell — accounts for `safe-area-inset-bottom`.**
  Nothing inside `#stage` (any iframe) may use it, or it double-counts. Matches the standing
  "don't use safe-area-inset-bottom in stage/iframe layout" guidance.

Result: identical, correct bottom spacing in Firefox, Safari (incl. collapsed toolbar),
DuckDuckGo, and installed/standalone — buttons clear the home indicator, Fuel sheets lose
the phantom gap, Arm and Fuel match.

## Verify on-device

Inspector: **https://overcastbtc.github.io/ff-icon-test/safearea/** — reports
`env(safe-area-inset-*)` in the top document **and inside an iframe** (the decisive
double-count check: iframe bottom should read 0 but will likely read ~34px), plus the
browser-chrome overlap (`innerHeight − visualViewport.height`) that explains Safari's shift.
Run it in Firefox / Safari / DuckDuckGo, in-browser and installed, and Copy the report.
