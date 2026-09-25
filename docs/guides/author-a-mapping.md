---
title: Author a Mapping
---

# Author a Mapping

This guide walks through the complete mapping authoring workflow using real, publicly accessible files you can use immediately. Every API call here is reproducible — copy the curl commands as-is and you will get real responses back.

**What you will build:** A YARRRML mapping that links the column structure of a CSVW metadata file to a target ontology pattern, then validate it produces correct RDF output.

**Worked example:** The [IOFMaterialsTutorial](https://github.com/Mat-O-Lab/IOFMaterialsTutorial) — five length measurements connected to an IOF ontology pattern for a measurement process. This tutorial uses an older pattern format; treat it as illustrative for the API mechanics. For current pattern examples, see the [Ontosphere benchmark tasks](https://github.com/ThHanke/ontosphere/tree/main/benchmarks/ontoauthor-mat/).

!!! info "Where do ontology patterns come from?"
    A pattern is a small Turtle file that describes the semantic structure of a measurement type — the entities involved, their types, and how they relate. You do not write patterns during the mapping workflow; you *use* them. Patterns are authored separately using a graph editor:

    - **[Ontosphere](https://thhanke.github.io/ontosphere)** (recommended) — browser-based RDF/OWL editor with built-in OWL 2 DL reasoning, SHACL validation, and a drag-and-drop workflow template catalog. No install required. See the [materials science benchmark tasks](https://github.com/ThHanke/ontosphere/tree/main/benchmarks/ontoauthor-mat/) for worked examples (steel alloy classification, composite materials, mechanical testing patterns).
    - **[Ontopanel](https://github.com/yuechenbam/yuechenbam.github.io)** — Draw.io plugin for graphical ontology authoring; established alternative, used in published DataStack workflows ([Nasrabadi et al., 2023](https://doi.org/10.1016/j.compind.2023.104016)).

    If a pattern for your measurement type already exists in your community's ontology library (e.g. the [PMDCO pattern library](https://github.com/materialdigital/core-ontology/tree/main/patterns/)), you can use it directly — no authoring needed. If you need a new pattern, see [Add a Use Case](add-a-use-case.md) for the pattern design step.

---

## Background: three concepts you need before the first step

This workflow sits at the intersection of REST APIs (familiar territory) and the semantic web (probably not). Three terms appear throughout — here is what each one means in plain engineering terms.

**RDF and triples**

RDF (Resource Description Framework) is a data model where every fact is expressed as a three-part statement called a **triple**:

```
subject  —  predicate  —  object
```

For example: `sample-001  hasYieldStrength  450MPa` is one triple. Think of it as a row in a key-value store where the key itself is also a named relationship, not just a string. A set of triples is a graph: nodes are subjects and objects, edges are predicates.

**IRIs**

Every subject, predicate, and object in RDF is identified by an **IRI** (Internationalized Resource Identifier) — essentially a URL used as a globally unique name. `http://qudt.org/schema/qudt/DerivedUnit` is the IRI for the QUDT DerivedUnit concept. IRIs look like web addresses but are primarily used as stable, universal identifiers, not necessarily as clickable links.

**YARRRML**

YARRRML is a YAML-based mapping language — think of it as a Jinja template, but instead of rendering HTML from a data context, it renders **RDF statements from CSV rows**. Each mapping rule says: "for each row where column X equals Y, emit a triple connecting this data IRI to that ontology concept IRI." You never write YARRRML by hand in this workflow — MapToMethod generates it for you from the API calls below.

---

## Prerequisites

- A CSVW JSON-LD document at a **publicly accessible URL**
- A pattern (template graph) in Turtle format at a **publicly accessible URL**
- `curl` installed (or use the [interactive API docs](https://maptomethod.matolab.org/api/docs))

!!! note "Local or intranet URLs"
    MapToMethod and RDFConverter fetch both documents directly over HTTP. Files on your laptop or an internal network are not reachable. Serve them via a public file host (e.g. GitHub raw, a public S3 bucket, or your institution's web server) before proceeding.

!!! info "What is Turtle?"
    Turtle (`.ttl`) is a compact text format for writing RDF triples, similar to how JSON is a format for structured data. Ontology patterns are distributed as Turtle files because the format is human-readable and widely supported.

---

## The inputs

| File | URL | Role |
|---|---|---|
| `measurements.csv` | [`…/measurements.csv`](https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements.csv) | Source CSV — 5 length measurements |
| `measurements-metadata.json` | [`…/measurements-metadata.json`](https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json) | CSVW JSON-LD produced by CSVToCSVW |
| `LengthMeasurement.ttl` | [`…/LengthMeasurement.ttl`](https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/LengthMeasurement.ttl) | Ontology pattern (template graph) in Turtle |

The CSV looks like this:

```
id;length [mm]
1;3,20
2;3,25
3;3.15
4;3,20
5;3,10
```

CSVToCSVW already processed it and produced the CSVW JSON-LD. The metadata file encodes each column with its IRI and QUDT unit annotation — this is what MapToMethod reads.

The ontology pattern (`LengthMeasurement.ttl`) is a reusable template graph: a small RDF document that describes how a length measurement *should* look in semantic terms — what entities exist, what their types are, and how they relate to each other. The mapping you author will wire your CSV columns into that template.

---

## Step 1 — Understand what types your CSVW exposes

`GET /api/types` returns every unique type IRI present in the document. In RDF, every entity has a type — its class — and the type is just another IRI. Think of it as the `__class__` of each node in the graph.

Run it against the CSVW:

```bash
curl "https://maptomethod.matolab.org/api/types?url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

**Response:**

```json
[
  "http://qudt.org/schema/qudt/DerivedUnit",
  "http://www.w3.org/ns/csvw#Column",
  "http://www.w3.org/ns/csvw#TableGroup",
  "http://www.w3.org/ns/prov#Activity",
  "http://www.w3.org/ns/prov#SoftwareAgent"
]
```

`csvw#Column` is what you want to map from — those are the data column entities. (The `csvw#` prefix is shorthand for the CSVW vocabulary IRI `http://www.w3.org/ns/csvw#`.)

Now run the same call against the template pattern:

```bash
curl "https://maptomethod.matolab.org/api/types?url=https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/LengthMeasurement.ttl"
```

**Response:**

```json
[
  "http://www.w3.org/2002/07/owl#AnnotationProperty",
  "http://www.w3.org/2002/07/owl#Class",
  "http://www.w3.org/2002/07/owl#NamedIndividual",
  "http://www.w3.org/2002/07/owl#ObjectProperty",
  "http://www.w3.org/2002/07/owl#Ontology",
  "https://spec.industrialontologies.org/ontology/core/Core/MeasurementProcess",
  "https://spec.industrialontologies.org/ontology/materials/Materials/Specimen",
  "https://spec.industrialontologies.org/ontology/qualities/Length"
]
```

`owl:NamedIndividual` entries are what you map *to*. A **named individual** in an ontology pattern is a specific, uniquely named entity — like a singleton object in a config file that represents a concept (e.g. `LengthData`). These are the target "slots" in the template that your CSV columns will fill.

---

## Step 2 — List the data entities (columns)

`GET /api/entities` returns a dict of named entities grouped by type. By default it queries for `oa:Annotation` and `csvw:Column` — the two entity types that typically represent mappable data columns.

```bash
curl "https://maptomethod.matolab.org/api/entities?url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

**Response:**

```json
{
  "entities": {
    "table-1-GID": {
      "uri": "https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/measurements.csv/table-1-GID",
      "property": "name",
      "text": "GID",
      "type": "http://www.w3.org/ns/csvw#Column"
    },
    "table-1-Unnamed0": {
      "uri": "https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/measurements.csv/table-1-Unnamed0",
      "property": "name",
      "text": "Unnamed0",
      "type": "http://www.w3.org/ns/csvw#Column"
    },
    "table-1-LengthMm": {
      "uri": "https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/measurements.csv/table-1-LengthMm",
      "property": "name",
      "text": "LengthMm",
      "type": "http://www.w3.org/ns/csvw#Column"
    }
  },
  "base_namespace": "https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json/"
}
```

Three columns: `GID` (row identifier, suppress output), `Unnamed0` (row index), and **`table-1-LengthMm`** — the measurement column we want to map.

The key to use in the `map` dict is the entity name: **`table-1-LengthMm`**. The `uri` field is the full IRI for that column — its globally unique identity in the graph.

---

## Step 3 — List the template entities

Run `/api/entities` on the pattern with `owl:NamedIndividual` as the type filter to see which named slots the template exposes:

```bash
curl "https://maptomethod.matolab.org/api/entities?url=https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/LengthMeasurement.ttl&types=http://www.w3.org/2002/07/owl%23NamedIndividual"
```

The template defines a `Specimen` with a `SpecimenLength` quality connected to a `LengthMeasurementProcess`. The named individual for the length data output is `LengthData` — this is the IRI the mapping will link your column to.

In other words: after the mapping runs, every row in your CSV will produce a triple connecting that row's column IRI to the `LengthData` concept defined in the ontology pattern.

---

## Step 4 — Build the `map` dict

The `map` dict pairs each data column name to the IRI of the template slot it should fill. This is the core of the mapping: "column X in my CSV corresponds to concept Y in the ontology pattern."

For the length measurement example:

```json
{
  "table-1-LengthMm": "https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/LengthMeasurement.ttl/LengthData"
}
```

- **Key:** the entity name from `/api/entities` output (`table-1-LengthMm`)
- **Value:** the IRI of the template individual — `base_namespace` of the template + individual name

!!! tip "Naming is exact and case-sensitive"
    The key must match the entity name from `/api/entities` character-for-character. A mismatch causes `rules_skipped > 0` in the validation step.

---

## Step 5 — Generate the YARRRML mapping

Recall from the background section: YARRRML is a YAML recipe that tells RDFConverter how to convert each CSV column into an RDF statement. You do not write it by hand — `POST /api/mapping` generates it from your `map` dict.

The `predicate` field in the request body is the **relationship IRI** — the middle part of the triple (`subject — predicate — object`). It specifies *how* the column connects to the template individual (e.g. "is a resource of", "measures", "is part of"). In this example, `RO_0010002` (`isResourceOf` from the BFO relations ontology) means "this column data is a resource of the LengthData process output."

```bash
curl -X POST "https://maptomethod.matolab.org/api/mapping" \
  -H "Content-Type: application/json" \
  -d '{
    "data_url":     "https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json",
    "template_url": "https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/LengthMeasurement.ttl",
    "predicate":    "http://purl.obolibrary.org/obo/RO_0010002",
    "map": {
      "table-1-LengthMm": "https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/LengthMeasurement.ttl/LengthData"
    }
  }' \
  --output my-mapping.yaml
```

The response is a YARRRML file. The actual generated mapping for this example:

```yaml
prefixes:
  bfo: 'http://purl.obolibrary.org/obo/'
  csvw: 'http://www.w3.org/ns/csvw#'
  data: 'https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json/'
  template: 'https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/LengthMeasurement.ttl/'
  xsd: 'http://www.w3.org/2001/XMLSchema#'
base: http://purl.matolab.org/mseo/mappings/
sources:
  columns:
    access: 'https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json'
    iterator: '$.tables[*].tableSchema.columns[*]'
    referenceFormulation: jsonpath
use_template_rowwise: 'false'
mappings:
  LengthData:
    sources: [columns]
    s: $(@id)
    condition:
      function: equal
      parameters:
        - [str1, $(name)]
        - [str2, table-1-LengthMm]
    po:
      - ['http://purl.obolibrary.org/obo/RO_0010002', 'template:LengthData~iri']
```

Reading this like a Jinja template: for each column object in the JSON where `name == "table-1-LengthMm"`, emit one triple: `<column IRI>  isResourceOf  <template:LengthData>`. The `po:` block is the predicate-object pair — the last two thirds of the triple.

For the full [YARRRML specification](https://rml.io/yarrrml/spec/), and to create or edit mappings interactively, use the [Matey online editor](https://rml.io/yarrrml/matey/). The [YARRRML tutorial](https://rml.io/yarrrml/tutorial/) covers source types, iterators, and condition functions.

---

## Step 6 — Validate: check that all rules match

Upload the mapping to a public URL (e.g. your own GitHub repo or a Gist), then call `/api/checkmapping`:

```bash
curl -X POST "https://rdfconverter.matolab.org/api/checkmapping" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

**Response (all rules matched):**

```json
{
  "rules_applicable": 1,
  "rules_skipped": 0
}
```

`rules_skipped == 0` is the quality gate. Every mapping rule found a matching column in the data. If `rules_skipped > 0`, at least one column name in the `map` dict did not match any column in the CSVW — see [Troubleshooting](#troubleshooting) below.

---

## Step 7 — Run a test conversion

Before uploading to CKAN, verify the output looks correct using `/api/test`:

```bash
curl -X POST "https://rdfconverter.matolab.org/api/test" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

The response includes a `triple_count` (how many RDF statements were produced), per-rule statistics, and a preview of the first statements produced. You should see one triple per measurement row linking the column IRI to `template:LengthData`.

---

## Step 8 — Produce the full joined RDF

Run the actual conversion using `/api/createrdf`:

```bash
curl -X POST "https://rdfconverter.matolab.org/api/createrdf?return_type=turtle" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json" \
  --output measurements-joined.ttl
```

The resulting Turtle (from the real IOFMaterialsTutorial run):

```turtle
@prefix iof:      <https://spec.industrialontologies.org/ontology/core/Core/> .
@prefix iof-mat:  <https://spec.industrialontologies.org/ontology/materials/Materials/> .
@prefix iof-qual: <https://spec.industrialontologies.org/ontology/qualities/> .
@prefix qudt:     <http://qudt.org/schema/qudt/> .
@prefix qunit:    <http://qudt.org/vocab/unit/> .
@prefix prov:     <http://www.w3.org/ns/prov#> .

<…/table-1-LengthMm>
    iof:isResourceOf <…/LengthMeasurement.ttl/LengthData> ;
    prov:wasDerivedFrom <…/measurements-metadata.json> .
```

Reading this as plain English: the `table-1-LengthMm` column entity **is a resource of** the `LengthData` concept in the IOF pattern, and it **was derived from** the CSVW metadata file. Two triples — each with a subject (the column IRI), a predicate (the relationship), and an object (the target IRI). This is what the mapping produced.

---

## Step 9 — Upload to CKAN

1. In CKAN, open the dataset that should hold the mapping (or create a new one)
2. **Resources → Add Resource → Upload File** — upload `my-mapping.yaml`
3. Set format to `YAML`
4. Add this dataset to the `mappings` group (Dataset → **Groups** tab → add `mappings`)

All future CSV uploads whose CSVW columns match `table-1-LengthMm` exactly will now produce a joined Turtle automatically, without any user action.

---

## Real-world examples

These are complete, working pipeline outputs you can inspect directly:

| Dataset | CSVW | Mapping | Joined Turtle |
|---|---|---|---|
| IOFMaterialsTutorial (length) | [measurements-metadata.json](https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json) | [measurements-map.yaml](https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml) | [measurements-joined.ttl](https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-joined.ttl) |
| BAMresearch DF-TEM-PAW (TEM) | [detection\_runs-metadata.json](https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs-metadata.json) | [detection\_runs-map.yaml](https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs-map.yaml) | [detection\_runs-joined.ttl](https://raw.githubusercontent.com/BAMresearch/DF-TEM-PAW/main/detection_runs-joined.ttl) |

---

## Troubleshooting {#troubleshooting}

`rules_skipped > 0` means one or more conditions in the mapping did not find a match in the data. In YARRRML terms: the `equal` condition in the mapping rule compared the column name against a string and found no row where they matched.

**Step 1 — compare entity names exactly**

Re-run `/api/entities` on your CSVW and compare the entity names character-by-character against what is in your `map` dict. YAML keys are case-sensitive.

**Step 2 — switch to `best_match` temporarily**

In your CKAN DataStack `.env`:

```bash
CKANINI__CKANEXT__CSVWMAPANDTRANSFORM__MAPPING_STRATEGY=best_match
```

`best_match` selects the highest-rated partial match so you can see partial output and inspect which triples are missing. Switch back to `exact` for production.

**Step 3 — use `/api/test` for per-rule diagnostics**

```bash
curl -X POST "https://rdfconverter.matolab.org/api/test" \
  -G \
  --data-urlencode "mapping_url=https://example.org/my-mapping.yaml" \
  --data-urlencode "data_url=https://example.org/my-data.csvw.json"
```

The `per_rule_statistics` in the response shows exactly which conditions matched and which did not, along with the values that were compared.

---

*Next: [Standalone APIs](standalone-apis.md) — run the same workflow as a shell script pipeline without CKAN.*
