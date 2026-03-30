# site-flow

AI-driven static website development workflow for Claude Code. Guides non-technical users from idea to professional website — no coding experience required.

## The Problem

Building a professional-looking website requires knowledge of HTML, CSS, frameworks, responsive design, and visual design principles. Non-technical users can't easily create sites that meet industry standards, and AI-generated sites often look generic and cookie-cutter.

## The Solution

`site-flow` is a Claude Code plugin that acts as an AI web design consultant. It:

1. **Discovers your needs** through simple, friendly questions (not technical jargon)
2. **Recommends professional design** tailored to your industry and brand
3. **Builds each page** with proper design tokens ensuring visual consistency
4. **Lets you preview and iterate** with quick feedback loops
5. **Separates content from code** so you can update text and images without touching code

## Installation

```bash
# Add the marketplace
/plugin marketplace add zetos-dev/site-flow

# Install the plugin
/plugin install site-flow@site-flow-marketplace
```

## Quick Start

```bash
# 1. Start your website project
/site-init

# 2. Build the website
/site-build --all

# 3. Preview in browser
/site-preview

# 4. Update content anytime
# Edit files in content/ folder, then:
/site-build --update

# Check progress anytime
/site-status
```

## How It Works

### `/site-init` — Set Up Your Project

Walks you through a few simple questions:
- What kind of site do you need?
- What's your brand name and what do you do?
- Who is your target audience?
- Any websites you like for inspiration?

Then AI automatically:
- Recommends page structure (you confirm)
- Recommends design style (you pick from 3-4 options)
- Recommends color scheme (you pick)
- Creates a `content/` folder with clear instructions for what content to prepare

### `/site-build` — Build Your Website

Builds each page with professional design quality:
- Pages are built sequentially for consistency
- Each page follows the design system exactly
- Responsive design (looks good on phones, tablets, desktops)

Options:
- `/site-build` — Build the next page
- `/site-build --all` — Build all pages
- `/site-build about` — Build a specific page
- `/site-build --update` — Update pages with your latest content

### `/site-preview` — See Your Website

- Opens your website in the browser
- Collects your feedback through simple questions
- Makes adjustments until you're happy

### `/site-status` — Check Progress

Shows a clear dashboard of:
- Which pages are built
- Which content is real vs placeholder
- What to do next

## Content Management

The `content/` folder is organized by page with Chinese filenames for easy understanding:

```
content/
├── README.md                    # How to manage your content
├── 01-homepage/
│   ├── README.md                # What this page needs
│   ├── hero-title.md            # Main headline
│   ├── hero-description.md      # Description text
│   └── images/                  # Page images
├── 02-about/
│   ├── README.md
│   ├── introduction.md
│   └── images/
└── shared/
    ├── navigation.md            # Navigation menu
    └── footer.md                # Footer content
```

Each folder has a README explaining exactly what content to prepare. Files come pre-filled with placeholder text — just replace with your own.

## Tech Stack

Automatically chosen based on your site:
- **Simple sites (1-2 pages)**: HTML + Tailwind CSS — zero build dependencies
- **Multi-page sites (3+)**: Astro + Tailwind CSS — component reuse, content collections

You never need to think about this — it's handled automatically.

## Design Quality

Every site gets a unique design through:
- **Industry-aware style recommendations** — fashion sites look different from tech sites
- **Custom design tokens** — unique color palettes, typography, spacing for each project
- **Multiple layout variations** — same section type, different visual approaches
- **Reference site analysis** — if you provide inspiration URLs, the AI incorporates their design language

## Architecture

```
site-flow/
├── plugins/site-flow/
│   ├── agents/
│   │   └── page-builder.md      # Sub-agent for building individual pages
│   └── commands/
│       ├── site-init.md          # Requirements discovery & blueprint
│       ├── site-build.md         # Build orchestrator
│       ├── site-preview.md       # Preview & feedback iteration
│       └── site-status.md        # Progress dashboard
```

Generated project files:
```
.site/                           # Project configuration (auto-managed)
├── config.json
├── site-blueprint.md
├── design-tokens.md
├── content-guide.md
├── page-spec-{slug}.md
└── page-{slug}-completion-report.md

content/                         # Your content (you manage this)
├── 01-homepage/
├── 02-about/
└── ...
```

## Design Philosophy

**"AI Recommends, User Confirms"** — Unlike developer-focused tools that ask technical questions, site-flow makes all technical decisions automatically and only asks the user about things they can meaningfully weigh in on: purpose, audience, visual style, and content.

## License

MIT
