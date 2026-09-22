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
- **Keep every loader animation compositor-only (transform/opacity).** The bar used to animate `width`, which runs on the main thread — capped at 60fps on iPhone and stalled by any JS running during load. It broke visibly on iPhone once the scroll scripts added load-time work. It now animates `transform: scaleX()` (`transform-origin: left`), including the JS finish to 100% in `fillBar()`.
- **`loaderDone()` / `window.__osoAfterLoader(fn)`**: the loader signals when it's gone (`oso:loader-done`; every path calls it, 5s safety net). Work that isn't needed while the loader covers the page — the scroll scripts' `ResizeObserver`s — waits for it, so nothing competes with the loader's animation.

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
- **Pin idiom, chained.** `#statement` is `240svh` (the runway) with `.statement-lane` `position: sticky` inside it — the same pattern as `#who-principles`. It's also the first section in `.page-body`, so it's what rides over the sticky hero; then `#who-we-are` (`z-index: 2`, `margin-top: -70svh`) slides over *it*. Hero → statement → about is one continuous chain of sheets.
- **Mobile (≤768px) sets the line in `BebasNeue` caps** at `min(20vw, 10svh)` — ~68% of the viewport, vs ~25% when it was Cabinet at its 34px floor. Bebas is condensed (~0.38em/char vs ~0.5em), and that is what buys the size: in Cabinet the word "strategy," fills the line before the type is big enough to hold the screen. **Desktop stays Cabinet Grotesk sentence case** — so the same sentence is caps on phones and sentence case on desktop. Deliberate, but if it ever reads as inconsistent, the fix is to carry Bebas up to desktop, not to shrink mobile.
  - `min(20vw, 10svh)`: the **vw term sizes it, the svh term is the safety valve.** The lane is one viewport tall and does not scroll, so a pure vw size overruns it on short-but-wide screens. Verified 360×640 / 390×844 / 430×932: 64–68% of viewport, no horizontal overflow, always inside the lane.
  - **The circle is retuned for caps** (`134%` / `158%` / `-0.13em`). Desktop pushes it *down* `+0.18em` for lowercase descenders; caps have none and at `line-height: 0.92` the cap mass sits above the line box centre, so it moves *up* instead. It also has to be wider — an ellipse narrows toward its top and bottom, and the desktop width clipped the D and the S at cap height.
