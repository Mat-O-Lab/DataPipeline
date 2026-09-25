---
title: "feat: Parallel authoring of all remaining DataStack documentation pages"
type: feat
status: active
date: 2026-09-24
origin: docs/plans/2026-09-24-002-feat-content-authoring-plan.md
---

# feat: Parallel authoring of all remaining DataStack documentation pages

## Overview

Author all 11 remaining stub and partial documentation pages to IOFMaterialsTutorial depth. Plan is structured for parallel execution: each unit writes exactly one page file, with no shared file writes between concurrent units. A pre-flight unit establishes a passing `mkdocs build --strict` baseline before parallel work begins; a final integration unit verifies the whole site after all pages are complete.

## Problem Frame

The MkDocs documentation site has 6 authored pages, 8 partial pages (content roughed in, gaps documented), and 3 stub pages (placeholders only). Each page has a per-page authoring brief in `docs/specs/content/*.json` encoding the narrative arc, exact reference URLs, required inline assets, reproducible walkthrough commands, and a done-when quality gate. The remaining authoring work can run in parallel because every page is a separate file.

*(see origin: docs/plans/2026-09-24-002-feat-content-authoring-plan.md)*

## Requirements Trace

- R1. Every remaining page reaches IOFMaterialsTutorial depth: all curl commands reproducible as-is with real public URLs, real API responses shown inline, no synthesized examples without `# illustrative example` label.
- R2. Per-page quality gates in `docs/specs/content/*.json` are all checked before a page is marked done.
- R3. `mkdocs build --strict` exits 0 for the full site when all units complete.
- R4. Each page is authored for its primary persona — P1 pages contain no Docker or API jargon; P2 pages use exact env var names and shell commands; P3 pages use IRI terminology and API contract language.
- R5. All authored pages cross-link to each other correctly (no broken anchors or dead links).

## Scope Boundaries

- No changes to `docs/specs/*.json` component spec files (they are source material, not output)
- No changes to authored pages unless a cross-link must be fixed (default-use-case, capability-map, author-a-mapping, standalone-apis, api-endpoints, csvtocsvw)
- No new site infrastructure (navigation, theme, CI) — separate task
- No per-repo README updates in upstream repos

### Deferred to Separate Tasks

- Deploying to GitHub Pages: separate PR after all pages pass
- Per-repo README cross-links in upstream repos: separate PRs per repo
- `docs/examples/` ground-truth example artifacts (Unit 0 from plan 002): can be created alongside or after this plan

## Context & Research

### Authoring Workflow (per page)

1. Read `docs/specs/content/<page-slug>.json`
2. Read all `reference_material.spec_files` listed in the spec
3. Fetch all `reference_material.live_api_calls` — capture real responses
4. Fetch all `reference_material.example_urls` values
5. Follow `narrative_arc.steps` in order — each step → a page section
6. Embed all `inline_assets_required` items — fetch from live source or local fixtures; label `# illustrative example` only if synthesized
7. Execute all `walkthrough_requirements` commands — include command + response verbatim inline
8. Check every `quality_gate` item
9. Run `mkdocs build --strict` — must exit 0

### Key Source Files

- `docs/specs/content/*.json` — 17 authoring briefs, one per page (primary input)
- `docs/specs/datastack.json` — DataStack services, env vars, post-boot steps
- `docs/specs/csvtocsvw-service.json`, `maptomethod-service.json`, `rdfconverter-service.json` — microservice API schemas
- `docs/specs/ckanext-csvtocsvw.json`, `ckanext-csvwmapandtransform.json`, `ckanext-fuseki.json` — extension config keys
- `docs/specs/samm-idta-pipeline-patterns.json` — SAMM/Catena-X two-stage pattern; `concrete_example_pa6gf30` section
- `docs/specs/omeroextractor-service.json`, `openbismantic-service.json` — extractor specs

### Real-World Sources

| Source | What to fetch |
|---|---|
| `https://raw.githubusercontent.com/Mat-O-Lab/DataStack/main/.env.example` | Real env var names and defaults |
| `https://csvtocsvw.matolab.org/info` | Version + config (quickstart, overview) |
| `https://maptomethod.matolab.org/info` | Version (overview) |
| `https://rdfconverter.matolab.org/info` | Version (overview) |
| `https://raw.githubusercontent.com/Mat-O-Lab/CSVToCSVW/main/examples/example2-metadata.json` | CSVW JSON-LD snippet (data-formats) |
| `https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml` | YARRRML snippet (data-formats) |
| `https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-joined.ttl` | Joined Turtle snippet (data-formats) |
| `https://futurecarproduction.materialsdata.space/api/3/action/package_search?q=PA6GF30&rows=1` | Live PA6GF30 dataset (samm-catena-x) |
| Local: `OmeroExtractor/tests/image83.json` and `image83.ttl` | Real fixture data (omeroextractor) |

