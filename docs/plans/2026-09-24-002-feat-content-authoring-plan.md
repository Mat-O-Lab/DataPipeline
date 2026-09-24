---
title: "feat: Author remaining DataStack documentation pages"
type: feat
status: active
date: 2026-09-24
origin: docs/plan.md
---

# feat: Author Remaining DataStack Documentation Pages

## Overview

Fill in the six highest-priority stub pages with real content — inline examples from live repos and CKAN instances, each page shaped around a story outline before content is written, and each audience approached with a distinct voice and entry point.

The goal is not technically complete reference pages. The goal is pages that make a new user say "I understand what this does and I know how to start."

## Problem Frame

The site scaffold is live. Stubs exist for all pages. The gap is: content that teaches, not just describes. Each audience needs a different story — a researcher needs to see the data transformation without needing to understand Docker; a data engineer needs the exact steps and environment variables; an ontology engineer needs the API contracts.

*(see origin: docs/plan.md)*

## Audience Personas

These personas drive tone, depth, and what examples to use on every page. Each page targets one primary persona; secondary personas may be noted.

| ID | Name | Background | Goal | What they don't need |
|---|---|---|---|---|
| **P1 — Researcher** | Lab scientist or data manager | Familiar with their domain data (CSV measurements, microscopy, material properties); not a developer | "Does this pipeline handle my data? What does the output look like?" | Docker internals, API parameters, RML syntax |
| **P2 — Data engineer** | DevOps / backend engineer | Comfortable with Docker Compose, REST APIs, YAML | "Deploy this stack, configure it correctly, get it running end-to-end without surprises" | Ontology theory, provenance terminology |
| **P3 — Ontology engineer** | Semantic web specialist | Familiar with OWL, SPARQL, YARRRML, ontology patterns | "What APIs does this expose? What are the exact input/output contracts? How does the mapping work?" | Step-by-step deploy instructions |
| **P4 — Developer/integrator** | Software developer extending the stack | Familiar with FastAPI, CKAN plugin system, Docker | "How do the components fit together? How do I extend this for a new data source?" | High-level pipeline narrative |

## Requirements Trace

- R1. `pipeline/default-use-case.md` — story arc for P1 (researcher), inline CSVW and Turtle examples from live CKAN
- R2. `guides/quickstart.md` — step-by-step deploy for P2 (data engineer), inline `.env` snippets, inline verification outputs
- R3. `guides/author-a-mapping.md` — mapping authoring workflow for P3 (ontology engineer), inline API requests and responses
- R4. `pipeline/capability-map.md` — resource type matrix for P1, capability table for all personas
- R5. `reference/api-endpoints.md`, `reference/configuration.md`, `reference/data-formats.md` — reference for P3 and P4, derived from spec files with inline examples
- R6. `guides/standalone-apis.md`, `guides/add-a-use-case.md` — P4 focused, inline curl examples

## Content Authoring Principle

Every unit follows a three-phase process:

1. **Story outline** — define the narrative arc: what does the reader know at the start, what do they need to understand by the end, what is the logical step sequence with no gaps
2. **Research inline examples** — fetch real examples from live CKAN instances, GitHub repos, and public API endpoints to use as inline code blocks
3. **Write** — follow the story outline, weaving in the inline examples for every key concept

The spec files are the source of truth for technical facts. The story outline and inline examples are what make those facts learnable.

## Scope Boundaries

- No screenshots — prose and inline code are sufficient and more portable
- No per-repo README updates — separate tasks per prior plan (2026-09-24-001)
- No OpenAPI rendering — link to `/api/docs` URLs
- `extractors/` pages (omeroextractor, openbismantic, ontop) — spec files incomplete; out of scope
- Advanced pipeline pages (`samm-catena-x.md`, `idta-aas.md`) — deferred below

### Deferred to Separate Tasks

- `docs/extractors/omeroextractor.md`, `docs/extractors/openbismantic.md`, `docs/extractors/ontop.md` — spec files marked TODO
- `pipeline/advanced/samm-catena-x.md`, `pipeline/advanced/idta-aas.md` — `docs/specs/samm-idta-pipeline-patterns.md` has full content; medium-priority follow-on
- Per-repo README cross-links — separate PRs per upstream repo

## Context & Research

### Relevant Source Files

