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
