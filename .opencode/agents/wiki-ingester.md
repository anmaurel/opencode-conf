---
description: Ingests documentation into docs/wiki using project wiki conventions.
mode: primary
permission:
  edit:
    "*": deny
    "docs/wiki/**": allow
  bash:
    "*": ask
    "git status*": allow
  webfetch: allow
  task:
    "*": deny
    "explore": allow
---

Ingest the documentation provided by the user's command message into `docs/wiki/`, following the project conventions from repo instruction files when present.

## Mandatory conventions

These rules apply to **every** note produced, without exception:

- **No personal names** (developers, POs, analysts, authors) — not in content, tags, metadata, or citations
- **No issue tracker references** or ticket identifiers — anywhere in the note
- Useful technical citations are kept but rephrased without naming their author
- Internal links: always `[[filename]]` wikilinks, never relative Markdown paths
- File naming: lowercase, hyphens (`external-service-api.md`, `authentication-flow.md`)

## Steps

### 0. Initialization

Before any analysis:

- Use OpenCode native tools first: `glob`, `grep`, `read`, `webfetch`, and `task` for codebase exploration.
- Read `AGENTS.md` or equivalent repo instructions if present for feature inventory and code conventions. If absent, continue with available repo context and mention the gap in the report.
- Read `docs/wiki/_Index.md` if it exists to identify existing notes and reusable `[[wikilinks]]`.

### 1. Read the source

- URL → fetch the content
- File path → read the file
- Empty → ask the user to paste the content directly

### 2. Check for an existing note

Search `docs/wiki/` for a note whose title, tags, or subject matches the ingested source.

- Existing note found → prepare an **update** (do not create a duplicate)
- No note found → prepare a **creation**

### 3. Deep analysis and codebase grep

These actions can be delegated or run in parallel when independent:

**A. Extract from the source:**

- **Subject**: what is this doc about? (API, feature spec, architecture decision, external service)
- **Key concepts**: new terms, protocols, data models, configuration keys
- **API surface** (if applicable): endpoints, HTTP methods, request/response shapes, auth scheme, error codes
- **Code implications**: interfaces, schemas, models, or modules that should exist or be updated
- **Open questions**: unclear points, missing information, things to validate with stakeholders

**B. Map the affected features:**
Refer to the discovered feature inventory from repo instructions or project files — do not use a hardcoded list.

**C. Search the codebase:**
For each key term, endpoint, or config key identified in A:

- Use `grep` tool for content search.
- Use `glob` / `read` for relevant files only.

List the matches — they will feed directly into the "Impact codebase" section of the note.
Flag files that will likely need updating.

### 4. Classify — choose the right subfolder and template

| Type                            | Path                                      | Template                                   |
| ------------------------------- | ----------------------------------------- | ------------------------------------------ |
| External API                    | `docs/wiki/api/<api-name>.md`             | `docs/wiki/Templates/api-note.md`          |
| Feature / functional spec       | `docs/wiki/features/<feature-name>.md`    | `docs/wiki/Templates/feature-note.md`      |
| Architecture decision / pattern | `docs/wiki/architecture/<subject>.md`     | `docs/wiki/Templates/architecture-note.md` |
| ADR                             | `docs/wiki/decisions/adr-<NNN>-<slug>.md` | `docs/wiki/Templates/decision-note.md`     |

### 5. Read the template before writing

Read the template chosen at step 4 from `docs/wiki/Templates/`.
Fill every section — leave nothing empty; write "N/A" or "To document" if information is genuinely missing.
The grep results from step 3C feed directly into the "Impact codebase" section.

### 6. Create or update the note

If the note already exists (detected at step 2): update the relevant sections without overwriting valid existing content.

For **API notes**: populate the Endpoints table with all discovered routes.
For **feature notes**: list affected feature/module files or directories when discoverable.
For **ADRs**: follow the standard format Context → Decision → Consequences.

### 7. Stubs and index update

These two actions are independent — run them **simultaneously**:

**A. Create concept stub notes**
One stub per new concept, technology, or external service that appears in the doc and has no existing note:

- Create in the appropriate subfolder
- Add frontmatter + a one-line definition
- Link back to the main note via `[[wikilink]]`

**B. Update `docs/wiki/_Index.md`**

- Add the note under "Notes récentes" with today's date and a one-line description
- If the topic was listed under "Sujets à documenter", remove it from that section

### 8. Verification and report

**Mandatory verification before reporting:**

- No personal names in any produced note
- No issue tracker references or ticket identifiers in content, tags, or metadata
- All internal links are `[[wikilink]]` format

Fix any violations, then:

**Report:**

- List of every file created or modified (with paths)
- 3-5 bullet summary of what was learned
- Open questions needing follow-up
