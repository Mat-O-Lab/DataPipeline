# DataStack Documentation Plan

## Goal

Build compelling documentation for the Mat-O-Lab DataStack pipeline — covering its default use case (CSV → semantic knowledge graph) and the full range of possibilities the modular architecture enables.

### Core Pipeline Principle (must open every doc)

> **Create complete, consistent metadata for a non-semantic resource — then use semantic technologies (RML/YARRRML) to transform that metadata further.**

Two stages, always:

1. **Metadata creation** — a source-specific tool reads a non-semantic resource (CSV file, ELN record, image server, SQL database, …) and produces complete, consistent, structured metadata as JSON-LD or RDF. This step bridges the gap between the native format and the semantic world.
2. **Semantic transformation** — RML mapping rules (authored in human-readable YARRRML) connect that metadata to a target ontology and produce a self-contained knowledge graph.

Everything else in the stack is automation and infrastructure around these two steps.

### Derived architectural properties

- **Ontology agnostic** — the target ontology lives entirely in the YARRRML mapping and template graph. The pipeline infrastructure is never touched when changing ontologies.
- **Resource-type extensible** — any tool that can produce complete JSON-LD/RDF metadata for a source type can plug into step 1. CSVToCSVW, OpenBISmantic, OmeroExtractor, Ontop, or anything else.
- **Microservice-first** — all transformation services are standalone REST APIs. CKAN extensions add automation on top, but are not required.

## Source Material

Per-component specs are in `docs/specs/`:

| File | Component |
|---|---|
| `specs/datastack.md` | DataStack Docker Compose orchestration |
| `specs/csvtocsvw-service.md` | CSVToCSVW microservice |
| `specs/maptomethod-service.md` | MapToMethod microservice |
| `specs/rdfconverter-service.md` | RDFConverter microservice |
| `specs/ckanext-csvtocsvw.md` | ckanext-csvtocsvw CKAN extension |
| `specs/ckanext-csvwmapandtransform.md` | ckanext-csvwmapandtransform CKAN extension |
| `specs/ckanext-fuseki.md` | ckanext-fuseki CKAN extension |
| `specs/openbismantic-service.md` | OpenBISmantic extractor (OpenBIS ELN → JSON-LD) — TODO |
| `specs/omeroextractor-service.md` | OmeroExtractor (Omero image server → JSON-LD) — TODO |
| `specs/ontop-integration.md` | Ontop SQL→RDF bridge — external tool, integration notes — TODO |
| `specs/samm-idta-pipeline-patterns.md` | SAMM/Catena-X + IDTA/AAS two-stage transformation patterns |

Live CKAN instances with real pipeline examples:
- https://futurecarproduction.materialsdata.space/
- https://dataportal.material-digital.de/

---

## Documentation Structure

```
docs/
  plan.md                          ← this file
  specs/                           ← per-component specs (done)
  pipeline/
    overview.md                    ← full pipeline diagram + narrative
    default-use-case.md            ← CSV → CSVW → mapping → RDF → Fuseki walkthrough
    capability-map.md              ← what the pipeline can/cannot do
  guides/
    quickstart.md                  ← deploy DataStack, upload CSV, see the magic
    author-a-mapping.md            ← MapToMethod workflow for domain experts
    add-a-use-case.md              ← adapt pipeline to a new domain
    standalone-apis.md             ← use microservices without CKAN
  reference/
    api-endpoints.md               ← consolidated API reference (all 3 microservices)
    configuration.md               ← all config keys in one place
    data-formats.md                ← input/output formats at each stage
```

---

## Phase 1 — Pipeline Overview Documents

### `pipeline/overview.md`

Content:
- Architecture diagram (Mermaid): all services + data flows
- Component roles in one table
- Entry points: CKAN UI, direct microservice REST APIs, CKAN API
- Automation vs manual steps

### `pipeline/default-use-case.md`

Walk through the exact sequence for a CSV upload in the DataStack:

