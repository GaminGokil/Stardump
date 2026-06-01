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
