---
description: "Build the website through staged sub-agents, or update content from the content/ directory"
argument-hint: "[page-name | --update | --all]"
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion]
---

# /site-flow:site-build — Website Build Orchestrator

You are the **orchestrator** for building a static website. The main conversation must stay lean: you do orchestration, environment readiness, dependency installation/setup, state checks, sub-agent dispatch, fallback coordination, report aggregation, and user communication. Bootstrap, page implementation, content refreshes, design revisions, and validation should be attempted in focused sub-agents first, with graceful in-session fallback when agent launch is unavailable. Do not make the workflow depend on git or worktrees.

**Arguments**: $ARGUMENTS
- No argument: Build the next pending page
- `<page-name>`: Build a specific page (e.g. `/site-flow:site-build about`)
- `--update`: Re-read `content/` and update built pages with newer content only
- `--all`: Build all pending pages sequentially

## Core Workflow Rule

Never handle initialization, dependency setup, framework bootstrap, page implementation, and validation in one long main-session flow.

Use staged handoffs:
1. Environment readiness
2. Discovery / plan check
3. Bootstrap
4. Page build loop
5. Validation

Execution policy:
- The main session may perform environment readiness, dependency installation/setup, state/report reads and writes, stage selection, and agent dispatch.
- The main session should attempt dedicated agent execution first for bootstrap, page work, content refreshes, preview-driven design edits, and validation.
- If agent launch succeeds, use the agent path.
- If agent launch fails or is unavailable because of harness/session/worktree/git constraints, continue in the current project directory as `main-session-fallback`.
- Git/worktree availability must never be treated as a user prerequisite.
- Agent-launch failure alone must not block the workflow; record the fallback reason in state and reports instead.

Do not assume git, branches, repositories, or worktrees. The normal workflow must work directly in the current project directory.

## Pre-check

1. Read `.site/config.json` — if missing, tell the user: `No website project found. Run /site-flow:site-init first.`
2. Read `.site/site-blueprint.md`
3. Read `.site/design-tokens.md`
4. If `.site/workflow-state.json` exists, read it. If it does not exist, infer state from project files and continue.
5. Use Glob to inspect existing project files (`package.json`, `src/`, `index.html`, `.site/page-*-completion-report.md`).

## Managed State

Prefer `.site/workflow-state.json` as the runtime source of truth.

Use this structure:

```json
{
  "stage": "initialized",
  "current_target": null,
  "completed_pages": [],
  "failed_pages": [],
  "bootstrap_strategy": "in-place",
  "seed_mode": "demo-ready",
  "content_profile": "enterprise-tech",
  "environment_readiness": {
    "status": "pending",
    "missing": [],
    "checked_at": null,
    "next_step": "",
    "resume_command": "/site-flow:site-build"
  },
  "delegation": {
    "policy": "prefer-agent-with-fallback",
    "last_agent": null,
    "last_execution_mode": null,
    "fallback_events": [],
    "stage_agents": {
      "bootstrap": null,
      "page_build": [],
      "content_update": [],
      "validation": null
    },
    "main_session_execution_violations": []
  },
  "last_action": "project initialized",
  "last_updated_at": "<ISO date>",
  "residue_checks": {
    "status": "pending",
    "warnings": [],
    "last_checked_at": null
  },
  "validation_results": {
    "status": "pending",
    "notes": [],
    "last_checked_at": null
  }
}
```

### Stage Transition Rules
- set `stage: environment_checking` while checking local requirements
- set `stage: environment_blocked` if required tools are missing
- set `stage: environment_ready` after the readiness check confirms Node.js/npm support for the chosen stack
- set `stage: bootstrapped` after bootstrap completes, residue scan passes, and the bootstrap report exists
- set `stage: building_page:<slug>` before building that page
- append a page slug to `completed_pages` only after its completion report exists
- append a page slug to `failed_pages` only if the page could not be completed cleanly
- set `stage: pages_built` after all requested pages complete
- set `stage: validated` after validation passes and the validation report exists
- set `stage: blocked` only if progress cannot continue because of a real build/input/environment failure
- record approved direct execution after agent-launch failure in `delegation.fallback_events`, not `delegation.main_session_execution_violations`
- record only unauthorized direct execution in `delegation.main_session_execution_violations`
- update `environment_readiness.checked_at`, `last_action`, and `last_updated_at` whenever state changes

