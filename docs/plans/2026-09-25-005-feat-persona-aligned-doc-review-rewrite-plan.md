---
title: "feat: Persona-aligned review and rewrite of all DataStack documentation pages"
type: feat
status: active
date: 2026-09-25
origin: docs/specs/personas.json
---

# feat: Persona-aligned review and rewrite of all DataStack documentation pages

## Overview

Review and rewrite all 18 DataStack documentation pages so that vocabulary, depth, tone, and content match the primary persona's skill profile. A new semantic-foundation page is authored from scratch. All other pages are reviewed against their spec's `audience` block (knows / does_not_know / key_gap) and rewritten where they violate persona fit. Building-block context (Block 1/2/3) and whitepaper SVGs are integrated where specified by the spec.

## Problem Frame

Existing pages were authored for content completeness (narrative arc, correct commands, real API responses) but not explicitly reviewed for persona fit. The `audience` block now added to each spec defines what each primary persona knows and — critically — what they do not know. Several pages likely:

- Use RDF/ontology jargon with P1 and P4 audiences who have no semantic web background
- Skip semantic context for P3 readers who need everything explained from first principles
- Include Docker/ops detail in pages targeting P1 evaluators who should not see it
- Fail the P4 double-gap: assuming domain vocabulary is shared, and omitting that semantic modeling exists as a discipline

The new `pipeline/semantic-foundation.md` page provides shared conceptual vocabulary (linked data, ontology stack, FAIR) that other pages can reference rather than re-explain.

## Requirements Trace

- R1. Every page's vocabulary, examples, and depth match what its primary persona knows and does not know (spec `audience.primary`).
- R2. Pages for P1 contain no unexplained Docker, RDF, SPARQL, or API jargon.
- R3. Pages for P3 introduce every semantic concept (RDF, YARRRML, SPARQL) from first principles — zero assumed prior semantic knowledge.
- R4. Pages for P4 do not assume domain vocabulary is shared; explain what gets modeled and why, not just how.
- R5. `pipeline/semantic-foundation.md` is authored from scratch per its spec, including all three SVGs.
- R6. Where a spec lists `inline_assets_required` SVGs, those SVGs appear in the page with the correct captions.
- R7. `mkdocs build --strict` exits 0 after all rewrites.

## Scope Boundaries

- No changes to `docs/specs/` files (they are source of truth, not output).
- No changes to MkDocs theme, navigation config, or CI — separate task.
- No changes to upstream repo READMEs.
- Persona review uses the spec `audience` block as the lens — not generic prose quality.

### Deferred to Separate Tasks

- Block 3 stubs (`pipeline/advanced/idta-aas.md`, `pipeline/advanced/samm-catena-x.md`): content requires AAS/Catena-X expertise; these pages are authored separately after Block 2 pages are complete.
- GitHub Pages deployment: separate PR after all pages pass `mkdocs build --strict`.

## Context & Research

### Persona Skill Profiles (from docs/specs/personas.json)

| Code | Name | Key gap |
|------|------|---------|
| P1 | Evaluator | No tech background; concept-first; no Docker/RDF/API jargon |
| P2 | Operator | Docker/Linux fluent; zero semantics — skip or defer RDF explanation |
| P3 | Data Engineer | Python/REST fluent; semantic web completely foreign — explain every RDF/SPARQL/YARRRML concept from scratch |
| P4 | Domain Scientist | Deep domain knowledge assumed universal; no ontology terms, no coding — explain what gets modeled and why |

### Page Status Summary

| Status | Pages |
|--------|-------|
| authored | capability-map, default-use-case, author-a-mapping, standalone-apis, csvtocsvw, api-endpoints |
| partial | index, overview, quickstart, configuration, omeroextractor, openbismantic, add-a-use-case, data-formats |
| stub | ontop, idta-aas (deferred), samm-catena-x (deferred) |
| draft (new) | semantic-foundation |

### SVGs Available (docs/assets/)

