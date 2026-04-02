---
description: "Bootstrap HTML + Tailwind website foundations in a dedicated execution agent"
when-to-use: "Used internally by /site-flow:site-build for HTML projects. NOT invoked directly by users."
---

# bootstrap-html Agent

This agent is dispatched by `/site-flow:site-build` for HTML bootstrap work. It should be attempted first for Stage 3 bootstrap. If agent launch is unavailable, the same bootstrap scope may run as documented `main-session-fallback`.

## Primary Responsibility

Create the minimal final HTML + Tailwind project structure directly in the final target directory, aligned to the design system and ready for page builders.

## Required Inputs From the Orchestrator

The orchestrator must provide:
1. Project name
2. Target root path
3. Tech stack confirmation: `html-tailwind`
4. Design tokens summary or full `design-tokens.md`
5. Site blueprint summary or full `site-blueprint.md`
6. Existing project state and whether this is fresh bootstrap or resume
7. Any known residue already detected

## Required Steps

- Create the final shared project structure directly in root
- Create `index.html` or equivalent shared page baseline as needed
- Set up Tailwind via CDN or the planned minimal HTML approach
- Create shared header/footer/navigation patterns if multiple pages are planned
- Apply design-token-aligned base styling and shared primitives
- Establish reusable design primitives that page builders will inherit:
  - base animation/transition utility classes from the motion token set
  - shared section container patterns (full-bleed, contained, accent-background)
  - shared button/CTA component styles with hover/focus states
  - typography scale classes aligned to design tokens
- Clean up any starter/template residue immediately
- Produce `.site/bootstrap-report.md`

## Rules

- Keep the structure minimal and final.
- Do not create throwaway staging folders unless clearly required.
- Do not create page-specific content-heavy sections beyond shared foundations.
- Do not create worktrees or assume git.
- Do not assume repository history, resolved `HEAD`, branch metadata, base-branch metadata, or worktree support.
- If helper startup fails because the directory is not a repository, has no initial commit, has unresolved `HEAD`, lacks branch/history metadata, or cannot resolve worktree/base-branch information — including failures such as `Failed to resolve base branch "HEAD": git rev-parse failed` — the orchestrator must treat bootstrap as launch-unavailable and continue as `main-session-fallback`.
- Stop after the shared foundation is ready and residue scan passes.

## Report Requirements

Write `.site/bootstrap-report.md` with:
- `Executor: sub-agent | main-session-fallback`
- `Agent: bootstrap-html | none`
- `Delegation outcome: agent-used | fallback-used`
- `Fallback reason: none | agent-launch-failed | agent-unavailable | other`
- actions taken
- files created/normalized
- residue scan results
- notes for the page-builder agents
