# Stardump v1 — Implementation Plan (subagent-ready)

## Context

Greenfield Hack Club "Stardance" submission. Repo currently contains only `AGENTS.md`, `CLAUDE.md` (one-line `@AGENTS.md` include), and `.git/`. No code yet.

Goal: ship a single-file static web app (`index.html`) per the PRD — worry input → twinkling-star → constellation → supernova, persisted to `localStorage`, deployed to GitHub Pages. ~16 active hours, solo, build weekend.

**Delegation model:** orchestrator (Claude Sonnet 4.6) hands each `S-xxx` step to a subagent. Each step below is written as a self-contained brief: the subagent gets the brief verbatim (plus the current `index.html`) and produces the diff for that step only. Subagents do not see the PRD or this plan's other steps — every constant, selector, regex, and easing they need is inlined into their brief.

**Hard constraints (binding — repeat in every subagent brief):**
- Single file: `index.html` with HTML, CSS, JS inline. **No npm, no build, no CDN, no frameworks, no `<script src=…>` for remote URLs, no `<link rel=stylesheet href=…>` for remote URLs.** Vanilla ES2020+.
- **Icons exception:** inline `<svg>` icons (drawn in-file, themed via `fill: currentColor`) are **allowed** — they add no request and no package, so they don't break the zero-dependency rule. Still banned: icon fonts (Font Awesome, etc.), external SVG sprites, and any CDN/`<link>`/`<script>` icon source. Decorative icons get `aria-hidden="true"`; reuse a shared class (e.g. `.icon-star`) rather than copy-pasting paths.
- No backend, no network calls, no analytics, no telemetry.
- State only in `localStorage` under key `stardump-v1-sky`.
- Desktop demo surface, click-only. Mobile must not break.
- Do **not** add features from the PRD out-of-scope list (APOD, audio, PNG export, decay, AI naming, settings, edit/undo, multiple skies).
- No `console.log` in committed code. No `linear` easing outside the twinkle loop.

**Process constraints (AGENTS.md):**
- One `S-xxx` micro-step per commit. Conventional Commits. Scopes: `sky`, `stars`, `supernova`, `constellation`, `naming`, `persist`, `input`, `docs`, `deploy`.
- Each commit body cites `Verify: V#,V#,V#`. V5 mandatory always.
- Append a block to `docs/PROGRESS.md` per commit.
- `chrome-devtools` MCP is the only required MCP (visual verify).

---

## Global design tokens

```css
:root {
  --bg-base: #02030a;
  --star-white: #fff5e8;
  --star-gold-mid: #ffc880;
  --star-gold-hot: #ff8a3d;
  --cool-1: #6aa0ff;
  --cool-2: #8fb8ff;
  --nebula-warm: rgba(120, 60, 180, 0.18);
  --nebula-cool: rgba(40, 90, 200, 0.16);
  --ui-text: rgba(255, 245, 232, 0.78);
  --ui-text-dim: rgba(255, 245, 232, 0.42);
  --ease-soft: cubic-bezier(0.22, 0.61, 0.36, 1);
  --ease-pop:  cubic-bezier(0.34, 1.56, 0.64, 1);
  --ease-fade: cubic-bezier(0.4, 0.0, 0.2, 1);
}
```

**Z-index layers:**
- `1` — background canvas
- `2` — constellation SVG (`pointer-events:none`)
- `3` — thought star DOM container
- `10` — UI overlays (brand, counter, name, input)
- `20` — open tooltip

**State shape:**
```js
const STATE = { stars: [], nextId: 0 };
const LS_KEY = "stardump-v1-sky";
```

---

## Step list