- **The type has NO entrance animation — deliberately.** It carried a scroll-scrubbed word-by-word opacity fill (`.fill-word`, 0.12 → 1 across 0 → +60%) and later a masked word roll-up. Both were removed: this is the site's positioning line, added specifically so visitors know what the studio does *without digging*, and any scrubbed reveal holds it illegible until the reader has scrolled most of a viewport. The line reads instantly; the circle is the only motion.
- **Timing** (viewport units from the section's top): circle draws **+20% → +52%**, cover starts at **+70%** (set by `height − |margin-top| − 100`), lane unsticks at **+140%**. Keep `height − |margin-top| − 100 >` the circle's end.
- **Circled word.** `data-circle-word="details"` on the `h2` makes the split JS wrap that word in `.circle-word` and nest `svg.statement-circle` inside it; trailing punctuation stays outside the loop. **It is drawn by `DrawSVGPlugin`**, scrubbed from `top top-=20%` over `+=32%`, so it finishes at +52% — well before the cover starts at +70%. `ease: 'none'` ties the pen to the scroll, and it reverses on scroll-up. To swap the drawing, replace only the `d` in `CIRCLE_SVG`.
- **GSAP is pinned at 3.13.0 because of this.** DrawSVGPlugin only became free at 3.13 and **404s on 3.12.5**, which is what the page used before. All three GSAP files must stay on the same version. Verified after the bump: 3.13.0 loads, 4 ScrollTriggers, 1 pin, zero console errors.
- **Don't hand-roll the dash maths again.** The first attempt used `stroke-dasharray`/`stroke-dashoffset` directly and needed `pathLength="1"` to survive the SVG being stretched to the word box, broke outright if `vector-effect: non-scaling-stroke` was added (it silently defeats the normalisation), and left the round linecap visible as a stray dot in the "hidden" state. DrawSVG has none of those failure modes; CSS must not set dasharray/dashoffset on this path.
- **The loop is generated, not hand-drawn**: an ellipse sampled at 24 points with smooth low-harmonic radius noise, ~2° tilt, and a 12% overshoot so the tail crosses back over the start. Sampling coarsely matters — at 64 points with integer coordinates the rounding jitter makes it read as a shaky, nervous scribble rather than a confident circling gesture. `stroke-width: 1.6` is in user units and scales with the word, giving ~3px desktop / ~1px mobile.
- The circle is **nudged down** (`calc(-50% + 0.18em)`), not centred: at `line-height: 1.04` a loop big enough to clear the glyphs would cut through the line above, so it hangs into the free space under the last line. Only safe because the circled word is last — circling a mid-line word needs different numbers.
- **Sticky killers apply**: `#statement` must not be `overflow: hidden` and `.statement-lane` must not have `will-change: transform`.
- **One runway for both breakpoints — do not re-shorten mobile.** It was once `190svh` / `-55svh` on mobile to save scroll, but the circle's range is viewport-relative, so the short runway started the cover before the circle finished and `#who-we-are` slid up over a half-drawn loop. Any change to `height` or `margin-top` must keep `height − |margin-top| − 100 >` the circle's end (currently 52).
- The word split runs **inside the GSAP guard**, so with no GSAP the line just renders at full opacity.

## Who We Are (`#who-we-are`) — one statement, one left edge
Restructured Aug 2026. The section was three fragments on three different alignment axes; it now makes a single argument.
- **Structure**: topbar eyebrow → `.who-split` (Venn left, content right). The right column holds `h2.who-statement-head` ("If you lose the details, you lose everything.") then `.who-copy` — everything left-aligned and sharing one left edge.
- **The "Bespoke brand strategy, design, and web…" line moved out to [`#statement`](#statement--pinned-positioning-line)**, where it now opens the page instead of sitting mid-About. It was doing two jobs here: re-answering *what we do* and duplicating the paragraph beneath it.
- **One headline, not two.** The old layout had an italic 300 statement and a bold 700 pull quote at nearly the same size, so neither led. There is now exactly one big type block.
- **Column order**: `h2.who-statement-head` → `.who-copy` (three paragraphs) → `.who-partner-line`. The **first paragraph pays off the headline** — details as craft (the painting analogy: not one big move, the considered moments inside it). Keep a details beat there; before it existed the section made a claim about details in the headline and never returned to it. Deliberately a **craft** argument, not a differentiation one: an earlier draft claimed details are what separate a client from their competitors, which overclaims (not every client has a unique differentiator) and duplicates the positioning job `#capabilities` already does. `.who-partner-line` holds "You're not a client to us. You're a partner." as a standalone emphasis line — sized *between* the headline and the body copy (`clamp(21px, 2vw, 32px)`, weight 500, full-opacity ink vs the copy's 0.65) so it reads as a beat rather than a second headline. Keep it in that band; pushing it up to headline scale re-creates the two-headline problem this section was restructured to fix.
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

## Mobile scroll smoothness (the hand-rolled scroll animations)
**CONFIRMED ROOT CAUSE (Sep 2026): Safari's 60fps cap on 120Hz iPhones.** Safari scrolls the page at 120fps on ProMotion devices but throttles everything the page drives itself — rAF, JS motion, scroll-linked animation — to 60fps (Safari feature flag "Prefer Page Rendering Updates near 60fps", **on by default**; turning it off on the device made the jumping disappear, which confirmed it). The capabilities words move **sideways** while the page moves **up**: vertical position rode the 120Hz scroll, horizontal updated at 60Hz → a staircase. That's why the pillars and the circle were fine (they move over a pinned, static background) and why DevTools device mode never showed it (laptop Chrome, no 120Hz Safari).
- **No scroll-linked technique fixes this** — faster JS, CSS scroll-driven animations, and GSAP `normalizeScroll()` were all tried. normalizeScroll was removed: it moved *page* scrolling onto the JS thread, dragging the whole site down to 60fps on those phones.
- **Fix: on touch devices (`(hover: none) and (pointer: coarse)`) the capabilities words don't scrub.** `html.cap-reveal` disables their scroll animation and gives them a 1.1s expo-out `transition`; JS adds `.cap-in` once each row's centre rises past 85% of the viewport (arithmetic on cached centres, one class per row, never removed). Time-based CSS transitions run at the full 120Hz. Desktop keeps the scroll scrub.
- **Do not use IntersectionObserver for this reveal.** The rows are parked off-screen *sideways* inside `#capabilities { overflow-x: clip }`, so their clipped intersection is empty and they would never register as visible.
- **Process wheel (mobile) had the same problem — fixed the same way.** Its label ring was scroll-scrubbed (rotation tied to scroll) plus GSAP `snap`, which moves the page programmatically; both are held to 60fps on iPhone. On touch devices the section gets `.wheel-steps`: the ScrollTrigger keeps the pin but drops scrub/snap, scroll only picks the active step, and the ring + counter-rotating labels turn to `step × 90°` via a shared 0.8s CSS transition. The ring's transform is written as an explicit `rotate(Ndeg)` string, not `gsap.set` — GSAP prints 0° as `translate(0px, 0px)`, and a transition between mismatched functions falls back to matrix interpolation, which renders a two-step jump (0°→180°) as a mirror flip. Narrow *desktop* windows keep the scrub + snap.
- Rule for future effects: on touch devices, **anything scroll-linked that moves or rotates must be time-based instead** — step it with CSS transitions, and don't use GSAP `snap` (it drives page scrolling from JS). The earlier "motion over a pinned background is OK" assumption was wrong: the wheel sits in a pinned section and still looked low-frame-rate.


**The real fix is CSS scroll-driven animations (`html.sda`).** Even after every JS optimisation below, the pillars and capabilities still jumped on a phone: the page scrolls on the browser's own scroll process while JS-moved elements update from scroll events a frame behind, so they visibly jitter against the page. DevTools device mode can't show this (no address bar, laptop CPU, no separate touch scroll process). Now, where `CSS.supports('animation-timeline: view()')` (Chrome, Safari/iOS 26+) and reduced motion is off, `window.__osoSDA` is true, `html` gets `.sda`, and:
- **Capabilities** run `@keyframes capSlide` on each row's own `view()` timeline, `animation-range: cover var(--cap-a) cover 50%` (cover 50% = row centre at viewport centre). JS converts each row's enter fraction into `--cap-a` and its start offset into `--cap-from`, once per measure. One-way latch = JS adds `.landed` (drops the animation) when a row reaches centre — cheap arithmetic on cached centres.
- **Pillars** read the *section's* named timeline (`view-timeline: --pillars block` on `#who-principles`) — rows are sticky-pinned, so their own view progress never changes. `animation-range: contain var(--r-a) contain var(--r-b)`: for a subject taller than the viewport, `contain` is exactly the pinned runway. JS sets `--travel` / `--r-a` / `--r-b` from the same step/travel maths as the fallback.
- JS never writes these transforms per frame in this mode. Browsers without support keep the JS fallback below, unchanged.
- `animation-timeline` must be declared **after** the `animation` shorthand, which resets it.
- **Headless Chrome cannot verify this.** In `--dump-dom` mode it renders no frames, so scroll timelines report progress 0 and rAF never fires; chained scroll-then-screenshot harnesses drop their output. Verified only that `.sda` is set, all six animations attach, and there are zero errors — the motion itself has to be checked on a real phone.

The fallback JS (and the rules it still follows):
- **Mobile browsers resize the viewport mid-scroll** as the address bar collapses. `resize` fires and `innerHeight` changes under the finger. Anything that re-measures on every resize, or does its maths with live `innerHeight`, jumps. (The pillars' re-measure called `rest()`, snapping every row to its final slot and back.)
- **Use `window.__osoViewport()`** — it reads `100svh` / `100lvh` probe elements, which by definition ignore the address bar. Its `key` only changes on a real resize, so resize handlers compare keys and skip everything else. `svh` for maths tied to svh-based layout (pinned runway); `lvh` where something must start below the tallest the viewport gets (rows parked off-screen).
- **No layout reads in the frame loop.** Row positions are cached in document coordinates at measure time; per-frame code does only `scrollY` arithmetic. Capabilities used to call `getBoundingClientRect` on every row every frame, interleaved with transform writes — a forced reflow per row per frame. A `ResizeObserver` on `<body>` re-reads the cached positions if the page above reflows (body size ignores the address bar).
- `translate3d` + the existing `will-change: transform` so rows slide as composited textures instead of repainting glyphs.
- Don't re-add a class on every scroll event (`is-scrolling`) — guard with `contains()` so style isn't invalidated many times a second.

## Key CSS Patterns
- **Horizontal overflow (index.html): `html` and `body` both use `overflow-x: clip` — NEVER `hidden`.** Anything parked off-screen (e.g. `.cap-scrub` words at `translateX(±100vw)`) widens the mobile layout viewport unless clipped, which shoves `position: fixed` elements (burger) off-screen and enables horizontal scroll. And `hidden` on body (when html isn't `visible`) turns body into a scroll container, which **kills the sticky hero slide-over**. `clip` clips without creating a scroll container. `#capabilities` also has `overflow-x: clip` as a local guard.
- `mix-blend-mode: difference` on fixed elements (`nav`, `#siteWordmark`) applies at **all breakpoints**, in all three files. It was briefly desktop-only (Aug 2026, commit `e4d44cb`) after being wrongly blamed for the mobile missing-burger / phantom-nav bug — the actual cause was off-screen `.cap-scrub` words widening the mobile layout viewport (fixed by `overflow-x: clip`). Don't re-scope it to desktop without a device repro that isolates blend mode itself.
- Clip/reveal: use `overflow: hidden` on parent + `translateY` on child (NOT `clip-path` — causes "eyelid" effect)
- 3D transforms need `transform-style: preserve-3d` on element and `perspective` on parent; avoid `overflow: hidden` on ancestors
- Scroll reveal: `.reveal` class sets initial hidden state, `.visible` triggers transition
- All three files share identical scrollbar, footer mobile, and loader CSS patterns

## Footer Effects (all three files)
- **Stretching wordmark** (Namma-style, **desktop only ≥769px** — matches Namma's own `innerWidth > 767` gate; on mobile the wordmark is only ~89px tall so the whole scrub fits in 89px of scroll, and it renders static instead). A `matchMedia` listener clears the inline transform when crossing below the breakpoint, and `will-change` is scoped to desktop so mobile gets no idle compositor layer. Motion is **eased, not written straight from scroll**: `current += (target - current) * (1 - Math.exp(-dt / TAU))` with `TAU = 90`ms — frame-rate independent, and it smooths the ~3 discrete mouse-wheel notches that span the short scrub range (the equivalent of GSAP `scrub` + Lenis on the reference site). The rAF loop self-halts when settled. Footer wordmark img sits in `.footer-wordmark-wrap`; JS scrubs `scaleY(0 → 1)` on the img with **`transform-origin: top`** (matches studionamma.com — their 51px ScrollTrigger offset is their footer's padding-bottom). We compute the below-space dynamically: `line = vh - (scrollHeight - wrapDocBottom)`, `p = (line - wrapTop) / wrapHeight`. Because the scrub range equals the wordmark's own height and it's top-anchored, its bottom edge stays pinned to the line — it grows/squashes *in place*, ending at exactly max scroll. Vanilla rAF scroll handler (no GSAP — case study pages don't load it). Measure the **wrap**, not the img, for the scrub range — `getBoundingClientRect` on the scaled img returns transformed values. Skipped under `prefers-reduced-motion`.
- **Duplicate-text rollovers**: `.fx-roll` (overflow hidden, `padding-bottom: 4px; margin-bottom: -4px`) holds `.fx-roll-a` + absolute `.fx-roll-b` (second copy, `aria-hidden`); hover on the parent (`a.footer-item`, `.footer-btn`, `.back-to-top-footer`) slides a → `-140%`, b → `0`. Applied to every footer link.

## Footer Layout (all three files)
Structure inside `<footer class="site-footer">` (flex column, `gap: 64px`, wordmark is the **last** element on the page):
1. `.footer-lead` — the contact block and the footer's typographic anchor: mono "Say hello" label → `a.footer-email` (Cabinet Grotesk, `clamp(30px, 5vw, 72px)`, `.fx-roll` + a `→` that nudges on hover) → `.footer-lead-note` ("A full brief or half an idea. Both work.")
2. `.footer-grid` — three labelled columns sharing one top and left edge: **Sitemap** (Home / Who we are / What we do / Our work / How we do it / Contact) · **Elsewhere** (LinkedIn, Instagram) · **Studio** ("Based in Boston. Working anywhere."). Column headings are `.footer-col-label`.
3. `.footer-meta` — thin bar on the site's hairline-gradient divider: © line left, Privacy Policy + back-to-top `#backToTop` right. Privacy is a `#openPrivacy` button on index, an `index.html#privacy` link on case pages.
4. `.footer-wordmark-wrap` — the stretching wordmark, final thing on the page
Mobile (≤768px): `.footer-grid` goes 2-up with **Studio spanning the full row beneath**; `.footer-meta` stacks. All links use `.footer-item` (DM Mono 11px uppercase) + `.fx-roll` rollovers.

**On the resemblance to studionamma.com:** the stretching wordmark is deliberately modelled on theirs and is staying. The rest was too — two columns with the right one right-aligned, then a single conversational row of uniform small mono with the copyright folded in — and was **restructured Aug 2026** to move away from it. The big email line is the main departure; don't collapse the footer back to uniformly small type. The old `.footer-menus` / `.footer-say` / `.say-hello` / `.say-arrow` classes are gone.
