# SAMM / IDTA Pipeline Patterns — Specification

**Live examples:** https://futurecarproduction.materialsdata.space  
**SAMM mapping datasets:** 26 packages (pattern: `samm-mapping-*`)  
**AAS/IDTA mapping dataset:** `aas-mapping-inspection-documents-of-steel-products`  
**Sample payloads + Fuseki:** `edcar_json_payloads` (29 JSON payloads, 30 pre-computed joined graphs, live SPARQL endpoint)

---

## Context

The pipeline is not limited to lab CSV files. The futurecarproduction.materialsdata.space instance demonstrates two additional resource types from automotive Industry 4.0 standards:

- **SAMM / Catena-X aspect model payloads** — flat JSON documents conforming to Eclipse Tractus-X semantic aspect models
- **IDTA / AAS submodels** — hierarchical JSON (Asset Administration Shell format, IEC 63278) describing digital twin aspects

Both are non-semantic data sources that require the same two-step pipeline principle: **create complete metadata → transform with semantic technologies**.

---

## Pattern A: SAMM / Catena-X Aspect Models — Two-Stage Transformation

SAMM (Semantic Aspect Meta-Model) is the Catena-X standard for automotive supply-chain data exchange. Each aspect model is published as a Turtle ontology (eclipse-tractusx/sldt-semantic-models). Payloads are flat JSON conforming to one model (e.g., `io.catenax.material_data:1.0.0`).

The pipeline applies a **two-stage transformation** because the source ontology (SAMM/CX) and the target ontology (PMDco, AutoMatCE) are structurally different:

### Stage 1: JSON payload → SAMM-aligned intermediate RDF (YARRRML/RML)

A YARRRML mapping (`.yaml`) uses JSONPath iterators to traverse the flat JSON payload and produce RDF triples in the SAMM aspect model's own vocabulary (CX namespace). Subjects get deterministic URIs. No unit conversion yet.

**Input:** Flat JSON (e.g., `materialName`, `density`, `youngsModulus`, `glassTransitionTemperature`)  
**Tool:** RDFConverter (`/api/createrdf` with the YARRRML mapping)  
**Output:** SAMM-patterned intermediate RDF graph  

### Stage 2: SAMM RDF → target ontology (SPARQL CONSTRUCT)

A SPARQL CONSTRUCT query reads the intermediate RDF and emits target-ontology-aligned triples. Each mapping dataset contains **two CONSTRUCT variants**:

| Query file | Target ontology |
|---|---|
| `*.samm-to-pmdco.construct.sparql` | PMD Core Ontology (materials science) |
| `*.samm-to-automatce.construct.sparql` | AutoMatCE ontology |

