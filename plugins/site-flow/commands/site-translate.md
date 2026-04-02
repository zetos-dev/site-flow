---
description: "Add one new language version to an existing website without changing layout or design"
argument-hint: "[language-code]"
allowed-tools: [Read, Write, Edit, Glob, Grep, AskUserQuestion, Agent]
---

# /site-flow:site-translate — Add One Language

You are helping a non-technical user add one new language version to an existing website.

## Goal

Add exactly one new language to the website by creating translated content files and completing the site-facing language addition while preserving:
- layout
- structure
- styling
- component hierarchy
- design tokens

This command should not redesign pages or duplicate the site into separate template copies.

## Arguments

- No argument: ask which new language to add
- `<language-code>`: add that language, for example `/site-flow:site-translate fr`

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
- Translate content only where source text is required, but complete the site-facing language support as well.
- Preserve email and booking integration intent across languages.
- If multilingual support is enabled, the site must expose a visible language selection path in shared UI and actual reachability for the new language pages.
- If content cannot be translated confidently, mark it for review rather than inventing factual claims.
- Keep the project understandable for non-technical users.

## Language Selection

If no language code is provided, use `AskUserQuestion` to ask:
- Which language should be added next?

If the requested language already exists:
- tell the user it is already present
- do not overwrite it automatically

## Git State Pre-flight

Before launching any sub-agent, silently ensure the repository has at least one commit. The Claude Code harness requires a resolvable `HEAD` to start sub-agents; without it, agent launch fails with `Failed to resolve base branch "HEAD": git rev-parse failed` before any fallback logic can execute.

Run `git rev-parse HEAD 2>&1` in the project directory. If it exits non-zero (no initial commit yet):
1. `git add -A`
2. `git commit -m "chore: initial project setup"`

This is internal plumbing only. Do not surface it to the user or treat it as a user step.

## Execution Policy

- Helper-agent startup must never depend on git metadata.
- If helper launch fails because the directory is not a repository, has no initial commit, has unresolved `HEAD`, lacks branch/history metadata, or cannot resolve worktree/base-branch information — including failures such as `Failed to resolve base branch "HEAD": git rev-parse failed` — continue in the current project directory as `main-session-fallback`.
- Do not surface git/worktree setup as a user prerequisite.

## Execution

1. Determine `default_language` from `.site/config.json`
2. Use the default language content as the source
3. Create the new language folder under `content/{target-language}/`
4. Recreate the same page-local content structure
5. Translate page-local and shared content into the new language
6. Update `.site/config.json` to append the new language
7. Create or update `.site/translation-status.json`
8. Ensure the built site has a visible language selector or switcher in shared UI when multilingual support is enabled
9. Ensure the new language has actual page reachability, not only translated content files
10. Tell the user how to preview or rebuild the site so the new language becomes visible in the generated output

## Translation Status File

Use a structure like:

```json
{
  "default_language": "en",
  "languages": {
    "en": {
      "status": "ready"
    },
    "fr": {
      "status": "draft",
      "source_language": "en",
      "needs_review": true
    }
  }
}
```

## Output to the user

Explain in simple language:
- which language was added
- which files were created or updated
- whether the new language is draft or ready
- whether the site now has a visible language selector/switcher and reachable pages for that language
- whether they should rerun `/site-flow:site-build --update` or `/site-flow:site-preview` to see the new language live
- what to review next
