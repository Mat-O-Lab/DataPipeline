---
title: API Endpoints
---

# API Endpoints

Public microservice endpoints for standalone use. All services expose interactive documentation at `/api/docs`.

| Service | Base URL | Interactive docs | Version |
|---|---|---|---|
| CSVToCSVW | https://csvtocsvw.matolab.org | [/api/docs](https://csvtocsvw.matolab.org/api/docs) | v1.3.5 |
| MapToMethod | https://maptomethod.matolab.org | [/api/docs](https://maptomethod.matolab.org/api/docs) | v1.1.5 |
| RDFConverter | https://rdfconverter.matolab.org | [/api/docs](https://rdfconverter.matolab.org/api/docs) | v1.3.3 |

`return_type` controls the serialization format of the output graph. Accepted values: `json-ld` · `n3` · `nt` · `hext` · `trig` · `turtle` · `longturtle` · `xml`. See [Semantic Foundation](../pipeline/semantic-foundation.md) for what these formats mean.

---

## CSVToCSVW (v1.3.5)

Annotates CSV files with schema metadata to produce machine-readable CSVW JSON-LD descriptors.

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/api/annotate` | Annotate CSV from URL → CSVW descriptor |
| `POST` | `/api/annotate_upload` | Annotate CSV from file upload → CSVW descriptor |
| `POST` | `/api/rdf` | Re-serialize an existing CSVW descriptor to a different output format |
| `GET` | `/info` | Service version and config |

**`POST /api/annotate` — annotate CSV from URL:**

Query param: `return_type` (output format, default: `json-ld`).

Request body:

| Field | Required | Description |
|---|---|---|
| `data_url` | yes | Public URL of the CSV file |
| `encoding` | no | Character encoding (default: `auto`, 30 options) |

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d '{"data_url": "https://github.com/Mat-O-Lab/CSVToCSVW/raw/main/examples/example2.csv",
       "encoding": "auto"}' \
  --output example2-metadata.json
```

Response: CSVW JSON-LD descriptor file (download).

**`POST /api/annotate_upload` — annotate CSV from file upload (no public URL required):**

Query param: `return_type` (output format, default: `json-ld`).

Request: `multipart/form-data` with a single `file` field containing the CSV.

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate_upload?return_type=json-ld" \
  -F "file=@sample.csv" \
  --output sample-metadata.json
```

Response: CSVW JSON-LD descriptor file (download).

**`POST /api/rdf` — re-serialize CSVW to a different output format:**

Query param: `return_type` (target format, default: `turtle`).

Request body:

| Field | Required | Description |
|---|---|---|
| `metadata_url` | yes | Public URL of an existing CSVW descriptor |

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/rdf?return_type=turtle" \
  -H "Content-Type: application/json" \
  -d '{"metadata_url": "https://raw.githubusercontent.com/Mat-O-Lab/CSVToCSVW/main/examples/example-metadata.json"}' \
  --output example.ttl
```

Response: graph file in the requested format (download). See [Semantic Foundation](../pipeline/semantic-foundation.md) for format details.

**`GET /info`** response:

```json
{"app_name": "CSVtoCSVW", "version": "v1.3.5", "config_name": "production",
 "server": "https://csvtocsvw.matolab.org"}
```

---

## MapToMethod (v1.1.5)

Inspects a CSVW descriptor to discover its typed entities, then generates a YARRRML mapping file from those entities to a template. See [Semantic Foundation](../pipeline/semantic-foundation.md) for an explanation of YARRRML mappings.

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/types` | List all typed entity categories found in a CSVW descriptor |
| `GET` | `/api/entities` | List named entities of specified categories |
| `POST` | `/api/mapping` | Generate a YARRRML mapping file |
| `GET` | `/info` | Service version |

**`GET /api/types` — list entity categories in a CSVW descriptor:**

Query param: `url` (public URL of a CSVW descriptor).

```bash
curl "https://maptomethod.matolab.org/api/types?url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