| ID | Scope | Key behavior | Verify |
|---|---|---|---|
| S-000 | Bootstrap | `index.html`, `README.md`, `LICENSE`, `docs/PROGRESS.md` | V1,V2,V5 |
| S-001a | Canvas | DPI-aware resize | V1,V2,V5 |
| S-001b | Stars | bgStar generation, static draw | V1,V2,V5 |
| S-001c | Twinkle | rAF loop, sin-wave alpha | V1,V2,V5 |
| S-002 | Parallax | Mouse → exponential smooth → layer offset | V1,V2,V5 |
| S-003 | Nebula | CSS pseudo-element radial gradients | V1,V2,V5 |
| S-004 | Input | Styled input bar, Enter → addThoughtStar stub | V1,V2,V5 |
| S-005a | Placement | placeStar() 60-attempt algo + STATE push | V1,V2,V5 |
| S-005b | Star DOM | renderStars(), core+glow markup | V1,V2,V5 |
| S-006a | Animations | fade-in (1.5s pop), pulse (3.4–5.6s soft) | V1,V3,V5 |
| S-006b | Tooltip | Click → tooltip + resolve button | V1,V2,V5 |
| S-007a | Supernova core | scale 1→14×, hue shift, glow expand | V1,V3,V5 |
| S-007b | Particles | 16 radial particles spray | V1,V3,V5 |
| S-008a | Constellation | SVG nearest-2-neighbor lines, stagger fade | V1,V2,V5 |
| S-009 | Naming | Keyword regex + fallback pool | V1,V2,V5 |
| S-010 | Counter | Brand mark + `N stars · the [name]` | V1,V2,V5 |
| S-011 | Persist | localStorage save/restore | V1,V4,V5 |
| S-012 | Polish | Easing audit, glow tuning, copy | V1,V3,V5 |
| S-013a | Edge cases | Long text, 50+ stars, maxlength confirm | V1,V4,V5 |
| S-013b | X-browser | Chrome + Safari @ 1920×1080 & 1440×900 | V1,V5,V6 |
| S-014 | Journal | JOURNAL.md with AI disclosure | V1,V5,V2 |
| S-015 | Deploy | GitHub Pages, live URL in README | V4,V5,V6 |

---

## Subagent batches (5 agents)

| Agent | Steps |
|---|---|
| Agent 1 | S-000, S-001a, S-001b, S-001c |
| Agent 2 | S-002, S-003, S-004 |
| Agent 3 | S-005a, S-005b, S-006a, S-006b |
| Agent 4 | S-007a, S-007b, S-008a, S-009, S-010 |
| Agent 5 | S-011, S-012, S-013a, S-014 |

S-013b (cross-browser) and S-015 (deploy) are orchestrator steps.

---

## Verify legend

| ID | Check |
|---|---|
| V1 | Zero console errors/warnings on load and interaction |
| V2 | Manual smoke of step's behavior against its FR |
| V3 | Animation feel @ 1920×1080 full screen |
| V4 | Mutate → reload → state restores exactly |
| V5 | `git diff` self-review: no stray files, no deps, no console.log, no out-of-scope feature |
| V6 | Chrome + Safari @ 1920×1080 and 1440×900 |

---

## Acceptance checklist (must be green before S-015)

- [ ] Loads in <1 s on a fresh tab
- [ ] Typing → star landing feels good (no jank)
- [ ] Supernova is genuinely impressive at full screen
- [ ] Constellation lines connect correctly, no orphan stars
- [ ] Procedural name updates when keyword mix changes
- [ ] Reload preserves all stars
- [ ] No console errors
- [ ] Works Chrome + Safari @ 1920×1080 and 1440×900
- [ ] README has 2-sentence summary + live URL
- [ ] JOURNAL.md exists with honest AI-assist disclosure
- [ ] Repo public, MIT licensed
- [ ] Live URL works on a fresh device with no cache

---

## Risks

- **Supernova underwhelms** → split S-007a/b, allow S-007c; always V3 at full screen.
- **Dependency creep** → V5 every commit; reject any diff with CDN URL or `import`.
- **Feature creep** → out-of-scope list is binding; reject APOD, audio, PNG, decay, settings.
- **Safari divergence** → `mix-blend-mode: screen` on nebula is the likely culprit; fallback to plain opacity.

---

# Stardump v1.x — Infinite pan + zoom canvas (Haiku-sized micro-steps)

## Context

Stardump renders everything in fixed viewport pixels: background stars (`bgStars` on `#sky`), thought stars (`#stars` DOM, `left/top` in screen px), constellation lines (`#constellation` SVG), and `placeStar()` all assume one screen-sized space. Goal: an **infinite canvas you can pan and zoom**, with a world-space camera. Fixed UI (brand, counter, constellation name, input, nebula) stays pinned to the viewport.

Decisions: **pan + zoom**; **recenter + place-in-view** (new thoughts land in the current view; double-click empty space glides back to the centroid of all stars). In-bounds vs the PRD out-of-scope list. Constraints unchanged: one `index.html`, no deps/build/CDN, vanilla ES2020+, `localStorage` only, mouse-only (wheel + drag are desktop inputs), no `console.log`, no `linear` easing outside the twinkle loop.

**Working style:** broken into 80 very small steps, each a single concrete edit a weak model (Claude Haiku) can complete from the step text alone. Steps grouped under commits (`S-016`…`S-026`); one commit per group, each cites `Verify:` and appends a `docs/PROGRESS.md` block (AGENTS.md §4–6).