### Required Sidecar Reports
Maintain these files where applicable:
- `.site/bootstrap-report.md`
- `.site/validation-report.md`

#### `.site/bootstrap-report.md` format

```markdown
# Bootstrap Report

**Tech Stack**: {tech_stack}
**Strategy**: {in-place | staging}
**Completed**: {ISO date}
**Executor**: {sub-agent | main-session-fallback}
**Agent**: {bootstrap-astro | bootstrap-html | none}
**Delegation outcome**: {agent-used | fallback-used}
**Fallback reason**: {none | agent-launch-failed | agent-unavailable | other}

## Actions Taken
- {action}
- {action}

## Files Created or Normalized
- `{path}` — {reason}

## Residue Scan
- README residue: {passed | warning | failed}
- Package name residue: {passed | warning | failed}
- Helper directory residue: {passed | warning | failed}
- Starter page residue: {passed | warning | failed}

## Notes
- {important note}
```

#### `.site/validation-report.md` format

```markdown
# Validation Report

**Checked**: {ISO date}
**Requested Scope**: {page or all pages}
**Executor**: {sub-agent | main-session-fallback}
**Agent**: {site-validator | none}
**Delegation outcome**: {agent-used | fallback-used}
**Fallback reason**: {none | agent-launch-failed | agent-unavailable | other}

## Build Verification
- Build command: `{command}`
- Result: {passed | failed}

## Residue Scan
- README residue: {passed | warning | failed}
- Package name residue: {passed | warning | failed}
- Helper directory residue: {passed | warning | failed}
- Starter page residue: {passed | warning | failed}

## Page Coverage
- Requested pages: {list}
- Built pages: {list}
- Missing completion reports: {list or none}

## Content Quality
- Lorem ipsum: {not found | found}
- Weak filler: {not found | found}
- Seeded-demo coherence: {passed | warning | failed}

## Visual Completeness
- Required visual sections complete: {passed | warning | failed}
- Hero / key visual quality: {passed | warning | failed}
- Wireframe-like sections: {not found | found}

## Image Source Coverage
- Real-user-assets: {summary}
- Stock-library: {summary}
- AI-generated: {summary}
- Designed graphics: {summary}
- Placeholder-minimal debt: {summary}

## Motion & Interaction Quality
- Motion expectation: {passed | warning | failed}
- Interactive polish: {passed | warning | failed}

## Finish Readiness
- Design richness: {passed | warning | failed}
- Next visual improvements: {list or none}

## Outcome
- Status: {passed | warning | failed}
- Next step: {short recommendation}
```


## Mode: --update

If `--update` is present:

1. Run the Environment Readiness stage first.
2. Scan `content/` for user-updated files.
3. Read completion reports to determine built pages.
4. For each built page with content changes, attempt the dedicated content-update agent first; if it cannot launch, continue as `main-session-fallback` in the current project directory.
5. Do **not** change structure, layout, component hierarchy, or visual language.
6. After updates, run the Validation stage.

### Content Update Agent Rules

Attempt `plugins/site-flow/agents/content-updater.md` first with these rules:

```text
You are updating content on an existing website page.

Rules:
- Only replace content that now has better source material.
- Preserve layout, structure, styling, and existing component boundaries.
- Preserve seeded-demo content where no real content has been supplied.
- Do not redesign the page.
- Do not change shared design tokens.
- Keep the page visually complete after the content swap.
- Do not regress imagery or motion richness while updating content.
- If agent launch is unavailable, execute the same scoped update as main-session-fallback and record the fallback reason.

Inputs you must use:
- Page name and file path(s)
- Content mapping
- Current content states per section: real / seeded-demo / placeholder-minimal
- Relevant design tokens for reference only
- Current image state and visual strategy per section
- Updated source content for the current page only
- Email-signup requirements only when the page includes an email capture surface
```

Then continue to **Validation Stage**.

## Mode: Regular Build

## Stage 1 — Environment Readiness

This stage should happen before bootstrap, page build, update, or preview-oriented commands.

This is the only execution stage that may run entirely in the main session.

### Goals
- Check whether the current machine is ready for the chosen stack
- Detect missing `node`, `npm`, and when relevant `npx`
- Explain missing requirements in plain language
- Return a clear resume path without attempting system installation automatically