- `fig-linked-data.svg` — Linked Data Foundation: URIs, CKAN, Fuseki, Sparklis, DCAT federation
- `fig1-ontology-stack.svg` — Industrial Ontology Stack: BFO → PMDCO → domain → application
- `fig2-semantic-layer-architecture.svg` — Semantic Layer Architecture: legacy → semantic layer → outputs
- `fig5-building-blocks.svg` — Building Block Dependency Graph: Block 1–5

### Authoring Workflow (per page)

1. Read `docs/specs/content/<page-slug>.json` — full spec including `audience` block
2. Read all `reference_material.spec_files` listed
3. Audit existing page (if any) against `audience.primary.does_not_know` — flag every violation
4. Audit against `audience.primary.key_gap` — flag every gap
5. Rewrite flagged sections; keep correct sections intact
6. Embed SVGs where spec lists them in `inline_assets_required`
7. Verify `narrative_arc.steps` order is preserved
8. Run `mkdocs build --strict` — must exit 0

### Persona Review Checklist (apply to every page)

- [ ] No term appears in the page that the primary persona's `does_not_know` list includes — unless it is explicitly introduced and explained
- [ ] `key_gap` is addressed: P1 → concept before system; P2 → ops focus, semantics deferred; P3 → semantic concepts explained from scratch; P4 → domain assumptions named, ontology modeling explained as a discipline
- [ ] Vocabulary matches what the persona knows: P2 uses Docker/env var language; P3 uses REST/Python analogies for semantic concepts; P4 uses lab/measurement language
- [ ] Depth is calibrated: P1 pages are concept + outcome, no commands; P4 pages show UI flows, not YARRRML syntax

## Key Technical Decisions

- **Semantic-foundation page first**: It establishes vocabulary (linked data, URI, DCAT, ontology stack, FAIR, Block 2) that all other pages can reference with a link rather than re-explaining. Authors of later pages should cross-link to it.
- **Review-then-rewrite, not replace**: Authored pages (capability-map, default-use-case, etc.) are audited first; only failing sections are rewritten. Correct content is preserved.
- **SVGs via MkDocs asset path**: Embed as `![caption](../assets/fig-linked-data.svg)` with relative paths; verify `mkdocs build --strict` resolves them.
- **Block 3 pages deferred**: `idta-aas` and `samm-catena-x` require specialized AAS/SAMM knowledge and are stubs — they are out of scope for this plan.

## Open Questions

### Resolved During Planning

- **Where do SVGs live?** Confirmed in `docs/assets/` — four SVGs copied from whitepaper repo.
- **Which pages are Block 3?** `pipeline/advanced/idta-aas.md` and `pipeline/advanced/samm-catena-x.md` — deferred.
- **Is ontop a stub?** Yes (38 lines) — included in Phase 5 as a P4 write.

### Deferred to Implementation

- Exact anchor names for cross-links to semantic-foundation from other pages — determined when semantic-foundation is written.
- Whether any `authored` page needs a full rewrite vs. targeted section edits — determined during audit step.

## Implementation Units

### Phase 1 — New Page: Semantic Foundation

- [ ] **Unit 1: Author pipeline/semantic-foundation.md**

**Goal:** Author the new conceptual anchor page from scratch per spec. Establishes shared vocabulary for all other pages.

**Requirements:** R1, R5, R6

**Dependencies:** None

**Files:**
- Create: `docs/pipeline/semantic-foundation.md`
- Read: `docs/specs/content/pipeline-semantic-foundation.json`
- Read: `docs/specs/personas.json`
- Read: `docs/specs/datastack.json`

**Approach:**
- Primary persona P1 (Evaluator): no jargon, concept-first, outcome-driven sentences
- Narrative order per spec: problem framing → linked data principle → `fig-linked-data.svg` → ontology stack (`fig1-ontology-stack.svg`) → Block 2 positioning (`fig5-building-blocks.svg`) → FAIR meaning → "where to go next" callout box (three paths by persona role)
- Use analogies throughout: ontology = structured dictionary, URI = permanent address, semantic layer = translation service
- Real PMD examples: tensile test specimen, steel S355, `dataportal.material-digital.de`
- No RDF syntax, no YARRRML, no Docker references on this page
- SVG embed syntax: `![caption](../assets/<filename>.svg)`
- End with admonition callout routing P2 → quickstart, P3 → author-a-mapping, P4 → standalone-apis