```
User uploads CSV to CKAN
  → ckanext-csvtocsvw detects format
  → CSVToCSVW annotates → CSVW JSON-LD added to dataset
  → CSVW imported to CKAN DataStore → queryable via Data API
  → ckanext-csvtocsvw triggers JSON-LD → Turtle conversion
  → ckanext-csvwmapandtransform scans `mappings` group
  → RDFConverter /api/checkmapping runs on all YAML mappings
  → Best match (exact strategy) selected
  → RDFConverter /api/createrdf → joined Turtle added to dataset
  → (Manual) fuseki_update → Fuseki dataset created (UUID-matched)
  → RDF uploaded as named graphs
  → SPARQL link resource created → users can query via Sparklis/YASGUI
```

Include: screenshots from live instances, example CKAN dataset JSON, example CSVW output, example joined Turtle snippet.

### `pipeline/capability-map.md`

Organise by **resource type**, not by use case. The central question is: "I have data in system X — can I get it into the pipeline?"

#### Resource Type Matrix

**Stage 1 — Metadata creation (source-type extractors):**

| Resource Type | Extractor | Output ontologies | Status | Notes |
|---|---|---|---|---|
| CSV / TSV / ASC lab files | CSVToCSVW v1.3.5 | CSVW, QUDT, OA, PROV-O | Production | Full CKAN automation via ckanext-csvtocsvw |
| OMERO microscopy image server | OmeroExtractor v0.0.4 | OME, OA, QUDT, PROV-O | Early stage | Public deployment; example: BAMresearch/DF-TEM-PAW; publication: doi:10.1007/s40192-023-00331-5 |
| OpenBIS ELN/LIMS | OpenBISmantic | Schema.org, DCAT, PROV-O | Demonstrator | Requires own openBIS; also produces RO-Crate |
| SAMM / Catena-X aspect model JSON | YARRRML mapping (Stage 1) | CX/SAMM namespace (intermediate) | Production | 26 mappings in live instance; then SPARQL CONSTRUCT bridges to PMDco/AutoMatCE |
| IDTA / AAS submodel JSON | YARRRML mapping (direct) | PMDco | Production | Single-stage; AAS structure maps directly |
| SQL databases | Ontop (external) | Any ontology (OBDA) | External tool | R2RML/OBDA; SPARQL-over-SQL; no Mat-O-Lab integration built yet |
| Any existing RDF / JSON-LD | — (direct ingest) | Already compatible | Built-in | ckanext-csvwmapandtransform checks any RDF format |
| JSON / XML (arbitrary) | RDFConverter direct | Any (via YARRRML) | Built-in | Manual YARRRML mapping; no semantic enrichment step |

**Stage 2 — Semantic transformation paths:**

| Path | Mechanism | Input | Output | When to use |
|---|---|---|---|---|
| YARRRML/RML (direct) | RDFConverter | Data + YARRRML mapping | Target ontology RDF | Source data maps directly to target ontology |
| YARRRML/RML + SPARQL INSERT | RDFConverter then Fuseki SPARQL Update | Data → intermediate RDF (in Fuseki) → INSERT query | Target ontology named graph in Fuseki | Source has its own ontology (SAMM/CX) that differs from target; INSERT reads both data graph + schema graph to produce PMDco; triggered manually via browser HTML helper page stored as CKAN resource |

**Pattern authoring (template graphs / patterns):**

| Terminology | Note |
|---|---|
| **pattern** | Current PMDCO term for what was previously called "graph prototype" |
| ~~graph prototype~~ | Obsolete — do not use |
| OntosphereIO | Recommended tool for building patterns; browser-only, AI-assisted, full OWL2DL reasoning + PMDCO autoshapes (SHACL). Repo: https://github.com/ThHanke/ontosphere |
| PMDCO pattern library | Reference patterns: https://github.com/materialdigital/core-ontology/tree/main/patterns/ |
| IOFMaterialsTutorial | Outdated (draw.io, old terminology) — concepts only, not tooling |

Each row = one entry point into the pipeline. After the extractor step, all resource types flow through the same MapToMethod → RDFConverter → Fuseki path.

#### What the Pipeline CAN do

- Ontology agnostic — template graph + YARRRML define the target ontology; change them, change the ontology
- Fully automated end-to-end for CSV (ckanext-csvtocsvw handles extraction + import)
- Auto-mapping discovery: one mapping covers all future uploads with matching structure
- SPARQL querying across all RDF resources in a dataset (Fuseki union default graph)
- OWL/RDFS inference (Fuseki reasoning modes)
- Standalone microservice use — all three REST APIs work without CKAN
- DCAT-compliant metadata harvesting
- 8 RDF serialization formats at every stage

