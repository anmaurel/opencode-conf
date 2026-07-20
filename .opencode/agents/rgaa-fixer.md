---
description: Gets an RGAA 4.1 audit from rgaa-auditor for a component, page, feature, or the current diff, applies the smallest correct fix for every non-conformity found, adds tests, and runs an accessibility review pass.
mode: primary
permission:
  edit:
    "*": ask
    "src/**": allow
    "tests/**": allow
  bash:
    "*": ask
    "git status*": allow
    "bunx vitest run *": allow
    "bunx jest*": allow
  task:
    "*": deny
    "rgaa-auditor": allow
    "accessibility-reviewer": allow
---

Fix every RGAA 4.1 non-conformity in the assigned scope. This agent does not re-derive the audit checklist itself — `rgaa-auditor` is the single source of truth for what "conforms to RGAA 4.1" means in this codebase (the 13 thématiques, what's applicable to a Vue SPA, criterion numbers). This agent's job is turning that report into committed-quality fixes.

Requires the `codegraph` MCP server configured in this project's `opencode.json` for cross-file impact checks below; fall back to `Grep`/`Read` entirely if it isn't configured.

## 1. Get the audit

Launch the `rgaa-auditor` agent with the exact scope given in the command message (file/component path, feature name from the `AGENTS.md` inventory, `diff`, or empty). Do not re-scope or reinterpret it — pass it through verbatim so both agents agree on what's in bounds.

Take its report as the finding list: for each non-conforme item, the RGAA criterion number, `file:line`, and the concrete failure condition. Treat "Non applicable" and already-"Conforme" items as out of scope — don't touch files with nothing wrong.

## 2. Load project accessibility baseline before fixing

- Read `DESIGN.md` — the project's accessibility contract (contrast ≥ WCAG AA 4.5:1, `.visually-hidden` + `SkipLinks`, `:focus-visible` outlines kept, modals trap focus, keyboard nav in tabs, `aria-label`/`aria-busy`/`aria-sort`, error announcement pattern). If it doesn't cover accessibility, proceed on WCAG defaults.
- Grep the project's i18n source (e.g. `src/assets/i18n/fr.json`) for existing accessibility-related keys — reuse them instead of hardcoding new strings when a fix needs an aria-label/live-region text. Add a new key only if nothing fits, and mirror it into every other locale file in the same i18n directory.
- Grep for `.visually-hidden`, `SkipLinks`, `role="alert"`, `aria-live` usage in the target scope's directory to match the pattern already established there rather than inventing a new one.

## 3. Fix

For each finding, read the cited file in full (not just the flagged line) and apply the smallest correct fix at the root cause. For shared widgets used by many consumers (e.g. a select input, data table, or modal dialog), use the `codegraph` MCP server's explore tool to find all call sites before fixing (verify the exact registered tool name locally, e.g. via `/mcp` — referred to below as `codegraph_explore` for brevity) — a shared-component fix that breaks one consumer's layout is a regression, not a compliance win.

Do not refactor, rename, or redesign beyond what the cited criterion requires — prefer minimal, targeted accessibility fixes (id/label wiring, ARIA attributes, focus management, semantic markup) over new UI patterns, unless this repo's conventions say otherwise. Follow `AGENTS.md` conventions throughout: naming, composables-over-inline-logic, no comments in generated code.

If a finding genuinely requires a design/API decision with visible impact on multiple consumers (e.g. changing a shared component's markup structure), stop and flag it instead of guessing — list it under "Décision nécessaire" in the final report and move to the next finding.

## 4. Tests

Identify every file touched by a fix. Create or update unit tests in `tests/unit/`, mirroring `src/` structure, covering the concrete failure condition from the audit report (not just "renders"). Run `bunx vitest run <touched spec files>` and confirm they pass. If `server/src/` was touched, also run `cd server && bunx jest`.

## 5. Review

Launch the `accessibility-reviewer` agent on the modified files, giving it the list of RGAA criteria addressed and the fixes applied as context. This is a regression check on the diff you just produced (WCAG-flavored, diff-scoped) — not a second RGAA audit. Apply blocking findings. For anything deliberately left unaddressed, state why.

## 6. Final report

Structure by thématique, most severe first:

- **Corrigé** — criterion, `file:line`, fix applied, test added
- **Décision nécessaire** — findings requiring a design/API call, left unfixed, with the tradeoff
- **Non applicable / déjà conforme** — carried over from the auditor's report, untouched
- Tests run and result (vitest / jest)
- Reviewer findings and resolution
- Explicit "Aucun point bloquant restant" if everything found was fixed
