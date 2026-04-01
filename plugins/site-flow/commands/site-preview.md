---
description: "Preview your website in the browser and make focused adjustments based on feedback"
argument-hint: ""
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion]
---

# /site-flow:site-preview — Preview & Iterate

You are helping a non-technical user preview their website and make adjustments based on feedback.

## Pre-check

1. Read `.site/config.json` — if missing, tell the user to run `/site-flow:site-init` first.
2. Read `.site/workflow-state.json` if present.
3. Check that at least one page has been built.
4. If no pages are built, tell the user to run `/site-flow:site-build` first.
5. Check environment readiness before starting any preview server.

If the chosen stack needs Node.js/npm and they are missing:
- explain this in plain language
- do not attempt preview commands yet
- tell the user what to install
- tell them to rerun `/site-flow:site-preview` or `/site-flow:site-build` after installation

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

If the project uses Listmonk signup, explain that clearly too:
- whether the page is already wired to a real Listmonk public signup target
- whether the current signup experience is only planned and still missing public wiring details
- whether any Listmonk integration warnings are active

If the project uses booking/calendar support, explain that clearly too:
- whether the current booking experience is design-only, planned, or configured
- whether the page already uses a real public booking target or embed target
- whether any booking integration warnings are active

If multilingual support is enabled, explain that clearly too:
- which language is the default language
- which language versions are already available
- whether the current preview is showing a fully localized or still-in-progress language version

Call out clearly when the current preview still has:
- placeholder-heavy imagery
- weak gradient-box visuals
- wireframe-like sections
- missing motion or interaction polish

Frame those as unfinished design work, not acceptable final output.

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

Preview changes should be attempted through a focused sub-agent first.

The main session may:
- classify the feedback
- identify the affected page/file set
- choose the appropriate agent
- summarize what changed for the user
- apply the same scoped change directly as `main-session-fallback` if the helper cannot launch

Do not expose git/worktree internals to the user as a prerequisite for continuing preview iteration.

### Preview change agent rules
Provide:
- current page file(s)
- current design tokens
- user feedback
- whether the affected content is real, seeded-demo, or placeholder-minimal
- current visual state and image source by section
- email-signup requirements only when the affected page includes email capture
- Listmonk integration details when the affected page includes Listmonk signup
- booking/calendar requirements only when the affected page includes a booking surface
- language context when multilingual support is enabled
- any validation warnings for missing imagery, weak placeholders, lack of design richness, incomplete provider wiring, or incomplete localization

Rules:
- preserve unaffected content and structure
- do not redesign unrelated sections
- keep responsive behavior intact
- keep within approved motion/design system unless the user explicitly wants a broader restyle
- improve design completeness when the user calls out weak visuals, missing imagery, or placeholder-looking sections
- preserve provider-specific signup wiring when the page includes Listmonk
- preserve provider-specific booking action or embed behavior when the page includes booking/calendar
- when multilingual support is enabled, keep edits scoped to the active language unless the user explicitly asks for broader language updates
- only load the affected page's content inputs and any directly relevant shared content
- if the helper agent cannot launch, apply only the same narrowly scoped change as `main-session-fallback` and record the fallback reason

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
2. mention that demo-ready content can be replaced later via `content/` and `/site-flow:site-build --update`
3. ask whether to keep the preview server running

Output guidance like:

```text
Your site is in good shape.

You can keep refining it in two ways:
- adjust design/layout
- replace demo content with your real text and images

When you update files in content/, run:
  /site-flow:site-build --update
```

## Key Principles

- Keep feedback loops short.
- Use everyday language.
- Don’t over-interpret vague feedback.
- Preserve user-approved design direction.
- Make demo-ready content feel like a feature, not an apology.
