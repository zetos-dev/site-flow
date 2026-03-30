---
description: "Build the website through staged sub-agents, or update content from the content/ directory"
argument-hint: "[page-name | --update | --all]"
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion]
---

# /site-build — Website Build Orchestrator

You are the **orchestrator** for building a static website. The main conversation must stay lean: you do orchestration, state checks, and user communication. All heavy execution work happens in isolated sub-agents.

**Arguments**: $ARGUMENTS
- No argument: Build the next pending page
- `<page-name>`: Build a specific page (e.g. `/site-build about`)
- `--update`: Re-read `content/` and update built pages with newer content only
- `--all`: Build all pending pages sequentially

## Core Workflow Rule

Never handle initialization, dependency setup, framework bootstrap, page implementation, and validation in one long main-session flow.

Use staged sub-agents with explicit handoffs:
1. Discovery / plan check
2. Environment setup agent
3. Bootstrap agent
4. Page builder agent (one page at a time)
5. Validation agent

The main session should summarize progress and update state, but not absorb the full implementation context of each stage.

## Pre-check

1. Read `.site/config.json` — if missing, tell the user: `No website project found. Run /site-init first.`
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
- set `stage: environment_ready` after the setup agent confirms readiness
- set `stage: bootstrapped` only after bootstrap completes and residue scan passes
- set `stage: building_page:<slug>` before dispatching a page-builder for that page
- append a page slug to `completed_pages` only after its completion report exists
- append a page slug to `failed_pages` only if the page could not be completed cleanly
- set `stage: pages_built` after all requested pages complete
- set `stage: validated` after validation passes
- set `stage: blocked` if progress cannot continue without intervention
- update `last_action` and `last_updated_at` whenever state changes

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

## Outcome
- Status: {passed | warning | failed}
- Next step: {short recommendation}
```


## Mode: --update

If `--update` is present:

1. Scan `content/` for user-updated files.
2. Read completion reports to determine built pages.
3. For each built page with content changes, dispatch one sub-agent to update content only.
4. Do **not** change structure, layout, component hierarchy, or visual language.
5. After updates, dispatch the validation agent.

### Content Update Agent Rules

Use a sub-agent prompt with these rules:

```text
You are updating content on an existing website page.

Rules:
- Only replace content that now has better source material.
- Preserve layout, structure, styling, and existing component boundaries.
- Preserve seeded-demo content where no real content has been supplied.
- Do not redesign the page.
- Do not change shared design tokens.

Inputs you must use:
- Page name and file path(s)
- Content mapping
- Current content states per section: real / seeded-demo / placeholder-minimal
- Relevant design tokens for reference only
```

Then continue to **Validation Stage**.

## Mode: Regular Build

## Stage 1 — Discovery / Plan Check

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

## Stage 2 — Environment Setup Agent

Dispatch a dedicated setup sub-agent before any bootstrap or page generation.

### Setup Agent Responsibilities
- Check required tooling for the chosen stack
- Inspect the target directory state
- Detect whether the project is already bootstrapped
- Decide bootstrap strategy
- Record findings for the orchestrator

### Setup Agent Constraints
- Must not create page files
- Must not install unrelated tools
- Must not improvise a copy-then-cleanup project migration flow

### Setup Agent Prompt Requirements

```text
You are the environment setup agent for a static website project.

Your job:
1. Inspect the target directory.
2. Determine whether the project is already bootstrapped.
3. Verify framework prerequisites.
4. Decide whether bootstrap is needed.
5. Report findings clearly.

Rules:
- Do not build any page.
- Do not create reusable site sections.
- Do not move project files between directories.
- If temporary staging seems necessary, recommend it explicitly instead of improvising it.
```

If setup says bootstrap is not needed, skip to Stage 4.

## Stage 3 — Bootstrap Agent

If the project is not initialized, dispatch a dedicated bootstrap sub-agent.

### Default Bootstrap Strategy
Use **deterministic in-place bootstrap**.

That means:
- initialize directly in the final project root
- normalize starter output immediately
- finish bootstrap with a residue scan before any page build begins

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

### Astro Bootstrap Agent Prompt Requirements

```text
You are bootstrapping a new Astro + Tailwind website project.

