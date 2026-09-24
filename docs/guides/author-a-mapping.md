---
title: Author a Mapping
---

# Author a Mapping

This guide walks through the complete mapping authoring workflow using real, publicly accessible files you can use immediately. Every API call here is reproducible — copy the curl commands as-is and you will get real responses back.

**What you will build:** A YARRRML mapping that links the column structure of a CSV CSVW metadata file to a target ontology pattern, then validate it produces correct RDF triples.

**Worked example:** The [IOFMaterialsTutorial](https://github.com/Mat-O-Lab/IOFMaterialsTutorial) — five length measurements connected to an IOF ontology pattern for a measurement process.

---

## Prerequisites

- A CSVW JSON-LD document at a **publicly accessible URL**
- A pattern (template graph) in Turtle at a **publicly accessible URL**
- `curl` installed (or use the [interactive API docs](https://maptomethod.matolab.org/api/docs))

!!! note "Local or intranet URLs"
    MapToMethod and RDFConverter fetch both documents directly over HTTP. Files on your laptop or an internal network are not reachable. Serve them via a public file host (e.g. GitHub raw, a public S3 bucket, or your institution's web server) before proceeding.

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

---

## Step 1 — Understand what types your CSVW exposes

`GET /api/types` returns every unique `rdf:type` IRI present in the document. Run it against the CSVW:

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

`csvw#Column` is what you want to map from — those are the data column entities.

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

`owl:NamedIndividual` — the named individuals in the pattern are the entities you map *to*.

---

## Step 2 — List the data entities (columns)

`GET /api/entities` returns a dict of named entities by type. By default it queries for `oa:Annotation` and `csvw:Column`.

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

The key to use in the `map` dict is the entity name: **`table-1-LengthMm`**.

---

## Step 3 — List the template entities

Run `/api/entities` on the pattern with `owl:NamedIndividual` as the type filter to see what named individuals the template exposes:

```bash
curl "https://maptomethod.matolab.org/api/entities?url=https://github.com/Mat-O-Lab/IOFMaterialsTutorial/raw/main/LengthMeasurement.ttl&types=http://www.w3.org/2002/07/owl%23NamedIndividual"
```

The template defines a `Specimen` with a `SpecimenLength` quality connected to a `LengthMeasurementProcess`. The named individual for the length data output is `LengthData` — this is the entity IRI the mapping will link to.

---

## Step 4 — Build the `map` dict

The `map` dict pairs each data column name to the IRI of the template individual it should map to. For the length measurement example:

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

`POST /api/mapping` with the full request body:

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

`rules_skipped == 0` is the quality gate. Every rule found a matching column. If `rules_skipped > 0`, see [Troubleshooting](#troubleshooting) below.

---

## Step 7 — Run a test conversion

Before uploading to CKAN, verify the output looks correct using `/api/test`:

```bash
curl -X POST "https://rdfconverter.matolab.org/api/test" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

The response includes `triple_count`, per-rule statistics, and a preview of the first triples produced. You should see one triple per measurement row linking the column IRI to `template:LengthData`.

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

Each measurement column is now linked to the `LengthData` individual in the IOF pattern graph via the `iof:isResourceOf` relation (BFO `RO_0010002`).

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

`rules_skipped > 0` means one or more conditions in the mapping did not find a match in the data.

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
