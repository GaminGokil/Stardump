# Stardump build log

## S-000 — bootstrap
- created index.html (CSS design tokens, empty script block), README.md, LICENSE

## S-001a — canvas + DPI resize
- #sky canvas fixed inset z-index:1, DPI-aware resize via setTransform

## S-001b — background star field
- 100–300 stars density 1/2800px², 3 depth layers (50/30/20%), 60/28/12 color mix
- mouse/smooth objects initialized for parallax (S-002)

## S-001c — twinkle loop
- rAF loop, alpha = baseAlpha*(0.7+0.3*sin(t*freq+phase)), no per-frame allocations

## S-002 — mouse parallax
- mousemove → exponential smooth (alpha=0.04) → layer offset up to 9px opposite cursor

## S-003 — nebula gradient
- two CSS pseudo-element radial gradients: warm purple upper-left, cool blue lower-right
- static, pointer-events:none, z-index:1

## S-004 — input bar
- fixed bottom-center, 520px max 80vw, pill border, ease-soft focus transition
- Enter → addThoughtStar stub + clear; STATE object initialized

## S-005a — placement algorithm
- placeStar(): 60-attempt min-distance (90px) constraint, falls back to unconstrained
- addThoughtStar() pushes to STATE.stars, calls renderStars()

## S-005b — star DOM rendering
- #stars container fixed inset z-index:3
- renderStars() diffs existing DOM nodes, positions via left/top
- .star-core warm-gold box-shadow glow, .star-glow radial gradient ring

## S-006a — fade-in + pulse animations
- star-in: 0→scale(0.6) → scale(1.12) → scale(1), 1.5s ease-pop, runs once
- star-pulse: opacity 1→0.78→1, 3.4–5.6s ease-soft, infinite, random phase offset
- animatedIds Set prevents fade-in replay on re-render

## S-006b — tooltip + resolve
- click star → tooltip (text + resolve button), one at a time, click-outside closes
- escapeHtml sanitizes user text before innerHTML
- triggerSupernova stub → removeStar for now (real animation S-007)

## S-007a — supernova core
- nova-core: scale 1→5→11→14, white→gold-mid→gold-hot, opacity 0 at end, 1.8s ease-fade
- nova-glow: 36px→200px expand, 1.8s ease-fade
- triggerSupernova adds .supernova class, removes star after 2000ms

## S-007b — supernova particles
- 16 radial .nova-particle spans with CSS custom props --tx/--ty
- nova-particle: translate 0→(tx,ty), opacity 0→1→0, 1.6s ease-fade

## S-008a — constellation lines
- SVG overlay z-index:2 pointer-events:none
- nearest-2-neighbor per star, deduplicated edges via sorted id key
- c-line fade-in 900ms ease-soft with 70ms stagger

## S-009 — constellation name
- keyword regex (8 tags) → named combinations, fallback pool (9 names) keyed by sum(text.length*(id+1))
- #constellation-name: italic Georgia 26px, 3px letter-spacing, fades in/out

## S-010 — counter + brand
- #brand fixed top-left: "stardump" uppercase, ui-text-dim
- #counter fixed top-right: "N stars · the [name]" when stars exist
- updateCounter() called from renderStars()

## S-011 — persistence
- LS_KEY = 'stardump-v1-sky', save on every addThoughtStar + removeStar
- loadState() on page load, pre-populates animatedIds so no fade-in replay on restore
- initial renderStars() call after loadState() to render persisted stars

## S-012 — polish pass
- star-core box-shadow: deeper (8/18/32px)
- star-glow opacity: 0.45 → 0.55
- #constellation-name: opacity:0 initial state
- easing audit passed: all transitions use ease tokens

## S-013a — edge case verification
- maxlength=120 confirmed on input
- tooltip -webkit-line-clamp:2 confirmed
- text.slice(0,120) in addThoughtStar confirmed
- placeStar fallback confirmed
- animatedIds ordering confirmed
- triggerSupernova → removeStar → saveState chain confirmed

## S-014 — journal
- JOURNAL.md filled in: what it is, what went well, what broke, AI disclosure, v1.1 wishlist