### Patterns to Follow

- Authored pages as depth templates: `docs/guides/author-a-mapping.md`, `docs/guides/standalone-apis.md` — section structure, code block style, cross-link syntax
- `docs/pipeline/default-use-case.md` — P1 persona language (no Docker/API jargon)
- `docs/reference/api-endpoints.md` — reference page layout (lookup-first, not sequential)
- MkDocs Material: `!!! warning`, `!!! note`, `!!! info` admonition syntax; `=== "Tab"` for tabbed content

## Key Technical Decisions

- **All page units run in parallel**: each unit writes exactly one `.md` file → zero shared write conflicts. Pre-flight and final integration are the only sequential gates.
- **Content spec JSON is the primary brief**: each ce-work agent reads its spec file first, then follows it. Agents must not invent content not in the spec or its referenced sources.
- **Illustrative example policy**: synthesized examples allowed only when live fetch fails; must be labeled `# illustrative example` as a code comment in the same block.
- **mkdocs build --strict per unit**: each unit runs `mkdocs build --strict` at the end — not just `mkdocs build`. Treats warnings as errors; catches broken links before they accumulate.
- **Tier 2 units unblock after Tier 1 completes**: Tier 2 units may cross-link to Tier 1 pages. Running Tier 2 after Tier 1 ensures those targets exist and `mkdocs build --strict` passes within each unit.

## Open Questions

### Resolved During Planning

- **Can all page units truly run in parallel?** Yes — each unit writes exactly one `.md` file. `mkdocs build --strict` per unit is also safe because MkDocs reads all files at build time; partial failures in other files do not block a unit's own build.
- **Do authored pages need re-authoring?** No — they passed their quality gates. Pre-flight includes a verification sweep for authored pages to confirm cross-links resolve; no content rewrite unless a cross-link is broken.
- **OmeroExtractor — no public deployment**: Use local test fixtures at `OmeroExtractor/tests/`. These are real data, not synthesized.
- **OpenBISmantic — no public deployment**: All curl examples use placeholder `https://your-openbismantic.example.org`; labeled clearly.
- **IDTA/AAS page — insufficient spec material**: If `samm-idta-pipeline-patterns.json` does not have AAS-specific YARRRML, keep as a short documented note page with `!!! info "Work in progress"` and cross-link to samm-catena-x. Do not pad with synthesized content.

### Deferred to Implementation

- Exact line count for truncated inline examples — truncate at a natural boundary within the ranges specified in each spec's quality gate
- Whether `navigation.instant` causes Mermaid rendering issues in `mkdocs serve` — test only if enabled; leave disabled by default

## High-Level Technical Design

> *This illustrates the intended parallelization structure and is directional guidance for review, not implementation specification.*

```
Pre-flight (sequential)
  └── mkdocs build --strict → baseline passes
  └── Verify authored pages cross-links resolve

Tier 1 Parallel (all 5 units independent)
  ├── Unit 1: guides/quickstart.md          (P2, fetch .env.example)
  ├── Unit 2: index.md                      (P1, use case gallery)
  ├── Unit 3: pipeline/advanced/samm-catena-x.md  (P3, SPARQL pattern)
  ├── Unit 4: reference/configuration.md    (P2/P4, CKANINI__ table)
  └── Unit 5: reference/data-formats.md     (P3/P4, format snippets)

Tier 2 Parallel (all 6 units independent; start after Tier 1 complete)
  ├── Unit 6: pipeline/overview.md          (P2/P3/P4, master diagram)
  ├── Unit 7: guides/add-a-use-case.md      (P4, decision tree)
  ├── Unit 8: extractors/omeroextractor.md  (P3, local fixtures)
  ├── Unit 9: extractors/openbismantic.md   (P4, placeholder curls)
  ├── Unit 10: pipeline/advanced/idta-aas.md (P3, single-stage pattern)
  └── Unit 11: extractors/ontop.md          (P4, short external tool)

Final integration (sequential)
  └── mkdocs build --strict → full site passes
  └── Fix any cross-link failures
  └── Commit
```