**Test scenarios:**
- Happy path: page renders in `mkdocs build --strict` with no warnings
- Persona check: no term from P1's `does_not_know` list appears unexplained
- SVG check: all three SVGs render (not broken img references)
- Navigation check: cross-links to quickstart, author-a-mapping, standalone-apis resolve

**Verification:**
- `mkdocs build --strict` exits 0
- A non-technical reader (P1) can read the page without encountering unexplained acronyms

---

### Phase 2 — P1 Pages: Evaluator Audience

- [ ] **Unit 2: Review + rewrite index.md (P1, partial)**

**Goal:** Ensure the site landing page serves an evaluator with no technical background — concept, proof point, and role-based routing only.

**Requirements:** R1, R2

**Dependencies:** Unit 1 (semantic-foundation cross-link target)

**Files:**
- Modify: `docs/index.md`
- Read: `docs/specs/content/index.json`

**Approach:**
- Audit: flag any Docker, API, SPARQL, or RDF terms not introduced with explanation
- Current page is 79 lines / partial — likely missing the persona routing and proof point
- Add cross-link to `pipeline/semantic-foundation.md` for readers who want conceptual grounding
- Narrative: pipeline principle → what outputs they get → audience entry point selector → one real-world proof point
- No system detail; no commands

**Test scenarios:**
- Persona check: P1's `does_not_know` terms absent or explained on entry
- Routing check: all three audience paths link to correct entry pages
- Build check: `mkdocs build --strict` exits 0

**Verification:** Page reads as a 2-minute orientation for a PI or data manager with no prior DataStack knowledge.

---

- [ ] **Unit 3: Review + rewrite pipeline/default-use-case.md (P1, authored)**

**Goal:** Verify the authored page stays concept-level for P1 and integrates the building-block context frame.

**Requirements:** R1, R2

**Dependencies:** Unit 1

**Files:**
- Modify: `docs/pipeline/default-use-case.md`
- Read: `docs/specs/content/pipeline-default-use-case.json`

**Approach:**
- Audit against P1 `does_not_know`: Docker, RDF, SPARQL, APIs — flag any that appear without explanation
- Page is 197 lines / authored — may be content-complete but persona-misaligned in spots
- Add a brief "where this fits" reference to Block 2 and semantic-foundation

**Test scenarios:**
- Persona check: no unexplained technical jargon for P1
- Concept check: automation chain is described in outcome terms (what happens to data), not service names
- Build check: `mkdocs build --strict` exits 0

**Verification:** P1 reader understands what happens to their CSV from upload to SPARQL endpoint without needing to know what SPARQL is.

---

- [ ] **Unit 4: Review + rewrite pipeline/capability-map.md (P1, authored)**

**Goal:** Ensure the capability map serves an evaluator assessing fit — outcomes and supported formats, not service internals.

**Requirements:** R1, R2

**Dependencies:** Unit 1

**Files:**
- Modify: `docs/pipeline/capability-map.md`
- Read: `docs/specs/content/pipeline-capability-map.json`

**Approach:**
- Audit against P1 `does_not_know`
- Page is 78 lines / authored — likely thin; check if extractor-capability table is present and described in P1 terms
- Capability descriptions should say "supports microscopy images from OMERO" not "invokes OmeroExtractor service via REST"

**Test scenarios:**
- Persona check: capability descriptions use outcome language, not service/API language
- Completeness check: all supported input formats are listed per spec
- Build check: `mkdocs build --strict` exits 0

**Verification:** A P1 reader can determine whether DataStack supports their data type within 2 minutes.

---

### Phase 3 — P2 Pages: Operator Audience

- [ ] **Unit 5: Review + rewrite pipeline/overview.md (P2, partial)**

**Goal:** Complete the pipeline overview for operators — full component topology with service names, ports, automation boundaries, no semantic explanation.

**Requirements:** R1, R3 (P2 variant: skip semantics)

**Dependencies:** Unit 1 (cross-link to semantic-foundation for curious readers)

