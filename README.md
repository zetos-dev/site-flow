# site-flow

AI-driven static website workflow for Claude Code. It helps non-technical users go from idea to a polished website preview with focused build stages, deterministic project bootstrap, environment readiness checks, and demo-ready seeded content.

## The Problem

Most AI website workflows break down in three ways:

1. **Everything happens in one long session** — planning, setup, install, page building, and validation all compete for the same context window.
2. **Framework bootstrap leaves residue** — starter files, temporary folders, or wrong metadata remain in the final project.
3. **The preview looks unfinished without assets** — users are asked for too much text and too many images before they can judge whether the website is actually good.

## The Solution

`site-flow` is a Claude Code plugin that acts like an AI web design consultant and build orchestrator. It:

1. **Discovers your needs** through simple, friendly questions
2. **Recommends a design direction** matched to your site type and audience
3. **Builds through focused sub-agent stages** so readiness checks, bootstrap, page work, and validation stay organized and isolated from the main session
4. **Uses deterministic bootstrap rules** so starter residue does not leak into the final project
5. **Seeds demo-ready text and real-image-first visual defaults** so the site looks close to finished before all real assets exist
6. **Requires design-rich output** with imagery, motion, hierarchy, and polish rather than scaffold-like placeholder pages
7. **Separates content from code** so real copy and images can replace demo content later

## Installation

```bash
/plugin marketplace add zetos-dev/site-flow
/plugin install site-flow@site-flow-marketplace
```

## Quick Start

```bash
# 1. Plan the website
/site-init

# 2. Build all pages
/site-build --all
# If this computer is missing required build tools, site-flow will pause,
# explain what to install, and let you resume afterward.

# 3. Preview in browser
/site-preview

# 4. Replace content later
/site-build --update

# 5. Check workflow status anytime
/site-status
```

## How It Works

### `/site-init` — Plan the Website

This command guides the user through a few simple questions:
- what kind of site they need
- what the brand/project does
- who the audience is
- what references or inspiration they like

Then it generates the planning system under `.site/`, including:
- project config
- workflow state
- site blueprint
- design tokens
- content guide
- page specs
- seeded demo content in `content/`

### `/site-build` — Orchestrate the Build

`/site-build` is a staged orchestrator, not a single monolithic coding pass.

It breaks work into:
1. environment readiness
2. discovery / plan check
3. bootstrap
4. page build loop
5. validation

The main session handles orchestration, readiness checks, installation/setup steps, and status reporting. Bootstrap, page implementation, content refreshes, design changes, and validation are attempted through focused sub-agents first, with graceful fallback in the current directory if agent launch is unavailable.

Options:
- `/site-build` — build the next pending page
- `/site-build --all` — build all remaining pages
- `/site-build about` — build one specific page
- `/site-build --update` — replace demo content with newer content from `content/`

### `/site-preview` — Review Quickly

Starts a local preview server, explains what is still demo content, highlights unfinished design debt if present, and helps the user make focused adjustments through sub-agents.

### `/site-status` — See Real Progress

Shows:
- current workflow stage
- environment readiness and missing tools
- built pages vs planned pages
- whether bootstrap residue checks passed
- whether validation passed
- how much of the site is using real content vs demo-ready content vs minimal placeholders
- what kind of imagery is currently being used (user, stock, AI, or placeholder)

## Environment Readiness

site-flow is designed for non-technical users. You do **not** need git or worktrees to use it.

Agent orchestration is an internal implementation detail. If helper-agent launch is unavailable, site-flow should continue in a compatible fallback mode instead of asking the user to configure git/worktree internals.

If the chosen site stack needs extra tools and your computer is missing them:
- site-flow checks first
- explains what is missing in plain language
- pauses cleanly instead of failing deep into the build
- lets you resume by running `/site-build` again after installation

For Astro projects, Node.js and npm are required.
For simple HTML projects, preview can sometimes fall back to a basic local server.

## Demo-Ready Content System

A core feature of `site-flow` is that the preview should still feel valuable before the user has collected every final asset.

### Seeded Text

When real copy is missing, the plugin generates polished, industry-appropriate demo content instead of lorem ipsum.

Examples of content profiles:
- enterprise-tech
- professional-services
- industrial-manufacturing
- finance-consulting
- healthcare-b2b
- luxury-brand

### Real-Image-First Defaults

The default goal is not to show a nice empty framework. It is to show a website that already feels close to finished.

Default image priority:
1. `real-user-assets`
2. `stock-library`
3. `ai-generated`
4. `designed-placeholder`

A practical hybrid is used for many sites:
- hero / key visual / case-study cover: AI-generated when it improves brand feel
- supporting sections: stock-library first
- designed placeholders: only as last fallback

A deliberate designed graphic can count as complete. A blank gradient slab, empty image frame, or generic box placeholder cannot.

### Designed Visual Placeholders

Examples:
- brand-gradient
- dashboard-mockup
- team-silhouette-panel
- abstract-office-scene
- device-frame
- metric-card-cluster
- logo-strip-placeholder
- case-study-cover-panel

### Motion Tokens

Design tokens can include a restrained motion system, such as:
- reveal-soft
- hover-lift
- border-glow
- stagger-cards
- panel-parallax-lite

## Bootstrap Philosophy

For Astro projects, the default strategy is **deterministic in-place bootstrap**.

That means:
- initialize in the final target root
- normalize starter output immediately
- do not rely on copy-then-delete flows
- do not leave cleanup for later stages

The validation step checks for residue such as:
- starter README files
- wrong package names
- helper scaffold folders like `.astro-starter/`
- untouched starter pages

## Content Management

Generated project content is organized by page:

```text
content/
├── README.md
├── 01-homepage/
│   ├── README.md
│   ├── hero-title.md
│   ├── hero-description.md
│   └── images/
├── 02-about/
│   ├── README.md
│   ├── introduction.md
│   └── images/
└── shared/
    ├── navigation.md
    └── footer.md
```

The content system distinguishes three states:
- `real`
- `seeded-demo`
- `placeholder-minimal`

That makes it easier to preview early, then progressively replace the most important content.

## Tech Stack

Chosen automatically:
- **1-2 pages**: HTML + Tailwind CSS
- **3+ pages**: Astro + Tailwind CSS

## Architecture

```text
site-flow/
├── plugins/site-flow/
│   ├── agents/
│   │   ├── bootstrap-astro.md
│   │   ├── bootstrap-html.md
│   │   ├── content-updater.md
│   │   ├── page-builder.md
│   │   └── site-validator.md
│   └── commands/
│       ├── site-init.md
│       ├── site-build.md
│       ├── site-preview.md
│       └── site-status.md
```

Generated project artifacts:

```text
.site/
├── config.json
├── workflow-state.json
├── site-blueprint.md
├── design-tokens.md
├── content-guide.md
├── bootstrap-report.md
├── validation-report.md
├── page-spec-{slug}.md
└── page-{slug}-completion-report.md
```

## Design Philosophy

**AI recommends, user confirms.**

The plugin makes technical decisions automatically, asks the user only about meaningful product/design choices, and aims to show a convincing website preview before all final assets are ready.

That preview should still look like a designed product: rich imagery, motion, hierarchy, and polished composition are expected. If the result still looks like a scaffold, wireframe, or placeholder shell, the workflow should treat that as unfinished work rather than acceptable output.

## License

MIT
