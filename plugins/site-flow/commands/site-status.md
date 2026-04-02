---
description: "Show current website project status, workflow stage, and content readiness overview"
argument-hint: "[page-name]"
allowed-tools: [Read, Glob]
---

# /site-flow:site-status — Website Project Status

Show the user a clear, friendly overview of their website project.

**Arguments**: $ARGUMENTS (optional page name for a detailed view)

## Steps

### 1. Load Project State

Read `.site/config.json`.
If missing, output:

```text
No website project found in this directory.
Run /site-flow:site-init to start a new website project.
```

Then exit.

Status must be computed from project files and `.site/` artifacts only.
Never require git history, resolved `HEAD`, branch detection, or worktree metadata to show project status.

Read these when present:
- `.site/workflow-state.json`
- `.site/site-blueprint.md`
- `.site/content-guide.md`
- `.site/validation-report.md`

Use `.site/config.json` as the source of truth for optional feature choices such as email support, booking support, and multilingual settings.

Use Glob to find:
- `.site/page-*-completion-report.md`
- page files in the project
- content folders under `content/`

### 2. Gather Status Signals

Determine:
- current workflow stage
- environment readiness
- missing tools if any
- total pages vs built pages
- whether bootstrap has completed
- whether residue warnings exist
- whether validation passed
- content readiness by page
- whether optional email support is disabled, design-only, planned/ready-for-integration with a provider, or already integrated
- whether optional booking support is disabled, design-only, planned/ready-for-integration with a provider, or already integrated
- whether multilingual support is disabled, reserved, active, or broken/incomplete
- default language and currently supported languages
- whether any translated page content appears missing or incomplete
- whether the site has a visible language selector/switcher when multilingual support is enabled
- whether enabled languages have actual page reachability, not only translated content folders
- whether `.site/integrations/booking.json` exists when booking support is enabled
- whether `.site/integrations/booking.json` exists when booking support is enabled
- whether Listmonk status appears planned, ready-for-integration, integrated, inconsistent, or broken
- whether booking status appears planned, ready-for-integration, integrated, inconsistent, or broken
- whether agent-first execution was used or fallback was used
- whether any main-session execution violations were recorded
- the latest fallback reason if present
- whether the site is visually finish-ready or still scaffold-like

When possible, summarize content states using:
- `real`
- `seeded-demo`
- `placeholder-minimal`

Also summarize visual readiness where possible:
- real user images
- stock-library images
- AI-generated images
- designed graphics
- placeholder-minimal debt
- design richness blockers

### 3. Display Overview

Use a compact dashboard like this:

```text
Website Project: {project_name}
Style: {design_style}
Tech: {friendly tech stack}
Stage: {workflow stage}
Environment: {ready / needs setup / not checked yet}
Missing tools: {none / Node.js / npm / npx}
Delegation: {agent-used / fallback-used / issue found}
Last fallback: {none / reason summary}
Email Support: {not planned / design-only / planned with Listmonk / ready-for-integration with Listmonk / integrated with Listmonk / planned with other service}
Listmonk Artifact: {n/a / present / missing}
Email Status: {n/a / planned / ready-for-integration / integrated / inconsistent / broken}
Booking Support: {not planned / design-only / planned / ready-for-integration / integrated}
Booking Artifact: {n/a / present / missing}
Booking Integration: {n/a / planned / ready-for-integration / integrated / inconsistent / broken}
Languages: {zh / zh, en / not enabled}
Default Language: {zh / n/a}
Translation Mode: {manual-assisted / n/a}
Translation Status: {single-language / multilingual-ready / multilingual-active / multilingual-broken}
Language Selector: {n/a / present / missing}
Language Reachability: {n/a / ready / partial / broken}

Progress: [{progress_bar}] {built_pages}/{total_pages} pages
Bootstrap: {complete / pending}
Residue Check: {passed / warning}
Validation: {passed / pending / needs fixes}
Finish Readiness: {ready / close / not ready}

| Page     | Status    | Content Mix                     | Visual Finish |
|----------|-----------|---------------------------------|---------------|
| Homepage | ✓ Built   | seeded-demo + stock/AI imagery  | strong        |
| About    | ✓ Built   | real + seeded-demo              | warning       |
| Services | ◻ Pending | not built yet                   | pending       |

Content Readiness:
- Real content: {summary}
- Demo-ready content: {summary}
- Minimal placeholders: {summary}

Image Readiness:
- User images: {summary}
- Stock images: {summary}
- AI-generated images: {summary}
- Designed graphics: {summary}
- Placeholder debt: {summary}

Design Richness:
- Motion & interaction polish: {summary}
- Scaffold-like sections: {summary}
- Main blockers: {summary}

Next Step:
- {most useful next command or install action}
- edit `.site/integrations/listmonk.json`, move it to `ready-for-integration` when public values are complete, then run `/site-flow:listmonk-integrate`
- edit `.site/integrations/booking.json`, move it to `ready-for-integration` when public values are complete, then run `/site-flow:booking-integrate`
- run `/site-flow:site-translate <language-code>` when multilingual support is enabled and the user wants to add one more language
```

### 4. Detailed Page View

If a page name is provided:

1. Read `.site/page-spec-{slug}.md`
2. Read `.site/page-{slug}-completion-report.md` if it exists
3. Inspect its content folder

Show:
- page status
- main sections
- content state by section
- what kind of imagery is being used
- whether real images are present
- whether the page includes email support and what mode/provider is planned if relevant
- whether the page includes Listmonk-backed email support, whether the integration artifact exists, and whether the page looks planned, ready-for-integration, inconsistent, or broken
- whether the page includes booking and what mode/provider is planned if relevant
- whether booking appears in the configured capture locations or looks missing
- which languages currently have content for this page
- whether any language version looks missing or incomplete
- whether the page is reachable through the visible language-switching path when multilingual support is enabled
- whether the page still has placeholder debt or scaffold-like sections
- what should be replaced next for best impact

Example:

```text
Page: Homepage
Status: Built
Built on: {date}

Sections:
- Hero — seeded-demo text, AI-generated hero visual, polished motion
- Trust Strip — placeholder-minimal logos, needs real brand marks
- Features — seeded-demo with stock imagery
- CTA — seeded-demo, finished visual treatment

Best Next Replacements:
- Real hero image or brand photography
- Real customer logos
- Final CTA wording
```

## Rules

- Use simple language.
- Focus on what the user should do next.
- Prefer actual file signals over stale status flags.
- Never treat missing git history, unresolved `HEAD`, or absent worktree metadata as a blocked project condition.
- If delegation metadata shows helper startup failed for git/worktree reasons, report fallback use plainly without implying the user must fix git first.
- Call out starter residue warnings clearly if any remain.
- Call out delegation violations clearly if any exist.
- Treat gradient-box visuals, empty frames, and wireframe-like sections as unfinished design debt.
- Make it obvious that demo-ready content is intentional and can be swapped later, while unfinished design output is not acceptable final quality.
- If Listmonk config and project state disagree, report the integration as inconsistent rather than silently trusting one file.
- If multilingual support is enabled but no visible selector/switcher or language reachability is found, report multilingual as broken/incomplete.