## Core model (canonical names + reference snippets)

Camera: `const view = { ox: 0, oy: 0, scale: 1 };` — world point `w` → screen `((w.x - ox) * scale, (w.y - oy) * scale)`.

```js
// R1 — screen→world helpers
function s2wx(sx){ return sx / view.scale + view.ox; }
function s2wy(sy){ return sy / view.scale + view.oy; }

// R2 — apply camera to foreground (DOM stars + SVG constellation group + open tooltip)
function applyView(){
  const t = view.scale;
  document.getElementById('stars').style.transform =
    `scale(${t}) translate(${-view.ox}px, ${-view.oy}px)`;
  const g = document.getElementById('cgroup');
  if (g) g.setAttribute('transform', `scale(${t}) translate(${-view.ox} ${-view.oy})`);
  if (openTooltip) openTooltip.style.transform = `translateY(-100%) scale(${1/t})`;
}

// R3 — eased lerp for recenter glide (cubic in/out; not linear)
function easeK(k){ return k < .5 ? 4*k*k*k : 1 - Math.pow(-2*k+2, 3)/2; }
function glideTo(tx, ty, ts){
  const sox=view.ox, soy=view.oy, ss=view.scale, t0=performance.now(), dur=600;
  (function step(now){
    const k = Math.min(1, (now - t0)/dur), e = easeK(k);
    view.ox = sox+(tx-sox)*e; view.oy = soy+(ty-soy)*e; view.scale = ss+(ts-ss)*e;
    applyView();
    if (k < 1) requestAnimationFrame(step); else saveState();
  })(performance.now());
}

// R4 — background parallax + damped zoom
const LAYER_P = [0.30, 0.55, 0.80];          // far→near follow factor
function bgScale(){ return 1 + (view.scale - 1) * 0.5; }  // distant stars zoom less
```

Foreground transform string is `scale(s) translate(-ox px,-oy px)` with `transform-origin:0 0`; `.star` `left/top` become **world** coords (no code change to `renderStars`). SVG `<g>` uses the attribute form (no `px`).

---

## S-016 `polish(sky): stronger cursor parallax + larger bg stars` — commit pending tree work
1. `git add -p index.html`; stage **only** the two already-applied hunks: `r: 0.4 + Math.random() * 1.3,` (~line 180) and the `ctx.arc(... (s.layer + 1) * 12 ...)` line (~line 220).
2. Commit with that message + `Verify: V1,V2,V5`.
3. Append a `docs/PROGRESS.md` block summarizing the parallax/size tuning.

## S-017 `fix(constellation): reset title below 5 stars` — commit pending tree work
4. Stage the remaining `renderConstellation` hunk (hoisted `nameEl` + `if (… < 5){ nameEl.style.opacity='0'; return; }`).
5. Commit with that message + `Verify: V1,V2,V4`.
6. Append a `docs/PROGRESS.md` block for the title reset.

## S-018 `feat(sky): camera scaffold at identity` (no visual change)
7. Add `const view = { ox: 0, oy: 0, scale: 1 };` immediately after `const STATE = …` (index.html:228).
8. Confirm `let openTooltip` (index.html:386) is declared before `applyView`; place camera helpers below `closeTooltip`.
9. Add helper `s2wx` from **R1**.
10. Add helper `s2wy` from **R1**.
11. Add the `applyView` function from **R2**.
12. In the `#stars` CSS rule (index.html:52) append `transform-origin: 0 0; will-change: transform;`.
13. In `renderConstellation` (index.html:302) after `const svg = …`, get-or-create `<g id="cgroup">` and append to `svg`.
14. Replace `svg.innerHTML = '';` with `g.innerHTML = '';` (keep before the `< 5` early-return).
15. Change the line append `svg.appendChild(line)` → `g.appendChild(line)`.
16. At the end of `renderConstellation` (after the name block) add `applyView();`.
17. After the init `renderStars();` (index.html:441) add `applyView();`.
18. Verify identity: sky pixel-identical, zero console errors. `Verify: V1,V2,V5`.

