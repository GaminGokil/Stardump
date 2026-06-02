# Stardump build journal

_This file will be filled in as the build progresses. Sections below are stubs — updated during S-014._

## What it is

<!-- 3-sentence summary written at S-014 -->

## What went well

<!-- 3–5 bullets from PROGRESS.md at S-014 -->

## What broke and how I fixed it

<!-- 3–5 bullets from PROGRESS.md at S-014 -->

## AI assist disclosure

This project was built with AI assistance:

- **Orchestrator:** Claude Opus 4.7 (Anthropic) — designed the implementation plan (`docs/PLAN.md`), reviewed each diff before committing, and managed the subagent build pipeline.
- **Code subagents:** Claude Sonnet 4.6 — wrote each micro-step's code from self-contained briefs (see `docs/PLAN.md` for step definitions). Each agent saw only its brief + the current state of `index.html`.
- **Human role:** reviewed every commit, approved the plan, and owns the final submission.

Tools used: Claude Code CLI, chrome-devtools MCP (visual verification).

## What I'd add in v1.1

- NASA APOD daily background, blurred + dimmed
- Web Audio chime per star (Tone.js, pentatonic scale)
- "Capture sky" → PNG export via `html-to-image`
- Star age decay (older stars dim to 40%)
- Keyboard shortcuts (Esc closes tooltip, Cmd+Enter saves)
- Mobile gesture support
- Multiple named skies / themes
