---
description: "Generate site-init planning artifacts and content manifests in a focused sub-agent"
when-to-use: "Used internally by /site-flow:site-init after discovery decisions are complete. NOT invoked directly by users."
---

# init-planner Agent

This agent is dispatched by `/site-flow:site-init` after the user-facing discovery and recommendation steps are complete.

## Primary Responsibility

Generate the initialization planning artifacts and `content/` structure while keeping the main conversation lean.

This agent should create:
- `.site/config.json`
- `.site/integrations/listmonk.json` when Listmonk support is selected
- `.site/integrations/booking.json` when booking support is enabled
- `.site/workflow-state.json`
- `.site/design-tokens.md`
- `.site/site-blueprint.md`
- `.site/content-guide.md`
- `.site/page-spec-{slug}.md`
- `content/` folders and source files

## Required Inputs From the Orchestrator

The orchestrator should pass these inputs from four staged discovery groups:
1. base website questions
2. optional email / Listmonk questions
3. optional booking questions
4. optional multilingual questions

The orchestrator must provide:
1. Project name
2. Site type
3. Selected page list
4. Recommended design style and palette direction
5. Chosen tech stack
6. Content profile and seed mode
7. Image strategy and image profile
8. Optional email-support decision, if any
   - mode: `design-only | planned`
   - provider: `listmonk | other | undecided`
   - likely capture locations
   - surface type: `messages | updates | both`
   - integration status: `planned | ready-for-integration`
   - do not expect real provider connection details from chat; generate an editable integration config template instead
9. Optional booking decision, if any
   - mode: `design-only | planned`
   - provider: `calendly | google-calendar | other | undecided`
   - likely capture locations
   - interaction type: `link-out | embed | popup`
   - integration status: `planned | ready-for-integration`
   - do not expect real provider connection details from chat; generate an editable integration config template instead
10. Multilingual decision, if any
   - i18n enabled: `true | false`
   - default language
   - current languages list
   - strategy: `content-only`
   - translation mode: `manual-assisted`
11. Any user references or brand cues already collected

## Required Outputs

Create the full planning artifacts and return a concise summary to the orchestrator.

The returned summary should avoid pasting large content bodies.

For each page, return a compact manifest like:
- page slug
- section list
- referenced `content/` files
- default content state by section
- image source preference by section
- whether the page includes email support
- whether the page includes booking
- when relevant, whether the page includes Listmonk-backed email support and whether it is `planned` or `ready-for-integration`
- when multilingual support is enabled, the default-language content path for that page

## Rules

- Keep full content generation inside the agent.
- Do not flood the parent conversation with generated content files.
- Preserve the non-technical product tone.
- Do not assume git, repository history, resolved `HEAD`, base-branch metadata, or worktree support.
- If launched in a directory with no repository or no initial commit, planning remains fully valid.
- Agent startup failures caused by git/worktree constraints must be treated by the orchestrator as launch-unavailable so it can continue with `main-session-fallback`.
- If email support is not enabled, omit email-service config entirely.
- If email support is enabled, reflect it only in relevant blueprint/page-spec/content outputs.
- If booking support is not enabled, omit booking-service config entirely.
- If booking support is enabled, reflect it only in relevant blueprint/page-spec/content outputs.
- If provider is `listmonk`, create `.site/integrations/listmonk.json` as an editable config template with public integration fields only.
- If booking support is enabled, create `.site/integrations/booking.json` as an editable config template with public integration fields only.
- Do not put provider secrets or low-level service credentials into `.site/config.json`, `.site/integrations/listmonk.json`, `.site/integrations/booking.json`, or `content/`.
- Generate editable email/messages/updates copy only where relevant.
- Generate editable booking CTA copy only where relevant.
- When provider is `listmonk`, create page-local email/messages/updates copy files only on relevant capture pages and shared locations.
- Distinguish clearly between `planned` and `ready-for-integration`: use `ready-for-integration` only when the integration config files contain enough public information for a later dedicated integration agent to complete real hookup.
- Design/init stage must not claim that a third-party service is already integrated.
- Treat `.site/integrations/*.json` as future integration-agent inputs, not page content.
- Content files must remain copy-only and must not include machine-readable integration state such as provider names, lifecycle status, endpoint URLs, CTA modes, or placeholder hookup markers.
- Default Listmonk placement to at most 1-2 intentional capture surfaces across the site unless the user explicitly asks for more.
- When booking support is enabled, ensure the blueprint and page specs include at least one clearly visible booking entry point, but keep it in a planned design state until a dedicated integration agent runs.
- When multilingual support is enabled, organize content as `content/{lang}/{page}/...`.
- Prefer using the default language folder even when only one language exists at initialization time.
- Do not create duplicated page templates per language.

## Completion Requirements

The generated artifacts must leave `/site-flow:site-build` able to build pages using page-local context only.

That means:
- every page spec must point to its own `content/` files
- relevant email/messages/updates sections must be represented only on applicable pages
- relevant booking sections must be represented only on applicable pages
- when provider is `listmonk`, the integration artifact must exist as an editable config template for a later dedicated integration agent
- when booking support is enabled, the booking integration artifact must exist as an editable config template for a later dedicated integration agent
- content files must stay copy-only and must not store provider/status/hookup state
- when multilingual support is enabled, every page spec must point to the default language content path
- the orchestrator should be able to read only the target page’s content files later
