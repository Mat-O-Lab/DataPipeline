---
title: Add a Use Case
---

# Add a Use Case

Extend the DataStack pipeline for a new data domain. Two independent dimensions to adapt: the **source type** (what data you bring in) and the **target ontology** (what knowledge graph you produce).

---

## Two Paths

```
New use case
├── New source type
│   → Add or deploy a new extractor → produces JSON-LD → rest of pipeline unchanged
└── New target ontology
    → Design a new pattern in OntosphereIO → rest of pipeline unchanged
```

Most new use cases involve both — a new instrument format and a new ontology structure. They can be worked on independently.

---

## Path A — New Source Type

The pipeline accepts anything that produces JSON-LD or RDF. To add a new source:

**Option 1: Write a YARRRML mapping for the raw data format**

If the source is JSON or XML, write a YARRRML mapping that reads it directly (no extractor step). RDFConverter supports JSONPath and XPath iterators:

```yaml
# YARRRML mapping reading from JSON source with JSONPath iterator
mappings:
  MaterialData:
    sources:
      - ["https://example.org/data.json~jsonpath", "$.measurements[*]"]
    subjects: "ex:measurement_$(id)"
    predicates-objects:
      - predicates: pmdco:hasValue
        objects: "$(tensile_strength)"
```

See [YARRRML tutorial](https://rml.io/yarrrml/tutorial/) for JSONPath and XPath iterator syntax.

**Option 2: Deploy an existing extractor**

| Data source | Extractor | Status |
|---|---|---|
| CSV / ASC / TSV | CSVToCSVW (included in DataStack) | Production |
| OMERO microscopy server | OmeroExtractor | Early stage |
| OpenBIS ELN/LIMS | OpenBISmantic | Demonstrator |
| SQL databases | Ontop (external) | External tool |

For OmeroExtractor and OpenBISmantic, deploy the extractor service separately and wire it to CKAN by adding the respective `ckanext-*` plugin to `CKAN__PLUGINS` in `.env`.

**Option 3: Write a new extractor**

Any service that accepts a source URL and returns JSON-LD can plug into the pipeline. The JSON-LD output must be served at a publicly accessible URL for MapToMethod and RDFConverter to reach it.

After the extractor step, ckanext-csvwmapandtransform picks up the JSON-LD resource and applies the standard mapping→transform chain.

---

## Path B — New Target Ontology

The target ontology lives entirely in the YARRRML mapping and the pattern (template graph). The pipeline infrastructure is never touched.

**Step 1: Design a pattern in OntosphereIO**

[OntosphereIO](https://github.com/ThHanke/ontosphere) is the recommended pattern authoring tool — browser-based, AI-assisted, full OWL2DL reasoning and PMDCO autoshapes (SHACL). The [PMDCO pattern library](https://github.com/materialdigital/core-ontology/tree/main/patterns/) has reusable reference patterns.

Patterns are Turtle RDF files. A minimal pattern defines:
- The target ontology class (e.g., `pmdco:TensileTestResult`)
- Named individuals for each measured property
- QUDT unit annotations where applicable

**Step 2: Generate a YARRRML mapping**

Use MapToMethod with your CSVW JSON-LD and new pattern Turtle — it generates the YARRRML rules automatically. See [Author a Mapping](author-a-mapping.md).

**Step 3: Choose the right transformation path**

| Situation | Path |
|---|---|
| Source data maps directly to target ontology | YARRRML/RML (direct) — default for CSV |
| Source has its own ontology (e.g. SAMM/CX) that differs structurally from target | YARRRML/RML → intermediate Fuseki graph → SPARQL CONSTRUCT |

Use SPARQL CONSTRUCT (Path 2) only when the intermediate ontology is complex enough that mapping rules alone can't bridge the gap — it requires writing a SPARQL CONSTRUCT query stored as a CKAN resource. See [Pipeline Overview — Two Transformation Paths](../pipeline/overview.md#two-transformation-paths).

---

## Configuration Changes

When adding a new use case, these are the settings most likely to need adjustment:

| Setting | Where | What to change |
|---|---|---|
| `ckanext.csvtocsvw.formats` | `.env` / `CKANINI__` | Add the new source file extension if not already listed |
| `ckanext.csvwmapandtransform.formats` | `.env` / `CKANINI__` | Add the extractor's output format if not JSON-LD |
| `ckanext.csvwmapandtransform.mapping_strategy` | `.env` / `CKANINI__` | Use `best_match` during development, `exact` in production |
| `CKAN__PLUGINS` | `.env` | Add the new extractor plugin name |

See [Configuration Reference](../reference/configuration.md) for all keys and their `CKANINI__` env forms.

---

## When to use `best_match` vs `exact`

| Strategy | When | Risk |
|---|---|---|
| `exact` | Production — final mapping for a known data structure | May silently produce no output if no mapping fully matches |
| `best_match` | Development — iterating on mapping design | May apply a partial mapping; inspect the joined Turtle for correctness |

Switch to `exact` once `rules_skipped == 0` for your mapping.

---

## Next steps

- **Use the microservices without CKAN:** [Standalone APIs](standalone-apis.md)
- **Reference all configuration keys:** [Configuration](../reference/configuration.md)
- **Understand pipeline capabilities and limitations:** [Capability Map](../pipeline/capability-map.md)
