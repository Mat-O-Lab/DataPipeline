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

First 10 lines from a real CSVToCSVW output:

```json
// Source: https://raw.githubusercontent.com/Mat-O-Lab/CSVToCSVW/main/examples/example2-metadata.json
{
  "@context": [
    "http://www.w3.org/ns/csvw",
    {
      "oa": "http://www.w3.org/ns/oa#",
      "label": "http://www.w3.org/2000/01/rdf-schema#label",
      "xsd": "http://www.w3.org/2001/XMLSchema#",
      "qudt": "http://qudt.org/schema/qudt/",
      "dc": "http://purl.org/dc/elements/1.1/",
      "prov": "http://www.w3.org/ns/prov#",
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
# Source: https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml
prefixes: {bfo: 'http://purl.obolibrary.org/obo/', csvw: 'http://www.w3.org/ns/csvw#',
  data: 'https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json/',
  template: 'https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/LengthMeasurement.ttl/',
  owl: 'http://www.w3.org/2002/07/owl#', rdf: 'http://www.w3.org/1999/02/22-rdf-syntax-ns#',
  rdfs: 'http://www.w3.org/2000/01/rdf-schema#', xml: 'http://www.w3.org/XML/1998/namespace',
  xsd: 'http://www.w3.org/2001/XMLSchema#'}
base: http://purl.matolab.org/mseo/mappings/
sources:
  annotations: {access: 'https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json',
    iterator: '$.notes[*]', referenceFormulation: jsonpath}
```

MapToMethod generates YARRRML automatically — manual editing is only needed for complex rules. YARRRML is converted to [RML](https://rml.io/specs/rml/) by the YARRRML Parser before execution by RML Mapper.

---

## Joined Turtle (Stage 3 output)

The joined Turtle is the final knowledge graph output — data from your CSV expressed in the target ontology (e.g., PMDco). It includes:

- One subject per measurement row, typed to the target ontology class
- QUDT quantity values with unit IRIs
- PROV-O provenance tracing back to the source CSV

```turtle
# Source: https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-joined.ttl
@prefix : <https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json/> .
@prefix csvw: <http://www.w3.org/ns/csvw#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix iof: <https://spec.industrialontologies.org/ontology/core/Core/> .
@prefix iof-mat: <https://spec.industrialontologies.org/ontology/materials/Materials/> .
@prefix iof-qual: <https://spec.industrialontologies.org/ontology/qualities/> .
@prefix mod: <https://w3id.org/mod#> .
@prefix mutil: <https://github.com/Mat-O-Lab/MSEO/raw/main/domain/util/readable_bfo_iris.ttl/> .
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