---

## Implementation Units

### Pre-flight

- [ ] **Unit 0: Establish strict baseline and verify authored pages** `[Prerequisite]`

**Goal:** Confirm `mkdocs build --strict` passes on the current site before parallel authoring begins, and verify all 6 authored pages have their cross-links intact.

**Requirements:** R3, R5

**Dependencies:** None — first unit

**Files:**
- No content changes expected; fix scaffold issues if found

**Approach:**
- Run `mkdocs build --strict` from repo root
- If failures: fix broken nav entries, stub pages with missing frontmatter, or dead cross-links — do not rewrite content
- For each authored page, confirm internal links resolve: check `default-use-case.md`, `capability-map.md`, `author-a-mapping.md`, `standalone-apis.md`, `api-endpoints.md`, `csvtocsvw.md`
- Confirm `#troubleshooting` anchor in `author-a-mapping.md` is present (was fixed in a prior session; verify it persists)

**Test scenarios:**
- Happy path: `mkdocs build --strict` exits 0 with no warnings
- Error path: any warning is treated as a failure and fixed before proceeding
- Integration: internal links from authored pages to stub pages (e.g. `capability-map.md → author-a-mapping.md`) resolve without 404

**Verification:**
- `mkdocs build --strict` exits 0
- No `WARNING` lines in build output

---

### Tier 1 — High Priority (Parallel)

- [ ] **Unit 1: `guides/quickstart.md`** `[Must — P2 onboarding]`

**Goal:** Data engineer can deploy DataStack from scratch and see a CSV processed end-to-end, knowing exactly what to verify at each step.

**Requirements:** R1, R2, R4

**Dependencies:** Unit 0

**Files:**
- Modify: `docs/guides/quickstart.md`

**Approach:**
- Read: `docs/specs/content/guides-quickstart.json`
- Fetch: `https://raw.githubusercontent.com/Mat-O-Lab/DataStack/main/.env.example` — extract real variable names and defaults
- Follow narrative arc: prerequisites → clone → configure .env → docker compose up → admin + API token → BACKGROUNDJOBS_API_TOKEN → mappings group → upload test CSV → optional Fuseki
- Every step ends with a `✓ Verify:` line — what the engineer sees when the step succeeded
- `.env` inline snippet must use real variable names from `.env.example`
- `BACKGROUNDJOBS_API_TOKEN` → `!!! warning` admonition (not a bullet)
- `mappings` group name → `!!! note` with case-sensitive label
- Fuseki step marked explicitly optional

**Patterns to follow:**
- `docs/guides/author-a-mapping.md` for step + verify format
- `docs/specs/datastack.json` `post_boot_setup` and `most_common_setup_failure` sections

**Test scenarios:**
- Happy path: all 9 story arc steps present and in order; no steps omitted
- Happy path: inline `.env` snippet has real variable names (not `YOUR_VALUE_HERE` placeholders)
- Happy path: every step has a `✓ Verify:` outcome line
- Edge case: `BACKGROUNDJOBS_API_TOKEN` step is a `!!! warning` admonition — not just a numbered bullet
- Edge case: `mappings` group name has `!!! note` marking it case-sensitive
- Error path: Fuseki step is explicitly optional, clearly distinguishable from required steps

**Verification:**
- `mkdocs build --strict` exits 0
- `!!! warning` admonition present for BACKGROUNDJOBS_API_TOKEN
- Inline `.env` snippet present with real variable names

---

- [ ] **Unit 2: `index.md`** `[Must — P1 landing page]`

**Goal:** Any visitor understands the pipeline principle, can identify which entry point matches them, and sees proof that this is production-proven.

**Requirements:** R1, R2, R4

**Dependencies:** Unit 0

**Files:**
- Modify: `docs/index.md`

**Approach:**
- Read: `docs/specs/content/index.json`
- Keep existing: core principle block quote, audience tab switcher (already present and working)
- Add: use case gallery — table with 3 rows (lab CSV, OMERO microscopy, SAMM/Catena-X), columns: Domain | Source format | Output | Real example link
- Add: proof-of-production — Hanke et al. 2023 DOI link + "26 Catena-X mappings in production" statistic
- P1 language throughout — no Docker, no API jargon in researcher-facing sections
- Tab content already routes audiences correctly; do not restructure tabs