Response: JSON array of type URIs found in the descriptor. These URIs identify the semantic categories of entities (e.g. table columns, measurement units). See [Semantic Foundation](../pipeline/semantic-foundation.md) for what these categories represent.

```json
["http://qudt.org/schema/qudt/DerivedUnit", "http://www.w3.org/ns/csvw#Column", ...]
```

**`GET /api/entities` — list named entities in a CSVW descriptor:**

Query params:

| Param | Required | Description |
|---|---|---|
| `url` | yes | Public URL of a CSVW descriptor |
| `types` | no | Comma-separated type URIs to filter by (default: `oa:Annotation,csvw:Column`) |

```bash
curl "https://maptomethod.matolab.org/api/entities?url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json&types=http://www.w3.org/ns/csvw%23Column"
```

Response: `entities` map (keyed by entity name) plus the `base_namespace` for the descriptor. Each entry contains `uri`, `property`, `text`, and `type`. The entity names (e.g. `table-1-LengthMm`) are used as keys in the `map` field of `/api/mapping`.

```json
{
  "entities": {
    "table-1-LengthMm": {"uri": "...", "property": "name", "text": "LengthMm", "type": "..."}
  },
  "base_namespace": "..."
}
```

**`POST /api/mapping` — generate a YARRRML mapping file:**

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

Request body fields:

| Field | Required | Description |
|---|---|---|
| `data_url` | yes | URL to CSVW descriptor (from CSVToCSVW output) |
| `template_url` | yes | URL to the pattern graph file the data will be mapped into |
| `predicate` | yes | URI of the relationship linking a data entity to its template counterpart — see [Semantic Foundation](../pipeline/semantic-foundation.md) |
| `map` | yes | Dict: entity name from `/api/entities` → individual IRI from the template |
| `data_types` | no | Entity categories to query from the data (default: `oa:Annotation`, `csvw:Column`) |
| `template_types` | no | Entity categories to query from the template |
| `use_template_rowwise` | no | If `true`, duplicate the template block once per data row (default: `false`) |

Response: YAML file download (`Content-Disposition: attachment`) — YARRRML mapping ready to pass to RDFConverter.

**`GET /info`** response:

```json
{"name": "MapToMethod", "version": "v1.1.5", "contact": "maptomethod@matolab.org", "mode": "development"}
```

---

## RDFConverter (v1.3.3)

Executes YARRRML mappings to produce a joined output graph, and provides utilities for validating, inspecting, and transforming mapping files. See [Semantic Foundation](../pipeline/semantic-foundation.md) for an explanation of YARRRML, RML, SPARQL, and graph output formats.

| Method | Path | Tag | Purpose |
|---|---|---|---|
| `POST` | `/api/createrdf` | convert | Execute mapping against URL-accessible data → output graph |
| `POST` | `/api/createrdfupload` | convert | Execute mapping against an uploaded data file → output graph |
| `POST` | `/api/checkmapping` | validate | Dry-run: count applicable vs skipped mapping rules |
| `POST` | `/api/yarrrmltorml` | convert | Translate YARRRML to RML (no execution) |
| `POST` | `/api/yarrrmltosparql` | convert | Translate YARRRML to a SPARQL SELECT query |
| `POST` | `/api/normalizeyarrrml` | convert | Normalize YARRRML nested-list `po:` blocks |
| `POST` | `/api/rdfvalidator` | validate | Validate a graph against a SHACL shapes file |
| `POST` | `/api/test` | debug | Diagnostic run with per-rule statistics |
| `GET` | `/info` | info | Service version |

**`POST /api/createrdf` — execute mapping against URL-accessible data:**

Query param: `return_type` (output format, default: `turtle`).

Query params (also accepted as request body):

| Param | Required | Description |
|---|---|---|
| `mapping_url` | yes | Public URL of the YARRRML mapping file |
| `data_url` | yes | Public URL of the CSVW descriptor (input data) |

```bash
curl -X POST "https://rdfconverter.matolab.org/api/createrdf?return_type=turtle" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json" \
  --output measurements-joined.ttl
```

Response schema:

```json
{
  "filename": "measurements-metadata-joined.ttl",
  "graph": "... Turtle string ...",
  "num_mappings_applied": 1,
  "num_mappings_skipped": 0
}
```

`num_mappings_skipped == 0` means every mapping rule matched data successfully.

**`POST /api/createrdfupload` — execute mapping against an uploaded data file:**

Use when your data file is not at a public URL.

Query param: `return_type` (output format, default: `turtle`). Query param: `mapping_url` (public URL of YARRRML mapping file, required).

Request: `multipart/form-data` with a `file` field containing the CSVW descriptor.

```bash
curl -X POST "https://rdfconverter.matolab.org/api/createrdfupload?return_type=turtle" \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  -F "file=@measurements-metadata.json" \
  --output measurements-joined.ttl
```

Response: same schema as `/api/createrdf` — `filename`, `graph`, `num_mappings_applied`, `num_mappings_skipped`.

**`POST /api/checkmapping` — check mapping applicability (dry-run):**

Same params as `/api/createrdf` but executes no conversion. Use this to validate a mapping before running the pipeline.

```bash
curl -X POST "https://rdfconverter.matolab.org/api/checkmapping" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

```json
{"rules_applicable": 1, "rules_skipped": 0}
```

`rules_skipped == 0` is the quality gate used by the CKAN automation (ckanext-csvwmapandtransform) before triggering a full conversion.

**`POST /api/yarrrmltorml` — translate YARRRML to RML:**

Converts a YARRRML mapping file to its lower-level RML representation without executing it. Useful for debugging or passing to RML-native tooling.

Query param: `mapping_url` (public URL of the YARRRML file).

```bash
curl -X POST "https://rdfconverter.matolab.org/api/yarrrmltorml" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --output measurements-map.rml.ttl
```

Response: RML file (Turtle serialization, download). See [Semantic Foundation](../pipeline/semantic-foundation.md) for the relationship between YARRRML and RML.

**`POST /api/yarrrmltosparql` — translate YARRRML to a SPARQL SELECT query:**

Converts a YARRRML mapping to an equivalent SPARQL SELECT query. Useful for inspecting what the mapping would retrieve or for integrating with SPARQL-native tooling.

Query param: `mapping_url` (public URL of the YARRRML file).

```bash
curl -X POST "https://rdfconverter.matolab.org/api/yarrrmltosparql" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --output measurements-query.sparql
```

Response: SPARQL SELECT query (text file, download). See [Semantic Foundation](../pipeline/semantic-foundation.md) for an introduction to SPARQL.

**`POST /api/normalizeyarrrml` — normalize YARRRML `po:` blocks:**

Rewrites nested-list predicate-object (`po:`) blocks in a YARRRML file into their normalized form. Run this before passing a hand-authored mapping to `/api/createrdf` if you encounter parse errors.

Query param: `mapping_url` (public URL of the YARRRML file).

```bash
curl -X POST "https://rdfconverter.matolab.org/api/normalizeyarrrml" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --output measurements-map-normalized.yaml
```

Response: normalized YARRRML file (YAML, download).

**`POST /api/rdfvalidator` — validate a graph against SHACL shapes:**

Query params: `shapes_url` (public URL of SHACL shapes file) · `rdf_url` (public URL of the graph to validate).

```bash
curl -X POST "https://rdfconverter.matolab.org/api/rdfvalidator" \
  -G \
  --data-urlencode "shapes_url=https://example.org/shapes.ttl" \
  --data-urlencode "rdf_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-joined.ttl"
```

Response: `{"conformance_report": "...", "shapes_graph": "..."}` — the conformance report is a human-readable summary of constraint violations (if any).

**`POST /api/test` — diagnostic run with per-rule statistics:**

Same params as `/api/createrdf`. Returns detailed per-rule output instead of the final graph — use this when debugging why a mapping rule is being skipped.

```bash
curl -X POST "https://rdfconverter.matolab.org/api/test" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

Response includes `per_rule_statistics`, `logs`, `triple_count`, and `output_preview`.

**`GET /info`** response:

```json
{"version": "v1.3.3"}
```
