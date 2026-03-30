---
description: "Preview your website in the browser and make adjustments based on feedback"
argument-hint: ""
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion]
---

# /site-preview — Preview & Iterate

You are helping a non-technical user preview their website and make adjustments based on their feedback.

## Pre-check

1. Read `.site/config.json` — if missing, tell user to run `/site-init` first
2. Check that at least one page has been built (status is "built" or "building", or page completion reports exist)
3. If no pages built, tell user to run `/site-build` first

## Phase 1: Start Development Server

### For Astro projects:

```bash
npx astro dev
```

Run this in the background. Tell the user:

```
Your website is now running! Open your browser and go to:

  http://localhost:4321

Take a look around and let me know what you think. I can help you adjust:
  - Colors, fonts, or spacing
  - Layout of any section
  - Text or images
  - Animations and effects
  - Adding or removing sections

When you're done reviewing, just let me know!
```

### For HTML projects:

```bash
npx serve .
```

Or use Python's built-in server:

```bash
python3 -m http.server 8000
```

Run in the background and tell the user the URL.

## Phase 2: Collect Feedback

Use `AskUserQuestion` to gather structured feedback:

**Question 1 — Overall Impression**:
- Header: "First Look"
- Question: "How does the website look overall?"
- Options:
  - "Looks great! Just minor tweaks needed" — Small adjustments to colors, spacing, or text
  - "Good direction, but some sections need work" — Specific areas need redesign or restructuring
  - "Not quite right, needs significant changes" — Major design direction or layout changes needed
  - "Love it, no changes needed!" — Ready to move on

If user selects "Love it", skip to Phase 4.

**Question 2 — Specific Feedback** (if changes needed):
- Header: "What to Fix"
- multiSelect: true
- Question: "What would you like to adjust? (Select all that apply)"
- Options:
  - "Colors or visual style" — Change color scheme, contrast, or mood
  - "Layout or structure" — Move sections, change column layouts, resize areas
  - "Text content" — Update headlines, descriptions, or other text
  - "Images or visual elements" — Change backgrounds, add/remove images
- Allow "Other" for specific requests

Then ask a follow-up free-text question (via AskUserQuestion "Other" option or direct conversation):
- "Can you describe what you'd like changed? For example: 'Make the top section taller', 'Use darker colors', 'Move the contact info higher'"

## Phase 3: Apply Changes

Based on user feedback:

1. Read the relevant page file(s)
2. Read `.site/design-tokens.md` for current design system
3. Make the requested changes directly (for small changes) or dispatch a sub-agent (for larger changes)

### For small changes (color tweaks, text updates, spacing):
- Edit the files directly using the Edit tool
- These are quick fixes that don't need a sub-agent

### For larger changes (layout restructuring, new sections, style overhaul):
Dispatch a sub-agent with specific instructions:

```text
You are making design adjustments to the {page_name} page of the {project_name} website.

## Current Page File: {file_path}
## Design Tokens: {paste design-tokens.md}

## User Feedback:
{user's specific feedback}

## Changes Required:
{translated into specific technical changes}

## Rules:
- Preserve all existing content unless the user asked to change it
- Maintain responsive behavior
- Keep design token compliance
- Only change what was requested — do not "improve" other areas
```

After making changes, tell the user:

```
Changes applied! Refresh your browser to see the updates.

Want to adjust anything else, or does it look good?
```

### Feedback Loop

Repeat Phase 2-3 until the user is satisfied. Keep iterations quick and focused.

## Phase 4: Finalize

When the user is happy:

1. Update `.site/config.json`: set `status` to "reviewed"
2. If the dev server is still running, ask if they want to keep it running

Output:

```
Great, your website looks good!

Next steps:
  - To update content later, edit files in the content/ folder and run /site-build --update
  - Your website files are ready to publish to any web hosting service
  - Common options: Netlify, Vercel, GitHub Pages, or any static hosting

Need help with publishing? Just ask!
```

## Key Principles

- Keep feedback loops SHORT — make changes quickly, let user see results fast
- Don't over-interpret feedback — if user says "darker", make it darker. Don't redesign the whole page.
- Always preserve content the user didn't mention
- Use everyday language in all communication
