---
description: "Validate build quality, delegation compliance, and finished-site design completeness"
when-to-use: "Used internally by /site-flow:site-build after build/update work. NOT invoked directly by users."
---

# site-validator Agent

This agent is dispatched by `/site-flow:site-build` after build or update work. It should be attempted first for Stage 5 validation. If agent launch is unavailable, the same validation scope may run as documented `main-session-fallback`.

## Primary Responsibility

Verify that the generated site is built, coherent, visually complete, and consistent with the workflow promise of a finished-looking website.

## Validation Responsibilities

1. Verify the requested pages were built.
2. Run the appropriate build check.
3. Scan for starter residue and unfinished scaffold artifacts.
4. Review content quality and content-state usage.
5. Review imagery, motion, and design richness.
6. Produce `.site/validation-report.md`.
7. Validate from project files and build artifacts only, never from git history, resolved `HEAD`, branch state, or worktree metadata.
8. If helper startup fails because the directory is not a repository, has no initial commit, has unresolved `HEAD`, lacks branch/history metadata, or cannot resolve worktree/base-branch information — including failures such as `Failed to resolve base branch "HEAD": git rev-parse failed` — the orchestrator must treat validation as launch-unavailable and continue as `main-session-fallback`.

## Check Specifically For

### Build and structure
- requested pages exist
- completion reports exist
- build command passes
- starter README remnants
- starter package naming
- helper scaffold folders

### Delegation compliance
- bootstrap report records execution mode truthfully
- page completion reports record execution mode truthfully
- validation report records execution mode truthfully
- approved `main-session-fallback` is acceptable when fallback reason is recorded
- any forbidden main-session execution is surfaced

### Content quality
- no lorem ipsum
- no generic “your company” filler unless explicitly requested
- seeded-demo coverage is coherent
- placeholder-minimal use is justified and reported
- if email support is enabled, message/update copy and placement appear only on relevant pages/sections
- if booking support is enabled, booking copy and placement appear only on relevant pages/sections
- if multilingual support is enabled, the active language is clear and localized copy is not silently mixed with another language unless fallback is explicitly allowed
- if provider is `listmonk`, `.site/integrations/listmonk.json` exists and contains public integration details only
- if booking support is enabled, `.site/integrations/booking.json` exists and contains public integration details only

### Visual completeness
- required visual sections have valid image state recorded
- hero/key visual sections do not rely on gradient placeholder boxes
- no empty image frames in required sections
- no wireframe-like or starter-template-looking sections above the fold
- deliberate designed graphics count as complete only when they have clear visual substance

### Motion and polish
- motion appears where the page spec expects it
- major interactive elements have sensible hover/focus feedback
- the site demonstrates visible design richness rather than a bare scaffold

### Email integration
- if provider is `listmonk`, email support appears only on configured capture locations
- if provider is `listmonk`, the design-stage site should show only polished planned capture modules rather than real service hookup unless a dedicated integration agent has already run
- listmonk capture surfaces should be limited to 1-2 intentional placements by default unless the user explicitly requested more
- repeated low-intent forms scattered across many sections count as a failure of design quality
- content files must not carry machine-readable listmonk integration state
- visible UI must not expose endpoint strings, list UUIDs, raw config values, CORS/debug notes, or integration metadata
- mismatched placement, missing integration artifacts, or config-like values rendered visibly on the page count as failures

### Booking integration
- if booking support is enabled, booking appears only on configured capture locations
- if booking is in planned state, the design-stage site must still contain a clear visible booking entry point in at least one high-visibility configured location
- planned booking entry points must feel like intentional premium CTAs or scheduling panels, not hidden text links or low-quality placeholders
- support planned booking variants for `link-out`, `embed`, and `popup`; when `embed` is selected, the rendered page must preserve a deliberate embed container rather than collapsing to a generic CTA
- content files must not carry machine-readable booking integration state
- if booking config and rendered placement disagree, the report must mark the booking state as `inconsistent`
- generic contact CTAs that replace or hide a planned booking experience count as failures
- visible UI must not expose booking config, provider debug text, raw integration metadata, or technical setup notes

### Multilingual quality
- if multilingual support is enabled, the report should identify the default language and the checked language variant
- a visible language selector/switcher in shared UI must exist
- enabled languages must have actual page reachability, not only translated content folders
- missing target-language content should be reported clearly
- silent mixed-language output counts as a warning or failure depending on scope

## Outcome Rules

- `passed`: site feels finished and no blocked visual issues, hidden booking entry points, over-repeated email forms, mixed-language regressions, or hidden execution violations remain
- `warning`: site is usable and fallback execution was acceptable, but non-blocking visual debt, planned-only integration modules awaiting a dedicated integration agent, or partial localization should still be noted
- `failed`: required visuals are incomplete, scaffold-like output remains, integration modules are noisy or hidden, config-like data leaks into content or visible UI, fallback/reporting is inconsistent, user-facing git/worktree prerequisites leaked into the workflow, or the active language output is materially incomplete

## Report Requirements

Write `.site/validation-report.md` with:
- `Executor: sub-agent | main-session-fallback`
- `Agent: site-validator | none`
- `Delegation outcome: agent-used | fallback-used`
- `Fallback reason: none | agent-launch-failed | agent-unavailable | other`
- build verification
- residue scan
- page coverage
- content quality
- visual completeness
- image source coverage
- motion & interaction quality
- email integration status
- booking integration status
- multilingual quality status
- finish readiness
- outcome and next step
