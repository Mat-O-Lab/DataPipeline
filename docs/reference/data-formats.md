---
title: Data Formats
---

# Data Formats

What format flows between each stage of the pipeline, and what you can expect at each hand-off.

## Pipeline Flow

| Stage | Input | Tool | Output | Available formats |
|---|---|---|---|---|
| **1 — Metadata creation** | CSV / TSV / ASC | CSVToCSVW `/api/annotate` | CSVW JSON-LD (W3C) | json-ld · n3 · nt · hext · trig · turtle · longturtle · xml |
| **2 — RDF conversion** | CSVW JSON-LD | CSVToCSVW `/api/rdf` | Turtle RDF | json-ld · n3 · nt · hext · trig · **turtle** · longturtle · xml |
| **3 — Semantic transform** | Turtle + YARRRML mapping | RDFConverter `/api/createrdf` | Joined Turtle (target ontology) | json-ld · n3 · nt · hext · trig · **turtle** · longturtle · xml |
| **4 — Triplestore load** | Turtle RDF | ckanext-fuseki → Fuseki | Named graph in Fuseki | SPARQL endpoint (query any serialization via `Accept` header) |

CSVToCSVW and RDFConverter share the same 8 `return_type` options — the same parameter name works for both services.

---

## CSVW JSON-LD (Stage 1 output)

W3C CSVW is a JSON-LD format that describes a CSV file's structure and annotates its columns with semantic metadata. See the [W3C CSVW primer](https://www.w3.org/TR/tabular-data-primer/) for specification.

Key elements in a CSVToCSVW output:

```json
{
  "@context": ["http://www.w3.org/ns/csvw", { "qudt": "...", "prov": "..." }],
  "@type": "http://www.w3.org/ns/csvw#TableGroup",
  "notes": [
    { "@type": "oa:Annotation", "oa:hasBody": { "@value": "PA6GF30" } }
  ],
  "tables": [{
    "tableSchema": {
      "columns": [
        { "name": "tensile_strength_MPa", "qudt:unit": { "@id": "unit:MegaPA" } }
      ]
    }
  }],
  "prov:wasGeneratedBy": { "@id": "https://csvtocsvw.matolab.org" }
}
```

| Element | Description |
|---|---|
| `notes` | Key-value metadata from rows above the data table (Open Annotation) |
| `tables[].tableSchema.columns` | One entry per data column, with QUDT unit annotation |
| `qudt:unit` | IRI from the [QUDT unit vocabulary](https://qudt.org/) — matched from column header name |
| `prov:wasGeneratedBy` | PROV-O provenance — records which service produced the metadata |

---

## YARRRML Mapping (authoring format)

YARRRML is a human-readable YAML syntax for writing RML mapping rules. See the [YARRRML spec](https://rml.io/yarrrml/spec/) and [interactive editor (Matey)](https://rml.io/yarrrml/matey/).

```yaml
prefixes:
  ex: "https://example.org/"
  pmdco: "https://example.org/pattern#"

mappings:
  TensileStrength:
    sources:
      - ["https://example.org/data.csvw.json~jsonpath", "$.tables[0].tableSchema.columns[*]"]
    subjects: "ex:result_$(name)"
    predicates-objects:
      - predicates: pmdco:hasValue
        objects:
          value: "$(tensile_strength_MPa)"
          datatype: xsd:double
```

MapToMethod generates YARRRML automatically — manual editing is only needed for complex rules. YARRRML is converted to [RML](https://rml.io/specs/rml/) by the YARRRML Parser before execution by RML Mapper.

---

## Joined Turtle (Stage 3 output)

The joined Turtle is the final knowledge graph output — data from your CSV expressed in the target ontology (e.g., PMDco). It includes:

- One subject per measurement row, typed to the target ontology class
- QUDT quantity values with unit IRIs
- PROV-O provenance tracing back to the source CSV

```turtle
@prefix pmdco: <https://github.com/materialdigital/core-ontology/tree/main/pmdco#> .
@prefix qudt:  <http://qudt.org/schema/qudt/> .
@prefix unit:  <http://qudt.org/vocab/unit/> .
@prefix prov:  <http://www.w3.org/ns/prov#> .

<https://example.org/result_23>
    a pmdco:TensileTestResult ;
    pmdco:hasTensileStrength [
        a qudt:QuantityValue ;
        qudt:numericValue "180"^^xsd:double ;
        qudt:unit unit:MegaPA
    ] ;
    prov:wasDerivedFrom <https://example.org/sample.csv> .
```

---

## Fuseki Named Graphs (Stage 4)

Fuseki loads each dataset's joined Turtle as a named graph. The graph IRI is derived from the CKAN dataset UUID. All graphs are queryable via the union default graph.

SPARQL query across all graphs in a dataset:

```sparql
SELECT ?s ?p ?o
WHERE { ?s ?p ?o }
LIMIT 100
```

Open the SPARQL endpoint via the Sparklis or YASGUI link in the CKAN dataset resource list.
