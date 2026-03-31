---
description: "Show current website project status, workflow stage, and content readiness overview"
argument-hint: "[page-name]"
allowed-tools: [Read, Glob]
---

# /site-status — Website Project Status

Show the user a clear, friendly overview of their website project.

**Arguments**: $ARGUMENTS (optional page name for a detailed view)

## Steps

### 1. Load Project State

Read `.site/config.json`.
If missing, output:

```text
No website project found in this directory.
Run /site-init to start a new website project.
```

Then exit.

Read these when present:
- `.site/workflow-state.json`
- `.site/site-blueprint.md`
- `.site/content-guide.md`
- `.site/validation-report.md`

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
- Call out starter residue warnings clearly if any remain.
- Call out delegation violations clearly if any exist.
- Treat gradient-box visuals, empty frames, and wireframe-like sections as unfinished design debt.
- Make it obvious that demo-ready content is intentional and can be swapped later, while unfinished design output is not acceptable final quality.
