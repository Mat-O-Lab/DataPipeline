---
title: Data Formats
---

# Data Formats

What format flows between each stage of the pipeline, and what you can expect at each hand-off.

If you are already comfortable with REST APIs and JSON but new to the semantic web, start here: the pipeline is a series of format transformations. Each stage has a well-defined input and output, and each service lets you choose the serialization format via a `return_type` parameter.

## Pipeline Flow

| Stage | Input | Tool | Output | Available formats |
|---|---|---|---|---|
| **1 — Metadata creation** | CSV / TSV / ASC | CSVToCSVW `/api/annotate` | CSVW JSON-LD (W3C) | json-ld · n3 · nt · hext · trig · turtle · longturtle · xml |
| **2 — RDF conversion** | CSVW JSON-LD | CSVToCSVW `/api/rdf` | Turtle RDF | json-ld · n3 · nt · hext · trig · **turtle** · longturtle · xml |
| **3 — Semantic transform** | Turtle + YARRRML mapping | RDFConverter `/api/createrdf` | Joined Turtle (target ontology) | json-ld · n3 · nt · hext · trig · **turtle** · longturtle · xml |
| **4 — Triplestore load** | Turtle RDF | ckanext-fuseki → Fuseki | Named graph in Fuseki | SPARQL endpoint (query any serialization via `Accept` header) |

CSVToCSVW and RDFConverter share the same 8 `return_type` options — the same parameter name works for both services:
`json-ld`, `n3`, `nt`, `hext`, `trig`, `turtle`, `longturtle`, `xml`.

---

## CSVW JSON-LD (Stage 1 output)

**What it is:** Think of CSVW as a JSON sidecar file for a CSV — it sits next to the CSV and annotates each column with machine-readable metadata: data types, measurement units, and links to ontology terms. It is written in JSON-LD (see below), which means standard JSON with extra `@context` keys that tie your column names to shared, globally-unique identifiers.

The [W3C CSVW primer](https://www.w3.org/TR/tabular-data-primer/) covers the full specification.

**Produced by:** CSVToCSVW `/api/annotate` — call it with a CSV URL and it returns a CSVW JSON-LD document.

**Consumed by:** CSVToCSVW `/api/rdf` — takes the CSVW document and converts the tabular data into RDF triples.

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

## Turtle RDF (Stages 2 and 3 output)

**What it is:** Turtle is the human-readable format for RDF — the same relationship to RDF that YAML has to JSON. Where JSON-LD embeds semantic metadata inside standard JSON structure, Turtle writes RDF triples directly as `subject predicate object .` statements with a clean prefix-shorthand syntax. If you need to inspect or diff the semantic graph output by hand, Turtle is the format to use.

RDF triples are the core data model: every fact is expressed as three parts — a subject (the thing), a predicate (the relationship), and an object (the value or related thing). A collection of triples is a graph.

**Produced by:** CSVToCSVW `/api/rdf` (Stage 2), RDFConverter `/api/createrdf` (Stage 3). Both default to Turtle when no `return_type` is specified.

**Consumed by:** RDFConverter `/api/createrdf` (takes Stage 2 Turtle as one of its inputs), Fuseki (Stage 4 load).

Example from a joined output, showing prefixes and a measurement triple:

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

Each `@prefix` line is a namespace shorthand — `iof:SomeConcept` expands to `https://spec.industrialontologies.org/ontology/core/Core/SomeConcept`. The actual data triples follow the prefix block.

---

## JSON-LD (alternative serialization)

**What it is:** JSON with semantic annotations — standard JSON structure with extra `@context` keys that link your data to shared ontology terms. If you need to process pipeline output in Python or JavaScript without an RDF library, requesting `return_type=json-ld` gives you a JSON document you can traverse with normal dict/object access. The `@context` block tells a semantic-aware tool what each key means globally, but you can ignore that block and treat the rest as plain JSON.

**Produced by:** Any pipeline service (CSVToCSVW, RDFConverter) when called with `return_type=json-ld`.

**Consumed by:** Any JSON parser — or an RDF library (e.g., `rdflib` in Python) that understands JSON-LD for further graph processing.

To request JSON-LD from CSVToCSVW stage 2:

```http
POST /api/rdf
Content-Type: application/json

{
  "csv_url": "https://example.org/data.csv",
  "csvw_url": "https://example.org/data-metadata.json",
  "return_type": "json-ld"
}
```

---

## RDF/XML and other serializations

RDF/XML (`return_type=xml`) is the original W3C serialization for RDF — verbose but widely supported by older tooling. The pipeline supports it as a `return_type` option on all services but it is not recommended for human inspection. Use Turtle for readability and JSON-LD for programmatic consumption.

The full list of `return_type` values supported by both CSVToCSVW and RDFConverter:

| `return_type` | Format | Notes |
|---|---|---|
| `turtle` | Turtle (.ttl) | Default; human-readable |
| `longturtle` | Turtle with full IRIs | No prefix shorthand |
| `json-ld` | JSON-LD (.jsonld) | Easiest to process in Python/JS |
| `n3` | Notation3 (.n3) | Turtle superset with extra features |
| `nt` | N-Triples (.nt) | One triple per line; easy to grep |
| `hext` | HexTuples | Newline-delimited JSON arrays |
| `trig` | TriG (.trig) | Turtle with named graph support |
| `xml` | RDF/XML (.rdf) | Legacy; widely supported |

---

## YARRRML Mapping (Stage 3 input, authoring format)

**What it is:** YARRRML is a human-readable YAML syntax for writing RML mapping rules. Think of it as a declarative transformation spec: you describe which columns in your CSVW map to which classes and properties in the target ontology. MapToMethod generates YARRRML automatically — manual editing is only needed for complex or custom rules.

See the [YARRRML spec](https://rml.io/yarrrml/spec/) and the [interactive editor (Matey)](https://rml.io/yarrrml/matey/).

**Produced by:** MapToMethod `/api/createmap` (automated), or hand-authored.

**Consumed by:** RDFConverter `/api/createrdf` — the service converts YARRRML to [RML](https://rml.io/specs/rml/) internally before executing the mapping.

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

The `prefixes` block defines namespace shorthands (same idea as Turtle `@prefix`). The `sources` block points to the CSVW JSON-LD document that provides the input data. Mapping rules referencing these sources follow in the `mappings` section.

---

## Fuseki Named Graphs (Stage 4)

Fuseki loads each dataset's joined Turtle as a named graph. The graph IRI is derived from the CKAN dataset UUID. All graphs are queryable via the union default graph.

SPARQL is a query language for RDF graphs — think of it as SQL for triples. A minimal query to inspect what landed:

```sparql
SELECT ?s ?p ?o
WHERE { ?s ?p ?o }
LIMIT 100
```

Open the SPARQL endpoint via the Sparklis or YASGUI link in the CKAN dataset resource list.
