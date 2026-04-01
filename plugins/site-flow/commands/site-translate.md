---
description: "Add one new language version to an existing website without changing layout or design"
argument-hint: "[language-code]"
allowed-tools: [Read, Write, Edit, Glob, Grep, AskUserQuestion, Agent]
---

# /site-flow:site-translate — Add One Language

You are helping a non-technical user add one new language version to an existing website.

## Goal

Add exactly one new language to the website by creating translated content files while preserving:
- layout
- structure
- styling
- component hierarchy
- design tokens

This command should not redesign pages or duplicate the site into separate template copies.

## Arguments

- No argument: ask which new language to add
- `<language-code>`: add that language, for example `/site-flow:site-translate en`

## Pre-check

1. Read `.site/config.json` — if missing, tell the user to run `/site-flow:site-init` first.
2. Read `.site/site-blueprint.md`
3. Read `.site/workflow-state.json` if present.
4. Check whether `i18n` exists in `.site/config.json`.
5. Detect the default language and existing language list.
6. Use Glob to inspect `content/` folders.

## Rules

- Add only one language at a time.
- Do not change page structure, design tokens, layout, or visual direction.
- Do not duplicate page source files just to create a new language.
- Prefer `content/{lang}/{page}/...` structure.
- Translate content only.
- Preserve email and booking/calendar integration intent across languages.
- If content cannot be translated confidently, mark it for review rather than inventing factual claims.
- Keep the project understandable for non-technical users.

## Language Selection

If no language code is provided, use `AskUserQuestion` to ask:
- Which language should be added next?

If the requested language already exists:
- tell the user it is already present
- do not overwrite it automatically

## Execution

1. Determine `default_language` from `.site/config.json`
2. Use the default language content as the source
3. Create the new language folder under `content/{target-language}/`
4. Recreate the same page-local content structure
5. Translate page-local and shared content into the new language
6. Update `.site/config.json` to append the new language
7. Create or update `.site/translation-status.json`

## Translation Status File

Use a structure like:

```json
{
  "default_language": "zh",
  "languages": {
    "zh": {
      "status": "ready"
    },
    "en": {
      "status": "draft",
      "source_language": "zh",
      "needs_review": true
    }
  }
}
```

## Output to the user

Explain in simple language:
- which language was added
- which files were created
- whether the new language is draft or ready
- what to review next
