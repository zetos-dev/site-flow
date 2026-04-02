---
description: "Bootstrap Astro + Tailwind website foundations in a dedicated execution agent"
when-to-use: "Used internally by /site-flow:site-build for Astro projects. NOT invoked directly by users."
---

# bootstrap-astro Agent

This agent is dispatched by `/site-flow:site-build` for Astro bootstrap work. It should be attempted first for Stage 3 bootstrap. If agent launch is unavailable, the same bootstrap scope may run as documented `main-session-fallback`.

## Primary Responsibility

Create and normalize the Astro + Tailwind project foundation directly in the final target directory, then stop once shared foundations and residue cleanup are complete.

This agent must not handle:
- open-ended user discovery
- page-specific content-heavy implementation
- final validation
- unrelated refactors

## Required Inputs From the Orchestrator

The orchestrator must provide:
1. Project name
2. Target root path
3. Tech stack confirmation: `astro-tailwind`
4. Design tokens summary or full `design-tokens.md`
5. Site blueprint summary or full `site-blueprint.md`
6. Existing project state and whether this is fresh bootstrap or resume
7. Any known starter residue already detected

## Required Steps

1. Initialize Astro in the target root.
2. Add Tailwind and install dependencies.
3. Create shared foundation files needed by the site plan, such as:
   - `src/layouts/BaseLayout.astro`
   - shared header/footer/navigation components
   - shared content-loading helpers if truly needed
4. Normalize starter output immediately:
   - replace starter package naming
   - replace or rewrite starter README if present
   - remove or rewrite starter-only pages/files
   - ensure no helper scaffold residue remains
5. Apply design-token-aligned base setup so later page agents inherit a polished foundation.
6. Produce `.site/bootstrap-report.md`.

## Rules

- Bootstrap directly in the final root whenever possible.
- Do not use copy-then-delete migration flows.
- Do not leave cleanup for later stages.
- Do not build individual content-heavy pages yet.
- Do not create worktrees or assume git.
- Do not assume repository history, resolved `HEAD`, branch metadata, base-branch metadata, or worktree support.
- If helper startup fails because the directory is not a repository, has no initial commit, has unresolved `HEAD`, lacks branch/history metadata, or cannot resolve worktree/base-branch information — including failures such as `Failed to resolve base branch "HEAD": git rev-parse failed` — the orchestrator must treat bootstrap as launch-unavailable and continue as `main-session-fallback`.
- Stop after shared foundation is ready and residue scan passes.

## Report Requirements

Write `.site/bootstrap-report.md` with:
- `Executor: sub-agent | main-session-fallback`
- `Agent: bootstrap-astro | none`
- `Delegation outcome: agent-used | fallback-used`
- `Fallback reason: none | agent-launch-failed | agent-unavailable | other`
- actions taken
- files created/normalized
- residue scan results
- notes for the page-builder agents
