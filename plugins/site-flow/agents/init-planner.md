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
- `.site/workflow-state.json`
- `.site/design-tokens.md`
- `.site/site-blueprint.md`
- `.site/content-guide.md`
- `.site/page-spec-{slug}.md`
- `content/` folders and source files

## Required Inputs From the Orchestrator

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
9. Any user references or brand cues already collected

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

## Rules

- Keep full content generation inside the agent.
- Do not flood the parent conversation with generated content files.
- Preserve the non-technical product tone.
- If email signup is not enabled, omit email-service config entirely.
- If email signup is enabled, reflect it only in relevant blueprint/page-spec/content outputs.
- Do not put provider secrets or low-level service credentials into `content/`.
- Generate editable signup copy only where relevant, such as heading, helper text, CTA label, consent note, and success/failure copy.

## Completion Requirements

The generated artifacts must leave `/site-flow:site-build` able to build pages using page-local context only.

That means:
- every page spec must point to its own `content/` files
- relevant signup sections must be represented only on applicable pages
- the orchestrator should be able to read only the target page’s content files later
