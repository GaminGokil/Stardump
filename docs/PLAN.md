# Stardump v1 — Implementation Plan (subagent-ready)

## Context

Greenfield Hack Club "Stardance" submission. Repo currently contains only `AGENTS.md`, `CLAUDE.md` (one-line `@AGENTS.md` include), and `.git/`. No code yet.

Goal: ship a single-file static web app (`index.html`) per the PRD — worry input → twinkling-star → constellation → supernova, persisted to `localStorage`, deployed to GitHub Pages. ~16 active hours, solo, build weekend.

**Delegation model:** orchestrator (Claude Sonnet 4.6) hands each `S-xxx` step to a subagent. Each step below is written as a self-contained brief: the subagent gets the brief verbatim (plus the current `index.html`) and produces the diff for that step only. Subagents do not see the PRD or this plan's other steps — every constant, selector, regex, and easing they need is inlined into their brief.

**Hard constraints (binding — repeat in every subagent brief):**
- Single file: `index.html` with HTML, CSS, JS inline. **No npm, no build, no CDN, no frameworks, no `<script src=…>` for remote URLs, no `<link rel=stylesheet href=…>` for remote URLs.** Vanilla ES2020+.
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