**Files:**
- Modify: `docs/pipeline/overview.md`
- Read: `docs/specs/content/pipeline-overview.json`
- Read: `docs/specs/datastack.json`

**Approach:**
- Audit: ensure Mermaid diagram is present (spec requires it); fill gaps in partial content
- P2 key gap: zero semantics — RDF/ontology mentions should be minimal and deferrable ("produces RDF — see Semantic Foundation for details")
- Component table: service name, image, role, public URL / internal only
- Add `fig2-semantic-layer-architecture.svg` if spec requires it (check spec)

**Test scenarios:**
- Persona check: RDF/SPARQL/ontology terms are not explained inline — they link out
- Diagram check: Mermaid diagram renders in `mkdocs build --strict`
- Completeness check: all services in `docs/specs/datastack.json` appear in component table
- Build check: `mkdocs build --strict` exits 0

**Verification:** A P2 operator can understand service topology, port layout, and automation vs. manual triggers without reading about ontologies.

---

- [ ] **Unit 6: Review + rewrite guides/quickstart.md (P2, partial)**

**Goal:** Complete quickstart for operators — exact commands, env var names, expected log output, no semantic context.

**Requirements:** R1

**Dependencies:** Unit 5

**Files:**
- Modify: `docs/guides/quickstart.md`
- Read: `docs/specs/content/guides-quickstart.json`
- Read: `docs/specs/datastack.json`

**Approach:**
- Page is 191 lines / partial — likely has structure but gaps in commands or post-boot steps
- Every command must be reproducible as-is (real env var names from `.env.example`, real CKAN UI paths)
- Warning admonition for `BACKGROUNDJOBS_API_TOKEN` (spec requirement)
- Note admonition for `mappings` group name (case-sensitive, spec requirement)
- No semantic explanation — if Fuseki/SPARQL appears, one sentence max then link to semantic-foundation

**Test scenarios:**
- Reproducibility check: every `docker compose` command uses real flags from DataStack repo
- Warning check: BACKGROUNDJOBS_API_TOKEN warning admonition present
- Note check: mappings group name note present
- Build check: `mkdocs build --strict` exits 0

**Verification:** A P2 operator can follow the page step-by-step and reach a running stack with a processed CSV.

---

- [ ] **Unit 7: Review + rewrite reference/configuration.md (P2, partial)**

**Goal:** Complete configuration reference for operators — all critical env vars with types, defaults, examples.

**Requirements:** R1

**Dependencies:** None

**Files:**
- Modify: `docs/reference/configuration.md`
- Read: `docs/specs/content/reference-configuration.json`
- Read: `docs/specs/datastack.json` (critical_env_vars section)

**Approach:**
- Page is 123 lines / partial — likely missing some vars or type/default details
- P2 language: infrastructure terms (env var, port, volume, token) — no ontology terms
- Group by service; include type annotation, default, required/optional, example value

**Test scenarios:**
- Coverage check: all `critical_env_vars` from `docs/specs/datastack.json` appear
- Format check: each var has type, default, and example
- Build check: `mkdocs build --strict` exits 0

**Verification:** A P2 operator can configure the stack from this page alone without reading source code.

---

### Phase 4 — P3 Pages: Data Engineer Audience

P3 key gap: **semantic web completely foreign — explain every RDF/SPARQL/YARRRML concept from first principles.** Every page in this phase must introduce semantic concepts with an analogy before using the term.

- [ ] **Unit 8: Review + rewrite guides/author-a-mapping.md (P3, authored)**

**Goal:** Verify the mapping guide introduces YARRRML and RML from scratch — no assumed semantic knowledge.

**Requirements:** R1, R3

**Dependencies:** Unit 1 (semantic-foundation provides PMDCO/ontology context)

**Files:**
- Modify: `docs/guides/author-a-mapping.md`
- Read: `docs/specs/content/guides-author-a-mapping.json`

**Approach:**
- Page is 342 lines / authored — likely content-complete; audit for assumed semantic prior knowledge
- Check: is "triple" explained before use? Is "YARRRML" introduced with an analogy (e.g., "a recipe that tells the pipeline how to convert each CSV column into an RDF statement")?
- P3 has Python/REST background — use these as analogies: YARRRML is to RDF as a Jinja template is to HTML
- Block 1 page — should reference `docs/assets/fig1-ontology-stack.svg` if spec requires it

