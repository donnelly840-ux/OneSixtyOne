# OneSixtyOne — Project Context

## Overview
Website for **onesixtyone.co** — a creative agency. Standalone HTML files, no build system, no frameworks. All CSS and JS is inline per file.

## File Structure
```
index.html                     — Homepage (single-page: hero, statement, about, principles, capabilities, work index, process, contact). Has loader + nav animation.
foxcroft.html                  — Case study 001 (Foxcroft, Inc.)
jaheim-harding.html            — Case study 002 (Jaheim Harding photography)
OneSixtyOne-Design-System.html — Design system reference
Photos/                        — Image assets
_archive/effects/              — Cut-but-reusable effects (see Saved for Later)
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

## Statement (`#statement`) — pinned positioning line
The first thing after the hero, so "what we do" is answered before the visitor has to dig. Holds the **"Bespoke brand strategy, design, and web for the people who care about the details."** line, which used to open the About section.
- **Pin idiom, chained.** `#statement` is `260svh` (the runway) with `.statement-lane` `position: sticky` inside it — the same pattern as `#who-principles`. It's also the first section in `.page-body`, so it's what rides over the sticky hero; then `#who-we-are` (`z-index: 2`, `margin-top: -70svh`) slides over *it*. Hero → statement → about is one continuous chain of sheets.
- **Animation:** the `.fill-word` scroll-fill (words light from `opacity: 0.12` → `1`, left to right) scrubbed across the pin, so the reader is walked through the whole line while the section holds still. This effect **moved here from the About statement** — `.who-statement-head` no longer has it and uses a plain `.reveal`.
- **Timing** (viewport units from the section's top): fill completes at **+60%**, cover starts at **+90%** (set by the `-70svh` margin), lane unsticks at **+160%**. That leaves a 30vh beat of finished statement before About starts covering. Changing `height` or `margin-top` moves all three — recheck that the fill still finishes before the cover starts.
- **Sticky killers apply**: `#statement` must not be `overflow: hidden` and `.statement-lane` must not have `will-change: transform`.
- Mobile shortens the runway to `190svh` / `-55svh` — 2.6 screens of scroll for one sentence is too much on a phone.
- The word split runs **inside the GSAP guard**, so with no GSAP the line just renders at full opacity.

## Who We Are (`#who-we-are`) — one statement, one left edge
Restructured Aug 2026. The section was three fragments on three different alignment axes; it now makes a single argument.
- **Structure**: topbar eyebrow → `.who-split` (Venn left, content right). The right column holds `h2.who-statement-head` ("If you lose the details, you lose everything.") then `.who-copy` — everything left-aligned and sharing one left edge.
- **The "Bespoke brand strategy, design, and web…" line moved out to [`#statement`](#statement--pinned-positioning-line)**, where it now opens the page instead of sitting mid-About. It was doing two jobs here: re-answering *what we do* and duplicating the paragraph beneath it.
- **One headline, not two.** The old layout had an italic 300 statement and a bold 700 pull quote at nearly the same size, so neither led. There is now exactly one big type block.
- **Column order**: `h2.who-statement-head` → `.who-copy` (two paragraphs) → `.who-partner-line` → `.partner-mark` (mobile only). `.who-partner-line` holds "You're not a client to us. You're a partner." as a standalone emphasis line — sized *between* the headline and the body copy (`clamp(21px, 2vw, 32px)`, weight 500, full-opacity ink vs the copy's 0.65) so it reads as a beat rather than a second headline. Keep it in that band; pushing it up to headline scale re-creates the two-headline problem this section was restructured to fix.
- **The old `.who-pullquote-text` / `.who-split-quote` are deleted** — the headline is `.who-statement-head`, restyled from italic 300 to bold 700.
- **Venn labels mirror `#capabilities` exactly**: Brand Strategy / Creative / Web. They used to be Design/Strategy/Craft — a third taxonomy that appeared nowhere else on the site, which is precisely why the diagram read as foreign. Keep these in sync with the three `.cap-title` rows.
- **Desktop (≥769px)**: `.who-ripple-wrap` holds the interactive `#vennDiagram` (three overlapping discipline circles + ripple rings + emblem).
- **Mobile (≤768px)**: no diagram at all — `.who-ripple-wrap` and `.venn-svg` are `display: none` and the section is a plain type stack (headline → copy → partner line). The `.partner-mark` stand-in (two overlapping circles, `#partnerLens` clipPath) was **deleted Aug 2026**; markup and CSS are both gone, recover from git if wanted.
- The Venn JS still builds its ~130 SVG rings on mobile even though nothing shows them. Left deliberately: gating the build on width breaks the desktop↔mobile resize workflow (build runs once at load), so any fix needs a `matchMedia` listener that builds lazily on first desktop match.
- Replaced a 340px dot-grid canvas ("emblem constellation") that sat *between* the statement and the quote, broke up the section, and carried no meaning. Its ~165 lines of canvas JS are deleted — mobile no longer runs a rAF/canvas workload here.
- The mark deliberately has **no `.reveal` class**: an IntersectionObserver on a `display: none` element is unreliable when resizing desktop→mobile (the main testing workflow), and a small decorative mark doesn't need a scroll animation.

## Process (`#process-section`) — journey instrument
- The wheel is gone; the process now uses the **canvas journey** the pillars used to run (a process genuinely *is* a journey, so the route line finally matches its content).
- Desktop only: pinned section, `.journey-track` slides horizontally, `#processCanvas` draws a route line with rolling hills between waypoints, and a glowing marker travels stop to stop. `snapTo` parks it at each stage.
- Stops are `.journey-stop` with `data-y` offsets (`0, -95, 80, -80, 90`) — header + Discovery / Strategy / Creative / Launch.
- Mobile drops the canvas entirely for a plain stacked list — a second pinned section on one page would be repetitive (the pillars already pin).
- Class names are deliberately `.journey-track` / `.journey-stop`, NOT `.journey-lane` — that one now belongs to the pillars' pinned accumulation.

## Principles (`#who-principles`) — pinned accumulation
Pattern borrowed from servetheagency.com, now used at **all breakpoints** (replaced the desktop journey canvas):
- `#who-principles` is `300svh` — the scroll runway. `.journey-lane` is `position: sticky` inside it (pins like the hero), one viewport tall.
- While pinned, JS slides each `.who-principle` row up from the **bottom of the viewport at exactly scroll speed** (1px scroll = 1px travel), then it hard-locks in its slot. Rows accumulate into a numbered index; the section releases after the last one lands.
- **No fade and no easing** — Serve's rows have no entrance keyframe at all; the "slide" is literally the page scrolling and the row stopping. Adding opacity or an ease makes it read as floaty/fading (tried both, both wrong).
- Travel per row is *measured* (gap between its locked slot and the viewport bottom), re-measured on resize + `fonts.ready`.
- Desktop rows are 3-col (`number | statement | explanation`) so they stay one line tall. **Mobile is 2-col with the explanation stacked under the statement** (`grid-row: 2`, 13px/1.5) — it used to be hidden, but "No layers." alone tells a prospective client nothing.
- **Mobile height budget is the constraint.** All four rows must fit ONE pinned viewport, so the mobile title is `clamp(30px, 8.6vw, 46px)` (eased down from 10vw/56px to make room) and the lane padding is trimmed. Verified at 360×640, 375×667 and 430×932: content ends 22–33px *above* the lane bottom with zero row overlaps. Enlarging the title or body here eats that headroom, and overflow makes rows collide rather than scroll — re-measure if you change either.
- **Sticky killers to watch:** `#who-principles` must not be `overflow: hidden` and `.journey-lane` must not have `will-change: transform` — either silently disables `position: sticky`.
- Knobs: `300svh` (hold length), `RISE`/`step` in the JS (travel + stagger).

## Capabilities (`#capabilities`) — centered manifesto
- Replaced the accordion here. Three `.cap-row.cap-scrub` rows, centered, each with a `.cap-title` (slot-machine `.roll-a`/`.roll-b` hover) + a mono `.cap-svcs` list.
- **The `.acc-*` accordion is fully gone from the site** (it left with the FAQ, Aug 2026) — markup, CSS, click handler, and the `document.fonts.ready` re-measure of the pre-opened panel. Recover from git if a future section wants it.
- **Scroll-scrub**: JS drives `translateX` from fully-off-screen → center. Each word's start offset is `vw/2 + ownWidth/2 + BUFFER` so wide and narrow words both begin *completely* hidden; narrower words get a proportionally shorter scroll range (`enters[i]`) so all move at the same px-per-scroll rate and land together at mid-screen.
- **Eased like the footer** (`TAU = 90`ms, frame-rate independent) — scroll input is too coarse (esp. touch/momentum) to write straight to transform.
- **One-way**: a row latches `landed[i]` once centered and never slides back out — scrolling back up must never hide the section's content.
- Hover roll is gated by `#capabilities.is-scrolling` (re-arms 260ms after scroll stops) so rows sliding under a parked cursor don't trigger the flip.
- Widths are measured on load, resize, and `document.fonts.ready` (Cabinet Grotesk changes them).

## Work Section
- `#work` on the homepage is the case-study index (see Pages & Nav above); the old horizontal strip/cards work section is fully removed (markup, CSS, and GSAP block)
- **Currently commented out for launch** (only one project is live). Re-enable = uncomment the `#work` block + the three "Our work" links (nav-pill, menu-overlay, footer).

## Saved for Later
- **Archived effects** live in `_archive/effects/` as self-contained reference files (markup + CSS + JS + notes): `brand-dna-node-network.html` (3D canvas node constellation), `journey-instrument-canvas.html` (canvas path + travelling marker), `blur-focus-text-reveal.html` (paragraphs focus in from blur), `process-wheel.html` (rotating compass dial with counter-rotating labels + cross-fading cards). All three were cut from the homepage Aug 2026 to reduce motion density — reusable elsewhere.
- **Stats bar** (brands shaped / years / independent / industries) — removed from homepage, restore when client roster is established. Full code saved in project memory: `project_stats_section.md`

## Key CSS Patterns
- **Horizontal overflow (index.html): `html` and `body` both use `overflow-x: clip` — NEVER `hidden`.** Anything parked off-screen (e.g. `.cap-scrub` words at `translateX(±100vw)`) widens the mobile layout viewport unless clipped, which shoves `position: fixed` elements (burger) off-screen and enables horizontal scroll. And `hidden` on body (when html isn't `visible`) turns body into a scroll container, which **kills the sticky hero slide-over**. `clip` clips without creating a scroll container. `#capabilities` also has `overflow-x: clip` as a local guard.
- `mix-blend-mode: difference` on fixed elements (`nav`, `#siteWordmark`) is **desktop-only** (`@media (min-width: 769px)`) — on mobile the nav holds only the burger and fixed blend layers cause rendering trouble.
- Clip/reveal: use `overflow: hidden` on parent + `translateY` on child (NOT `clip-path` — causes "eyelid" effect)
- 3D transforms need `transform-style: preserve-3d` on element and `perspective` on parent; avoid `overflow: hidden` on ancestors
- Scroll reveal: `.reveal` class sets initial hidden state, `.visible` triggers transition
- All three files share identical scrollbar, footer mobile, and loader CSS patterns

## Footer Effects (all three files)
- **Stretching wordmark** (Namma-style, **desktop only ≥769px** — matches Namma's own `innerWidth > 767` gate; on mobile the wordmark is only ~89px tall so the whole scrub fits in 89px of scroll, and it renders static instead). A `matchMedia` listener clears the inline transform when crossing below the breakpoint, and `will-change` is scoped to desktop so mobile gets no idle compositor layer. Motion is **eased, not written straight from scroll**: `current += (target - current) * (1 - Math.exp(-dt / TAU))` with `TAU = 90`ms — frame-rate independent, and it smooths the ~3 discrete mouse-wheel notches that span the short scrub range (the equivalent of GSAP `scrub` + Lenis on the reference site). The rAF loop self-halts when settled. Footer wordmark img sits in `.footer-wordmark-wrap`; JS scrubs `scaleY(0 → 1)` on the img with **`transform-origin: top`** (matches studionamma.com — their 51px ScrollTrigger offset is their footer's padding-bottom). We compute the below-space dynamically: `line = vh - (scrollHeight - wrapDocBottom)`, `p = (line - wrapTop) / wrapHeight`. Because the scrub range equals the wordmark's own height and it's top-anchored, its bottom edge stays pinned to the line — it grows/squashes *in place*, ending at exactly max scroll. Vanilla rAF scroll handler (no GSAP — case study pages don't load it). Measure the **wrap**, not the img, for the scrub range — `getBoundingClientRect` on the scaled img returns transformed values. Skipped under `prefers-reduced-motion`.
- **Duplicate-text rollovers**: `.fx-roll` (overflow hidden, `padding-bottom: 4px; margin-bottom: -4px`) holds `.fx-roll-a` + absolute `.fx-roll-b` (second copy, `aria-hidden`); hover on the parent (`a.footer-item`, `.footer-btn`, `.back-to-top-footer`) slides a → `-140%`, b → `0`. Applied to every footer link.

## Footer Layout (all three files, Namma-style)
Structure inside `<footer class="site-footer">` (flex column, `gap: 88px`, wordmark is the **last** element on the page):
1. `.footer-menus` — two `.footer-col`s, space-between: left = site nav (Home / Who we are / What we do / Our work / How we do it / Contact; hash anchors on index, `index.html#…` on case pages), right (right-aligned) = LinkedIn, Instagram (text links, no icons), Privacy Policy (button `#openPrivacy` on index, `index.html#privacy` link on case pages), back-to-top `#backToTop`
2. `.footer-say` — conversational mono row: location line · "Big project?…" · **"Say hello →"** (`.say-hello`) · mailto link · © line (`.footer-note` allows wrapping, `max-width: 24ch`). The `.say-arrow` is **desktop-only (≥769px)**: the row is horizontal there so the arrow points across at the email address sitting next to it, but the row stacks at ≤768px where a right-pointing arrow would aim at nothing. It nudges 4px on `.footer-say:hover`.
3. `.footer-wordmark-wrap` — the stretching wordmark, final thing on the page
Mobile (≤768px): `.footer-say` stacks vertically; `.footer-menus` stays two-column. All links use `.footer-item` (DM Mono 11px uppercase) + `.fx-roll` rollovers.