**Patterns to follow:**
- Existing `docs/index.md` tab structure
- `docs/pipeline/default-use-case.md` for P1 prose tone

**Test scenarios:**
- Happy path: use case gallery table has exactly 3 rows with real example links (IOFMaterialsTutorial, BAMresearch DF-TEM-PAW, futurecarproduction.materialsdata.space)
- Happy path: Hanke et al. 2023 DOI URL present as a link (not bare text)
- Happy path: "26 Catena-X mappings" or similar production proof-point present
- Edge case: P1-facing sections contain no Docker, docker-compose, or API endpoint terminology

**Verification:**
- `mkdocs build --strict` exits 0
- Use case gallery table present with 3 rows
- Publication linked

---

- [ ] **Unit 3: `pipeline/advanced/samm-catena-x.md`** `[Should — P3 advanced pattern]`

**Goal:** Ontology engineer understands why SAMM models need two stages, how the SPARQL INSERT bridge works, and can replicate the pattern for their domain.

**Requirements:** R1, R2, R4

**Dependencies:** Unit 0

**Files:**
- Modify: `docs/pipeline/advanced/samm-catena-x.md`

**Approach:**
- Read: `docs/specs/content/pipeline-advanced-samm-catena-x.json`
- Read: `docs/specs/samm-idta-pipeline-patterns.json` — `concrete_example_pa6gf30`, `two_path_comparison`, `execution_mechanism`
- Fetch: `https://futurecarproduction.materialsdata.space/api/3/action/package_search?q=PA6GF30&rows=1` — find mapping resource URL
- Fetch the YARRRML mapping YAML — use first 15 lines as inline snippet (must show `samm:` prefix in prefixes block)
- SPARQL INSERT snippet from `samm-idta-pipeline-patterns.json` `sparql_insert_structure` — one property end-to-end (4 node types)
- Two-stage explanation: Stage 1 = YARRRML to CX intermediate namespace, Stage 2 = SPARQL CONSTRUCT/INSERT to PMDco
- Trigger mechanism: explicitly state it is NOT automated — browser helper page only
- `two_path_comparison` table from spec → comparison table in page

**Patterns to follow:**
- `docs/guides/author-a-mapping.md` for P3 API-contract language
- `docs/specs/samm-idta-pipeline-patterns.json` as sole technical source

**Test scenarios:**
- Happy path: Stage 1 / Stage 2 division explicit with separate named sections
- Happy path: inline YARRRML snippet shows real `samm:` or CX-namespace prefix
- Happy path: inline SPARQL INSERT snippet shows at least one complete property bridge
- Happy path: PA6GF30 live example linked (futurecarproduction.materialsdata.space)
- Edge case: trigger mechanism labeled NOT automated — no automated sync hooks
- Integration: cross-link to `pipeline/advanced/idta-aas.md` (Unit 10) for the single-stage contrast

**Verification:**
- `mkdocs build --strict` exits 0
- Two inline code blocks: YARRRML + SPARQL
- Trigger mechanism explicitly not-automated

---

- [ ] **Unit 4: `reference/configuration.md`** `[Should — P2/P4 operator reference]`

**Goal:** Operator can find any configuration key, its CKANINI__ env var form, its default, and its effect — without hunting through multiple files.

**Requirements:** R1, R2, R4

**Dependencies:** Unit 0

**Files:**
- Modify: `docs/reference/configuration.md`

**Approach:**
- Read: `docs/specs/content/reference-configuration.json`
- Read: `docs/specs/ckanext-csvtocsvw.json`, `docs/specs/ckanext-csvwmapandtransform.json`, `docs/specs/ckanext-fuseki.json`, `docs/specs/datastack.json`
- One section per component: DataStack core .env, ckanext-csvtocsvw, ckanext-csvwmapandtransform, ckanext-fuseki
- Per section: table with columns `Config key (ini form)` | `CKANINI__ env var form` | `Default` | `Description`
- Both ini key and env var form must appear side-by-side in each table row — operators use env var form in `.env`
- CKANINI__ convention already explained in current page — keep and verify it's accurate
- `BACKGROUNDJOBS_API_TOKEN`: document in DataStack core section with the same warning as quickstart
- `db_url` in ckanext-csvwmapandtransform: document with note that it inherits from `sqlalchemy.url`
- `rdfconverter_timeout_check: 30` and `rdfconverter_timeout_create: 120`: include defaults
- ckanext-fuseki: `auto_hooks_state = DISABLED` — document with note that auto sync hooks are commented out

