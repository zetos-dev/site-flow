---
description: "Initialize a static website project — discover requirements, recommend design, generate blueprint and demo-ready seed plan"
argument-hint: "[project-directory]"
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion, WebFetch]
---

# /site-init — Website Project Initialization

You are guiding a **non-technical user** through setting up a professional static website. Use everyday language. Avoid frontend jargon unless absolutely necessary.

**Arguments**: $ARGUMENTS (optional project directory path; otherwise use current directory)

## Product Principle

This workflow should help the user see something close to a finished website early.

Default behavior:
- use `demo-ready` seeded content unless the user clearly wants sparse placeholders
- prefer real-looking default imagery first: stock-library for most sections, AI-generated images for key visuals when available, and designed placeholders only as fallback
- make the generated plan coherent enough that `/site-build` can produce polished pages without asking for lots of materials first

## Pre-check: Existing Project

If `.site/config.json` already exists:

1. Read it and inspect current status.
2. If the project is completed, ask whether to start a new planning cycle.
3. If unfinished, ask whether to continue or start fresh.
4. If the user chooses fresh, only reset `.site/` managed planning/build files. Do not delete their source code or content files.

## Phase 1 — Discovery

Use `AskUserQuestion` to collect a first round of simple inputs.

Ask in one batch:

1. **Site Type**
   - Business / Company Website
   - Personal Portfolio / Resume
   - Product / Service Landing Page
   - Event / Campaign Page

2. **Brand / Project Description**
   - ask as free-text via Other
   - question: brand name plus 1-2 sentence description

3. **Audience**
   - B2B
   - B2C
   - Specific industry
   - Mixed audience

4. **Inspiration**
   - I have websites I like
   - I’ll trust your recommendations
   - I have logo / brand assets

If the user provides reference URLs, use `WebFetch` to extract:
- color tendencies
- typography mood
- layout patterns
- image style
- overall tone

Then infer:
- site type category
- industry / business profile
- likely visual expectations
- likely content profile for seeded demo material

## Phase 2 — Guided Recommendations

Recommend choices rather than asking open-ended technical questions.

### 2a. Recommend Site Structure
Use `AskUserQuestion` with a multi-select page list tailored to the site type.

Examples:
- business: Homepage, About, Services/Products, Contact
- portfolio: Homepage, Work, About, Testimonials, Contact
- event: Homepage, Event Details, Schedule, Speakers, Registration
- landing: Homepage, Features, Pricing, FAQ, Contact/Get Started

### 2b. Recommend Design Style
Present 3-4 style directions matched to the user’s business type and references.

### 2c. Recommend Color Palette
Present 2-3 palette choices with compact preview blocks.

## Phase 3 — Planning Outputs

After confirmation, generate the planning artifacts.

### 3a. Determine Tech Stack Automatically
- 1-2 pages: `html-tailwind`
- 3+ pages: `astro-tailwind`

Do not ask the user to choose the stack.

After planning, the workflow should check whether this computer is ready to build and preview the chosen stack. If something important is missing, it should pause, explain what to install in simple language, and let the user continue later.

### 3b. Create `.site/config.json`
Include the usual project metadata plus default seeded-content settings.

Required fields:

```json
{
  "project_name": "<brand>",
  "site_type": "<business|portfolio|landing|event|other>",
  "tech_stack": "<html-tailwind|astro-tailwind>",
  "design_style": "<style-slug>",
  "pages": ["<slug>"],
  "total_pages": 4,
  "current_page": 0,
  "status": "initialized",
  "seed_mode": "demo-ready",
  "content_profile": "<enterprise-tech|professional-services|...>",
  "bootstrap_strategy": "in-place",
  "image_strategy": "hybrid",
  "image_sources": ["real-user-assets", "stock-library", "ai-generated", "designed-placeholder"],
  "image_profile": "<industry-appropriate style profile>"
}
```

### 3c. Create `.site/workflow-state.json`
This file tracks build orchestration state.

Use this exact structure:

