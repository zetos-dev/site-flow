---
description: "Complete real Listmonk integration for an already-designed site-flow website"
argument-hint: ""
allowed-tools: [Read, Write, Edit, Glob, Grep, Agent, AskUserQuestion]
---

# /site-flow:listmonk-integrate — Complete Listmonk Hookup

Use this only after the site has already been designed and built.

This workflow performs the real Listmonk integration stage. It does **not** redesign the site.

## Steps

1. Read `.site/config.json`. If missing, tell the user to run `/site-flow:site-init` first.
2. Read `.site/integrations/listmonk.json`. If missing, explain that Listmonk has not been prepared for this site yet.
3. Read `.site/workflow-state.json` when present.
4. Confirm that the site has already been built.
5. Inspect the configured capture locations and relevant page files.
6. Confirm the integration lifecycle state is `ready-for-integration` or ask the user for missing public values.
7. Launch `plugins/site-flow/agents/listmonk-integrator.md` to complete the real hookup.
8. Summarize what changed, what surfaces were integrated, and what to preview next.

## Rules

- Treat this as the dedicated integration stage.
- Do not redesign unrelated sections.
- Preserve the visual quality established by site design/build.
- Keep Listmonk placement limited to the intended reserved surfaces.
- If required public values are missing, stop and explain exactly what is needed.
- Do not expose raw config values or JSON on the page.
- After integration, tell the user to preview the affected pages.

## User Communication

Use simple language.

If integration succeeds, tell the user:
- which pages or modules are now connected
- that the site design was preserved
- to run `/site-flow:site-preview` to check the result visually
