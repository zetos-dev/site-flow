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
- `.site/integrations/listmonk.json` when Listmonk is the selected provider
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
3. optional booking / calendar questions
4. optional multilingual questions

The orchestrator must provide:
1. Project name
2. Site type
3. Selected page list
4. Recommended design style and palette direction
5. Chosen tech stack
6. Content profile and seed mode
7. Image strategy and image profile
8. Optional email-signup decision, if any
   - mode: `design-only | planned | configured`
   - provider: `listmonk | other | undecided`
   - likely capture locations
   - form factor if known
   - integration status: `planned | configured`
   - when provider is `listmonk`, public integration details only:
     - `base_url`
     - `public_signup_url` or public subscription endpoint path
     - `list_name` or `list_id` if known
     - `opt_in`: `single | double` if known
     - whether the user wants to paste the values now or edit the generated config later
9. Optional booking / calendar decision, if any
   - mode: `design-only | planned | configured`
   - provider: `calendly | google-calendar | other | undecided`
   - likely capture locations
   - interaction type: `link-out | embed | popup`
   - form factor if known
   - public booking URL or embed URL only when available
   - integration status: `planned | configured`
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
- whether the page includes email signup
- whether the page includes booking/calendar
- when relevant, whether the page includes Listmonk signup and whether it is `planned` or `configured`
- when multilingual support is enabled, the default-language content path for that page

## Rules

- Keep full content generation inside the agent.
- Do not flood the parent conversation with generated content files.
- Preserve the non-technical product tone.
- If email signup is not enabled, omit email-service config entirely.
- If email signup is enabled, reflect it only in relevant blueprint/page-spec/content outputs.
- If booking/calendar is not enabled, omit calendar-service config entirely.
- If booking/calendar is enabled, reflect it only in relevant blueprint/page-spec/content outputs.
- If provider is `listmonk`, create `.site/integrations/listmonk.json` with public integration details only.
- Do not put provider secrets or low-level service credentials into `.site/config.json`, `.site/integrations/listmonk.json`, or `content/`.
- Generate editable signup copy only where relevant, such as heading, helper text, CTA label, consent note, and success/failure copy.
- Generate editable booking/calendar CTA copy only where relevant.
- When provider is `listmonk`, create page-local signup copy files only on relevant capture pages and shared locations.
- Distinguish clearly between `planned` and `configured`: use `configured` only when there is enough public signup information for page build to render a real provider-specific integration.
- When multilingual support is enabled, organize content as `content/{lang}/{page}/...`.
- Prefer using the default language folder even when only one language exists at initialization time.
- Do not create duplicated page templates per language.

## Completion Requirements

The generated artifacts must leave `/site-flow:site-build` able to build pages using page-local context only.

That means:
- every page spec must point to its own `content/` files
- relevant signup sections must be represented only on applicable pages
- relevant booking/calendar sections must be represented only on applicable pages
- when provider is `listmonk`, the integration artifact must exist and point to public signup details only
- when multilingual support is enabled, every page spec must point to the default language content path
- the orchestrator should be able to read only the target page’s content files later
