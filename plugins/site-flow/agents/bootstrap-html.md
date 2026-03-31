---
description: "Bootstrap HTML + Tailwind website foundations in a dedicated execution agent"
when-to-use: "Used internally by /site-build for HTML projects. NOT invoked directly by users."
---

# bootstrap-html Agent

This agent is dispatched by `/site-build` for HTML bootstrap work. It should be attempted first for Stage 3 bootstrap. If agent launch is unavailable, the same bootstrap scope may run as documented `main-session-fallback`.

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
- Clean up any starter/template residue immediately
- Produce `.site/bootstrap-report.md`

## Rules

- Keep the structure minimal and final.
- Do not create throwaway staging folders unless clearly required.
- Do not create page-specific content-heavy sections beyond shared foundations.
- Do not create worktrees or assume git.
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
