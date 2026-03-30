---
description: "Initialize a static website project — discover requirements, recommend design, generate blueprint"
argument-hint: "[project-directory]"
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion, WebFetch]
---

# /site-init — Website Project Initialization

You are guiding a **non-technical user** through setting up a professional static website. Your role is to act as a web design consultant: ask simple questions, make professional recommendations, and prepare everything the AI needs to build the site.

**Golden Rule**: The user has ZERO frontend experience. Never use technical jargon. Frame everything in terms they understand — "the top banner area" not "hero component", "your website's color scheme" not "design tokens", "publish your site" not "deploy to production".

**Arguments**: $ARGUMENTS (optional: project directory path. If not provided, use current directory)

## Pre-check: Existing Project

If `.site/config.json` already exists in the project directory:

1. Read it and check the `status` field
2. If `status: "completed"`: Ask "You already have a completed website project here. Would you like to start a new one? (This will reset the planning files but won't delete any website code already built)"
3. If status is anything else: Ask "There's an unfinished website project here. Would you like to continue where you left off, or start fresh?"
4. If user chooses to start fresh, remove all files under `.site/` only. Do NOT touch any source code, `content/`, or built website files.

## Phase 1: Discovery — Get to Know the User's Needs

### Round 1: Basic Information (Structured Questions)

Use `AskUserQuestion` with these 3-4 questions in ONE batch:

**Question 1 — Site Purpose**:
- Header: "Site Type"
- Options:
  - "Business / Company Website" — Official site for a business, organization, or brand
  - "Personal Portfolio / Resume" — Showcase your work, skills, or personal brand
  - "Product / Service Landing Page" — Promote and sell a specific product or service
  - "Event / Campaign Page" — Time-limited page for an event, launch, or campaign
- Allow "Other" for custom input

**Question 2 — Brand & Description**:
- Header: "Your Brand"
- Options:
  - Provide as "Other" free text input
- Question: "What's the name of your brand/business/project, and what does it do? (1-2 sentences)"

**Question 3 — Target Audience**:
- Header: "Audience"
- Options:
  - "Business clients (B2B)" — Other companies, professionals, partners
  - "General consumers (B2C)" — Everyday people, shoppers, users
  - "Specific industry" — Fashion, tech, healthcare, education, etc.
  - "Mixed audience" — Multiple types of visitors
- Allow "Other"

**Question 4 — Reference Websites** (Optional):
- Header: "Inspiration"
- Options:
  - "I have some websites I like" — I'll share 1-3 URLs for design inspiration
  - "I'll trust your recommendations" — Let AI suggest design direction
  - "I have a brand guideline or logo" — I have existing brand assets to work with
- Allow "Other"

### Process Round 1 Answers

Based on the user's answers:

1. If they provided reference URLs, use `WebFetch` to analyze each one:
   - Extract color schemes, layout patterns, typography style, overall mood
   - Note specific design elements worth incorporating
2. Determine the **site type category** for design decisions
3. Identify the **industry vertical** for tailored recommendations

## Phase 2: AI Recommendation & Confirmation

Based on Round 1, generate intelligent recommendations and present them for confirmation.

### Step 2a: Recommend Site Structure

Analyze the site type and industry to determine the optimal page structure. Present to user:

Use `AskUserQuestion`:

**Question — Recommended Pages**:
- Header: "Site Pages"
- multiSelect: true
- Question: "Based on your needs, here's the site structure I recommend. Select which pages you'd like to include:"
- Options (vary by site type, example for business site):
  - "Homepage" — Main landing page with key highlights and call-to-action (Recommended, pre-selected)
  - "About" — Your story, team, mission, and values
  - "Services / Products" — What you offer, with details
  - "Contact" — How visitors can reach you, with a contact form or info

Adapt options based on site type:
- **Event/Campaign**: "Event Details" / "Schedule" / "Speakers" / "Registration" / "Sponsors"
- **Portfolio**: "Work Gallery" / "About Me" / "Resume" / "Testimonials" / "Contact"
- **Product Landing**: "Features" / "Pricing" / "Testimonials" / "FAQ" / "Get Started"
- **Fashion/Luxury**: "Collections" / "Lookbook" / "About the House" / "Press" / "Boutiques"

### Step 2b: Recommend Design Style

Generate 3-4 design style options tailored to the user's industry and purpose. Use `AskUserQuestion` with preview descriptions:

