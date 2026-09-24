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

The mapping uses multiple JSONPath iterators. Each iterator selects a specific `SubmodelElementCollection` by `idShort` and emits one group of PMDco triples. The snippet below is illustrative of the real pattern applied to a steel alloy AAS submodel.

```yaml
# illustrative example — structure matches the AAS single-stage pattern described in
# docs/specs/samm-idta-pipeline-patterns.json, pattern_b_idta_aas
prefixes:
  rr:   'http://www.w3.org/ns/r2rml#'
  rml:  'http://semweb.mmlab.be/ns/rml#'
  ql:   'http://semweb.mmlab.be/ns/ql#'
  pmd:  'https://w3id.org/pmd/co/'
  qudt: 'http://qudt.org/schema/qudt/'
  unit: 'http://qudt.org/vocab/unit/'
  xsd:  'http://www.w3.org/2001/XMLSchema#'

sources:
  tensileStrength:
    access: '$(data_url)'
    referenceFormulation: jsonpath
    # Filter: pick the Property element whose idShort is 'TensileStrength'
    iterator: "$.submodelElements[?(@.idShort=='MechanicalProperties')].value[?(@.idShort=='TensileStrength')]"

mappings:
  TensileStrengthQuality:
    sources: [tensileStrength]
    s: pmd:quality_tensile_$(value)
    po:
      - [a, pmd:TensileStrength]
      - [qudt:numericValue, $(value), xsd:decimal]
      - [qudt:unit, unit:MegaPascal]
```

Key characteristics:

- **Iterator per property** — one JSONPath filter per `idShort` keeps each mapping rule focused and readable.
- **No schema graph join** — the `idShort` values are stable across AAS instances; no external TTL is needed to resolve their semantics.
- **Direct PMDco output** — the `po` block maps straight to the target ontology. There is no intermediate CX or SAMM namespace.

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

!!! info "Detailed walkthrough coming"
    A step-by-step walkthrough with a real AAS submodel download, full YARRRML file, and Fuseki query is planned. In the meantime, the [SAMM / Catena-X page](samm-catena-x.md) provides the most complete end-to-end example of the DataStack pipeline, including the Stage 1 YARRRML structure that the AAS pattern mirrors.
