# AGENTS.md

This repository is a Hugo blog built on top of the PaperMod theme.

These instructions apply to any AI coding agent or assistant working in this codebase.

## Stack And Architecture

- Static site generator: Hugo
- Theme: PaperMod
- Theme source lives in `themes/PaperMod` as a Git submodule
- Project-specific customizations live outside the theme through Hugo overrides

## Core Rules

- Do not edit `themes/PaperMod` directly unless the user explicitly asks for a theme patch.
- Prefer Hugo overrides in `layouts/`, `layouts/partials/`, `assets/css/extended/`, and `static/`.
- Treat `public/` and `resources/` as generated output. Do not hand-edit them.
- Keep the site compatible with PaperMod conventions instead of replacing theme behavior unnecessarily.
- Preserve dark-theme behavior unless the user explicitly asks to redesign it.

## Preferred Customization Strategy

When changing the UI, use this order of preference:

1. Update Hugo content or front matter if the change is content-driven.
2. Add or adjust an override in `layouts/` or `layouts/partials/`.
3. Add scoped styles in `assets/css/extended/`.
4. Only override a full PaperMod template if a partial or smaller override is not enough.

## Theme Safety Rules

- Never move project-specific code into `themes/PaperMod`.
- Do not copy large PaperMod templates into the project unless it is required.
- If a template must be overridden, keep the override as small as possible.
- Reuse partials to avoid duplicating markup across list, taxonomy, and section pages.
- Favor Hugo partials over repeated inline template blocks.

## Layout And Template Guidelines

- Prefer reusable partials for shared UI patterns such as heroes, cards, filters, and search.
- Keep page templates declarative and thin. Put repeated rendering logic into partials.
- Avoid coupling JavaScript to fragile DOM relationships like `parentElement` chains.
- Prefer `data-*` attributes for behavior hooks instead of depending on CSS class names alone.
- Keep Hugo template logic readable. Use small variables for derived state instead of deeply nested inline expressions.

## CSS Guidelines

- Scope styles to the component or page being changed.
- Avoid broad global selectors that can leak into unrelated pages.
- Remember that PaperMod concatenates all files in `assets/css/extended/*.css`; every new file is global by default.
- Prefer small, purpose-specific CSS files over adding everything into one large stylesheet.
- Reuse shared component classes for common UI patterns such as buttons.
- Validate both desktop and mobile layouts after changing spacing, widths, grids, or sticky elements.
- Validate dark theme after any color, border, background, or card change.

## JavaScript Guidelines

- Keep JavaScript minimal; prefer Hugo and HTML-first solutions when possible.
- If behavior is needed, isolate it in a reusable partial or asset instead of embedding duplicated scripts.
- Behavior should survive moderate layout refactors without needing DOM rewrites.
- Avoid introducing frameworks or build complexity unless explicitly requested.

## Content Guidelines

- Blog posts live under `content/posts/`.
- Use clear front matter for `title`, `description`, `summary`, `tags`, `categories`, `cover`, and `image` when needed.
- Keep category and taxonomy naming consistent; avoid accidental duplicates caused by capitalization or spelling drift.
- Prefer concise descriptions and summaries because they affect cards, metadata, and sharing output.

## File Placement Guidelines

- Shared templates: `layouts/partials/`
- Section or page overrides: `layouts/<section>/` or `layouts/_default/`
- Shared component styles: `assets/css/extended/`
- Static images and assets: `static/`
- Theme code: `themes/PaperMod/` only when explicitly requested

## Validation Checklist

After making changes, run the following when relevant:

- `hugo`

Then verify:

- The site builds without template errors
- The affected page renders correctly
- Mobile layout still works
- Dark theme still works
- Search, filters, and navigation still work if the changed page uses them

## Current Project Conventions

- Prefer overrides over theme edits
- Prefer reusable partials over duplicated markup
- Prefer isolated component styles over page-coupled shared classes
- Keep the implementation simple and maintainable for future Hugo/PaperMod updates

## If Unsure

- Choose the solution that minimizes coupling to PaperMod internals
- Choose the smallest override that solves the problem
- Ask before making theme-submodule edits or large structural rewrites