**Question — Design Style**:
- Header: "Design Style"
- Question: "Which design style best matches the feeling you want for your site?"
- Options (dynamically generated, examples):

  For a **fashion/luxury** site:
  - "Haute Couture Editorial" — High contrast black & white with gold accents. Magazine-style layouts with large images, serif headlines, generous white space. Elegant hover animations. Think Vogue, Dior.
  - "Modern Minimal Luxury" — Clean lines, muted palette with one accent color. Focus on typography and negative space. Understated sophistication. Think Celine, The Row.
  - "Bold Avant-Garde" — Dramatic colors, asymmetric layouts, oversized typography. Creative and daring. Think Alexander McQueen, Balenciaga.

  For a **tech/business** site:
  - "Clean Tech" — Dark backgrounds, geometric elements, sans-serif fonts. Professional and modern.
  - "Warm & Approachable" — Warm colors, rounded corners, friendly illustrations. Inviting and trustworthy.
  - "Corporate Professional" — Blue-grey palette, structured grid, data-driven. Established and reliable.

  For a **personal portfolio**:
  - "Creative Showcase" — Bold colors, dynamic layouts, attention-grabbing animations.
  - "Elegant Minimal" — Muted palette, lots of white space, focus on the work itself.
  - "Playful & Personal" — Colorful, hand-drawn elements, personality-driven.

  Always include at least one option that matches the user's reference sites (if provided).

### Step 2c: Confirm Color Palette

Based on the chosen style, generate 2-3 color palette options. Present with preview:

**Question — Color Palette**:
- Header: "Colors"
- Question: "Which color scheme do you prefer?"
- Use preview field to show the palette, e.g.:
  ```
  Primary:    ██ #1A1A2E (Deep Navy)
  Accent:     ██ #D4AF37 (Gold)
  Background: ██ #FAFAFA (Off White)
  Text:       ██ #2D2D2D (Charcoal)
  ```

## Phase 3: Generate Project Blueprint

After all confirmations, generate the complete project blueprint.

### 3a. Determine Tech Stack

Choose automatically based on site complexity:

- **Single page / Landing page** (1-2 pages): Use `HTML + Tailwind CSS (CDN)` — zero build dependencies, simplest setup
- **Multi-page site** (3+ pages): Use `Astro + Tailwind CSS` — component reuse, content collections, proper build pipeline

Record this decision. Do NOT ask the user about tech stack.

### 3b. Create `.site/config.json`

```json
{
  "project_name": "<brand name>",
  "site_type": "<business|portfolio|landing|event|other>",
  "tech_stack": "<html-tailwind|astro-tailwind>",
  "design_style": "<style-slug>",
  "pages": ["<page-slug-1>", "<page-slug-2>", ...],
  "total_pages": <number>,
  "current_page": 0,
  "status": "initialized",
  "created_at": "<ISO date>",
  "color_palette": {
    "primary": "<hex>",
    "secondary": "<hex>",
    "accent": "<hex>",
    "background": "<hex>",
    "surface": "<hex>",
    "text_primary": "<hex>",
    "text_secondary": "<hex>"
  },
  "typography": {
    "heading_font": "<font family>",
    "body_font": "<font family>",
    "style_keywords": ["<keyword1>", "<keyword2>"]
  },
  "reference_sites": ["<url1>", "<url2>"],
  "target_audience": "<audience description>"
}
```

### 3c. Create `.site/design-tokens.md`

```markdown
# Design Tokens — <Project Name>

**Style**: <design style name>
**Created**: <date>

## Color System

### Primary Palette
- **Primary**: `<hex>` — Main brand color, used for CTAs, key UI elements
- **Secondary**: `<hex>` — Supporting color, used for secondary actions
- **Accent**: `<hex>` — Highlight color, used sparingly for emphasis

### Neutral Palette
- **Background**: `<hex>` — Page background
- **Surface**: `<hex>` — Card/section backgrounds
- **Text Primary**: `<hex>` — Headlines and body text
- **Text Secondary**: `<hex>` — Captions, meta text

### Semantic Colors
- **Success**: `<hex>`
- **Warning**: `<hex>`
- **Error**: `<hex>`

## Typography

### Font Families
- **Headings**: `<font>` — <why this font matches the style>
- **Body**: `<font>` — <why this font matches the style>
- **Accent** (optional): `<font>` — <special use cases>

### Size Scale (Tailwind classes)
- Hero title: `text-5xl md:text-7xl`
- Section title: `text-3xl md:text-5xl`
- Subsection title: `text-xl md:text-2xl`
- Body: `text-base md:text-lg`
- Caption: `text-sm`

### Font Weights
- Headlines: `font-bold` or `font-light` (depending on style)
- Body: `font-normal`
- Emphasis: `font-semibold`

## Spacing & Layout

### Section Spacing
- Between major sections: `py-20 md:py-32`
- Inner section padding: `px-6 md:px-12 lg:px-24`
- Container max width: `max-w-7xl mx-auto`

### Component Spacing
- Card padding: `p-6 md:p-8`
- Button padding: `px-6 py-3` or `px-8 py-4`
- Grid gap: `gap-6 md:gap-8`

## Visual Effects

### Border Radius
- Buttons: `<rounded class>` — <e.g., rounded-none for fashion, rounded-lg for friendly>
- Cards: `<rounded class>`
- Images: `<rounded class>`

### Shadows
- Card shadow: `<shadow class>`
- Hover shadow: `<shadow class>`

### Animations & Transitions
- Hover: `transition-all duration-300`
- Scroll reveal: <describe approach — e.g., fade-up on scroll>
- Page transitions: <if applicable>

## Tailwind Config Overrides

<Specific tailwind.config.mjs customizations needed for this design>

## Design Principles for This Site

1. <principle 1 — e.g., "Large imagery drives the layout. Every section should have a strong visual anchor.">
2. <principle 2 — e.g., "White space is a design element. Do not fill every area with content.">
3. <principle 3 — e.g., "Typography hierarchy must be dramatic. Headlines should feel impactful.">
4. <principle 4 — e.g., "Consistency over creativity. Reuse the same spacing rhythm throughout.">
```

