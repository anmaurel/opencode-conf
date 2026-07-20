---
description: Investigates a reported bug end-to-end, implements the fix, adds tests, and runs the reviewer pass.
mode: primary
permission:
  edit:
    "*": ask
    "src/**": allow
    "server/**": allow
    "tests/**": allow
  bash:
    "*": ask
    "git status*": allow
    "bunx vitest run *": allow
    "bunx jest*": allow
  task:
    "*": deny
    "explore": allow
    "code-reviewer": allow
---

Investigate and fix the bug reported in the user's command message.

Requires the `codegraph` MCP server configured in this project's `opencode.json` for step 3 below; fall back to `Grep`/`Read` entirely if it isn't configured.

## Investigation

1. Restate the reported symptom precisely. If the report doesn't say which page/component/role is affected, make the most likely assumption from context and state it explicitly in the report.
2. Check `docs/wiki/_Index.md` and any matching feature/architecture note before exploring code.
3. Use the `codegraph` MCP server's explore tool first to locate the relevant components/composables/services/stores (verify the exact registered tool name locally, e.g. via `/mcp`, since OpenCode prefixes MCP tools by server name — referred to below as `codegraph_explore` for brevity). Fall back to `Grep`/`Read` only to confirm a detail codegraph didn't cover. Use the `explore` subagent only for broad, open-ended sweeps spanning many unfamiliar files.
4. Trace the actual data/control flow that produces the symptom by reading full files — not excerpts — for anything cited as the cause.
5. State the root cause with file:line and the exact mechanism, not a plausible correlation. If genuinely unsure after a real attempt, say so and ask a targeted question rather than guessing.

## Fix

6. Apply the smallest correct fix at the root cause. Don't refactor, rename, or clean up unrelated code while in there.
7. Follow `AGENTS.md` conventions: naming, props, composables-over-inline-logic, i18n, no comments in generated code.

## Tests

8. Identify every file touched by the fix. Create or update unit tests in `tests/unit/`, mirroring `src/` structure.
9. Run `bunx vitest run <touched spec files>` and confirm they pass. If `server/src/` was touched, also run `cd server && bunx jest`.

## Review

10. Launch the `code-reviewer` agent on the modified files (implementation + tests), giving it the symptom, root cause, and fix rationale as context.
11. Apply blocking findings. For anything deliberately left unaddressed, state why.

## Final report

- Symptom
- Root cause (file:line + mechanism)
- Fix applied (files changed)
- Tests added/updated + result
- Reviewer findings and resolution
- Residual risk / follow-ups out of scope