## S-013b — cross-browser smoke
- Chrome 1920×1080: zero console errors, star field + nebulae render, constellation lines stagger in, supernova fires and star is removed, counter updates, persistence confirmed (reload restores stars)
- Chrome 1440×900: identical result, layout scales correctly, no overflow
- Bug found and fixed: counter template used `the ${name}` but constellation names already include "The" prefix → changed to `${name}` to eliminate double-the
- No Safari-specific fixes needed for Chrome smoke pass; mix-blend-mode renders correctly in Chrome

## S-016 — polish(sky): stronger cursor parallax + larger bg stars
- Parallax factor increased from 0.2 to 0.4 (doubled depth effect)
- Background star size scaled by layer+1 (was fixed): `ctx.arc(..., s.r * (s.layer + 1), ...)`  instead of `ctx.arc(..., s.r, ...)`
- Visual depth more pronounced; far stars no longer feel flat

## S-017 — fix(constellation): reset title below 5 stars  
- Constellation name visibility tied to star count
- If `STATE.stars.length < 5`, `#constellation-name` opacity→0, render early-return
- Prevents orphan title text when few stars exist

## S-018 — feat(sky): camera scaffold at identity
- Added `const view = { ox: 0, oy: 0, scale: 1 };` — camera state
- Helpers `s2wx(sx)` / `s2wy(sy)` convert screen→world coords
- `applyView()` applies scale+translate to `#stars`, `#cgroup`, and open tooltip
- SVG constellation moved into `<g id="cgroup">` for grouped transform
- Identity transform on load: sky pixel-identical, zero drift

## S-019 — feat(sky): drag-to-pan foreground
- `mousedown` listener: left-button-only, skips input/tooltip/star targets, captures offset
- `mousemove` listener: 4px hysteresis before `moved=true`, applies pan to `view.ox/oy`
- `mouseup` listener: clears `moved` flag, saves state
- Cursor: `grab` idle → `grabbing` while panning
- Click handler guard: `if(moved) return;` suppresses tooltip after drag
- Foreground (stars+constellation) pans smoothly; background (S-020) pending

## S-020 — feat(sky): infinite wrapped background
- `genStars()` now generates stars across 2× viewport tile: `tileW = innerWidth*2, tileH = innerHeight*2`
- Star count scales with tile area: `clamp(tileW*tileH/2800, 200, 900)` (was 100–300 fixed)
- `LAYER_P = [0.30, 0.55, 0.80]` — parallax factors per layer (far→near); offsets = `view.ox*p`
- `loop()` computes wrapped base coords: `((s.x - offX) % tileW + tileW) % tileW` for continuous tiling
- Tiling loop draws star copies across viewport + margin (`gx/gy ±tileW/tileH`), one path per star
- Panning now reveals an endless starfield; layers drift at different rates for depth
- Fixed initialization order: moved `const view = {...}` before `loop()` call to avoid ReferenceError

## S-021 — feat(stars): place new thoughts in current view
- `placeStar()` now computes viewport bounds in world space: `wx0/wx1` (horizontal), `wy0/wy1` (vertical)
- Bounds account for camera offset and scale: `wx0 = view.ox + M/view.scale` (left edge)
- Vertical margin includes 80px for input bar: `wy1 = view.oy + (innerHeight - M - 80)/view.scale`
- Sampling: `x = wx0 + Math.random()*(wx1-wx0)` yields world coords in current view
- Min-distance constraint (90 units) unchanged; applies in world space as before
- Fallback return also uses viewport rectangle, ensuring consistent placement
- New thoughts now land on-screen when added, even after panning/zooming away

## S-022 — feat(sky): scroll-to-zoom foreground
- `wheel` listener (`passive:false`) captures world-space anchor before scaling: `wx = s2wx(e.clientX)`
- Scale: `view.scale = clamp(view.scale * Math.exp(-e.deltaY * 0.0015), 0.4, 3)` — exponential, clamped
- Re-anchor: `view.ox = wx - e.clientX/view.scale` keeps the cursor point fixed in world space
- `applyView()` called immediately; debounced `saveState` fires 400ms after last wheel event
- Tooltip counter-scaled at creation: `transformOrigin: left bottom`, `scale(1/view.scale)` keeps text readable
- Clamp confirmed: min 0.4, max 3.0; zero console errors

