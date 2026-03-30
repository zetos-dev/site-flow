# site-flow

AI-driven static website workflow for Claude Code. It helps non-technical users go from idea to a polished website preview with staged sub-agents, deterministic project bootstrap, and demo-ready seeded content.

## The Problem

Most AI website workflows break down in three ways:

1. **Everything happens in one long session** — planning, setup, install, page building, and validation all compete for the same context window.
2. **Framework bootstrap leaves residue** — starter files, temporary folders, or wrong metadata remain in the final project.
3. **The preview looks unfinished without assets** — users are asked for too much text and too many images before they can judge whether the website is actually good.

## The Solution

`site-flow` is a Claude Code plugin that acts like an AI web design consultant and build orchestrator. It:

1. **Discovers your needs** through simple, friendly questions
2. **Recommends a design direction** matched to your site type and audience
3. **Builds through staged sub-agents** so setup, bootstrap, page work, and validation are isolated
4. **Uses deterministic bootstrap rules** so starter residue does not leak into the final project
5. **Seeds demo-ready text and visual placeholders** so the site looks close to finished before all real assets exist
6. **Separates content from code** so real copy and images can replace demo content later

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
1. discovery / plan check
2. environment setup agent
3. bootstrap agent
4. page builder agent (one page at a time)
5. validation agent

This keeps the main session lighter and makes failures easier to isolate.

Options:
- `/site-build` — build the next pending page
- `/site-build --all` — build all remaining pages
- `/site-build about` — build one specific page
- `/site-build --update` — replace demo content with newer content from `content/`

### `/site-preview` — Review Quickly

Starts a local preview server, explains what is still demo content, and helps the user make focused adjustments.

### `/site-status` — See Real Progress

Shows:
- current workflow stage
- built pages vs planned pages
- whether bootstrap residue checks passed
- whether validation passed
- how much of the site is using real content vs demo-ready content vs minimal placeholders

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

### Designed Visual Placeholders

When real images are missing, the workflow prefers reusable visual archetypes over weak empty boxes.

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
│   │   └── page-builder.md
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

## License

MIT