- `docs/plan.md` — content outlines, step sequences, table definitions for every page
- `docs/specs/datastack.md` — env vars, post-boot steps, service topology
- `docs/specs/csvtocsvw-service.md` — API endpoints, output format list
- `docs/specs/ckanext-csvtocsvw.md` — Path 1 and Path 2 processing logic
- `docs/specs/ckanext-csvwmapandtransform.md` — mapping discovery, selection strategies
- `docs/specs/ckanext-fuseki.md` — fuseki_update flow, named graph conventions
- `docs/specs/maptomethod-service.md` — mapping authoring workflow, API endpoints
- `docs/specs/rdfconverter-service.md` — /api/checkmapping, /api/createrdf, /api/test
- `docs/references.md` — live CKAN instances, public microservice URLs, publication DOI
- `docs/specs/samm-idta-pipeline-patterns.md` — capability-map background

### Live Research Sources (fetch during each unit)

| Source | What to fetch | Use in unit |
|---|---|---|
| `https://dataportal.material-digital.de/api/3/action/package_show?id=cross-project-use-case` | CSVW JSON-LD resource URL, joined Turtle resource URL | Unit 1 inline examples |
| `https://csvtocsvw.matolab.org/api/docs` | Endpoint parameter docs | Unit 5 reference |
| `https://maptomethod.matolab.org/api/docs` | Endpoint parameter docs | Units 3, 5 |
| `https://rdfconverter.matolab.org/api/docs` | Endpoint parameter docs | Units 3, 5 |
| `https://github.com/Mat-O-Lab/DataStack` | Sample `.env.example` or README for real variable names | Unit 2 |
| PMDCO pattern library `https://github.com/materialdigital/core-ontology/tree/main/patterns/` | One pattern example (Turtle snippet) | Unit 3 |

### External References — Use, Don't Rewrite

These are well-documented upstream resources. Content pages must **link to them** rather than paraphrasing or re-explaining them. One sentence + a link beats three paragraphs of re-explanation.

| Topic | Resource | What to link to |
|---|---|---|
| YARRRML syntax / rules | https://rml.io/yarrrml/spec/ | Rule syntax, prefixes, iterators — link when introducing YARRRML |
| YARRRML interactive editor | https://rml.io/yarrrml/matey/ | Use to create/test mapping examples; link from author-a-mapping |
| YARRRML tutorial | https://rml.io/yarrrml/tutorial/ | Step-by-step intro — link from author-a-mapping and standalone-apis |
| W3C CSVW primer | https://www.w3.org/TR/tabular-data-primer/ | Link when first introducing CSVW output format |
| RML spec | https://rml.io/specs/rml/ | For P3 audience wanting RML details; link from reference pages |
| QUDT units ontology | https://qudt.org/ | For unit annotations in CSVW — link once from default-use-case |
| PMDCO core ontology | https://github.com/materialdigital/core-ontology | Link when mentioning PMDco as a target ontology |
| PMDCO pattern library | https://github.com/materialdigital/core-ontology/tree/main/patterns/ | Template graph examples — link from author-a-mapping |
| OntosphereIO | https://github.com/ThHanke/ontosphere | Recommended pattern authoring tool — link from author-a-mapping, capability-map |
| PROV-O | https://www.w3.org/TR/prov-o/ | Link once when provenance annotations appear in output |
| CSVToCSVW OpenAPI | https://csvtocsvw.matolab.org/api/docs | Primary reference for all CSVToCSVW endpoints |
| MapToMethod OpenAPI | https://maptomethod.matolab.org/api/docs | Primary reference for all MapToMethod endpoints |
| RDFConverter OpenAPI | https://rdfconverter.matolab.org/api/docs | Primary reference for all RDFConverter endpoints |
| Hanke et al. 2023 | https://link.springer.com/article/10.1007/s40192-023-00331-5 | Peer-reviewed pipeline description — cite from index.md and default-use-case |

**Authoring rule:** If explaining a concept already covered by one of these resources requires more than 3 sentences, replace it with a one-sentence summary and a link.

### Existing Patterns to Follow

- `docs/pipeline/overview.md` — section structure, cross-linking style, Mermaid usage
- `docs/index.md` — `===` tabbed content, block quote, comparison table style
- Stub pages use frontmatter `title:` + `!!! info "Coming soon"` — replace admonition completely; keep frontmatter

## Key Technical Decisions

