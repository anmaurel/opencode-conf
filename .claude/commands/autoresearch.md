Conduct autonomous technical research on the topic in $ARGUMENTS and produce structured notes in `docs/wiki/`, in the context of the cdt-frontend project (Vue 3, TypeScript, Vite, Pinia, Vitest).

## Mandatory conventions

These rules apply to **every** note produced, without exception:

- **No personal names** (developers, POs, analysts, authors) — not in content, tags, metadata, or citations
- **No Jira references** (`CDT-XXXX` or any other ticket identifier) — anywhere in the note
- Useful technical citations are kept but rephrased without naming their author
- Internal links: always `[[filename]]` wikilinks, never relative Markdown paths
- File naming: lowercase, hyphens (`vue-query.md`, `axios-interceptors.md`)

## Steps

### 0. Initialization

Before any search:

- Read `AGENTS.md` to know the exact stack, code conventions, and complete features inventory (`src/features/`)
- Read `docs/wiki/_Index.md` to identify existing notes and reusable `[[wikilinks]]`
- If a note already partially covers the topic → prepare an update rather than a new file

### 1. Parse the topic

From `$ARGUMENTS`, identify:

- The core subject (library, pattern, protocol, concept)
- The angle: best practices? comparison? migration? integration?
- The relevant project area (authentication, state management, testing, build, API layer, etc.)

If `$ARGUMENTS` is empty, ask the user for a topic and angle.

### 2. Check existing wiki

- Read `docs/wiki/_Index.md`
- Search `docs/wiki/` for notes whose tags or title overlap with the topic
- If a relevant note exists: update it rather than creating a duplicate, and reuse existing `[[wikilinks]]`

### 3. Round 1 — broad search

Launch all 3 searches **simultaneously**:

- **Search A**: official docs or reference spec
- **Search B**: recent developments / releases (last 12 months)
- **Search C**: real-world usage patterns in Vue 3 + TypeScript projects

### 4. Targeted searches — fill the gaps

After round 1, list what is still unclear and run 2-3 targeted follow-up searches:

- Integration specifics for the stack identified in AGENTS.md (Vite, Pinia, Vitest, Axios, existing patterns)
- Known issues, gotchas, or breaking changes
- Comparison with the approach currently used in the project (if applicable)

### 5. Synthesize findings

Structure the synthesis around:

- What it is and why it matters for the project
- How it fits with existing codebase patterns (features inventory, AxiosProxy, Pinia store pattern, composables) — based on the AGENTS.md read at step 0
- Concrete integration steps or code patterns
- Trade-offs vs. the current approach
- Sources (all URLs used)

### 6. Read the appropriate template

Read the matching template from `docs/wiki/Templates/` before writing:

| Subject                 | Template                                   |
| ----------------------- | ------------------------------------------ |
| Library / tool to adopt | `docs/wiki/Templates/architecture-note.md` |
| Pattern / technique     | `docs/wiki/Templates/architecture-note.md` |
| Feature-scoped research | `docs/wiki/Templates/feature-note.md`      |
| API / external service  | `docs/wiki/Templates/api-note.md`          |

Fill every section of the template — leave nothing empty; write "N/A" or "To document" if information is genuinely missing.

### 7. Create or update the main note

| Subject type            | Path                                       |
| ----------------------- | ------------------------------------------ |
| Library / tool to adopt | `docs/wiki/architecture/<tool-name>.md`    |
| Pattern / technique     | `docs/wiki/architecture/<pattern-name>.md` |
| Feature-scoped research | `docs/wiki/features/<feature-name>.md`     |
| API / external service  | `docs/wiki/api/<service-name>.md`          |

If the note already exists (detected at step 2): update the relevant sections without overwriting valid existing content.

### 8. Create concept stub notes

One stub per new concept, library, or technology discovered that deserves its own note.
Link back to the main note via `[[wikilink]]`.

### 9. ADR — conditional

Create an ADR in `docs/wiki/decisions/adr-<NNN>-<slug>.md` **only if** the research concludes with one of these outcomes:

- Adoption of a new library or tool replacing an existing one
- Explicit rejection of an approach documented as an alternative
- Decided migration from one pattern to another

**Do not create an ADR for**: knowledge updates, exploratory research without an actionable conclusion, confirmations of existing practices, comparisons without a decision.

The NNN number follows the last ADR in `docs/wiki/decisions/`. Use the `docs/wiki/Templates/decision-note.md` template.

### 10. Update `docs/wiki/_Index.md`

Add the note under **"Notes récentes"** with today's date, title, and keywords.
If the topic was listed under "Sujets à documenter", remove it from that section.

### 11. Pre-delivery verification

Check that all produced notes comply with:

- No personal names (developer, PO, analyst, source author)
- No Jira references (`CDT-XXXX` or any ticket identifier)
- All internal links are `[[wikilink]]` format, not relative Markdown paths

Fix any violations before reporting.

### 12. Report

- Files created or modified (with paths)
- Key findings (5 bullets max)
- Recommended next action (implement, discuss with team, open ADR, etc.)
- Open questions for further research
- Confirmation that no-names / no-Jira conventions are respected