### Rules
- Do not assume git, repositories, branches, or worktrees.
- Do not create worktrees.
- Do not treat missing worktree support as an error condition.
- Do not run system-level installation commands inside a helper.
- If required tools are missing, stop here and update `.site/workflow-state.json`.

### Readiness outcomes
- `ready`
- `blocked` with missing tool list

### Example blocked guidance
- "Before I can build this website, this computer needs Node.js and npm."
- "Install Node.js, then run `/site-flow:site-build` again and I will continue from here."

Only after readiness is confirmed should the workflow continue.

## Stage 2 — Discovery / Plan Check

1. Determine requested scope:
   - next page
   - specific page
   - all remaining pages
2. Read relevant `.site/page-spec-{slug}.md` files.
3. Inspect existing build state from completion reports.
4. Determine whether this is:
   - fresh build
   - resume build
   - rebuild of a specific page
5. Determine project type:
   - Astro + Tailwind
   - HTML + Tailwind

### Main-session discovery rules

Stage 2 may run in the main session because it is orchestration and planning work only.

The main session may:
- inspect the target directory
- detect whether `node`, `npm`, and when relevant `npx` are available
- detect whether the project is already bootstrapped
- explain missing requirements in plain language
- decide which dedicated agent to dispatch next

The main session must not:
- create page files
- install tools automatically beyond explicit environment/setup steps already approved by the user
- create worktrees
- improvise copy-then-cleanup project migration flows
- treat missing git/worktree support as an error
- perform bootstrap or page edits directly

## Stage 3 — Bootstrap

### Default Bootstrap Strategy
Use **deterministic in-place bootstrap**.

That means:
- initialize directly in the final project root
- normalize starter output immediately
- finish bootstrap with a residue scan before any page build begins

### Mandatory execution rule
Bootstrap should be attempted in a dedicated bootstrap agent first:
- Astro projects: `plugins/site-flow/agents/bootstrap-astro.md`
- HTML projects: `plugins/site-flow/agents/bootstrap-html.md`

If the bootstrap agent cannot launch, continue via `main-session-fallback` in the current project directory and record the fallback reason in state and the bootstrap report.

The main session should normally prepare inputs, dispatch the correct bootstrap agent, and record the resulting report/state. Fallback execution is acceptable when agent launch is unavailable.

### Forbidden Bootstrap Behavior
- Do not scaffold into hidden helper directories and then copy everything into root.
- Do not rely on a final `rm -rf` cleanup step.
- Do not leave starter metadata in place for later cleanup.
- Do not let page-builder fix bootstrap leftovers.

### Allowed Fallback Strategy
Only if in-place bootstrap is clearly blocked, use staging under:
- `.site/bootstrap-work/`

If staging is used:
- only copy a whitelisted final file set into root
- never copy the entire scaffold tree blindly
- treat staging as internal implementation detail

### Bootstrap agent contracts
Dispatch one of these dedicated agent files with the project context and stack-specific inputs:
- `plugins/site-flow/agents/bootstrap-astro.md`
- `plugins/site-flow/agents/bootstrap-html.md`

### Residue Scan Gate

After bootstrap, inspect for residue before allowing any page build.

Examples of residue warnings:
- default Astro starter README still present
- starter `package.json.name`
- helper directories like `.astro-starter/`
- untouched starter `src/pages/index.astro`

If residue is found, fix it before continuing.

## Stage 4 — Page Build Loop

Build pages **sequentially**, never in parallel. Later pages may reuse patterns established earlier.

For each page:

### Step 4.1 — Prepare Context

1. Read `.site/page-spec-{slug}.md`
2. Read `.site/design-tokens.md`
3. Read `.site/site-blueprint.md`
4. Read `.site/content-guide.md` if present
5. Read `.site/config.json` and check whether optional email signup is enabled and relevant to this page
6. Check `content/{NN}-{page-name}/` for user content
7. Read 1-2 existing pages/components for established patterns if this is not the first page
8. Read previous completion reports if relevant

### Step 4.2 — Build Structured Inputs

Construct a page-builder prompt that explicitly provides:
- project name
- tech stack
- complete design tokens
- complete page specification
- relevant optional feature decisions from `.site/config.json`
- content mapping
- section-by-section content state map: `real | seeded-demo | placeholder-minimal`
- image source plan per section
- image state map per section
- seeded visual archetypes allowed only as fallback for the page
- motion token set allowed for the page
- references to existing patterns
- actual source material only for the target page
- email-signup requirements only when this page is one of the configured capture locations

