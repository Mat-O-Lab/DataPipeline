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

`return_type` options (all services): `json-ld` · `n3` · `nt` · `hext` · `trig` · `turtle` · `longturtle` · `xml`

---

## CSVToCSVW (v1.3.5)

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/api/annotate` | Annotate CSV from URL → CSVW |
| `POST` | `/api/annotate_upload` | Annotate CSV from file upload → CSVW |
| `POST` | `/api/rdf` | Convert existing CSVW to another RDF serialization |
| `GET` | `/info` | Service version and config |

**`POST /api/annotate` — Annotate CSV from URL:**

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate?return_type=json-ld" \
  -H "Content-Type: application/json" \
  -d '{"data_url": "https://github.com/Mat-O-Lab/CSVToCSVW/raw/main/examples/example2.csv",
       "encoding": "auto"}' \
  --output example2-metadata.json
```

Request body: `data_url` (URI) + `encoding` (default: `auto`, 30 options).

**`POST /api/annotate_upload` — Upload CSV directly (no public URL required):**

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/annotate_upload?return_type=json-ld" \
  -F "file=@sample.csv" \
  --output sample-metadata.json
```

**`POST /api/rdf` — Convert existing CSVW to Turtle:**

```bash
curl -X POST "https://csvtocsvw.matolab.org/api/rdf?return_type=turtle" \
  -H "Content-Type: application/json" \
  -d '{"metadata_url": "https://raw.githubusercontent.com/Mat-O-Lab/CSVToCSVW/main/examples/example-metadata.json"}' \
  --output example.ttl
```

**`GET /info`** response:

```json
{"app_name": "CSVtoCSVW", "version": "v1.3.5", "config_name": "production",
 "server": "https://csvtocsvw.matolab.org"}
```

---

## MapToMethod (v1.1.5)

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/types` | Get all `rdf:type` IRIs from a document |
| `GET` | `/api/entities` | Get named entities of specified types |
| `POST` | `/api/mapping` | Generate YARRRML mapping |
| `GET` | `/info` | Service version |

**`GET /api/types` — list entity types in a CSVW:**

```bash
curl "https://maptomethod.matolab.org/api/types?url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

Response: `["http://qudt.org/schema/qudt/DerivedUnit", "http://www.w3.org/ns/csvw#Column", ...]`

**`GET /api/entities` — list named entities in a CSVW:**

```bash
curl "https://maptomethod.matolab.org/api/entities?url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json&types=http://www.w3.org/ns/csvw%23Column"
```

Query params: `url` (URI to document) · `types` (comma-separated type URIs, default: `oa:Annotation,csvw:Column`)

Response: `{"entities": {"table-1-LengthMm": {"uri": "...", "property": "name", "text": "LengthMm", "type": "..."}, ...}, "base_namespace": "..."}`

**`POST /api/mapping` — generate YARRRML mapping:**

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
| `data_url` | yes | URL to data metadata (JSON-LD/CSVW) |
| `template_url` | yes | URL to template knowledge graph (Turtle/RDF) |
| `predicate` | yes | RDF property linking data entity to template entity |
| `map` | yes | Dict: entity-name (from `/api/entities`) → template individual IRI |
| `data_types` | no | RDF types to query from data (default: `oa:Annotation`, `csvw:Column`) |
| `template_types` | no | RDF types to query from template |
| `use_template_rowwise` | no | If true, duplicate template block per data row (default: false) |

Response: YAML file download (`Content-Disposition: attachment`) — YARRRML mapping.

**`GET /info`** response:

```json
{"name": "MapToMethod", "version": "v1.1.5", "contact": "maptomethod@matolab.org", "mode": "development"}
```

---

## RDFConverter (v1.3.3)

| Method | Path | Tag | Purpose |
|---|---|---|---|
| `POST` | `/api/createrdf` | convert | Full RDF conversion from URL-accessible data |
| `POST` | `/api/createrdfupload` | convert | Full RDF conversion from uploaded file content |
| `POST` | `/api/checkmapping` | validate | Dry-run: count applicable vs skipped rules |
| `POST` | `/api/yarrrmltorml` | convert | Convert YARRRML → RML only (no execution) |
| `POST` | `/api/yarrrmltosparql` | convert | Convert YARRRML → SPARQL SELECT query |
| `POST` | `/api/normalizeyarrrml` | convert | Normalize YARRRML nested-list `po:` blocks |
| `POST` | `/api/rdfvalidator` | validate | Validate RDF against SHACL shapes graph |
| `POST` | `/api/test` | debug | Diagnostic run with per-rule statistics |
| `GET` | `/info` | info | Service version |

**`POST /api/createrdf` — convert URL-accessible data to Turtle:**

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

**`POST /api/checkmapping` — check mapping applicability (dry-run):**

```bash
curl -X POST "https://rdfconverter.matolab.org/api/checkmapping" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

```json
{"rules_applicable": 1, "rules_skipped": 0}
```

`rules_skipped == 0` is the quality gate for CKAN automation (used by ckanext-csvwmapandtransform).

**`POST /api/test` — diagnostic run with per-rule statistics:**

```bash
curl -X POST "https://rdfconverter.matolab.org/api/test" \
  -G \
  --data-urlencode "mapping_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-map.yaml" \
  --data-urlencode "data_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-metadata.json"
```

Response includes `per_rule_statistics`, `logs`, `triple_count`, and `output_preview` — use this when debugging mapping rules.

**`POST /api/rdfvalidator` — validate RDF against SHACL shapes:**

```bash
curl -X POST "https://rdfconverter.matolab.org/api/rdfvalidator" \
  -G \
  --data-urlencode "shapes_url=https://example.org/shapes.ttl" \
  --data-urlencode "rdf_url=https://raw.githubusercontent.com/Mat-O-Lab/IOFMaterialsTutorial/main/measurements-joined.ttl"
```

Response: `{"conformance_report": "...", "shapes_graph": "..."}`

**`GET /info`** response:

```json
{"version": "v1.3.3"}
```