**Patterns to follow:**
- `docs/reference/api-endpoints.md` table style
- `docs/specs/ckanext-csvwmapandtransform.json` `configuration` array as row source

**Test scenarios:**
- Happy path: all 4 component sections present
- Happy path: each table row shows both ini key AND CKANINI__ env var form
- Happy path: ckanext-csvwmapandtransform section has all 9 config keys from spec
- Edge case: `db_url` has note about `sqlalchemy.url` inheritance
- Edge case: ckanext-fuseki auto_hooks_state = DISABLED documented honestly
- Edge case: `BACKGROUNDJOBS_API_TOKEN` in DataStack section is flagged as most common setup failure

**Verification:**
- `mkdocs build --strict` exits 0
- 4 component sections present
- CKANINI__ env form visible in every table row

---

- [ ] **Unit 5: `reference/data-formats.md`** `[Could — P3/P4 format reference]`

**Goal:** Any reader can determine the format at each pipeline stage and see a short inline example of each format.

**Requirements:** R1, R2, R4

**Dependencies:** Unit 0

**Files:**
- Modify: `docs/reference/data-formats.md`

**Approach:**
- Read: `docs/specs/content/reference-data-formats.json`
- 4-row pipeline flow table already present — verify all 4 stages and 8 return_type options are accurate
- Fetch and add inline snippets for each format:
  - CSVW JSON-LD: first 10 lines of `https://raw.githubusercontent.com/Mat-O-Lab/CSVToCSVW/main/examples/example2-metadata.json`
  - YARRRML: first 10 lines of `https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml`
  - Joined Turtle: first 8 lines of `https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-joined.ttl`
- Label each snippet with source URL as a comment
- Note that CSVToCSVW and RDFConverter share the same 8 `return_type` options
- Link W3C CSVW primer when CSVW first introduced

**Patterns to follow:**
- `docs/reference/api-endpoints.md` compact section format

**Test scenarios:**
- Happy path: 4-stage flow table present, accurate stage names and tools
- Happy path: 8 return_type options listed (json-ld, n3, nt, hext, trig, turtle, longturtle, xml)
- Happy path: CSVW JSON-LD snippet fetched from real GitHub URL (not synthesized)
- Happy path: at least 2 of 3 additional format snippets present (YARRRML, joined Turtle)
- Edge case: snippets labeled with source URL comment

**Verification:**
- `mkdocs build --strict` exits 0
- 4-row table present
- At least 2 inline format examples from real GitHub sources

---

### Tier 2 — Remaining Partials and Stubs (Parallel, after Tier 1)

- [ ] **Unit 6: `pipeline/overview.md`** `[Should — P2/P3/P4 architecture]`

**Goal:** Any technical reader can understand the full system topology — all services, their roles, data flows, and the automation boundary.

**Requirements:** R1, R2, R4

**Dependencies:** Unit 0; Tier 1 should be complete (cross-links to Unit 4 configuration page)

**Files:**
- Modify: `docs/pipeline/overview.md`

**Approach:**
- Read: `docs/specs/content/pipeline-overview.json`
- Read: `docs/specs/datastack.json` — `services` array (12 services), `config_convention`
- Fetch: `/info` from all 3 public services for accurate version numbers
- Mermaid diagram: all 12 services, data flow arrows (solid), CKAN automation (dashed), Fuseki manual trigger (labeled)
- Component table: 12 rows, columns: Service | Docker image | Role | Public URL
- Two transformation paths section: direct YARRRML/RML (single-stage) vs. YARRRML + SPARQL CONSTRUCT (two-stage)
- Automation boundary: explicit list of what CKAN automates vs. what is manual
- Cross-link: each service → its component or extractor page

**Patterns to follow:**
- Swim lane Mermaid diagram in `docs/pipeline/default-use-case.md` for diagram style
- `docs/specs/datastack.json` services array for accurate image names and roles

**Test scenarios:**
- Happy path: Mermaid diagram renders and shows all 12 services
- Happy path: component table has all 12 services from datastack.json
- Happy path: dashed vs. solid arrows distinguish automated from manual flows
- Happy path: two transformation paths described as separate sections or table rows
- Integration: all service cross-links in component table resolve

**Verification:**
- `mkdocs build --strict` exits 0
- Mermaid diagram present
- Component table: 12 rows