- **Story outline before writing:** Every unit begins by drafting the narrative arc (what the reader knows → what they learn → how they verify). The outline is written in comments or scratch notes, not committed — it shapes the content but doesn't appear in the final page.
- **Inline examples are mandatory:** No content page is complete without at least one inline code block showing a real input, a real API response, or a real output artifact. Synthesized examples are acceptable only when live data is unavailable — label them `# illustrative example`.
- **Persona-matched language:** P1 pages avoid Docker and API terminology. P2 pages use shell commands and env var names directly. P3 pages use precise API contract language. P4 pages assume framework familiarity.
- **Link upstream docs, don't re-explain them:** YARRRML, RML, CSVW, QUDT, PROV-O, and PMDco are all well-documented externally. Use the External References table. One sentence + a link is always better than a paragraph of re-explanation.
- **`mkdocs build --strict` after each unit:** Treats warnings as errors — catches broken links before they accumulate.
- **`navigation.instant` check:** If Mermaid diagrams break in `mkdocs serve` after writing Unit 1 (which may add a sequence diagram), disable `navigation.instant` in `mkdocs.yml`.

## Open Questions

### Resolved During Planning

- **Primary example source for Unit 1:** `dataportal.material-digital.de` dataset `cross-project-use-case` — full pipeline run available via CKAN API
- **Which transformation path for `default-use-case.md`:** Path 1 (CSV → CSVW → Turtle → RDFConverter → Fuseki) only — P1 audience
- **Template graph source for Unit 3:** PMDCO pattern library — one real pattern snippet as the template example
- **Mapping strategy to cover in Unit 3:** Explain `exact` as default, `best_match` for iteration

### Deferred to Implementation

- Exact CSVW JSON-LD snippet length — truncate at natural boundary, around 20–30 lines
- Whether a Mermaid flow diagram improves `default-use-case.md` over a numbered list — decide during story outline phase
- Whether `navigation.instant` should be disabled — test during `mkdocs serve` in Unit 1

## Implementation Units

- [ ] **Unit 0: Annotated example artifacts**

**Goal:** Prepare a small set of focused, annotated code examples that are explained line by line — used as the teaching material across all content pages. Created once, referenced everywhere.

**Requirements:** Foundation for R1–R6 (all units use these examples)

**Dependencies:** None — this is the first unit

**Files:**
- Create: `docs/examples/sample.csv` — the input CSV
- Create: `docs/examples/sample.csvw.json` — the CSVW JSON-LD output for sample.csv
- Create: `docs/examples/sample.yarrrml.yaml` — one complete annotated YARRRML mapping
- Create: `docs/examples/sample-joined.ttl` — the RDF output produced by the mapping
- Create: `docs/examples/checkmapping-response.json` — example checkmapping API response

**Story outline:**
```
[Reader encounters these examples before or during reading any content page]
Each example is self-contained and annotated with comments explaining each part.
Each example is short enough to read fully (20–40 lines max).
```

**Approach:**

`sample.csv` — a minimal 5-column, 5-row materials science measurement file:
- Columns: `sample_id`, `temperature_C`, `tensile_strength_MPa`, `yield_strength_MPa`, `elongation_pct`
- 3–4 data rows
- This is the canonical input that every pipeline example traces through

`sample.csvw.json` — the CSVW JSON-LD that CSVToCSVW produces for `sample.csv`:
- Fetch from `POST https://csvtocsvw.matolab.org/api/annotate` with `data_url` pointing to a raw GitHub URL of `sample.csv` once uploaded, OR construct by hand following the CSVW spec structure the service produces
- Annotate every key line with a comment: `# QUDT unit annotation`, `# column IRI`, `# provenance record`
- Keep to 40 lines max — truncate with `// ... (truncated)`

`sample.yarrrml.yaml` — one complete YARRRML mapping connecting `sample.csvw.json` columns to a PMDco pattern:
- Fetch one real YARRRML mapping from `https://futurecarproduction.materialsdata.space/` (any CSV mapping in the `mappings` group via CKAN API) OR use the YARRRML tutorial as reference
- Annotate each section: `# prefixes block`, `# mapping subject`, `# predicate-object pair`, `# iterator`
- Show exactly one complete mapping rule end-to-end (not a multi-hundred-line file)
- 30–40 lines max

`sample-joined.ttl` — the Turtle output produced by running the mapping:
- 20–30 lines showing one subject with its quality, datum, and unit value
- Annotate with `# PMDco quality`, `# QUDT unit`, `# provenance statement`