## S-019 `feat(sky): drag-to-pan foreground`
19. Add state line near the camera: `let isDown=false, moved=false, downX=0, downY=0, startOx=0, startOy=0;`.
20. Add `window` `mousedown` listener: `if(e.button!==0) return;` then `if(e.target.closest('#input-wrap, .tooltip, .star')) return;`.
21. In `mousedown`, set `isDown=true; moved=false; downX=e.clientX; downY=e.clientY; startOx=view.ox; startOy=view.oy;`.
22. Add `window` `mousemove` listener guarded by `if(!isDown) return;`; compute `const ddx=e.clientX-downX, ddy=e.clientY-downY;`.
23. In that `mousemove`, set `if(!moved && Math.hypot(ddx,ddy)>4){ moved=true; document.body.classList.add('panning'); }`.
24. In that `mousemove`, when `moved`: `view.ox = startOx - ddx/view.scale; view.oy = startOy - ddy/view.scale; applyView();`.
25. Add `window` `mouseup` listener: `isDown=false; document.body.classList.remove('panning'); if(moved) saveState();`.
26. In the `#stars` click handler (index.html:420) add as first line `if(moved) return;` to suppress tooltip after a drag.
27. Add CSS `body{ cursor: grab; }`.
28. Add CSS `body.panning{ cursor: grabbing; }`.
29. Verify: drag moves stars + lines together; a plain click still opens a tooltip. `Verify: V1,V2,V5`.

## S-020 `feat(sky): infinite wrapped background`
30. Add `let tileW = 0, tileH = 0;` beside `let bgStars = [];` (index.html:163).
31. Add `const LAYER_P = [0.30,0.55,0.80];` from **R4** near the top of the script.
32. In `genStars` set `tileW = innerWidth * 2; tileH = innerHeight * 2;` at the top.
33. In `genStars` change `count` to `clamp(Math.floor((tileW*tileH)/2800), 200, 900)`.
34. In `genStars` set star `x: Math.random()*tileW` and `y: Math.random()*tileH`.
35. In `loop`, keep `dx/dy` mouse-parallax.
36. In the bg `for` loop, compute `const p = LAYER_P[s.layer];` and `const offX = view.ox*p - dx*(s.layer+1)*12;`.
37. Add `const offY = view.oy*p - dy*(s.layer+1)*12;`.
38. Compute wrapped base `let bx = ((s.x-offX)%tileW + tileW)%tileW;` and `let by = ((s.y-offY)%tileH + tileH)%tileH;`.
39. Replace the single `ctx.arc(...)` with a tiling double-loop drawing copies covering the viewport (`gx=bx-tileW … innerWidth+tileW` step `tileW`, same for `gy`).
40. Move the `ctx.globalAlpha`/`ctx.fillStyle` set to before the tiling loop (per star, once).
41. Verify: panning reveals an endless field; far layers drift slower; smooth at 1920×1080. `Verify: V1,V2,V3`.

## S-021 `feat(stars): place new thoughts in current view`
42. In `placeStar` (index.html:249) add `const M = 70;` margin.
43. Compute `const wx0 = view.ox + M/view.scale;` and `const wx1 = view.ox + (innerWidth - M)/view.scale;`.
44. Compute `const wy0 = view.oy + M/view.scale;` and `const wy1 = view.oy + (innerHeight - M - 80)/view.scale;`.
45. In the attempt loop, `const x = wx0 + Math.random()*(wx1-wx0);` and `const y = wy0 + Math.random()*(wy1-wy0);`.
46. Keep the existing `Math.hypot` 90-unit min-distance check (world units, unchanged).
47. Replace the fallback `return { x, y }` to sample the same rect.
48. Verify: pan far away, type a thought → it lands on-screen. `Verify: V1,V2,V5`.

## S-022 `feat(sky): scroll-to-zoom foreground`
49. Add `window` `wheel` listener with `{ passive:false }`; first line `e.preventDefault();`.
50. In `wheel`, capture `const wx = s2wx(e.clientX), wy = s2wy(e.clientY);`.
51. In `wheel`, set `view.scale = clamp(view.scale * Math.exp(-e.deltaY * 0.0015), 0.4, 3);` (reuse `clamp`, index.html:167).
52. In `wheel`, re-anchor: `view.ox = wx - e.clientX/view.scale; view.oy = wy - e.clientY/view.scale;`.
53. In `wheel`, call `applyView();`.
54. Add debounced save: `let zoomSaveT;` and in `wheel` `clearTimeout(zoomSaveT); zoomSaveT=setTimeout(saveState, 400);`.
55. In the tooltip append (index.html:436) add `tip.style.transformOrigin='left bottom'; tip.style.transform='translateY(-100%) scale('+(1/view.scale)+')';`.
56. Verify: wheel zooms toward the cursor; supernova still fires; tooltip readable. `Verify: V1,V2,V3`.

