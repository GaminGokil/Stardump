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