## fix(sky): bg stars render as triangles + lag (moveTo + cull)
- Root cause (regression from S-020 tiling loop): multiple `ctx.arc()` in one `beginPath()` with no `ctx.moveTo()` between them — Canvas joins consecutive arcs with a straight line, wiring every circle to the next → "triangles"/spokes
- Each star's `fill()` filled one large self-intersecting polygon spanning the whole 2× tile; ×900 stars/frame caused the lag (idle had dropped to ~37fps)
- Fix 1: `ctx.moveTo(gx + s.r, gy)` before each `arc` → clean disjoint circles, no connecting lines
- Fix 2: off-viewport cull — tile is 2× the viewport so at most one copy per star is ever on-screen; `if (gx + s.r < 0 || gx - s.r > innerWidth) continue;` (and same for gy) skips the ~8/9 wasted arc fills
- Measured @ 1920×1080: idle / pan / zoom all 60fps, 0 long frames (was ~37fps); zero console errors; stars render as discrete points with full edge-to-edge coverage

## fix(sky): star clicks dead after a pan (stale moved flag)
- Found during an S-018→S-022 audit: after any drag-pan, the first click on a star failed to open its tooltip
- Cause (regression in S-019): a star's `mousedown` early-returns (panning never starts on a star) but it returned *before* `moved = false`, so the flag stayed stale-true from the previous pan; the `#stars` click handler's `if (moved) return;` then suppressed the tooltip until the user clicked empty space to reset
- Fix: hoist `moved = false` above the star/input/tooltip early-return so the flag tracks the current press cycle
- Verified @ 1920×1080: pan → click star opens tooltip; a pure pan opens no tooltip; clean clicks unaffected; zero console errors
- Audit also reviewed (no change needed): `applyView` transform math correct; tooltip counter-scale correct; `placeStar` at zoom acceptable. Background-not-coupled-to-zoom and view-not-persisted are expected — they are the next planned steps (S-023, S-025)

## S-023 — feat(sky): couple background to zoom
- `bgScale() = 1 + (view.scale - 1) * 0.5` — background follows zoom at half the foreground rate
- `bs`, tile period `TW = tileW*bs` / `TH = tileH*bs` hoisted once per frame (independent of star)
- Per-star radius `r = s.r*bs`; wrap math and off-viewport cull operate in scaled space
- Zoom now reads cohesive front-to-back instead of foreground sliding over a static field
- Verified @ 1920×1080: bgScale 0.4→0.7, 1→1, 3→2; 60fps / 0 long frames at both zoom extremes; bg stars visibly grow with zoom; no triangles; zero console errors

## S-024 — feat(sky): double-click recenter glide
- `easeK(k)` cubic in/out (not linear) drives a 600ms `glideTo(tx,ty,ts)` that lerps `ox/oy/scale` via rAF, `saveState()` on arrival
- `recenter()` targets the centroid of `STATE.stars` at scale 1 (origin when empty): `tx = cx - (innerWidth/ts)/2`
- `dblclick` listener ignores `.star, #input-wrap, .tooltip`; recenters otherwise
- Verified @ 1920×1080: from ox/oy 4000+ @ scale 2.5, double-click empty space glides smoothly (eased ~600ms) and lands exactly on the centroid at scale 1; double-clicking a star does not recenter; zero console errors

## S-025 — feat(persist): save camera + migrate
- `saveState()` now persists `view: { ox, oy, scale }` alongside stars/nextId
- `loadState()` restores it behind an `if (data.view)` guard → legacy saves (no view key) load at identity
- Sanitized: `ox/oy` via `+x || 0`, `scale` via `clamp(+scale || 1, 0.4, 3)` (matches wheel-zoom clamp)
- Init order `loadState() → renderStars() → applyView()` already applies the restored camera
- Verified @ 1920×1080 (V4): set ox/oy/scale → reload restores camera + stars exactly (incl. `#stars` transform); legacy save (no view) → identity, stars intact; corrupt view (ox NaN, scale 99) → sanitized to 0/0 and clamp 3; zero console errors