**Test scenarios:**
- Semantic intro check: "triple", "RDF", "YARRRML", "predicate" each introduced with explanation on first use
- Analogy check: at least one software-engineering analogy per new semantic concept
- Build check: `mkdocs build --strict` exits 0

**Verification:** A P3 reader with no RDF background can follow the guide without needing to look up "what is a triple."

---

- [ ] **Unit 9: Review + rewrite extractors/csvtocsvw.md (P3, authored)**

**Goal:** Ensure the CSV extractor page explains what CSVW is and why it matters before showing the API.

**Requirements:** R1, R3

**Dependencies:** Unit 8 (mapping context established)

**Files:**
- Modify: `docs/extractors/csvtocsvw.md`
- Read: `docs/specs/content/extractors-csvtocsvw.json`

**Approach:**
- Page is 230 lines / authored — audit for unexplained semantic terms (CSVW, JSON-LD, metadata file)
- CSVW concept: "a JSON sidecar file that annotates your CSV columns with data types, units, and ontology term links — turning a plain spreadsheet into self-describing data"
- API contract language appropriate for P3; ontology terms explained by reference to semantic-foundation

**Test scenarios:**
- Concept check: CSVW explained before API is shown
- Jargon check: JSON-LD, metadata file, annotation — all introduced
- Build check: `mkdocs build --strict` exits 0

**Verification:** A P3 reader understands what CSVToCSVW produces and why before they call the API.

---

- [ ] **Unit 10: Review + rewrite reference/api-endpoints.md (P3, authored)**

**Goal:** Confirm API reference is complete and uses P3-appropriate language (REST contract, not semantic theory).

**Requirements:** R1

**Dependencies:** None

**Files:**
- Modify: `docs/reference/api-endpoints.md`
- Read: `docs/specs/content/reference-api-endpoints.json`

**Approach:**
- Page is 207 lines / authored — likely content-complete; audit for balance: P3 needs HTTP contract language, not ontology explanation
- Any RDF output format mentioned should note "see Semantic Foundation for what this means" rather than explaining inline
- Check: all endpoints have method, path, request/response schema, example

**Test scenarios:**
- Contract check: each endpoint has method + path + example request + example response
- Depth check: semantic terms not over-explained (link out instead)
- Build check: `mkdocs build --strict` exits 0

**Verification:** A P3 engineer can integrate with every DataStack API from this page alone.

---

- [ ] **Unit 11: Review + rewrite reference/data-formats.md (P3, partial)**

**Goal:** Complete data formats reference — what each format is, what produces it, what consumes it.

**Requirements:** R1, R3

**Dependencies:** Unit 9 (CSVW context)

**Files:**
- Modify: `docs/reference/data-formats.md`
- Read: `docs/specs/content/reference-data-formats.json`

**Approach:**
- Page is 107 lines / partial — likely missing format details or examples
- Each format: what it is (analogy if semantic), what produces it, what consumes it, example snippet
- P3 analogies for RDF formats: Turtle = "human-readable RDF like YAML is human-readable JSON"

**Test scenarios:**
- Coverage check: CSVW, JSON-LD, Turtle, RDF/XML all documented
- Analogy check: each semantic format introduced with a software analogy
- Build check: `mkdocs build --strict` exits 0

**Verification:** A P3 reader knows which format each service produces and can consume them programmatically.

---

- [ ] **Unit 12: Review + rewrite extractors/omeroextractor.md (P3, partial)**

**Goal:** Complete OMERO extractor page — what it does, JSON-LD output structure, DF-TEM-PAW example, maturity status.

**Requirements:** R1, R3

**Dependencies:** Unit 9 (CSV extractor established extractor pattern)

**Files:**
- Modify: `docs/extractors/omeroextractor.md`
- Read: `docs/specs/content/extractors-omeroextractor.json`

