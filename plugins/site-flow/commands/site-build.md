---
description: "Build the website pages using sub-agents, or update content from the content/ directory"
argument-hint: "[page-name | --update | --all]"
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion]
---

# /site-build — Website Build Orchestrator

You are the **orchestrator** for building a static website. You dispatch sub-agents to build each page, validate results, and coordinate overall progress.

**Arguments**: $ARGUMENTS
- No argument: Build the next pending page
- `<page-name>`: Build a specific page (e.g., `/site-build about`)
- `--update`: Re-read content from `content/` directory and update all built pages with real content
- `--all`: Build all pending pages sequentially

## Pre-check

1. Read `.site/config.json` — if missing, tell user: "No website project found. Run `/site-init` first to set up your project."
2. Read `.site/site-blueprint.md` for overall structure
3. Read `.site/design-tokens.md` for design system

## Mode: --update (Content Update)

If `--update` flag is present:

1. Scan the `content/` directory for all user-modified files
2. Compare with placeholder content — identify what the user has changed
3. For each page with updated content:
   - Read the corresponding page source file(s)
   - Read the updated content files from `content/NN-page-name/`
   - Dispatch a sub-agent to update the page with real content
   - The sub-agent should ONLY replace content — do NOT change layout, design, or structure
4. Run build verification
5. Output summary of what was updated

**Sub-agent prompt for content update**:
```text
You are updating content on an existing website page.

## Rules
- ONLY replace placeholder text and images with the real content provided
- Do NOT change any layout, styling, structure, or design
- Do NOT add new sections or remove existing ones
- Preserve all CSS classes, HTML structure, and component architecture
- If a content file is empty or unchanged from placeholder, skip it

## Page to update: {page_name}
## Page file(s): {file_paths}
## Content source: {content_directory_path}

## Content mapping:
{mapping of content files to page sections}

## Design tokens (for reference only — do not change):
{paste relevant design tokens}
```

Skip to "Build Verification" section after content updates are done.

## Mode: Regular Build

### Phase 1: Project Initialization (First Build Only)

Check if the website project has been initialized (e.g., `package.json` or `index.html` exists in the project root or `src/` directory).

If NOT initialized:

#### For Astro + Tailwind projects (`tech_stack: "astro-tailwind"` in config):

Dispatch an initialization sub-agent:

```text
You are setting up a new Astro website project with Tailwind CSS.

## Project: {project_name}
## Directory: {project_directory}

## Setup Steps

1. Initialize Astro project:
   - Run `npm create astro@latest . -- --template minimal --no-install --no-git --typescript relaxed`
   - Run `npx astro add tailwind --yes`
   - Run `npm install`

2. Configure Tailwind with custom design tokens:
   {paste complete design-tokens.md — colors, typography, spacing}

3. Create base layout component at `src/layouts/BaseLayout.astro`:
   - HTML document structure with lang, meta tags, font imports
   - Responsive viewport meta
   - Global styles
   - Slot for page content

4. Create shared components:
   - `src/components/Header.astro` — Navigation bar
     - Navigation items: {nav items from blueprint}
     - Style: {nav style from blueprint — fixed/sticky, transparent/solid}
     - Mobile: hamburger menu with slide-out panel
   - `src/components/Footer.astro` — Footer
     - Content: {footer content from blueprint}
     - Style: {footer style from blueprint}

5. Create content collection schema (if applicable):
   - Define content types in `src/content/config.ts`

## Design Tokens (MUST apply these exactly):
{paste FULL design-tokens.md content}

## Nav Items:
{from content/shared/navigation.md or blueprint}

## Footer Content:
{from content/shared/footer.md or blueprint}

## Key Constraints:
- All text must support both Chinese and English characters
- Must be fully responsive (mobile-first)
- Use Google Fonts for custom typography — include proper font imports
- Follow Tailwind best practices — utility-first, no custom CSS unless absolutely necessary
```

#### For HTML + Tailwind projects (`tech_stack: "html-tailwind"` in config):

Dispatch an initialization sub-agent:

```text
You are setting up a simple static website with HTML and Tailwind CSS via CDN.

## Project: {project_name}
## Directory: {project_directory}

## Setup Steps

1. Create project structure:
   ```
   /
   ├── index.html
   ├── css/
   │   └── custom.css (minimal, only for things Tailwind can't do)
   ├── js/
   │   └── main.js (minimal, only for interactive elements)
   └── images/
   ```

2. Create base HTML template with:
   - Tailwind CSS CDN with custom config script
   - Google Fonts import
   - Responsive viewport meta
   - Navigation header (shared across pages if multi-page)
   - Footer (shared across pages)

## Design Tokens (inject into Tailwind config):
{paste FULL design-tokens.md}

## Key Constraints:
- Use Tailwind Play CDN with inline config for custom colors/fonts
- No build step required — pure static files
- Mobile-first responsive design
```

### Phase 2: Page Build Loop

Determine which pages to build based on the argument and build order from `.site/site-blueprint.md`.

**CRITICAL: Build pages sequentially, NOT in parallel.** Later pages may reuse components or patterns established in earlier pages.

For each page to build:

#### Step 2.1: Prepare Page Context

1. Read `.site/page-spec-{slug}.md` for detailed page specification
2. Read `.site/design-tokens.md` for design system
3. Check `content/{NN}-{page-name}/` for any user-provided real content
4. If this is NOT the first page, scan existing pages for established patterns:
   - Use Glob to find component files
   - Read 1-2 existing page files to extract coding patterns
5. Read previous page completion reports (if any) for context

#### Step 2.2: Construct Page Builder Prompt

**THIS IS THE MOST CRITICAL STEP.** The sub-agent's output quality is directly proportional to prompt quality.

Build the prompt using this template:

```text
You are building a page for the {project_name} website.

## Project Context

**Project**: {project_name}
**Tech Stack**: {tech_stack}
**Design Style**: {design_style_description}
**Your Task**: Build the "{page_name}" page

## MANDATORY: Read These Documents First

Read these files IN ORDER before writing any code:

1. `.site/page-spec-{slug}.md` — Your detailed page specification
2. `.site/design-tokens.md` — Design system (colors, fonts, spacing)
3. `.site/site-blueprint.md` — Overall site structure

{if existing_pages}
4. Study these existing files for coding patterns:
   {list of existing page/component files with explanations}
{endif}

## Design Tokens (COMPLETE — use these exactly)

{paste FULL design-tokens.md content — do NOT just reference the file}

## Page Specification (COMPLETE)

{paste FULL page-spec-{slug}.md content}

## Content to Use

{For each section, specify:}
- If user has provided real content in content/{NN}-{page}/: paste it
- If no real content: paste the placeholder content from page-spec

## Content File Mapping

{Map each content slot to its source file in content/ directory, so future --update can find them}

## Implementation Requirements

1. **Responsive Design**: Mobile-first. Test mental model at 375px, 768px, 1024px, 1440px widths.
2. **Accessibility**: Proper heading hierarchy, alt text for images, sufficient color contrast.
3. **Performance**: Lazy-load images below the fold. Use appropriate image formats.
4. **Animations**: {specific animation requirements from page-spec}
5. **Typography**: Use exact font families and sizes from design tokens. Import Google Fonts if not already imported.

## Coding Standards

{For Astro projects:}
- Create the page at `src/pages/{slug}.astro`
- Extract reusable sections as components in `src/components/`
- Use the BaseLayout component
- Use Astro's built-in features (no unnecessary JavaScript)

{For HTML projects:}
- Create `{slug}.html` in the project root
- Reuse the HTML template structure from index.html
- Keep JavaScript minimal — only for essential interactions (mobile menu, scroll animations)

## Visual Quality Bar

- The page MUST look like a professionally designed website
- Use proper visual hierarchy — clear H1 → H2 → body text progression
- Ensure adequate white space between sections
- Images/gradients should feel intentional, not decorative afterthoughts
- Interactive elements (buttons, links, cards) must have hover states
- The design should feel cohesive with other pages on the site

## Self-Review Checklist

Before finishing, verify:
- [ ] All sections from page-spec are implemented
- [ ] Design tokens are applied correctly (colors, fonts, spacing)
- [ ] Page is responsive at all breakpoints
- [ ] Navigation links work correctly
- [ ] Content slots match the content file mapping
- [ ] No placeholder "lorem ipsum" text — use meaningful placeholder content
- [ ] Hover states and animations work
- [ ] Page follows the same patterns as existing pages (if any)

## After Completion

Create `.site/page-{slug}-completion-report.md` with:
- Files created/modified
- Sections implemented
- Content sources used (real vs placeholder)
- Any deviations from spec and why
- Recommendations for content the user should prioritize replacing
```

#### Step 2.3: Dispatch Page Builder Sub-Agent

```text
Agent(
  prompt: <constructed page builder prompt>,
  description: "Build page: {page_name}",
  model: "sonnet"
)
```

Use `model: "sonnet"` by default. Use `"opus"` only for the homepage (most complex) or after a sonnet attempt fails.

#### Step 2.4: Validate Page Build

After sub-agent returns:

1. Read the generated page file(s) — verify they exist
2. Check that page uses design tokens correctly (spot-check a few Tailwind classes)
3. Verify responsive classes are present (look for `md:` and `lg:` prefixes)
4. Read `.site/page-{slug}-completion-report.md`
5. Update `.site/config.json`: increment `current_page`

#### Step 2.5: Handle Issues

- **Minor issues** (missing hover state, wrong color): Fix directly
- **Major issues** (broken layout, missing sections): Re-dispatch with more specific prompt
- **Missing dependencies** (component not found): Create the component and re-dispatch

#### Step 2.6: Report Progress

Output a friendly progress update:

```
✓ {Page Name} page is ready!
  - Sections: {list of sections built}
  - Content: {real / placeholder}
  - Files: {list of files created}

{if more pages remain}
Next up: {next page name}
Continue? (y/n)
{endif}
```

### Phase 3: Build Verification

After all pages are built (or after --update):

1. For Astro projects: Run `npx astro build` and check for errors
2. For HTML projects: Verify all internal links point to existing files
3. Fix any build errors

### Phase 4: Completion

When all pages are built:

1. Update `.site/config.json`: set `status` to "built"
2. Output completion summary:

```
Your website is ready! 🎉

Pages built:
  ✓ Homepage
  ✓ About
  ✓ Services
  ✓ Contact

Content status:
  - 3 pages using placeholder content
  - 1 page with your real content

Next steps:
  1. Run /site-preview to see your website in the browser
  2. Update content in the content/ folder, then run /site-build --update
  3. When you're happy with everything, your site is ready to publish!
```

## Key Principles

- Each page-builder sub-agent gets clean context — this is by design
- The prompt IS the sub-agent's entire knowledge — include everything it needs
- If a page build fails, make the prompt more specific
- Always paste full design tokens in the prompt — never just reference the file
- Maintain visual consistency across pages by sharing pattern references
- Build global components (nav, footer, layout) FIRST before any pages