## S-026 — polish(sky): zoom/pan polish + cross-browser (audit, no code change)
- Audit step — all five checks passed with the code already consistent; no `index.html` change needed
- Easing: zero `linear` in the file; all CSS `transition:` rules (lines 48/74/91/133) use `var(--ease-soft)`; animations use `--ease-pop/soft/fade`; glide uses `easeK` (cubic). Twinkle "linear" exception is the `Math.sin` alpha calc in `loop()`, not the literal keyword
- Cursor: `body{cursor:grab}` idle / `body.panning{cursor:grabbing}`; class added in `mousemove`, removed **unconditionally** in `mouseup` (before the `if(moved)` save) → cannot stick after release
- Clamp consistency: `wheel` and `loadState` both `clamp(…, 0.4, 3)`; recenter `ts = 1` sits inside that range — identical bounds everywhere scale is set
- Performance @ 1920×1080: combined pan+zoom 60.6fps, 0 long frames. Tooltip readability: counter-scale holds the tooltip at an identical 180×74px / 13px font at scale 1, 0.4 and 3
- Cross-size smoke: 1920×1080 and 1440×900 both clean — canvas matches viewport, full pan→zoom→tooltip→recenter chain works, layout correct, zero console errors. Safari not testable on Windows (deferred per "Safari if available")
- Note for follow-up (not S-026): repo has accumulated test-artifact PNGs + `.playwright-mcp/` / `.worktrees/` untracked, and `/docs` in `.gitignore` forces `git add -f` for this log — candidates for a `.gitignore` cleanup


## S-028 - polish(stars): inline SVG star icons
- PLAN.md: added an "Icons exception" to the hard constraints - inline `<svg>` icons (drawn in-file, themed via `fill: currentColor`) are allowed (no request, no package); icon fonts / external sprites / CDN/`<link>`/`<script>` sources still banned; decorative icons get `aria-hidden="true"` and reuse a shared `.icon-star` class
- New `.icon-star` CSS: `width/height: 1em`, `vertical-align: -0.12em`, `fill: currentColor`, filter transition - scales with text and inherits color
- Replaced both unicode `<glyph>` star glyphs with the same inline 8-point-star `<svg>` path: welcome "begin" and the tooltip "mark as resolved" button
- Fixed three prior welcome-CSS bugs surfaced in the same block: `display: inline-clock` -> `inline-flex` (+ align-items/gap), missing `;` after `font-size: 28px`, stray `<` left after the old begin glyph
- welcome-begin restyled: idle gold-mid -> hover gold-hot with text-shadow + icon drop-shadow glow (eased)
- Copy fix (same overlay): "wehn a worry passes" -> "when a worry passes"
- Verified @ 1920x1080: welcome renders gold "begin" + crisp SVG star (display inline-flex confirmed, icon fill resolves to gold via currentColor at 1em); clicking a star opens the tooltip with the SVG resolve icon; no unicode star glyph anywhere in the DOM; zero console messages on load and after both interactions; saved sky/state restored exactly after the first-run test. Verify: V1,V2,V5

## S-029 - feat(input): Escape closes tooltip / dismisses welcome
- New `window` `keydown` listener: on `Escape`, if a tooltip is open -> `closeTooltip()` and return; else if the welcome overlay is showing (`#welcome.show`) -> `dismissWelcome()`
- Placed as the last statement in the script (after `maybeShowWelcome()`); reuses existing `openTooltip` / `closeTooltip` / `dismissWelcome`, zero new deps, no out-of-scope feature
- Tooltip takes priority over welcome; non-Escape keys ignored (no false triggers)
- Side fix: reverted a re-introduced welcome-copy typo found unsaved in the editor buffer ("wehn" -> "when", originally fixed in S-028) so the commit ships clean copy
- Authored by hand-typing into Cursor via windows-mcp (Cursor-focus verified before each action per docs/WINDOWS_MCP.md); the one-word typo revert was a direct edit for precision
- Verified @ 1920x1080 (chrome-devtools, served via http://localhost:8765): welcome shows -> Escape removes `.show`, sets seenWelcome, removes overlay after fade; star tooltip open -> Escape closes it (openTooltip null, no `.tooltip` in DOM); 'a' key leaves tooltip intact; zero console messages on load and after all interactions. Verify: V1,V2,V5