---

- [ ] **Unit 7: `guides/add-a-use-case.md`** `[Could — P4 extension guide]`

**Goal:** Developer knows exactly what to change to extend the pipeline for a new source type or new target ontology.

**Requirements:** R1, R2, R4

**Dependencies:** Unit 0; Tier 1 should be complete (cross-links to configuration page)

**Files:**
- Modify: `docs/guides/add-a-use-case.md`

**Approach:**
- Read: `docs/specs/content/guides-add-a-use-case.json`
- Decision tree (ASCII or Mermaid): New use case → new source type OR new target ontology — each branch shows what changes and what stays the same
- Path A (new source): four options — deploy existing extractor, extend existing, write new FastAPI service, use RDFConverter direct — each with one-line description and link
- Path B (new target): OntosphereIO → pattern → MapToMethod → mapping → rest unchanged
- Configuration table: which keys to change for each path (formats, plugins, mapping_strategy)
- When to use two-stage SPARQL CONSTRUCT (link to `pipeline/advanced/samm-catena-x.md`)
- Done checklist: extractor produces JSON-LD, rules_skipped == 0, joined Turtle visible in CKAN

**Patterns to follow:**
- `docs/specs/ckanext-csvwmapandtransform.json` for mapping_strategy config key
- `docs/specs/datastack.json` for CKAN__PLUGINS list

**Test scenarios:**
- Happy path: two adaptation paths (source / ontology) as distinct named sections
- Happy path: four source-type options listed under Path A
- Happy path: configuration table shows which keys change per path
- Integration: cross-link to `pipeline/advanced/samm-catena-x.md` resolves
- Integration: cross-link to `reference/configuration.md` (Unit 4) resolves

**Verification:**
- `mkdocs build --strict` exits 0
- Decision tree or two-path structure present
- Configuration table present

---

- [ ] **Unit 8: `extractors/omeroextractor.md`** `[Could — P3/P1 extractor spec]`

**Goal:** Reader understands what OmeroExtractor does, what the real output looks like (from test fixtures), and how the DF-TEM-PAW example connects.

**Requirements:** R1, R2, R4

**Dependencies:** Unit 0

**Files:**
- Modify: `docs/extractors/omeroextractor.md`

**Approach:**
- Read: `docs/specs/content/extractors-omeroextractor.json`
- Read: `docs/specs/omeroextractor-service.json`
- Use local test fixtures: `OmeroExtractor/tests/image83.json` (first 20 lines for JSON-LD snippet) and `OmeroExtractor/tests/image83.ttl` (first 15 lines for Turtle snippet)
- Label fixtures: `# from OmeroExtractor tests/image83.json (v0.0.4)`
- Status badge: `!!! warning "Early stage (v0.0.4) — API may change"` at top
- No public OMERO deployment — state clearly; do not provide placeholder curl examples
- BAMresearch/DF-TEM-PAW linked as real-world production example
- Hanke et al. 2023 DOI linked
- Downstream connection: JSON-LD output → MapToMethod → RDFConverter path

**Patterns to follow:**
- `docs/extractors/openbismantic.md` (Unit 9) output for extractor page structure
- `docs/specs/omeroextractor-service.json` `test_fixtures` section

**Test scenarios:**
- Happy path: status badge present at top (early stage / v0.0.4)
- Happy path: JSON-LD snippet from real test fixture (image83.json), labeled with source
- Happy path: Turtle snippet from real test fixture (image83.ttl), labeled with source
- Happy path: BAMresearch DF-TEM-PAW linked as production example
- Happy path: publication DOI linked
- Edge case: no public deployment note — no placeholder curl examples offered

**Verification:**
- `mkdocs build --strict` exits 0
- Status badge present
- Two inline snippets from test fixtures

---

- [ ] **Unit 9: `extractors/openbismantic.md`** `[Could — P4 extractor spec]`

**Goal:** Developer with an openBIS instance understands how to deploy OpenBISmantic and what it exposes.

**Requirements:** R1, R2, R4

**Dependencies:** Unit 0

**Files:**
- Modify: `docs/extractors/openbismantic.md`

