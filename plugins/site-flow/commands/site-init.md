---
description: "Initialize a static website project — discover requirements, recommend design, generate blueprint and demo-ready seed plan"
argument-hint: "[project-directory]"
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion, WebFetch]
---

# /site-flow:site-init — Website Project Initialization

You are guiding a **non-technical user** through setting up a professional static website. Use everyday language. Avoid frontend jargon unless absolutely necessary.

**Arguments**: $ARGUMENTS (optional project directory path; otherwise use current directory)

## Product Principle

This workflow should help the user see something close to a finished website early.

Default behavior:
- use `demo-ready` seeded content unless the user clearly wants sparse placeholders
- prefer real-looking default imagery first: stock-library for most sections, AI-generated images for key visuals when available, and designed placeholders only as fallback
- make the generated plan coherent enough that `/site-flow:site-build` can produce polished pages without asking for lots of materials first
- require visual richness: imagery, motion, hierarchy, depth, and deliberate composition should make the site feel like a finished design, not a scaffold
- treat gradient placeholder boxes and empty visual frames as failure states for design-heavy sections unless the page spec explicitly allows an abstract finished graphic treatment

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

Discovery batching rule:
- Batch 1: base website questions only
- Batch 2: optional email/Listmonk questions only if email support is needed
- Batch 3: optional booking/calendar questions only if booking support is needed
- Batch 4: optional multilingual questions only if multilingual support is needed
- Do not combine multiple optional feature areas into one AskUserQuestion batch.

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
- event: Homepage, Event Details, Schedule, Speakers, Contact
- landing: Homepage, Features, Pricing, FAQ, Contact/Get Started

### 2b. Recommend Design Style
Present 3-4 style directions matched to the user’s business type and references.

### 2c. Recommend Color Palette
Present 2-3 palette choices with compact preview blocks.

### 2d. Ask About Optional Email Support
Run this as a separate AskUserQuestion batch after the base website questions are complete.

Ask one simple non-technical question before generating planning artifacts:
- Does this website need email support for messages, feedback, newsletter updates, or site updates?

If yes, ask follow-ups in plain language:
- Should this be shown as a contact/messages surface, an updates/newsletter surface, or both?
- If they already know it, do they want to plan around `Listmonk`, another service, or leave it undecided?

Rules:
- Do not ask for Listmonk URLs, endpoints, list UUIDs, or other connection details in chat during initialization.
- Generate editable config files instead, so the user can fill real integration details in later.
- Use `design-only` when the user only wants the experience represented in the design.
- Use `planned` when the feature is enabled but the integration config still needs real values.
- Use `configured` only when the generated integration config file later contains enough public details for real provider wiring.
- Tell the user they can edit the generated config file after site generation and rerun build/update.

Keep this optional.
- If the user does not need email support, do not force empty integration placeholders into the project config.
- If the user wants email support, decide likely capture locations such as homepage hero CTA, footer updates block, contact page, or a dedicated messages/updates section.

### 2e. Ask About Optional Booking / Calendar
Run this as a separate AskUserQuestion batch only after the optional email batch is complete.
Do not combine booking/calendar follow-ups with email or multilingual follow-ups.

Ask one simple non-technical question before generating planning artifacts:
- Does this website need appointment booking or calendar scheduling?

If yes, ask follow-ups in plain language:
- Is this just for the page design right now, or should the project prepare for a real booking service later?
- Where should this appear: `homepage hero`, `contact page`, `booking section`, or `footer CTA`?
- Should visitors reach booking through a `link button`, an `embedded section`, or a `popup/modal`?

Rules:
- Do not ask for booking URLs, embed URLs, or provider connection details in chat during initialization.
- Generate an editable calendar config file instead, so the user can fill real integration details in later.
- Use `design-only` when the user only wants the booking experience represented visually.
- Use `planned` when the feature is enabled but the integration config still needs real values.
- Use `configured` only when the generated integration config file later contains enough public details for real provider wiring.
- Tell the user they can edit the generated config file after site generation and rerun build/update.

Keep this optional.
- If the user does not need booking, do not force empty integration placeholders into the project config.
- If the user wants booking, decide likely capture locations such as homepage hero CTA, contact page, a dedicated booking block, or a footer CTA.

### 2f. Ask About Multilingual Support
Run this as the final optional AskUserQuestion batch after the email and booking decisions are complete.
Keep this batch lightweight.

Ask one simple non-technical question before generating planning artifacts:
- Does this website need more than one language?

If yes, ask follow-ups in plain language:
- What should the main/default language be?
- If they already know a likely next language, what is it?

Rules:
- Do not force the user to supply all translated content during initialization.
- Treat the default language as the only required language at init time.
- Use multilingual planning only as a structure and workflow reservation unless the user explicitly wants translation work now.
- Prefer a shared-site structure with language-specific content files rather than separate site copies.

