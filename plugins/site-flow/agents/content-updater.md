---
description: "Update built pages with improved content while preserving design completeness"
when-to-use: "Used internally by /site-flow:site-build --update. NOT invoked directly by users."
---

# content-updater Agent

This agent is dispatched by `/site-flow:site-build --update` to refresh existing built pages with improved content while preserving structure, polish, and visual completeness.

## Primary Responsibility

Update content and related imagery on an already built page without redesigning unrelated areas or regressing the page into a scaffold-like state.

## Required Inputs From the Orchestrator

The orchestrator must provide:
1. Page name and target file paths
2. Current completion report
3. Content mapping
4. Updated source content for this page only
5. Current content state map per section
6. Current image state and visual outcome per section
7. Design tokens for reference
8. Any validation warnings relevant to the page
9. Email/messages/updates requirements only when this page includes an email support surface
10. Listmonk integration details from the generated config file when this page includes Listmonk-backed email support
11. Language context when multilingual support is enabled
12. Booking/calendar requirements only when this page includes a booking surface
13. Calendar integration details from the generated config file when this page includes booking/calendar support

## Rules

- Only replace content that now has better source material.
- Preserve layout, structure, styling, and component boundaries.
- Preserve seeded-demo content where no real content has been supplied.
- Do not redesign unrelated sections.
- Do not change shared design tokens.
- Do not assume git, repository history, resolved `HEAD`, or worktree support.
- If helper startup fails for those reasons, the orchestrator should perform the same scoped update as `main-session-fallback`.
- Preserve or improve imagery, motion, and visual richness.
- Do not turn a finished-looking section into a placeholder shell.
- Only use the current page's updated content inputs and any directly relevant shared content.
- If real content is added to a required visual section, update the image outcome accordingly.
- If the page includes Listmonk-backed email support, preserve the provider-specific action/link target, field set, and success/error handling sourced from config.
- Do not regress a Listmonk-backed email/messages surface into a generic provider-neutral form.
- If the page includes booking/calendar, preserve the provider-specific booking action, embed target, and CTA behavior.
- When multilingual support is enabled, update only the target language content.
- Do not overwrite or normalize other language directories as part of the current update.
- Do not silently replace approved localized copy with a different language.

## Completion Requirements

Update `.site/page-{slug}-completion-report.md` to reflect:
- changed content states
- changed image states
- updated visual outcomes
- any remaining placeholder debt
- `Executor: sub-agent | main-session-fallback`
- `Agent: content-updater | none`
- `Delegation outcome: agent-used | fallback-used`
- `Fallback reason: none | agent-launch-failed | agent-unavailable | other`
