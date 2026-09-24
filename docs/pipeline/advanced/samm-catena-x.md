---
title: SAMM / Catena-X Pipeline Pattern
---

# SAMM / Catena-X Pipeline Pattern

Two-stage transformation for automotive supply-chain data: flat JSON payloads conforming to Catena-X SAMM aspect models → SAMM-aligned intermediate RDF → PMDco / AutoMatCE knowledge graph.

This pattern is used for all 26 `samm-mapping-*` datasets on [futurecarproduction.materialsdata.space](https://futurecarproduction.materialsdata.space) and covers domains from material properties and composition through sustainability, recycling, and circular economy strategies.

---

## Why two stages?

Direct YARRRML-to-PMDco is straightforward when source data has no established domain ontology — you map literals directly to your target classes. SAMM aspect models break this assumption: the source JSON already conforms to a published, versioned ontology (`io.catenax.material_data:1.0.0` and siblings) with its own class hierarchy, property names, and unit vocabulary.

Trying to produce PMDco triples in a single YARRRML pass would mean embedding the entire ontology translation logic inside JSONPath expressions. The two-stage pattern separates the concerns cleanly:

| Stage | What it does | Tool |
|---|---|---|
| 1 | Lift flat JSON to SAMM-aligned intermediate RDF (preserves source semantics) | RDFConverter `/api/createrdf` + YARRRML |
| 2 | Translate intermediate RDF to target ontology (PMDco / AutoMatCE) | SPARQL CONSTRUCT/INSERT on Fuseki |

For the single-stage contrast — when AAS JSON maps directly to PMDco without an intermediate ontology step — see [IDTA / AAS Submodel Pipeline](idta-aas.md).

---

## Stage 1 — JSON payload → SAMM-aligned intermediate RDF

**Tool:** RDFConverter `/api/createrdf`

**Inputs:**

- Flat JSON payload conforming to a Catena-X SAMM aspect model (e.g. `io.catenax.material_data:1.0.0`)
- YARRRML `.yaml` mapping file stored as a CKAN resource in the `mappings` group

**Output:** SAMM-patterned intermediate RDF graph (e.g. `material_data_test_pa6gf30-joined.ttl`, 96 KB)

The mapping uses JSONPath iterators to traverse the flat JSON and emit RDF triples in the SAMM/CX namespace with deterministic subject URIs. The key characteristic is that the output stays in the CX namespace — it is not yet PMDco.

### YARRRML mapping structure

The snippet below shows the characteristic prefix block and iterator structure. The `samm:` prefix anchors the mapping to the Eclipse Tractus-X SAMM meta-model; `mat:` is the aspect-model-specific namespace for the `MaterialData` entity.

```yaml
# illustrative example — structure matches real PA6GF30 mappings on futurecarproduction.materialsdata.space
prefixes:
  rr:   'http://www.w3.org/ns/r2rml#'
  rml:  'http://semweb.mmlab.be/ns/rml#'
  ql:   'http://semweb.mmlab.be/ns/ql#'
  samm: 'urn:bamm:io.openmanufacturing:meta-model:1.0.0#'
  mat:  'urn:bamm:io.catenax.material_data:1.0.0#'
  cx:   'https://w3id.org/catenax/ontology/core#'
  qudt: 'http://qudt.org/schema/qudt/'
  unit: 'http://qudt.org/vocab/unit/'
  xsd:  'http://www.w3.org/2001/XMLSchema#'

sources:
  materialData:
    access: '$(data_url)'
    referenceFormulation: jsonpath
    iterator: '$.materialData[*]'

mappings:
  MaterialData:
    sources: [materialData]
    s: mat:MaterialData_$(materialId)
    po:
      - [a, mat:MaterialData]
      - [mat:materialId,    $(materialId)]
      - [mat:density,       $(density), xsd:decimal]
      - [mat:youngsModulus, $(youngsModulus), xsd:decimal]
```

Every property name in the `po` block matches the SAMM aspect model schema exactly — this is what lets Stage 2 join the data graph with the schema graph.

---

## Stage 2 — SAMM RDF → target ontology (SPARQL CONSTRUCT/INSERT)

**Tool:** SPARQL engine (Fuseki)

**Inputs (two named graphs in Fuseki):**

| Graph | Contents |
|---|---|
| `dataGraph` | `mat:MaterialData` instances with literal property values — the Stage 1 output |
| `schemaGraph` | `samm:Property` definitions with unit and datatype metadata — the SAMM aspect model schema TTL |

**Output:** Named graph `<pmdco>` inside the Fuseki dataset (exportable as `pmdco-full.ttl`)

### SPARQL INSERT structure

For each property, the CONSTRUCT clause emits four interconnected nodes:

```sparql
PREFIX pmd:  <https://w3id.org/pmd/co/>
PREFIX obo:  <http://purl.obolibrary.org/obo/>
PREFIX qudt: <http://qudt.org/schema/qudt/>
PREFIX unit: <http://qudt.org/vocab/unit/>
PREFIX mat:  <urn:bamm:io.catenax.material_data:1.0.0#>
PREFIX xsd:  <http://www.w3.org/2001/XMLSchema#>

INSERT {
  GRAPH <pmdco> {

    # Node 1 — physical material entity (BFO material)
    ?material a obo:BFO_0000040 .

    # Node 2 — quality inhering in the material
    ?quality a pmd:PMD_0000952 ;              # e.g. Density quality class
             obo:RO_0000080 ?material .       # inheres in

    # Node 3 — scalar measurement datum
    ?datum   a obo:OBI_0001931 ;
             obo:RO_0000052 ?quality ;         # characteristic of
             obo:IAO_0000417 ?quality .        # is quality measured as

    # Node 4 — numeric quantity value + unit
    ?qv      a qudt:QuantityValue ;
             qudt:numericValue ?numVal ;
             qudt:unit         unit:KiloGM-PER-M3 ;
             obo:OBI_0001938   ?datum .        # has value specification
  }
}
WHERE {
  GRAPH ?dataGraph {
    ?materialInst a mat:MaterialData ;
                  mat:density ?rawVal ;
                  mat:materialId ?id .
  }
  # Bind deterministic SHA256-based IRIs
  BIND(IRI(CONCAT("https://example.org/material/", SHA256(STR(?id))))   AS ?material)
  BIND(IRI(CONCAT("https://example.org/quality/density/", SHA256(STR(?id)))) AS ?quality)
  BIND(IRI(CONCAT("https://example.org/datum/density/",  SHA256(STR(?id)))) AS ?datum)
  BIND(IRI(CONCAT("https://example.org/qv/density/",     SHA256(STR(?id)))) AS ?qv)
  # Unit conversion: raw value already in kg/m³ → factor 1
  BIND(xsd:decimal(?rawVal) AS ?numVal)
}
```

The real `pmdco-mapping-insert.sparql` repeats this four-node block for all 13 properties:
`stressAtBreak`, `flexuralStrength`, `youngsModulus`, `flexuralModulus`, `strainAtBreak`, `impactStrength`, `density`, `meltingTemperature`, `glassTransitionTemperature`, `humidity`, `waterAbsorption`, `linearThermalExpansionCoefficientParallel`, `linearThermalExpansionCoefficientTransverse`.

One unit conversion is non-trivial: `mat:kiloJoulePerSquareMeter × 1000 → unit:J-PER-M2` (impact strength). All other properties use factor 1.

IRI strategy throughout is SHA256 hash-based for full determinism across repeated inserts.

---

## CKAN dataset structure

Each `samm-mapping-*` dataset on the portal carries four resource types:

| Resource | Format | Role |
|---|---|---|
| `*.json-to-samm.yaml` | YAML | Stage 1 YARRRML mapping |
| SAMM aspect model | TTL | Upstream ontology (external GitHub link, becomes `schemaGraph`) |
| `*.samm-to-pmdco.construct.sparql` | SPARQL | Stage 2 CONSTRUCT → PMDco |
| `*.samm-to-automatce.construct.sparql` | SPARQL | Stage 2 CONSTRUCT → AutoMatCE |

---

## Live example — PA6GF30 cross-project use case

The [Cross-Project Use Case dataset](https://dataportal.material-digital.de/dataset/cross-project-use-case) at dataportal.material-digital.de demonstrates the full pipeline for **PA6GF30** (30% glass-filled polyamide 6), sourced from a Catena-X industrial dataspace participant via Eclipse Dataspace Connector (EDC).

End-to-end workflow orchestrated by `cross_project_usecase.ipynb`:

1. **Retrieve** — pull PA6GF30 JSON payload from Catena-X dataspace via EDC
2. **JSON → RDF** — RDFConverter `/api/createrdf` + Stage 1 YARRRML → `material_data_test_pa6gf30-joined.ttl` (96 KB, SAMM-aligned RDF)
3. **RDF → PMDco** — SPARQL INSERT reads from `dataGraph` (Step 2 TTL) + `schemaGraph` (SAMM aspect model) → named graph `<pmdco>` in Fuseki
4. **Export** — `pmdco-full.ttl` — standalone PMDco knowledge graph

---

## Trigger mechanism — NOT automated

!!! warning "Manual step required"
    Stage 2 is **not triggered automatically** by the pipeline. It requires a human to initiate it explicitly.

**Tool:** `pmdco-mapping-insert.html` — a browser helper page stored as a CKAN resource.

**How it works:**

1. Open the HTML file in a browser
2. Verify the Fuseki endpoint URL and the SPARQL file URL shown in the form
3. Click **Run**

Internally, the JavaScript fetches the `.sparql` file from CKAN and POSTs it to Fuseki `/$`/update` with `Content-Type: application/sparql-update`.

**Prerequisites before clicking Run:**

- `dataGraph` must already be loaded in Fuseki (via `fuseki_update` action, triggered by the Stage 1 YARRRML conversion)
- `schemaGraph` (SAMM aspect model TTL) must also be loaded in Fuseki via `fuseki_update`

Stage 1 (YARRRML → intermediate RDF → Fuseki load) is fully automated by the pipeline. Stage 2 is the deliberate manual gate.

---

## When to use this pattern vs. direct YARRRML

| Criterion | Direct YARRRML/RML | Two-stage SAMM/SPARQL CONSTRUCT |
|---|---|---|
| **Input** | Raw data (JSON, CSV, XML, RDF) | Existing RDF graph |
| **Output** | RDF knowledge graph | Reshaped / re-ontologized RDF graph |
| **Purpose** | Data → RDF lifting | Ontology → ontology transformation |
| **IRI strategy** | Template-based | SHA256 hash-based or concatenation |
| **Unit handling** | None (preserves raw values) | Regex parsing + conversion factors in VALUES table |
| **Trigger** | RDFConverter API (automated) | SPARQL engine — Fuseki or any SPARQL 1.1 endpoint |
| **When source already has an ontology** | Not ideal — must embed translation in JSONPath | Use this — CONSTRUCT handles the bridge cleanly |

**Key insight:** SPARQL CONSTRUCT is the ontology-translation layer. When source data already has an established ontology (SAMM/CX) but your target uses a different one (PMDco), CONSTRUCT handles the bridge without touching the original mapping or data. The source YARRRML mapping stays clean and portable; the ontology mapping lives in a separate, inspectable `.sparql` file.

---

## Replicating this pattern for your domain

To apply the two-stage pattern to a new SAMM aspect model:

1. **Identify** your SAMM aspect model TTL and its namespace IRI — this becomes `schemaGraph`
2. **Author a Stage 1 mapping** — YARRRML with the SAMM namespace in `prefixes`, JSONPath iterators matching the JSON payload structure; follow the [mapping authoring guide](../../guides/author-a-mapping.md)
3. **Upload Stage 1 resources to CKAN** — JSON payload, YARRRML `.yaml`, SAMM TTL (as external link resource)
4. **Load both graphs into Fuseki** — use the `fuseki_update` CKAN action for the Stage 1 TTL output, and separately for the SAMM TTL
5. **Author Stage 2 SPARQL** — copy the four-node pattern above, replace the PMDco class IRIs and QUDT units with those appropriate for your properties, add a VALUES table if mapping many properties in one query
6. **Store the `.sparql` file and `*-insert.html` helper as CKAN resources** on the same dataset

For the single-stage approach (no intermediate ontology), see [IDTA / AAS Submodel Pipeline](idta-aas.md).
