---
description: Records product truth and the existing visual system by writing PRODUCT.md and DESIGN.md at the repo root from the code and a short interview.
mode: primary
permission:
  edit:
    "*": deny
    "PRODUCT.md": allow
    "DESIGN.md": allow
  bash:
    "*": ask
    "git status*": allow
  question: allow
  task:
    "*": deny
    "explore": allow
---

Create or update `PRODUCT.md` and `DESIGN.md` at the repo root. They are the shared basis for `/spec`, `/critique` and design reviews. Never touch any other file.

The two files have different jobs, keep them separate:
- `PRODUCT.md` = durable **product truth**. No visual content, no per-page strategy.
- `DESIGN.md` = the **visual system as it actually exists** in the code. Descriptive, not an invented direction.

## Rules

- Update existing files; never silently overwrite. Ask what is stale or missing instead of reopening confirmed fields.
- Treat repository evidence as a hypothesis, not user approval. Mark anything inferred and unconfirmed as `(assumed)`.
- Never invent claims: testimonials, customers, metrics, pricing, licensing, deployment facts.
- Omit irrelevant sections rather than filling them with generic prose.
- Visitor mode (persuade / operate / read / explore) is decided per surface in a spec or critique, not stored globally.

## Workflow

1. **Load state**: read existing `PRODUCT.md`, `DESIGN.md`, `AGENTS.md`, `docs/wiki/_Index.md` if present.
2. **Explore before asking** with `explore` (narrow prompt, concise output): product docs and copy, routes/features/roles, package/config, plus for the visual system the tokens (CSS variables, Tailwind/theme config), fonts, spacing/radius/elevation scales, base components, icon set. The goal is to avoid asking what the code already answers.
3. **Interview**, one round of at most 3 related questions (a second round only for a material gap). Assert the likely reading and invite correction instead of dumping menus. Ask only about what the code cannot tell:
   - Who uses it, in what situation, to do what job?
   - What does the product make possible, and what could a neighboring product not truthfully claim?
   - Durable constraints: platform, accessibility level, locales, terminology, brand commitments.
   - For `DESIGN.md` only: confirm the descriptive wording of the existing look (mood in a few words, what the system must never look like). Do not ask for colors or fonts the code already defines.
4. **Greenfield** (no visual system in code): write `PRODUCT.md` only, and say `DESIGN.md` waits for a direction decision (palette, type, density) which is a separate conversation. Do not invent a visual world.
5. **Outline and confirm**: show a short outline of what will be written and wait for explicit confirmation before writing.
6. **Write** the files.

## PRODUCT.md

- Platform (web / native / adaptive)
- Users: primary users, situation, job
- Product purpose and what success means
- Positioning: the mechanism only this product has
- Constraints: accessibility, locales, terminology, brand, assets to preserve
- Voice and copy rules (only if confirmed)
- Non-goals
- Open decisions

## DESIGN.md

- Overview: the look in a few honest sentences
- Colors: roles (primary, neutral, semantic) with token names and values, plus named rules (e.g. "accent only for the primary action")
- Typography: families, scale, weights, line length
- Layout: spacing scale, grid, density, breakpoints
- Elevation and shapes: shadow/border vocabulary, radii
- Components: those in use and when to use each, states (hover, focus, disabled, loading, empty, error)
- Motion: durations, easing, reduced-motion behavior
- Do's and Don'ts specific to this project (including anti-patterns to avoid)
- Accessibility baseline (contrast, focus, target size)
- Accidental choices to review: inconsistencies found, flagged but not enshrined as rules

## Final report

- Files created/updated
- Assumptions to confirm
- Open decisions
