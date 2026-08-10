# OneSixtyOne — Project Context

## Overview
Website for **onesixtyone.co** — a creative agency. Standalone HTML files, no build system, no frameworks. All CSS and JS is inline per file.

## File Structure
```
index.html                     — Homepage (single-page: hero, about, principles, capabilities, brand DNA, work index, process, faq, contact). Has loader + nav animation.
foxcroft.html                  — Case study 001 (Foxcroft, Inc.)
jaheim-harding.html            — Case study 002 (Jaheim Harding photography)
OneSixtyOne-Design-System.html — Design system reference
Photos/                        — Image assets
CLAUDE.md                      — This file
```
(Note: `about.html` / `why-us.html` no longer exist — the site consolidated into a single-page `index.html`.)

## Pages & Nav
Desktop nav items: **Who we are · What we do · Our work · How we do it** (+ fixed Contact button). Mobile menu adds **Contact**.
- `index.html` uses hash anchors (`#who-we-are`, `#capabilities`, `#work`, `#process-section`, `#contact`)
- Case study pages (`foxcroft.html`, `jaheim-harding.html`) share identical chrome; each is one HTML file per project, named after the client. They use cross-page anchors (`index.html#capabilities` etc.); "Our work" links to `index.html#work`; wordmark + Contact link back to `index.html`; a "← Selected work" backlink returns to the index.
- Nav uses `mix-blend-mode: difference` so it stays legible over both dark and light content
- Homepage "Our work" (`#work`) is the **work index**: a `.cs-work-list` of `.cs-work-item` cards (alternating image side), each linking to a case study page. Add a new case study by cloning a case study HTML file + adding a card here.
- Case study pages reuse the `.cs-*` component styles (statement, story, shots, quote, deliverables, close). To add a live-site + secondary link, use `.cs-cta-row` (see jaheim-harding.html).

## Loader
- Runs on fresh load only (suppressed on internal `sessionStorage` nav)
- **Duration:** 1.25s animation, overlay clears at 1.45s, overflow restored 500ms later (prevents scrollbar-shift jank)
- **Desktop:** spinning emblem (`loaderCoin` keyframe, `rotateY 720deg`) + number counter 0→100
- **Mobile:** spinning emblem + CSS bar loader (pure CSS, no JS)
- Emblem: `transform-style: preserve-3d`, parent has `perspective: 500px`

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

## Work Section
- `#work` on the homepage is the case-study index (see Pages & Nav above); the old horizontal strip/cards work section is fully removed (markup, CSS, and GSAP block)

## Saved for Later
- **Stats bar** (brands shaped / years / independent / industries) — removed from homepage, restore when client roster is established. Full code saved in project memory: `project_stats_section.md`

## Key CSS Patterns
- **Horizontal overflow (index.html): `html` and `body` both use `overflow-x: clip` — NEVER `hidden`.** Anything parked off-screen (e.g. `.cap-scrub` words at `translateX(±100vw)`) widens the mobile layout viewport unless clipped, which shoves `position: fixed` elements (burger) off-screen and enables horizontal scroll. And `hidden` on body (when html isn't `visible`) turns body into a scroll container, which **kills the sticky hero slide-over**. `clip` clips without creating a scroll container. `#capabilities` also has `overflow-x: clip` as a local guard.
- `mix-blend-mode: difference` on fixed elements (`nav`, `#siteWordmark`) is **desktop-only** (`@media (min-width: 769px)`) — on mobile the nav holds only the burger and fixed blend layers cause rendering trouble.
- Clip/reveal: use `overflow: hidden` on parent + `translateY` on child (NOT `clip-path` — causes "eyelid" effect)
- 3D transforms need `transform-style: preserve-3d` on element and `perspective` on parent; avoid `overflow: hidden` on ancestors
- Scroll reveal: `.reveal` class sets initial hidden state, `.visible` triggers transition
- All three files share identical scrollbar, footer mobile, and loader CSS patterns

## Footer Effects (all three files)
- **Stretching wordmark** (Namma-style): footer wordmark img sits in `.footer-wordmark-wrap`; JS scrubs `scaleY(0 → 1)` on the img with **`transform-origin: top`** (matches studionamma.com — their 51px ScrollTrigger offset is their footer's padding-bottom). We compute the below-space dynamically: `line = vh - (scrollHeight - wrapDocBottom)`, `p = (line - wrapTop) / wrapHeight`. Because the scrub range equals the wordmark's own height and it's top-anchored, its bottom edge stays pinned to the line — it grows/squashes *in place*, ending at exactly max scroll. Vanilla rAF scroll handler (no GSAP — case study pages don't load it). Measure the **wrap**, not the img, for the scrub range — `getBoundingClientRect` on the scaled img returns transformed values. Skipped under `prefers-reduced-motion`.
- **Duplicate-text rollovers**: `.fx-roll` (overflow hidden, `padding-bottom: 4px; margin-bottom: -4px`) holds `.fx-roll-a` + absolute `.fx-roll-b` (second copy, `aria-hidden`); hover on the parent (`a.footer-item`, `.footer-btn`, `.back-to-top-footer`) slides a → `-140%`, b → `0`. Applied to every footer link.

## Footer Layout (all three files, Namma-style)
Structure inside `<footer class="site-footer">` (flex column, `gap: 88px`, wordmark is the **last** element on the page):
1. `.footer-menus` — two `.footer-col`s, space-between: left = site nav (Home / Who we are / What we do / Our work / How we do it / Contact; hash anchors on index, `index.html#…` on case pages), right (right-aligned) = LinkedIn, Instagram (text links, no icons), Privacy Policy (button `#openPrivacy` on index, `index.html#privacy` link on case pages), back-to-top `#backToTop`
2. `.footer-say` — conversational mono row: location line · "Big project?…" · "Let's talk." · mailto link · © line (`.footer-note` allows wrapping, `max-width: 24ch`)
3. `.footer-wordmark-wrap` — the stretching wordmark, final thing on the page
Mobile (≤768px): `.footer-say` stacks vertically; `.footer-menus` stays two-column. All links use `.footer-item` (DM Mono 11px uppercase) + `.fx-roll` rollovers.
