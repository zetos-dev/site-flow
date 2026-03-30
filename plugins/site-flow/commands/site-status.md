---
description: "Show current website project status and progress overview"
argument-hint: "[page-name]"
allowed-tools: [Read, Glob]
---

# /site-status — Website Project Status

Show the user a clear overview of their website project progress.

**Arguments**: $ARGUMENTS (optional: page name for detailed view)

## Steps

### 1. Load Project State

Read `.site/config.json` — if missing, output:
```
No website project found in this directory.
Run /site-init to start a new website project.
```
And exit.

### 2. Gather Status Information

1. Read `.site/site-blueprint.md` for page list
2. Use Glob to find all `.site/page-*-completion-report.md` files
3. Check `content/` directory for user-provided content vs placeholder
4. Determine overall progress

### 3. Display Overview

Output a friendly status dashboard:

```
Website Project: {project_name}
Style: {design_style}
Tech: {tech_stack_friendly_name}

Progress: [{progress_bar}] {completed}/{total} pages

| Page         | Status      | Content    |
|--------------|-------------|------------|
| Homepage     | ✓ Built     | Placeholder |
| About        | ✓ Built     | Real ✓     |
| Services     | ◻ Pending   | —          |
| Contact      | ◻ Pending   | —          |

Content Readiness:
  ✓ content/01-homepage/ — 2 of 5 files updated
  ◻ content/02-about/ — not started
  ◻ content/03-services/ — not started
  ✓ content/04-contact/ — 1 of 2 files updated

Commands:
  /site-build          Build the next pending page (Services)
  /site-build --all    Build all remaining pages
  /site-build --update Update built pages with your latest content
  /site-preview        Preview your website in the browser
```

### 4. Detailed Page View (if argument provided)

If user provides a page name (e.g., `/site-status homepage`):

1. Read `.site/page-spec-{slug}.md` for the page specification
2. Read `.site/page-{slug}-completion-report.md` if it exists
3. Check content files for this page

Output:

```
Page: {page_name}
Status: {Built / Pending}
{if built}Built on: {date}{endif}

Sections:
  ✓ Hero Banner — {real content / placeholder}
  ✓ Features Grid — {real content / placeholder}
  ✓ Testimonials — {real content / placeholder}
  ✓ Call to Action — {real content / placeholder}

Content Files (content/{NN}-{name}/):
  ✓ hero-title.md — Updated with real content
  ◻ hero-description.md — Still using placeholder
  ◻ highlight-1.md — Still using placeholder
  ✓ images/hero-background.jpg — Provided

{if completion report exists}
Last Build Notes:
  {summary from completion report}
{endif}
```

## Rules

- Use simple, non-technical language
- Show actionable next steps
- Use visual indicators (✓, ◻, progress bars) for quick scanning
- Check actual file existence, don't rely solely on config.json status