Goal:
Create a clean project in the final root directory, then normalize it immediately.

Required steps:
1. Initialize Astro in the target root.
2. Add Tailwind and install dependencies.
3. Create shared foundation files:
   - src/layouts/BaseLayout.astro
   - shared header/footer/navigation components
   - shared content loading utilities if needed
4. Normalize starter output immediately:
   - replace starter package naming
   - replace starter README if present
   - remove or rewrite starter-only pages/files
   - ensure no .astro-starter or similar residue remains
5. Produce a bootstrap report.

Rules:
- Do not use copy-then-rm project migration flows.
- Do not leave cleanup for later stages.
- Do not build individual content-heavy pages yet.
- Stop after shared foundation is ready and residue scan passes.
```

### HTML Bootstrap Agent Prompt Requirements

```text
You are bootstrapping a simple HTML + Tailwind static website.

Goal:
Create the minimal final project structure directly in root.

Required steps:
- Create index.html and shared static assets structure
- Set up Tailwind via CDN
- Create shared header/footer patterns if multiple pages are planned
- Apply design tokens
- Produce a bootstrap report

Rules:
- Keep the structure minimal and final
- Do not create throwaway staging folders unless explicitly required
```

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
5. Check `content/{NN}-{page-name}/` for user content
6. Read 1-2 existing pages/components for established patterns if this is not the first page
7. Read previous completion reports if relevant

### Step 4.2 — Build Structured Inputs

Construct a page-builder prompt that explicitly provides:
- project name
- tech stack
- complete design tokens
- complete page specification
- content mapping
- section-by-section content state map: `real | seeded-demo | placeholder-minimal`
- seeded visual archetypes allowed for the page
- motion token set allowed for the page
- references to existing patterns

Use this exact input shape inside the prompt:

```text
Content State Map:
- Hero:
  - state: seeded-demo
  - content_files:
    - content/01-homepage/hero-title.md
    - content/01-homepage/hero-description.md
  - visual_archetype: brand-gradient
  - motion_tokens:
    - reveal-soft
- Logo Strip:
  - state: placeholder-minimal
  - content_files:
    - content/01-homepage/logo-strip.md
  - visual_archetype: logo-strip-placeholder
  - motion_tokens:
    - stagger-cards
- Features:
  - state: real
  - content_files:
    - content/01-homepage/feature-1.md
    - content/01-homepage/feature-2.md
  - visual_archetype: none
  - motion_tokens:
    - hover-lift
```

If a section has no real image, the orchestrator must still provide one approved `visual_archetype` or explicitly set it to `none`.


### Step 4.3 — Page Builder Agent Rules

The page builder sub-agent must:
- build only the requested page
- use `real` content when provided
- otherwise use `seeded-demo` content by default
- only fall back to `placeholder-minimal` if necessary
- use approved visual archetypes for missing images
- use only approved motion tokens
- create/update the page completion report

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
Prefer designed placeholder archetypes over empty boxes or generic gradients alone.

Examples:
- brand-gradient
- dashboard-mockup
- team-silhouette-panel
- abstract-office-scene
- device-frame
- metric-card-cluster
- logo-strip-placeholder
- case-study-cover-panel

### Motion Seeding
Use motion tokens defined in design tokens, for example:
- reveal-soft
- hover-lift
- border-glow
- stagger-cards
- panel-parallax-lite

Do not improvise unrelated animation systems.

## Stage 5 — Validation Agent

Always finish a build or update run with a validation sub-agent.

### Validation Agent Responsibilities
- run build verification
- scan for starter residue
- verify all requested pages exist
- verify completion reports exist
- check content quality:
  - no lorem ipsum
  - no generic “your company” filler unless explicitly requested
  - seeded-demo coverage is coherent
- report next actions clearly

### Validation Agent Prompt Requirements

```text
You are the validation agent for a generated static website.

Your job:
1. Verify the requested pages were built.
2. Run the appropriate build check.
3. Scan for starter residue and unfinished scaffold artifacts.
4. Review content quality and content-state usage.
5. Produce a concise validation report.

Check specifically for:
- starter README remnants
- starter package naming
- helper scaffold folders
- missing completion reports
- sections still using weak placeholders instead of seeded-demo content
```

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