```json
{
  "stage": "initialized",
  "current_target": null,
  "completed_pages": [],
  "failed_pages": [],
  "bootstrap_strategy": "in-place",
  "seed_mode": "demo-ready",
  "content_profile": "<profile>",
  "environment_readiness": {
    "status": "pending",
    "missing": [],
    "checked_at": null,
    "next_step": "",
    "resume_command": "/site-build"
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

### Stage values
Allowed `stage` values:
- `initialized`
- `environment_checking`
- `environment_blocked`
- `environment_ready`
- `bootstrapped`
- `building_page:<slug>`
- `pages_built`
- `validated`
- `reviewed`
- `blocked`

### Stage transition rules
- After `/site-init`: `initialized`
- While checking local requirements: `environment_checking`
- If required tools are missing: `environment_blocked`
- After setup confirms readiness: `environment_ready`
- After bootstrap finishes and residue scan passes: `bootstrapped`
- While building a page: `building_page:<slug>`
- After all requested pages are built: `pages_built`
- After validation passes: `validated`
- After preview/review approval: `reviewed`
- If a non-environment blocking error prevents progress: `blocked`

### Update rules
- `current_target` should contain the active page slug during page build, otherwise `null`
- `completed_pages` should append successful page slugs only once
- `failed_pages` should list pages that could not be completed cleanly
- `environment_readiness.status`: `pending | checking | ready | blocked`
- `environment_readiness.missing` should list missing tools like `node`, `npm`, or `npx`
- `environment_readiness.next_step` should be a plain-language next action for the user
- `environment_readiness.resume_command` should usually be `/site-build`
- `last_action` should be a short human-readable summary
- `last_updated_at` must be refreshed whenever the file changes
- `residue_checks.status`: `pending | passed | warning | failed`
- `validation_results.status`: `pending | passed | warning | failed`


### 3d. Create `.site/design-tokens.md`
In addition to colors, typography, spacing, and visual principles, include:
- approved motion token set
- approved visual placeholder archetypes
- design principles for seeded demo output

Required sections:
- Color system
- Typography
- Spacing & layout
- Border radius / shadows
- Motion tokens
- Visual archetypes
- Tailwind config overrides
- Design principles

### 3e. Create `.site/site-blueprint.md`
Describe:
- page list and build order
- key sections per page
- shared navigation/footer structure
- overall conversion path
- where seeded demo content is acceptable vs where real content is preferred

### 3f. Create `.site/content-guide.md`
This file explains how content and image states work.

It must define:
- `real` — user-supplied content/assets
- `seeded-demo` — polished AI-generated text for preview/demo quality
- `placeholder-minimal` — last-resort lightweight placeholder

It must also define image sourcing priority:
- `real-user-assets`
- `stock-library`
- `ai-generated`
- `designed-placeholder`

Also explain:
- which sections most benefit from real assets first
- which sections can safely use stock imagery first
- which sections should prefer AI-generated hero/key-visual treatment
- that the site can be previewed before all materials are ready

### 3g. Create `.site/page-spec-{slug}.md`
Each page spec must include:
- page purpose
- section list
- layout intent
- content requirements by section
- content state defaults by section
- image source preference by section
- allowed visual archetypes only as fallback for image-heavy sections
- allowed motion tokens for the page
- accessibility / responsiveness notes

Use this structure:

```markdown
# Page Spec: {Page Name}

## Purpose
{what this page needs to accomplish}

## Route
`/{slug}`

## Sections
1. {Section Name}
2. {Section Name}

## Section Requirements

### {Section Name}
- Goal: {what the section should communicate}
- Layout: {hero / split / grid / cards / etc.}
- Content files:
  - `content/{NN}-{page}/...`
- Default content state: `real | seeded-demo | placeholder-minimal`
- Image source preference: `real-user-assets | stock-library | ai-generated | designed-placeholder`
- Allowed visual archetypes (fallback only): {list}
- Allowed motion tokens: {list}
- Notes: {accessibility / responsive notes}
```


### 3h. Create `content/` structure
Generate page folders and shared folders with clear, editable source files.

Pre-fill them with either:
- real content placeholders if the user supplied content
- otherwise seeded demo content aligned to the chosen content profile

Never use lorem ipsum.

## Seeded Demo Content Rules

Default seed mode is `demo-ready`.

### Text
Generate industry-appropriate seeded content for:
- hero copy
- service summaries
- about/story sections
- proof points / stats language
- FAQ
- calls to action
- footer copy

Rules:
- make it feel intentional and polished
- keep claims illustrative, not falsely specific
- avoid fake customer names unless clearly labeled fictional examples

### Images / Visuals
Default image goal: the user should see something close to a finished site, not just framed placeholders.

Plan image sourcing in this order:
1. `real-user-assets`
2. `stock-library`
3. `ai-generated`
4. `designed-placeholder`

For pages with strong brand impact, plan a hybrid default:
- hero / key visual / case-study cover: prefer `ai-generated`
- ordinary supporting sections: prefer `stock-library`
- use `designed-placeholder` only when the earlier sources are unavailable

### Motion
Define a restrained motion system using named tokens such as:
- reveal-soft
- hover-lift
- border-glow
- stagger-cards
- panel-parallax-lite

## User Communication Rules

- Keep the conversation simple and confidence-building.
- Recommend; do not overwhelm.
- Frame the output as something the user can preview quickly.
- Make it clear they can replace demo content later without rebuilding the entire design direction.
- Do not assume the user knows git, repositories, branches, or worktrees.
- Explain environment setup in everyday language when the chosen stack needs extra tools.
