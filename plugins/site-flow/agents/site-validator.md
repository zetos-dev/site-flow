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
- if booking/calendar is enabled, booking copy and placement appear only on relevant pages/sections
- if multilingual support is enabled, the active language is clear and localized copy is not silently mixed with another language unless fallback is explicitly allowed
- if provider is `listmonk`, `.site/integrations/listmonk.json` exists and contains public integration details only
- if booking/calendar support is enabled, `.site/integrations/calendar.json` exists and contains public integration details only

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
- if provider is `listmonk` and status is `configured`, built pages contain provider-specific Listmonk public endpoint or action/link targets and matching copy
- if provider is `listmonk` and status is `planned`, the report must warn that provider wiring is incomplete rather than passing silently
- generic provider-neutral email/message markup on a Listmonk-backed page counts as a failure when enough provider-specific config data exists
- mismatched public targets or missing integration artifacts count as failures

### Booking/calendar integration
- if booking/calendar is enabled, booking appears only on configured capture locations
- if booking/calendar status is `configured`, built pages contain the provider-specific public booking target or embed target where expected
- if booking/calendar status is `planned`, the report must warn that provider wiring is incomplete rather than passing silently
- generic contact CTAs that replace a configured booking experience count as failures

### Multilingual quality
- if multilingual support is enabled, the report should identify the default language and the checked language variant
- missing target-language content should be reported clearly
- silent mixed-language output counts as a warning or failure depending on scope

## Outcome Rules

- `passed`: site feels finished and no blocked visual issues, broken provider wiring, broken booking wiring, mixed-language regressions, or hidden execution violations remain
- `warning`: site is usable and fallback execution was acceptable, but non-blocking visual debt, planned-only email wiring, planned-only booking wiring, or partial localization should still be noted
- `failed`: required visuals are incomplete, scaffold-like output remains, provider-specific email or booking wiring is broken or missing where expected, fallback/reporting is inconsistent, user-facing git/worktree prerequisites leaked into the workflow, or the active language output is materially incomplete

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
- booking/calendar integration status
- multilingual quality status
- finish readiness
- outcome and next step
