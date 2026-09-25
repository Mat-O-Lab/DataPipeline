---
title: IDTA / AAS Submodel Pattern
---

# IDTA / AAS Submodel Pattern

Single-stage YARRRML/RML transformation: Asset Administration Shell (AAS) submodel JSON maps directly to PMDco — no intermediate ontology layer required.

---

## What are AAS submodels?

The Asset Administration Shell is the IDTA/IEC 63278 standard for digital twins in Industry 4.0. An AAS organises machine-readable asset information into **submodels** — typed, hierarchical JSON documents. Each submodel contains typed submodel elements:

| Element type | Role |
|---|---|
| `SubmodelElementCollection` | Groups related properties (e.g. `MechanicalProperties`) |
| `Property` | A single typed value, identified by `idShort` |

Every element carries an `idShort` (human-readable identifier), a `valueType`, and — for `Property` — a `value`. This standardised metamodel vocabulary is what makes single-stage mapping possible.

---

## Why single-stage works

SAMM/Catena-X data is flat JSON that only gains ontological meaning when joined with its schema graph — hence two stages. AAS JSON is different: the metamodel vocabulary (`idShort`, `valueType`, `SubmodelElementCollection`) is stable and standardised across all AAS implementations. A YARRRML mapping can navigate the hierarchy directly using JSONPath iterators that filter by element type and `idShort`, emitting PMDco triples in a single pass.

| | SAMM / Catena-X | IDTA / AAS |
|---|---|---|
| **Source structure** | Flat JSON, semantics in external schema | Hierarchical JSON, semantics in metamodel |
| **Intermediate ontology needed?** | Yes — SAMM namespace stage first | No — iterate directly on `idShort` |
| **Stages** | Two (YARRRML + SPARQL CONSTRUCT) | One (YARRRML/RML) |
| **Stage 2 trigger** | Manual (browser HTML helper) | Automatic (RDFConverter pipeline) |

For the two-stage pattern and the reasoning behind it, see [SAMM / Catena-X Pipeline Pattern](samm-catena-x.md).

---

## YARRRML mapping structure

The real `InspectionDocumentsOfSteelProducts.json-to-pmdco.yaml` (published on [futurecarproduction.materialsdata.space](https://futurecarproduction.materialsdata.space/dataset/e608c577-19a8-48d5-ad5e-310ab741d11b)) maps an AAS-aligned steel inspection document JSON directly to PMDco in a single pass:

```yaml
prefixes:
  ex:   http://www.example.org/#
  pmd:  https://w3id.org/pmd/co/
  obo:  http://purl.obolibrary.org/obo/
  tto:  https://w3id.org/pmd/tto/
  qudt: https://qudt.org/schema/qudt/
  rdfs: http://www.w3.org/2000/01/rdf-schema#

mappings:
  material:
    sources: root
    s: ex:316-4401_material
    po:
      - [a, pmd:PMD_0000000]
      - [a, obo:BFO_0000040]
      - [rdfs:label, "N/A"]
      - p: obo:RO_0000086
        o:
          - mapping: tensile_strength
          - mapping: yield_strength
          - mapping: elongation_after_fracture

  tensile_strength_value:
    sources: tensile_src
    s: ex:316-4401_tensile_strength_value
    po:
      - [a, qudt:QuantityValue]
      - [rdfs:label, "Tensile Strength Mean"]
      - [qudt:numericValue, $(value)]
      - [qudt:unit, qudt:MegaPA~iri]

  fraction_carbon:
    sources: carbon_src
    s: ex:316-4401_fraction_carbon
    po:
      - [a, pmd:PMD_0025997]
      - [obo:IAO_0000039, obo:UO_0000163~iri]
      - [obo:OBI_0001937, $(value)]
```

The input JSON (`InspectionDocument_316_4401_alloy.json`) and the resulting PMDco TTL are published at [dataportal.material-digital.de — Steel Inspection Document Mapro](https://dataportal.material-digital.de/dataset/a725c4a1-a463-47f8-bad3-fe84b73ef0d3).

Key characteristics:

- **Direct PMDco output** — `po` blocks map straight to `pmd:`, `obo:`, `qudt:` — no intermediate namespace.
- **No schema graph join** — source semantics are fully captured by the JSON structure and field names; no external TTL is needed.
- **Composition as mass fractions** — `fraction_carbon` and siblings use `pmd:PMD_0025997` (mass fraction) with `obo:UO_0000163` (percent) as unit.

---

## Mapped properties (steel alloy example)

The `pattern_b_idta_aas` pattern maps:

- **Material identity** — central material resource typed as physical material object
- **Mechanical properties** — tensile strength, yield strength, elongation as PMD quality instances with QUDT values in MegaPascals
- **Composition** — PMD composition specification with element-specific mass fractions (C, Cr, Mn, Mo, Ni, N, P, Si, S) linked via OBO part-of relations

---

## Validation

After the RDFConverter produces the TTL, validate the mapping quality with `checkmapping`:

```bash
curl -X POST 'https://rdfconverter.matolab.org/api/checkmapping' \
  -G \
  --data-urlencode 'mapping_url=<your_aas_yarrrml_url>' \
  --data-urlencode 'data_url=<your_aas_json_url>'
```

Expected response:

```json
{"rules_applicable": N, "rules_skipped": 0}
```

A non-zero `rules_skipped` count indicates that some iterators found no matching elements — check your `idShort` filter expressions.

---

## When to use this pattern

Use the AAS single-stage pattern when:

- Your source data is a valid AAS submodel JSON (IEC 63278 structure with `idShort` identifiers)
- The target ontology is PMDco (or another ontology you can map to directly)
- You do not need to preserve the AAS/IDTA namespace in the output graph

Use the [two-stage SAMM pattern](samm-catena-x.md) when:

- Your source JSON conforms to a versioned SAMM/Catena-X aspect model
- You need to preserve source semantics in an intermediate named graph for provenance or downstream SPARQL queries
- The ontology translation logic is complex enough to warrant a separate SPARQL CONSTRUCT file

The full 11.9 KB YARRRML file covers mechanical properties (tensile strength, yield strength, elongation), chemical composition fractions (C, Cr, Mn, Mo, Ni, N, P, Si, S), and material identity — all in one pass.
