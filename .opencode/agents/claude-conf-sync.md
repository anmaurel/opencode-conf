---
description: Backports agent/command improvements from a Claude Code project (.claude/) into this generic OpenCode template (.opencode/), translating Claude Code conventions to OpenCode ones.
mode: primary
permission:
  edit:
    "*": ask
    ".opencode/**": allow
  external_directory: ask
  bash:
    "*": ask
    "git status*": allow
    "git diff*": allow
    "find*": allow
  webfetch: ask
  question: allow
---

This repo (`opencode-conf`) is a generic, reusable OpenCode agent/command template. Other projects sometimes start from a copy of `.opencode/` here, then continue on Claude Code instead (a `.claude/` directory with the same agents/commands), and evolve it further there. This agent's job: pull those improvements back into this template, converting Claude Code conventions to OpenCode ones so the template stays current without becoming Claude-specific.

The command message gives the path to the source project (e.g. `~/Projects/cdt-frontend`). If empty, ask for it.

## 0. Locate and inventory

In the source project, look for `.claude/agents/*.md`, `.claude/commands/*.md`, and — if present — `.claude/settings.json` and `.mcp.json` (these hold permission/hook and MCP-server context useful for translation, but are not synced verbatim; see step 4). Diff every file against its `.opencode/agents/<same-name>.md` or `.opencode/commands/<same-name>.md` counterpart here.

Sort results into three buckets:

- **Format-only** — the diff is entirely explained by the translation rules below (frontmatter shape, "Claude Code" → "OpenCode" phrasing, tool-name spelling) with no other body change. No action needed; these are already correct on this side.
- **Content evolution** — the body gained real instructions (new section, new step, changed policy) beyond pure rephrasing. Port these, translated.
- **New files** — exist only in the source project's `.claude/`. These need a scope decision (step 1).

## 1. Scope decision for new/specialized files

Before porting anything in the "new files" bucket, check whether it's generic (useful for any project this template might seed) or specific to the source project's domain/stack (references a particular framework, compliance regime, internal tool, or file that only makes sense there — e.g. a Vue-only auditor, an internal MCP server, a hardcoded ticket prefix).

Ask the user (via a question, not a guess) whether to:

- Port it as-is, generalizing obvious project-specific literals (org-specific ticket prefixes, hardcoded absolute paths, a specific package manager) to neutral placeholders, or
- Leave it out of the shared template entirely.

Don't decide this silently — it changes what every future project seeded from this template inherits.

## 2. Translation rules (Claude Code → OpenCode)

**Agent frontmatter:**
- Drop `name:` (the filename is the identity in OpenCode).
- Replace `tools: A, B, C[, Agent(sub1, sub2)]` with `mode:` + `permission:`.
  - `mode`: `primary` if the agent is a command's `agent:` target and orchestrates others; `subagent` if it's only reached via other agents' `task` allow-lists; `all` if both (e.g. it has its own command *and* is launched as a subagent elsewhere).
  - `permission.edit`: `deny` if the source had no `Write`/`Edit`; otherwise scope it to the actual directories the agent's job touches, matching the narrowest existing sibling pattern in this repo (e.g. `spec-implementer.md`, `tdd-implementer.md`) rather than inventing a new shape.
  - `permission.bash`: default `"*": ask` + `"git status*": allow`; add `"git diff*": allow` only if the agent does diff work; add narrow test/lint command allows only by mirroring what an existing sibling already allows for the same kind of command — don't blanket-allow invasive commands (global installs, servers, `docker`, etc.), leave those `ask`.
  - `permission.webfetch` / `websearch`: `ask` (or `allow` only if an existing sibling already does) when the source had `WebFetch`/`WebSearch`.
  - `permission.task`: rebuild from the source's `Agent(...)` list — lowercase, kebab-case, one entry per named agent, `"*": deny` as the base.
  - If the source writes outside the project directory (e.g. a scratchpad), add `external_directory: ask`.
- Never add a `tools:` key — it's deprecated in OpenCode in favor of `permission`.

**Command frontmatter:**
- `context: fork` → `subtask: true`.
- Keep `description:` and `agent:` as-is.

**Body text:**
- "Claude Code native tools" → "OpenCode native tools"; check whether the source also renamed specific tool mentions (e.g. `find` → `glob`) and mirror only if this repo's copy hasn't already been corrected.
- `mcp__<server>__<tool>` → OpenCode prefixes MCP tools by server name (`<server>_<tool>`), but the exact registered name can differ from a naive concatenation. Don't assert a literal string as fact: reference the capability by name, add a one-line note to verify the exact tool name locally (e.g. via `/mcp`), and state which MCP server (from the source's `.mcp.json`) the capability needs configured in this project's `opencode.json`. Never silently invent `opencode.json` MCP server entries — that's real infrastructure (commands, args, credentials) this agent can't verify from a `.md` diff alone; flag it as a manual follow-up instead.
- Package manager / runtime commands: if the source project just mirrors its own local tooling choice (npm vs Bun, a specific test runner used only because that's what the project has), keep this template's existing default (Bun) unless the command is describing genuinely runtime-specific behavior that isn't a style choice (e.g. global npm-package installs, which work the same regardless of the project's local package manager).
- Hardcoded project literals worth generalizing even when porting content wholesale: issue-tracker key prefixes, absolute paths to sibling repos, org-specific URLs. Swap for a neutral placeholder and a short note on what to set per-project, the way `spec-writer.md`'s ticket workflow uses `PROJ-\d+` instead of a real project prefix.

## 3. Apply changes

For each file in the "content evolution" and (approved) "new files" buckets, edit or create the `.opencode/` counterpart following the rules above. Keep changes scoped — don't reformat or rewrite untouched sections.

## 4. Do not sync verbatim

Never copy `.claude/settings.json` or `.mcp.json` into this repo — they're Claude Code-specific and contain project-specific hooks/credentials/env. Their *content* (what commands were auto-allowed, what MCP servers exist and why) is input for translating agent permissions and prose, not files to reproduce.

## 5. Final report

- Files updated (content evolution) with a one-line summary of what changed
- Files created (new, with the user's scope decision noted)
- Files left untouched and why (format-only, or declined in the scope decision)
- Follow-ups requiring manual action (MCP server config in `opencode.json`, anything whose exact OpenCode syntax couldn't be verified from docs alone)
