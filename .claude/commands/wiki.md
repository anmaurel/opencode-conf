Ingest the documentation provided in $ARGUMENTS into `docs/wiki/`, following the project conventions in CLAUDE.md / AGENTS.md.

## Mandatory conventions

These rules apply to **every** note produced, without exception:

- **No personal names** (developers, POs, analysts, authors) — not in content, tags, metadata, or citations
- **No Jira references** (`CDT-XXXX` or any other ticket identifier) — anywhere in the note
- Useful technical citations are kept but rephrased without naming their author
- Internal links: always `[[filename]]` wikilinks, never relative Markdown paths
- File naming: lowercase, hyphens (`loan-application-api.md`, `authentication-sso.md`)

## Steps

### 0. Initialization

Before any analysis:

- Read `AGENTS.md` for the complete features inventory (`src/features/`) and code conventions
- Read `docs/wiki/_Index.md` to identify existing notes and reusable `[[wikilinks]]`

### 1. Read the source

- URL → fetch the content
- File path → read the file
- Empty → ask the user to paste the content directly

### 2. Check for an existing note

Search `docs/wiki/` for a note whose title, tags, or subject matches the ingested source.

- Existing note found → prepare an **update** (do not create a duplicate)
- No note found → prepare a **creation**

### 3. Deep analysis and codebase grep

These actions can be run in parallel:

**A. Extract from the source:**

- **Subject**: what is this doc about? (API, feature spec, architecture decision, external service)
- **Key concepts**: new terms, protocols, data models, configuration keys
- **API surface** (if applicable): endpoints, HTTP methods, request/response shapes, auth scheme, error codes
- **TypeScript implications**: interfaces or types that should exist or be updated in `src/`
- **Open questions**: unclear points, missing information, things to validate with the backend team

**B. Map the affected features:**
Refer to the complete features inventory in AGENTS.md (section "Features Inventory") — do not use a hardcoded list.

**C. Grep the codebase:**
For each key term, endpoint, or config key identified in A:

- `grep -r "<keyword>" src/`
- `grep -r "<endpoint-path>" src/`

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
For **feature notes**: list the files in `src/features/<name>/` that are affected.
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
- No Jira references (`CDT-XXXX`) in content, tags, or metadata
- All internal links are `[[wikilink]]` format

Fix any violations, then:

**Report:**

- List of every file created or modified (with paths)
- 3-5 bullet summary of what was learned
- Open questions needing follow-up