**Approach:**
- Read: `docs/specs/content/extractors-openbismantic.json`
- Read: `docs/specs/openbismantic-service.json`
- Status badge: `!!! info "Demonstrator — no active public deployment"` at top
- Content negotiation curl pair: two curls, same URL, different `Accept` headers. Use placeholder `https://your-openbismantic.example.org`; label clearly: `# replace with your OpenBISmantic URL`
- openBIS hierarchy (Space → Project → Collection → Object → Dataset) as prose with brief explanation
- API endpoints table: 12 endpoints from spec
- RO-Crate export section: `/export_bundle`, link to RO-Crate spec
- Two deployment modes: full stack vs. API-only; show required env vars for each
- Downstream: RO-Crate ZIP → RDFConverter for semantic mapping

**Patterns to follow:**
- `docs/extractors/csvtocsvw.md` for extractor page structure
- `docs/specs/openbismantic-service.json` `api_endpoints`, `deployment_modes`, `configuration` arrays

**Test scenarios:**
- Happy path: status badge present (demonstrator, no public deployment)
- Happy path: content negotiation curl pair with two Accept headers shown
- Happy path: openBIS hierarchy explained
- Happy path: two deployment modes (full stack / API-only) documented
- Happy path: required env vars (OPENBIS_URL, BASE_URL, ADMIN_PASS) listed
- Edge case: all curl placeholder URLs labeled `# replace with your OpenBISmantic URL`

**Verification:**
- `mkdocs build --strict` exits 0
- Status badge present
- Content negotiation curl pair present

---

- [ ] **Unit 10: `pipeline/advanced/idta-aas.md`** `[Could — P3 single-stage pattern]`

**Goal:** Ontology engineer understands why AAS submodels can map in a single stage and how to write the YARRRML.

**Requirements:** R1, R2, R4

**Dependencies:** Unit 0; Unit 3 (samm-catena-x.md) should exist for the contrast cross-link

**Files:**
- Modify: `docs/pipeline/advanced/idta-aas.md`

**Approach:**
- Read: `docs/specs/content/pipeline-advanced-idta-aas.json`
- Read: `docs/specs/samm-idta-pipeline-patterns.json` — look for any `aas_*` keys or AAS-specific sections
- If spec has AAS YARRRML content: include inline snippet (10–15 lines) showing submodelElement iterator
- If spec has no AAS content: write a short honest page — what AAS is, why single-stage works (direct structure-to-PMDco alignment), cross-link to SAMM page for contrast, `!!! info "Detailed walkthrough coming"` admonition
- Single-stage vs. two-stage contrast: one clear paragraph; link to `pipeline/advanced/samm-catena-x.md`
- Do not pad with synthesized AAS YARRRML if no real source exists

**Patterns to follow:**
- `docs/pipeline/advanced/samm-catena-x.md` (Unit 3) output for structure consistency

**Test scenarios:**
- Happy path: single-stage vs. two-stage contrast explained
- Happy path: cross-link to `pipeline/advanced/samm-catena-x.md` resolves
- Edge case: if no AAS YARRRML source available, page uses `!!! info` admonition (not a blank stub)
- Edge case: no synthesized YARRRML without explicit `# illustrative example` label

**Verification:**
- `mkdocs build --strict` exits 0
- Single-stage explanation present
- Cross-link to samm-catena-x resolves

---

- [ ] **Unit 11: `extractors/ontop.md`** `[Could — P4 external tool stub]`

**Goal:** Reader knows Ontop is an external OBDA bridge, when to consider it, and where to find documentation.

**Requirements:** R2, R4

**Dependencies:** Unit 0

**Files:**
- Modify: `docs/extractors/ontop.md`

**Approach:**
- Read: `docs/specs/content/extractors-ontop.json`
- Short page — no more than one screen of content
- `!!! info "External tool — not maintained by Mat-O-Lab"` at top
- One paragraph: what Ontop does (virtual SPARQL over SQL, no ETL, R2RML/OBDA mapping)
- One paragraph: how it could connect to DataStack (Ontop SPARQL endpoint as JSON-LD source for downstream mapping)
- Links: Ontop official docs (`https://ontop-vkg.org/`), W3C R2RML spec
- One sentence: "No Mat-O-Lab integration built yet — contributions welcome"
- Do not invent integration steps or configuration that does not exist

**Patterns to follow:**
- Keep as brief as `docs/extractors/openbismantic.md` status section

**Test scenarios:**
- Happy path: external tool admonition present
- Happy path: Ontop official docs linked
- Happy path: honest "no integration built" statement present
- Edge case: page does not contain fabricated configuration or curl examples