**Approach:**
- Page is 188 lines / partial — check spec for gaps (DF-TEM-PAW example, maturity status)
- JSON-LD output: show real example from spec or live instance
- P3 framing: "OmeroExtractor connects to your OMERO server via the OMERO API and produces a JSON-LD annotation file — the same format as a CSVW metadata file but for microscopy datasets"

**Test scenarios:**
- Completeness check: DF-TEM-PAW example present (or marked illustrative)
- Maturity check: current maturity status documented
- Build check: `mkdocs build --strict` exits 0

**Verification:** A P3 engineer knows whether OmeroExtractor supports their OMERO setup and what output to expect.

---

### Phase 5 — P4 Pages: Domain Scientist Audience

P4 double gap: **(1) no semantic web knowledge, (2) assumes their domain concepts are universal.** Every page must (a) not assume domain vocabulary is shared, and (b) explain that semantic modeling exists as a discipline and what it does.

- [ ] **Unit 13: Review + rewrite guides/add-a-use-case.md (P4, partial)**

**Goal:** Complete the add-a-use-case guide in P4 language — lab workflow framing, no YARRRML syntax visible.

**Requirements:** R1, R4

**Dependencies:** Unit 1 (semantic-foundation), Unit 8 (mapping guide for cross-link)

**Files:**
- Modify: `docs/guides/add-a-use-case.md`
- Read: `docs/specs/content/guides-add-a-use-case.json`

**Approach:**
- Page is 176 lines / partial — likely has steps but may use technical language P4 won't understand
- P4 framing: "your data describes a tensile test — but the pipeline needs to know what 'yield strength' means in a way any computer can understand. A mapping file is the bridge."
- UI-first: show what the CKAN UI looks like at each step, not just API calls
- Do not show YARRRML syntax; reference author-a-mapping for that ("your data engineer will handle this part")

**Test scenarios:**
- Language check: no unexplained YARRRML, RDF, SPARQL on this page
- Domain check: page uses measurement/lab examples, not abstract data terms
- Handoff check: page explicitly routes to author-a-mapping for the mapping step
- Build check: `mkdocs build --strict` exits 0

**Verification:** A domain scientist can follow this guide to submit a new use case without writing any code or mapping syntax.

---

- [ ] **Unit 14: Review + rewrite guides/standalone-apis.md (P4, authored)**

**Goal:** Verify standalone APIs guide uses P4 language — form/UI focus, outcome descriptions, no coding.

**Requirements:** R1, R4

**Dependencies:** None

**Files:**
- Modify: `docs/guides/standalone-apis.md`
- Read: `docs/specs/content/guides-standalone-apis.json`

**Approach:**
- Page is 196 lines / authored — audit for P4 double gap violations
- P4 does not know what a REST API is — any API call shown must be framed as "click this button" or use a form UI, not curl
- Outcome first: "upload your CSV, get FAIR data back" before explaining any technical step
- Domain example: steel tensile data or lab measurement scenario

**Test scenarios:**
- Jargon check: "REST", "endpoint", "curl" absent or explained for P4
- Outcome check: page opens with what the scientist gets, not what the system does
- Build check: `mkdocs build --strict` exits 0

**Verification:** A materials scientist can use the standalone APIs from this guide without developer help.

---

- [ ] **Unit 15: Review + rewrite extractors/openbismantic.md (P4, partial)**

**Goal:** Complete OpenBIS extractor page in P4 language — what it does for their lab data, not how it works internally.

**Requirements:** R1, R4

**Dependencies:** Unit 13

**Files:**
- Modify: `docs/extractors/openbismantic.md`
- Read: `docs/specs/content/extractors-openbismantic.json`

**Approach:**
- Page is 189 lines / partial — check what's missing per spec
- P4 framing: "if your lab uses OpenBIS to manage experiments, OpenBISMantic can extract that data automatically"
- Explain what OpenBIS is (one sentence) — do not assume P4 knows the term even if they use it
- P4 double gap: OpenBIS users may not know their own data follows ontological patterns — explain that their experiment records already describe processes and materials, and that the extractor makes that explicit