**Architecture of the CONSTRUCT queries:**
- `VALUES` table maps 16 SAMM properties (density, Young's modulus, glass transition temperature, etc.) to their target class, QUDT unit, numeric conversion factor, and semantic role (`quality`, `identifier`, `information`, `temporal_interval`, `measurement_datum`)
- `WHERE` clause: extracts literals from SAMM entities, applies regex numeric parsing, performs unit conversions
- IRI strategy: SHA256 hash-based deterministic IRIs
- `CONSTRUCT` clause: emits BFO/OBO-grounded triples — qualities bearing on material entities (`obo:RO_0000080`), part-whole relations (`obo:BFO_0000051`), scalar value specifications (OBI numeric/categorical)

**Input:** SAMM-patterned intermediate RDF (in Fuseki or via SPARQL endpoint)  
**Tool:** Any SPARQL engine (Fuseki); INSERT to persist results  
**Output:** PMDco- or AutoMatCE-aligned knowledge graph  

### Resources stored in CKAN per SAMM mapping dataset

| Resource | Format | Role |
|---|---|---|
| `*.json-to-samm.yaml` | YAML | Stage 1 YARRRML mapping |
| SAMM aspect model (link) | TTL | Upstream ontology (external GitHub) |
| `*.samm-to-pmdco.construct.sparql` | SPARQL | Stage 2 CONSTRUCT → PMDco |
| `*.samm-to-automatce.construct.sparql` | SPARQL | Stage 2 CONSTRUCT → AutoMatCE |

---

## Pattern B: IDTA / AAS Submodels — Single-Stage YARRRML/RML

IDTA (Industrial Digital Twin Association) governs the Asset Administration Shell (AAS) standard (IEC 63278). AAS submodels are hierarchical JSON with typed submodel elements: `SubmodelElementCollection`, `Property`, with elements identified by `idShort`.

Because AAS JSON is already semantically structured (standardized metamodel vocabulary), a single YARRRML mapping can navigate the hierarchy and map **directly** to PMDco — no intermediate ontology stage needed.

### Mapping approach

Multiple JSONPath iterators, each filtering by element type and `idShort` value, extract individual properties. Output follows PMDco patterns directly.

**Example (Inspection Documents of Steel Products):**
- Central material resource typed as a physical material object
- Mechanical properties (tensile strength, yield strength, elongation) as PMD quality instances with QUDT values in MegaPascals
- Chemical composition as a PMD composition specification with 9 element-specific mass fractions (C, Cr, Mn, Mo, Ni, N, P, Si, S) linked via OBO part-of relations

**Input:** AAS submodel JSON  
**Tool:** RDFConverter (`/api/createrdf` with the YARRRML mapping)  
**Output:** PMDco-aligned knowledge graph directly  

---

## Concrete Example: Cross-Project Use Case (PA6GF30 / Catena-X → PMDco)

**Dataset:** https://dataportal.material-digital.de/dataset/cross-project-use-case  
**Material:** PA6GF30 (30% glass-filled polyamide 6)  
**Source:** Catena-X industrial dataspace via Eclipse Dataspace Connector (EDC)  
**Notebook:** `cross_project_usecase.ipynb` (orchestrates all steps end-to-end)

### Full 4-Step Workflow

```
Step 1 — Retrieve
  Pull PA6GF30 JSON payload from Catena-X dataspace participant via EDC

Step 2 — JSON → RDF (RDFConverter)
  Input:  PA6GF30 JSON + YARRRML mapping (SAMM Stage 1)
  Output: material_data_test_pa6gf30-joined.ttl (96 KB, SAMM-aligned RDF)
  Tool:   RDFConverter /api/createrdf

Step 3 — RDF → PMDco (SPARQL INSERT)
  Input:  Fuseki dataset containing:
    - ?dataGraph: the joined TTL from Step 2 (mat:MaterialData instances)
    - ?schemaGraph: MaterialDataAM3010.ttl (SAMM aspect model schema)
  Query:  pmdco-mapping-insert.sparql
  Output: named graph <pmdco> inside the Fuseki dataset
  Tool:   pmdco-mapping-insert.html (browser helper — see below)

Step 4 — Export
  Output: pmdco-full.ttl — standalone PMDco knowledge graph
```

### SPARQL INSERT Query Structure

The query (`pmdco-mapping-insert.sparql`) reads from two named graphs discovered by pattern — no hardcoded graph IRIs:

- **`?dataGraph`** — `mat:MaterialData` instances with literal property values (from Step 2)
- **`?schemaGraph`** — SAMM `samm:Property` definitions with unit and datatype metadata

For each matched numeric property, it mints four nodes into `GRAPH <pmdco>`:

| Node | Type | Role |
|---|---|---|
| `?material` | `obo:BFO_0000040` | Physical material entity |
| `?quality` | PMDco quality class (e.g. `pmd:PMD_0000952`) | Quality inhering in the material |
| `?datum` | `obo:OBI_0001931` | Scalar measurement datum |
| `?qv` | `qudt:QuantityValue` | Numeric value + unit |

Linking: `obo:RO_0000086` (has quality), `obo:RO_0000052` (characteristic of), `obo:IAO_0000417` (is quality measured as), `obo:OBI_0001938` (has value specification).

**13 properties mapped:** stressAtBreak, flexuralStrength, youngsModulus, flexuralModulus, strainAtBreak, impactStrength, density, meltingTemperature, glassTransitionTemperature, humidity, waterAbsorption, linearThermalExpansionCoefficientParallel, linearThermalExpansionCoefficientTransverse.

**Unit conversion:** `mat:kiloJoulePerSquareMeter` × 1000 → `unit:J-PER-M2`. All others map 1:1.

**IRI strategy:** string concatenation from source entity IRI + property local name (deterministic, no hashing needed because SAMM source IRIs are already stable).

### Execution Mechanism: Browser HTML Helper

`pmdco-mapping-insert.html` is a small browser-side page stored as a CKAN resource. It is **not automated** — it requires a human to:

1. Open the HTML page in a browser
2. Verify the pre-filled values:
   - **Endpoint:** `https://dataportal.material-digital.de/dataset/{uuid}/fuseki/$/update` (Fuseki SPARQL Update)
   - **SPARQL file URL:** download URL of `pmdco-mapping-insert.sparql` from CKAN
3. Click "Run"

Internally: JavaScript `runInsert()` fetches the `.sparql` file from CKAN, then `POST`s it to the Fuseki update endpoint with `Content-Type: application/sparql-update`. Displays HTTP status and response.

**Prerequisites before running:** both `?dataGraph` and `?schemaGraph` must already be loaded into the Fuseki dataset (by triggering `fuseki_update` from ckanext-fuseki for the respective resources).

---

## Two Transformation Paths Compared

| Aspect | YARRRML/RML (Stage 1 or standalone) | SPARQL CONSTRUCT (Stage 2) |
|---|---|---|
| Input | Raw data (JSON, CSV, XML, RDF) | Existing RDF graph |
| Output | RDF knowledge graph | Reshaped / re-ontologized RDF graph |
| Primary purpose | Data → RDF lifting (annotation) | Ontology → ontology transformation |
| Property mapping | Field-by-field JSON key → RDF predicate | Parameterized VALUES table (many properties at once) |
| IRI strategy | Template-based | SHA256 hash-based |
| Unit handling | None (preserves raw values) | Regex parsing + conversion factors in VALUES |
| Trigger | RDFConverter API | SPARQL engine (Fuseki, any SPARQL 1.1 endpoint) |
| Storage in CKAN | YAML resource in `mappings` group | SPARQL resource in dataset |

**Key insight:** SPARQL CONSTRUCT is the ontology-translation layer. When source data already has an established ontology (SAMM/CX) but the target platform uses a different one (PMDco), CONSTRUCT handles the bridge without touching the original mapping or data.

---

## Scale of the SAMM Use Case

26 SAMM mapping datasets on futurecarproduction.materialsdata.space cover the full automotive materials lifecycle:
material properties, composition, batch, sustainability, recycling, vehicle information, component characteristics, simulation, homologation, dismantling, circular economy strategies, and feedback-to-design.

Each dataset follows the same 4-resource structure — YARRRML + SAMM TTL link + 2 CONSTRUCT queries. This demonstrates the pipeline's strength: **one consistent pattern, replicated across an entire data standard domain**.

---

## Implications for Documentation

- The pipeline handles **both data annotation** (CSV/JSON → RDF) and **ontology translation** (RDF → different RDF) as first-class capabilities
- SAMM/Catena-X integration shows industrial applicability beyond materials-science labs
- 26 aspect model mappings = ready-made FAIR data infrastructure for automotive supply chains
- Two-stage pattern can be generalized: any domain with an established ontology (e.g., PROV-O, Schema.org, SAMM/CX) can be bridged to any target ontology via a CONSTRUCT layer