**Verification:**
- `mkdocs build --strict` exits 0
- External tool admonition present
- Ontop official docs linked

---

### Final Integration

- [ ] **Unit 12: Full site integration and commit** `[Required]`

**Goal:** Full site passes `mkdocs build --strict` with all 17 pages authored, all cross-links resolved, and the commit lands on `feat/mkdocs-documentation-site`.

**Requirements:** R3, R5

**Dependencies:** All Units 0–11

**Files:**
- Modify: any files with broken cross-links discovered during final build
- No content rewrites — cross-link fixes only at this stage

**Approach:**
- Run `mkdocs build --strict` from repo root
- For any `WARNING - Doc file '...' contains a link '...', but there is no such anchor`: fix the anchor or the link — choose whichever preserves intent
- For any `WARNING - Doc file '...' contains a link to '...', but it is not found`: fix the target path
- After all warnings cleared, run `mkdocs build --strict` once more to confirm clean
- Commit all changes: include summary of which pages were authored/completed in commit body

**Test scenarios:**
- Happy path: `mkdocs build --strict` exits 0 with zero warnings
- Integration: every cross-link between authored pages resolves — spot-check: default-use-case → author-a-mapping, quickstart → configuration, samm-catena-x → idta-aas, capability-map → all extractor pages
- Integration: Mermaid diagram in `pipeline/overview.md` renders (verify with `mkdocs serve`)

**Verification:**
- `mkdocs build --strict` exits 0
- Zero WARNING lines in build output
- Commit created on `feat/mkdocs-documentation-site`

---

## System-Wide Impact

- **Interaction graph:** MkDocs resolves all internal links at build time — a broken anchor in any page causes `--strict` failure. Each unit's per-unit `mkdocs build --strict` check catches its own failures; Unit 12 catches cross-unit link failures.
- **Unchanged invariants:** `docs/specs/*.json` component files are read-only for all units — no unit modifies spec files. Authored pages (6 pages) are not rewritten unless a cross-link must be fixed.
- **API surface parity:** None — this is documentation authoring, not code changes.

## Risks & Dependencies

| Risk | Mitigation |
|---|---|
| Public services down during authoring (csvtocsvw, maptomethod, rdfconverter) | Use pre-computed examples from GitHub raw (example2-metadata.json, measurements-map.yaml, measurements-joined.ttl). Label illustrative only if source URL not available at all |
| DataStack .env.example not at expected URL | Fall back to `docs/specs/datastack.json` env var list; label any synthesized values clearly |
| futurecarproduction.materialsdata.space PA6GF30 search returns no results | Use samm-idta-pipeline-patterns.json `concrete_example_pa6gf30` section directly; note live instance URL without fetching |
| OmeroExtractor test fixtures missing from local repo | Verify fixture paths before Unit 8 starts; if missing, write a short prose description of the JSON-LD structure from spec |
| IDTA/AAS spec has no real YARRRML source | Write honest short page with `!!! info` admonition rather than synthesized content |
| Tier 2 cross-links to Tier 1 pages fail during per-unit `mkdocs build --strict` | Per-unit build catches this; fix cross-link before marking unit done. Unit 12 is final safety net |

## Documentation / Operational Notes

- After Unit 12 commits, merge `feat/mkdocs-documentation-site` → `main` to trigger GitHub Actions deployment to GitHub Pages (requires GitHub Pages to be configured: repo Settings → Pages → source = `gh-pages` branch)
- One-time setup: add `CKAN_SITE_URL` and other deploy variables to GitHub Actions secrets if not already present

## Sources & References

- **Origin document:** [docs/plans/2026-09-24-002-feat-content-authoring-plan.md](docs/plans/2026-09-24-002-feat-content-authoring-plan.md)
- **Content spec plan:** [docs/plans/2026-09-24-003-feat-content-specs-plan.md](docs/plans/2026-09-24-003-feat-content-specs-plan.md)
- **Requirements:** [docs/brainstorms/documentation-site-requirements.md](docs/brainstorms/documentation-site-requirements.md)
- **Per-page briefs:** `docs/specs/content/*.json` (17 files)
- **Component specs:** `docs/specs/*.json` (10 files)
- **Depth benchmark:** [IOFMaterialsTutorial](https://github.com/Mat-O-Lab/IOFMaterialsTutorial)
- **Live instances:** https://futurecarproduction.materialsdata.space, https://dataportal.material-digital.de
