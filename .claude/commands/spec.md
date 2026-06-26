Generate a spec from $ARGUMENTS if it is a plain description, or implement from $ARGUMENTS if it is a path to an existing spec file.

**Rule:** if $ARGUMENTS ends with `.md`, treat it as a file path (implement mode). Otherwise, treat it as a description (generate mode).

---

## Generate mode — `/spec <description>`

Create a spec file in `docs/specs/` from a plain-text description of what needs to be built.

### 0. Initialisation

- Read `AGENTS.md` (features inventory, conventions, naming rules)
- Read `docs/wiki/_Index.md` to identify relevant wiki notes to link

### 1. Identify the feature scope

From the description, determine:

- Which feature(s) from the inventory are involved (`src/features/<name>/`)
- Which shared services, composables, or stores are likely affected

### 2. Explore the codebase (parallel)

Run simultaneously:

**A. Grep for existing types and API classes**

- `grep -r "<keyword>" src/` for each key term in the description
- Identify the `*Api.ts` classes and `*ApiResponse` interfaces already in place

**B. Read the relevant feature folder**

- List `src/features/<name>/` to map existing composables, components, services
- Read the API class if it exists

### 3. Build the spec

Read `docs/specs/_template.md`, then produce `docs/specs/<slug>.md` where `<slug>` is a lowercase-hyphenated summary of the description. Never include a Jira reference (`CDT-XXXX`) in the slug, the title, or the body.

Fill every section precisely:

- **Acceptance criteria**: concrete, testable, observable behaviours — not vague intentions
- **APIs involved**: real endpoints with their TypeScript return types (from step 2A)
- **Components / Composables**: list actual files from step 2B, not hypothetical ones
- **Types / Models**: paste the real existing interfaces, or draft new ones consistent with the `*ApiResponse` pattern
- **Expected tests**: name the spec files to create/update in `tests/unit/`
- **Wiki links**: link relevant notes via `[[filename]]` (no extension)

Set `status: draft` in frontmatter.

### 4. Report

- Path of the created spec file
- 3-bullet summary of what will be built
- List any ambiguities the developer should resolve before setting status to `ready`

---

## Implement mode — `/spec <path>`

Read the spec and implement it end-to-end.

### 0. Initialisation

- Read the spec at `<path>` fully
- Read `AGENTS.md` for conventions
- Read any `[[wiki-note]]` linked in the spec's "Liens wiki" section

### 1. Explore relevant code (parallel)

Run simultaneously:

**A.** Read each file listed under "Composants / Composables → À modifier"
**B.** Read the `*Api.ts` class(es) named in "APIs concernées"
**C.** Read the existing test files for the affected feature in `tests/unit/`

### 2. Set spec status to `in-progress`

Update the frontmatter `status` field to `in-progress` before writing any source code.

### 3. Implement

Work through the spec in this order:

1. Types / Modèles — create or extend `*ApiResponse` interfaces and domain models
2. API class — add or update methods in `*Api.ts`
3. Composable — implement logic (state, API calls, transformations)
4. Component — wire the composable; keep the component thin
5. i18n keys — add to the relevant JSON file if needed

Follow all conventions from AGENTS.md strictly:

- No inline comments or JSDoc
- Reactive refs prefixed `r_`
- Props in snake_case
- BEM CSS classes

### 4. Tests

Write unit tests for every new composable or utility.
Run `npm run lint` and `npx vitest run <spec-file>` to verify.

### 5. Set spec status to `done`

Update `status: done` in the spec frontmatter.

### 6. Report

- Files created or modified (with paths)
- Test results summary
- Any deviations from the spec (with justification)