`checkmapping-response.json` — what a successful checkmapping call returns:
```json
{
  "rules_applicable": 4,
  "rules_skipped": 0
}
```
And a failing example (rules_skipped > 0) with explanation comment.

**Research to do:**
1. POST `https://csvtocsvw.matolab.org/api/annotate` with a real CSV URL (use `https://raw.githubusercontent.com/Mat-O-Lab/DataStack/main/` or any publicly accessible CSV) — capture real CSVW JSON-LD output
2. `GET https://futurecarproduction.materialsdata.space/api/3/action/package_search?q=csv&groups=mappings&rows=5` — find a CSV mapping YAML resource, fetch it, extract one annotated rule
3. Alternatively, use `https://rml.io/yarrrml/matey/` to create a minimal YARRRML mapping from `sample.csv` to a simple ontology property

**Patterns to follow:**
- Keep examples short enough to explain in a single prose paragraph
- Annotation comments use `#` for YAML/Turtle/CSV, `//` for JSON (or inline prose in the page)
- Each example file should be self-contained — no external imports needed to understand it

**Test expectation:** none — these are static example files, not behavioral code. Verify: `mkdocs build --strict` exits 0 (files are in `docs/examples/` which must be added to `exclude` in `mkdocs.yml` if they shouldn't appear in nav, or added to nav under a new "Examples" section)

**Verification:**
- All 5 files exist and are ≤ 40 lines
- `sample.yarrrml.yaml` shows exactly one complete rule with annotation comments
- `sample-joined.ttl` traces back to properties visible in `sample.csvw.json`

---

- [ ] **Unit 1: `pipeline/default-use-case.md`**

**Primary persona:** P1 — Researcher  
**Goal:** Researcher reads the page and understands exactly what happens to their CSV file and what they get back — without needing to know any Docker or API details

**Requirements:** R1

**Dependencies:** None

**Files:**
- Modify: `docs/pipeline/default-use-case.md`

**Story outline:**
```
[Reader knows: they have a CSV file with measurements and want FAIR data]
→ What is "default use case" — CSV from a lab instrument
→ What happens automatically (zero-click pipeline steps 1–7)
→ What they see in the CKAN interface at each step
→ Real example: show actual CSVW JSON-LD (truncated) for a lab CSV
→ What mapping is and why it's needed (one sentence — link to author-a-mapping)
→ Real example: show actual joined Turtle output (truncated)
→ What they can do with the SPARQL endpoint
→ Where to go next (link to quickstart, overview, capability-map)
[Reader knows: the pipeline flow end-to-end, what their data looks like at each stage]
```

**Research to do before writing:**
1. `GET https://dataportal.material-digital.de/api/3/action/package_show?id=cross-project-use-case` — find the JSON-LD resource download URL and Turtle resource download URL
2. Fetch the JSON-LD resource (first 30 lines) → use as the CSVW inline example
3. Fetch the joined Turtle resource (first 20 lines) → use as the joined Turtle inline example
4. Note the resource names and dataset structure visible in the CKAN UI (describe in prose)

**Approach:**
- Open with the core principle block quote (consistent with `docs/index.md`)
- Automation chain as a numbered flow: each step names the component, what it does, and what artifact appears in the CKAN dataset afterward. P1 language: "CKAN automatically creates a metadata file", not "ckanext-csvtocsvw calls /api/annotate"
- Insert CSVW JSON-LD code block (`json`) at step 2 — show the column annotations including QUDT unit IRI
- Insert joined Turtle code block (`turtle`) at step 5 — show subject + quality + unit value triples
- Step 7 (Fuseki) is **manual** — say it clearly, explain why (link to capability-map limitations)
- "What you get" section: CKAN dataset with 4 resources, a SPARQL endpoint, queryable data
- "What's next" links: `../guides/quickstart.md`, `../guides/author-a-mapping.md`, `../pipeline/capability-map.md`

**Patterns to follow:**
- `docs/pipeline/overview.md` for section structure; `docs/index.md` for tone with P1 audience
- `docs/specs/ckanext-csvtocsvw.md` Path 1 and Path 2 for accuracy

**Test scenarios:**
- Happy path: automation chain has all steps from `docs/plan.md` lines 88–98; none omitted
- Happy path: inline CSVW code block shows at least one column with QUDT unit annotation
- Happy path: inline Turtle code block shows RDF triples with unit/value
- Edge case: fuseki_update step explicitly labeled as **manual** — not automatic
- Edge case: page contains zero references to Docker, `docker compose`, or internal service names — P1 language only
- Integration: all cross-links resolve in `mkdocs serve`

**Verification:**
- `mkdocs build --strict` exits 0
- Two inline code blocks present (CSVW + Turtle)
- fuseki manual step clearly marked
- No Docker terminology on page

---

- [ ] **Unit 2: `guides/quickstart.md`**

**Primary persona:** P2 — Data engineer  
**Goal:** Engineer reads the page and can deploy a running DataStack from scratch in one session, knowing exactly what to verify at each step

**Requirements:** R2

**Dependencies:** None

**Files:**
- Modify: `docs/guides/quickstart.md`

**Story outline:**
```
[Reader knows: they want to deploy DataStack; they have Docker Compose]
→ Prerequisites (2 minutes): Docker Compose, a domain/IP, Git
→ Clone + first look at .env.example — what to fill in
→ docker compose up — what to expect in the logs
→ Create admin + API token — exact UI path
→ Set BACKGROUNDJOBS_API_TOKEN — why jobs don't run without it
→ Create mappings group — exact name, why it matters
→ Upload a test CSV — what appears in the dataset
→ Optional: trigger Fuseki, open SPARQL
[Reader knows: stack is running, CSV processed, SPARQL available]
```

**Research to do before writing:**
1. Check `https://github.com/Mat-O-Lab/DataStack` README or `.env.example` for the real variable names and defaults to show in the inline snippet
2. Note the key `CKANINI__` variables that must be set for the pipeline to work (from `docs/specs/datastack.md`)
3. Identify what log output a successful `docker compose up` produces (describe from `docs/specs/datastack.md` service list)

**Approach:**
- Prerequisites box at top: Docker Compose v2, public-facing URL or LAN access, Git
- Inline `.env` snippet showing the 5–6 variables that must be set before first boot (minimal set for a working stack)
- Each step ends with a "✓ Verify" line — what the engineer sees when the step succeeded. Example: "✓ Verify: CKAN home page loads at http://localhost:5000"
- `BACKGROUNDJOBS_API_TOKEN` gets its own warning admonition — the most common setup failure
- `mappings` group creation: exact name, case-sensitive — note in an `!!! note`
- CSV upload step: describe what the engineer sees in the CKAN dataset (resources tab shows 3–4 auto-created resources)
- P2 language: use exact env var names, exact shell commands, exact UI paths

**Patterns to follow:**
- `docs/specs/datastack.md` Post-Boot Setup for step accuracy
- `docs/components/datastack.md` Key Environment Variables table

**Test scenarios:**
- Happy path: all 8 steps from `docs/plan.md` lines 169–178 present and in order
- Happy path: inline `.env` snippet present with real variable names
- Happy path: every step has a "✓ Verify" indicator
- Edge case: `BACKGROUNDJOBS_API_TOKEN` is a warning admonition — not just a bullet point
- Edge case: `mappings` group name is marked case-sensitive with an `!!! note`
- Error path: step 8 (Fuseki) is optional and clearly not required for the basic pipeline

**Verification:**
- `mkdocs build --strict` exits 0
- Inline `.env` snippet present
- All 8 steps accounted for
- `BACKGROUNDJOBS_API_TOKEN` warning admonition present

---

- [ ] **Unit 3: `guides/author-a-mapping.md`**

**Primary persona:** P3 — Ontology engineer  
**Goal:** Ontology engineer can author a YARRRML mapping for their data and upload it to CKAN, understanding each API call and its expected output

**Requirements:** R3

**Dependencies:** Unit 2 recommended as prerequisite for readers (not file dependency)

**Files:**
- Modify: `docs/guides/author-a-mapping.md`

**Story outline:**
```
[Reader knows: their CSV data has a CSVW; they have an ontology pattern in Turtle]
→ What MapToMethod does (one paragraph — the "compiler" metaphor: data structure + ontology pattern → mapping rules)
→ Prerequisites: CSVW URL, pattern/template Turtle URL, both publicly accessible
→ Step 1: Explore data types — GET /api/types (inline response: list of type URIs)
→ Step 2: Explore data entities — GET /api/entities (inline response: column dict)
→ Step 3: Explore template entities — same endpoints on template URL
→ Step 4: Build the map dict — show a real map dict example
→ Step 5: Generate the mapping — POST /api/mapping (inline: first 10 lines of resulting YARRRML)
→ Step 6: Validate — POST /api/checkmapping → rules_skipped == 0 (inline: response JSON)
→ Step 7: Debug if needed — POST /api/test (inline: brief TestMappingResult snippet)
→ Step 8: Upload to CKAN mappings group
→ "What happens next" — auto-transform for all future matching uploads
[Reader knows: the full mapping authoring cycle with real API shapes at every step]
```

**Research to do before writing:**
1. `GET https://maptomethod.matolab.org/api/types?url=<example_csvw_url>` — fetch a real type list response as inline JSON example
2. `GET https://maptomethod.matolab.org/api/entities?url=<example_csvw_url>` — fetch a real entities response (first 5 entries) as inline JSON example
3. Check PMDCO pattern library (`https://github.com/materialdigital/core-ontology/tree/main/patterns/`) — find one short pattern (Turtle, 15–20 lines) to use as template example
4. Find a real YARRRML mapping in `https://futurecarproduction.materialsdata.space/` (any SAMM or CSV mapping) to show first 15 lines as inline YAML example
5. `POST https://rdfconverter.matolab.org/api/checkmapping` with example → show response `{rules_applicable, rules_skipped}`

**Approach:**
- "Compiler" metaphor in the intro: MapToMethod takes the shape of your data (CSVW columns) and the shape of your target ontology (pattern entities) and writes the mapping rules for you
- Each step includes: the API call, the inline request/response, and a one-sentence explanation of what to do with the result
- The `map` dict step gets a mini-example showing real column names mapped to real IRI values
- `rules_skipped == 0` is the quality gate — inline the actual JSON response so the reader knows exactly what to look for
- Troubleshooting section: `rules_skipped > 0` means column names in the map don't match the data — use `best_match` temporarily, compare entity names
- P3 language: use IRI terminology, API parameter names, YARRRML vocabulary

**Patterns to follow:**
- `docs/specs/maptomethod-service.md` Typical Workflow
- `docs/specs/rdfconverter-service.md` /api/checkmapping and /api/test

**Test scenarios:**
- Happy path: all 8 steps in story outline are covered in order
- Happy path: inline JSON for `/api/types` response present
- Happy path: inline JSON for `/api/entities` response present (at least 3 columns shown)
- Happy path: inline YARRRML snippet present (first ~10 lines of a real mapping)
- Happy path: inline `/api/checkmapping` response JSON showing `rules_skipped: 0`
- Edge case: troubleshooting section present for `rules_skipped > 0` case
- Error path: "both URLs must be publicly accessible" prerequisite is prominent — not buried

**Verification:**
- `mkdocs build --strict` exits 0
- At least 4 inline code blocks (types response, entities response, YARRRML snippet, checkmapping response)
- Troubleshooting section present

---

- [ ] **Unit 4: `pipeline/capability-map.md`**

**Primary persona:** P1 (researcher scanning), P3 (ontology engineer reading in detail)  
**Goal:** Any reader can determine in under 2 minutes whether the pipeline handles their data source and what outcome to expect

**Requirements:** R4

**Dependencies:** None

**Files:**
- Modify: `docs/pipeline/capability-map.md`

**Story outline:**
```
[Reader knows: they have a data source and want to know if the pipeline supports it]
→ Intro: "The pipeline is resource-type agnostic — any extractor that produces JSON-LD can plug in"
→ Stage 1 matrix: 8 rows, one per resource type — extractor, output ontologies, production status
→ Stage 2 paths: 2 rows — direct YARRRML/RML vs two-stage YARRRML + SPARQL CONSTRUCT
→ Pattern authoring: OntosphereIO vs deprecated tools (graph prototype → pattern terminology)
→ CAN section: 8 bullet points
→ CANNOT section: 7 bullet points including fuseki manual trigger clearly named
[Reader knows: whether their use case is supported and what limitations to plan around]
```

**Research to do before writing:**
- No new live research needed — `docs/plan.md` lines 108–161 has the full table content
- Verify extractor status ("Production" / "Early stage" / "External tool") against `docs/specs/` files before copying
- Cross-check CANNOT items against actual spec limitations (not outdated)

**Approach:**
- Intro: one paragraph, P1 language — "You bring the data; the pipeline brings the transformation"
- Stage 1 matrix: copy from `docs/plan.md` lines 111–119, link extractor column entries to extractor pages
- Stage 2 paths: compact 2-row table, link to `pipeline/overview.md#two-transformation-paths`
- Pattern authoring table: include the "graph prototype → pattern" terminology note prominently
- CAN section: tight bullet list, no sub-bullets
- CANNOT section: equally tight, `fuseki_update` manual trigger is item 1 (most common surprise)

**Patterns to follow:**
- `docs/pipeline/overview.md` table style
- `docs/plan.md` lines 108–161 as primary source

**Test scenarios:**
- Happy path: Stage 1 matrix has exactly 8 rows, all from `docs/plan.md`
- Happy path: CANNOT section lists fuseki manual trigger as a limitation
- Edge case: "pattern" used, not "graph prototype"
- Integration: all extractor links in the matrix resolve in `mkdocs serve`

**Verification:**
- `mkdocs build --strict` exits 0
- 8-row Stage 1 matrix present
- CAN and CANNOT sections both present

---

- [ ] **Unit 5: Reference pages — api-endpoints, configuration, data-formats**

**Primary persona:** P3 (ontology engineer), P4 (developer)  
**Goal:** Complete reference tables derived from spec files, with inline examples for every non-obvious entry

**Requirements:** R5

**Dependencies:** Best written after Units 1–3 because those pages cross-link here

**Files:**
- Modify: `docs/reference/api-endpoints.md`
- Modify: `docs/reference/configuration.md`
- Modify: `docs/reference/data-formats.md`

**Story outlines:**

`api-endpoints.md`:
```
[Reader knows: they want to call a specific endpoint directly]
→ One section per service (CSVToCSVW, MapToMethod, RDFConverter)
→ Per service: table of endpoints; inline curl example for the primary endpoint
→ Link to live /api/docs for full OpenAPI spec
```

`configuration.md`:
```
[Reader knows: they want to configure or debug a specific setting]
→ Intro: CKANINI__ convention (one short explanation + example)
→ One section per component: DataStack .env, ckanext-csvtocsvw, ckanext-csvwmapandtransform, ckanext-fuseki
→ Per section: table with ini key, CKANINI__ env form, default, description
```

`data-formats.md`:
```
[Reader knows: they want to understand what format flows between stages]
→ One table: Stage | Input | Tool | Output | Formats available
→ 4 rows covering the full pipeline
→ Inline snippet per row: one short example of the output format
```

**Research to do before writing:**
1. Fetch `https://csvtocsvw.matolab.org/api/openapi.json` — extract endpoint list and key parameters
2. Fetch `https://maptomethod.matolab.org/api/openapi.json` — same
3. Fetch `https://rdfconverter.matolab.org/api/openapi.json` — same
4. One curl example per service (POST /api/annotate for CSVToCSVW, GET /api/types for MapToMethod, POST /api/checkmapping for RDFConverter)

**Approach:**
- `api-endpoints.md`: group by service with `###` subheading; table has Method / Path / Purpose / Key params / Response; inline curl example per service using the public matolab.org URLs
- `configuration.md`: show `CKANINI__` env form in a `code` column alongside the ini key — this is the form operators actually use; show the DataStack compose snippet for each extension's key variables
- `data-formats.md`: 4-row pipeline flow table; note that CSVToCSVW and RDFConverter share the same 8 `return_type` options

**Patterns to follow:**
- `docs/components/datastack.md` table style
- Spec files as authoritative source — do not paraphrase keys or values

**Test scenarios:**
- Happy path: `api-endpoints.md` covers all 3 services; total endpoints match spec files (CSVToCSVW: 3, MapToMethod: 3, RDFConverter: 6)
- Happy path: `configuration.md` has both ini key and CKANINI__ env form for each extension
- Happy path: `data-formats.md` has 4-stage flow; 8 output formats listed
- Happy path: at least one inline curl example per service in `api-endpoints.md`
- Edge case: upload endpoints documented separately from URL-based endpoints

**Verification:**
- `mkdocs build --strict` exits 0
- Three inline curl examples in api-endpoints.md
- CKANINI__ env forms visible in configuration.md

---

- [ ] **Unit 6: `guides/standalone-apis.md` and `guides/add-a-use-case.md`**

**Primary persona:** P4 — Developer / integrator  
**Goal:** Developer can use any microservice without CKAN, and can adapt the pipeline for a new data domain

**Requirements:** R6

**Dependencies:** Units 1–5 (cross-links)

**Files:**
- Modify: `docs/guides/standalone-apis.md`
- Modify: `docs/guides/add-a-use-case.md`

**Story outlines:**

`standalone-apis.md`:
```
[Reader knows: they want to run pipeline steps without CKAN]
→ Why standalone: lighter setup, CI/CD pipelines, scripted batch processing
→ Per service: one curl command (the minimal working call), what it returns, link to /api/docs
→ Chaining the three services: curl pipeline showing CSV → CSVW → YARRRML → RDF as a shell script
→ Upload endpoints for private data (not publicly accessible)
```

`add-a-use-case.md`:
```
[Reader knows: they want to extend the pipeline for a new data source or target ontology]
→ Two dimensions: new source type vs new target ontology
→ New source: choose or write an extractor → produces JSON-LD → rest of pipeline unchanged
→ New target ontology: write a template pattern in OntosphereIO → rest of pipeline unchanged
→ Configuration: formats list, exact vs best_match, plugin list
→ When to use SPARQL CONSTRUCT stage 2 vs direct YARRRML/RML
```

**Research to do before writing:**
1. Real curl chain example: use public matolab.org endpoints to construct a working 3-step shell script (CSV URL → annotate → mapping → createrdf)
2. Confirm which `CKAN__PLUGINS` values are needed for a new CKAN extension (from `docs/specs/datastack.md`)

**Approach:**
- `standalone-apis.md`: code-heavy page; each service has a named section with one `bash` code block showing the complete curl call; chain section has a multi-line shell script
- `add-a-use-case.md`: decision-tree structure; two main paths (new source vs new ontology) with clear branching points; configuration table at the end showing which keys to change

**Patterns to follow:**
- `docs/guides/quickstart.md` (Unit 2 output) for guide tone
- `docs/specs/rdfconverter-service.md` for curl parameters

**Test scenarios:**
- Happy path: `standalone-apis.md` has a curl example for each of the 3 services
- Happy path: chaining shell script present showing full CSV → RDF pipeline
- Happy path: `add-a-use-case.md` covers both "new source" and "new target ontology" paths
- Integration: all cross-links to reference pages resolve in `mkdocs serve`

**Verification:**
- `mkdocs build --strict` exits 0
- Three curl examples in `standalone-apis.md`
- Chaining script present
- Two adaptation paths (source / ontology) in `add-a-use-case.md`

---

## System-Wide Impact

- **Interaction graph:** `mkdocs build --strict` after every unit — broken links accumulate silently with soft build, fail loudly with strict. Run strict from Unit 1 onward.
- **Unchanged invariants:** `docs/specs/` files remain authoritative; content pages surface and link to spec material — they do not replace it.
- **Persona consistency:** If a page is written for P1 (researcher) but starts explaining Docker internals, it's wrong. Review each page's primary persona before marking done.
- **Inline examples are required:** A unit without at least one inline code block is incomplete.

## Risks & Dependencies

| Risk | Mitigation |
|---|---|
| Live CKAN API returns 503 during Unit 1 research | Fall back to a synthetic 20-line CSVW snippet based on spec; label `# illustrative example` |
| Public matolab.org service down during Unit 3/5/6 research | Note endpoint shape from spec + OpenAPI JSON; label example as illustrative |
| Inline examples become stale as services update | Examples are labeled with version comments (e.g., `# CSVToCSVW v1.3.5`); update when specs update |
| Story outline expands into full tutorial scope | Scope guard: each unit's story outline ends with a "Reader knows" statement — if the final "knows" sounds like a full course objective, cut it |

## Documentation / Operational Notes

- Run `mkdocs build --strict` (not plain `mkdocs build`) — catches broken cross-links
- After all units: merge `feat/mkdocs-documentation-site` to `main` → GitHub Actions deploys to GitHub Pages
- One-time GitHub Pages setup: repo Settings → Pages → source = `gh-pages` branch

## Sources & References

- **Origin document:** [docs/plan.md](docs/plan.md)
- Prior plan: [docs/plans/2026-09-24-001-feat-documentation-site-mkdocs-plan.md](docs/plans/2026-09-24-001-feat-documentation-site-mkdocs-plan.md)
- Component specs: `docs/specs/` (all files)
- All external URLs: [docs/references.md](docs/references.md)
- Live CKAN API for examples: `https://dataportal.material-digital.de/api/3/action/package_show?id=cross-project-use-case`
- Public microservice OpenAPI JSONs: csvtocsvw.matolab.org, maptomethod.matolab.org, rdfconverter.matolab.org