## S-023 `feat(sky): couple background to zoom`
57. Add `function bgScale(){ return 1 + (view.scale-1)*0.5; }` from **R4**.
58. In the bg `for` loop add `const bs = bgScale();` and `const TW = tileW*bs, TH = tileH*bs;`.
59. Change wrapped base to scaled space: `let bx = (((s.x-offX)*bs)%TW + TW)%TW;` and matching `by` with `TH`.
60. In the tiling loop use `gx+=TW`/`gy+=TH` and bounds `innerWidth+TW`/`innerHeight+TH`.
61. Scale the radius: `ctx.arc(gx, gy, s.r*bs, 0, Math.PI*2);`.
62. Verify: zoom feels cohesive front-to-back. `Verify: V1,V3,V5`.

## S-024 `feat(sky): double-click recenter glide`
63. Add `easeK` from **R3**.
64. Add `glideTo` from **R3**.
65. Add `function recenter(){ … }` skeleton with `let tx=0, ty=0;` and `const ts=1;`.
66. In `recenter`, when `STATE.stars.length`: centroid `cx = avg x, cy = avg y` (reduce over STATE.stars).
67. In `recenter`, set `tx = cx - (innerWidth/ts)/2; ty = cy - (innerHeight/ts)/2;` then `glideTo(tx,ty,ts);`.
68. Add `window` `dblclick` listener: `if(e.target.closest('.star, #input-wrap, .tooltip')) return; recenter();`.
69. Verify: pan/zoom away, double-click empty space → smooth glide framing all stars (origin if none). `Verify: V1,V2,V3`.

## S-025 `feat(persist): save camera + migrate`
70. In `saveState` (index.html:234) add `view: { ox:view.ox, oy:view.oy, scale:view.scale }` to the stringified object.
71. In `loadState` (index.html:238) after parsing, add `if(data.view){ … }` guard.
72. Inside that guard set `view.ox = +data.view.ox||0; view.oy = +data.view.oy||0;`.
73. Inside that guard set `view.scale = clamp(+data.view.scale||1, 0.4, 3);`.
74. In the init sequence (index.html:440-441) ensure order: `loadState(); renderStars(); applyView();`.
75. Verify migration: an old save with no `view` loads at identity, stars in place. `Verify: V1,V4,V5`.

## S-026 `polish(sky): zoom/pan polish + cross-browser`
76. Easing audit: glide uses `easeK` (cubic); only the twinkle loop uses linear; CSS transitions use `--ease-*` tokens.
77. Cursor states: `grab` idle, `grabbing` during pan; not stuck after `mouseup`.
78. Clamp consistency: scale clamp identical in `wheel`, `loadState`, recenter target.
79. Performance: `performance` trace while panning + zooming at 1920×1080; ~60fps, no long frames; tooltip readable at scale 0.4 and 3.
80. Cross-browser smoke at 1920×1080 and 1440×900 (Chrome; Safari if available); zero console errors; final `git diff` self-review. `Verify: V1,V3,V6`.

---

## Files touched
All edits in `index.html` (plus `docs/PROGRESS.md` log blocks and the two housekeeping commits). No new files, no dependencies.

## Reuse / leverage
- `clamp()` (index.html:167) — scale + load clamping.
- `renderStars()` (index.html:268) — unchanged; `.star left/top` become world coords once `#stars` is transformed.
- `triggerSupernova` / tooltip (index.html:398-438) — particles & tooltip ride the transformed container; only the tooltip counter-scale (step 55) is new.
- `saveState`/`loadState` (index.html:234-247) — extended with `view`.
- The S-017 title-reset early-return stays intact inside the refactored `renderConstellation`.

## Risks / notes
- **Background repetition**: tile = 2× viewport + three layer rates makes repeats hard to spot; procedural seeding is a future upgrade, out of scope.
- **Constellation across the void**: place-in-view can produce long nearest-neighbor lines after far pans — acceptable; flag in the S-021 commit.
- **Atomic interim**: after S-019 the background doesn't yet extend while panning (fixed in S-020); still runs clean — valid commit boundary.
- **Zoom crispness**: glows are gradients/box-shadows (not bitmaps) → smooth; confirm at scale extremes in step 79.

## End-to-end verification (chrome-devtools MCP @ 1920×1080, served over http://localhost)
Add 5+ stars → title appears → drag to pan (stars + lines move, bg endless) → type a thought (lands in view) → wheel zoom in/out (cursor-anchored, supernova impressive, tooltip readable) → double-click empty space (glides home) → reload (camera + stars restore exactly). Zero console errors throughout; `git diff` clean per commit.