Use this exact input shape inside the prompt:

```text
Content State Map:
- Hero:
  - state: seeded-demo
  - content_files:
    - content/01-homepage/hero-title.md
    - content/01-homepage/hero-description.md
  - motion_tokens:
    - reveal-soft

Image Source Plan:
- Hero:
  - preferred: ai-generated
  - fallback: stock-library
  - final_fallback: designed-placeholder
- Team Section:
  - preferred: stock-library
  - fallback: designed-placeholder
- Contact Background:
  - preferred: stock-library
  - fallback: designed-placeholder
```

If a section has no real image, the orchestrator must still provide a preferred image source and a fallback path.

Do not carry the whole site's `content/` corpus in the main conversation context. Only load and inject the current page's referenced content files and any directly relevant shared content.

### Step 4.3 — Page Builder Agent Rules

Attempt `plugins/site-flow/agents/page-builder.md` first for every page. The page builder agent must:
- build only the requested page
- use `real` content when provided
- otherwise use `seeded-demo` content by default
- only fall back to `placeholder-minimal` if necessary
- use approved visual archetypes only as fallback
- use only approved motion tokens
- create/update the page completion report
- preserve or improve design richness during implementation

If the page-builder agent cannot launch, the main session may execute the same scoped page task as `main-session-fallback` and must record the fallback reason.

## Seeded Demo Content Policy

This workflow defaults to `demo-ready` output.

That means when real content is missing, the built pages should still feel close to a finished website.

### Text Seeding
Use industry-aware seeded content that is:
- specific enough to feel intentional
- internally consistent across pages
- safe for demos
- never lorem ipsum
- never fake real customer claims presented as fact

### Visual Seeding
Default image goal: help the user see a near-finished site quickly.

Use this priority:
1. real-user-assets
2. stock-library
3. ai-generated
4. designed-placeholder

For hero or key-visual sections, a hybrid strategy is acceptable:
- hero / key visual / case-study cover: prefer `ai-generated`
- supporting sections: prefer `stock-library`
- use `designed-placeholder` only as the last fallback

A blank gradient block, empty frame, or generic decorative slab does not count as finished design for a required visual section.

### Design richness requirement
This workflow is for finished-looking website design, not scaffold output.

Every completed page should show visible design richness, including where appropriate:
- meaningful imagery or deliberate graphic composition
- hierarchy, contrast, rhythm, and depth
- polished section-specific visual storytelling
- intentional motion and interaction feedback
- refined hero and CTA treatments

Treat these as failures in required or above-the-fold sections:
- generic gradient placeholder blocks
- empty image frames
- starter-template-looking layouts
- box-only sections that leave the design burden to the user

### Motion Seeding
Use motion tokens defined in design tokens, for example:
- reveal-soft
- hover-lift
- border-glow
- stagger-cards
- panel-parallax-lite

Do not improvise unrelated animation systems.

## Stage 5 — Validation Agent

Always finish a build or update run with a validation stage. Attempt `plugins/site-flow/agents/site-validator.md` first.

If the validation agent cannot launch, continue via `main-session-fallback` and record the fallback reason in state and the validation report.

### Validation responsibilities
- run build verification
- scan for starter residue
- verify all requested pages exist
- verify completion reports exist
- check content quality:
  - no lorem ipsum
  - no generic “your company” filler unless explicitly requested
  - seeded-demo coverage is coherent
- verify required visual sections are complete
- fail pages that still rely on gradient placeholder boxes, empty frames, or wireframe-like design in required visual areas
- verify motion/interaction polish where the page spec expects it
- report next actions clearly

Validation must explicitly distinguish between:
- acceptable finished visuals: real assets, stock imagery, AI-generated imagery, deliberate designed graphics
- unacceptable unfinished visuals: blank placeholders, generic gradient slabs, empty mock frames

## Required Reports

Maintain these artifacts where applicable:
- `.site/workflow-state.json`
- `.site/bootstrap-report.md`
- `.site/page-{slug}-completion-report.md`
- `.site/validation-report.md`

## User Communication Rules

- Keep the user-facing summary short and non-technical.
- Tell the user what stage is happening now.
- If blocked, explain exactly what is blocked.
- When build work completes, summarize:
  - pages built or updated
  - whether residue scan passed
  - whether the site is ready for preview
