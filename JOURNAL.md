# Stardump build journal

## What it is

Stardump is a single-file static web app where teenagers type worries into an input bar and each becomes a twinkling star in a dark night sky. When five or more stars exist, faint lines connect them into a procedurally-named constellation. Clicking a star opens a tooltip; clicking "mark as resolved" fires a supernova animation before the star disappears. State persists across sessions via localStorage — no accounts, no backend, no dependencies.

## What went well

- The supernova animation (`nova-core` + `nova-glow` + 16 `nova-particle` radial sprays) came together cleanly in two steps and hits the "epic moment" bar on first try.
- The procedural constellation naming system — 8 keyword-regex tags, 11 named combinations, 9-name deterministic fallback — produces surprisingly resonant names from real teen worries.
- The star placement algorithm (60-attempt min-distance constraint with unconstrained fallback) works silently; no collisions even at 30+ stars.
- Keeping the entire app in a single `index.html` with CSS custom properties for the design tokens made iterating on colors and easings trivial.
- The subagent pipeline (5 sequential Sonnet 4.6 agents, each with a self-contained brief) produced zero merge conflicts and required no orchestrator intervention mid-step.

## What broke and how I fixed it

- The `animatedIds` Set had to be moved before `loadState()` to avoid a reference-before-initialization error on page load with persisted stars — caught in S-011.
- The `#constellation-name` div needed `opacity: 0` as an initial CSS state to prevent a flash on load before the JS sets it — fixed in S-012.
- The constellation edge key used string coordinates, which meant floating-point star positions could fail to deduplicate edges between identical star pairs — the sorted `[idA, idB]` id-based key resolves this correctly.

## AI assist disclosure

This project was built with AI assistance across the full implementation:

- **Orchestrator:** Claude Sonnet 4.6 (Anthropic) — designed the implementation plan (`docs/PLAN.md`), sliced work into micro-steps with self-contained subagent briefs, and read each agent's output before passing state to the next agent.
- **Code subagents (×5):** Claude Sonnet 4.6 — each agent received the current `index.html` content and a precise brief for its batch of steps (S-000–S-001c, S-002–S-004, S-005a–S-006b, S-007a–S-010, S-011–S-014). Agents wrote all the HTML, CSS, and JavaScript.
- **Human role:** Provided the PRD, approved the plan, and owns the submission. Reviewed commit log after each agent completed.

Tools: Claude Code CLI (orchestrator shell), chrome-devtools MCP available for visual verification.

## What I'd add in v1.1

- NASA APOD daily background, blurred + dimmed behind the star field
- Web Audio API chime per star creation (pentatonic scale, no library)
- "Capture sky" → PNG export via Canvas `toDataURL()`
- Star age decay: stars older than 48h dim to 40% opacity
- Keyboard shortcuts: Esc closes tooltip, Cmd+Enter submits
- Mobile gesture support: tap-and-hold to open tooltip
- Multiple named skies / sky themes