## Phase 3 — Planning Outputs

After confirmation, generate the planning artifacts.

Prefer a focused sub-agent for this planning/content generation stage so the main conversation stays lean. Attempt `plugins/site-flow/agents/init-planner.md` first. If the agent cannot launch, continue in the current session as documented fallback.

Execution policy:
- helper-agent startup must never depend on git
- if startup fails because the directory is not a repository, has no initial commit, has unresolved `HEAD`, lacks git history, or cannot resolve worktree/base-branch metadata, continue in the current project directory as `main-session-fallback`
- do not surface git/worktree setup as a user prerequisite
- record the fallback reason in managed state or reports where applicable, but do not block initialization

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
  "image_profile": "<industry-appropriate style profile>",
  "optional_features": {
    "email_service": {
      "mode": "<design-only|planned|configured>",
      "provider": "<listmonk|other|undecided>",
      "capture_locations": ["<homepage-hero|footer|contact|updates-block>"],
      "surface_type": "<messages|updates|both>",
      "integration_status": "<planned|configured>"
    },
    "calendar_service": {
      "mode": "<design-only|planned|configured>",
      "provider": "<calendly|google-calendar|other|undecided>",
      "capture_locations": ["<homepage-hero|contact|booking-section|footer>"],
      "interaction_type": "<link-out|embed|popup>",
      "integration_status": "<planned|configured>"
    }
  },
  "i18n": {
    "enabled": true,
    "default_language": "<zh|en|ja|fr|...>",
    "languages": ["<default-language>"],
    "strategy": "content-only",
    "translation_mode": "manual-assisted",
    "fallback_language": "<default-language>"
  }
}
```

Rules:
- Omit `optional_features.email_service` entirely when email support is not needed.
- Omit `optional_features.calendar_service` entirely when booking/calendar is not needed.
- Use `design-only` when the user only wants the email or booking experience represented in the design.
- Use `planned` when the feature is enabled but its generated integration config file still needs real public values.
- Use `configured` when the generated integration config file contains enough public details for real provider wiring.
- When provider is `listmonk`, create `.site/integrations/listmonk.json` as an editable config template for build, preview, status, validation, and later updates.
- When booking/calendar support is enabled, create `.site/integrations/calendar.json` as an editable config template for build, preview, status, validation, and later updates.
- Never store provider secrets or admin-only credentials in generated project files.
- Tell the user they can edit the generated integration config files later and rerun `/site-flow:site-build` or `/site-flow:site-build --update`.
- When multilingual support is enabled, initialize the project with a single default language and reserve structure for future language additions.
- Prefer shared page structure with language-specific content rather than duplicating site templates per language.

### 3c. Create `.site/integrations/listmonk.json`
When `optional_features.email_service.provider` is `listmonk`, create a dedicated editable provider config artifact.

Use a structure like:

```json
{
  "provider": "listmonk",
  "base_url": "<https://lists.example.com>",
  "public_subscription_endpoint": "</api/public/subscription>",
  "public_signup_url": "<optional-public-url>",
  "list_name": "<newsletter-or-updates>",
  "list_id": "<optional-public-id>",
  "list_uuid": "<optional-public-uuid>",
  "opt_in": "<single|double|unknown>",
  "copy": {
    "headline": "<editable default>",
    "helper_text": "<editable default>",
    "button_label": "<editable default>",
    "success_message": "<editable default>",
    "error_message": "<editable default>"
  },
  "status": "<planned|configured>"
}
```

Rules:
- Create this file only when the provider is `listmonk`.
- Create it as an editable template even if the user did not supply real values during initialization.
- Store public integration details only.
- Never put API keys, admin tokens, SMTP credentials, or other secrets in this file.
- Keep copy fields editable and aligned with generated `content/` files.
- Tell the user they can update this file later and rerun build/update.

### 3d. Create `.site/integrations/calendar.json`
When `optional_features.calendar_service` is enabled, create a dedicated editable calendar config artifact.

Use a structure like:

```json
{
  "provider": "<calendly|google-calendar|other|undecided>",
  "public_booking_url": "<optional-public-url>",
  "embed_url": "<optional-public-url>",
  "interaction_type": "<link-out|embed|popup>",
  "copy": {
    "headline": "<editable default>",
    "helper_text": "<editable default>",
    "button_label": "<editable default>"
  },
  "status": "<planned|configured>"
}
```

Rules:
- Create this file only when booking/calendar support is enabled.
- Create it as an editable template even if the user did not supply real values during initialization.
- Store public integration details only.
- Never put secrets or admin-only credentials in this file.
- Tell the user they can update this file later and rerun build/update.

### 3e. Create `.site/workflow-state.json`
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
- After `/site-flow:site-init`: `initialized`
- While checking local requirements: `environment_checking`
- If required tools are missing: `environment_blocked`
- After setup confirms readiness: `environment_ready`
- After bootstrap finishes, residue scan passes, and the bootstrap report exists: `bootstrapped`
- While building a page: `building_page:<slug>`
- After all requested pages are built and each completion report exists: `pages_built`
- After validation passes and the validation report exists: `validated`
- After preview/review approval: `reviewed`
- If a real non-recoverable environment/input/build error prevents progress: `blocked`

### Update rules
- `current_target` should contain the active page slug during page build, otherwise `null`
- `completed_pages` should append successful page slugs only once
- `failed_pages` should list pages that could not be completed cleanly
- `environment_readiness.status`: `pending | checking | ready | blocked`
- `environment_readiness.missing` should list missing tools like `node`, `npm`, or `npx`
- `environment_readiness.next_step` should be a plain-language next action for the user
- `environment_readiness.resume_command` should usually be `/site-flow:site-build`
- `delegation.policy` should be `prefer-agent-with-fallback`
- `delegation.last_agent` should record the most recent executor agent when one was used
- `delegation.last_execution_mode` should be `sub-agent | main-session-fallback`
- `delegation.fallback_events` should record approved fallback uses with stage, target, reason, and timestamp
- `delegation.stage_agents` should record bootstrap, page, update, and validation agents used
- `delegation.main_session_execution_violations` should record only unauthorized direct execution, not approved fallback
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

### 3f. Create `.site/site-blueprint.md`
Describe:
- page list and build order
- key sections per page
- shared navigation/footer structure
- overall conversion path
- where seeded demo content is acceptable vs where real content is preferred
- when email support is enabled, where email/messages or updates surfaces belong in the funnel and which pages/sections should surface them

### 3g. Create `.site/content-guide.md`
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
- that this workflow aims for a visually complete website, not a placeholder scaffold
- that designed placeholders must still feel intentional and finished, not like blank gradient boxes

### 3h. Create `.site/page-spec-{slug}.md`
Each page spec must include:
- page purpose
- section list
- layout intent
- content requirements by section
- content state defaults by section
- image source preference by section
- visual requirement by section: `required | recommended | optional`
- imagery kind by section: `photo | illustration | abstract-brand-graphic | logo-strip | ui-mockup`
- whether placeholders are allowed and at what cap
- allowed visual archetypes only as fallback for image-heavy sections
- allowed motion tokens for the page
- accessibility / responsiveness notes
- finish cues for how the page should feel visually complete
- when email support is enabled, section-level email/messages or updates requirements only on relevant pages

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

## Finish Cues
- Above-the-fold impression: {finished / editorial / premium / energetic / calm / bold}
- Required design elements: {imagery, motion, proof elements, layered composition, etc.}
- Failure states to avoid: {wireframe look, empty visual boxes, generic gradient slabs}

## Section Requirements

### {Section Name}
- Goal: {what the section should communicate}
- Layout: {hero / split / grid / cards / etc.}
- Content files:
  - `content/{NN}-{page}/...`
- Default content state: `real | seeded-demo | placeholder-minimal`
- Visual requirement: `required | recommended | optional`
- Imagery kind: `photo | illustration | abstract-brand-graphic | logo-strip | ui-mockup`
- Image source preference: `real-user-assets | stock-library | ai-generated | designed-placeholder`
- Placeholder allowed: `yes | no`
- Placeholder cap: `none | temporary | final-allowed`
- Allowed visual archetypes (fallback only): {list}
- Allowed motion tokens: {list}
- Notes: {accessibility / responsive notes}
```


### 3i. Create `content/` structure
Generate page folders and shared folders with clear, editable source files.

Pre-fill them with either:
- real content placeholders if the user supplied content
- otherwise seeded demo content aligned to the chosen content profile

Never use lorem ipsum.

The planning/content generation stage should prefer a focused sub-agent.
- The sub-agent may generate the full `content/` tree and page specs.
- The main session should receive a concise per-page content manifest instead of pasting large content bodies back into the conversation.
- For each page, summarize only:
  - page slug
  - section names
  - referenced `content/` files
  - default content state by section
  - image source preference by section

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

Design rule:
- a deliberate graphic composition can count as complete
- a blank gradient rectangle or empty frame cannot

The planning outputs should set `/site-flow:site-build` up to produce a visually rich site with imagery, motion, hierarchy, and polish rather than a basic shell.

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
- Agent orchestration is an internal implementation detail, not a user requirement.
- Never ask the user to configure git/worktrees just to continue building a site.
- Explain environment setup in everyday language when the chosen stack needs extra tools.