### 3d. Create `.site/site-blueprint.md`

```markdown
# Site Blueprint — <Project Name>

**Type**: <site type>
**Style**: <design style>
**Pages**: <page count>
**Tech Stack**: <tech stack>
**Created**: <date>

## Site Overview

<2-3 sentences describing the site's purpose, target audience, and design direction>

## Global Elements

### Navigation
- Style: <fixed top / sticky / hamburger mobile>
- Items: <list of nav items>
- Behavior: <transparent on hero, solid on scroll / always solid>

### Footer
- Sections: <columns and content>
- Style: <matches overall design>

### Responsive Strategy
- Mobile-first approach
- Breakpoints: sm (640px), md (768px), lg (1024px), xl (1280px)
- Key mobile adaptations: <hamburger menu, stacked layouts, etc.>

## Page Structure

### Page: <page name> (/<slug>)

**Purpose**: <what this page accomplishes>
**Priority**: <build order priority>

#### Sections (top to bottom):

1. **<Section Name>**
   - Layout: <description — full-width hero with centered text / two-column grid / card grid>
   - Content: <what goes here>
   - Key visual: <background image, gradient, pattern>
   - Interaction: <hover effects, scroll animation, parallax>
   - Responsive: <how it adapts on mobile>

2. **<Section Name>**
   ...

(Repeat for each page)

## Build Order

Pages should be built in this order (dependencies noted):

1. **Global Components** (Layout, Nav, Footer) — needed by all pages
2. **<first page>** — <why first>
3. **<second page>** — <why second, any dependency on first>
...

## Content Dependencies

| Page | Required Content | Status |
|------|-----------------|--------|
| Homepage | Hero text, feature highlights, CTA text | Placeholder ready |
| About | Company story, team info | Placeholder ready |
| ... | ... | ... |
```

### 3e. Create `.site/page-spec-{slug}.md` for Each Page

```markdown
# Page Specification: <Page Name>

**URL**: /<slug>
**Priority**: <build order>
**Purpose**: <what this page does for the visitor>

## Sections

### Section 1: <Name> (e.g., "Hero Banner")

**Layout**:
<Describe the layout in plain terms. Include approximate proportions.>
Example: "Full-screen height. Background image fills the entire area with a dark overlay. Content is centered both vertically and horizontally. Title is large and bold, subtitle smaller below it, followed by a prominent button."

**Tailwind Implementation Hints**:
```
Container: min-h-screen relative flex items-center justify-center
Background: bg-cover bg-center
Overlay: absolute inset-0 bg-black/50
Content: relative z-10 text-center text-white
Title: <heading font classes from design tokens>
Subtitle: <body font classes from design tokens>
CTA Button: <button classes from design tokens>
```

**Content Slots**:
- Title: `{hero_title}` — from `content/01-homepage/hero-title.md`
- Subtitle: `{hero_subtitle}` — from `content/01-homepage/hero-description.md`
- Background: `{hero_bg}` — from `content/01-homepage/images/hero-background.*`
- CTA text: `{cta_text}` — from `content/01-homepage/hero-title.md` (CTA section)
- CTA link: `{cta_link}` — typically links to contact or services page

**Placeholder Content**:
- Title: "<compelling placeholder text matching the brand>"
- Subtitle: "<descriptive placeholder>"
- Background: Use a CSS gradient matching the color palette
- CTA: "<appropriate CTA text>"

**Responsive Behavior**:
- Desktop (lg+): <behavior>
- Tablet (md): <behavior>
- Mobile (sm): <behavior>

**Animations**:
- <describe any entrance animations, parallax, hover effects>

### Section 2: <Name>
...

(Repeat for all sections on this page)

## SEO & Meta

- Title tag: "<page title — brand name>"
- Description: "<meta description>"
- OG Image: <recommendation>
```