**Test scenarios:**
- P4 language check: no unexplained RDF/API terms
- Domain assumption check: OpenBIS explained on first use
- Modeling explanation: page explains what gets modeled and why, not just how
- Build check: `mkdocs build --strict` exits 0

**Verification:** A lab scientist using OpenBIS understands what data gets extracted and what they gain from it.

---

- [ ] **Unit 16: Author extractors/ontop.md (P4, stub)**

**Goal:** Write the Ontop extractor page from near-scratch — it is a 38-line stub.

**Requirements:** R1, R4

**Dependencies:** Unit 1 (semantic-foundation), Unit 15 (extractor pattern established)

**Files:**
- Modify: `docs/extractors/ontop.md`
- Read: `docs/specs/content/extractors-ontop.json`

**Approach:**
- Primary P4: domain scientist with a relational database (SQL) wanting FAIR output
- P4 framing: "Ontop connects to your relational database — the same database your ELN or LIMS uses — and makes its data queryable using shared scientific vocabulary, without moving the data"
- No SPARQL syntax visible; no OWL; explain "virtual knowledge graph" as "a live view of your database in semantic form"
- Include current maturity and limitations per spec

**Test scenarios:**
- Concept check: virtual knowledge graph explained with analogy
- Language check: no unexplained SPARQL/OWL/RDF on this page
- Limitation check: known limitations documented
- Build check: `mkdocs build --strict` exits 0

**Verification:** A domain scientist with a SQL database understands whether and how Ontop applies to their setup.

---

### Phase 6 — Integration

- [ ] **Unit 17: Integration verification**

**Goal:** Verify all 16 rewritten pages plus semantic-foundation pass `mkdocs build --strict` with correct cross-links.

**Requirements:** R7

**Dependencies:** Units 1–16

**Files:**
- Read: all modified `.md` files
- Run: `mkdocs build --strict`

**Approach:**
- Fix any broken anchor cross-links introduced during rewrites
- Verify SVG asset paths resolve (relative paths from each page's directory to `assets/`)
- Check navigation entries for semantic-foundation are in `mkdocs.yml`

**Test scenarios:**
- Build check: `mkdocs build --strict` exits 0 with no warnings
- Link check: no broken internal cross-links
- SVG check: all four SVG assets referenced in pages resolve

**Verification:** Site builds clean; all pages accessible in local `mkdocs serve`.

## System-Wide Impact

- **Cross-link graph:** semantic-foundation is the hub — units 2–16 all link to it; it must be authored first (Unit 1)
- **SVG asset paths:** relative paths differ by page depth (`../assets/` vs `../../assets/`) — verify per page
- **Navigation:** `mkdocs.yml` must include `pipeline/semantic-foundation.md` — check before Unit 17
- **Unchanged invariants:** `docs/specs/content/*.json` files are not modified by any unit

## Risks & Dependencies

| Risk | Mitigation |
|------|------------|
| SVG relative paths break for deeply nested pages (e.g., `pipeline/advanced/`) | Unit 17 verifies with `mkdocs build --strict`; fix per page during authoring |
| P4 domain examples require materials science knowledge to write correctly | Use spec's example_urls and whitepaper examples (steel S355, tensile test) — these are already vetted |
| Authored pages (capability-map, default-use-case, author-a-mapping) need more than targeted edits | Audit step in each unit surfaces this; full rewrite is acceptable outcome |
| semantic-foundation page sets wrong vocabulary that later pages import | Unit 1 must pass persona check (P1, no jargon) before units 2+ start |

## Sources & References

- `docs/specs/personas.json` — P1–P4 skill profiles (primary source of truth)
- `docs/specs/content/*.json` — per-page specs with audience, narrative arc, inline assets
- `docs/specs/datastack.json` — DataStack service and env var reference
- `semantics-industry-whitepaper/rework/whitepaper-body.md` — §2, §4 Building Blocks (conceptual source for semantic-foundation)
- SVGs: `docs/assets/fig-linked-data.svg`, `fig1-ontology-stack.svg`, `fig2-semantic-layer-architecture.svg`, `fig5-building-blocks.svg`
- Prior plan: `docs/plans/2026-09-24-004-feat-parallel-content-authoring-plan.md`