#### What it CANNOT do (current limitations)

- Fuseki auto-sync hooks are commented out — Fuseki upload requires manual trigger
- Private/internal data URLs need upload endpoints, not URL-based endpoints
- No built-in data validation before mapping — bad input may produce empty graphs silently (`exact` strategy)
- No knowledge graph versioning — Fuseki dataset is deleted and recreated on each sync
- MapToMethod requires publicly accessible URLs for both data and template
- No streaming or incremental updates — full re-sync on every change
- Non-CSV extractors (OpenBISmantic, OmeroExtractor) require separate deployment and CKAN extension wiring

---

## Phase 2 — User Guides

### `guides/quickstart.md`

Audience: data managers / researchers who want to deploy DataStack.

Steps:
1. Clone DataStack repo
2. Configure `.env`
3. `docker compose up`
4. Create admin user + API token
5. Set `BACKGROUNDJOBS_API_TOKEN` + restart
6. Create `mappings` group
7. Upload a CSV → watch the automation happen
8. (Optional) trigger Fuseki sync, open SPARQL endpoint

### `guides/author-a-mapping.md`

Audience: domain experts who want to connect their data to an ontology.

Steps:
1. Have a CSVW JSON-LD (from CSVToCSVW) and a template ontology/graph
2. Open MapToMethod UI at `maptomethod.matolab.org`
3. Use `/api/types` to explore template types
4. Use `/api/entities` to enumerate columns and template entities
5. Build the entity `map`
6. `POST /api/mapping` → download YARRRML
7. Validate: upload to RDFConverter `/api/checkmapping` — check `rules_skipped == 0`
8. Optional: test with `/api/test` for detailed output preview
9. Upload YARRRML to CKAN `mappings` group as a YAML resource
10. Future CSVs with same structure will auto-transform

### `guides/add-a-use-case.md`

Audience: developers adapting the pipeline to a new domain.

Cover:
- What to change (formats lists, mapping strategy, ontology)
- How to design a template graph for a new domain
- How to extend the `mappings` group with new mappings
- When to use `best_match` vs `exact` strategy
- How to use microservices standalone (without CKAN)

### `guides/standalone-apis.md`

Audience: developers who want to use the microservices outside CKAN.

Cover:
- CSVToCSVW: annotate any CSV → CSVW with a single HTTP POST
- MapToMethod: generate YARRRML from any JSON-LD + template graph
- RDFConverter: run any YARRRML mapping against any data
- Chaining the three services manually
- Example curl commands for each endpoint
- Using upload endpoints for private data

---

## Phase 3 — Reference

### `reference/api-endpoints.md`

Consolidated table of all API endpoints across all three microservices: method, path, purpose, key params, response shape.

### `reference/configuration.md`

All configuration keys for all three CKAN extensions and DataStack `.env`, organized by component.

### `reference/data-formats.md`

What format flows between each stage of the pipeline:
- CSV → CSVW (JSON-LD)
- CSVW → Turtle
- Turtle + YARRRML → joined Turtle
- Turtle → Fuseki named graphs

---

## Priority Order

1. `pipeline/overview.md` — the anchor document everything links to
2. `pipeline/default-use-case.md` — most compelling narrative, uses live instances for examples
3. `guides/quickstart.md` — highest user demand
4. `guides/author-a-mapping.md` — key for adoption (mapping authoring is the main barrier)
5. `pipeline/capability-map.md` — important for setting expectations
6. `reference/*` — can be generated largely from specs
7. `guides/add-a-use-case.md` + `guides/standalone-apis.md` — later

---

## Live Instance Examples to Use

Fetch from CKAN API:
```
GET https://futurecarproduction.materialsdata.space/api/3/action/package_list
GET https://futurecarproduction.materialsdata.space/api/3/action/package_show?id=<id>
GET https://dataportal.material-digital.de/api/3/action/package_list
```

Look for datasets that have: original CSV + CSVW JSON-LD + joined Turtle + SPARQL resource — those are the best end-to-end examples.