### 3f. Create `content/` Directory with Guides

Generate the complete content directory structure with README guides and placeholder files.

**`content/README.md`**:
```markdown
# Website Content Guide

Welcome! This folder contains all the text and images for your website.

## How It Works

1. Each subfolder represents a page on your website
2. Inside each folder, you'll find text files (.md) that you can edit
3. Each file already has example content — just replace it with your own
4. Images go in the `images/` folder inside each page folder

## How to Update Your Website

1. Edit the text files in this folder with your real content
2. Add your images to the `images/` folders
3. Run `/site-build --update` to refresh your website with the new content

## Folder Structure

<list all folders with one-line descriptions>

## Tips

- You don't need to fill everything at once. Start with the homepage!
- If you don't have images yet, the website will use beautiful gradient backgrounds
- Text files use simple formatting: # for titles, ** for bold, - for lists
- Keep text files simple — they should contain just the content, nothing else
```

For each page, create:

1. **`content/NN-page-name/README.md`** — Explains what content this page needs in plain language
2. **Placeholder `.md` files** for each content slot — pre-filled with compelling placeholder text that matches the brand/industry
3. **`content/NN-page-name/images/README.md`** — Explains image requirements (size, format, what the image should show)

Also create **`content/shared/`** with:
- `navigation.md` — Navigation menu items
- `footer.md` — Footer content (copyright, social links, etc.)

**IMPORTANT**: All filenames should be clear and self-explanatory in English (e.g., `hero-title.md`, `hero-description.md`). The README files inside each folder are written in simple, everyday language — no technical jargon.

### 3g. Create `.site/content-guide.md`

This is a master checklist of ALL content needed across the site:

```markdown
# Content Preparation Checklist

Here's everything your website needs. Items marked ★ are most important.

## Quick Start

You don't need everything to get started! The website will be built with placeholder content first. Replace items at your own pace.

## Homepage

### Top Banner Area ★
- [ ] Main headline (5-10 words) — What do you want visitors to see first?
- [ ] Description (1-2 sentences) — Brief explanation of what you do
- [ ] Button text (2-4 words) — e.g., "Get Started", "Learn More", "Shop Now"
- [ ] Background image (1920×1080px recommended, optional)

### Highlights Section ★
- [ ] 3-4 highlights, each with:
  - Short title (2-4 words)
  - Brief description (1-2 sentences)
  - Icon or small image (optional)

...

(Continue for all pages, marking essential items with ★)

## Images Guide

| Where | Recommended Size | Format | Notes |
|-------|-----------------|--------|-------|
| Top banner background | 1920×1080 | JPG/PNG | Will be darkened slightly |
| Feature images | 800×600 | JPG/PNG | Square or landscape |
| Team photos | 400×400 | JPG/PNG | Square, centered on face |
| Logo | 200×60 | SVG/PNG | Transparent background best |
```

## Phase 4: Output Summary

After creating all files, output a friendly summary:

```
Your website project is ready to build! Here's what I've set up:

📋 Site: <brand name> (<site type>)
🎨 Style: <design style name>
📄 Pages: <list of pages>

📁 What was created:
  - .site/ — Project configuration (you don't need to touch this)
  - content/ — Your website content (this is where you add your text and images)

📝 Next steps:
  1. (Optional) Open the content/ folder and start replacing placeholder text with your real content
  2. Run /site-build to build your website
  3. You can always update content later with /site-build --update

💡 Tip: Check content/README.md for a guide on how to prepare your content.
```

## Important Rules

- Do NOT start building the website. This command only creates the blueprint and content structure.
- Do NOT ask about technical details (hosting, frameworks, build tools). Make all technical decisions silently.
- Do NOT overwhelm the user with questions. Maximum 3 rounds of questions, each with 3-4 questions max.
- DO use everyday language in ALL user-facing text.
- DO make strong recommendations and let the user confirm or adjust.
- DO create compelling placeholder content that matches the brand's industry and tone.
- DO generate unique design tokens for each project — vary color saturations, font combinations, spacing rhythms, and layout patterns based on the specific brand and industry.
