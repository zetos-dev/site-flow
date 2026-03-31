---
description: "Preview your website in the browser and make focused adjustments based on feedback"
argument-hint: ""
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion]
---

# /site-preview — Preview & Iterate

You are helping a non-technical user preview their website and make adjustments based on feedback.

## Pre-check

1. Read `.site/config.json` — if missing, tell the user to run `/site-init` first.
2. Read `.site/workflow-state.json` if present.
3. Check that at least one page has been built.
4. If no pages are built, tell the user to run `/site-build` first.
5. Check environment readiness before starting any preview server.

If the chosen stack needs Node.js/npm and they are missing:
- explain this in plain language
- do not attempt preview commands yet
- tell the user what to install
- tell them to rerun `/site-preview` or `/site-build` after installation

## Phase 1 — Start Preview Server

### For Astro projects
Only run a preview server after confirming Node.js/npm readiness.

If environment readiness is blocked:
- explain that Astro preview needs Node.js and npm
- do not fall back to Python as if it were equivalent
- tell the user to install the missing tools and rerun the command

If ready, run in the background:

```bash
npx astro dev
```

### For HTML projects
If Node-based preview tools are available, run in the background:

```bash
npx serve .
```

If Node-based tools are unavailable but the site is plain static HTML, you may use this explicit fallback:

```bash
python3 -m http.server 8000
```

Tell the user this is a simple local preview fallback for static HTML.

Then tell the user where to open the site.

## Phase 2 — Frame the Preview Clearly

Before asking for feedback, explain the current content state in plain language.

If the site is using seeded demo content, say so clearly:
- some text is polished demo content
- some images may come from default stock-library selections
- some hero or key visuals may be AI-generated
- some visuals may still be designed placeholders
- this is intentional so the user can judge the design before gathering every final asset

If `.site/validation-report.md` exists, summarize any active warnings.

## Phase 3 — Collect Structured Feedback

Use `AskUserQuestion`.

### Question 1 — Overall Impression
- Looks great, only small tweaks
- Good direction, some sections need work
- Not quite right, needs bigger changes
- Love it, ready to move on

If the user loves it, skip to finalization.

### Question 2 — What Kind of Changes?
Use multi-select:
- Colors or visual style
- Layout or section structure
- Text tone or wording
- Images or visual placeholders
- Motion or interaction feel

Then ask for short free-text clarification if needed.

## Phase 4 — Apply Changes

For small changes:
- edit directly

For larger changes:
- dispatch a focused sub-agent

### Large-change sub-agent rules
Provide:
- current page file(s)
- current design tokens
- user feedback
- whether the affected content is real, seeded-demo, or placeholder-minimal

Rules:
- preserve unaffected content and structure
- do not redesign unrelated sections
- keep responsive behavior intact
- keep within approved motion/design system unless the user explicitly wants a broader restyle

## Phase 5 — Feedback Loop

Keep iteration quick:
1. apply change
2. tell the user to refresh
3. ask if the result is closer

When discussing content, distinguish clearly between:
- changing the design direction
- refining seeded demo content
- replacing with real user content/assets

## Phase 6 — Finalize

When the user is happy:
1. update managed state to reflect review completion
2. mention that demo-ready content can be replaced later via `content/` and `/site-build --update`
3. ask whether to keep the preview server running

Output guidance like:

```text
Your site is in good shape.

You can keep refining it in two ways:
- adjust design/layout
- replace demo content with your real text and images

When you update files in content/, run:
  /site-build --update
```

## Key Principles

- Keep feedback loops short.
- Use everyday language.
- Don’t over-interpret vague feedback.
- Preserve user-approved design direction.
- Make demo-ready content feel like a feature, not an apology.
