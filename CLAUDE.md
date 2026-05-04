# OneSixtyOne — Project Context

## Overview
Website for **onesixtyone.co** — a creative agency. Three standalone HTML files, no build system, no frameworks. All CSS and JS is inline per file.

## File Structure
```
index.html       — Homepage (most complex, has loader + nav animation)
about.html       — About page
why-us.html      — Why Us page
Photos/          — All images and SVGs (including Union.svg emblem)
CLAUDE.md        — This file
```

## Pages & Nav
All three pages share the same nav structure. Nav items: **About · Capabilities · Work · Why Us · Contact**
- `index.html` uses hash anchors (`#capabilities`, `#work`, `#contact`)
- `about.html` / `why-us.html` use `index.html#capabilities` etc. for cross-page anchors
- Hash is cleaned from URL on arrival via `history.replaceState` in JS
- Nav logo links to `#` on index, `index.html` on other pages

## Loader
- Runs on fresh load only (suppressed on internal `sessionStorage` nav)
- **Duration:** 1.25s animation, overlay clears at 1.45s, overflow restored 500ms later (prevents scrollbar-shift jank)
- **Desktop:** spinning emblem (`loaderCoin` keyframe, `rotateY 720deg`) + number counter 0→100
- **Mobile:** spinning emblem + CSS bar loader (pure CSS, no JS)
- Emblem: `transform-style: preserve-3d`, parent has `perspective: 500px`
- `buildStrip()` calls are null-safe — removing the work section won't crash JS

## Nav Animation (index.html desktop only)
Slot-machine rise effect triggered by `nav-in` class added at loader clear:
- Mechanism: `overflow: hidden` on `.nav-logo` and `.nav-links li`; children (`span` / `a`) translate from `translateY(110%)` → `translateY(0)`
- Logo text wrapped in `<span>` for this to work
- **Rise duration:** 1.75s `cubic-bezier(0.22, 1, 0.36, 1)` (expo-out)
- **Stagger:** logo at 0s, item 1 at 0.04s, then +0.09s per item (About=0.04, Capabilities=0.09, Work=0.18, Why Us=0.27, Contact=0.36)

## Hero (index.html)
- Title rises with same slot-machine effect (`hero-in` class, `transition-delay: 0.08s`)
- Title text wrapped in `<span class="hero-title-inner">` inside `h1.panel-title`
- `#hero .panel-title` has `overflow: hidden`
- Hero is excluded from scroll-reveal system

## Scrollbar
- Hidden by default, appears while scrolling via `is-scrolling` class on `<html>`
- `scrollbar-gutter: stable` prevents layout shift when scrollbar appears/disappears
- JS: adds `is-scrolling` on scroll, removes it 1s after scroll stops
- Applied to all three files

## Desktop Nav Style
- Transparent background, no border, top gradient (`rgba(0,0,0,0.55)` → transparent, 180px, `z-index: 399`)
- Nav animation only on `index.html` — other pages show nav immediately (no loader delay)

## Scroll Reveal
- Panels (excluding `#hero`) use `IntersectionObserver` with `reveal` → `visible` classes
- `opacity: 0; transform: translateY(32px)` → `opacity: 1; transform: translateY(0)`
- Process section has its own observer with step stagger
- Work heading (`work-head`) watched separately via `dark-section` observer

## Work Section
- Currently placeholder (no real projects yet) — strip tracks are empty
- **Plan:** rebuild with concept brands once client roster is established
- Strip tracks: `strip1` (fwd, 55s) and `strip2` (rev, 70s) — images injected via JS

## Saved for Later
- **Stats bar** (brands shaped / years / independent / industries) — removed from homepage, restore when client roster is established. Full code saved in project memory: `project_stats_section.md`

## Key CSS Patterns
- Clip/reveal: use `overflow: hidden` on parent + `translateY` on child (NOT `clip-path` — causes "eyelid" effect)
- 3D transforms need `transform-style: preserve-3d` on element and `perspective` on parent; avoid `overflow: hidden` on ancestors
- Scroll reveal: `.reveal` class sets initial hidden state, `.visible` triggers transition
- All three files share identical scrollbar, footer mobile, and loader CSS patterns

## Footer Mobile Fix
All three files have matching mobile footer CSS:
```css
@media (max-width: 600px) {
    .site-footer { padding: 20px 24px; flex-direction: column; align-items: flex-start; gap: 14px; }
    .footer-left { flex-direction: column; align-items: flex-start; gap: 14px; }
    .footer-divider { display: none; }
}
```
