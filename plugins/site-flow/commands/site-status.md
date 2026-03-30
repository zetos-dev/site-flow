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
- total pages vs built pages
- whether bootstrap has completed
- whether residue warnings exist
- whether validation passed
- content readiness by page

When possible, summarize content states using:
- `real`
- `seeded-demo`
- `placeholder-minimal`

Also summarize visual readiness where possible:
- real images
- designed visual archetypes
- minimal placeholders

### 3. Display Overview

Use a compact dashboard like this:

```text
Website Project: {project_name}
Style: {design_style}
Tech: {friendly tech stack}
Stage: {workflow stage}

Progress: [{progress_bar}] {built_pages}/{total_pages} pages
Bootstrap: {complete / pending}
Residue Check: {passed / warning}
Validation: {passed / pending / needs fixes}

| Page     | Status    | Content Mix                     |
|----------|-----------|---------------------------------|
| Homepage | ✓ Built   | seeded-demo + archetype visuals |
| About    | ✓ Built   | real + seeded-demo              |
| Services | ◻ Pending | not built yet                   |

Content Readiness:
- Real content: {summary}
- Demo-ready content: {summary}
- Minimal placeholders: {summary}

Next Step:
- {most useful next command}
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
- whether real images are present
- what should be replaced next for best impact

Example:

```text
Page: Homepage
Status: Built
Built on: {date}

Sections:
- Hero — seeded-demo text, brand-gradient visual
- Trust Strip — placeholder-minimal logos
- Features — seeded-demo
- CTA — seeded-demo

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
- Make it obvious that demo-ready content is intentional and can be swapped later.
